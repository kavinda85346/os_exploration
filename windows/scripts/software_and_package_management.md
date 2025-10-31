### Software and Package Management

##### This script is used for software and package management.

##### Steps involved in the batch scripting process!

1. List installed programs.
2. Check for software updates.
3. Install or uninstall software.
4. Automate dependency installation.
5. Launch applications on schedule.

```
@echo off
REM software_automation.bat
REM Target: Windows 10/11 (winget + schtasks)
REM Run as Administrator for install/uninstall and to create system scheduled tasks.

:setlocal
echo Interactive Software Automation (Windows - CMD)
:menu
echo.
echo 1) List installed programs
echo 2) Check for software updates
echo 3) Install or uninstall software
echo 4) Automate dependency installation
echo 5) Launch applications on schedule
echo q) Quit
set /p choice=Choice: 
if "%choice%"=="1" goto list
if "%choice%"=="2" goto check
if "%choice%"=="3" goto manage
if "%choice%"=="4" goto deps
if "%choice%"=="5" goto schedule
if /I "%choice%"=="q" goto end
echo Invalid choice.
goto menu

REM Step 1. List installed programs.
echo Commencing the installed programs!
:list
echo === winget list ===
winget list
echo.
pause
goto menu
echo Ending the installed programs!

REM Step 2. Check for software updates
echo Commencing software update checking process!
:check
echo Checking for winget-upgrades (this lists upgradable packages)...
winget upgrade
echo.
echo To upgrade packages, select them using winget upgrade <id> or 'winget upgrade --all' (requires admin).
pause
goto menu
echo Ending software update checking process!

REM Step 3. Install or uninstall software.
echo Commencing software installing / uninstalling process!
:manage
echo Install (i) or Uninstall (u)?
set /p iu=Choice (i/u): 
if /I "%iu%"=="i" (
  set /p pkg=Package id or name to install (e.g. firefox): 
  echo Installing %pkg% ...
  winget install --id "%pkg%" --accept-package-agreements --accept-source-agreements
) else if /I "%iu%"=="u" (
  set /p pkg=Package id or name to uninstall: 
  echo Uninstalling %pkg% ...
  winget uninstall --id "%pkg%" --accept-package-agreements --accept-source-agreements
) else (
  echo Invalid option.
)
pause
goto menu
echo Ending software installing / uninstalling process!

REM Step 4. Automate dependency installation.
echo Commencing dependancy installation process!
:deps
echo Dependency installation:
echo 1) Install multiple winget packages
echo 2) pip requirements.txt (Python)
set /p dopt=Choice: 
if "%dopt%"=="1" (
  set /p list=Enter package ids/names separated by commas: 
  for %%P in (%list:,= %) do (
    echo Installing %%P
    winget install --id "%%P" --accept-package-agreements --accept-source-agreements
  )
) else if "%dopt%"=="2" (
  set /p req=Path to requirements.txt:
  if exist "%req%" (
    echo Using pip to install user packages...
    python -m pip install --user -r "%req%"
  ) else (
    echo File not found.
  )
) else (
  echo Cancelled.
)
pause
goto menu
echo Ending dependancy installation process!

REM Step 5. Launch applications on schedule.
echo Commencing application launching process!
:schedule
echo Schedule a command using schtasks.
set /p taskname=Task name: 
set /p cmdline=Full command line (use quotes around path if necessary): 
echo Frequency options: DAILY, WEEKLY, ONCE, MINUTE
set /p freq=Frequency (e.g. DAILY): 
set /p time=Start time (HH:MM, 24-hour) or for MINUTE use "0001": 
REM Create the scheduled task (run as current user)
schtasks /Create /SC %freq% /TN "%taskname%" /TR "%cmdline%" /ST %time% /F
if %ERRORLEVEL% EQU 0 (
  echo Task created.
) else (
  echo Failed to create task. Try running this batch as Administrator.
)
pause
goto menu
echo Ending application launching process!

:end
echo Task Completed Successfully!
endlocal
```

##### Steps involved in the powershell scripting process!

1. List installed programs.
2. Check for software updates.
3. Install or uninstall software.
4. Automate dependency installation.
5. Launch applications on schedule.

