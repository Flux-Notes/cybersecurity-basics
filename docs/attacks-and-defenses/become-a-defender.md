# 🛡️ Become a Defender — The Complete Blue Team Roadmap

> *"Security is not a product, but a process. Defense is not a wall — it's a way of thinking."*

This guide covers the **A to Z journey** of becoming a skilled cybersecurity defender / blue teamer — from foundational concepts to advanced threat hunting, incident response, and enterprise security operations. Built as the companion to the Hacker roadmap.

---

## 1. What is Blue Teaming?

**Blue Teaming** is the practice of defending systems, networks, and data against attacks. While red teams think like attackers, blue teamers think like protectors — building, monitoring, and improving security posture continuously.

### Red vs. Blue vs. Purple
| Team | Role | Focus |
|------|------|-------|
| 🔴 **Red Team** | Offensive | Simulate real-world attacks |
| 🔵 **Blue Team** | Defensive | Detect, respond, recover |
| 🟣 **Purple Team** | Collaborative | Red + Blue working together |
| ⚪ **White Team** | Administrative | Rules of engagement, scoring |

### The Defender's Core Mission
- **Prevent** — Stop attacks before they happen
- **Detect** — Find attacks that slip through
- **Respond** — Contain and eliminate threats
- **Recover** — Restore normal operations
- **Learn** — Improve defenses from every incident

---

## 2. The Defender's Mindset

### Think Like an Attacker to Defend Like a Pro
You don't need to be an expert attacker, but you must understand:
- What attackers want (data, money, disruption, persistence)
- How attackers operate (the kill chain, TTPs)
- What evidence attacks leave behind (logs, artifacts, anomalies)

### Key Traits of Great Defenders
- **Curiosity** — Always ask "why is this happening?"
- **Skepticism** — Assume breach; trust nothing by default
- **Attention to detail** — Attackers hide in the noise
- **Composure under pressure** — Incidents happen at 2 AM
- **Communication** — Translate technical findings for leadership
- **Continuous improvement** — Every incident is a learning opportunity

### Assume Breach Mentality
Modern defense is built on the assumption that attackers **will** get in. The question is: how fast can you detect and evict them?

> Mean Time to Detect (MTTD) and Mean Time to Respond (MTTR) are the metrics that matter most.

---

## 3. Foundational Knowledge

Before specializing, build these fundamentals:

### 3.1 Networking Essentials
- TCP/IP stack, OSI model
- Subnetting, VLANs, routing
- DNS, DHCP, HTTP/S, SMTP, FTP, SSH, SMB
- Firewalls, IDS/IPS, proxies, load balancers
- Packet analysis with Wireshark/tcpdump

### 3.2 Operating Systems
- **Windows** — Registry, Event Logs, Active Directory, PowerShell, Task Scheduler, Services
- **Linux** — syslog, auditd, cron, processes, file permissions, systemd
- **macOS** — Unified logging, launch agents, Gatekeeper, SIP

### 3.3 Security Concepts
- CIA Triad (Confidentiality, Integrity, Availability)
- Authentication vs. Authorization
- Principle of Least Privilege
- Defense in Depth
- Zero Trust Architecture
- Risk = Threat × Vulnerability × Impact

---

## 4. Security Frameworks & Standards

Frameworks give structure to defense. Know these inside-out.

### 4.1 MITRE ATT&CK Framework
The most important framework for defenders. Maps adversary **Tactics, Techniques, and Procedures (TTPs)**.

```
Tactics (the "why") — 14 categories:
Reconnaissance → Resource Development → Initial Access → Execution
→ Persistence → Privilege Escalation → Defense Evasion
→ Credential Access → Discovery → Lateral Movement
→ Collection → Command & Control → Exfiltration → Impact

Techniques (the "how") — 500+ specific methods
Sub-techniques — More granular detail

Use ATT&CK to:
- Map your detection coverage
- Prioritize what to detect
- Understand attack paths
- Build threat hunt hypotheses
```

### 4.2 NIST Cybersecurity Framework (CSF)
```
IDENTIFY   → Know your assets, risks, and environment
PROTECT    → Implement safeguards
DETECT     → Monitor for events
RESPOND    → Take action on detected events
RECOVER    → Restore capabilities after incidents
```

### 4.3 Cyber Kill Chain (Lockheed Martin)
```
1. Reconnaissance     → Attacker researches the target
2. Weaponization      → Creates exploit + payload
3. Delivery           → Sends malicious content
4. Exploitation       → Exploit executes on target
5. Installation       → Malware installs persistence
6. C2 (Command & Control) → Attacker connects to implant
7. Actions on Objectives  → Exfil, ransomware, sabotage

Defender goal: Break the chain as early as possible.
Earlier = cheaper to stop. Stage 1 detection > Stage 7 detection.
```

### 4.4 Other Key Frameworks
| Framework | Purpose |
|-----------|---------|
| **ISO 27001** | Information security management system |
| **CIS Controls** | Prioritized security best practices (18 controls) |
| **SOC 2** | Security trust criteria for service organizations |
| **PCI DSS** | Payment card industry security standard |
| **HIPAA** | Healthcare data security requirements |
| **GDPR** | EU data protection regulation |
| **OWASP ASVS** | Application security verification standard |

---

## 5. Security Operations Center (SOC)

The SOC is the nerve center of enterprise defense.

### 5.1 SOC Analyst Tiers
| Tier | Role | Responsibilities |
|------|------|-----------------|
| **Tier 1** | Alert Triage | Monitor alerts, initial triage, escalate |
| **Tier 2** | Incident Handler | Deep investigation, containment |
| **Tier 3** | Threat Hunter | Proactive hunting, advanced analysis |
| **SOC Manager** | Leadership | Operations, metrics, strategy |

### 5.2 SOC Daily Workflow
```
Morning:
- Review overnight alerts and escalations
- Check threat intel feeds for new IOCs
- Review dashboards for anomalies

Throughout the day:
- Triage new alerts (true positive vs false positive)
- Investigate escalated incidents
- Update tickets and documentation
- Correlate events across data sources

Ongoing:
- Tune detection rules (reduce false positives)
- Update runbooks from lessons learned
- Attend threat intel briefings
```

### 5.3 Key SOC Metrics
| Metric | Description | Target |
|--------|-------------|--------|
| **MTTD** | Mean Time to Detect | < 1 hour |
| **MTTR** | Mean Time to Respond | < 4 hours |
| **False Positive Rate** | Alerts that aren't real threats | < 20% |
| **Dwell Time** | How long attacker was undetected | < 24 hours |
| **Alert Volume** | Alerts per analyst per day | Manageable (<100) |

---

## 6. Log Management & SIEM

Logs are the defender's eyes. Without them, you're flying blind.

