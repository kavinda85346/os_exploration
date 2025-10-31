### Task Management

##### This script is used for task management.

##### Steps involved in the process!

1. Run scripts on startup/login.
2. Schedule recurring backups.
3. Automate shutdown/restart.
4. Run reports daily/weekly.
5. Set custom reminders or notifications.

```
#!/bin/bash

# Step 1. Run script on startup/login.
echo "Commencing script runnning process on startup/login"
Copy this file to ~/.config/autostart/ or add to ~/.bash_profile:
echo "~/automation.sh &" >> ~/.bash_profile
echo "Ending script runnning process on startup/login"

# Step 2. Schedule recurring backups.
echo "Commencing backup recurring process!"
Add with: (crontab -l; echo "0 2 * * * /path/to/automation.sh backup") | crontab -
backup_dir="$HOME/backups"
source_dir="$HOME/Documents"
mkdir -p "$backup_dir"
backup_file="$backup_dir/backup_$(date +%F).tar.gz"

if [[ "$1" == "backup" ]]; then
  tar -czf "$backup_file" "$source_dir"
  echo "Backup completed: $backup_file"
  exit 0
fi
echo "Ending backup recurring process!"

# Step 3. Automate shutdown/restart.
echo "Commencing shutdowning/restarting process!"
if [[ "$1" == "shutdown" ]]; then
  echo "Shutting down in 1 minute..."
  sudo shutdown -h +1
  exit 0
elif [[ "$1" == "restart" ]]; then
  echo "Restarting now..."
  sudo shutdown -r now
  exit 0
fi
echo "Ending shutdowning/restarting process!"

# Step 4. Run reports daily/weekly.
echo "Commencing daily/weekly report running process!"
# Simulate a report (disk usage)
report_dir="$HOME/reports"
mkdir -p "$report_dir"
du -h "$HOME" > "$report_dir/disk_report_$(date +%F).txt"
echo "Ending daily/weekly report running process!"

# Step 5. Set custom reminders or notifications.
echo "Commencing custom reminders/notifications sending process!"
notify-send "Automation Task" "Report generated successfully for $(date)"
echo "Ending custom reminders/notifications sending process!"

echo "All automation tasks completed."
```


