### Software and Package Management

##### This script is used for software and package management.

##### Steps involved in the process!

1. List installed programs.
2. Check for software updates.
3. Install or uninstall software.
4. Automate dependency installation.
5. Launch applications on schedule.

```
#!/usr/bin/env zsh
# software_automation.zsh
# Target: macOS (zsh + Homebrew)
# Usage: ./software_automation.zsh

autoload -U colors && colors

#Step 1. List installed programs.
echo "Commencing installed programs listing process!"
list_installed() {
  echo "=== Homebrew formulae and casks ==="
  if command -v brew >/dev/null 2>&1; then
    brew list --formula | sort | less
    echo "=== Casks ==="
    brew list --cask | sort | less
  else
    echo "Homebrew not found. Install it from https://brew.sh"
  fi
  echo "=== pip packages (user) ==="
  pip3 list --user || echo "pip3 not found"
}
echo "Ending installed programs listing process!"

#Step 2. Check for software updates.
echo "Commencing software update checking process!"
check_updates() {
  if command -v brew >/dev/null 2>&1; then
    echo "Fetching brew updates..."
    brew update
    echo "=== Outdated brew formulae/casks ==="
    brew outdated
  else
    echo "Homebrew not found."
  fi
  echo "=== pip outdated (user) ==="
  pip3 list --outdated --user || true
  echo "System software updates via softwareupdate (macOS):"
  sudo softwareupdate -l || echo "softwareupdate listing failed or no updates"
}
echo "Ending software update checking process!"

#Step 3. Install or uninstall software.
echo "Commencing software installing / uninstalling process!"
install_or_uninstall() {
  echo "Install (i) or Uninstall (u)?"
  read -r iu
  if [[ $iu == "i" ]]; then
    read -rp "Install via brew (formula/cask)? Type 'f' for formula, 'c' for cask: " ft
    if [[ $ft == "f" ]]; then
      read -rp "Formula name: " name
      brew install "$name"
    else
      read -rp "Cask name: " name
      brew install --cask "$name"
    fi
  elif [[ $iu == "u" ]]; then
    read -rp "Uninstall via brew (formula/cask)? 'f'/'c': " ft
    read -rp "Name to remove: " name
    if [[ $ft == "f" ]]; then
      brew uninstall "$name"
    else
      brew uninstall --cask "$name"
    fi
  else
    echo "Invalid input."
  fi
}
echo "Ending software installing / uninstalling process!"

#Step 4. Automate dependency installation.
echo "Commencing dependancy installation process!"
automate_dependency_installation() {
  echo "Dependency installation choices:"
  echo "1) Brew packages"
  echo "2) pip requirements.txt"
  read -r opt
  case $opt in
    1)
      read -rp "Enter brew packages separated by spaces: " pkgs
      for p in ${(z)pkgs}; do
        brew install "$p"
      done
      ;;
    2)
      read -rp "Path to requirements.txt: " req
      if [[ -f "$req" ]]; then
        pip3 install --user -r "$req"
      else
        echo "File not found."
      fi
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
  echo "Schedule a command via crontab (macOS supports cron)"
  read -rp "Command to run: " cmd
  read -rp "Cron schedule (e.g. 0 4 * * *): " cs
  if [[ -z $cmd || -z $cs ]]; then
    echo "Invalid input."
    return
  fi
  (crontab -l 2>/dev/null; echo "$cs $cmd") | crontab -
  echo "Added to crontab: $cs $cmd"
}
echo "Ending application launching process!"

main_menu() {
  while true; do
    echo "Choose:"
    echo "1) List installed programs"
    echo "2) Check for software updates"
    echo "3) Install or uninstall software"
    echo "4) Automate dependency installation"
    echo "5) Launch applications on schedule"
    echo "q) Quit"
    read -r choice
    case $choice in
      1) list_installed ;;
      2) check_updates ;;
      3) install_or_uninstall ;;
      4) automate_dependency_installation ;;
      5) schedule_launch ;;
      q) echo "Done."; break ;;
      *) echo "Invalid." ;;
    esac
    read -rp "Press Enter to continue..."
  done
}

main_menu
```