### 6.1 Critical Log Sources
```
Windows:
├── Security Event Log     → Logins, privilege use, policy changes
├── System Event Log       → Service starts/stops, hardware errors
├── Application Event Log  → Application errors and events
├── PowerShell logs        → Script execution (enable enhanced logging!)
├── Sysmon logs            → Process creation, network connections, file events
└── Windows Firewall logs  → Network traffic allowed/blocked

Linux:
├── /var/log/auth.log      → Authentication events (Ubuntu/Debian)
├── /var/log/secure        → Auth events (RHEL/CentOS)
├── /var/log/syslog        → General system messages
├── /var/log/audit/audit.log → Auditd events
├── /var/log/apache2/      → Web server access/error logs
└── journalctl             → systemd journal

Network:
├── Firewall logs          → Allow/deny decisions
├── DNS query logs         → Domain lookups (critical for C2 detection)
├── DHCP logs              → IP assignment history
├── NetFlow/IPFIX          → Traffic metadata (no content)
├── Proxy logs             → Web requests
└── VPN logs               → Remote access

Applications:
├── Web application logs   → HTTP requests, errors
├── Database logs          → Queries, login attempts
├── Email gateway logs     → Inbound/outbound mail
└── EDR/AV logs            → Endpoint detections
```

### 6.2 Windows Event IDs — Must Know
| Event ID | Description | Why It Matters |
|----------|-------------|----------------|
| **4624** | Successful logon | Baseline normal logins |
| **4625** | Failed logon | Brute force detection |
| **4648** | Logon with explicit credentials | Pass-the-hash indicator |
| **4672** | Special privileges assigned | Admin login |
| **4688** | Process created | Track execution |
| **4698** | Scheduled task created | Persistence mechanism |
| **4720** | User account created | Unauthorized accounts |
| **4732** | User added to security group | Privilege escalation |
| **4768** | Kerberos TGT requested | AD authentication |
| **4769** | Kerberos service ticket requested | Kerberoasting indicator |
| **4771** | Kerberos pre-auth failed | AS-REP roasting |
| **4776** | NTLM authentication | Credential attacks |
| **7045** | New service installed | Malware persistence |

### 6.3 Sysmon — Enhanced Windows Logging
Sysmon (System Monitor) provides critical visibility beyond native Windows logging. Install on all Windows endpoints.

```xml
<!-- Key Sysmon Event IDs -->
Event ID 1  → Process Create (with full command line)
Event ID 3  → Network Connection
Event ID 6  → Driver Loaded
Event ID 7  → Image Loaded (DLL)
Event ID 8  → CreateRemoteThread (process injection indicator)
Event ID 10 → ProcessAccess (credential dumping indicator)
Event ID 11 → FileCreate
Event ID 12/13 → Registry events
Event ID 15 → FileCreateStreamHash (ADS)
Event ID 22 → DNS Query
Event ID 23 → FileDelete
Event ID 25 → ProcessTampering
```

Install with SwiftOnSecurity or Olaf Hartong's config:
```powershell
# Install Sysmon with config
sysmon64.exe -accepteula -i sysmonconfig.xml

# Update config
sysmon64.exe -c sysmonconfig.xml
```

### 6.4 SIEM Platforms
A SIEM (Security Information and Event Management) aggregates, correlates, and alerts on log data.

| SIEM | Type | Best For |
|------|------|----------|
| **Splunk** | Commercial | Enterprise, most features |
| **Microsoft Sentinel** | Cloud (Azure) | Microsoft-heavy environments |
| **IBM QRadar** | Commercial | Large enterprises |
| **Elastic SIEM** | Open source + commercial | Budget-conscious teams |
| **Wazuh** | Open source | SMBs, home labs |
| **Chronicle** | Cloud (Google) | Threat hunting at scale |
| **Graylog** | Open source | Log management |

### 6.5 Writing SIEM Detection Rules
```
Good detection rule principles:
1. High fidelity   → Few false positives
2. High coverage   → Catches real attacks
3. Context-rich    → Includes relevant fields
4. Actionable      → Analyst knows what to do

Example Splunk query — Detect brute force:
index=wineventlog EventCode=4625
| stats count by src_ip, user
| where count > 10
| sort -count

Example — Detect PowerShell encoded commands (common malware technique):
index=wineventlog EventCode=4688 
CommandLine="*-EncodedCommand*" OR CommandLine="*-enc *"
| table _time, host, user, CommandLine

Example — Detect suspicious scheduled tasks:
index=wineventlog EventCode=4698
| eval suspicious=if(match(TaskName,"(?i)(update|svchost|system32)"), "maybe_legit", "investigate")
| table _time, host, user, TaskName, suspicious
```

---

## 7. Endpoint Detection & Response (EDR)

EDR tools provide deep visibility into endpoint activity and enable rapid response.

### 7.1 EDR Capabilities
- Real-time process monitoring and telemetry
- File system and registry monitoring
- Network connection visibility
- Behavioral detection (not just signatures)
- Threat hunting interface
- Remote response (isolate, kill process, delete file)
- Timeline reconstruction for forensics

### 7.2 Major EDR Platforms
| Platform | Vendor | Notes |
|----------|--------|-------|
| **CrowdStrike Falcon** | CrowdStrike | Industry leader, cloud-native |
| **Microsoft Defender for Endpoint** | Microsoft | Deep Windows integration |
| **SentinelOne** | SentinelOne | Strong autonomous response |
| **Carbon Black** | VMware | Enterprise-focused |
| **Cortex XDR** | Palo Alto | Part of broader XDR platform |
| **Wazuh** | Open source | Free, integrates with ELK |
| **Velociraptor** | Open source | Threat hunting, DFIR |

### 7.3 What to Hunt for on Endpoints
```
Suspicious process relationships:
- Word/Excel spawning cmd.exe or PowerShell
- Browser spawning wscript.exe or mshta.exe
- svchost.exe with unusual parent

Malicious execution paths:
- Execution from C:\Users\Public\
- Execution from C:\Temp\
- Execution from %AppData%
- Double extensions (invoice.pdf.exe)

Living-off-the-land binaries (LOLBins):
- certutil.exe -decode / -urlcache
- mshta.exe http://...
- regsvr32 /i /s /n /u http://...
- wscript.exe / cscript.exe running unusual scripts
- bitsadmin.exe /transfer

Credential dumping indicators:
- lsass.exe being accessed by unusual processes
- procdump.exe, mimikatz.exe, sekurlsa
- Volume shadow copy deletion (vssadmin delete shadows)
```

---

## 8. Network Security Monitoring (NSM)

Monitor the wire. Attacks always leave network traces.

### 8.1 Network Security Tools
```bash
# Zeek (formerly Bro) — network analysis framework
# Generates structured logs: conn.log, dns.log, http.log, ssl.log, files.log

# Suricata — IDS/IPS + NSM
suricata -c /etc/suricata/suricata.yaml -i eth0

# tcpdump — packet capture
tcpdump -i eth0 -w capture.pcap
tcpdump -i eth0 'port 53' -A       # DNS queries
tcpdump -i eth0 'tcp[13] == 2'     # SYN packets

# Wireshark — GUI packet analysis
# Key filters for defenders:
http.request.method == "POST"      # Data exfil
dns.qry.name contains ".onion"     # Tor usage
tcp.flags.syn==1 && tcp.flags.ack==0  # Port scans
ssl.handshake.extensions_server_name  # SNI (domain in HTTPS)
```

