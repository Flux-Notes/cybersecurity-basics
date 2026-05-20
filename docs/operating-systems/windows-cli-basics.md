# Windows CLI Basics for Cybersecurity

A comprehensive reference covering Windows command-line fundamentals — CMD and PowerShell — including file system navigation, user management, networking, process management, registry, scripting, and security-relevant techniques used in penetration testing, hardening, and incident response.

---

## 1. Command Prompt (CMD) Basics

### Opening CMD

| Method | Notes |
|--------|-------|
| `Win + R` → `cmd` | Standard user |
| `Win + R` → `cmd` → `Ctrl+Shift+Enter` | Elevated (Run as Administrator) |
| Right-click Start → Terminal (Admin) | Windows 11 |
| `runas /user:Administrator cmd` | Run as another user |

### CMD Syntax & Operators

```cmd
:: This is a comment

:: Run second command only if first succeeds
command1 && command2

:: Run second command regardless
command1 & command2

:: Run second if first FAILS
command1 || command2

:: Pipe output to next command
ipconfig | findstr IPv4

:: Redirect stdout to file (overwrite)
ipconfig > output.txt

:: Redirect stdout to file (append)
ipconfig >> output.txt

:: Redirect stderr
command 2> errors.txt

:: Redirect both stdout and stderr
command > output.txt 2>&1

:: Discard output
command > nul 2>&1
```

### Useful CMD Shortcuts

| Shortcut | Action |
|----------|--------|
| `F7` | Command history popup |
| `↑ / ↓` | Cycle through history |
| `Tab` | Autocomplete path/command |
| `Ctrl+C` | Kill running command |
| `Ctrl+Z` | Send EOF |
| `cls` | Clear screen |
| `doskey /history` | View command history |

---

## 2. Navigation & File System

### Navigation Commands

```cmd
:: Print current directory
cd
echo %CD%

:: Change directory
cd C:\Windows\System32
cd ..                   :: parent directory
cd \                    :: root of current drive
cd /d D:\               :: change drive and directory
D:                      :: switch to D: drive

:: List directory contents
dir
dir /a                  :: include hidden and system files
dir /a:h                :: only hidden files
dir /s                  :: recursive
dir /o:d                :: sort by date
dir /b                  :: bare format (names only)
dir /q                  :: show owner
dir C:\Windows\*.exe /s :: search recursively

:: Clear screen
cls
```

### File Operations

```cmd
:: Create a file
echo. > file.txt
echo content > file.txt
type nul > file.txt

:: Create a directory
mkdir C:\test
mkdir C:\path\to\nested   :: creates all intermediate dirs (Windows 10+)
md C:\test                :: shorthand

:: Copy files
copy file.txt backup.txt
copy C:\src\file.txt D:\dest\
xcopy C:\src\ D:\dest\ /E /I /H    :: recursive, include hidden
robocopy C:\src D:\dest /E         :: robust copy (preferred)

:: Move / rename
move file.txt newname.txt
move file.txt C:\destination\
ren file.txt newname.txt           :: rename only

:: Delete
del file.txt
del /f file.txt                    :: force delete read-only
del /q file.txt                    :: quiet (no confirmation)
rmdir C:\folder
rmdir /s /q C:\folder              :: recursive, quiet

:: Display file contents
type file.txt
more file.txt              :: paginated
```

### Searching for Files

```cmd
:: Find files by name
dir /s /b C:\*password*
dir /s /b C:\*.config

:: where command (find executables in PATH)
where python
where cmd

:: PowerShell (more flexible)
Get-ChildItem -Path C:\ -Recurse -Filter "*.config" -ErrorAction SilentlyContinue
Get-ChildItem -Path C:\ -Recurse -Include "password*","credential*" -ErrorAction SilentlyContinue

:: Find files modified recently
forfiles /p C:\ /s /m *.* /d -1 /c "cmd /c echo @path"
```

### Alternate Data Streams (ADS)

```cmd
:: Create an ADS
echo hidden_data > file.txt:stream

:: List ADS
dir /r file.txt

:: Read ADS
more < file.txt:stream

:: PowerShell
Get-Item .\* -Stream *
Get-Content .\file.txt -Stream stream
```

---

## 3. File & Folder Permissions (icacls)

```cmd
:: View permissions
icacls C:\path\to\file
icacls C:\path\to\folder

:: Grant permissions
icacls file.txt /grant username:(R)        :: Read
icacls file.txt /grant username:(W)        :: Write
icacls file.txt /grant username:(F)        :: Full Control
icacls file.txt /grant username:(RX)       :: Read & Execute
icacls C:\folder /grant username:(OI)(CI)(F)  :: Inherit to subfolders

:: Deny permissions
icacls file.txt /deny username:(W)

:: Remove permissions
icacls file.txt /remove username

:: Reset permissions to inherited
icacls C:\folder /reset /T

:: Take ownership
takeown /f C:\file.txt
takeown /f C:\folder /r /d y               :: recursive

:: Inheritance flags
:: (OI) - Object Inherit (applies to files in folder)
:: (CI) - Container Inherit (applies to subfolders)
:: (IO) - Inherit Only
:: (NP) - No Propagate Inherit

:: Save and restore ACLs
icacls C:\folder /save acl_backup.txt /T
icacls C:\folder /restore acl_backup.txt
```

