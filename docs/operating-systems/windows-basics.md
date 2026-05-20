# Windows Basics for Cybersecurity

A comprehensive reference covering Windows internals, architecture, security features, attack surfaces, and defensive techniques — from foundational concepts to advanced topics used in penetration testing and incident response.

---

## 1. Windows Architecture

### Kernel vs. User Mode

Windows operates in two privilege levels:

| Mode | Description | Examples |
|------|-------------|---------|
| **Kernel Mode** | Full hardware access, unrestricted | NT kernel, drivers, HAL |
| **User Mode** | Restricted, isolated per process | Applications, Win32 subsystem |

Transitions between modes (syscalls) are a key security boundary.

### Major Components

```
┌─────────────────────────────────────────────┐
│              User Mode                       │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐  │
│  │  Win32   │  │ Services │  │   Apps    │  │
│  │Subsystem │  │ (svchost)│  │ (user)    │  │
│  └────┬─────┘  └────┬─────┘  └─────┬─────┘  │
├───────┼─────────────┼──────────────┼─────────┤
│       │       Kernel Mode          │         │
│  ┌────▼─────────────▼──────────────▼──────┐  │
│  │         Windows NT Executive           │  │
│  │  (I/O Manager, Memory Manager,         │  │
│  │   Process Manager, Security Ref Mon)   │  │
│  └────────────────┬───────────────────────┘  │
│  ┌────────────────▼───────────────────────┐  │
│  │         NT Kernel (ntoskrnl.exe)       │  │
│  └────────────────┬───────────────────────┘  │
│  ┌────────────────▼───────────────────────┐  │
│  │     Hardware Abstraction Layer (HAL)   │  │
└──┴────────────────────────────────────────┴──┘
                  Hardware
```

### Key System Files

| File | Location | Role |
|------|----------|------|
| `ntoskrnl.exe` | `\Windows\System32\` | NT kernel and executive |
| `hal.dll` | `\Windows\System32\` | Hardware abstraction |
| `lsass.exe` | `\Windows\System32\` | Authentication, stores credentials |
| `csrss.exe` | `\Windows\System32\` | Win32 subsystem |
| `winlogon.exe` | `\Windows\System32\` | Login/logout, SAS handling |
| `services.exe` | `\Windows\System32\` | Service Control Manager |
| `svchost.exe` | `\Windows\System32\` | Host process for Windows services |
| `explorer.exe` | `\Windows\` | Shell / desktop |

**Security note:** Malware commonly masquerades as these processes. Verify path, parent process, and digital signature.

---

## 2. File System

### NTFS

Windows primarily uses **NTFS (New Technology File System)**. Key security features:

- **Access Control Lists (ACLs)** – Every file and folder has a DACL (Discretionary ACL) and SACL (System ACL)
- **Alternate Data Streams (ADS)** – Hidden data streams attached to files (used by malware to hide payloads)
- **NTFS Permissions** – Full Control, Modify, Read & Execute, Read, Write
- **Encryption (EFS)** – Per-file encryption tied to user certificates

### Important Directories

| Path | Purpose |
|------|---------|
| `C:\Windows\System32\` | Core system binaries (64-bit) |
| `C:\Windows\SysWOW64\` | 32-bit binaries on 64-bit Windows |
| `C:\Windows\Temp\` | Temporary files (common malware drop location) |
| `C:\Users\<user>\AppData\` | Per-user app data (Roaming, Local, LocalLow) |
| `C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\` | Per-user startup folder |
| `C:\ProgramData\` | Shared app data across users |
| `C:\Windows\Prefetch\` | Prefetch files (forensic evidence of execution) |
| `C:\$Recycle.Bin\` | Recycle bin (forensic artifact) |
| `C:\Windows\System32\drivers\etc\hosts` | Local DNS override |

### Alternate Data Streams (ADS)

ADS allow data to be hidden inside a file without affecting its visible size:

```cmd
# Create an ADS
echo malicious > harmless.txt:hidden_stream

# List ADS (cmd)
dir /r harmless.txt

# Access ADS
more < harmless.txt:hidden_stream

# PowerShell - list all ADS in a directory
Get-Item -Path .\* -Stream *
```

**Security relevance:** Malware may store payloads or configuration in ADS. Mark-of-the-Web (MoTW) is stored as `Zone.Identifier` ADS on downloaded files.

### File Permissions

```cmd
# View file ACL
icacls C:\path\to\file

# Set permissions
icacls C:\path\to\file /grant username:(R)

# Take ownership
takeown /f C:\path\to\file
```

---

## 3. Registry

The Windows Registry is a hierarchical database storing OS and application configuration.

### Hive Structure

| Hive | Abbreviation | Contents |
|------|-------------|---------|
| `HKEY_LOCAL_MACHINE` | HKLM | Machine-wide settings, services, drivers |
| `HKEY_CURRENT_USER` | HKCU | Settings for the currently logged-in user |
| `HKEY_USERS` | HKU | Settings for all user profiles |
| `HKEY_CLASSES_ROOT` | HKCR | File associations, COM objects |
| `HKEY_CURRENT_CONFIG` | HKCC | Current hardware profile |

### Registry Files on Disk

| Hive | File Location |
|------|--------------|
| HKLM\SYSTEM | `C:\Windows\System32\config\SYSTEM` |
| HKLM\SAM | `C:\Windows\System32\config\SAM` |
| HKLM\SECURITY | `C:\Windows\System32\config\SECURITY` |
| HKLM\SOFTWARE | `C:\Windows\System32\config\SOFTWARE` |
| HKCU | `C:\Users\<user>\NTUSER.DAT` |

### Security-Relevant Registry Keys

**Autorun / Persistence:**
```
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
```

**Services:**
```
HKLM\SYSTEM\CurrentControlSet\Services\
```

**Installed software:**
```
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\
```

**User accounts (SAM):**
```
HKLM\SAM\SAM\Domains\Account\Users\
```

**LSA secrets:**
```
HKLM\SECURITY\Policy\Secrets\
```

**UAC settings:**
```
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\EnableLUA
```

### Registry Commands

```cmd
# Query a key
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion"