### 8.2 What to Monitor on the Network
```
Indicators of Command & Control (C2):
- Regular, beaconing intervals to external IPs
- DNS requests to DGA (randomly generated) domains
- Long-duration connections to unusual ports
- Encrypted traffic to non-standard ports
- HTTPS to IPs rather than domain names
- High data transfer to cloud storage (exfiltration)

Lateral Movement Indicators:
- SMB connections between workstations
- RDP from non-admin workstations
- WMI/PowerShell remoting activity
- Pass-the-hash (NTLM to many hosts quickly)
- Port scanning from internal host

DNS Anomalies:
- DNS queries for recently registered domains
- High volume of NXDOMAIN responses (malware scanning)
- Unusually long DNS TXT records (DNS tunneling)
- DNS to non-corporate resolvers
```

### 8.3 Network Segmentation
```
Best practice network architecture:

Internet
    │
  [WAF]
    │
  [DMZ] ──── Web servers, mail relay, DNS
    │
  [Firewall]
    │
  [Corporate LAN]
    ├── [User VLAN]      (workstations)
    ├── [Server VLAN]    (file, app, database servers)
    ├── [Admin VLAN]     (privileged access workstations)
    ├── [IoT VLAN]       (printers, cameras, building systems)
    └── [OT/ICS VLAN]   (industrial control systems)

Key rules:
- User VLAN cannot reach Server VLAN directly
- No lateral movement between workstations
- Admin VLAN strictly controlled
- All cross-VLAN traffic logged
```

---

## 9. Threat Intelligence

Know your enemy before they attack you.

### 9.1 Types of Threat Intelligence
| Type | Description | Example |
|------|-------------|---------|
| **Strategic** | High-level trends, actor motivations | "APT28 targeting energy sector" |
| **Operational** | Specific campaigns and TTPs | "New phishing campaign using DocuSign lure" |
| **Tactical** | Specific IOCs (IPs, hashes, domains) | `192.168.1.1`, `malware.exe` hash |
| **Technical** | Vulnerability and exploit details | CVE-2024-XXXX details |

### 9.2 Indicators of Compromise (IOCs)
```
IP-based IOCs:
- Malicious IPs (C2 servers, known scanners)
- Tor exit nodes
- Anonymizing proxies/VPNs

Domain-based IOCs:
- Malicious domains
- Typosquatted domains (gooogle.com)
- DGA-generated domains
- Recently registered domains

File-based IOCs:
- MD5/SHA1/SHA256 hashes of malware
- Malicious file names
- Suspicious file paths

Behavioral IOCs:
- Registry keys created by malware
- Mutex names used by malware
- Network patterns (beacon interval, URI patterns)
- YARA rules matching malware patterns
```

### 9.3 Threat Intel Feeds & Platforms
| Source | Type | Cost |
|--------|------|------|
| **MISP** | Open source platform | Free |
| **OpenCTI** | Open source platform | Free |
| **AlienVault OTX** | Community feeds | Free |
| **VirusTotal** | File/URL/IP reputation | Free + paid |
| **Shodan** | Internet scanning data | Free + paid |
| **Recorded Future** | Commercial intel | Paid |
| **Mandiant Advantage** | Commercial intel | Paid |
| **CISA AIS** | Government feeds | Free |
| **AbuseCH** | Malware/botnet feeds | Free |

### 9.4 Integrating Threat Intel
```python
# Check IP against VirusTotal API
import requests

def check_ip(ip, api_key):
    url = f"https://www.virustotal.com/api/v3/ip_addresses/{ip}"
    headers = {"x-apikey": api_key}
    response = requests.get(url, headers=headers)
    data = response.json()
    
    malicious = data['data']['attributes']['last_analysis_stats']['malicious']
    return malicious > 0

# Enrich SIEM alerts automatically with IOC lookups
# Tag alerts with threat actor attribution
# Auto-block known malicious IPs via firewall API
```

---

## 10. Vulnerability Management

Find and fix your weaknesses before attackers do.

### 10.1 Vulnerability Management Process
```
1. ASSET INVENTORY
   → Know every device, application, and service
   → Use CMDB (Configuration Management Database)
   → Automated discovery tools

2. SCANNING
   → Run authenticated vulnerability scans
   → Scan frequency: critical assets weekly, others monthly

3. ASSESSMENT
   → Prioritize by CVSS score, exploitability, asset criticality
   → Check if exploit is publicly available
   → Consider compensating controls

4. REMEDIATION
   → Patch critical vulns within 72 hours
   → High vulns within 30 days
   → Medium vulns within 90 days

5. VERIFICATION
   → Re-scan after patching
   → Confirm remediation was successful

6. REPORTING
   → Track metrics over time
   → Report to leadership on risk reduction
```

### 10.2 Vulnerability Scanning Tools
```bash
# Nessus (gold standard commercial)
# Install from tenable.com
service nessusd start
# Access at https://localhost:8834

# OpenVAS / Greenbone (free alternative)
gvm-start
# Access web UI at https://127.0.0.1:9392

# Qualys (cloud-based, enterprise)
# Deploy lightweight agent on all assets

# Nuclei (fast, template-based)
nuclei -u https://target.com -t cves/
nuclei -l targets.txt -t vulnerabilities/ -o results.txt

# Trivy (container/cloud scanning)
trivy image nginx:latest
trivy fs /path/to/project
```

### 10.3 Patch Management
```
Patch Tuesday (Microsoft): 2nd Tuesday of each month
- Critical patches: test + deploy within 72 hours
- Important patches: deploy within 30 days

Emergency patching (0-day):
- Assess exploitability and exposure
- Apply compensating controls (WAF rule, firewall block) immediately
- Accelerated patching timeline

Patch testing pipeline:
Dev/Test → Staging → Production (canary) → Full rollout
Never patch production directly without testing.
```

### 10.4 CVSS Scoring
```
CVSS v3.1 Score ranges:
0.1 – 3.9  → Low
4.0 – 6.9  → Medium
7.0 – 8.9  → High
9.0 – 10.0 → Critical

Key CVSS factors:
- Attack Vector: Network/Adjacent/Local/Physical
- Attack Complexity: Low/High
- Privileges Required: None/Low/High
- User Interaction: None/Required
- Scope: Unchanged/Changed
- Confidentiality/Integrity/Availability Impact: None/Low/High

Beyond CVSS — also consider:
- Is an exploit publicly available? (check ExploitDB, Metasploit)
- Is it being actively exploited in the wild? (check CISA KEV)
- Is the affected asset internet-facing?
- What data does the asset hold?
```

---

## 11. Identity & Access Management (IAM)

Most breaches involve compromised credentials. Identity is the new perimeter.

### 11.1 Core IAM Principles
```
Principle of Least Privilege:
- Every account has only the access it needs to do its job
- Nothing more, nothing less
- Review and revoke regularly

Zero Trust Identity:
- Never trust, always verify
- Verify identity for every request, every time
- Even internal traffic must authenticate

Separation of Duties:
- No single person can complete a sensitive transaction alone
- Admin accounts separate from daily-use accounts
- Break-glass accounts for emergency access
```

