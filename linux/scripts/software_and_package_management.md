### Software and Package Management

##### This script is used for software and package management.

##### Steps involved in the process!

1. List installed programs.
2. Check for software updates.
3. Install or uninstall software.
4. Automate dependency installation.
5. Launch applications on schedule.

```
#!/usr/bin/env bash
# software_automation.sh
# Target: Debian / Ubuntu (apt)
# Usage: sudo ./software_automation.sh  (some actions require sudo)

set -euo pipefail

require_sudo() {
  if [ "$EUID" -ne 0 ]; then
    echo "Some actions need root. You may be prompted for your password via sudo."
  fi
}

pause() { read -rp "Press Enter to continue..."; }

#Step 1. List installed programs.
echo "Commencing installed programs listing process!"
list_installed() {
  dpkg-query -W -f='${binary:Package}\t${Version}\n' | sort | less
  echo
  echo "Also: pip packages (system/user) -- choose:"
  echo "1) System pip (requires sudo)"
  echo "2) User pip"
  read -r choice
  if [ "$choice" = "1" ]; then
    sudo pip3 list || echo "pip3 not found"
  else
    pip3 list --user || echo "pip3 not found"
  fi
}
echo "Ending installed programs listing process!"

#Step 2. Check for software updates.
echo "Commencing software update checking process!"
check_updates() {
  sudo apt update
  echo
  echo "=== Upgradable apt packages ==="
  apt list --upgradable 2>/dev/null || echo "No apt upgrades or 'apt list' unsupported"
  echo
  echo "Check pip packages for outdated (user/system)?"
  echo "1) System pip (sudo)"
  echo "2) User pip"
  read -r pchoice
  if [ "$pchoice" = "1" ]; then
    sudo pip3 list --outdated || true
  else
    pip3 list --outdated --user || true
  fi
}
echo "Ending software update checking process!"

#Step 3. Install or uninstall software.
echo "Commencing software installing / uninstalling process!"
install_or_uninstall() {
  read -r iu
  if [ "$iu" = "i" ]; then
    read -rp "Package name to install (apt): " pkg
    read -rp "Also install via pip? (leave blank to skip) " pipname
    read -rp "Proceed to install $pkg via apt? (y/N) " yn
    if [[ "$yn" =~ ^[Yy] ]]; then
      sudo apt install -y "$pkg"
      echo "Installed $pkg (apt)."
    else
      echo "Skipped apt install."
    fi
    if [ -n "$pipname" ]; then
      read -rp "Install $pipname via pip3 (system)? (y/N) " yn2
      if [[ "$yn2" =~ ^[Yy] ]]; then
        sudo pip3 install "$pipname"
      fi
    fi
  elif [ "$iu" = "u" ]; then
    read -rp "Package name to remove (apt): " pkg
    read -rp "Confirm UNINSTALL of $pkg (This will remove files). (y/N) " yn
    if [[ "$yn" =~ ^[Yy] ]]; then
      sudo apt remove -y "$pkg"
      echo "Removed $pkg."
    fi
  else
    echo "Invalid choice."
  fi
}
echo "Ending software installing / uninstalling process!"

#Step 4. Automate dependency installation.
echo "Commencing dependancy installation process!"
automate_dependency_installation() {
  echo "Dependency automation options:"
  echo "1) Install from apt list (space-separated)"
  echo "2) Install from requirements.txt (pip)"
  echo "3) Create a small script to auto-install packages"
  read -r opt
  case "$opt" in
    1)
      read -rp "Enter apt packages separated by spaces: " pkgs
      echo "About to install: $pkgs"
      read -rp "Proceed? (y/N) " ok
      if [[ "$ok" =~ ^[Yy] ]]; then
        sudo apt update
        sudo apt install -y $pkgs
      fi
      ;;
    2)
      read -rp "Path to requirements.txt: " req
      if [ -f "$req" ]; then
        read -rp "Install into system pip or user? (system/user) " scope
        if [ "$scope" = "system" ]; then
          sudo pip3 install -r "$req"
        else
          pip3 install --user -r "$req"
        fi
      else
        echo "File not found: $req"
      fi
      ;;
    3)
      read -rp "Enter package names for a small installer script: " pkgs
      cat > ./auto_install_deps.sh <<EOF
#!/usr/bin/env bash
sudo apt update
sudo apt install -y $pkgs
EOF
      chmod +x ./auto_install_deps.sh
      echo "Created ./auto_install_deps.sh"
      ;;
    *)
      echo "Cancelled."
      ;;
  esac
}
echo "Ending dependancy installation process!"

#Step 5. Launch applications on schedule.
echo "Commencing application launching process!"
schedule_launch() {
  echo "Schedule a command to run (via crontab)."
  read -rp "Enter full command to run (e.g. /usr/bin/python3 /home/user/app.py): " cmd
  read -rp "Enter cron schedule (e.g. '0 3 * * *' for daily 3:00am): " cronentry
  # validate simple non-empty values
  if [ -z "$cmd" ] || [ -z "$cronentry" ]; then
    echo "Empty inputs — aborting."
    return
  fi
  # Add to current user's crontab
  (crontab -l 2>/dev/null; echo "$cronentry $cmd") | crontab -
  echo "Added to crontab: $cronentry $cmd"
  echo "You can view crontab with: crontab -l"
}
echo "Ending application launching process!"

echo "Task has been completed successfully!"

main_menu() {
  while true; do
    cat <<'MENU'
Interactive Software Automation (Ubuntu/Debian)
Choose:
1) List installed programs
2) Check for software updates
3) Install or uninstall software
4) Automate dependency installation
5) Launch applications on schedule
q) Quit
MENU
    read -rp "Choice: " ch
    case "$ch" in
      1) list_installed; pause ;;
      2) check_updates; pause ;;
      3) install_or_uninstall; pause ;;
      4) automate_dependency_installation; pause ;;
      5) schedule_launch; pause ;;
      q|Q) echo "Bye."; break ;;
      *) echo "Invalid choice." ;;
    esac
  done
}

require_sudo
main_menu
```