```
<#
software_automation.ps1
Target: Windows PowerShell (Windows 10/11)
Run as Administrator for installs/uninstalls/schtasks creation.
#>

function Pause { Read-Host "Press Enter to continue..." }

# Step 1. List installed programs.
Write-Host "Commencing installed programs listing process!"
function List-Installed {
    Write-Host "=== winget list ==="
    winget list
    Write-Host "`n=== PowerShell Get-Package ==="
    Get-Package -ErrorAction SilentlyContinue | Out-Host
    Write-Host "`n=== pip packages (user) ==="
    python -m pip list --user 2>$null
}
Write-Host "Ending installed programs listing process!"

# Step 2. Check for software updates.
Write-Host "Commencing software update checking process!"
function Check-Updates {
    Write-Host "=== winget upgrades (available) ==="
    winget upgrade
    Write-Host "`n=== Windows Update (requires PSWindowsUpdate module) ==="
    if (Get-Module -ListAvailable -Name PSWindowsUpdate) {
        Import-Module PSWindowsUpdate
        Get-WindowsUpdate -AcceptAll -IgnoreReboot
    } else {
        Write-Host "PSWindowsUpdate module not found. Install it? (y/N)"
        $r = Read-Host
        if ($r -match '^[Yy]') {
            Install-Module -Name PSWindowsUpdate -Scope CurrentUser -Force
            Import-Module PSWindowsUpdate
            Get-WindowsUpdate -AcceptAll -IgnoreReboot
        } else {
            Write-Host "Skipping Windows Update check."
        }
    }
}
Write-Host "Ending software update checking process!"

# Step 3. Install or uninstall software.
Write-Host "Commencing software installing / uninstalling process!"
function Install-Or-Uninstall {
    $choice = Read-Host "Install (i) or Uninstall (u)?"
    if ($choice -eq 'i') {
        $pkg = Read-Host "Enter package id or name to install (winget)"
        winget install --id $pkg --accept-package-agreements --accept-source-agreements
    } elseif ($choice -eq 'u') {
        $pkg = Read-Host "Enter package id or name to uninstall (winget)"
        winget uninstall --id $pkg --accept-package-agreements --accept-source-agreements
    } else {
        Write-Host "Invalid choice."
    }
}
Write-Host "Ending software installing / uninstalling process!"

# Step 4. Automate dependency installation.
Write-Host "Commencing dependancy installation process!"
function Automate-Dependencies {
    Write-Host "1) Install list of winget packages"
    Write-Host "2) pip requirements.txt"
    $opt = Read-Host "Choice"
    switch ($opt) {
        '1' {
            $list = Read-Host "Enter package ids/names separated by commas"
            $pkgs = $list -split ',\s*'
            foreach ($p in $pkgs) {
                Write-Host "Installing $p"
                winget install --id $p --accept-package-agreements --accept-source-agreements
            }
        }
        '2' {
            $req = Read-Host "Path to requirements.txt"
            if (Test-Path $req) {
                python -m pip install --user -r $req
            } else {
                Write-Host "File not found."
            }
        }
        default {
            Write-Host "Cancelled."
        }
    }
}
Write-Host "Ending dependancy installation process!"

#Step 5. Launch applications on schedule.
Write-Host "Commencing application launching process!"
function Schedule-Launch {
    $taskName = Read-Host "Task name"
    $cmd = Read-Host "Full command (including args) to run"
    Write-Host "Schedule type examples: DAILY, WEEKLY, ONCE, MINUTE"
    $freq = Read-Host "Frequency (e.g. DAILY)"
    $time = Read-Host "Start time (HH:mm, 24-hour)"
    $createCmd = "schtasks /Create /SC $freq /TN `"$taskName`" /TR `"$cmd`" /ST $time /F"
    Write-Host "Creating scheduled task..."
    Invoke-Expression $createCmd
}
Write-Host "Ending application launching process!"

Write-Host "Task Completed Successfully!"

function Main-Menu {
    while ($true) {
        "`nInteractive Software Automation (PowerShell)"
        "1) List installed programs"
        "2) Check for software updates"
        "3) Install or uninstall software"
        "4) Automate dependency installation"
        "5) Launch applications on schedule"
        "q) Quit"
        $c = Read-Host "Choice"
        switch ($c) {
            '1' { List-Installed; Pause }
            '2' { Check-Updates; Pause }
            '3' { Install-Or-Uninstall; Pause }
            '4' { Automate-Dependencies; Pause }
            '5' { Schedule-Launch; Pause }
            'q' { break }
            default { "Invalid"; Pause }
        }
    }
}

Main-Menu
```


