### Security Management

##### This script is used for vulnerability monitoring and safeguarding the system.

##### Steps involved in the batch script process!

1. Monitor system logs for keywords
2. Audit login history
3. Check for failed logins
4. Automatically block suspicious IPs
5. Run antivirus or malware scans
6. Encrypt / decrypt files

```
@echo off
REM security_workflow.bat (Windows cmd)
REM This .bat is a simple orchestration for:
REM 1) query security logs for keywords / failed logins
REM 2) audit login history (query sessions)
REM 3) block IPs using Windows Firewall (netsh)
REM 4) run Windows Defender quick scan (if available)
REM 5) encrypt/decrypt files with openssl (if installed) - otherwise use PowerShell script
REM Run as Administrator.

SETLOCAL ENABLEDELAYEDEXPANSION

REM === configuration ===
SET KEYWORDS=failed;error;unauthorized
SET FAILED_EVENT_ID=4625
SET BLOCKED_DB=%TEMP%\blocked_ips_win.txt
SET BLOCK_THRESHOLD=5

if "%1"=="" goto usage

REM Step 1. Monitor system logs for keywords
echo Commencing monitoring system logs process!
REM Search Security log for Event ID 4625 (failed logon)
wevtutil qe Security /q:"*[System[(EventID=%FAILED_EVENT_ID%)]]" /f:text /c:50
goto :eof
echo Ending monitoring system logs process!

REM Step 2. Audit login history
echo Commencing audting login history process!
query user
goto :eof
echo Ending audting login history process!

REM Step 3. Check for failed logins
echo Commencing failed logins checking process!
REM Export 200 recent failed logon events, extract IpAddress
wevtutil qe Security /q:"*[System[(EventID=%FAILED_EVENT_ID%)]]" /f:xml /c:200 > "%TEMP%\failed.xml"
powershell -NoProfile -Command ^
  "$xml=[xml](Get-Content '%TEMP%\failed.xml');" ^
  "$ips=$xml.Event.System | ForEach-Object { $_ } ;" ^
  "$xml.Event | ForEach-Object { $_.EventData.Data } | Where-Object { $_ -match '\d+\.\d+\.\d+\.\d+' } | Group-Object | Sort-Object Count -Descending | Format-Table Count, Name -AutoSize"
del "%TEMP%\failed.xml"
goto :eof
echo Ending failed logins checking process!

REM Step 4. Automatically block suspicious IPs
echo Commencing suspicious IP blocking process!
REM This uses PowerShell for parsing and then netsh to add firewall rules
powershell -NoProfile -Command ^
  "$events = Get-WinEvent -FilterHashtable @{LogName='Security'; Id=%FAILED_EVENT_ID%} -MaxEvents 300;" ^
  "$ips = $events | ForEach-Object { ($_.Properties | ForEach-Object { $_.Value }) -match '\d+\.\d+\.\d+\.\d+' | Out-Null; [regex]::Matches($_.ToXml(),'\d+\.\d+\.\d+\.\d+') | ForEach-Object { $_.Value } };" ^
  "$ips | Group-Object | Where-Object { $_.Count -ge %BLOCK_THRESHOLD% } | ForEach-Object { Write-Output $_.Name }" > "%TEMP%\to_block.txt"
for /f "usebackq delims=" %%I in ("%TEMP%\to_block.txt") do (
  echo Blocking %%I via netsh
  netsh advfirewall firewall add rule name="AutoBlock_%%I" dir=in action=block remoteip=%%I
  echo %%I %DATE% %TIME% >> "%BLOCKED_DB%"
)
del "%TEMP%\to_block.txt"
goto :eof
echo Ending suspicious IP blocking process!

REM Step 5. Run antivirus or malware scans
echo Commencing antivirus scanning process!
powershell -NoProfile -Command "if (Get-Command -Name 'Start-MpScan' -ErrorAction SilentlyContinue) { Start-MpScan -ScanType QuickScan } else { Write-Output 'Windows Defender cmdlets not found' }"
goto :eof
echo Ending antivirus scanning process!

REM Step 6. Encrypt / decrypt files
echo Commencing encrypting / decrypting files process!
if "%2"=="" (
  echo Usage: %0 encrypt inputfile [outputfile]
  goto :eof
)
SET IN=%2
IF "%3"=="" (SET OUT=%IN%.enc) ELSE SET OUT=%3
REM Use openssl if installed; otherwise recommend PowerShell Protect-CmsMessage
where openssl >nul 2>nul
IF ERRORLEVEL 1 (
  echo OpenSSL not found. Use the PowerShell script for encryption (run the PowerShell version).
) ELSE (
  openssl enc -aes-256-cbc -pbkdf2 -salt -in "%IN%" -out "%OUT%"
  echo Encrypted "%IN%" -> "%OUT%"
)
goto :eof

:decrypt
if "%2"=="" (
  echo Usage: %0 decrypt inputfile [outputfile]
  goto :eof
)
SET IN=%2
IF "%3"=="" (SET OUT=%~dpn2.dec%~x2) ELSE SET OUT=%3
where openssl >nul 2>nul
IF ERRORLEVEL 1 (
  echo OpenSSL not found. Use the PowerShell script for decryption.
) ELSE (
  openssl enc -aes-256-cbc -d -pbkdf2 -in "%IN%" -out "%OUT%"
  echo Decrypted "%IN%" -> "%OUT%"
)
goto :eof
echo Ending encrypting / decrypting files process!

:usage
echo Usage: %0 ^<command^>
echo Commands:
echo   monitor_logs
echo   audit_logins
echo   check_failed
echo   auto_block
echo   run_av
echo   encrypt inputfile [outputfile]
echo   decrypt inputfile [outputfile]
exit /b 1
```