# Add a key
reg add "HKCU\Software\Test" /v TestValue /t REG_SZ /d "data"

# Delete a key
reg delete "HKCU\Software\Test" /v TestValue

# Export a hive
reg export HKLM\SOFTWARE C:\backup\software.reg
```

---

## 4. User Accounts & Groups

### Account Types

| Type | Description |
|------|-------------|
| **Local account** | Exists only on the local machine |
| **Domain account** | Managed by Active Directory |
| **Built-in accounts** | Administrator, Guest, SYSTEM, LOCAL SERVICE, NETWORK SERVICE |
| **Service accounts** | Used by services to run with specific privileges |
| **Managed Service Accounts (MSA/gMSA)** | AD-managed, auto-rotating passwords |

### Built-in Security Principals

| Account | SID | Purpose |
|---------|-----|---------|
| `SYSTEM` | S-1-5-18 | Highest local privilege; used by OS |
| `LOCAL SERVICE` | S-1-5-19 | Reduced privilege service account |
| `NETWORK SERVICE` | S-1-5-20 | Network access, reduced local privilege |
| `Everyone` | S-1-1-0 | All users including anonymous |
| `Authenticated Users` | S-1-5-11 | All users who authenticated |
| `BUILTIN\Administrators` | S-1-5-32-544 | Local admin group |

### Security Identifiers (SIDs)

SIDs uniquely identify security principals:

```
S-1-5-21-<domain>-500   → Built-in Administrator
S-1-5-21-<domain>-501   → Guest
S-1-5-21-<domain>-1000+ → Regular users (RID ≥ 1000)
```

### User Management Commands

```cmd
# List local users
net user

# Show details of a user
net user username

# List local groups
net localgroup

# List members of a group
net localgroup Administrators

# Add user to Administrators
net localgroup Administrators username /add

# Create a new user
net user newuser Password123 /add

# PowerShell - list local users
Get-LocalUser

# PowerShell - list local groups
Get-LocalGroup
```

---

## 5. Authentication & Credential Storage

### Authentication Protocols

| Protocol | Description | Security |
|----------|-------------|---------|
| **NTLM** | Challenge-response; older protocol | Weak — vulnerable to pass-the-hash, relay attacks |
| **NTLMv2** | Improved version of NTLM | Better, but still vulnerable to relay |
| **Kerberos** | Ticket-based; default for domain | Strong — preferred for AD environments |
| **LDAP** | Used for directory lookups | Can be secured with LDAPS (LDAP over TLS) |

### NTLM Authentication Flow

```
Client                        Server
  │── Request resource ──────▶│
  │◀── NTLM Challenge ─────── │  (random 8-byte nonce)
  │── NTLM Response ─────────▶│  (hash of password + challenge)
  │◀── Accept/Deny ─────────── │
```

**Attack:** An attacker can capture the NTLM challenge/response and crack it offline, or relay it to another service.

### Kerberos Authentication Flow

```
Client          KDC (Key Distribution Center)        Service
  │─── AS-REQ (TGT request) ──────▶│
  │◀── AS-REP (TGT) ──────────────── │
  │─── TGS-REQ (service ticket) ───▶│
  │◀── TGS-REP (service ticket) ───── │
  │──────────── AP-REQ ────────────────────────────▶│
  │◀─────────── AP-REP ──────────────────────────── │
```

### Credential Storage

| Storage | Location | Contents |
|---------|----------|---------|
| **SAM database** | `C:\Windows\System32\config\SAM` | Local user NTLM hashes |
| **NTDS.dit** | `C:\Windows\NTDS\NTDS.dit` (DC only) | All AD user hashes |
| **LSA Secrets** | `HKLM\SECURITY\Policy\Secrets\` | Service account passwords, cached domain credentials |
| **Credential Manager** | `%AppData%\Microsoft\Credentials\` | Saved Windows/web credentials |
| **LSASS process memory** | In memory at runtime | Plaintext passwords, NTLM hashes, Kerberos tickets (pre-Win8.1) |

### Cached Credentials

Windows caches domain credentials locally (default: last 10 logons) so users can log in offline. Stored as MS-CACHEv2 hashes:

```
HKLM\SECURITY\Cache
```

**Security note:** Cached credential hashes are slower to crack than NT hashes but still crackable.

### Windows Hello & Modern Auth

- **Windows Hello** – Biometric/PIN authentication, backed by TPM
- **FIDO2/WebAuthn** – Passwordless authentication standard
- **Azure AD / Entra ID** – Cloud-based identity with MFA support

---

## 6. Processes & Threads

### Process Concepts

Every process has:
- A unique **PID** (Process ID)
- A **parent PID (PPID)**
- A **security token** (determines privileges)
- Its own **virtual address space**
- One or more **threads**

### Security Token

A security token attached to each process/thread contains:
- User SID
- Group SIDs
- Privileges (e.g. `SeDebugPrivilege`, `SeImpersonatePrivilege`)
- Integrity level (Low, Medium, High, System)

### Integrity Levels (Mandatory Integrity Control)

| Level | Usage |
|-------|-------|
| Untrusted (0) | Anonymous processes |
| Low (1) | Sandboxed apps (e.g. browser tabs) |
| Medium (2) | Normal user processes |
| High (3) | Elevated (UAC elevated) processes |
| System (4) | OS processes running as SYSTEM |

### Process Commands

```cmd
# List running processes
tasklist