---

## 4. User & Group Management

### User Commands

```cmd
:: List local users
net user

:: Show details of a user
net user username

:: Create a user
net user newuser Password123! /add

:: Delete a user
net user username /delete

:: Change password
net user username NewPassword123!

:: Enable / disable account
net user username /active:yes
net user username /active:no

:: Add user to a group
net localgroup Administrators username /add

:: Remove user from group
net localgroup Administrators username /delete

:: PowerShell equivalents
Get-LocalUser
New-LocalUser -Name "testuser" -Password (ConvertTo-SecureString "Pass123!" -AsPlainText -Force)
Remove-LocalUser -Name "testuser"
Enable-LocalUser -Name "testuser"
Disable-LocalUser -Name "testuser"
```

### Group Commands

```cmd
:: List local groups
net localgroup

:: Show group members
net localgroup Administrators
net localgroup "Remote Desktop Users"

:: Create a group
net localgroup groupname /add

:: Delete a group
net localgroup groupname /delete

:: Domain equivalents
net user /domain
net group /domain
net group "Domain Admins" /domain

:: PowerShell
Get-LocalGroup
Get-LocalGroupMember -Group "Administrators"
Add-LocalGroupMember -Group "Administrators" -Member "username"
```

### Current User & Privileges

```cmd
:: Current user
whoami

:: User SID
whoami /user

:: Group memberships
whoami /groups

:: Privileges
whoami /priv

:: All info
whoami /all
```

---

## 5. System Information

```cmd
:: Detailed system info
systeminfo
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type" /C:"Hotfix"

:: Hostname
hostname

:: Environment variables
set
set USERNAME
set PATH

:: OS version shortcut
ver
winver        :: GUI version dialog

:: Hardware info (WMI)
wmic computersystem get model,manufacturer,totalphysicalmemory
wmic cpu get name,numberofcores,maxclockspeed
wmic bios get serialnumber,smbiosbiosversion

:: Installed hotfixes / patches
wmic qfe list brief
systeminfo | findstr "Hotfix"

:: PowerShell
Get-ComputerInfo
Get-ComputerInfo | Select-Object OsName, OsVersion, CsName, CsDomainRole
[System.Environment]::OSVersion
```

---

## 6. Process Management

```cmd
:: List running processes
tasklist
tasklist /v                          :: verbose (includes window titles)
tasklist /svc                        :: show hosted services per process
tasklist /fi "imagename eq svchost.exe"   :: filter by name
tasklist /fi "pid eq 1234"           :: filter by PID
tasklist /fi "status eq running"

:: Kill a process
taskkill /PID 1234
taskkill /PID 1234 /F                :: force kill
taskkill /IM notepad.exe /F          :: kill by name
taskkill /IM notepad.exe /F /T       :: kill process tree

:: Start a process
start notepad.exe
start "" "C:\Program Files\app.exe"
start /min cmd /c command            :: minimized
start /b command                     :: background (no new window)

:: Run as different user
runas /user:administrator cmd
runas /user:DOMAIN\admin powershell

:: PowerShell
Get-Process
Get-Process | Select-Object Name, Id, Path, CPU, WorkingSet
Get-Process -Name "svchost" | Select-Object *
Stop-Process -Id 1234
Stop-Process -Name "notepad" -Force
Start-Process notepad.exe
Start-Process cmd -Verb RunAs        :: elevated
```

### Process Hashing (Verify Legitimacy)

```powershell
# Get hash of a running process executable
Get-Process | Select-Object Name, Id, Path | ForEach-Object {
    if ($_.Path) {
        $hash = Get-FileHash $_.Path -Algorithm SHA256 -ErrorAction SilentlyContinue
        [PSCustomObject]@{ Name=$_.Name; PID=$_.Id; Hash=$hash.Hash; Path=$_.Path }
    }
}
```

---

## 7. Services

```cmd
:: List all services
sc query type= all
sc query state= all

:: Query specific service
sc query wuauserv
sc query "Windows Update"

:: Show service configuration (binary path, start type)
sc qc wuauserv
sc qc "Windows Defender"

:: Start / stop / pause a service
sc start wuauserv
sc stop wuauserv
sc pause wuauserv
sc continue wuauserv

:: Enable / disable service
sc config wuauserv start= auto
sc config wuauserv start= disabled
sc config wuauserv start= demand     :: manual

:: Create a service
sc create MySvc binpath= "C:\path\service.exe" start= auto
sc description MySvc "My Service Description"

:: Delete a service
sc delete MySvc

:: net commands
net start
net stop wuauserv
net start wuauserv

:: PowerShell
Get-Service
Get-Service -Name "wuauserv" | Select-Object *
Start-Service -Name "wuauserv"
Stop-Service -Name "wuauserv"
Set-Service -Name "wuauserv" -StartupType Automatic
New-Service -Name "MySvc" -BinaryPathName "C:\path\service.exe"
```

---

## 8. Networking

### IP & Interface Information

```cmd
:: Full network configuration
ipconfig /all

:: IP address only
ipconfig

:: DNS cache
ipconfig /displaydns

:: Flush DNS
ipconfig /flushdns

:: Release and renew DHCP
ipconfig /release
ipconfig /renew

:: PowerShell
Get-NetAdapter
Get-NetIPAddress
Get-NetIPConfiguration
```