##### Steps involved in the powershell script process!

1. Monitor system logs for keywords
2. Audit login history
3. Check for failed logins
4. Automatically block suspicious IPs
5. Manage firewall rules
6. Run antivirus or malware scans
7. Encrypt / decrypt files

```
<#
.SYNOPSIS
  PowerShell script implementing monitoring, auditing, failed-login checks, auto-blocking (Windows Firewall),
  running Windows Defender scans, and AES encrypt/decrypt functions.

.Run as Administrator.
#>

# --- Configuration ---
$Keywords = @('failed','error','unauthorized')
$FailedEventId = 4625
$BlockThreshold = 5
$BlockedDb = "$env:TEMP\blocked_ips_ps.csv"
$BlockDurationSeconds = 3600

function Log([string]$msg){ "$((Get-Date).ToString('s')) $msg" }

# Step 1. Monitor system logs for keywords
echo "Commencing monitoring system logs process!"
function Monitor-LogsOnce {
  Log "Searching Security and System logs for keywords: $($Keywords -join ',')"
  # Search recent events
  Get-WinEvent -FilterHashtable @{LogName='Security'; StartTime=(Get-Date).AddHours(-1)} -ErrorAction SilentlyContinue |
    Where-Object { ($_.Message -match ($Keywords -join '|')) } |
    Select-Object TimeCreated, Id, MachineName, Message -First 100
}
echo "Ending monitoring system logs process!"

# Step 2. Audit login history
echo "Commencing auditing login history process!"
function Audit-LoginHistory {
  Log "Querying session/last-logon like info"
  qwinsta 2>$null
  # You can also query event log for successful logons (4624)
  Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624; StartTime=(Get-Date).AddDays(-7)} -MaxEvents 200 |
    Select-Object TimeCreated, @{n='Account';e={($_.Properties[5].Value)}}, @{n='Ip';e={ ($_.Properties | ForEach-Object { $_.Value } | Where-Object { $_ -match '\d+\.\d+\.\d+\.\d+' }) -join ',' }}
}
echo "Ending auditing login history process!"

# Step 3. Check for failed logins
echo "Commencing checking failed logins process!"
function Check-FailedLogins {
  Log "Searching for failed logins (EventID $FailedEventId)"
  $events = Get-WinEvent -FilterHashtable @{LogName='Security'; Id=$FailedEventId; StartTime=(Get-Date).AddDays(-1)} -MaxEvents 1000
  $ips = foreach ($e in $events) {
    $e.Message -match '\d+\.\d+\.\d+\.\d+' | Out-Null
    [regex]::Matches($e.Message,'\d+\.\d+\.\d+\.\d+') | ForEach-Object { $_.Value }
  }
  $ips | Group-Object | Sort-Object Count -Descending | Select-Object Count, Name
}
echo "Ending checking failed logins process!"

# Step 4. Automatically block suspicious IPs
echo "Commencing suspicious IP blocking process!"
function Auto-Block-IPs {
  Log "Auto-block suspicious IPs with threshold $BlockThreshold"
  New-Item -Path $BlockedDb -ItemType File -Force | Out-Null
  $events = Get-WinEvent -FilterHashtable @{LogName='Security'; Id=$FailedEventId; StartTime=(Get-Date).AddDays(-1)} -MaxEvents 1000
  $ips = foreach ($e in $events) {
    [regex]::Matches($e.Message,'\d+\.\d+\.\d+\.\d+') | ForEach-Object { $_.Value }
  }
  $toBlock = $ips | Group-Object | Where-Object {$_.Count -ge $BlockThreshold} | Select-Object -ExpandProperty Name
  foreach ($ip in $toBlock) {
    if (Select-String -Path $BlockedDb -Pattern "^$ip," -Quiet) {
      Log "$ip already blocked (record exists)"
    } else {
      Log "Blocking $ip via New-NetFirewallRule"
      New-NetFirewallRule -DisplayName "AutoBlock_$ip" -Direction Inbound -RemoteAddress $ip -Action Block | Out-Null
      "$ip,$(Get-Date).AddSeconds($BlockDurationSeconds).ToUniversalTime().ToString('o')" | Out-File -FilePath $BlockedDb -Append
    }
  }
}
echo "Ending suspicious IP blocking process!"

# Step 5. Manage firewall rules
echo "Commencing firewall rules management process!"
function Allow-Port {
  param([int]$Port)
  New-NetFirewallRule -DisplayName "Allow_TCP_Port_$Port" -Direction Inbound -Action Allow -Protocol TCP -LocalPort $Port
  Log "Allowed TCP $Port"
}
function Remove-Allow-Port {
  param([int]$Port)
  Get-NetFirewallRule -DisplayName "Allow_TCP_Port_$Port" -ErrorAction SilentlyContinue | Remove-NetFirewallRule -Confirm:$false
  Log "Removed allow for TCP $Port if existed"
}
echo "Ending firewall rules management process!"

# Step 6. Run antivirus or malware scans
echo "Commencing antivirus scanning process!"
function Run-AntivirusScan {
  if (Get-Command -Name Start-MpScan -ErrorAction SilentlyContinue) {
    Log "Starting Windows Defender QuickScan"
    Start-MpScan -ScanType QuickScan
  } else {
    Log "Windows Defender cmdlets are not available on this host."
  }
}
echo "Ending antivirus scanning process!"

# Step 7. Encrypt / decrypt files
echo "Commencing encrypting / decrypting files process!"
function Encrypt-File {
  param([string]$InFile, [string]$OutFile=$(Join-Path (Split-Path $InFile) ((Split-Path $InFile -Leaf) + '.enc')))
  if (-not (Test-Path $InFile)) { throw "Input file not found" }
  $pw = Read-Host -AsSecureString "Password"
  $b = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($pw)
  $plain = [Runtime.InteropServices.Marshal]::PtrToStringBSTR($b)
  [Runtime.InteropServices.Marshal]::ZeroFreeBSTR($b) | Out-Null

  $aes = [System.Security.Cryptography.Aes]::Create()
  $aes.KeySize = 256
  $aes.GenerateIV()
  $salt = New-Object byte[] 16
  [System.Security.Cryptography.RNGCryptoServiceProvider]::Create().GetBytes($salt)
  $derive = New-Object System.Security.Cryptography.Rfc2898DeriveBytes($plain, $salt, 100000)
  $key = $derive.GetBytes(32)
  $aes.Key = $key

  $outStream = [System.IO.File]::Create($OutFile)
  $outStream.Write($salt,0,$salt.Length) # prefix salt
  $outStream.Write($aes.IV,0,$aes.IV.Length) # prefix IV
  $cryptoStream = New-Object System.Security.Cryptography.CryptoStream($outStream, $aes.CreateEncryptor(), [System.Security.Cryptography.CryptoStreamMode]::Write)
  $inBytes = [System.IO.File]::ReadAllBytes($InFile)
  $cryptoStream.Write($inBytes,0,$inBytes.Length)
  $cryptoStream.FlushFinalBlock()
  $cryptoStream.Close()
  $outStream.Close()
  Log "Encrypted $InFile -> $OutFile"
}

function Decrypt-File {
  param([string]$InFile, [string]$OutFile)
  if (-not (Test-Path $InFile)) { throw "Input file not found" }
  $pw = Read-Host -AsSecureString "Password"
  $b = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($pw)
  $plain = [Runtime.InteropServices.Marshal]::PtrToStringBSTR($b)
  [Runtime.InteropServices.Marshal]::ZeroFreeBSTR($b) | Out-Null

  $fs = [System.IO.File]::OpenRead($InFile)
  $salt = New-Object byte[] 16; $fs.Read($salt,0,16) | Out-Null
  $iv = New-Object byte[] 16; $fs.Read($iv,0,16) | Out-Null
  $derive = New-Object System.Security.Cryptography.Rfc2898DeriveBytes($plain, $salt, 100000)
  $key = $derive.GetBytes(32)
  $aes = [System.Security.Cryptography.Aes]::Create()
  $aes.KeySize = 256
  $aes.Key = $key
  $aes.IV = $iv
  $cs = New-Object System.Security.Cryptography.CryptoStream($fs, $aes.CreateDecryptor(), [System.Security.Cryptography.CryptoStreamMode]::Read)
  if (-not $OutFile) { $OutFile = ($InFile -replace '\.enc$','') + '.dec' }
  $out = [System.IO.File]::Create($OutFile)
  $buffer = New-Object byte[] 4096
  while (($read = $cs.Read($buffer,0,$buffer.Length)) -gt 0) {
    $out.Write($buffer,0,$read)
  }
  $out.Close(); $cs.Close(); $fs.Close()
  Log "Decrypted $InFile -> $OutFile"
}
echo "Ending encrypting / decrypting files process!"

# Simple CLI dispatch
param($Command, $Arg1, $Arg2)
switch ($Command) {
  'monitor' { Monitor-LogsOnce }
  'audit' { Audit-LoginHistory }
  'check_failed' { Check-FailedLogins }
  'auto_block' { Auto-Block-IPs }
  'allow_port' { Allow-Port -Port ([int]$Arg1) }
  'remove_allow_port' { Remove-Allow-Port -Port ([int]$Arg1) }
  'run_av' { Run-AntivirusScan }
  'encrypt' { Encrypt-File -InFile $Arg1 -OutFile $Arg2 }
  'decrypt' { Decrypt-File -InFile $Arg1 -OutFile $Arg2 }
  default {
    "Usage: .\SecurityWorkflow.ps1 <command> [args]"
    "Commands: monitor | audit | check_failed | auto_block | allow_port <port> | remove_allow_port <port> | run_av | encrypt <in> [out] | decrypt <in> [out]"
  }
}
```