### 11.2 Multi-Factor Authentication (MFA)
```
MFA factors:
- Something you KNOW (password, PIN)
- Something you HAVE (phone, hardware token, smart card)
- Something you ARE (fingerprint, face, retina)

MFA types (strongest to weakest):
1. Hardware keys (FIDO2/WebAuthn) → YubiKey, Titan Key → BEST
2. Authenticator app (TOTP) → Google Auth, Microsoft Auth → GOOD
3. Push notification → Duo, Okta Verify → GOOD (beware MFA fatigue attacks)
4. SMS OTP → WEAK (SIM swapping, SS7 attacks)
5. Email OTP → WEAK (if email is compromised)

Enforce MFA on:
- All admin accounts (mandatory)
- All remote access (VPN, RDP)
- All cloud console access
- All privileged systems
- Ideally: all accounts
```

### 11.3 Privileged Access Management (PAM)
```
PAM controls for admin accounts:
- Separate admin accounts from daily-use accounts
- All admin actions logged and audited
- Just-In-Time (JIT) access: elevate only when needed
- Session recording for privileged sessions
- Password vaulting (auto-rotate service account passwords)

PAM Tools:
- CyberArk       → Enterprise PAM leader
- BeyondTrust    → Privileged remote access
- HashiCorp Vault → Secrets management (DevOps)
- Delinea        → Enterprise PAM
- KeePass/Bitwarden → Basic password management
```

### 11.4 Active Directory Hardening
```powershell
# Disable legacy authentication protocols
# Disable NTLM (use Kerberos instead)
# Enable Protected Users security group for admins
Add-ADGroupMember -Identity "Protected Users" -Members "AdminUser"

# Audit privileged group memberships
Get-ADGroupMember -Identity "Domain Admins" -Recursive

# Enable audit policies
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Account Logon" /success:enable /failure:enable

# Tiered admin model:
# Tier 0: Domain Controllers, AD, PKI
# Tier 1: Servers, applications
# Tier 2: Workstations, end users
# Admins for Tier 0 CANNOT log into Tier 2 machines
```

---

## 12. Incident Response (IR)

When something goes wrong, a well-practiced IR process separates chaos from control.

### 12.1 The IR Lifecycle (NIST SP 800-61)
```
1. PREPARATION
   → IR plan documented and tested
   → Team roles and contacts established
   → Tools and access pre-positioned
   → Tabletop exercises conducted

2. DETECTION & ANALYSIS
   → Identify the incident
   → Determine scope and severity
   → Collect and preserve evidence
   → Notify stakeholders

3. CONTAINMENT
   → Short-term: isolate affected systems
   → Long-term: apply patches, change credentials
   → Prevent further damage while preserving evidence

4. ERADICATION
   → Remove malware and attacker artifacts
   → Close the initial access vector
   → Address root cause

5. RECOVERY
   → Restore systems from clean backups
   → Validate integrity before restoring to production
   → Monitor closely for re-infection

6. POST-INCIDENT ACTIVITY
   → Lessons learned meeting
   → Update IR plan
   → Improve detection coverage
   → Report to leadership
```

### 12.2 Incident Severity Classification
| Severity | Description | Example | Response Time |
|----------|-------------|---------|---------------|
| **P1 Critical** | Major breach, data exfil, ransomware | Active ransomware spreading | 15 minutes |
| **P2 High** | Compromised admin account, malware isolated | Phishing with credential theft | 1 hour |
| **P3 Medium** | Malware on single endpoint, policy violation | Trojan on one workstation | 4 hours |
| **P4 Low** | Policy violation, suspicious but unconfirmed | Unusual login from new country | 24 hours |

### 12.3 Containment Strategies
```bash
# Network isolation (EDR or manual)
# CrowdStrike: Network Contain host from console

# Windows: Block all traffic via firewall
netsh advfirewall set allprofiles firewallpolicy blockinbound,blockoutbound

# Linux: Drop all traffic
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
# Allow only management access
iptables -A INPUT -s ANALYST_IP -j ACCEPT

# Disable AD account immediately
Disable-ADAccount -Identity "compromised.user"

# Reset credentials
Set-ADAccountPassword -Identity "compromised.user" -Reset

# Revoke active sessions
# Office 365:
Revoke-AzureADUserAllRefreshToken -ObjectId user@company.com
```

### 12.4 Forensic Evidence Collection
```bash
# Live response — collect volatile data FIRST (lost on reboot)
# Order of volatility (collect in this order):
# 1. CPU registers, cache
# 2. Running processes
# 3. Network connections
# 4. Memory (RAM)
# 5. Disk

# Windows live response
tasklist /v > processes.txt
netstat -ano > connections.txt
ipconfig /all > network.txt
wmic process list full > full_processes.txt
wevtutil qe Security /rd:true /f:text > security_log.txt

# Memory acquisition
winpmem_3.3.exe memory.raw    # Windows memory dump
avml /proc/kcore memory.lime  # Linux memory

# Disk imaging (never work on originals)
dd if=/dev/sda of=disk.img bs=64K status=progress
# Verify integrity
sha256sum disk.img > disk.img.sha256

# Linux live response
ps auxf > processes.txt
ss -tulpn > network.txt
last -n 50 > logins.txt
cat /var/log/auth.log > auth.txt
find / -newer /tmp/timestamp -type f 2>/dev/null > recent_files.txt
```

### 12.5 Memory Forensics with Volatility
```bash
# Identify OS profile
volatility -f memory.raw imageinfo

# List processes
volatility -f memory.raw --profile=Win10x64 pslist
volatility -f memory.raw --profile=Win10x64 pstree  # Tree view

# Find hidden/injected processes
volatility -f memory.raw --profile=Win10x64 psscan   # Scan for EPROCESS
volatility -f memory.raw --profile=Win10x64 malfind  # Find injected code

# Network connections
volatility -f memory.raw --profile=Win10x64 netscan

# Extract processes
volatility -f memory.raw --profile=Win10x64 procdump -p 1234 -D ./output/

# Registry hives
volatility -f memory.raw --profile=Win10x64 hivelist
volatility -f memory.raw --profile=Win10x64 printkey -K "SOFTWARE\Microsoft\Windows\CurrentVersion\Run"

# Extract credentials from memory
volatility -f memory.raw --profile=Win10x64 hashdump
volatility -f memory.raw --profile=Win10x64 mimikatz
```

---

## 13. Digital Forensics

### 13.1 Forensic Principles
```
1. Don't alter the original evidence
   → Work on copies/images only
   → Use write blockers for disk imaging

2. Document everything
   → Chain of custody
   → Every action taken, timestamps, tools used

3. Verify integrity
   → Hash everything before and after
   → SHA256 preferred over MD5

4. Be methodical
   → Follow a consistent process
   → Don't jump to conclusions
```