# List processes with services
tasklist /svc

# Kill a process
taskkill /PID 1234 /F

# PowerShell - detailed process info
Get-Process | Select-Object Name, Id, Path

# Show parent-child relationships (Sysinternals)
pslist -t

# Check process integrity level (Sysinternals)
accesschk -p <PID>
```

### Important Process Relationships

```
System (PID 4)
└── smss.exe         (Session Manager)
    └── csrss.exe    (Win32 subsystem)
    └── winlogon.exe (Login)
        └── userinit.exe
            └── explorer.exe (Shell)
    └── services.exe (SCM)
        └── svchost.exe (many instances, each hosting services)
    └── lsass.exe    (Authentication)
```

**Security note:** A `cmd.exe` with parent `winword.exe` is suspicious — indicates macro-based code execution.

---

## 7. Services

### What Are Services?

Windows services are long-running executables that perform specific functions. Managed by the **Service Control Manager (SCM)** (`services.exe`).

### Service Types

| Type | Description |
|------|-------------|
| **Win32OwnProcess** | Runs in its own process |
| **Win32ShareProcess** | Shares a process with other services (svchost.exe) |
| **Kernel driver** | Runs in kernel mode |
| **File system driver** | Special kernel driver for file systems |

### Service Start Types

| Type | Value | Description |
|------|-------|-------------|
| Automatic | 2 | Starts at boot |
| Manual | 3 | Started on demand |
| Disabled | 4 | Cannot be started |
| Automatic (Delayed Start) | 2 | Starts shortly after boot |

### Service Commands

```cmd
# List all services
sc query type= all

# Query a specific service
sc query wuauserv

# Start / stop a service
sc start <service_name>
sc stop <service_name>

# Show service config (binary path, start type)
sc qc <service_name>

# PowerShell
Get-Service
Get-Service -Name "wuauserv" | Select-Object *
```

### Service Security

Services run under specific accounts. The account determines privilege level:
- **LocalSystem (SYSTEM)** – Most privileged; avoid unless necessary
- **LocalService** – Reduced privileges
- **NetworkService** – Network access with reduced local privileges
- **Custom service account** – Best practice; principle of least privilege

---

## 8. Networking

### Key Networking Commands

```cmd
# IP configuration
ipconfig /all

# DNS cache
ipconfig /displaydns

# Flush DNS cache
ipconfig /flushdns

# Active connections
netstat -ano

# Show open ports with owning process
netstat -anob

# ARP table
arp -a

# Routing table
route print

# DNS lookup
nslookup example.com

# Trace route
tracert example.com

# Test connectivity
ping example.com
Test-NetConnection -ComputerName example.com -Port 443  # PowerShell
```

### `netstat` Output Fields

```
Proto  Local Address    Foreign Address   State     PID
TCP    0.0.0.0:445      0.0.0.0:0         LISTENING  4
TCP    192.168.1.5:52341 93.184.216.34:443 ESTABLISHED 1234
```

**Security note:** Unknown `ESTABLISHED` connections to external IPs, or unusual `LISTENING` ports, are indicators of compromise.

### SMB (Server Message Block)

SMB is Windows' native file-sharing and printer-sharing protocol.

| Version | Port | Notes |
|---------|------|-------|
| SMBv1 | TCP 445 / 139 | Legacy; highly vulnerable (EternalBlue/WannaCry) |
| SMBv2/v3 | TCP 445 | Current; should be used exclusively |

```cmd
# Check SMB version support
Get-SmbServerConfiguration | Select-Object EnableSMB1Protocol, EnableSMB2Protocol

# Disable SMBv1
Set-SmbServerConfiguration -EnableSMB1Protocol $false
```

### Common Windows Ports

| Port | Protocol | Service |
|------|----------|---------|
| 53 | TCP/UDP | DNS |
| 88 | TCP/UDP | Kerberos |
| 135 | TCP | RPC Endpoint Mapper |
| 139 | TCP | NetBIOS Session Service |
| 389 | TCP | LDAP |
| 443 | TCP | HTTPS |
| 445 | TCP | SMB |
| 636 | TCP | LDAPS |
| 3389 | TCP | RDP |
| 5985/5986 | TCP | WinRM (HTTP/HTTPS) |

---

## 9. Windows Firewall

### Overview

Windows Defender Firewall filters inbound and outbound traffic based on rules. Supports three profiles:

| Profile | Used When |
|---------|----------|
| **Domain** | Computer is joined to a domain |
| **Private** | Connected to a trusted private network |
| **Public** | Connected to a public/untrusted network |

### Firewall Commands

```cmd
# Check firewall status
netsh advfirewall show allprofiles

# Enable/disable firewall
netsh advfirewall set allprofiles state on
netsh advfirewall set allprofiles state off

# Add an inbound rule
netsh advfirewall firewall add rule name="Allow RDP" protocol=TCP dir=in localport=3389 action=allow

# Delete a rule
netsh advfirewall firewall delete rule name="Allow RDP"

# PowerShell - list firewall rules
Get-NetFirewallRule | Where-Object Enabled -eq True