### Active Connections & Ports

```cmd
:: All connections and listening ports
netstat -ano

:: Show process names (requires elevation)
netstat -anob

:: Only listening ports
netstat -ano | findstr LISTENING

:: Specific port
netstat -ano | findstr :443
netstat -ano | findstr :4444

:: PowerShell (more detail)
Get-NetTCPConnection
Get-NetTCPConnection -State Listen
Get-NetTCPConnection -State Established
Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess
```

### Connectivity Testing

```cmd
:: Ping
ping google.com
ping -n 4 -w 1000 192.168.1.1     :: 4 packets, 1 second timeout

:: Traceroute
tracert google.com
tracert -d google.com              :: no DNS resolution

:: DNS lookup
nslookup example.com
nslookup example.com 8.8.8.8      :: use specific DNS server
nslookup -type=MX example.com

:: Test TCP port connectivity (PowerShell)
Test-NetConnection -ComputerName example.com -Port 443
Test-NetConnection -ComputerName 192.168.1.1 -Port 22
(New-Object Net.Sockets.TcpClient).Connect("192.168.1.1", 22)

:: Check if port is open (quick)
telnet 192.168.1.1 22
```

### ARP & Routing

```cmd
:: ARP table
arp -a

:: Add static ARP entry
arp -s 192.168.1.1 AA-BB-CC-DD-EE-FF

:: Routing table
route print
netstat -r

:: Add a route
route add 10.10.10.0 mask 255.255.255.0 192.168.1.1

:: Delete a route
route delete 10.10.10.0

:: PowerShell
Get-NetNeighbor                    :: ARP table
Get-NetRoute                       :: routing table
```

### SMB & Network Shares

```cmd
:: List network shares
net share

:: Map a network drive
net use Z: \\server\share
net use Z: \\server\share /user:domain\username password

:: Disconnect
net use Z: /delete
net use * /delete

:: List sessions to this machine
net session

:: List connections from this machine
net use

:: View shares on a remote host
net view \\server

:: PowerShell
Get-SmbShare
Get-SmbConnection
Get-SmbSession
New-SmbMapping -LocalPath Z: -RemotePath \\server\share
```

---

## 9. Windows Firewall

```cmd
:: Check firewall status
netsh advfirewall show allprofiles

:: Enable / disable
netsh advfirewall set allprofiles state on
netsh advfirewall set allprofiles state off

:: Show all rules
netsh advfirewall firewall show rule name=all

:: Show rules for a specific port
netsh advfirewall firewall show rule name=all | findstr "22"

:: Add an inbound rule
netsh advfirewall firewall add rule name="Allow SSH" protocol=TCP dir=in localport=22 action=allow

:: Add an outbound rule
netsh advfirewall firewall add rule name="Block Telnet" protocol=TCP dir=out remoteport=23 action=block

:: Block an IP
netsh advfirewall firewall add rule name="Block IP" dir=in interface=any action=block remoteip=192.168.1.100

:: Delete a rule
netsh advfirewall firewall delete rule name="Allow SSH"

:: Reset to defaults
netsh advfirewall reset

:: PowerShell
Get-NetFirewallRule
Get-NetFirewallRule | Where-Object Enabled -eq True | Select-Object DisplayName, Direction, Action
New-NetFirewallRule -DisplayName "Allow SSH" -Direction Inbound -LocalPort 22 -Protocol TCP -Action Allow
Remove-NetFirewallRule -DisplayName "Allow SSH"
Set-NetFirewallRule -DisplayName "Allow SSH" -Enabled False
```

---

## 10. Registry

```cmd
:: Query a key
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion"
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Run"
reg query HKLM /f "password" /t REG_SZ /s    :: search recursively

:: Add a value
reg add "HKCU\Software\Test" /v TestValue /t REG_SZ /d "data"
reg add "HKLM\SOFTWARE\Test" /v DwordVal /t REG_DWORD /d 1

:: Delete a value
reg delete "HKCU\Software\Test" /v TestValue
reg delete "HKCU\Software\Test" /f         :: delete key and all subkeys

:: Export
reg export "HKLM\SOFTWARE\Test" C:\backup.reg

:: Import
reg import C:\backup.reg

:: Common security keys
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
reg query "HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
reg query "HKLM\SYSTEM\CurrentControlSet\Services"
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Lsa"

:: PowerShell
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
Get-ChildItem "HKLM:\SYSTEM\CurrentControlSet\Services"
Set-ItemProperty "HKCU:\Software\Test" -Name "Value" -Value "data"
New-ItemProperty "HKCU:\Software\Test" -Name "Value" -Value "data" -PropertyType String
Remove-ItemProperty "HKCU:\Software\Test" -Name "Value"
```

---

## 11. Scheduled Tasks

