# Operating System Security

A comprehensive reference covering operating system security concepts, mechanisms, attack surfaces, and defensive techniques across Windows and Linux — from foundational theory to practical hardening and incident response.

---

## 1. Operating System Security Fundamentals

### What is OS Security?

Operating system security encompasses the policies, controls, and mechanisms that protect an OS from unauthorized access, misuse, modification, and denial of service. The OS is the foundation of all software — a compromised OS means everything running on it is compromised.

### Core Security Objectives (CIA Triad + AAA)

| Objective | Description |
|-----------|-------------|
| **Confidentiality** | Prevent unauthorized disclosure of information |
| **Integrity** | Prevent unauthorized modification of data or code |
| **Availability** | Ensure the system is accessible when needed |
| **Authentication** | Verify the identity of users and processes |
| **Authorization** | Control what authenticated entities can do |
| **Accountability** | Track actions to a responsible entity (auditing) |

### Security Design Principles

| Principle | Description | Example |
|-----------|-------------|---------|
| **Least Privilege** | Grant only minimum necessary rights | Services run as dedicated low-privilege accounts |
| **Defense in Depth** | Layer multiple independent controls | Firewall + AV + HIPS + logging |
| **Fail Secure** | Deny access on failure, not grant | Default-deny firewall policy |
| **Economy of Mechanism** | Keep security mechanisms simple | Simple ACL model vs complex custom logic |
| **Complete Mediation** | Check every access, every time | OS verifies permissions on every file open |
| **Open Design** | Security through logic, not obscurity | Published crypto algorithms |
| **Separation of Privilege** | Require multiple conditions for access | Two-factor authentication |
| **Least Common Mechanism** | Minimize shared resources | Process isolation, separate address spaces |
| **Psychological Acceptability** | Security must be usable | UAC prompts vs silently blocking everything |

---

## 2. OS Architecture & Security Boundaries

### Privilege Rings

Modern CPUs implement hardware-enforced privilege levels:

```
        Ring 0  ──  Kernel (OS core, drivers)
       Ring 1   ──  Rarely used (some hypervisors)
      Ring 2    ──  Rarely used
     Ring 3     ──  User applications
```

- **Ring 0 (Kernel Mode)** – Unrestricted hardware access; OS kernel, device drivers
- **Ring 3 (User Mode)** – Restricted; cannot directly access hardware or kernel memory

A **syscall** (system call) is the controlled mechanism for user-mode code to request kernel services. This transition point is heavily audited and is a common attack target.

### Kernel vs. User Space

```
┌────────────────────────────────────────────────────────┐
│                     User Space                          │
│   Process A    │    Process B    │    Process C         │
│ (isolated VA)  │  (isolated VA)  │  (isolated VA)       │
├────────────────────────────────────────────────────────┤
│              System Call Interface                       │
├────────────────────────────────────────────────────────┤
│                    Kernel Space                          │
│  Process Mgmt │ Memory Mgmt │ File Systems │ Networking  │
│  Security Ref │ Device Mgrs │ IPC          │ Scheduler   │
├────────────────────────────────────────────────────────┤
│              Hardware Abstraction Layer                  │
├────────────────────────────────────────────────────────┤
│                      Hardware                            │
└────────────────────────────────────────────────────────┘
```

**Security implications:**
- A vulnerability in kernel-mode code can compromise the entire system
- User-mode exploits are contained to that process (unless they escalate)
- Kernel exploits bypass all OS-level security controls

### Virtual Address Spaces

Each process runs in its own isolated virtual address space. The OS memory manager translates virtual addresses to physical addresses — a process cannot read or write another process's memory without explicit OS permission (e.g. `ptrace`, `ReadProcessMemory`).

**Attack relevance:** Process injection techniques (DLL injection, shellcode injection) abuse legitimate OS mechanisms to write into another process's address space.

---

## 3. Authentication

### Authentication Factors

| Factor | Type | Examples |
|--------|------|---------|
| Something you **know** | Knowledge | Password, PIN, security question |
| Something you **have** | Possession | Smart card, TOTP token, hardware key |
| Something you **are** | Inherence | Fingerprint, face, iris, voice |
| Somewhere you **are** | Location | IP range, GPS geofence |

**Multi-Factor Authentication (MFA)** requires two or more different factor types. Two passwords = not MFA (same factor type).

### Password Storage

Passwords must never be stored in plaintext. The OS stores a one-way hash:

| Algorithm | Used In | Security |
|-----------|---------|---------|
| LM hash | Legacy Windows (pre-Vista) | Broken — split into 7-char chunks, all uppercase |
| NTLM (MD4) | Windows SAM/AD | Weak — no salting, fast to crack |
| MD5 (`$1$`) | Legacy Linux | Weak — fast, brute-forceable |
| SHA-256 (`$5$`) | Linux | Better |
| SHA-512 (`$6$`) | Modern Linux | Good |
| bcrypt | Applications | Strong — intentionally slow |
| yescrypt (`$y$`) | Modern Linux | Strong — memory-hard |
| Argon2 | Modern applications | Best — memory and time hard |

**Salting** adds a unique random value to each password before hashing, preventing rainbow table attacks and ensuring identical passwords produce different hashes.

### Windows Authentication

**Local Authentication (SAM):**
```
User enters credentials
        │
        ▼
Winlogon → LSA → Authentication Package (MSV1_0)
        │
        ▼
Compare NTLM hash against SAM database
        │
        ▼
Success → Create access token with user SID + groups + privileges
```

**Domain Authentication (Kerberos):**
```
Client          KDC (AS)           KDC (TGS)          Service
  │── AS-REQ ──▶│                     │                  │
  │◀── AS-REP ──│ (TGT issued)        │                  │
  │─────────────────── TGS-REQ ──────▶│                  │
  │◀──────────────────── TGS-REP ─────│ (service ticket) │
  │──────────────────────────────────────── AP-REQ ──────▶│
  │◀─────────────────────────────────────── AP-REP ───────│
```

**Linux Authentication (PAM):**

PAM (Pluggable Authentication Modules) provides a flexible authentication framework. Authentication stacks are defined per service in `/etc/pam.d/`:

```
auth    required    pam_unix.so         # check /etc/shadow
auth    required    pam_faillock.so     # account lockout
account required    pam_unix.so         # account validity
password required   pam_pwquality.so    # password complexity
session required    pam_unix.so         # session setup
```

### Single Sign-On (SSO)

| Protocol | Description | Common Use |
|----------|-------------|-----------|
| **Kerberos** | Ticket-based; domain SSO | Active Directory |
| **SAML** | XML-based; enterprise web SSO | Corporate apps |
| **OAuth 2.0** | Authorization delegation | API access |
| **OpenID Connect** | Identity layer on OAuth 2.0 | Web login ("Login with Google") |
| **LDAP** | Directory-based auth | Internal apps |

---

## 4. Authorization & Access Control

### Access Control Models

**Discretionary Access Control (DAC)**
- Object owners control who can access their resources
- Used by default in Windows and Linux (file permissions, ACLs)
- Weakness: owners can inadvertently grant excessive access; malware runs with owner's full rights

**Mandatory Access Control (MAC)**
- OS enforces access based on security labels/policy; users cannot override
- Examples: SELinux, AppArmor, Windows Integrity Levels
- Strength: limits damage from compromised processes

**Role-Based Access Control (RBAC)**
- Access granted based on job role, not individual identity
- Examples: Active Directory groups, Linux sudoers groups, AWS IAM roles
- Simplifies large-scale permission management

**Attribute-Based Access Control (ABAC)**
- Access decisions based on attributes of user, resource, and environment
- Most flexible; used in cloud IAM policies
- Example: "Allow access if user.department = Finance AND resource.classification = Internal AND time = BusinessHours"

**Rule-Based Access Control**
- Fixed rules applied to all users
- Example: firewall rules, time-based access restrictions

### Access Control Lists (ACLs)

An ACL is a list of Access Control Entries (ACEs) attached to an object. Each ACE specifies a subject and their allowed/denied permissions.

**Windows ACL structure:**
```
Object (e.g. file)
  └── Security Descriptor
        ├── Owner SID
        ├── Group SID
        ├── DACL (Discretionary ACL) ─── who can access
        │     ├── ACE: Allow SYSTEM Full Control
        │     ├── ACE: Allow Administrators Full Control
        │     └── ACE: Allow Alice Read & Execute
        └── SACL (System ACL) ─────────── what to audit
              └── ACE: Audit Everyone Write (success+fail)
```

**Linux permission model** is a simplified ACL:
```
Owner  Group  Others
 rwx    r-x    r--
```

Extended ACLs (`setfacl`/`getfacl`) provide per-user and per-group granularity beyond the basic model.

### Principle of Least Privilege in Practice

```
❌ Bad:  Web server running as root/SYSTEM
✅ Good: Web server running as www-data (no shell, no home dir)

❌ Bad:  All admins share a single "admin" account
✅ Good: Named accounts; privilege granted only when needed (sudo/runas)

❌ Bad:  Service account in Domain Admins
✅ Good: Service account with only the rights it needs

❌ Bad:  All users are local administrators
✅ Good: Standard user accounts; separate admin account for admin tasks
```

---

## 5. Process Security

### Process Isolation

Each process runs in its own virtual address space. The OS kernel enforces boundaries — a process cannot access another process's memory or files beyond its granted permissions.

**Mechanisms:**
- Virtual memory (hardware MMU + OS page tables)
- File system permissions checked on every access
- Security tokens attached to each process/thread

### Security Tokens (Windows)

Every Windows process carries a security token containing:
- User SID
- Group SIDs (including special groups like Everyone, Authenticated Users)
- Privileges (e.g. `SeDebugPrivilege`, `SeImpersonatePrivilege`)
- Integrity level
- Session ID

**Token types:**
- **Primary token** – Assigned to a process at creation
- **Impersonation token** – Allows a thread to act as a different security context (server impersonating a client)

### Integrity Levels (Windows MIC)

Mandatory Integrity Control assigns an integrity level to every process and object:

```
Untrusted (0)  → Anonymous processes
Low       (1)  → Sandboxed apps (browser tabs, email attachments)
Medium    (2)  → Normal user processes (default)
High      (3)  → Elevated processes (UAC-elevated)
System    (4)  → OS services (SYSTEM account)
```

A process cannot write to objects with a higher integrity level than its own — a Low integrity process cannot modify Medium integrity files, even if the DAC would allow it.

### Linux Process Credentials

Each Linux process has:
- **Real UID/GID** – Who actually owns the process
- **Effective UID/GID** – What permissions the process actually uses
- **Saved UID/GID** – Saved copy for privilege transitions
- **Supplementary groups** – Additional group memberships

**SUID mechanism:** When a SUID binary executes, the effective UID becomes the file owner's UID (often root), allowing temporary privilege elevation for specific tasks.

### Sandboxing

Sandboxes restrict what a process can do, even if it's compromised:

| Sandbox | Platform | Mechanism |
|---------|----------|-----------|
| **Seccomp** | Linux | Restrict which syscalls a process can make |
| **Namespaces** | Linux | Isolate process view of resources (PID, network, mount, user) |
| **cgroups** | Linux | Limit resource usage (CPU, memory, I/O) |
| **AppContainer** | Windows | UWP app isolation; Low integrity + capability model |
| **Job Objects** | Windows | Limit process group resources and capabilities |
| **Capsicum** | FreeBSD | Capability-based sandbox |
| **Pledge/Unveil** | OpenBSD | Restrict syscalls and filesystem access per process |

**Browser security** combines multiple sandbox layers: renderer processes run at Low/Untrusted integrity, with seccomp-BPF on Linux, to limit damage from web exploits.

---

## 6. Memory Security

### Common Memory Attacks

**Buffer Overflow**

Writing beyond the end of a fixed-size buffer overwrites adjacent memory — potentially including the return address on the stack, allowing an attacker to redirect execution.

```
Stack (before overflow):
[buffer: 16 bytes][saved EBP][return address]

Stack (after overflow):
[AAAA...AAAA...AA][AAAA....][attacker address]
                              ↑ Now points to attacker's shellcode
```

**Heap Overflow** – Overflow on the heap; can overwrite heap metadata or adjacent objects.

