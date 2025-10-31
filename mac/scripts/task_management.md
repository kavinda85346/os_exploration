### Task Management

##### This script is used for task management.

##### Steps involved in the process!

1. Run scripts on startup/login.
2. Schedule recurring backups.
3. Automate shutdown/restart.
4. Run reports daily/weekly.
5. Set custom reminders or notifications.

```
#!/bin/zsh

# Step 1. Run scripts on startup/login.
echo "Commencing running startup/login scripts process!"
Run on login: add "source ~/automation.zsh &" to ~/.zprofile
echo "Ending running startup/login scripts process!"

# Step 2. Schedule recurring backups.
echo "Commencing backups recurring process!"
backup_dir="$HOME/backups"
mkdir -p $backup_dir
tar -czf $backup_dir/zbackup_$(date +%F).tar.gz ~/Documents
echo "Ending backups recurring process!"

# Step 3. Automate shutdown/restart.
echo "Commencing shutdowning/restarting process!"
if [[ $1 == "shutdown" ]]; then
  sudo shutdown -h now
elif [[ $1 == "restart" ]]; then
  sudo shutdown -r now
fi
echo "Ending shutdowning/restarting process!"

# Step 4. Run reports daily/weekly.
echo "Commencing daily/weekly report running process!"
mkdir -p ~/reports
df -h > ~/reports/storage_report_$(date +%F).txt
echo "Ending daily/weekly report running process!"

# Step 5. Set custom reminders or notifications.
echo "Commencing reminders/notifications sending process!"
osascript -e 'display notification "Backup & Report completed" with title "Zsh Automation"'
echo "Ending reminders/notifications sending process!"

echo "Task Completed Successfully!"
```