```cmd
:: List all scheduled tasks
schtasks /query /fo LIST
schtasks /query /fo TABLE /v
schtasks /query /fo CSV > tasks.csv

:: Show specific task
schtasks /query /tn "\Microsoft\Windows\WindowsUpdate\Automatic App Update"

:: Create a task
schtasks /create /tn "MyTask" /tr "C:\script.bat" /sc daily /st 09:00
schtasks /create /tn "MyTask" /tr "C:\script.bat" /sc onlogon /ru SYSTEM
schtasks /create /tn "MyTask" /tr "C:\script.bat" /sc minute /mo 15

:: Run a task immediately
schtasks /run /tn "MyTask"

:: Delete a task
schtasks /delete /tn "MyTask" /f

:: Enable / disable
schtasks /change /tn "MyTask" /enable
schtasks /change /tn "MyTask" /disable

:: PowerShell
Get-ScheduledTask
Get-ScheduledTask | Where-Object State -ne Disabled
Get-ScheduledTask -TaskName "MyTask" | Select-Object *
Get-ScheduledTask | Where-Object { $_.TaskPath -notlike "\Microsoft\*" }  :: non-Microsoft tasks
Register-ScheduledTask ...
Unregister-ScheduledTask -TaskName "MyTask" -Confirm:$false
```

---

## 12. Event Logs

```cmd
:: List available logs
wevtutil el

:: Query a log (last 10 entries)
wevtutil qe Security /c:10 /rd:true /f:text

:: Query by Event ID
wevtutil qe Security /q:"*[System[(EventID=4625)]]" /c:20 /f:text

:: Query by time
wevtutil qe Security /q:"*[System[TimeCreated[@SystemTime>='2024-01-01T00:00:00.000Z']]]" /f:text

:: Export log
wevtutil epl Security C:\backup\Security.evtx

:: Clear a log (requires elevation)
wevtutil cl Security

:: PowerShell
Get-WinEvent -LogName Security -MaxEvents 50
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625}
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624; StartTime=(Get-Date).AddHours(-24)}
Get-WinEvent -LogName Security | Where-Object { $_.Id -in @(4624,4625,4648,4672) }
Get-WinEvent -LogName Security | Export-Csv C:\events.csv -NoTypeInformation

:: Key Security Event IDs (quick reference)
:: 4624  - Successful logon
:: 4625  - Failed logon
:: 4648  - Logon with explicit credentials (runas)
:: 4672  - Special privileges assigned
:: 4698  - Scheduled task created
:: 4720  - User account created
:: 4732  - Member added to Administrators group
:: 7045  - New service installed (System log)
```

---

## 13. PowerShell Essentials

### Execution Policy

```powershell
# Check policy
Get-ExecutionPolicy
Get-ExecutionPolicy -List

# Set policy
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
Set-ExecutionPolicy Bypass -Scope Process        # current session only

# Run script bypassing policy (attacker technique)
powershell.exe -ExecutionPolicy Bypass -File script.ps1
powershell.exe -ep bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://...')"
```

### Variables & Data Types

```powershell
# Variables
$name = "Alice"
$number = 42
$array = @("item1", "item2", "item3")
$hash = @{ Key = "Value"; Key2 = "Value2" }

# String operations
"Hello $name"                           # interpolation
"Path: " + $env:SystemRoot
$str.ToUpper()
$str.Split(",")
$str -replace "old", "new"
$str -like "*pattern*"                  # wildcard match
$str -match "regex"                     # regex match

# Arrays
$array[0]
$array.Count
$array += "item4"
$array | Where-Object { $_ -like "item*" }

# Hashtables
$hash["Key"]
$hash.Key
$hash.Keys
$hash.Values
```

### Flow Control

```powershell
# If / elseif / else
if ($value -eq 0) {
    Write-Host "Zero"
} elseif ($value -gt 0) {
    Write-Host "Positive"
} else {
    Write-Host "Negative"
}

# Comparison operators
# -eq   equal
# -ne   not equal
# -gt   greater than
# -lt   less than
# -ge   greater or equal
# -le   less or equal
# -like wildcard match
# -match regex match
# -in   value in collection
# -contains collection contains value

# For loop
for ($i = 0; $i -lt 10; $i++) {
    Write-Host $i
}

# ForEach
foreach ($item in $array) {
    Write-Host $item
}

# ForEach-Object (pipeline)
Get-Process | ForEach-Object { Write-Host $_.Name }
Get-Process | ForEach-Object Name        # shorthand

# Where-Object (filter)
Get-Process | Where-Object { $_.CPU -gt 10 }
Get-Process | Where-Object CPU -gt 10   # simplified syntax

# While
while ($condition) {
    # ...
}
```

### Functions

```powershell
function Get-OpenPorts {
    param(
        [string]$ComputerName = "localhost",
        [int[]]$Ports = @(22, 80, 443, 3389)
    )
    foreach ($port in $Ports) {
        $result = Test-NetConnection -ComputerName $ComputerName -Port $port -WarningAction SilentlyContinue
        [PSCustomObject]@{
            Host = $ComputerName
            Port = $port
            Open = $result.TcpTestSucceeded
        }
    }
}

Get-OpenPorts -ComputerName "192.168.1.1" -Ports @(22, 80, 443)
```

### Useful Pipeline Patterns

```powershell
# Select specific properties
Get-Process | Select-Object Name, Id, CPU

# Sort
Get-Process | Sort-Object CPU -Descending

# Group
Get-Process | Group-Object -Property Company

# Measure
Get-Process | Measure-Object CPU -Sum -Average -Maximum

# Format output
Get-Process | Format-Table Name, Id, CPU -AutoSize
Get-Process | Format-List *

# Export
Get-Process | Export-Csv processes.csv -NoTypeInformation
Get-Process | ConvertTo-Json | Out-File processes.json

# Filter with multiple conditions
Get-Process | Where-Object { $_.CPU -gt 10 -and $_.Name -notlike "idle" }
```

