### User Management

##### This script is used for user management.

##### Steps involved in the batch script process!

1. Create local users.
2. Assigning permissions to users.
3. Reset user passwords.
4. Locking users.
5. Delete local users.
6. Check active sessions.
7. Logoff users.

```
@echo off
REM user_flow.bat - sequential user management (Batch)
REM Must run from elevated cmd.exe (Administrator).
SETLOCAL ENABLEDELAYEDEXPANSION

REM DRY_RUN=true prints commands; set to false to execute
set DRY_RUN=true

REM Configure users and password
set USERS=demo_user1 demo_user2
set PASSWORD=DemoP@ssw0rd123!

REM Step 1. Create local users.
echo Commencing local user creation process!
for %%U in (%USERS%) do (
  net user %%U >nul 2>&1
  if errorlevel 1 (
    if "%DRY_RUN%"=="true" (
      echo + net user %%U %PASSWORD% /add
    ) else (
      echo + net user %%U %PASSWORD% /add
      net user %%U %PASSWORD% /add
    )
  ) else (
    echo User %%U exists; skipping.
  )
)
echo Ending local user creation process!

REM Step 2. Assigning permissions to users.
echo Commencing user permission assigning process!
for %%U in (%USERS%) do (
  if "%DRY_RUN%"=="true" (
    echo + net localgroup Administrators %%U /add
  ) else (
    net localgroup Administrators %%U /add
  )
)
echo Ending user permission assigning process!

REM Step 3. Reset user passwords.
echo Commencing user password reseting process!
for %%U in (%USERS%) do (
  if "%DRY_RUN%"=="true" (
    echo + net user %%U %PASSWORD%
  ) else (
    net user %%U %PASSWORD%
  )
)
echo Ending user password reseting process!

REM Step 4. Locking users.
echo Commencing user locking process!
for %%U in (%USERS%) do (
  if "%DRY_RUN%"=="true" (
    echo + net user %%U /active:no
  ) else (
    net user %%U /active:no
  )
)
echo Ending user locking process!

REM Step 5. Delete local users.
echo Commencing local user deletion process!
for %%U in (%USERS%) do (
  if "%DRY_RUN%"=="true" (
    echo + net user %%U /delete
  ) else (
    net user %%U /delete
  )
)
echo Ending local user deletion process!

REM Step 6. Check active sessions.
echo Commencing active session checking process!
if "%DRY_RUN%"=="true" (
  echo + query user
) else (
  query user
)
echo --- Switch user demo (runas) ---
for %%U in (%USERS%) do (
  echo NOTE: runas would prompt for password; printing the command:
  echo runas /user:%%U "whoami"
)
echo Ending active session checking process!

REM Step 7. Logoff users.
echo Commencing logging off users process!
echo To log off a session: logoff <sessionID>
if "%DRY_RUN%"=="true" (
  echo + logoff <sessionID>
)
echo Ending logging off users process!

echo Task Complete!
```

##### Steps involved in the powershell script process!

1. Create local users.
2. Assigning permissions to users.
3. Reset user passwords.
4. Locking users.
5. Delete local users.
6. Check active sessions.
7. Logoff users.

```
<#
user_flow.ps1 - sequential user management automation (PowerShell)
Run in an elevated PowerShell prompt (Run as Administrator).
Default: DRY_RUN = $true -> prints actions only.
#>

param()
$ErrorActionPreference = "Stop"

$DRY_RUN = $true   # <- set to $false to execute
$Users = @("demo_user1","demo_user2")
$PasswordPlain = "DemoP@ssw0rd123!"
$Groups = @("Administrators")  # groups to add users to

function Do-Run {
    param([string]$Cmd, [ScriptBlock]$Action)
    if ($DRY_RUN) {
        Write-Host "+ $Cmd"
    } else {
        Write-Host "+ $Cmd"
        & $Action
    }
}

# Step 1. Create local users.
Write-Host "Commencing local user creation process!"
foreach ($u in $Users) {
    if (Get-LocalUser -Name $u -ErrorAction SilentlyContinue) {
        Write-Host "User $u exists; skipping."
    } else {
        $securePwd = ConvertTo-SecureString $PasswordPlain -AsPlainText -Force
        Do-Run "New-LocalUser -Name $u -Password ****** -NoPasswordExpired" {
            New-LocalUser -Name $u -Password $securePwd -FullName $u -Description "Created by user_flow.ps1"
        }
    }
}
Write-Host "Ending local user creation process!"

# Step 2. Assigning permissions to users.
Write-Host "Commencing user permission assigning process!"
foreach ($u in $Users) {
    foreach ($g in $Groups) {
        if (Get-LocalGroup -Name $g -ErrorAction SilentlyContinue) {
            Do-Run "Add-LocalGroupMember -Group $g -Member $u" {
                Add-LocalGroupMember -Group $g -Member $u -ErrorAction SilentlyContinue
            }
        } else {
            Write-Host "Group $g not found; skipping."
        }
    }
}
Write-Host "Ending user permission assigning process!"

# Step 3. Reset user passwords.
Write-Host "Commencing user password reseting process!"
foreach ($u in $Users) {
    if (Get-LocalUser -Name $u -ErrorAction SilentlyContinue) {
        $securePwd = ConvertTo-SecureString $PasswordPlain -AsPlainText -Force
        Do-Run "Set-LocalUser -Name $u -Password ******" {
            Set-LocalUser -Name $u -Password $securePwd
        }
    } else {
        Write-Host "User $u missing; cannot reset password."
    }
}
Write-Host "Ending user password reseting process!"

# Step 4. Locking users.
Write-Host "Commencing user locking process!"
foreach ($u in $Users) {
    if (Get-LocalUser -Name $u -ErrorAction SilentlyContinue) {
        Do-Run "Disable-LocalUser -Name $u" {
            Disable-LocalUser -Name $u
        }
    } else {
        Write-Host "User $u missing; skipping lock."
    }
}
Write-Host "Ending user locking process!"

# Step 5. Delete local users.
Write-Host "Commencing deleting local users process!"
foreach ($u in $Users) {
    if (Get-LocalUser -Name $u -ErrorAction SilentlyContinue) {
        Do-Run "Remove-LocalUser -Name $u" {
            Remove-LocalUser -Name $u
        }
    } else {
        Write-Host "User $u missing; nothing to delete."
    }
}
Write-Host "Ending deleting local users process!"

# Step 6. Check active sessions.
Write-Host "Commencing checking active sessions process!"
Do-Run "query user" {
    cmd /c query user
}
Write-Host "--- Switch user demo (Start-Process -Credential) ---"
foreach ($u in $Users) {
    Write-Host "Note: Running interactive commands as another user requires credentials; performing non-interactive whoami demo if possible."
    # Demonstrate using runas to run a single command — needs password; we will just print command
    $cmd = "runas /user:$env:COMPUTERNAME\$u `"whoami`""
    Write-Host "+ $cmd"
    # Actual non-interactive credential usage is not safe to hardcode; skip in DRY_RUN mode.
}
Write-Host "Ending checking active sessions process!"

# Step 7. Logoff users.
Write-Host "Commencing user logging off process!"
Do-Run "query session" {
    cmd /c query session
}
# To log off by session id, user may need to parse query output; print example
Write-Host "To force logoff: run 'logoff <sessionID>' as admin."
Write-Host "Commencing user logging off process!"

Write-Host "Task Completed!"
```