# PowerShell - add rule
New-NetFirewallRule -DisplayName "Block Port 4444" -Direction Inbound -LocalPort 4444 -Protocol TCP -Action Block
```

---

## 10. Active Directory

### What is Active Directory?

Active Directory (AD) is Microsoft's directory service for centralized identity and access management in Windows domain environments.

### Core Components

| Component | Description |
|-----------|-------------|
| **Domain** | Security boundary; collection of objects sharing a policy |
| **Forest** | Top-level container; one or more domains |
| **Domain Controller (DC)** | Server hosting AD; handles authentication |
| **Organizational Unit (OU)** | Container for organizing objects |
| **LDAP** | Protocol used to query/modify AD |
| **Global Catalog** | Partial replica of all objects in a forest |

### AD Objects

- **Users** – Represent people or service accounts
- **Computers** – Domain-joined machines
- **Groups** – Collections of users/computers
  - Security groups (used for permissions)
  - Distribution groups (used for email)
- **GPO** – Group Policy Objects linked to OUs

### Privileged AD Groups

| Group | Privileges |
|-------|-----------|
| `Domain Admins` | Full control over the domain |
| `Enterprise Admins` | Full control over the forest |
| `Schema Admins` | Can modify the AD schema |
| `Administrators` | Local admin on DCs |
| `Account Operators` | Manage user accounts |
| `Backup Operators` | Backup/restore privileges (can access all files) |
| `Server Operators` | Manage DCs |
| `DNSAdmins` | Manage DNS (can be abused for privesc) |

### AD Enumeration Commands

```cmd
# Domain info
net user /domain
net group /domain
net group "Domain Admins" /domain

# PowerShell AD module (requires RSAT or domain controller)
Get-ADUser -Filter * -Properties *
Get-ADGroup -Filter * | Select-Object Name
Get-ADGroupMember "Domain Admins"
Get-ADComputer -Filter * | Select-Object Name, OperatingSystem

# LDAP query via PowerShell
([ADSI]"LDAP://DC=example,DC=com")
```

---

## 11. Group Policy

### What is Group Policy?

Group Policy allows administrators to enforce security settings and configurations across all machines in a domain.

### Policy Application Order (LSDOU)

```
Local → Site → Domain → Organizational Unit
```

Later policies override earlier ones (OU beats Domain).

### Key Security Policies

```
Computer Configuration\Windows Settings\Security Settings\
  ├── Account Policies\Password Policy
  │     ├── Minimum password length
  │     ├── Password complexity
  │     └── Maximum password age
  ├── Account Policies\Account Lockout Policy
  │     ├── Lockout threshold
  │     └── Lockout duration
  ├── Local Policies\Audit Policy
  │     ├── Audit logon events
  │     └── Audit object access
  └── Local Policies\User Rights Assignment
        ├── Allow log on locally
        └── Deny access to this computer from the network
```

### Group Policy Commands

```cmd
# Force group policy update
gpupdate /force

# View applied policies
gpresult /r

# Verbose GPO report (HTML)
gpresult /h C:\gpo-report.html

# View local security policy
secpol.msc
```

---

## 12. Windows Security Features

### User Account Control (UAC)

UAC prevents unauthorized system changes by requiring elevation for administrative tasks. Standard users see a credential prompt; admin users see a consent prompt.

**Bypass techniques (for awareness):**
- `fodhelper.exe` auto-elevation
- `eventvwr.exe` COM hijack
- Token impersonation

**Defense:** Set UAC to "Always Notify"; disable auto-elevation for built-in accounts.

### Windows Defender Antivirus

- Real-time protection, cloud-based analysis, behavior monitoring
- Managed via `Windows Security` GUI or PowerShell (`Defender` module)

```powershell
# Run a quick scan
Start-MpScan -ScanType QuickScan

# Check status
Get-MpComputerStatus

# Update signatures
Update-MpSignature
```

### Windows Defender SmartScreen

Filters known-malicious executables and websites; checks against Microsoft's cloud reputation service.

### AppLocker

Allows administrators to whitelist which applications, scripts, and DLLs can run.

```
Application Identity service must be running for AppLocker to enforce rules.
```

Policy types:
- **Executable rules** – `.exe`, `.com`
- **Script rules** – `.ps1`, `.bat`, `.vbs`, `.js`
- **DLL rules** – `.dll`, `.ocx`
- **Packaged app rules** – Store apps

### Windows Defender Credential Guard

Isolates LSASS into a **Virtual Trust Level (VTL1)** using virtualization-based security (VBS), preventing tools like Mimikatz from dumping credentials from LSASS memory.

**Requirements:** UEFI, 64-bit, Hyper-V, TPM 2.0

### Windows Defender Exploit Guard

Successor to EMET; provides:
- **Attack Surface Reduction (ASR)** rules — block macros, Office spawning processes, etc.
- **Controlled Folder Access** — ransomware protection
- **Network protection** — block connections to known-bad IPs
- **Exploit protection** — DEP, ASLR, SEHOP mitigations

### BitLocker

Full-disk encryption for Windows volumes.
- Protects data at rest if drive is removed
- Uses TPM to seal encryption key
- Recovery key stored in AD or Azure AD

```cmd
# Check BitLocker status
manage-bde -status