### Remote Execution

```powershell
# One-off remote command
Invoke-Command -ComputerName target -ScriptBlock { whoami; hostname }
Invoke-Command -ComputerName target -Credential (Get-Credential) -ScriptBlock { Get-Process }

# Interactive remote session
Enter-PSSession -ComputerName target
Enter-PSSession -ComputerName target -Credential $cred

# Run script on remote host
Invoke-Command -ComputerName target -FilePath C:\script.ps1

# Multiple targets
Invoke-Command -ComputerName server1, server2, server3 -ScriptBlock { ipconfig }
```

---

## 14. WMI & CIM

WMI (Windows Management Instrumentation) provides deep access to system information and configuration.

```powershell
# System info
Get-WmiObject Win32_OperatingSystem | Select-Object Caption, Version, BuildNumber
Get-WmiObject Win32_ComputerSystem | Select-Object Name, Domain, Manufacturer, Model
Get-WmiObject Win32_BIOS | Select-Object SerialNumber, SMBIOSBIOSVersion

# Hardware
Get-WmiObject Win32_Processor | Select-Object Name, NumberOfCores, MaxClockSpeed
Get-WmiObject Win32_PhysicalMemory | Select-Object Capacity, Speed
Get-WmiObject Win32_DiskDrive | Select-Object Model, Size, InterfaceType

# Running processes (with command line - useful for IR)
Get-WmiObject Win32_Process | Select-Object Name, ProcessId, ParentProcessId, CommandLine
Get-WmiObject Win32_Process | Where-Object { $_.CommandLine -like "*powershell*" }

# Services
Get-WmiObject Win32_Service | Select-Object Name, State, StartMode, PathName
Get-WmiObject Win32_Service | Where-Object { $_.State -eq "Running" }

# Installed software
Get-WmiObject Win32_Product | Select-Object Name, Version, InstallDate
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | Select-Object DisplayName, DisplayVersion

# Network adapters
Get-WmiObject Win32_NetworkAdapterConfiguration | Where-Object { $_.IPEnabled } | Select-Object Description, IPAddress, MACAddress

# Startup items
Get-WmiObject Win32_StartupCommand | Select-Object Name, Command, Location, User

# Shares
Get-WmiObject Win32_Share

# Logged-in users
Get-WmiObject Win32_ComputerSystem | Select-Object UserName

# Modern CIM equivalents (preferred)
Get-CimInstance Win32_OperatingSystem
Get-CimInstance Win32_Process
Get-CimInstance Win32_Service | Where-Object State -eq "Running"
```

### WMI for Persistence (Detection)

```powershell
# List WMI event subscriptions (common persistence mechanism)
Get-WMIObject -Namespace root\subscription -Class __EventFilter
Get-WMIObject -Namespace root\subscription -Class __EventConsumer
Get-WMIObject -Namespace root\subscription -Class __FilterToConsumerBinding
```

---

## 15. Active Directory (CMD & PowerShell)

### Basic Domain Enumeration

```cmd
:: Domain info
net user /domain
net group /domain
net group "Domain Admins" /domain
net group "Enterprise Admins" /domain

:: Domain controllers
nltest /dclist:domain.local
netdom query dc

:: Current domain
echo %USERDOMAIN%
echo %LOGONSERVER%
```

### PowerShell AD Module

```powershell
# Import module (requires RSAT or running on DC)
Import-Module ActiveDirectory

# Domain info
Get-ADDomain
Get-ADForest

# Users
Get-ADUser -Filter * | Select-Object Name, SamAccountName, Enabled
Get-ADUser -Identity alice -Properties *
Get-ADUser -Filter { Enabled -eq $true } | Select-Object Name, LastLogonDate
Get-ADUser -Filter { PasswordNeverExpires -eq $true } | Select-Object Name

# Computers
Get-ADComputer -Filter * | Select-Object Name, OperatingSystem
Get-ADComputer -Filter { OperatingSystem -like "*Server*" }

# Groups
Get-ADGroup -Filter * | Select-Object Name, GroupScope
Get-ADGroupMember "Domain Admins" | Select-Object Name, SamAccountName
Get-ADGroupMember "Domain Admins" -Recursive

# Organizational Units
Get-ADOrganizationalUnit -Filter *

# Find accounts with SPNs (Kerberoasting targets)
Get-ADUser -Filter { ServicePrincipalName -ne "$null" } -Properties ServicePrincipalName | Select-Object Name, ServicePrincipalName

# Find accounts with no Kerberos pre-auth (ASREPRoast targets)
Get-ADUser -Filter { DoesNotRequirePreAuth -eq $true } | Select-Object Name

# Domain password policy
Get-ADDefaultDomainPasswordPolicy

# Fine-grained password policies
Get-ADFineGrainedPasswordPolicy -Filter *

# Locked-out accounts
Search-ADAccount -LockedOut

# Inactive accounts (not logged in 90 days)
Search-ADAccount -AccountInactive -TimeSpan 90.00:00:00 -UsersOnly
```