**Use-After-Free (UAF)** – Accessing memory after it has been freed; freed memory may be reallocated with attacker-controlled content.

**Format String** – Passing user input as a format string (`printf(user_input)`) allows reading/writing arbitrary memory via `%x`, `%n` specifiers.

**Integer Overflow** – Arithmetic overflow causing incorrect size calculations, leading to under-allocated buffers.

### Memory Protections

**ASLR (Address Space Layout Randomization)**

Randomizes base addresses of stack, heap, libraries, and executable at load time. An attacker cannot hardcode addresses in their exploit.

```
Without ASLR:  libc always at 0xb7e00000
With ASLR:     libc at 0xb7e00000 this run, 0xf7a12000 next run
```

- Windows: enabled by default; configurable with `/DYNAMICBASE` linker flag
- Linux: `kernel.randomize_va_space = 2` (full ASLR)
- Bypass: information leak vulnerabilities that reveal actual addresses

**DEP / NX (Data Execution Prevention / No-Execute)**

Marks memory regions as either executable or writable, but not both. Code on the stack or heap cannot be executed directly.

- Hardware: CPU NX/XD bit in page table entries
- Windows: DEP; configurable per-process or system-wide
- Linux: NX bit; enforced by kernel
- Bypass: Return-Oriented Programming (ROP) — reuses existing executable code gadgets

**Stack Canaries**

A random value (canary) placed between local variables and the return address. Checked before function return — if modified (by overflow), execution is terminated.

```
[local vars][CANARY][saved EBP][return address]
                ↑
         checked on return; abort if changed
```

- GCC: `-fstack-protector-strong`
- Windows: `/GS` compiler flag (Security Cookie)
- Bypass: information leak to read canary value; overwrite in a way that skips canary check

**CFI (Control Flow Integrity)**

Restricts indirect calls and jumps to a set of valid targets determined at compile time. Prevents exploitation techniques that hijack control flow to arbitrary locations.

- Windows: Control Flow Guard (CFG)
- Clang/LLVM: `-fsanitize=cfi`
- Intel: CET (Control-flow Enforcement Technology) — hardware shadow stack

**RELRO (Relocation Read-Only)** — Linux: makes the GOT (Global Offset Table) read-only after dynamic linking, preventing GOT overwrites.

**Safe Stack / Shadow Stack** — Keeps return addresses on a separate protected stack, preventing stack-based return address overwrites.

### Windows Memory Protections

| Protection | Description |
|-----------|-------------|
| DEP | Data Execution Prevention — NX enforcement |
| ASLR | Address space randomization |
| CFG | Control Flow Guard — validates indirect call targets |
| SEHOP | Structured Exception Handler Overwrite Protection |
| Heap Integrity | Detects heap metadata corruption |
| ACG | Arbitrary Code Guard — prevents dynamic code generation |
| CIG | Code Integrity Guard — requires signed code |

---

## 7. File System Security

### Permissions Model Comparison

| Feature | Linux (POSIX) | Windows (NTFS) |
|---------|--------------|----------------|
| Basic model | Owner/Group/Others + rwx | ACL with multiple ACEs |
| Granularity | 3 permission sets | Per-user/group ACEs |
| Inheritance | Manual or set-GID dir | Configurable inheritance flags |
| Special bits | SUID, SGID, Sticky | — |
| Extended attrs | `setfacl` / `getfacl` | Built into NTFS ACL |
| Encryption | EFS (via mount), dm-crypt | EFS, BitLocker |
| Auditing | auditd with file watches | SACL on files/folders |

### Sensitive File Locations

**Linux:**

| File | Sensitivity | Risk if Compromised |
|------|------------|---------------------|
| `/etc/shadow` | Critical | Password hash exposure → cracking |
| `/etc/passwd` | High | User enumeration |
| `/etc/sudoers` | Critical | Privilege escalation |
| `/etc/ssh/sshd_config` | High | SSH misconfiguration |
| `~/.ssh/id_rsa` | Critical | Direct SSH key theft |
| `~/.bash_history` | Medium | Command history → credential exposure |
| `/var/log/auth.log` | High | Authentication event visibility |
| `/proc/[pid]/mem` | Critical | Process memory access |

**Windows:**

| File / Location | Sensitivity | Risk if Compromised |
|----------------|------------|---------------------|
| `C:\Windows\System32\config\SAM` | Critical | Local credential hashes |
| `C:\Windows\System32\config\SYSTEM` | Critical | Boot key for decrypting SAM |
| `C:\Windows\NTDS\NTDS.dit` | Critical | All AD credential hashes |
| `C:\Windows\System32\config\SECURITY` | Critical | LSA secrets, cached creds |
| `%AppData%\Microsoft\Credentials\` | High | Saved Windows credentials |
| `C:\Windows\Prefetch\` | Medium | Execution artifacts |
| `C:\Windows\System32\winevt\Logs\` | High | Security event logs |

### Temporary File Security

`/tmp` and `C:\Windows\Temp` are common attacker drop locations:
- World-writable; any process can write files here
- Malware often stages payloads, extracts archives, or creates named pipes here
- Should be monitored; executables appearing in `/tmp` are highly suspicious

**Linux `/tmp` hardening:**
```
# Mount /tmp with noexec, nosuid, nodev
/tmp  tmpfs  tmpfs  defaults,noexec,nosuid,nodev  0 0
```

### Disk Encryption

| Solution | Platform | Scope |
|----------|----------|-------|
| **BitLocker** | Windows | Full volume; TPM-backed |
| **EFS** | Windows | Per-file; user certificate-bound |
| **dm-crypt / LUKS** | Linux | Full disk / partition |
| **eCryptfs** | Linux | Per-directory (used in Ubuntu home dir encryption) |
| **FileVault 2** | macOS | Full disk |
| **VeraCrypt** | Cross-platform | Full disk or container files |

**Threat model:** Disk encryption protects data at rest (theft of physical media). It does NOT protect against a running, compromised OS — an attacker with OS-level access can read decrypted data.

---

## 8. User Account Control & Privilege Separation

### UAC (Windows)

UAC prevents unauthorized changes by requiring explicit approval for administrative operations:

```
Standard User token (Medium integrity)
        │
        ▼
   Admin action needed
        │
        ├── If admin account → Consent prompt (click Yes/No)
        └── If standard user → Credential prompt (enter admin password)
        │
        ▼