# Enable BitLocker
Enable-BitLocker -MountPoint "C:" -EncryptionMethod Aes256 -UsedSpaceOnly -TpmProtector
```

### Windows Sandbox

Isolated, temporary desktop environment for running untrusted applications; discarded after closing.

---

## 13. Logging & Event IDs

### Log Locations

| Log | Path | Contents |
|-----|------|---------|
| Security | `%SystemRoot%\System32\winevt\Logs\Security.evtx` | Auth, account, policy changes |
| System | `...Logs\System.evtx` | OS events, driver failures |
| Application | `...Logs\Application.evtx` | App errors and info |
| PowerShell | `...Logs\Microsoft-Windows-PowerShell/Operational.evtx` | PS execution |
| Sysmon | `...Logs\Microsoft-Windows-Sysmon/Operational.evtx` | Process, network, registry events |

### Critical Security Event IDs

**Authentication:**

| Event ID | Description |
|----------|-------------|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4634 | Account logoff |
| 4648 | Logon using explicit credentials (runas) |
| 4672 | Special privileges assigned to new logon |
| 4768 | Kerberos TGT requested |
| 4769 | Kerberos service ticket requested |
| 4771 | Kerberos pre-authentication failed |
| 4776 | NTLM authentication attempt |

**Account Management:**

| Event ID | Description |
|----------|-------------|
| 4720 | User account created |
| 4722 | User account enabled |
| 4724 | Password reset |
| 4728 | Member added to security-enabled global group |
| 4732 | Member added to local Administrators group |
| 4756 | Member added to universal security group |

**Policy & System:**

| Event ID | Description |
|----------|-------------|
| 4698 | Scheduled task created |
| 4702 | Scheduled task updated |
| 4697 | Service installed |
| 4704 | User right assigned |
| 7045 | New service installed (System log) |

**Object Access:**

| Event ID | Description |
|----------|-------------|
| 4663 | Object access attempt (requires SACL) |
| 4656 | Handle to object requested |
| 5140 | Network share accessed |
| 5145 | Network share file access check |

**Logon Types (in event 4624):**

| Type | Description |
|------|-------------|
| 2 | Interactive (local logon) |
| 3 | Network (SMB, etc.) |
| 4 | Batch (scheduled task) |
| 5 | Service logon |
| 7 | Unlock |
| 8 | Network cleartext (IIS basic auth) |
| 9 | New credentials (runas) |
| 10 | Remote interactive (RDP) |
| 11 | Cached interactive (cached credentials) |

### Event Log Commands

```powershell
# Query last 20 failed logon events
Get-WinEvent -LogName Security | Where-Object { $_.Id -eq 4625 } | Select-Object -First 20

# Query by time range
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624; StartTime=(Get-Date).AddHours(-24)}

# Export events to CSV
Get-WinEvent -LogName Security | Export-Csv -Path C:\security-events.csv

# cmd - query event log
wevtutil qe Security /q:"*[System[(EventID=4625)]]" /c:10 /f:text
```

### Sysmon

Sysmon (System Monitor) from Sysinternals provides rich telemetry beyond native Windows logging.

Key Sysmon event IDs:

| Event ID | Description |
|----------|-------------|
| 1 | Process creation (includes command line) |
| 2 | File creation time changed |
| 3 | Network connection |
| 7 | Image (DLL) loaded |
| 8 | Remote thread creation |
| 10 | Process access (handle to another process) |
| 11 | File created |
| 12/13/14 | Registry create/set/rename |
| 15 | File stream created (ADS) |
| 22 | DNS query |

---

## 14. PowerShell for Security

### PowerShell Security Features

| Feature | Description |
|---------|-------------|
| **Execution Policy** | Controls which scripts can run (not a security boundary) |
| **Constrained Language Mode** | Restricts PS capabilities; used with AppLocker/WDAC |
| **Script Block Logging** | Logs all script blocks to event log (ID 4104) |
| **Module Logging** | Logs module usage (ID 4103) |
| **AMSI** | Antimalware Scan Interface; scans scripts before execution |
| **Transcription** | Records all console input/output to a file |

### Execution Policy

```powershell
# Check current policy
Get-ExecutionPolicy -List

# Set policy (per scope)
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser

# Bypass execution policy (attacker technique; for awareness)
powershell.exe -ExecutionPolicy Bypass -File script.ps1
```

### Enable Script Block Logging via Registry

```
HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
  EnableScriptBlockLogging = 1
  EnableScriptBlockInvocationLogging = 1
```

### Useful Security One-Liners

```powershell
# List all local admin group members
Get-LocalGroupMember -Group "Administrators"

# Find scheduled tasks
Get-ScheduledTask | Where-Object State -ne Disabled

# List running services with their executable
Get-WmiObject Win32_Service | Select-Object Name, State, PathName

# Check for unsigned binaries in System32
Get-ChildItem C:\Windows\System32\*.exe | Get-AuthenticodeSignature | Where-Object Status -ne Valid

# List all open TCP connections with process
Get-NetTCPConnection -State Established | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, OwningProcess

# Find files modified in the last 24 hours
Get-ChildItem -Path C:\ -Recurse -File -ErrorAction SilentlyContinue | Where-Object { $_.LastWriteTime -gt (Get-Date).AddDays(-1) }

# Dump all autoruns (PowerShell equivalent)
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"

# Check AppLocker policies
Get-AppLockerPolicy -Effective | Select-Object -ExpandProperty RuleCollections
```

---

## 15. Command Line Tools

### Built-in Security Tools

| Tool | Purpose |
|------|---------|
| `whoami` | Current user, groups, and privileges |
| `net` | User, group, share, session management |
| `icacls` | File/directory permissions |
| `cacls` | Legacy permissions tool |
| `runas` | Run program as another user |
| `sc` | Service management |
| `reg` | Registry management |
| `sfc` | System File Checker |
| `cipher` | EFS and file wiping |
| `auditpol` | Audit policy configuration |
| `secedit` | Security template analysis |
| `wmic` | WMI command-line interface |
| `certutil` | Certificate management (abused for downloads) |
| `bitsadmin` | Background transfer service (abused for downloads) |

### `whoami` Deep Dive

```cmd
# Current user
whoami

# User SID
whoami /user

# All group memberships
whoami /groups

# Current privileges
whoami /priv

# All info
whoami /all
```

### WMIC Examples

```cmd
# Running processes
wmic process list brief

# Process with command line
wmic process get name,processid,commandline

# Installed software
wmic product get name,version

# Startup items
wmic startup list full