### 13.2 Disk Forensics
```bash
# Image a disk (forensically sound)
dcfldd if=/dev/sdb of=evidence.img hash=sha256 hashlog=evidence.hash

# Mount image read-only for analysis
mkdir /mnt/evidence
mount -o ro,loop evidence.img /mnt/evidence

# File recovery (recover deleted files)
foremost -i evidence.img -o recovered/
photorec evidence.img   # GUI-based recovery

# Autopsy (GUI forensic tool)
# Open evidence.img → Timeline analysis, keyword search, file recovery

# Artifact locations (Windows):
%AppData%\Microsoft\Windows\Recent\   → Recent files
%LocalAppData%\Temp\                  → Temp files
C:\$Recycle.Bin\                      → Deleted files
C:\Windows\Prefetch\                  → Execution evidence
C:\Windows\System32\winevt\Logs\      → Event logs
NTUSER.DAT                            → User registry hive
```

### 13.3 Windows Artifacts for Forensics
| Artifact | Location | What It Tells You |
|----------|----------|-------------------|
| **Prefetch** | C:\Windows\Prefetch\ | Program was executed |
| **LNK files** | AppData\Roaming\Microsoft\Windows\Recent | Files accessed |
| **MFT** | $MFT (NTFS) | File creation/modification timeline |
| **Amcache.hve** | C:\Windows\AppCompat\ | Program execution history |
| **ShimCache** | SYSTEM registry | Executable metadata |
| **Browser history** | AppData\Local\[browser] | Web activity |
| **Jump Lists** | AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations | Recently opened files |
| **Event Logs** | C:\Windows\System32\winevt\Logs\ | System/security events |
| **Recycle Bin** | C:\$Recycle.Bin\ | Deleted files |
| **Volume Shadow Copies** | System-managed | Historical file versions |

---

## 14. Threat Hunting

Don't wait for alerts. Go find the attacker proactively.

### 14.1 Threat Hunting Methodology
```
HUNT CYCLE:

1. HYPOTHESIS
   → Based on threat intel, ATT&CK, or gut instinct
   → Example: "APT groups targeting our sector use Cobalt Strike"
   → Example: "We may have undetected lateral movement via WMI"

2. INVESTIGATION
   → Search logs for evidence supporting or refuting hypothesis
   → Look for TTPs, not just IOCs
   → Hunt across multiple data sources

3. DISCOVERY
   → Found something? Document it fully
   → Is it a confirmed threat or unknown-benign?
   → Escalate to IR if confirmed malicious

4. IMPROVEMENT
   → Whether you found something or not, improve detections
   → Create/tune SIEM rules based on hunt findings
   → Document hunt playbook for repeatability
```

### 14.2 Hunting Hypotheses Examples
```
Based on ATT&CK T1053 (Scheduled Tasks):
"Attackers may have created scheduled tasks for persistence"
Hunt: Search for task creation events (EID 4698) outside business hours
      with unusual task names or paths pointing to user-writable directories

Based on ATT&CK T1071 (Application Layer Protocol - C2):
"Infected endpoints may be beaconing to C2 over HTTPS"
Hunt: Find hosts making regular, periodic connections to the same external IP
      Look for JA3/JA3S fingerprints matching known C2 tools

Based on ATT&CK T1003 (Credential Dumping):
"Attackers may have dumped LSASS"
Hunt: Find processes that opened a handle to lsass.exe with PROCESS_VM_READ
      Search for procdump.exe, comsvcs.dll MiniDump usage
```

### 14.3 Hunting with Data
```
Splunk threat hunt queries:

# Detect beaconing (periodic connections)
index=network sourcetype=firewall_logs
| stats count, avg(bytes_out), stdev(bytes_out) by src_ip, dest_ip
| where stdev(bytes_out) < 100 AND count > 100
| sort -count

# Find processes connecting to uncommon external IPs
index=sysmon EventCode=3 NOT dest_ip IN ("10.*","172.16.*","192.168.*")
| stats count by host, Image, dest_ip, dest_port
| where count > 10

# Detect PowerShell downloading from internet
index=sysmon EventCode=1 
(CommandLine="*Net.WebClient*" OR CommandLine="*DownloadString*" OR CommandLine="*WebRequest*")
| table _time, host, user, CommandLine

# Find DLL injection (CreateRemoteThread)
index=sysmon EventCode=8
| table _time, host, SourceImage, TargetImage, StartAddress
```

---

## 15. Hardening Systems

Reduce your attack surface before attackers find it.

### 15.1 Windows Hardening
```powershell
# Enable Windows Defender and keep updated
Set-MpPreference -DisableRealtimeMonitoring $false

# Enable Attack Surface Reduction (ASR) rules
# Block Office macros from spawning child processes
Add-MpPreference -AttackSurfaceReductionRules_Ids D4F940AB-401B-4EFC-AADC-AD5F3C50688A -AttackSurfaceReductionRules_Actions Enabled

# Disable unnecessary services
Stop-Service -Name "Spooler" -Force; Set-Service -Name "Spooler" -StartupType Disabled
Stop-Service -Name "RemoteRegistry" -Force; Set-Service -Name "RemoteRegistry" -StartupType Disabled

# Enable PowerShell logging
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Force
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Name "EnableScriptBlockLogging" -Value 1

# Disable SMBv1 (EternalBlue target)
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol

# Enable Windows Firewall on all profiles
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True

# Credential Guard (prevents credential theft)
# Enable via Group Policy or:
reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard" /v "EnableVirtualizationBasedSecurity" /t REG_DWORD /d 1 /f
```

### 15.2 Linux Hardening
```bash
# Update everything
apt update && apt upgrade -y

# Configure automatic security updates
apt install unattended-upgrades
dpkg-reconfigure unattended-upgrades

# Disable unused services
systemctl disable avahi-daemon
systemctl disable cups
systemctl disable bluetooth

# SSH hardening (/etc/ssh/sshd_config)
PermitRootLogin no
PasswordAuthentication no          # Key-based auth only
PubkeyAuthentication yes
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers specificuser             # Restrict who can SSH

# Enable firewall (UFW)
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw enable

# Configure auditd
auditctl -w /etc/passwd -p wa -k passwd_changes
auditctl -w /etc/shadow -p wa -k shadow_changes
auditctl -w /var/log/ -p wa -k log_changes
auditctl -a exit,always -F arch=b64 -S execve -k exec_commands

# Restrict SUID binaries
find / -perm -4000 -type f 2>/dev/null | xargs ls -la
# Remove SUID from unnecessary binaries
chmod u-s /usr/bin/unnecessary_binary

# Fail2ban (ban repeated failed logins)
apt install fail2ban
systemctl enable fail2ban --now
```

### 15.3 Network Hardening
```
Firewall best practices:
- Default DENY on all inbound and outbound
- Whitelist only required traffic
- Separate rules per VLAN/zone
- Log all denied traffic
- Review rules quarterly; remove stale rules

DNS Hardening:
- Use DNS over HTTPS (DoH) or DNS over TLS (DoT)
- Block known malicious domains at DNS resolver
- Implement DNSSEC
- Use internal DNS; don't expose resolvers externally

Email Security:
- SPF     → Authorize which servers can send on your behalf
- DKIM    → Cryptographic signature on emails
- DMARC   → Policy for SPF/DKIM failures (reject/quarantine)
- Configure: v=spf1 include:your-provider ~all
- Enable anti-phishing and sandboxing on email gateway

Web Proxy:
- Route all HTTP/S through proxy
- Block uncategorized and malicious domains
- SSL inspection (decrypt, inspect, re-encrypt)
- Block high-risk file types (.exe, .dll, .ps1) from download
```

