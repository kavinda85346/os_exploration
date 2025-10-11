### System Maintenance & Cleanup

##### This script is used to perform maintenances and cleanups of the system.

##### Steps involved in the process!

1. System restore point or snapshot creation.
2. Backing up the logs / configuration of system.
3. Clear files in the Recycle Bin.
4. Clean cache of the packages.
5. Delete temporary / cache files of system.
6. Run system updates / upgrades.
7. Clean cache of the packages.
8. Delete temporary / cache files of system.
9. Disk space reporting.

```
#!/usr/bin/env bash    # The script header and meta data.

set -euo pipefail    # Handles errors and ensures the safe execution of the script.

# Define the backup path and timestamp for the backup.
BACKUP_ROOT="/var/backups/system_maintenance"
TIMESTAMP="$(date +%Y-%m-%d_%H-%M-%S)"    
BACKUP_DIR="${BACKUP_ROOT}/backup_${TIMESTAMP}"
PATHS_TO_BACKUP=("/var/log" "/etc")  # tweak as needed
mkdir -p "$BACKUP_DIR"

# Step 1. System restore point or snapshot creation.
echo "Commencing restore point or snapshot creation process!"
if command -v tmutil >/dev/null 2>&1; then
  echo "macOS detected — creating APFS local snapshot"
  tmutil localsnapshot
  echo "tmutil snapshot done."
else
  echo "No snapshot tool found — falling back to rsync directory backup (not atomic)."
fi
echo "Ending restore point or snapshot creation process!"

# Step 2. Backing up the logs / configuration of system.
echo "Commencing the logs/configuration backing up process!"
for p in "${PATHS_TO_BACKUP[@]}"; do
  if [ -e "$p" ]; then
    dest="${BACKUP_DIR}/$(basename "$p")"
    mkdir -p "$dest"
    rsync -aH --delete "$p"/ "$dest"/
    echo "Backed up $p -> $dest"
  else
    echo "Path not found: $p"
  fi
done
echo "Ending the logs/configuration backing up process!"

# Step 3. Clear files in the Recycle Bin.
echo "Commencing removing the files in the recycle bin process!"
if [ -d "$HOME/.Trash" ]; then
  rm -rf "$HOME/.Trash"/* || true
  echo "Cleared macOS Trash"
fi
echo "Ending removing the files in the recycle bin process!"

# Step 4. Clean cache of the packages.
echo "Commencing removing the cache of the packages!"
if command -v apt >/dev/null 2>&1; then
  apt-get clean
  apt-get autoremove -y
  echo "apt caches cleaned"
elif command -v dnf >/dev/null 2>&1; then
  dnf clean all
  dnf autoremove -y
  echo "dnf caches cleaned"
elif command -v yum >/dev/null 2>&1; then
  yum clean all
  yum autoremove -y
  echo "yum caches cleaned"
elif command -v pacman >/dev/null 2>&1; then
  pacman -Scc --noconfirm || true
  echo "pacman cache cleaned"
fi
if command -v brew >/dev/null 2>&1; then
  brew cleanup
  echo "brew cleaned"
fi
echo "Ending removing the cache of the packages!"

# Step 5. Delete temporary/cache files of system.
echo "Commencing removing the temporary / cache files of system!"
# System temp
rm -rf /tmp/* || true
rm -rf /var/tmp/* || true
# User caches (best-effort)
if [ -d "$HOME/.cache" ]; then
  find "$HOME/.cache" -mindepth 1 -maxdepth 1 -type d -exec rm -rf {} + || true
  echo "User cache cleaned"
fi
echo "Ending removing the temporary / cache files of system!"

# Step 6. Run system updates / upgrades.
echo "Commencing the system updating / upgrading process!"
if command -v apt >/dev/null 2>&1; then
  apt-get update && apt-get upgrade -y
elif command -v dnf >/dev/null 2>&1; then
  dnf upgrade -y
elif command -v yum >/dev/null 2>&1; then
  yum update -y
elif command -v pacman >/dev/null 2>&1; then
  pacman -Syu --noconfirm
elif command -v brew >/dev/null 2>&1; then
  brew update && brew upgrade
else
  echo "No known package manager found to perform updates."
fi
echo "Ending the system updating / upgrading process!"

# Step 7. Clean cache of the packages.
echo "Commencing removing the cache of the packages!"
if command -v apt >/dev/null 2>&1; then
  apt-get clean
  apt-get autoremove -y
  echo "apt caches cleaned"
elif command -v dnf >/dev/null 2>&1; then
  dnf clean all
  dnf autoremove -y
  echo "dnf caches cleaned"
elif command -v yum >/dev/null 2>&1; then
  yum clean all
  yum autoremove -y
  echo "yum caches cleaned"
elif command -v pacman >/dev/null 2>&1; then
  pacman -Scc --noconfirm || true
  echo "pacman cache cleaned"
fi
if command -v brew >/dev/null 2>&1; then
  brew cleanup
  echo "brew cleaned"
fi
echo "Ending removing the cache of the packages!"

# Step 8. Delete temporary/cache files of system.
echo "Commencing removing the temporary / cache files of system!"
# System temp
rm -rf /tmp/* || true
rm -rf /var/tmp/* || true
# User caches (best-effort)
if [ -d "$HOME/.cache" ]; then
  find "$HOME/.cache" -mindepth 1 -maxdepth 1 -type d -exec rm -rf {} + || true
  echo "User cache cleaned"
fi
echo "Ending removing the temporary / cache files of system!"

# Step 9. Disk space reporting.
echo "Commencing disk space reporting process!"
df -hT --total 2>/dev/null || df -h
echo ""
echo "Summary of largest directories on root (top 10):"
du -xhd1 / 2>/dev/null | sort -hr | head -n 10 || true
echo "Endinng disk space reporting process!"

# Dispalying the cron job finishing message
echo "Maintenance run complete. Backups at: $BACKUP_DIR"
```