Elevated token (High integrity) for that operation only
```

**UAC bypass techniques (awareness):**
- Auto-elevation of trusted binaries (e.g. `fodhelper.exe`, `eventvwr.exe`)
- DLL hijacking in elevated processes
- COM object hijacking

**UAC is not a security boundary** — it is a convenience mechanism. A determined attacker with user-level code execution can often bypass it.

### sudo (Linux)

`sudo` provides controlled privilege elevation:

```
User runs: sudo command
        │
        ▼
PAM authentication (password prompt)
        │
        ▼
Check /etc/sudoers for authorization
        │
        ├── Allowed → Execute as root (or specified user)
        └── Denied  → Log attempt, refuse
```

**Sudoers best practices:**
```bash
# Good — specific commands only
alice  ALL=(root) /usr/bin/systemctl restart nginx, /usr/bin/apt update

# Bad — unrestricted sudo
alice  ALL=(ALL) ALL

# Very bad — passwordless unrestricted
alice  ALL=(ALL) NOPASSWD: ALL
```

### Privileged Access Workstations (PAW)

A dedicated, hardened machine used exclusively for administrative tasks:
- No email, web browsing, or general use
- Network-isolated from user workstations
- All admin activity performed from PAW only
- Prevents credential theft via user workstation compromise

### Just-In-Time (JIT) Access

Grant elevated privileges only when needed, for a limited time:
- Azure PIM (Privileged Identity Management)
- CyberArk, BeyondTrust PAM solutions
- Eliminates standing privileges — reduces the window of exposure

---

## 9. Mandatory Access Control (MAC)

### SELinux

SELinux assigns a **security context** (label) to every process, file, and socket. The policy defines what labels can interact with what:

```
system_u:system_r:httpd_t:s0      ← Apache process context
system_u:object_r:httpd_sys_content_t:s0  ← Web content context
system_u:object_r:shadow_t:s0     ← /etc/shadow context

Apache (httpd_t) can read httpd_sys_content_t
Apache (httpd_t) CANNOT read shadow_t  ← policy denies
```

Even if Apache is compromised, it cannot read `/etc/shadow` because the MAC policy prevents it.

**SELinux policy types:**
- **Targeted** – Only specific high-risk daemons are confined (default on RHEL/Fedora)
- **Strict** – All processes confined
- **MLS** – Multi-Level Security; Bell-LaPadula confidentiality model

```bash
# Check status
getenforce       # Enforcing / Permissive / Disabled
sestatus

# View file context
ls -Z /etc/shadow
ls -Z /var/www/html/

# View process context
ps -eZ | grep httpd

# Check SELinux denials
grep "avc: denied" /var/log/audit/audit.log
audit2why < /var/log/audit/audit.log

