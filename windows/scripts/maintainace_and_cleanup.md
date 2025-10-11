### System Maintenance & Cleanup

##### This task can be done using powershell / batch script.

##### This script is used to perform maintenances and cleanups of the system.

##### Steps involved in the batch script process!

1. Create a backup directory.
2. Create a system restore point / snapshot.
3. Backing up logs and configurations.
4. Clear the recycle bin.
5. Clean package manage caches.
6. Delete temporary files.
7. Clean Windows Components Store.
8. Trigger windows update.
9. Disk space reporting.

```
# "REM" non-execuatble command which is used to add comments into .bat script files.

@echo off    # Stops command echoing so the output looks clean.
REM maintenance.bat — Windows maintenance wrapper. Run as Administrator.    # The REM comment states that this script should run in administrator mode.
REM --------- CONFIG ----------
set BACKUP_ROOT=C:\Backups\system_maintenance    # Setting up the root directory to store the backup.
set TIMESTAMP=%DATE:~10,4%-%DATE:~4,2%-%DATE:~7,2%_%TIME:~0,2%-%TIME:~3,2%-%TIME:~6,2%    # Settings the timestamp in the format YYYY-MM-DD_HH-MM-SS using date/time string slicing.
set BACKUP_DIR=%BACKUP_ROOT%\backup_%TIMESTAMP%    # Creates a unique backup folder path using the timestamp.
set LOGS_TO_BACKUP="C:\Windows\System32\winevt\Logs" "C:\Windows\System32\config"    # Lists directories to back up (event logs, registry configs). You can add more.
REM Add more paths as needed
REM --------------------------

REM Step 1. Create a backup directory.
echo Creating backup folder: %BACKUP_DIR%
mkdir "%BACKUP_DIR%"

REM Step 2. Create a system restore point / snapshot.
echo Commencing system restore point / snapshot creation processs!
powershell -Command "Try { Checkpoint-Computer -Description 'AutomatedBackup_%TIMESTAMP%' ... }"
echo Ending system restore point / snapshot creation processs!

REM Step 3. Backing up logs and configurations.
echo Commencing backing up logs and configurations process!
for %%P in (%LOGS_TO_BACKUP%) do (
  if exist %%~P (
    robocopy "%%~P" "%BACKUP_DIR%\%%~nP" /MIR /NFL /NDL
  ) else (
    echo Path not found: %%~P
  )
)
echo Ending backing up logs and configurations process!

REM Step 4. Clear the recycle bin.
echo Commencing clearing the recycle bin process!
powershell -Command "Clear-RecycleBin -Force"
echo Ending clearing the recycle bin process!

REM Step 5. Clean package manage caches.
echo Commencing cleaning package manage cache process!
powershell -Command "if (Get-Command choco) { choco clean --yes } ..."
echo Ending cleaning package manage cache process!

REM Step 6. Delete temporary files.
echo Commencing deleting the temporary files process!
del /q /f /s "%TEMP%\*"
for /d %%d in (%TEMP%\*) do rd /s /q "%%d"
echo Ending the temporary file deletion process!

REM Step 7. Clean Windows Components Store.
echo Commencing cleaning Windows Components Store!
dism /online /Cleanup-Image /StartComponentCleanup
echo Ending cleaning Windows Components Store!

REM Step 8. Trigger windows update.
echo Commencing windows update and installation process!
powershell -Command "Start-Process -FilePath 'usoclient.exe' -ArgumentList 'StartScan'"
echo Ending windows update and installation process!

REM Step 9. Clean package manage caches.
echo Commencing cleaning package manage cache process!
powershell -Command "if (Get-Command choco) { choco clean --yes } ..."
echo Ending cleaning package manage cache process!

REM Step 10. Delete temporary files.
echo Commencing deleting the temporary files process!
del /q /f /s "%TEMP%\*"
for /d %%d in (%TEMP%\*) do rd /s /q "%%d"
echo Ending the temporary file deletion process!

REM Step 11. Clean Windows Components Store.
echo Commencing cleaning Windows Components Store!
dism /online /Cleanup-Image /StartComponentCleanup
echo Ending cleaning Windows Components Store!

REM Step 12. Disk space reporting.
echo Commencing disk space report genearation process!
powershell -Command "Get-PSDrive -PSProvider FileSystem | Select-Object Name, ..."
echo Ending disk space report genearation process!

echo Maintenance run complete. Backups stored under %BACKUP_DIR%.
pause
```

##### Steps involved in the powershell script process!

1. Create System Restore Point / Snapshot.
2. Backing up logs and configuration files.
3. Clean the recycle bin.
4. Clean the package manage caches.
5. Delete temporary files.
6. Clean Windows Component Store.
7. Update windows and install updates.
8. Clean the package manage caches.
9. Delete temporary files.
10. Clean Windows Component Store.
11. Disk generation report creation.