### 15.4 CIS Benchmarks
Use CIS Benchmarks as your hardening baseline:
```
Available for: Windows, Linux, macOS, cloud, containers, network devices

Tools to audit against CIS:
- CIS-CAT Pro (official tool)
- OpenSCAP (open source)
- Lynis (Linux)
- Tenable/Nessus (plugin)

Example CIS Control 6 — Access Control Management:
- Maintain inventory of accounts
- Use unique credentials per account
- Disable dormant accounts (inactive > 45 days)
- Restrict admin privileges
```

---

## 16. Cloud Security

Secure your cloud like your on-premises environment — it's not someone else's problem.

### 16.1 Shared Responsibility Model
```
AWS/Azure/GCP is responsible for:
- Physical security of data centers
- Hypervisor and host infrastructure
- Managed service availability

YOU are responsible for:
- Your data
- Your application code
- Your IAM configurations
- Your network controls (security groups, VPCs)
- Encryption of data at rest and in transit
- Monitoring and logging
```

### 16.2 AWS Security Hardening
```bash
# Enable CloudTrail (audit all API calls — MANDATORY)
aws cloudtrail create-trail --name main-trail --s3-bucket-name audit-logs

# Enable Config (track configuration changes)
aws configservice put-configuration-recorder ...

# Enable GuardDuty (threat detection)
aws guardduty create-detector --enable

# Enable Security Hub (aggregated findings)
aws securityhub enable-security-hub

# S3 bucket security
# Block all public access by default
aws s3api put-public-access-block --bucket mybucket \
  --public-access-block-configuration "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"

# Enable S3 server-side encryption
aws s3api put-bucket-encryption ...

# IAM best practices
# Enable MFA for all users
# Use IAM roles instead of access keys
# Rotate access keys regularly
aws iam generate-credential-report
aws iam get-credential-report

# Check for public-facing resources
aws ec2 describe-security-groups --filters "Name=ip-permission.cidr,Values=0.0.0.0/0"
```

### 16.3 Cloud Security Tools
| Tool | Purpose |
|------|---------|
| **Prowler** | AWS/Azure/GCP security assessment |
| **ScoutSuite** | Multi-cloud security auditing |
| **CloudMapper** | AWS environment visualization |
| **Pacu** | AWS exploitation (offensive) |
| **Trivy** | Container/IaC vulnerability scanning |
| **Checkov** | IaC security scanning (Terraform, etc.) |
| **AWS Security Hub** | Centralized cloud security findings |
| **Azure Defender** | Azure native threat protection |

---

## 17. Application Security (AppSec)

Shift security left — fix vulnerabilities during development, not after deployment.

### 17.1 Secure Development Lifecycle (SDL)
```
REQUIREMENTS → DESIGN → DEVELOPMENT → TESTING → DEPLOYMENT → MAINTENANCE

Security activities at each stage:
Requirements: Threat modeling, abuse cases, security requirements
Design:        Architecture review, data flow diagrams, attack surface analysis
Development:   Secure coding standards, IDE security plugins, peer review
Testing:       SAST, DAST, SCA, penetration testing
Deployment:    Configuration review, secrets management, hardened images
Maintenance:   Vulnerability management, patch management, monitoring
```

### 17.2 Static Application Security Testing (SAST)
```bash
# Semgrep (open source, polyglot)
semgrep --config=auto ./src

# Bandit (Python)
bandit -r ./python_project/

# ESLint security plugin (JavaScript)
npm install eslint-plugin-security
eslint --plugin security ./src

# SonarQube (enterprise)
# Integrates into CI/CD pipeline
# Tracks code quality + security over time

# CodeQL (GitHub)
# .github/workflows/codeql.yml
```

### 17.3 Software Composition Analysis (SCA)
```bash
# Check for vulnerable dependencies

# OWASP Dependency-Check
dependency-check.sh --project myapp --scan ./

# Snyk
snyk test
snyk monitor

# npm audit
npm audit
npm audit fix

# pip-audit (Python)
pip-audit

# Trivy (containers, IaC)
trivy fs ./
```

### 17.4 Secrets Management
```
Never store secrets in code. Use:
- HashiCorp Vault (dynamic secrets, encryption as a service)
- AWS Secrets Manager / Parameter Store
- Azure Key Vault
- GitHub Secrets / GitLab CI Variables

Detect secrets already in code:
- git-secrets (pre-commit hook)
- TruffleHog (scan git history)
- Gitleaks (fast secret scanning)

trufflehog git file:///path/to/repo
gitleaks detect --source . -v
```

---

## 18. Security Awareness & Human Defense

Technology can't fix humans. Train them instead.

### 18.1 Security Awareness Program
```
Training components:
- Annual security awareness training (mandatory)
- Phishing simulation campaigns (monthly)
- Role-specific training (finance, HR, IT)
- New employee onboarding security module
- Regular security newsletters / advisories

Key topics to cover:
- Phishing recognition
- Password hygiene and MFA
- Social engineering tactics
- Incident reporting procedures
- Clean desk / clear screen policy
- Safe web browsing
- Mobile device security
- Data classification and handling
```

### 18.2 Phishing Simulation
```
Tools:
- GoPhish (open source)
- KnowBe4 (commercial)
- Proofpoint Security Awareness
- Cofense (commercial)

GoPhish quick setup:
# Start GoPhish
./gophish

# Configure:
1. Sending profile (SMTP relay)
2. Landing page (clone real login page)
3. Email template (lure)
4. Campaign (target group + schedule)

Measure:
- Open rate
- Click rate
- Credential submission rate
- Report rate (did they report it?)

Focus on improving report rate, not just reducing click rate.
```

---

## 19. Compliance & Governance

Security needs to align with business and legal requirements.

### 19.1 Key Regulations & Standards
| Standard | Who It Applies To | Key Requirements |
|----------|------------------|-----------------|
| **GDPR** | EU data processors/controllers | Data protection, breach notification (72hr) |
| **PCI DSS** | Cardholder data environments | 12 requirements, quarterly scans |
| **HIPAA** | US healthcare | PHI protection, audit controls |
| **SOC 2** | SaaS/service companies | Trust service criteria audit |
| **ISO 27001** | Any organization | ISMS implementation and audit |
| **NIST CSF** | US government + voluntary | Risk-based framework |
| **CIS Controls** | Any organization | 18 prioritized controls |

### 19.2 Risk Management
```
Risk = Likelihood × Impact

Risk treatment options:
- ACCEPT    → Acknowledge risk, document decision
- MITIGATE  → Apply controls to reduce likelihood or impact
- TRANSFER  → Cyber insurance, third-party contracts
- AVOID     → Stop the risky activity entirely

Risk register:
| Risk | Likelihood | Impact | Score | Owner | Treatment | Status |
|------|-----------|--------|-------|-------|-----------|--------|
| Ransomware | High | Critical | 20 | CISO | Mitigate (EDR+backups) | In progress |
```

