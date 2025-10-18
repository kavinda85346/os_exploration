### File and Directory Management 

##### This task can be done using powershell / batch script.

##### This script is used to file and directory management.

##### Steps involved in the batch script process!

1. Create the today's work folder.
2. Copy files from project/template files to today's work folder.
3. Rename the report files in the created folder with a timestamp.
4. If a work folder exsists, compress it and copy to the backup directory.
5. Sync project folder to backup.
6. Generate a directory report.

```
# Define directories and variables
set "PROJECT_DIR=C:\Projects"
set "BACKUP_DIR=C:\ProjectBackups"
set "TODAY=%DATE:~10,4%-%DATE:~4,2%-%DATE:~7,2%"
set "TODAY_DIR=%PROJECT_DIR%\Work_%TODAY%"

# Check wither today's work folder exsists!
if not exist "%TODAY_DIR%"

Step 1. Create the today's work folder.
echo "Commencing today's work folder creation process!"
mkdir "%TODAY_DIR%"
echo "Ending today's work folder creation process!"

# Step 2. Copy files from project/template files to today's work folder.
echo "Commencing copying project template files to the todays work folder!"
xcopy "%PROJECT_DIR%\templates\*" "%TODAY_DIR%\" /E /I /Y
echo "Ending copying project template files to the todays work folder!"

# Step 3. Rename the report files in the created folder with a timestamp.
echo "Commencing renaming files in the folder with a timestamp."
for %%f in ("%TODAY_DIR%\report*.txt") do (
    ren "%%f" "%%~nf_%TIME:~0,2%%TIME:~3,2%%TIME:~6,2%%~xf"
)
echo "Ending renaming files process!"

# Step 4. If a work folder exsists, compress it and copy to the backup directory
echo "Commencing compressing files and copying to the backup directory process!"
for /f "delims=" %%d in ('dir "%PROJECT_DIR%\Work_*" /ad /b /o-d') do (
    if not defined LAST (
        set "LAST=%%d"
    ) else (
        powershell Compress-Archive -Path "%PROJECT_DIR%\%%d" -DestinationPath "%BACKUP_DIR%\%%d.zip"
        goto :done
    )
)
:done
echo "Ending compressing files and copying to the backup directory process!"

# Step 5. Sync project folder to backup.
echo "Commencing project folder syncing process!"
xcopy "%PROJECT_DIR%\*" "%BACKUP_DIR%\latest\" /E /I /Y
echo "Ending project folder syncing process!"

# Step 6. Generate a directory report.
echo "Commencing directory report generation process!"
dir "%PROJECT_DIR%" /s > "%BACKUP_DIR%\directory_report_%TODAY%.txt"
echo "Ending directory report generation process!"

echo All tasks completed successfully!
pause
```

##### Steps involved in the powershell script process!

1. Create the today's work folder.
2. Copy files from project/template files to today's work folder.
3. Rename the report files in the created folder with a timestamp.
4. If a work folder exsists, compress it and copy to the backup directory.
5. Sync project folder to backup.
6. Generate a directory report.

```
# Define directories and variables
$ProjectDir = "C:\Projects"
$BackupDir = "C:\ProjectBackups"
$Today = Get-Date -Format "yyyy-MM-dd"
$TodayDir = "$ProjectDir\Work_$Today"

Step 1. Create the today's work folder.
Write-Host "Commencing today's work folder creation process!"
if (!(Test-Path $TodayDir)) { New-Item -ItemType Directory -Path $TodayDir }
Write-Host "Ending today's work folder creation process!"

# Step 2. Copy files from project/template files to today's work folder.
Write-Host "Commencing copying project template files to the todays work folder!"
Copy-Item "$ProjectDir\templates\*" $TodayDir -Recurse -Force
Write-Host "Ending copying project template files to the todays work folder!"

# Step 3. Rename the report files in the created folder with a timestamp.
Write-Host "Commencing renaming files in the folder with a timestamp."
Get-ChildItem "$TodayDir\report*.txt" | ForEach-Object {
    $timestamp = Get-Date -Format "HHmmss"
    Rename-Item $_ -NewName ($_.BaseName + "_$timestamp" + $_.Extension)
}
Write-Host "Ending renaming files process!"

# Step 4. If a work folder exsists, compress it and copy to the backup directory
Write-Host "Commencing compressing files and copying to the backup directory process!"
$folders = Get-ChildItem $ProjectDir -Directory | Where-Object Name -like "Work_*" | Sort-Object Name -Descending
if ($folders.Count -gt 1) {
    $Yesterday = $folders[1].FullName
    Compress-Archive -Path $Yesterday -DestinationPath "$BackupDir\$($folders[1].Name).zip" -Force
}
Write-Host "Ending compressing files and copying to the backup directory process!"

# Step 5. Sync project folder to backup.
Write-Host "Commencing project folder syncing process!"
robocopy $ProjectDir "$BackupDir\latest" /MIR
Write-Host "Ending project folder syncing process!

# Step 6. Generate a directory report.
Write-Host "Commencing directory report generation process!"
Get-ChildItem $ProjectDir -Recurse | Select-Object FullName, Length, LastWriteTime |
    Out-File "$BackupDir\directory_report_$Today.txt"
Write-Host "Ending directory report generation process!"

Write-Host "All tasks completed successfully!"
```