### LDAP Queries (No AD Module)

```powershell
# LDAP query using .NET
$searcher = New-Object DirectoryServices.DirectorySearcher
$searcher.Filter = "(&(objectClass=user)(objectCategory=person))"
$searcher.FindAll() | ForEach-Object { $_.Properties["samaccountname"] }

# Find Domain Admins
$searcher.Filter = "(&(objectClass=user)(memberOf=CN=Domain Admins,CN=Users,DC=domain,DC=local))"
```

---

## 16. Credential & Authentication Commands

```cmd
:: View stored credentials
cmdkey /list

:: Add stored credential
cmdkey /add:server01 /user:domain\alice /pass:Password123

:: Delete stored credential
cmdkey /delete:server01

:: Run with stored credential
runas /savecred /user:administrator cmd

:: Manage certificates
certmgr.msc            :: GUI
certutil -store My      :: list personal certs
certutil -hashfile file.exe SHA256   :: hash a file

:: Decode base64 (certutil abuse)
certutil -decode encoded.txt decoded.exe
```

```powershell
# Credential object
$cred = Get-Credential
$cred = New-Object PSCredential("username", (ConvertTo-SecureString "Password" -AsPlainText -Force))

# NTLM hash of a string (for testing)
$password = "Password123"
$bytes = [System.Text.Encoding]::Unicode.GetBytes($password)
[System.BitConverter]::ToString([System.Security.Cryptography.MD4]::Create().ComputeHash($bytes)) -replace "-"

# Credential Manager via PowerShell (requires CredentialManager module)
Get-StoredCredential
```

---

## 17. Text Processing

```cmd
:: Search within files (findstr)
findstr "password" C:\config.txt
findstr /i "error" C:\*.log            :: case-insensitive
findstr /s "password" C:\*.config      :: recursive
findstr /r "[0-9]\{1,3\}\.[0-9]\{1,3\}" file.txt   :: regex
findstr /v "^#" C:\config.txt          :: exclude comment lines
findstr /c:"exact phrase" file.txt     :: exact phrase

:: Count lines
find /c "error" logfile.txt

:: More / type
type file.txt | more                    :: paginated
type file.txt | findstr /i "error"

:: PowerShell text processing
Select-String -Path *.log -Pattern "error" -CaseSensitive
Select-String -Path C:\logs\*.log -Pattern "Failed password"
Get-Content auth.log | Select-String "Failed" | Measure-Object   :: count
Get-Content file.txt | Where-Object { $_ -notmatch "^#" }        :: exclude comments

# Top IPs from a log
Get-Content access.log |
    ForEach-Object { ($_ -split " ")[0] } |
    Group-Object | Sort-Object Count -Descending |
    Select-Object -First 20 Count, Name
```

---

## 18. Hashing & Encoding

```powershell
# File hash
Get-FileHash file.exe
Get-FileHash file.exe -Algorithm MD5
Get-FileHash file.exe -Algorithm SHA1
Get-FileHash file.exe -Algorithm SHA256
Get-FileHash file.exe -Algorithm SHA256 | Select-Object Hash

# Verify a file against known hash
$expected = "ABC123..."
$actual = (Get-FileHash file.exe -Algorithm SHA256).Hash
if ($expected -eq $actual) { "Hash matches" } else { "MISMATCH" }

# Hash a string
$string = "Hello World"
$bytes = [System.Text.Encoding]::UTF8.GetBytes($string)
$hash = [System.Security.Cryptography.SHA256]::Create().ComputeHash($bytes)
[System.BitConverter]::ToString($hash) -replace "-"

# Base64 encode
[System.Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("hello"))

# Base64 decode
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("aGVsbG8="))

# Base64 encode a command (common attacker technique)
$cmd = "whoami"
$encoded = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($cmd))
powershell.exe -EncodedCommand $encoded
```

```cmd
:: certutil hashing
certutil -hashfile file.exe MD5
certutil -hashfile file.exe SHA256
```

---

## 19. Autoruns & Persistence Locations

### Checking Autorun Locations

```powershell
# Run registry keys
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce"
Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce"

# Winlogon keys (shell, userinit hijacking)
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" |
    Select-Object Shell, Userinit

# Services
Get-WmiObject Win32_Service | Select-Object Name, StartMode, PathName, State

# Scheduled tasks (non-Microsoft)
Get-ScheduledTask | Where-Object { $_.TaskPath -notlike "\Microsoft\*" } |
    Select-Object TaskName, TaskPath, @{n='Action';e={$_.Actions.Execute}}

# Startup folders
Get-ChildItem "C:\Users\$env:USERNAME\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup"
Get-ChildItem "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"

# WMI subscriptions
Get-WMIObject -Namespace root\subscription -Class __EventFilter
Get-WMIObject -Namespace root\subscription -Class __EventConsumer

# All startup items via WMI
Get-WmiObject Win32_StartupCommand | Select-Object Name, Command, Location, User
```

---

## 20. Privilege Escalation (Awareness)

### Recon Commands