```
# Configuring the header
<#
maintenance.ps1 — Windows maintenance using PowerShell.
Run as Administrator.
CONFIGURE:
 - $BackupRoot : where backups are stored
 - $PathsToBackup : array of folders to copy (logs, configs)
 - Edit package manager sections to match your tooling (Chocolatey, winget, etc.)
#>

# Configuring the variables
$BackupRoot = "C:\Backups\system_maintenance"
$timestamp = (Get-Date).ToString("yyyy-MM-dd_HH-mm-ss")
$BackupDir = Join-Path $BackupRoot "backup_$timestamp"
$PathsToBackup = @("C:\Windows\System32\winevt\Logs", "C:\Windows\System32\config") # edit as needed
New-Item -ItemType Directory -Path $BackupDir -Force | Out-Null

# Step 1. Create System Restore Point / Snapshot.
Write-Output "Commencing system restore point / snapshot creation process!"
try {
    Checkpoint-Computer -Description "AutomatedBackup_$timestamp" -RestorePointType "MODIFY_SETTINGS"
    Write-Output "Restore point created."
} catch {
    Write-Warning "Could not create restore point: $_"
}
Write-Output "Ending system restore point / snapshot creation process!"

# Step 2. Backing up logs and configuration files.
Write-Output "Commencing logs and configuration backing up process!"
foreach ($p in $PathsToBackup) {
    if (Test-Path $p) {
        $dest = Join-Path $BackupDir ([IO.Path]::GetFileName($p))
        robocopy $p $dest /MIR /NFL /NDL | Out-Null
        Write-Output "Backed up $p -> $dest"
    } else {
        Write-Warning "Path not found: $p"
    }
}
Write-Output "Ending logs and configuration backing up process!"

# Step 3. Clean the recycle bin.
Write-Output "Commencing recycle bin cleaning process!"
try {
    Clear-RecycleBin -Force -ErrorAction Stop
    Write-Output "Recycle Bin cleared."
} catch {
    Write-Warning "Failed to clear Recycle Bin: $_"
}
Write-Output "Ending recycle bin cleaning process!"

# Step 4. Clean the package manage caches.
Write-Output "Commencing package cache cleaning process!"
if (Get-Command choco -ErrorAction SilentlyContinue) {
    choco clean --yes
    Write-Output "Chocolatey cache cleaned."
} else {
    Write-Output "Chocolatey not installed."
}
# Add other package managers if you use them
Write-Output "Ending package cache cleaning process!"

# Step 5. Deleting temporary files
Write-Output "Commencing deleting the temporary files!"
try {
    $temp = $env:TEMP
    Get-ChildItem -Path $temp -Force -ErrorAction SilentlyContinue | Remove-Item -Recurse -Force -ErrorAction SilentlyContinue
    Write-Output "Temp cleaned: $temp"
} catch {
    Write-Warning "Error cleaning temp: $_"
}
Write-Output "Ending the temparary file deletion process!"

# Step 6. Clean Windows Component Store.
Write-Output "Commencing Windows Component Store cleaning process!"
dism /online /Cleanup-Image /StartComponentCleanup
Write-Output "Ending Windows Component Store cleaning process!"

# Step 7. Update windows and install updates.
Write-Output "Commencing windows updating and installation process!"
# Option A: Use PSWindowsUpdate module (if available)
if (Get-Module -ListAvailable -Name PSWindowsUpdate) {
    Install-Module PSWindowsUpdate -Force -Scope CurrentUser -ErrorAction SilentlyContinue
    Import-Module PSWindowsUpdate
    Get-WindowsUpdate -Install -AcceptAll -AutoReboot
} else {
    Write-Warning "PSWindowsUpdate not installed. Attempting usoclient if present..."
    if (Get-Command usoclient.exe -ErrorAction SilentlyContinue) {
        Start-Process -FilePath "usoclient.exe" -ArgumentList "StartScan"
    } else {
        Write-Warning "No reliable programmatic Windows Update method found. Consider installing PSWindowsUpdate from PowerShell Gallery."
    }
}
Write-Output "Ending windows updating and installation process!"

# Step 8. Clean the package manage caches.
Write-Output "Commencing package cache cleaning process!"
if (Get-Command choco -ErrorAction SilentlyContinue) {
    choco clean --yes
    Write-Output "Chocolatey cache cleaned."
} else {
    Write-Output "Chocolatey not installed."
}
# Add other package managers if you use them
Write-Output "Ending package cache cleaning process!"

# Step 9. Deleting temporary files
Write-Output "Commencing deleting the temporary files!"
try {
    $temp = $env:TEMP
    Get-ChildItem -Path $temp -Force -ErrorAction SilentlyContinue | Remove-Item -Recurse -Force -ErrorAction SilentlyContinue
    Write-Output "Temp cleaned: $temp"
} catch {
    Write-Warning "Error cleaning temp: $_"
}
Write-Output "Ending the temparary file deletion process!"

# Step 10. Clean Windows Component Store.
Write-Output "Commencing Windows Component Store cleaning process!"
dism /online /Cleanup-Image /StartComponentCleanup
Write-Output "Ending Windows Component Store cleaning process!"

# Step 11. Disk generation report creation.
Write-Output "Commencing disk report generation process!"
Get-PSDrive -PSProvider FileSystem | Select-Object Name, @{Name='FreeGB';Expression={[math]::Round($_.Free/1GB,2)}}, @{Name='SizeGB';Expression={[math]::Round($_.Used/1GB + $_.Free/1GB,2)}} | Format-Table -AutoSize
Write-Output "Ending disk report generation process!"

Write-Output "Maintenance finished. Backups in $BackupDir"
```