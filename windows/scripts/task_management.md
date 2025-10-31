### Task Management

##### This script is used for task management.

##### Steps involved in the batch scripting process!

1. Run scripts on startup/login.
2. Schedule recurring backups.
3. Automate shutdown/restart.
4. Run reports daily/weekly.
5. Set custom reminders or notifications.

```
@echo off

:: Step 1. Run scripts on startup/login.
echo Commencing running scripts on startup/login!
:: Place a shortcut to this script in:
:: %APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup
echo Ending running scripts on startup/login!

:: Step 2. Schedule recurring backups.
echo Commencing backup recurring backup process!
:: Example command (run once manually to create):
:: schtasks /create /sc daily /tn "DailyBackup" /tr "%~dp0automation.bat backup" /st 02:00

set backupdir=%USERPROFILE%\Backups
set sourcedir=%USERPROFILE%\Documents
if "%1"=="backup" (
    if not exist "%backupdir%" mkdir "%backupdir%"
    set filename=backup_%date:~-4%%date:~4,2%%date:~7,2%.zip
    powershell Compress-Archive -Path "%sourcedir%" -DestinationPath "%backupdir%\%filename%"
    echo Backup completed at %time%.
    exit /b
)
echo Ending backup recurring backup process!

:: Step 3. Automate shutdown/restart.
echo Commencing shutdowning/restarting process!
if "%1"=="shutdown" (
    shutdown /s /t 60
    exit /b
)
if "%1"=="restart" (
    shutdown /r /t 0
    exit /b
)
echo Ending shutdowning/restarting process!

:: Step 4. Run reports daily/weekly.
echo Commencing daily/weekly report running process!
set reportdir=%USERPROFILE%\Reports
if not exist "%reportdir%" mkdir "%reportdir%"
systeminfo > "%reportdir%\system_report_%date:~-4%%date:~4,2%%date:~7,2%.txt"
echo Ending daily/weekly report running process!

:: Step 5. Set custom reminders or notifications.
echo Commencing reminders/notifications sending process!
powershell -Command "New-BurntToastNotification -Text 'Automation Task', 'Backup and Report done!'"
echo Ending reminders/notifications sending process!

echo Automation tasks completed.
```

##### Steps involved in the powershell scripting process!

1. Run scripts on startup/login.
2. Schedule recurring backups.
3. Automate shutdown/restart.
4. Run reports daily/weekly.
5. Set custom reminders or notifications.

```
# Step 1. Run scripts on startup/login.
Write-Host "Commencing running scripts on startup/login process!"
Copy shortcut of this script to:
$env:APPDATA\Microsoft\Windows\Start Menu\Programs\Startup
Write-Host "Ending running scripts on startup/login process!"

# Step 2. Schedule recurring backups.
Write-Host "Commencing backups recurring process!"
Register-ScheduledTask -Action (New-ScheduledTaskAction -Execute "pwsh" -Argument "-File `"$PSScriptRoot\automation.ps1`" backup") -Trigger (New-ScheduledTaskTrigger -Daily -At 2am) -TaskName "DailyBackup"

$BackupDir = "$env:USERPROFILE\Backups"
$SourceDir = "$env:USERPROFILE\Documents"
if ($args -contains "backup") {
    if (!(Test-Path $BackupDir)) { New-Item -ItemType Directory -Path $BackupDir | Out-Null }
    $BackupFile = "$BackupDir\backup_$(Get-Date -Format yyyyMMdd).zip"
    Compress-Archive -Path $SourceDir -DestinationPath $BackupFile -Force
    Write-Host "Backup completed: $BackupFile"
    exit
}
Write-Host "Ending backups recurring process!"

# Step 3. Automate shutdown/restart.
Write-Host "Commencing shutdowning/restarting process!"
if ($args -contains "shutdown") { Stop-Computer -Force }
if ($args -contains "restart")  { Restart-Computer -Force }
Write-Host "Ending shutdowning/restarting process!"

# Step 4. Run reports daily/weekly.
Write-Host "Commencing daily/weekly reports running process!"
$ReportDir = "$env:USERPROFILE\Reports"
if (!(Test-Path $ReportDir)) { New-Item -ItemType Directory -Path $ReportDir | Out-Null }
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10 |
Out-File "$ReportDir\TopProcesses_$(Get-Date -Format yyyyMMdd).txt"
Write-Host "Ending daily/weekly reports running process!"

# Step 5. Set custom reminders or notifications.
Write-Host "Commencing reminder/notification sending process!"
Import-Module BurntToast -ErrorAction SilentlyContinue
New-BurntToastNotification -Text "Automation Task", "Backup and report completed successfully."
Write-Host "Ending reminder/notification sending process!"

Write-Host "Task has completed successfully!"
```