```cmd
:: System info for exploit matching
systeminfo
systeminfo | findstr /B /C:"OS Name" /C:"OS Version"

:: Current privileges
whoami /priv
whoami /groups

:: Installed patches (find missing patches)
wmic qfe list brief
systeminfo | findstr "KB"

:: Unquoted service paths
wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v """

:: Weak service permissions
accesschk.exe -uwcqv "Everyone" *
accesschk.exe -uwcqv "Authenticated Users" *

:: AlwaysInstallElevated
reg query "HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer" /v AlwaysInstallElevated
reg query "HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer" /v AlwaysInstallElevated
```

```powershell
# Weak registry permissions on services
Get-Acl "HKLM:\SYSTEM\CurrentControlSet\Services\VulnService"

# Writable directories in system PATH
$env:PATH -split ";" | ForEach-Object {
    if (Test-Path $_) {
        $acl = Get-Acl $_ -ErrorAction SilentlyContinue
        $acl.Access | Where-Object { $_.IdentityReference -match "Everyone|Users|Authenticated" -and $_.FileSystemRights -match "Write|FullControl" } |
            ForEach-Object { Write-Host "Writable: $_" }
    }
}

# SeImpersonatePrivilege (Potato attacks)
whoami /priv | Select-String "SeImpersonatePrivilege|SeAssignPrimaryTokenPrivilege"

# Token privileges
[System.Security.Principal.WindowsIdentity]::GetCurrent()
```

---

## 21. Defense & Security Hardening

### Quick Security Checks

```powershell
# Check UAC level
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" |
    Select-Object EnableLUA, ConsentPromptBehaviorAdmin

# Check if Defender is running
Get-MpComputerStatus | Select-Object AMServiceEnabled, AntispywareEnabled, AntivirusEnabled, RealTimeProtectionEnabled

# Defender status
Get-MpComputerStatus

# Check SMBv1 (should be disabled)
Get-SmbServerConfiguration | Select-Object EnableSMB1Protocol

# Check Windows Update
Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 10

# Audit policy
auditpol /get /category:*

# Local security policy values
secedit /export /cfg C:\secpolicy.cfg
```

### Disabling Dangerous Features

```powershell
# Disable SMBv1
Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force

# Disable PowerShell v2 (bypasses AMSI)
Disable-WindowsOptionalFeature -Online -FeatureName MicrosoftWindowsPowerShellV2Root -NoRestart

# Enable Credential Guard (requires reboot + hardware support)
# Via Group Policy: Computer Configuration > Admin Templates > System > Device Guard

# Enable LSASS protection
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RunAsPPL" -Value 1

# Enable audit process creation
auditpol /set /subcategory:"Process Creation" /success:enable

# Enable command line logging in process creation events
Set-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit" `
    -Name "ProcessCreationIncludeCmdLine_Enabled" -Value 1

# Enable PowerShell logging
Set-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" `
    -Name "EnableScriptBlockLogging" -Value 1
Set-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging" `
    -Name "EnableModuleLogging" -Value 1
```

---

## 22. Forensics & Incident Response

### Volatile Data Collection

```powershell
# Create IR output directory
$IRDir = "C:\IR_$(Get-Date -Format 'yyyyMMdd_HHmmss')"
New-Item -ItemType Directory -Path $IRDir

# System info
systeminfo | Out-File "$IRDir\systeminfo.txt"

# Running processes (with full details)
Get-WmiObject Win32_Process | Select-Object Name, ProcessId, ParentProcessId, CommandLine, ExecutablePath |
    Export-Csv "$IRDir\processes.csv" -NoTypeInformation

# Network connections
Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess |
    Export-Csv "$IRDir\connections.csv" -NoTypeInformation

# Listening ports with process
netstat -anob | Out-File "$IRDir\netstat.txt"

# Services
Get-WmiObject Win32_Service | Select-Object Name, State, StartMode, PathName |
    Export-Csv "$IRDir\services.csv" -NoTypeInformation

# Scheduled tasks
Get-ScheduledTask | Export-Csv "$IRDir\scheduled_tasks.csv" -NoTypeInformation

# Autoruns
Get-WmiObject Win32_StartupCommand | Export-Csv "$IRDir\autoruns.csv" -NoTypeInformation

# Logged-in users
query user | Out-File "$IRDir\logged_users.txt"

# Recent logon events
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624; StartTime=(Get-Date).AddDays(-1)} |
    Export-Csv "$IRDir\logon_events.csv" -NoTypeInformation

# Installed software
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" |
    Select-Object DisplayName, DisplayVersion, InstallDate |
    Export-Csv "$IRDir\installed_software.csv" -NoTypeInformation

# Hotfixes
Get-HotFix | Export-Csv "$IRDir\hotfixes.csv" -NoTypeInformation

Write-Host "IR data collected to $IRDir"
```

### Investigating a Suspicious Process

```powershell
# Full process details
$PID = 1234
Get-WmiObject Win32_Process -Filter "ProcessId = $PID" |
    Select-Object Name, ProcessId, ParentProcessId, CommandLine, ExecutablePath, CreationDate

# Verify executable signature
$path = (Get-Process -Id $PID).Path
Get-AuthenticodeSignature $path

# Hash the executable
Get-FileHash $path -Algorithm SHA256

# Parent process
$ppid = (Get-WmiObject Win32_Process -Filter "ProcessId = $PID").ParentProcessId
Get-WmiObject Win32_Process -Filter "ProcessId = $ppid" | Select-Object Name, ProcessId, CommandLine