# User accounts
wmic useraccount list full
```

### Living Off the Land (LOLBins)

Tools built into Windows that are commonly abused by attackers:

| Binary | Abuse Technique |
|--------|----------------|
| `certutil.exe` | Download files, decode base64 |
| `bitsadmin.exe` | Download files via BITS |
| `mshta.exe` | Execute HTA files (HTML applications) |
| `regsvr32.exe` | Execute DLLs / remote scriptlets (Squiblydoo) |
| `rundll32.exe` | Execute DLLs |
| `wscript.exe` / `cscript.exe` | Execute VBScript / JScript |
| `msiexec.exe` | Install MSI packages from URLs |
| `installutil.exe` | Execute .NET assemblies |
| `wmic.exe` | Remote code execution via WMI |
| `powershell.exe` | Broad scriptable capabilities |

---

## 16. Scheduled Tasks & Startup Locations

### Scheduled Tasks

Scheduled tasks are a common persistence and lateral movement mechanism.

```cmd
# List all scheduled tasks
schtasks /query /fo LIST /v

# Create a scheduled task
schtasks /create /tn "MyTask" /tr "C:\Windows\System32\cmd.exe" /sc daily /st 08:00

# Delete a task
schtasks /delete /tn "MyTask" /f

# PowerShell
Get-ScheduledTask
Get-ScheduledTask -TaskName "MyTask" | Select-Object *
```

Task XML files stored at: `C:\Windows\System32\Tasks\`

### Startup Locations (Persistence Checklist)

```
Registry:
  HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
  HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
  HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
  HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
  HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon (Userinit, Shell)
  HKLM\SYSTEM\CurrentControlSet\Services\

Filesystem:
  C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\
  C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\
  C:\Windows\System32\drivers\

Scheduled Tasks:
  C:\Windows\System32\Tasks\

Services:
  Services registered in SCM

WMI:
  WMI Event Subscriptions (permanent; survive reboots)
```

---

## 17. Common Attack Techniques

### Pass-the-Hash (PtH)

Using a captured NTLM hash to authenticate without knowing the plaintext password.

```
Attacker obtains NTLM hash → passes hash in NTLM authentication → gains access as that user
```

Tools: Mimikatz, Impacket (`psexec.py`), CrackMapExec

**Mitigation:** Disable NTLM where possible; use Credential Guard; enable Protected Users security group.

### Pass-the-Ticket (PtT)

Stealing a Kerberos TGT or service ticket from memory and using it to authenticate.

```
Steal .kirbi ticket from LSASS → inject into current session → authenticate as ticket owner
```

**Mitigation:** Credential Guard; monitor for 4768/4769 anomalies; short ticket lifetimes.

### Overpass-the-Hash

Use an NTLM hash to request a Kerberos TGT, combining PtH and PtT.

### Kerberoasting

Request service tickets for accounts with SPNs (Service Principal Names), then crack the ticket offline.

```powershell
# Find Kerberoastable accounts (PowerView)
Get-DomainUser -SPN

# Request service tickets
Invoke-Kerberoast -OutputFormat Hashcat
```

**Mitigation:** Use strong, long, randomly-generated passwords for service accounts; use Managed Service Accounts (MSAs).

### ASREPRoasting

Targets accounts with Kerberos pre-authentication disabled. AS-REP response contains encrypted data crackable offline.

**Mitigation:** Ensure all accounts require pre-authentication (the default).

### Golden Ticket Attack

Forge a TGT using the **KRBTGT account's NTLM hash**, granting persistent, near-unlimited access to the domain.

**Mitigation:** Rotate KRBTGT password twice after a compromise; monitor for anomalous TGTs with very long lifetimes.

### Silver Ticket Attack

Forge a Kerberos service ticket using a service account's hash. Doesn't touch the DC; harder to detect.

**Mitigation:** Protect service account credentials; enable PAC validation.

### DCSync Attack

Impersonate a Domain Controller to replicate credential data from AD using DRSUAPI (Directory Replication Service).

```
Requires: DS-Replication-Get-Changes and DS-Replication-Get-Changes-All rights
```

**Mitigation:** Monitor event ID 4662 for unusual replication activity; restrict who holds replication rights.

### Credential Dumping with Mimikatz

```
# Dump credentials from LSASS (requires SYSTEM or SeDebugPrivilege)
privilege::debug
sekurlsa::logonpasswords

# Dump SAM hashes
lsadump::sam

# DCSync
lsadump::dcsync /user:Administrator
```

**Mitigation:** Credential Guard; Protected Users group; EDR with LSASS protection.

### LLMNR / NBT-NS Poisoning

When DNS fails, Windows falls back to LLMNR and NBT-NS for name resolution. Attackers respond to these broadcasts to capture NTLMv2 hashes.

```
Victim → "Who is FileServer?" (LLMNR broadcast)
Attacker → "I am FileServer!" → victim sends NTLMv2 hash
```

Tools: Responder

**Mitigation:** Disable LLMNR and NBT-NS via Group Policy.

### Bloodhound / AD Attack Path Analysis

BloodHound visualizes attack paths through Active Directory by mapping relationships between objects.

```powershell
# Collect data (SharpHound)
.\SharpHound.exe -c All
```

**Mitigation:** Audit group memberships and ACL delegations; remove unnecessary admin rights.

---

## 18. Lateral Movement

### Techniques

| Technique | Protocol | Tool |
|-----------|----------|------|
| **PsExec** | SMB + RPC | Sysinternals PsExec, Impacket |
| **WMI** | RPC / DCOM | `wmic /node:`, Impacket |
| **WinRM** | HTTP 5985/5986 | `Enter-PSSession`, Evil-WinRM |
| **RDP** | TCP 3389 | `mstsc.exe` |
| **SMB** | TCP 445 | Pass-the-Hash via SMB |
| **DCOM** | RPC | MMC20.Application, ShellBrowserWindow |

### WinRM Remote Session

```powershell
# Create a remote PS session
Enter-PSSession -ComputerName target -Credential (Get-Credential)

