### Network Management 

##### This script is used for network management.

##### Steps involved in the batch script process!

1. Monitor network connectivity.
2. Ping Test.
3. Port connectivity test.
4. Restart network adapters.
5. Connect/Disconnect VPN.
6. Map/Unmap network drive.
7. Download resources from URL

```
@echo off
REM Define directories and variables
set LOGFILE=%USERPROFILE%\network_automation.log
set VPNNAME=MyVPN
set DOWNLOADURL=https://example.com/sample.txt
set DESTFILE=%USERPROFILE%\sample.txt
set TARGET=google.com
set PORT=443

REM Step 1. Monitor network connectivity.
echo Commencing network connectivity monitoring process!
ping -n 1 %TARGET% >nul
if errorlevel 1 (
  echo No internet connection! >> "%LOGFILE%"
) else (
  echo Internet is up. >> "%LOGFILE%"
)
echo Ending network connectivity monitoring process!

REM Step 2. Ping Test
echo Commencing ping test
for %%H in (8.8.8.8 1.1.1.1 %TARGET%) do (
  ping -n 2 %%H >nul && echo Ping OK: %%H >> "%LOGFILE%" || echo Ping FAIL: %%H >> "%LOGFILE%"
)
echo Ending ping test

REM Step 3. Port connectivity test
echo Commencing port connectivity test
powershell -Command "Test-NetConnection %TARGET% -Port %PORT%" >> "%LOGFILE%"
echo Ending port connectivity test"

REM Step 4. Restart network adapters
echo Commencing network adapter restarting process!
netsh interface set interface "Wi-Fi" admin=disabled
netsh interface set interface "Wi-Fi" admin=enabled
echo Ending network adapter restarting process!

REM Step 5. Connect/Disconnect VPN.
echo Commencing VPN management process!
rasdial %VPNNAME%
rasdial %VPNNAME% /disconnect
echo Ending VPN management process!

REM Step 6. Map/Unmap network drive
echo Commencing Network Drive management process!
net use Z: \\server\share /persistent:yes
echo Ending Network Drive management process!

REM Step 7. Download resources from URL
echo Commencing resource download from URL process!
powershell -Command "Invoke-WebRequest -Uri '%DOWNLOADURL%' -OutFile '%DESTFILE%'"
echo Ending resource download from URL process!

echo [%date% %time%] Tasks completed. >> "%LOGFILE%"
```

##### Steps involved in the powershell script process!

1. Monitor network connectivity.
2. Ping Test.
3. Port connectivity test.
4. Restart network adapters.
5. Connect/Disconnect VPN.
6. Map/Unmap network drive.
7. Download resources from URL

```
# Define directories and variables
$LogFile = "$env:USERPROFILE\network_automation.log"
$VPNName = "MyVPN"
$DriveLetter = "Z:"
$NetworkPath = "\\server\share"
$DownloadURL = "https://example.com/sample.txt"
$DestFile = "$env:USERPROFILE\sample.txt"
$Target = "google.com"
$Port = 443

# Step 1. Monitor network connectivity.
Write-Output "Commencing network connectivity monitoring process!"
if (Test-Connection -ComputerName $Target -Count 1 -Quiet) {
    Write-Output "Internet is up." | Tee-Object -FilePath $LogFile -Append
} else {
    Write-Output "No internet connection!" | Tee-Object -FilePath $LogFile -Append
}
Write-Output "Ending network connectivity monitoring process!"

# Step 2. Ping Test
Write-Output "Commencing ping test"
$Hosts = @("8.8.8.8", "1.1.1.1", $Target)
foreach ($h in $Hosts) {
    if (Test-Connection $h -Count 2 -Quiet) {
        Write-Output "Ping OK: $h"
    } else {
        Write-Output "Ping FAIL: $h"
    }
} | Tee-Object -FilePath $LogFile -Append
Write-Output "Ending ping test"

# Step 3. Port connectivity test
Write-Output "Commencing port connectivity test"
Test-NetConnection $Target -Port $Port | Tee-Object -FilePath $LogFile -Append
Write-Output "Ending port connectivity test"

# Step 4. Restart network adapters
Write-Output "Commencing network adapter restarting process!"
Restart-NetAdapter -Name "Wi-Fi" -Confirm:$false
Write-Output "Ending network adapter restarting process!"

# Step 5. Connect/Disconnect VPN.
Write-Output "Commencing VPN management process!"
rasdial $VPNName || rasdial $VPNName /disconnect
Write-Output "Ending VPN management process!"

# Step 6. Map/Unmap network drive
Write-Output "Commencing Network Drive management process!"
net use Z: \\server\share /persistent:yes
Write-Output "Ending Network Drive management process!"

# Step 7. Download resources from URL
Write-Output "Commencing resource download from URL process!"
Invoke-WebRequest -Uri $DownloadURL -OutFile $DestFile
Write-Output "Downloaded: $DestFile" | Tee-Object -FilePath $LogFile -Append
Write-Output "Ending resource download from URL process!"

Write-Output "[$(Get-Date)] Tasks completed." | Tee-Object -FilePath $LogFile -Append
```