# Open network connections
Get-NetTCPConnection | Where-Object OwningProcess -eq $PID

# DLLs loaded by process (Sysinternals listdlls, or via WMI is limited)
# Use Sysinternals: listdlls.exe -d <pid>
```

### Key Forensic Artifacts

```powershell
# Prefetch (evidence of execution)
Get-ChildItem C:\Windows\Prefetch\ | Sort-Object LastWriteTime -Descending | Select-Object -First 20

# Recently accessed files
Get-ChildItem "$env:APPDATA\Microsoft\Windows\Recent\" | Sort-Object LastWriteTime -Descending

# Browser history locations
# Chrome: $env:LOCALAPPDATA\Google\Chrome\User Data\Default\History
# Firefox: $env:APPDATA\Mozilla\Firefox\Profiles\*.default\places.sqlite
# Edge:  $env:LOCALAPPDATA\Microsoft\Edge\User Data\Default\History

# Recycle Bin
Get-ChildItem "C:\`$Recycle.Bin\" -Force -Recurse

# Shadow copies
vssadmin list shadows
Get-WmiObject Win32_ShadowCopy | Select-Object ID, InstallDate, VolumeName

# AmCache (program execution evidence)
# Located at: C:\Windows\AppCompat\Programs\Amcache.hve
# Parse with: AmcacheParser.exe (Eric Zimmermann tools)

# Recently modified files in sensitive paths
Get-ChildItem C:\Windows\System32 -File |
    Where-Object { $_.LastWriteTime -gt (Get-Date).AddDays(-7) } |
    Sort-Object LastWriteTime -Descending | Select-Object Name, LastWriteTime

# Files in temp directories (common drop locations)
Get-ChildItem C:\Windows\Temp, $env:TEMP -File -ErrorAction SilentlyContinue |
    Sort-Object CreationTime -Descending
```

### Quick Triage One-Liners

```powershell
# Unsigned running processes
Get-Process | Where-Object { $_.Path } | ForEach-Object {
    $sig = Get-AuthenticodeSignature $_.Path -ErrorAction SilentlyContinue
    if ($sig.Status -ne "Valid") {
        [PSCustomObject]@{ Name=$_.Name; PID=$_.Id; Status=$sig.Status; Path=$_.Path }
    }
} | Format-Table -AutoSize

# Processes with network connections
Get-NetTCPConnection -State Established | ForEach-Object {
    $proc = Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue
    [PSCustomObject]@{
        Process = $proc.Name
        PID = $_.OwningProcess
        LocalPort = $_.LocalPort
        RemoteAddress = $_.RemoteAddress
        RemotePort = $_.RemotePort
    }
} | Sort-Object RemoteAddress | Format-Table -AutoSize

# Non-Microsoft scheduled tasks with actions
Get-ScheduledTask | Where-Object { $_.TaskPath -notlike "\Microsoft\*" } | ForEach-Object {
    [PSCustomObject]@{
        Task = $_.TaskName
        Path = $_.TaskPath
        Action = ($_.Actions | ForEach-Object { $_.Execute }) -join "; "
        State = $_.State
    }
} | Format-Table -AutoSize

# UID 0 equivalent — local admin accounts
Get-LocalGroupMember -Group "Administrators" | Select-Object Name, PrincipalSource

# New accounts created in last 7 days
Get-LocalUser | Where-Object { $_.PasswordLastSet -gt (Get-Date).AddDays(-7) }
```

---

## Quick Reference

### CMD Essentials

```
Navigation : cd, dir, tree, pushd, popd
Files      : copy, move, del, ren, xcopy, robocopy, type, more
Users      : net user, net localgroup, whoami
Network    : ipconfig, netstat, ping, tracert, nslookup, arp, route
Processes  : tasklist, taskkill
Services   : sc, net start/stop
Registry   : reg query/add/delete/export
Logs       : wevtutil
Search     : dir /s, findstr
```

### PowerShell Essentials

```
Files      : Get-ChildItem, Copy-Item, Move-Item, Remove-Item, Get-Content
Users      : Get-LocalUser, Get-LocalGroupMember
Network    : Get-NetTCPConnection, Test-NetConnection, Get-NetAdapter
Processes  : Get-Process, Stop-Process, Start-Process
Services   : Get-Service, Start-Service, Stop-Service
Registry   : Get-ItemProperty, Set-ItemProperty
Events     : Get-WinEvent
WMI        : Get-WmiObject, Get-CimInstance
Security   : Get-AuthenticodeSignature, Get-FileHash, Get-MpComputerStatus
AD         : Get-ADUser, Get-ADGroup, Get-ADComputer
```

---

## Further Reading

- [Microsoft Docs – Command-Line Reference](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands)
- [Microsoft Docs – PowerShell](https://docs.microsoft.com/en-us/powershell/)
- [LOLBAS Project](https://lolbas-project.github.io/)
- [PayloadsAllTheThings – Windows PrivEsc](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)
- [MITRE ATT&CK – Windows](https://attack.mitre.org/matrices/enterprise/windows/)
- [Sysinternals Suite](https://docs.microsoft.com/en-us/sysinternals/)
- [Eric Zimmermann Forensic Tools](https://ericzimmerman.github.io/)