# Run a command remotely
Invoke-Command -ComputerName target -ScriptBlock { whoami } -Credential $cred

# Enable WinRM on target (requires admin)
Enable-PSRemoting -Force
```

### Detection

- Event 4624 (logon type 3) for network logons
- Event 4648 for explicit credential use
- Event 7045 for new service installations (PsExec)
- Sysmon event 3 (network connections)

---

## 19. Privilege Escalation

### Common Windows PrivEsc Vectors

**Unquoted Service Paths:**

If a service binary path has spaces and is unquoted, Windows searches each path component:
```
C:\Program Files\My App\service.exe
→ Windows tries: C:\Program.exe, C:\Program Files\My.exe, ...
```

```cmd
# Find unquoted service paths
wmic service get name,pathname | findstr /i /v "C:\Windows\\" | findstr /i /v """
```

**Weak Service Permissions:**

```cmd
# Check service permissions (Sysinternals accesschk)
accesschk.exe -uwcqv "Authenticated Users" *
accesschk.exe -uwcqv "Everyone" *
```

**Weak Registry Permissions:**

```powershell
# Check registry key permissions on services
Get-Acl "HKLM:\SYSTEM\CurrentControlSet\Services\VulnService"
```

**DLL Hijacking:**

Windows searches for DLLs in this order (SafeDllSearchMode enabled):
1. Application directory
2. System directory (`System32`)
3. 16-bit system directory
4. Windows directory
5. Current directory
6. Directories in `PATH`

If a writable directory appears before the legitimate DLL location, an attacker can place a malicious DLL there.

**AlwaysInstallElevated:**

If both of these registry keys are set to 1, MSI packages install with SYSTEM privileges:
```
HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated
HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated
```

**Token Impersonation:**

Accounts with `SeImpersonatePrivilege` (common for service accounts) can impersonate other tokens. Tools: Juicy Potato, Rogue Potato, PrintSpoofer.

**Stored Credentials:**

```cmd
# List stored credentials
cmdkey /list

# Use stored credentials
runas /savecred /user:administrator cmd.exe
```

---

## 20. Persistence Mechanisms

### Registry Autoruns

```powershell
# Add malicious autorun (attacker technique; for awareness)
reg add "HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" /v "Updater" /t REG_SZ /d "C:\malware.exe"
```

### Scheduled Tasks

```cmd
schtasks /create /tn "WindowsUpdate" /tr "C:\backdoor.exe" /sc onlogon /ru SYSTEM
```

### Services

```cmd
sc create backdoor binpath= "C:\malware.exe" start= auto
sc start backdoor
```

### WMI Event Subscriptions

Persistent WMI subscriptions survive reboots and are invisible in normal tools:

```powershell
# Create WMI persistence (attacker technique; for awareness)
$FilterArgs = @{Name='PersistFilter'; EventNameSpace='root\CimV2'; QueryLanguage="WQL"; Query="SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'"}
$Filter = New-CimInstance -Namespace root/subscription -ClassName __EventFilter -Property $FilterArgs
```

Detection: Check `HKLM\SOFTWARE\Microsoft\WBEM\ESS\` and query WMI subscriptions:
```powershell
Get-WMIObject -Namespace root\subscription -Class __EventFilter
Get-WMIObject -Namespace root\subscription -Class __EventConsumer
```

### DLL Hijacking for Persistence

Place malicious DLL in a writable directory that a legitimate, auto-starting application loads.

---

## 21. Defense & Hardening

### CIS Benchmarks for Windows

The **CIS (Center for Internet Security) Benchmarks** provide prescriptive hardening guidance. Key recommendations:

- Disable unnecessary services (e.g. Telnet, FTP server, Remote Registry)
- Enable audit policies for logon, account management, object access, policy changes
- Configure password policy: min 14 chars, complexity, 90-day max age
- Set account lockout: 5 attempts, 30-minute lockout
- Disable SMBv1
- Enable Windows Defender and keep signatures updated
- Enable BitLocker on all drives
- Restrict local administrator accounts
- Enable Credential Guard and Device Guard where possible

### Disable Unnecessary Features

```powershell
# Disable SMBv1
Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force

# Disable LLMNR (GPO preferred, or registry)
# HKLM\SOFTWARE\Policies\Microsoft\Windows NT\DNSClient\EnableMulticast = 0

# Disable NBT-NS (per-adapter via WMI)
$adapters = Get-WmiObject Win32_NetworkAdapterConfiguration
$adapters | Where-Object { $_.TcpipNetbiosOptions -ne 2 } | ForEach-Object { $_.SetTcpipNetbios(2) }

# Disable PowerShell v2 (older version bypasses AMSI)
Disable-WindowsOptionalFeature -Online -FeatureName MicrosoftWindowsPowerShellV2Root
```

### Protected Users Security Group

Members of the **Protected Users** group receive automatic protections:
- Cannot use NTLM, Digest, or CredSSP
- Kerberos tickets use AES only; no RC4
- No credential caching
- TGT lifetime limited to 4 hours

Add privileged accounts (Domain Admins) to this group.

### Attack Surface Reduction (ASR) Rules

Key ASR rules to enable:

| Rule | Description |
|------|-------------|
| Block Office macros from spawning child processes | Stops macro-based malware |
| Block execution of potentially obfuscated scripts | AMSI-level detection |
| Block credential stealing from LSASS | LSASS dump protection |
| Block untrusted and unsigned processes from USB | USB malware |
| Block persistence through WMI event subscriptions | WMI persistence |

```powershell
# Enable an ASR rule (requires Defender)
Add-MpPreference -AttackSurfaceReductionRules_Ids <RULE_GUID> -AttackSurfaceReductionRules_Actions Enabled
```

### Logging Recommendations

Enable the following via Group Policy or `auditpol`:

```cmd
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Account Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Account Management" /success:enable /failure:enable
auditpol /set /subcategory:"Process Creation" /success:enable
auditpol /set /subcategory:"Policy Change" /success:enable /failure:enable
auditpol /set /subcategory:"Privilege Use" /success:enable /failure:enable
```

Also deploy **Sysmon** with a community config (e.g. SwiftOnSecurity or olafhartong's config).

---

## 22. Forensics & Incident Response

### Volatile Data (Collect First)

When responding to an incident, collect volatile (in-memory) data before anything else:

```powershell
# Running processes
Get-Process | Select-Object Name, Id, Path, StartTime | Export-Csv processes.csv

# Network connections
Get-NetTCPConnection | Export-Csv connections.csv

# Logged-in users
query user

# Open files / handles
handle.exe (Sysinternals)

# Clipboard
Get-Clipboard

# Environment variables
Get-ChildItem Env:
```

### Key Forensic Artifacts

| Artifact | Location | What It Shows |
|----------|----------|--------------|
| **Prefetch** | `C:\Windows\Prefetch\` | Evidence of program execution (last 8 run times) |
| **ShimCache** | Registry `HKLM\SYSTEM\...\AppCompatCache` | Programs that ran (even deleted ones) |
| **AmCache** | `C:\Windows\appcompat\Programs\Amcache.hve` | Executable metadata, first run time |
| **Recent files** | `%AppData%\Microsoft\Windows\Recent\` | Recently opened files (LNK files) |
| **Jump Lists** | `%AppData%\Microsoft\Windows\Recent\AutomaticDestinations\` | Recently opened files per application |
| **Browser history** | User profile → browser directories | Websites visited |
| **MFT** | `C:\$MFT` | NTFS file system metadata (requires raw access) |
| **VSS / Shadow Copies** | `\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopyN\` | Previous file versions |
| **Event logs** | `C:\Windows\System32\winevt\Logs\` | Authentication, process, policy events |
| **$Recycle.Bin** | `C:\$Recycle.Bin\` | Deleted files with original paths |
| **SAM / SYSTEM / SECURITY** | `C:\Windows\System32\config\` | User hashes, LSA secrets |
| **NTDS.dit** | DC only — `C:\Windows\NTDS\` | All AD credentials |

### Memory Analysis

Memory forensics can recover running processes, network connections, injected code, and credentials.

Tools: **Volatility 3**

```bash
# Image info
python vol.py -f memory.dmp windows.info

# Running processes
python vol.py -f memory.dmp windows.pslist
python vol.py -f memory.dmp windows.pstree

# Network connections
python vol.py -f memory.dmp windows.netstat

# Injected code / hollowed processes
python vol.py -f memory.dmp windows.malfind

# LSASS credential extraction
python vol.py -f memory.dmp windows.lsadump
```

### Quick Triage Commands

```powershell
# All autorun locations
Get-CimInstance Win32_StartupCommand | Select-Object Name, Command, Location, User

# Recently modified files in sensitive paths
Get-ChildItem C:\Windows\System32 -File | Where-Object { $_.LastWriteTime -gt (Get-Date).AddDays(-7) } | Sort-Object LastWriteTime -Descending

# Suspicious scheduled tasks
Get-ScheduledTask | Where-Object { $_.TaskPath -notlike "\Microsoft\*" } | Select-Object TaskName, TaskPath

# Unsigned running processes
Get-Process | ForEach-Object {
  try {
    $sig = Get-AuthenticodeSignature $_.Path -ErrorAction Stop
    if ($sig.Status -ne 'Valid') { $_ | Select-Object Name, Id, Path }
  } catch {}
}

# WMI subscriptions
Get-WMIObject -Namespace root\subscription -Class __EventFilter | Select-Object Name, Query
Get-WMIObject -Namespace root\subscription -Class __EventConsumer
```

---

## Quick Reference: Key Windows Security Concepts

| Concept | Summary |
|---------|---------|
| **SAM** | Local user credential store (NTLM hashes) |
| **LSA** | Local Security Authority; handles authentication |
| **LSASS** | LSA Subsystem Service; prime target for credential dumping |
| **NTLM** | Legacy auth protocol; vulnerable to PtH and relay attacks |
| **Kerberos** | Ticket-based auth; default for AD; vulnerable to Kerberoasting, Golden/Silver tickets |
| **SID** | Security Identifier; uniquely identifies every security principal |
| **ACL/ACE** | Access Control List / Entry; defines permissions on objects |
| **UAC** | User Account Control; prevents unauthorized elevation |
| **Integrity Level** | Mandatory access control mechanism (Low/Medium/High/System) |
| **Credential Guard** | VBS-based LSASS isolation; prevents credential dumping |
| **AMSI** | Antimalware Scan Interface; scans scripts at runtime |
| **ASR** | Attack Surface Reduction; Defender rules blocking common attack patterns |
| **AppLocker** | Application whitelisting by publisher, path, or hash |
| **WMI** | Windows Management Instrumentation; powerful admin API; abused for lateral movement and persistence |
| **LOLBins** | Living Off the Land Binaries; legitimate tools abused by attackers |

---

## Further Reading

- [Microsoft Security Documentation](https://docs.microsoft.com/en-us/security/)
- [MITRE ATT&CK for Windows](https://attack.mitre.org/matrices/enterprise/windows/)
- [CIS Windows Benchmarks](https://www.cisecurity.org/benchmark/microsoft_windows_desktop)
- [LOLBAS Project](https://lolbas-project.github.io/)
- [PayloadsAllTheThings – Windows PrivEsc](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)
- [BloodHound Documentation](https://bloodhound.readthedocs.io/)
- [Sysinternals Suite](https://docs.microsoft.com/en-us/sysinternals/)