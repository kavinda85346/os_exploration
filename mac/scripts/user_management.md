### User Management

##### This script is used for user management.

##### Steps involved in the process!

1. Create local users.
2. Assigning permissions to users.
3. Reset user passwords.
4. Locking users.
5. Delete local users.
6. Check active sessions.
7. Logoff users.

```
#!/usr/bin/env zsh
# user_flow.zsh - sequential user management automation (ZSH / macOS)
# Run as root or with sudo. Default: DRY_RUN=true
set -euo pipefail

DRY_RUN=true   # <-- set to false to actually perform actions
USERS=("demo_user1" "demo_user2")
PASSWORD="DemoP@ssw0rd123!"
GROUPS=("admin")  # macOS commonly uses 'admin' for sudo privileges

echocmd() { echo "+ $*"; }
run() {
  if [[ "$DRY_RUN" == "true" ]]; then
    echocmd "$*"
  else
    echocmd "$*"
    eval "$@"
  fi
}

if [[ $(id -u) -ne 0 ]]; then
  echo "WARNING: Run as root/with sudo for full effect."
fi

# Step 1. Create local users.
echo "Commencing local users creating process!"
for u in "${USERS[@]}"; do
  if id "$u" &>/dev/null; then
    echo "User $u exists."
  else
    # Create user with sysadminctl (macOS 10.13+)
    run "sysadminctl -addUser $u -fullName $u -password $PASSWORD || dscl . -create /Users/$u"
    # ensure home dir
    run "createhomedir -c -u $u >/dev/null 2>&1 || true"
  fi
done
echo "Ending local users creating process!"

# Step 2. Assigning permissions to users.
echo "Commencing user permission assiging process!"
for u in "${USERS[@]}"; do
  for g in "${GROUPS[@]}"; do
    if dscl . -read /Groups/$g >/dev/null 2>&1; then
      run "dscl . -append /Groups/$g GroupMembership $u || dscl . -append /Groups/$g user $u"
    else
      echo "Group $g not found; skipping."
    fi
  done
done
echo "Ending user permission assiging process!"

# Step 3. Reset user passwords.
echo "Commencing user password reseting process!"
for u in "${USERS[@]}"; do
  if id "$u" &>/dev/null; then
    run "dscl . -passwd /Users/$u $PASSWORD || sysadminctl -resetPasswordFor $u -newPassword $PASSWORD"
  else
    echo "User $u missing."
  fi
done
echo "Ending user password reseting process!"

# Step 4. Locking users.
echo "Commencing user locking process!"
for u in "${USERS[@]}"; do
  if id "$u" &>/dev/null; then
    run "dscl . -create /Users/$u UserShell /usr/bin/false"
  fi
done
echo "Ending user locking process!"

# Step 5. Delete local users.
echo "Commencing local user deletion process!"
for u in "${USERS[@]}"; do
  if id "$u" &>/dev/null; then
    run "sysadminctl -deleteUser $u || dscl . -delete /Users/$u"
    run "rm -rf /Users/$u || true"
  fi
done
echo "Ending local user deletion process!"

# Step 6. Check active sessions.
echo "Commencing active session checking process!"
run "who || w"
for u in "${USERS[@]}"; do
  if id "$u" &>/dev/null; then
    run "su - $u -c 'whoami'"
  fi
done
echo "Ending active session checking process!"

# Step 7. Logoff users.
echo "Commencing logging off users process!"
for u in "${USERS[@]}"; do
  if id "$u" &>/dev/null; then
    UID=$(id -u $u)
    run "pkill -KILL -u $UID || true"
  fi
done
echo "Ending logging off users process!"

echo "Task Completed!"
```