### 19.3 Security Metrics for Leadership
```
Executive metrics that matter:
- Mean Time to Detect (MTTD)
- Mean Time to Respond (MTTR)
- % of critical vulnerabilities patched within SLA
- Phishing click rate trend
- Number of confirmed incidents per quarter
- Security coverage (% of endpoints with EDR)
- MFA adoption rate
- Third-party risk assessment completion rate

Avoid vanity metrics:
- "We blocked X threats" (doesn't convey risk)
- Total alerts (noise, not signal)
```

---

## 20. Building a Home Security Lab

Practice defending in your own environment.

### 20.1 Lab Architecture
```
Recommended setup:

Host machine (your PC/Mac):
└── Hypervisor (VMware Workstation, VirtualBox, Proxmox)
    ├── Attacker VM (Kali Linux)
    ├── Windows Server 2022 (Domain Controller)
    ├── Windows 10/11 (Domain-joined workstation)
    ├── Ubuntu Server (Linux target)
    └── Security Stack VM:
        ├── Wazuh SIEM (log aggregation + HIDS)
        ├── TheHive (incident management)
        ├── Zeek (network analysis)
        └── Graylog / ELK (log visualization)

Network:
- Host-only network: isolated lab traffic
- Management network: access your VMs
- Simulate: attacker → compromise Windows workstation → lateral movement to DC
- Then: detect in SIEM, hunt in logs, respond
```

### 20.2 Free Tools for Your Lab
| Tool | Purpose | Install |
|------|---------|---------|
| **Wazuh** | SIEM + HIDS | wazuh.com |
| **TheHive** | Incident management | thehive.strangebee.com |
| **MISP** | Threat intel platform | misp-project.org |
| **Zeek** | Network analysis | zeek.org |
| **Velociraptor** | DFIR + threat hunting | github.com/Velocidex |
| **Graylog** | Log management | graylog.org |
| **Shuffle** | SOAR automation | shuffler.io |

### 20.3 Practice Scenarios
```
Scenario 1 — Malware Infection:
1. Run malware sample on Windows VM (use theZoo or MalwareBazaar)
2. Detect via SIEM / EDR
3. Perform live response
4. Memory and disk forensics
5. Write incident report

Scenario 2 — Phishing + Credential Theft:
1. Send phishing email with GoPhish
2. Victim clicks, credentials captured
3. Attacker uses credentials to log in
4. Detect via failed/successful login analysis
5. Respond and contain

Scenario 3 — Ransomware:
1. Simulate ransomware execution (safe simulator)
2. Detect file encryption activity
3. Isolate host
4. Restore from backup
5. Identify patient zero and initial access
```

---

## 21. Tools Arsenal — Blue Team Edition

### 21.1 Essential Tools by Category

**SIEM & Log Management**
| Tool | Purpose |
|------|---------|
| Splunk | Enterprise SIEM leader |
| Microsoft Sentinel | Cloud-native SIEM |
| Elastic SIEM | Open-source option |
| Wazuh | Free SIEM + HIDS |
| Graylog | Log management |

**Endpoint Security**
| Tool | Purpose |
|------|---------|
| CrowdStrike Falcon | EDR leader |
| Microsoft Defender for Endpoint | Windows native EDR |
| SentinelOne | Autonomous EDR |
| Sysmon | Enhanced Windows logging (free) |
| Velociraptor | DFIR and threat hunting |

**Network Security**
| Tool | Purpose |
|------|---------|
| Zeek | Network analysis framework |
| Suricata | IDS/IPS + NSM |
| Wireshark/tcpdump | Packet analysis |
| Brim | Fast Zeek log analysis |
| NetworkMiner | Network forensics |

**Vulnerability Management**
| Tool | Purpose |
|------|---------|
| Nessus | Industry-standard scanner |
| OpenVAS | Free alternative |
| Qualys | Cloud-based scanning |
| Trivy | Container scanning |
| Nuclei | Fast template-based scanning |

**Forensics & IR**
| Tool | Purpose |
|------|---------|
| Autopsy | GUI forensic analysis |
| Volatility | Memory forensics |
| KAPE | Windows artifact collection |
| FTK Imager | Disk imaging |
| CyberChef | Data encoding/decoding |
| IRIS | Incident response platform |

**Threat Intel**
| Tool | Purpose |
|------|---------|
| MISP | Threat intel platform |
| OpenCTI | CTI management |
| VirusTotal | File/URL/IP reputation |
| Cortex | Automated observable analysis |
| AlienVault OTX | Community threat feeds |

---

## 22. Certifications Roadmap — Blue Team

### 22.1 Certification Path by Level

**Entry Level**
| Cert | Org | Focus |
|------|-----|-------|
| CompTIA Security+ | CompTIA | Security fundamentals |
| CompTIA Network+ | CompTIA | Networking |
| CompTIA CySA+ | CompTIA | Security analyst skills |
| AWS Security Specialty | AWS | Cloud security |

**Intermediate**
| Cert | Org | Focus |
|------|-----|-------|
| **BTL1** (Blue Team Labs Level 1) | Security Blue Team | Hands-on SOC skills |
| **eWPT** | eLearnSecurity | Web app defense |
| GCIH | GIAC | Incident handling |
| GCFE | GIAC | Forensic examiner |
| GCFA | GIAC | Advanced forensics |

**Advanced**
| Cert | Org | Focus |
|------|-----|-------|
| **GCIA** | GIAC | Intrusion analysis |
| **GCTI** | GIAC | Cyber threat intelligence |
| GNFA | GIAC | Network forensics |
| CISSP | ISC2 | Security management |
| CISM | ISACA | Security management |

### 22.2 Recommended Learning Order
```
1. CompTIA Security+ (baseline)
2. BTL1 (hands-on SOC skills)
3. CompTIA CySA+ (analyst skills)
4. GCIH (incident response)
5. GCFA / GCFE (forensics specialization)
6. CISSP (if moving into management)
```

---

## 23. Career Paths — Blue Team

### 23.1 Blue Team Roles
| Role | Avg Salary (US) | Description |
|------|----------------|-------------|
| **SOC Analyst L1** | $50K–$70K | Alert triage and escalation |
| **SOC Analyst L2** | $70K–$95K | Incident investigation |
| **SOC Analyst L3 / Threat Hunter** | $95K–$130K | Proactive hunting, advanced IR |
| **Incident Responder** | $90K–$140K | Lead incident investigations |
| **Digital Forensics Analyst** | $85K–$130K | Evidence collection and analysis |
| **Threat Intelligence Analyst** | $90K–$140K | Track actors, report on threats |
| **Security Engineer** | $110K–$160K | Build and maintain security tooling |
| **Detection Engineer** | $100K–$150K | Write and tune detection rules |
| **Security Architect** | $140K–$200K | Design security infrastructure |
| **CISO** | $180K–$400K+ | Executive security leadership |

### 23.2 Building Your Portfolio
- **GitHub** — Host detection rules, scripts, SIEM queries
- **Blog** — Document incident analyses, tool tutorials, CTF writeups
- **TryHackMe / CyberDefenders** — Build your blue team profile
- **CTF competitions** — Blue team CTFs (BOTS, CyberDefenders challenges)
- **Home lab** — Document your setup and practice scenarios
- **LinkedIn** — Share your learning journey and connect

