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
#!/usr/bin/env bash
# user_flow.sh - sequential user management automation (BASH)
# Run as root or with sudo. Default: DRY_RUN=true
set -euo pipefail

DRY_RUN=true   # <-- set to false to actually perform actions
USERS=("demo_user1" "demo_user2")
PASSWORD="DemoP@ssw0rd123!"
GROUPS=("sudo" "docker")  # groups to add each user to (if available)

echocmd() { echo "+ $*"; }
run() {
  if [ "$DRY_RUN" = true ]; then
    echocmd "$*"
  else
    echocmd "$*"
    eval "$@"
  fi
}

# Check for root
if [ "$(id -u)" -ne 0 ]; then
  echo "WARNING: It's recommended to run as root. Some commands may fail."
fi

# Step 1. Create local users
echo "Commencing user creation process!"
for u in "${USERS[@]}"; do
  if id "$u" &>/dev/null; then
    echo "User $u already exists; skipping creation."
  else
    if command -v adduser &>/dev/null; then
      run "adduser --disabled-password --gecos '' $u || true"
    else
      run "useradd -m -s /bin/bash $u || true"
    fi
  fi
done
echo "Ending user creation process!"

# Step 2. Assigning permissions to users.
echo "Commencing user permission assigning process!"
for u in "${USERS[@]}"; do
  for g in "${GROUPS[@]}"; do
    if getent group "$g" >/dev/null 2>&1; then
      run "usermod -aG $g $u || gpasswd -a $u $g || true"
    else
      echo "Group $g not found; skipping for user $u."
    fi
  done
done
echo "Ending user permission assigning process!"

# Step 3. Reset user passwords.
echo "Commencing user password reseting process!"
for u in "${USERS[@]}"; do
  if id "$u" &>/dev/null; then
    run "echo \"$u:$PASSWORD\" | chpasswd"
  else
    echo "Cannot reset password for $u: user does not exist."
  fi
done
echo "Ending user password reseting process!"

# Step 4. Locking users.
echo "Commencing user locking process!"
for u in "${USERS[@]}"; do
  if id "$u" &>/dev/null; then
    run "usermod -L $u || passwd -l $u"
  else
    echo "Cannot lock $u: user does not exist."
  fi
done
echo "Ending user locking process!"

# Step 5. Delete local users.
echo "Commencing local user deleting process!"
for u in "${USERS[@]}"; do
  if id "$u" &>/dev/null; then
    run "userdel -r $u || deluser --remove-home $u || true"
  else
    echo "User $u does not exist; nothing to delete."
  fi
done
echo "Ending local user deleting process!"

# Step 6. Checking active sessions.
echo "Commencing active session checking process!"
run "who || w || users"
echo "--- Switch user (demo run of whoami for each user) ---"
for u in "${USERS[@]}"; do
  if id "$u" &>/dev/null; then
    run "su - $u -c 'whoami' || sudo -u $u whoami"
  else
    echo "Cannot switch to $u (does not exist)."
  fi
done
echo "Ending active session checking process!"

# Step 7. Logoff users.
echo "Commencing loging off users process!"
for u in "${USERS[@]}"; do
  if id "$u" &>/dev/null; then
    UID=$(id -u "$u")
    run "pkill -KILL -u $UID || killall -u $u || true"
  else
    echo "Cannot log off $u (does not exist)."
  fi
done
echo "Ending loging off users process!"

echo "All tasks completed successfully!"
```