# Temporarily switch to Permissive (debug)
setenforce 0
```

### AppArmor

AppArmor uses file path-based profiles to confine applications:

```
# /etc/apparmor.d/usr.sbin.nginx
/usr/sbin/nginx {
  /var/www/html/** r,        # read web content
  /var/log/nginx/** w,       # write logs
  /etc/nginx/** r,           # read config
  deny /etc/shadow r,        # explicitly deny shadow
  network tcp,               # allow TCP
}
```

Simpler to configure than SELinux; path-based (not label-based).

### Windows Mandatory Integrity Control

Windows MIC prevents low-integrity processes from modifying higher-integrity objects:

```
Internet Explorer (Low) → cannot write to %APPDATA% (Medium)
Notepad (Medium)        → cannot write to C:\Windows\ (System)
Malware dropped from browser → stays at Low integrity, limited damage
```

---

## 10. Logging & Auditing

### Why Logging Matters

Logs are the foundation of:
- **Detection** – Identifying attacks in progress or after the fact
- **Forensics** – Reconstructing what happened during an incident
- **Compliance** – Demonstrating adherence to regulations (PCI-DSS, SOC2, HIPAA)
- **Accountability** – Tying actions to identities

### What to Log

| Category | Events |
|----------|--------|
| **Authentication** | Logon success/failure, logoff, account lockout |
| **Authorization** | Access denied events, privilege use |
| **Account management** | User/group creation, modification, deletion |
| **Process execution** | Process creation with command line |
| **Network** | Connections established, DNS queries |
| **File access** | Reads/writes to sensitive files (via SACL/auditd) |
| **Configuration changes** | Policy changes, service installs, registry modifications |
| **System events** | Boot, shutdown, crashes |

### Windows Audit Policy

```cmd
:: Enable via auditpol
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Account Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Account Management" /success:enable /failure:enable
auditpol /set /subcategory:"Process Creation" /success:enable
auditpol /set /subcategory:"Policy Change" /success:enable /failure:enable
auditpol /set /subcategory:"Privilege Use" /success:enable /failure:enable
auditpol /set /subcategory:"Object Access" /success:enable /failure:enable

:: View current policy
auditpol /get /category:*
```

**Critical Windows Event IDs:**

| Event ID | Description | Why It Matters |
|----------|-------------|---------------|
| 4624 | Successful logon | Baseline; anomalies = lateral movement |
| 4625 | Failed logon | Brute force detection |
| 4648 | Explicit credential logon | runas / pass-the-hash |
| 4672 | Special privileges assigned | Privileged session start |
| 4688 | Process creation (with cmd line) | Malware execution |
| 4698 | Scheduled task created | Persistence |
| 4720 | User account created | Backdoor account |
| 4732 | Added to Administrators | Privilege escalation |
| 4776 | NTLM authentication | NTLM relay detection |
| 7045 | New service installed | Persistence / lateral movement |

### Linux Audit (auditd)

```bash
# File access monitoring
auditctl -w /etc/passwd -p rwa -k passwd_watch
auditctl -w /etc/shadow -p rwa -k shadow_watch
auditctl -w /etc/sudoers -p wa -k sudoers_watch
auditctl -w /root/.ssh/ -p wa -k root_ssh

# Monitor all executions
auditctl -a always,exit -F arch=b64 -S execve -k exec_log

# Monitor privilege escalation
auditctl -a always,exit -F arch=b64 -S setuid -S setgid -k setuid_log

# Search audit log
ausearch -k passwd_watch
ausearch -m EXECVE --start today
ausearch -ua 0             # all root activity

# Reports
aureport --summary
aureport --login --failed
aureport --exec
```

### Log Management Best Practices

- **Centralize logs** – Ship to a SIEM (Splunk, ELK, Microsoft Sentinel) immediately; local logs can be deleted/tampered
- **Protect log integrity** – Write-once storage; cryptographic log signing (e.g. Windows Event Log forward-only subscription)
- **Retention** – Minimum 90 days online; 1 year archived (industry standard; regulatory requirements vary)
- **Alert on key events** – Don't just collect; define detection rules for high-value events
- **Include context** – Timestamp, hostname, user, process, source IP — not just "login failed"

---

## 11. Vulnerability Classes in Operating Systems

### Local Privilege Escalation (LPE)

An attacker with limited local access elevates to root/SYSTEM. Common vectors:

| Vector | Description |
|--------|-------------|
| **Kernel exploit** | Bug in kernel code executed in Ring 0 context |
| **SUID/SGID abuse** | Misconfigured or vulnerable SUID binary |
| **Sudo misconfiguration** | Overly permissive sudoers rules |
| **Writable service binary** | Attacker replaces binary run by a privileged service |
| **Unquoted service path** | Windows path parsing leads to malicious binary execution |
| **DLL hijacking** | Placing malicious DLL where a privileged process loads it |
| **Token impersonation** | Abusing `SeImpersonatePrivilege` (e.g. Potato attacks) |
| **Cron/Task abuse** | Writable script executed by root cron job |
| **PATH hijacking** | Writable directory at start of PATH; malicious binary shadows real one |
| **Capabilities abuse** | Linux capability (e.g. `cap_setuid`) on exploitable binary |

### Kernel Vulnerabilities

Kernel bugs are particularly dangerous because exploitation immediately grants Ring 0 access:

**Common kernel vulnerability types:**
- **Use-After-Free** in kernel objects (slabs, structures)
- **Race conditions** in concurrent kernel code
- **Null pointer dereferences** with attacker-controlled NULL page
- **Integer overflows** in size calculations
- **Insufficient input validation** in syscall handlers

**Notable examples:**
- **DirtyCOW (CVE-2016-5195)** – Race condition in Linux copy-on-write; allowed writing to read-only files; exploited for root escalation
- **Dirty Pipe (CVE-2022-0847)** – Linux pipe buffer vulnerability; arbitrary file writes including SUID binaries
- **EternalBlue (MS17-010)** – SMB buffer overflow; used by WannaCry and NotPetya
- **PrintNightmare (CVE-2021-34527)** – Windows Print Spooler; allowed arbitrary DLL loading as SYSTEM

### Race Conditions (TOCTOU)

Time-of-Check to Time-of-Use: the state of a resource changes between when it is checked and when it is used.

```
Thread A: Check permissions on /tmp/file  ← allowed (symlink not yet there)
Thread B: Replace /tmp/file with symlink to /etc/shadow
Thread A: Write to /tmp/file              ← now writes to /etc/shadow
```

Mitigation: Atomic operations; O_NOFOLLOW; proper locking.

---

## 12. Secure Boot & Firmware Security

### Boot Chain

The boot process is a chain of trust — each stage verifies the next:

```
Power On
    │
    ▼
UEFI Firmware (stored in flash)
    │  Verifies bootloader signature (Secure Boot)
    ▼
Bootloader (GRUB / Windows Boot Manager)
    │  Verifies kernel signature
    ▼
Kernel
    │  Verifies kernel modules (module signing)
    ▼
Init system (systemd / Windows Session Manager)
    │
    ▼
User Space
```

### Secure Boot

UEFI Secure Boot ensures only cryptographically signed bootloaders and kernels run on the hardware:

- Platform Key (PK) → Key Exchange Key (KEK) → Database of allowed signatures (db)
- Unsigned or invalidly signed code is rejected before execution

**Security significance:** Prevents bootkit malware (MBR/UEFI rootkits) from persisting below the OS.

### TPM (Trusted Platform Module)

A dedicated security chip that provides:
- **Secure key storage** – Keys sealed to the TPM; cannot be exported
- **Platform attestation** – Prove to a remote party the machine is in a known-good state
- **Measured boot** – Records hash of each boot stage in PCRs (Platform Configuration Registers)
- **BitLocker** – TPM seals the volume encryption key; only releases it if boot measurements match

**TPM-based attacks:**
- **Cold boot attack** – Freeze RAM to preserve encryption keys after shutdown; read keys before they decay
- **Evil maid** – Physical access to modify boot chain; TPM attestation detects unauthorized changes

### UEFI Vulnerabilities

- **BootHole (CVE-2020-10713)** – GRUB2 buffer overflow bypassing Secure Boot
- **LogoFAIL** – UEFI image parsers exploitable to inject code during boot
- Firmware updates often unsigned or infrequently applied

---

## 13. Virtualization & Container Security

### Hypervisors

A hypervisor creates and manages virtual machines (VMs):

| Type | Description | Examples |
|------|-------------|---------|
| **Type 1 (Bare Metal)** | Runs directly on hardware | VMware ESXi, Hyper-V, KVM |
| **Type 2 (Hosted)** | Runs on top of a host OS | VMware Workstation, VirtualBox |

**Security benefit:** VM isolation — a compromised guest cannot directly access other guests or the hypervisor (in theory).

**VM escape:** Exploiting a hypervisor vulnerability to break out of a VM and execute code on the host. High severity — bypasses all guest-level security.

### Virtualization-Based Security (VBS)

Windows uses Hyper-V to create isolated virtual trust levels (VTLs):

```
VTL 1 (Secure World)   ← Credential Guard, HVCI, Secure Kernel
────────────────────────────────────────────────────────────
VTL 0 (Normal World)   ← Windows OS, applications
```

- **Credential Guard** – Isolates LSASS into VTL1; even SYSTEM processes in VTL0 cannot dump credentials
- **HVCI (Hypervisor-Protected Code Integrity)** – Kernel code must be signed; prevents kernel exploits loading unsigned drivers

### Containers

Containers share the host OS kernel but are isolated via Linux namespaces and cgroups:

```
Host Kernel
├── Container A (namespace: PID, NET, MNT, UTS, IPC, USER)
├── Container B (namespace: PID, NET, MNT, UTS, IPC, USER)
└── Container C (namespace: PID, NET, MNT, UTS, IPC, USER)
```

**Container vs VM security:**

| Aspect | Container | VM |
|--------|-----------|-----|
| Isolation | Namespace-based (weaker) | Hardware-enforced (stronger) |
| Kernel | Shared with host | Separate kernel per VM |
| Attack surface | Host kernel exposed | Hypervisor exposed (smaller) |
| Performance | Near-native | Slight overhead |

**Container escape:** Exploiting a vulnerability to break out of the container namespace and access the host. Common vectors:
- Privileged containers (`--privileged`)
- Exposed Docker socket (`/var/run/docker.sock`)
- Kernel exploits (shared kernel)
- Misconfigured capabilities

**Container hardening:**
```bash
# Never run as root in container
USER nonroot

# Drop all capabilities; add only what's needed
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE image

# Read-only filesystem
docker run --read-only image

# No privileged
docker run --security-opt no-new-privileges image

# Use seccomp profile
docker run --security-opt seccomp=/path/to/profile.json image
```

---

## 14. Network Security at the OS Level

### OS-Level Network Protections

**IP Source Routing** – Attacker specifies route for packets; bypasses routing-based controls.
```bash
# Disable (Linux)
sysctl -w net.ipv4.conf.all.accept_source_route=0
```

**ICMP Redirect** – Attacker redirects traffic through a malicious router.
```bash
sysctl -w net.ipv4.conf.all.accept_redirects=0
sysctl -w net.ipv4.conf.all.send_redirects=0
```

**SYN Flood Protection** (SYN Cookies):
```bash
sysctl -w net.ipv4.tcp_syncookies=1
```

**IP Forwarding** – Should be disabled on non-router systems:
```bash
sysctl -w net.ipv4.ip_forward=0
```

**Broadcast ICMP** (Smurf attack amplification):
```bash
sysctl -w net.ipv4.icmp_echo_ignore_broadcasts=1
```

### Host-Based Firewalls

Every OS should run a host-based firewall as a defense-in-depth control, even behind network firewalls:

| OS | Tool |
|----|------|
| Windows | Windows Defender Firewall (`netsh advfirewall`, `New-NetFirewallRule`) |
| Linux (Debian) | UFW (`ufw`) over iptables |
| Linux (RHEL) | firewalld over nftables |
| Linux (advanced) | iptables / nftables directly |

**Default policy principle:**
```
Inbound:  DEFAULT DENY → explicitly allow what is needed
Outbound: DEFAULT ALLOW → optionally restrict to known-good destinations
```

### Common Exposed Services & Risks

| Service | Port | Risk if Exposed |
|---------|------|----------------|
| SSH | 22 | Brute force; key compromise; CVEs |
| RDP | 3389 | BlueKeep; brute force; MitM |
| SMB | 445 | EternalBlue; relay attacks; ransomware spread |
| WinRM | 5985/5986 | Lateral movement; credential relay |
| Telnet | 23 | Cleartext credentials |
| FTP | 21 | Cleartext credentials |
| rsh/rlogin | 514/513 | Trust-based auth; no encryption |
| VNC | 5900 | Weak auth; no encryption by default |
| Redis | 6379 | Unauthenticated by default; RCE |
| MongoDB | 27017 | Unauthenticated by default |

---

## 15. Patch Management

### Why Patching Matters

The majority of successful attacks exploit known vulnerabilities with public patches available. Organizations that fall behind on patching are disproportionately exposed:

- **Equifax (2017)** – Apache Struts CVE-2017-5638; patch available 2 months before breach
- **WannaCry (2017)** – MS17-010 (EternalBlue); patch available 59 days before outbreak
- **NotPetya (2017)** – Same MS17-010 vector

### Patch Management Process

```
1. Inventory
   └── Know what OS versions, software, and components are running

2. Monitor
   └── Subscribe to vendor advisories, CVE feeds (NVD, vendor bulletins)

3. Prioritize
   └── CVSS score + exploitability + asset criticality
       Critical/High with public exploits → patch within 24-72 hours
       High → patch within 7-14 days
       Medium → patch within 30 days
       Low → patch within 90 days

4. Test
   └── Deploy to dev/staging first; validate no breakage

5. Deploy
   └── Automated deployment (WSUS, Ansible, SCCM, Chef)

6. Verify
   └── Confirm patch applied; scan for remaining vulnerable systems

7. Document
   └── Record what was patched, when, and by whom
```

### Windows Patching

```powershell
# Check installed patches
Get-HotFix | Sort-Object InstalledOn -Descending
wmic qfe list brief /format:table

# Last Windows Update check
(New-Object -ComObject Microsoft.Update.AutoUpdate).Results.LastSearchSuccessDate

# Force Windows Update check (via PSWindowsUpdate module)
Install-Module PSWindowsUpdate
Get-WindowsUpdate
Install-WindowsUpdate -AcceptAll -AutoReboot
```

### Linux Patching

```bash
# Debian / Ubuntu
apt update && apt upgrade -y
apt list --upgradeable

# Check for security updates only
unattended-upgrades --dry-run
apt upgrade -s | grep "^Inst" | grep -i security

# RHEL / CentOS / Rocky
dnf update -y
dnf check-update --security
dnf update --security -y

# Check CVEs affecting installed packages
dnf updateinfo list security

# Automatic security updates (Debian)
apt install unattended-upgrades
dpkg-reconfigure unattended-upgrades
```

---

## 16. Endpoint Detection & Security Tools

### Antivirus / Anti-Malware

**Detection approaches:**

| Approach | Description | Limitation |
|----------|-------------|-----------|
| **Signature-based** | Match file hash or byte patterns to known malware | Misses novel/modified malware |
| **Heuristic** | Flag suspicious code patterns | False positives; evasion via obfuscation |
| **Behavioral** | Monitor process actions at runtime | Performance overhead; evasion via living-off-the-land |
| **Cloud-based reputation** | Submit file hash to cloud for verdict | Privacy concerns; requires connectivity |
| **ML/AI-based** | Trained models to classify files | Can be adversarially fooled |

### EDR (Endpoint Detection & Response)

EDR solutions provide richer telemetry and detection than AV:

| Capability | Description |
|-----------|-------------|
| Process telemetry | Process creation, parent-child relationships, command lines |
| File events | Creates, modifies, deletes |
| Network events | DNS queries, TCP connections per process |
| Registry events | Key/value creation, modification |
| Memory scanning | Detect injected code, hollowed processes |
| Response actions | Remote isolation, process kill, file quarantine |

Examples: CrowdStrike Falcon, Microsoft Defender for Endpoint, SentinelOne, Carbon Black.

### HIDS / HIPS

- **HIDS (Host-based IDS)** – Detect suspicious activity via log analysis or file integrity monitoring
  - Examples: OSSEC, Wazuh, Tripwire
- **HIPS (Host-based IPS)** – Actively block suspicious behavior in real time
  - Examples: Windows Defender Exploit Guard, CrowdStrike prevention mode

### File Integrity Monitoring (FIM)

Detects unauthorized changes to critical system files:

```bash
# AIDE (Advanced Intrusion Detection Environment) — Linux
aide --init                    # create baseline database
aide --check                   # compare current state to baseline

# Tripwire — Linux
tripwire --init
tripwire --check

# Windows — Sysmon event ID 11 (file creation), 2 (file time change)
# Windows Defender for Endpoint — built-in FIM
```

Critical files to monitor:
- `/etc/passwd`, `/etc/shadow`, `/etc/sudoers`
- `/etc/ssh/sshd_config`
- `/bin`, `/sbin`, `/usr/bin` — system binaries
- `C:\Windows\System32\` — Windows system files
- All web application files

### AMSI (Antimalware Scan Interface) — Windows

AMSI allows security software to scan content before it's executed — including PowerShell scripts, VBA macros, and JavaScript:

```
PowerShell script
      │
      ▼
AMSI provider (Defender or third-party)
      │
      ├── Clean  → Execute
      └── Malicious → Block + Alert
```

Common AMSI bypass techniques (awareness): patching `AmsiScanBuffer` in memory, obfuscation, reflection-based loading.

---

## 17. OS Hardening Checklist

### Universal Principles

```
[ ] Apply all security patches; establish a regular patching cadence
[ ] Remove or disable all unnecessary services, features, and software
[ ] Change all default credentials immediately
[ ] Enable host-based firewall with default-deny inbound policy
[ ] Enable comprehensive audit logging; forward to central SIEM
[ ] Enforce least privilege — no standing admin rights for daily use
[ ] Enable disk encryption (BitLocker / LUKS)
[ ] Implement MFA for all administrative and remote access
[ ] Segment the network — limit what systems can reach each other
[ ] Maintain an asset inventory — you can't protect what you don't know about
```

### Windows Hardening

```
[ ] Disable SMBv1 (Set-SmbServerConfiguration -EnableSMB1Protocol $false)
[ ] Disable LLMNR and NBT-NS (Group Policy)
[ ] Enable Credential Guard (VBS + TPM required)
[ ] Enable LSASS protection (RunAsPPL = 1)
[ ] Enable Windows Defender ASR rules
[ ] Disable PowerShell v2
[ ] Enable Script Block Logging and Transcription
[ ] Set UAC to "Always Notify"
[ ] Enable Windows Defender Firewall on all profiles
[ ] Remove local admin rights from standard users
[ ] Add privileged accounts to Protected Users group
[ ] Deploy Sysmon with a hardened configuration
[ ] Configure AppLocker or WDAC for application whitelisting
[ ] Enable BitLocker on all drives
[ ] Enforce strong password policy via Group Policy
[ ] Disable Remote Registry service
[ ] Disable Telnet, FTP, and other legacy protocols
```

### Linux Hardening

```
[ ] Disable root SSH login (PermitRootLogin no)
[ ] Disable password authentication for SSH (PasswordAuthentication no)
[ ] Change SSH port from 22 (optional; reduces noise)
[ ] Enable and configure UFW / iptables default-deny
[ ] Install and configure fail2ban
[ ] Enable SELinux (Enforcing) or AppArmor
[ ] Set restrictive umask (027 or 077)
[ ] Disable unused network services (disable with systemctl)
[ ] Enable auditd with file and syscall monitoring rules
[ ] Mount /tmp with noexec, nosuid, nodev
[ ] Set GRUB password to prevent single-user mode access
[ ] Disable USB storage if not needed (blacklist usb-storage)
[ ] Enable sysctl kernel hardening parameters
[ ] Restrict cron to root (chmod 700 /etc/cron*)
[ ] Remove or disable SUID from non-essential binaries
[ ] Enable core dump restrictions (fs.suid_dumpable = 0)
[ ] Configure PAM for strong password policy and account lockout
[ ] Deploy and configure AIDE or Tripwire for FIM
```

---

## 18. Incident Response at the OS Level

### IR Phases

```
Preparation → Identification → Containment → Eradication → Recovery → Lessons Learned
```

### Identification — Signs of Compromise

**Processes:**
- Unexpected processes running; processes with unusual parent-child relationships
- Processes with no file path on disk (deleted executables still running)
- Known system binaries (`svchost.exe`, `ls`) running from unexpected locations
- Processes making outbound connections on unusual ports

**Network:**
- Outbound connections to unknown external IPs
- Internal lateral movement (unusual SMB, RDP, WinRM connections)
- DNS queries to newly registered or high-entropy domains
- Large outbound data transfers (exfiltration)

**Persistence:**
- New scheduled tasks or services
- New autorun registry entries
- New SUID binaries or modified system files
- New user accounts or group membership changes
- Modified `.bashrc`, `.profile`, or Windows shell registry keys

**Logs:**
- Spike in failed authentication events
- Successful logon after repeated failures
- Logon from unusual hours or locations
- Cleared event logs (`wevtutil cl Security`)
- Gaps in log timeline (log tampering)

### Containment Actions

```bash
# Linux — isolate host from network
iptables -I INPUT -j DROP
iptables -I OUTPUT -j DROP
iptables -I INPUT -s trusted_admin_ip -j ACCEPT

# Windows — isolate host
New-NetFirewallRule -DisplayName "IR Block All In" -Direction Inbound -Action Block
New-NetFirewallRule -DisplayName "IR Block All Out" -Direction Outbound -Action Block
# Or use Windows Defender for Endpoint isolation (single click in portal)

# Disable a compromised account
net user compromised_user /active:no     # Windows
usermod -L compromised_user             # Linux

# Kill a malicious process (after memory capture if possible)
taskkill /PID <pid> /F                  # Windows
kill -9 <pid>                           # Linux
```

### Evidence Collection Order (Volatility Principle)

Collect most volatile data first:

```
1. System time (timestamp reference)
2. Running processes (memory-resident; lost on reboot)
3. Network connections (ephemeral)
4. Logged-in users
5. Open files and handles
6. Clipboard / environment
7. RAM dump (if feasible)
8. Disk image (non-volatile; can be done later)
9. Log files
10. Configuration files
```

---

## 19. Common CVEs & Vulnerability Patterns

### Historical High-Impact OS Vulnerabilities

| CVE | System | Type | Impact |
|-----|--------|------|--------|
| CVE-2017-0144 (EternalBlue) | Windows SMB | Buffer overflow | Remote code execution as SYSTEM |
| CVE-2021-34527 (PrintNightmare) | Windows Print Spooler | Privilege escalation | SYSTEM via DLL loading |
| CVE-2021-4034 (PwnKit) | Linux polkit | Buffer overflow | Local root on most Linux distros |
| CVE-2022-0847 (Dirty Pipe) | Linux kernel | Pipe buffer flaw | Write to read-only files; root via SUID |
| CVE-2016-5195 (DirtyCOW) | Linux kernel | Race condition | Write to read-only files; root |
| CVE-2020-0796 (SMBGhost) | Windows SMBv3 | Integer overflow | Remote code execution |
| CVE-2019-0708 (BlueKeep) | Windows RDP | Use-after-free | Pre-auth remote code execution |
| CVE-2014-6271 (Shellshock) | Bash | Command injection | RCE via environment variable |
| CVE-2021-3156 (Baron Samedit) | sudo | Heap overflow | Root on most Linux/macOS |
| CVE-2023-23397 | Windows Outlook | NTLM relay | Hash capture without user interaction |

### Vulnerability Scoring (CVSS)

CVSS (Common Vulnerability Scoring System) v3.1 rates vulnerabilities 0–10:

| Score | Severity |
|-------|---------|
| 0.0 | None |
| 0.1 – 3.9 | Low |
| 4.0 – 6.9 | Medium |
| 7.0 – 8.9 | High |
| 9.0 – 10.0 | Critical |

**CVSS Base Metrics:**
- **Attack Vector** – Network / Adjacent / Local / Physical
- **Attack Complexity** – Low / High
- **Privileges Required** – None / Low / High
- **User Interaction** – None / Required
- **Scope** – Unchanged / Changed
- **Confidentiality / Integrity / Availability Impact** – None / Low / High

---

## 20. Security Compliance Frameworks

### CIS Benchmarks

The Center for Internet Security publishes hardening benchmarks for every major OS. Two levels:

- **Level 1** – Essential security; minimal performance/functionality impact
- **Level 2** – Defense-in-depth; may impact functionality

```bash
# Assess compliance with CIS benchmarks
# Tools: OpenSCAP (Linux), CIS-CAT (cross-platform)

# OpenSCAP on RHEL
oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis \
    --results scan-results.xml \
    /usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml
```

### DISA STIGs

Security Technical Implementation Guides — US DoD hardening standards for every major OS, application, and device. More prescriptive than CIS benchmarks.

### NIST SP 800-53

Security and Privacy Controls for Information Systems — comprehensive framework used by US federal agencies. Relevant control families:

| Family | Focus |
|--------|-------|
| AC – Access Control | Authentication, authorization, least privilege |
| AU – Audit | Logging, log protection, review |
| CM – Configuration Management | Baseline configs, patch management |
| IA – Identification & Authentication | MFA, password policy |
| SC – System & Communications | Encryption, network segmentation |
| SI – System Integrity | Malware protection, FIM, patching |

### SOC 2

Trust Services Criteria used for service provider audits. Security category maps closely to OS controls: access control, monitoring, change management, risk assessment.

---

## Quick Reference Summary

| Topic | Key Concept |
|-------|------------|
| **Authentication** | Never store plaintext passwords; use salted, slow hashes; enable MFA |
| **Authorization** | Least privilege; DAC for normal use; MAC for high-security systems |
| **Memory safety** | ASLR + DEP/NX + stack canaries together significantly raise exploit cost |
| **Kernel security** | Kernel bugs = full system compromise; keep patched; restrict kernel module loading |
| **Logging** | Log everything; centralize immediately; alert on key events |
| **Patching** | Most breaches exploit known CVEs; patch critical/high within days |
| **Hardening** | Remove everything unnecessary; default-deny; validate with benchmarks |
| **Sandboxing** | Contain breach impact; browser/mail in restricted context |
| **Disk encryption** | Protects at rest only; not a substitute for OS-level access controls |
| **Secure boot** | Validates boot chain integrity; blocks persistent firmware/bootkit malware |
| **IR** | Preserve volatile data first; isolate before eradicating; document everything |

---

## Further Reading

- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/)
- [NIST SP 800-53](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final)
- [MITRE ATT&CK – Enterprise](https://attack.mitre.org/)
- [CVE / NVD](https://nvd.nist.gov/)
- [DISA STIGs](https://public.cyber.mil/stigs/)
- [Linux Kernel Security Subsystem](https://www.kernel.org/doc/html/latest/security/index.html)
- [Microsoft Security Baseline](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-security-configuration-framework/windows-security-baselines)
- [Windows Internals (book)](https://docs.microsoft.com/en-us/sysinternals/resources/windows-internals)
- [The Linux Programming Interface (book)](https://man7.org/tlpi/)