---

## 24. Labs & Practice Platforms — Blue Team

| Platform | Focus | Cost |
|----------|-------|------|
| **TryHackMe** (Blue Team paths) | SOC, DFIR, threat hunting | Free + Premium |
| **CyberDefenders** | Blue team CTF challenges | Free + paid |
| **Blue Team Labs Online** | Hands-on blue team skills | Free + paid |
| **LetsDefend** | SOC analyst simulation | Free + paid |
| **AttackIQ Academy** | Purple team, adversary simulation | Free |
| **SANS Holiday Hack** | Annual challenge, forensics heavy | Free |
| **Boss of the SOC (BOTS)** | Splunk-based investigation CTF | Free |
| **Splunk Training** | SIEM fundamentals | Free tier |
| **Blue Team Village CTF** | DEF CON blue team challenges | Free |

---

## 25. Resources & Communities

### 25.1 Must-Read Books
| Book | Author | Focus |
|------|--------|-------|
| *The Practice of Network Security Monitoring* | Richard Bejtlich | NSM fundamentals |
| *Applied Incident Response* | Steve Anson | IR handbook |
| *Intelligence-Driven Incident Response* | Rebekah Brown & Scott Roberts | Threat-intel-driven IR |
| *The Art of Memory Forensics* | Ligh, Case, Levy, Walters | Memory forensics |
| *Blue Team Handbook* | Don Murdoch | SOC reference |
| *Crafting the InfoSec Playbook* | Jeff Bollinger et al. | NSM and IR |
| *Computer Forensics: Incident Response Essentials* | Warren Kruse | DFIR fundamentals |

### 25.2 Essential Websites
```
Learning:
- tryhackme.com/paths (blue team paths)
- cyberdefenders.org         (blue team CTFs)
- letsdefend.io              (SOC simulation)
- attackiq.com/academy       (purple team)
- splunk.com/en_us/training  (SIEM training)

References:
- attack.mitre.org            MITRE ATT&CK
- d3fend.mitre.org            MITRE D3FEND (defensive counterpart)
- car.mitre.org               Cyber Analytics Repository
- lolbas-project.github.io    LOLBins reference
- sigma.socprime.com          SIGMA detection rules
- github.com/SigmaHQ/sigma   SIGMA rule repository
- uncoder.io                  Translate SIGMA to any SIEM

Threat Intel:
- any.run                     Interactive malware sandbox
- hybrid-analysis.com         Free malware analysis
- bazaar.abuse.ch             Malware samples
- urlhaus.abuse.ch            Malicious URL feeds
- threatfox.abuse.ch          IOC feeds
- virustotal.com              File/URL/IP reputation
```

### 25.3 YouTube Channels
| Channel | Focus |
|---------|-------|
| **Gerald Auger (Simply Cyber)** | Blue team career, SOC skills |
| **13Cubed** | DFIR, memory forensics |
| **John Hammond** | Malware analysis, CTF |
| **HuskyHacks** | Threat hunting, blue team |
| **SANS Institute** | Webinars, forensics |
| **Splunk** | SIEM tutorials |
| **MITRE ATT&CK** | Framework walkthrough |

### 25.4 Communities
- **Discord**: TryHackMe, CyberDefenders, Blue Team Village, SANS Community
- **Reddit**: r/blueteamsec, r/netsec, r/cybersecurity, r/AskNetsec
- **Twitter/X**: Follow #dfir, #threathunting, #blueteam, #soc
- **DEF CON Blue Team Village**: Annual CTF and talks
- **FIRST.org**: Global incident response community
- **Slack**: SANS Community, Bloodhound Gang

---

## Quick Reference: Defender's Cheat Sheet

### Incident Triage Checklist
```
□ Identify affected system(s) — hostname, IP, user
□ Determine incident type — malware, phishing, data breach, etc.
□ Assess severity — P1/P2/P3/P4
□ Notify relevant stakeholders per escalation matrix
□ Isolate affected system(s) if necessary
□ Collect volatile evidence (memory, running processes, connections)
□ Preserve logs — pull and store before rotation
□ Document timeline of events
□ Identify initial access vector (how did attacker get in?)
□ Identify all affected systems (lateral movement?)
□ Eradicate attacker artifacts
□ Restore from clean backup
□ Verify integrity before returning to production
□ Conduct lessons learned within 72 hours
```

### First 5 Things to Check on a Suspicious Windows Host
```powershell
# 1. Who is logged in and what's their privilege level?
query session
whoami /groups

# 2. What processes are running and where are they from?
Get-Process | Select-Object Name, Id, Path | Sort-Object Name
wmic process get name,executablepath,commandline

# 3. What network connections are active?
netstat -ano
Get-NetTCPConnection | Where-Object {$_.State -eq "Established"}

# 4. What's scheduled to run?
schtasks /query /fo LIST /v
Get-ScheduledTask | Where-Object {$_.State -ne "Disabled"}

# 5. What was recently created or modified?
Get-ChildItem C:\Users -Recurse -File | Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(-1)}
```

### Common IOC Patterns to Hunt
```
Malware persistence locations:
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup

Suspicious process parents:
winword.exe → cmd.exe          (macro attack)
excel.exe → powershell.exe     (macro attack)
outlook.exe → wscript.exe      (email attachment)
iexplore.exe → cmd.exe         (browser exploit)
msiexec.exe → powershell.exe   (MSI-based malware)

C2 beacon patterns:
- Regular connections every 60s, 120s, 300s
- User-Agent strings not matching browser fingerprint
- HTTP POST to long, random-looking URLs
- HTTPS to IP addresses (not domain names)
- Encoded/encrypted POST body with consistent size
```

### Log Analysis One-Liners
```bash
# Failed SSH logins by IP
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr

# Successful logins
grep "Accepted" /var/log/auth.log | awk '{print $9, $11}' | sort | uniq -c | sort -nr

# Web server 404s (scanning activity)
awk '$9==404 {print $1, $7}' /var/log/apache2/access.log | sort | uniq -c | sort -nr

# Large file transfers (potential exfiltration)
awk '{if($10 > 1000000) print $1, $7, $10}' /var/log/apache2/access.log

# PowerShell download (Windows event log via PowerShell)
Get-WinEvent -LogName 'Microsoft-Windows-PowerShell/Operational' |
Where-Object {$_.Message -match "DownloadString|WebClient|Invoke-Expression"} |
Select-Object TimeCreated, Message
```

---

> 🔵 **Remember:** Every system you protect holds someone's data, privacy, or livelihood. The defender's work is quiet, often invisible — but it matters enormously.
>
> The best defenders never stop learning. The threat landscape evolves daily, and so should you. Stay curious. Stay vigilant. Defend relentlessly. 🛡️

---

*Last Updated: 2026 | Part of the [Cybersecurity-Basics](https://github.com/yourusername/Cybersecurity-Basics) repository*
*Companion guide: [Become-a-Hacker.md](./Become-a-Hacker.md)*