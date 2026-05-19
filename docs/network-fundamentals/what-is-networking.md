# What is Networking?

> A comprehensive guide to understanding computer networking — from the basics to advanced concepts.

---

## 1. Introduction to Networking

A **computer network** is a collection of devices (computers, servers, phones, printers) connected together so they can communicate and share resources.

Networking is the backbone of the internet, cloud computing, and every digital service we use daily. Understanding how networks work is foundational to:
- IT and system administration
- Cybersecurity (you can't defend what you don't understand)
- Software development (APIs, web apps, distributed systems)
- Cloud computing and DevOps

### Why Networking Matters
- Every website visit, email, and video call relies on networking
- Security attacks almost always involve the network
- Misconfigured networks are one of the most common attack vectors
- Network knowledge is required for most IT and security certifications

### Key Concepts at a Glance
| Concept | What it means |
|---------|--------------|
| **IP Address** | Unique logical address for a device on a network |
| **MAC Address** | Unique physical address burned into a network card |
| **Protocol** | Set of rules for how devices communicate |
| **Packet** | Unit of data transmitted over a network |
| **Bandwidth** | Maximum data transfer capacity of a connection |
| **Latency** | Delay between sending and receiving data |
| **Router** | Device that forwards packets between networks |
| **Switch** | Device that connects devices within the same network |

---

## 2. Types of Networks

### By Geographic Scale

| Type | Full Name | Scale | Example |
|------|-----------|-------|---------|
| **PAN** | Personal Area Network | ~10 meters | Bluetooth devices around a person |
| **LAN** | Local Area Network | Building/campus | Home network, office network |
| **MAN** | Metropolitan Area Network | City-wide | City Wi-Fi, cable TV network |
| **WAN** | Wide Area Network | Country/global | The Internet, corporate WAN |

### By Architecture

**Client-Server Network**
- Centralized model — servers provide resources, clients consume them
- Examples: web servers, email servers, file servers
- Easier to manage and secure at scale

**Peer-to-Peer (P2P) Network**
- All devices are equal — each can act as client and server
- Examples: BitTorrent, early file-sharing networks
- Harder to secure, but decentralized

### By Medium

**Wired (Ethernet)**
- Uses physical cables (Cat5e, Cat6, Cat6a, fibre optic)
- More reliable, faster, harder to intercept
- Common in offices and data centres

**Wireless (Wi-Fi)**
- Uses radio waves (2.4GHz, 5GHz, 6GHz bands)
- Convenient, but more susceptible to interference and interception

---

## 3. Network Topologies

Topology describes the physical or logical arrangement of devices on a network.

### Bus Topology
```
Device1 --- Device2 --- Device3 --- Device4
|___________________________________|
              Single cable (bus)
```
- All devices share one cable
- Simple, cheap — but a single cable failure breaks everything
- Largely obsolete today

### Star Topology
```
        Device1
           |
Device4 — Switch — Device2
           |
        Device3
```
- All devices connect to a central switch or hub
- Most common in modern LANs
- Central device failure affects all — but individual device failures are isolated

### Ring Topology
```
Device1 → Device2 → Device3
   ↑                    ↓
Device4 ← ← ← ← ← ← ←
```
- Data travels in one (or both) directions around the ring
- Used in some WANs and older Token Ring networks
- One break can disrupt the whole network (unless dual ring)

### Mesh Topology
```
Device1 — Device2
  | \    / |
  |  Device3 |
  | /    \ |
Device4 — Device5
```
- Every device connects to multiple others
- Highly redundant — multiple paths available
- Expensive and complex; used in critical infrastructure and the internet backbone

### Hybrid Topology
- Combination of two or more topologies
- Most real-world enterprise networks are hybrid (star-bus, star-mesh, etc.)

---

## 4. The OSI Model

The **OSI (Open Systems Interconnection) model** is a conceptual framework that standardizes how different network systems communicate. It has **7 layers**, each with a specific role.

```
┌─────────────────────────────────────────────┐
│  Layer 7 — Application                      │  ← HTTP, FTP, DNS, SMTP
├─────────────────────────────────────────────┤
│  Layer 6 — Presentation                     │  ← Encryption, Compression, SSL/TLS
├─────────────────────────────────────────────┤
│  Layer 5 — Session                          │  ← Sessions, NetBIOS, RPC
├─────────────────────────────────────────────┤
│  Layer 4 — Transport                        │  ← TCP, UDP, ports
├─────────────────────────────────────────────┤
│  Layer 3 — Network                          │  ← IP, ICMP, routing
├─────────────────────────────────────────────┤
│  Layer 2 — Data Link                        │  ← MAC addresses, Ethernet, switches
├─────────────────────────────────────────────┤
│  Layer 1 — Physical                         │  ← Cables, signals, bits
└─────────────────────────────────────────────┘
```

### Mnemonic
> **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing
> (Application, Presentation, Session, Transport, Network, Data Link, Physical)

### Layer Details

| Layer | Name | Protocol Data Unit | Key Protocols/Devices | Function |
|-------|------|-------------------|----------------------|----------|
| 7 | Application | Data | HTTP, HTTPS, FTP, DNS, SMTP, SSH | User-facing network services |
| 6 | Presentation | Data | SSL/TLS, JPEG, ASCII, encryption | Data formatting, encryption/decryption |
| 5 | Session | Data | NetBIOS, RPC, PPTP | Establishing, managing, terminating sessions |
| 4 | Transport | Segment | TCP, UDP | End-to-end delivery, error checking, ports |
| 3 | Network | Packet | IP, ICMP, ARP, routers | Logical addressing and routing |
| 2 | Data Link | Frame | Ethernet, MAC, switches, ARP | Physical addressing, error detection |
| 1 | Physical | Bits | Cables, hubs, radio waves, NICs | Raw bit transmission |

### Why the OSI Model Matters in Security
- Attacks target specific layers (e.g., ARP spoofing at Layer 2, IP spoofing at Layer 3, XSS at Layer 7)
- Security tools operate at specific layers (firewalls at L3/L4, WAFs at L7)
- Troubleshooting follows the OSI model — start at Layer 1 and work up

---

## 5. The TCP/IP Model

The **TCP/IP model** (also called the Internet model) is the practical framework the modern internet is built on. It has **4 layers** and maps roughly to the OSI model.

```
┌─────────────────────────────────┐
│  Application Layer              │  ← OSI Layers 5, 6, 7
│  (HTTP, DNS, FTP, SMTP, SSH)    │
├─────────────────────────────────┤
│  Transport Layer                │  ← OSI Layer 4
│  (TCP, UDP)                     │
├─────────────────────────────────┤
│  Internet Layer                 │  ← OSI Layer 3
│  (IP, ICMP, ARP)               │
├─────────────────────────────────┤
│  Network Access Layer           │  ← OSI Layers 1, 2
│  (Ethernet, Wi-Fi, MAC)        │
└─────────────────────────────────┘
```

### TCP vs UDP

| Feature | TCP | UDP |
|---------|-----|-----|
| Full Name | Transmission Control Protocol | User Datagram Protocol |
| Connection | Connection-oriented (3-way handshake) | Connectionless |
| Reliability | Guaranteed delivery, retransmits lost packets | No guarantee — fire and forget |
| Speed | Slower (overhead for reliability) | Faster (less overhead) |
| Order | Maintains packet order | No ordering |
| Use Cases | Web browsing, email, file transfer | Video streaming, DNS, VoIP, gaming |

### The TCP 3-Way Handshake
```
Client              Server
  |                   |
  |──── SYN ─────────►|   "I want to connect"
  |                   |
  |◄─── SYN-ACK ──────|   "OK, I acknowledge"
  |                   |
  |──── ACK ─────────►|   "Great, connection established"
  |                   |
  |═══ Data flows ════|
```

> **Security note:** SYN flood attacks exploit this handshake by sending many SYN packets without completing them, exhausting server resources.

---

## 6. IP Addressing

Every device on a network needs an **IP address** — a logical address used to identify and locate it.

### IPv4

- 32-bit address written in **dotted decimal** notation
- Format: `X.X.X.X` where each X is 0–255
- Example: `192.168.1.105`
- Provides ~4.3 billion unique addresses (nearly exhausted)

#### IPv4 Address Classes (Classful — legacy, but still tested)
| Class | Range | Default Subnet | Use |
|-------|-------|---------------|-----|
| A | 1.0.0.0 – 126.255.255.255 | /8 (255.0.0.0) | Large networks |
| B | 128.0.0.0 – 191.255.255.255 | /16 (255.255.0.0) | Medium networks |
| C | 192.0.0.0 – 223.255.255.255 | /24 (255.255.255.0) | Small networks |
| D | 224.0.0.0 – 239.255.255.255 | N/A | Multicast |
| E | 240.0.0.0 – 255.255.255.255 | N/A | Reserved/Experimental |

#### Private IP Ranges (RFC 1918 — not routable on the internet)
| Range | CIDR | Common Use |
|-------|------|-----------|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | Large corporate networks |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | Medium networks |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | Home/small office networks |

#### Special Addresses
| Address | Purpose |
|---------|---------|
| `127.0.0.1` | Loopback — refers to "this machine" |
| `0.0.0.0` | Default route / unspecified |
| `255.255.255.255` | Broadcast to all hosts on local network |
| `169.254.x.x` | APIPA — auto-assigned when DHCP fails |

### IPv6

- 128-bit address written in **hexadecimal** with colons
- Example: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
- Shortened: `2001:db8:85a3::8a2e:370:7334`
- Provides 340 undecillion (3.4 × 10³⁸) addresses
- Built-in features: IPSec support, no NAT needed, auto-configuration

---

## 7. Subnetting

Subnetting divides a large network into smaller, more manageable sub-networks (subnets).

### Why Subnet?
- Reduces broadcast traffic
- Improves security (isolates network segments)
- More efficient IP address allocation
- Enables routing between segments

### CIDR Notation
CIDR (Classless Inter-Domain Routing) uses a `/prefix` to denote the subnet mask.

```
192.168.1.0/24
            └── 24 bits for network, 8 bits for hosts
                Subnet mask: 255.255.255.0
                Hosts available: 2^8 - 2 = 254
```

### Subnet Cheat Sheet
| CIDR | Subnet Mask | # of Hosts | # of Subnets (from /24) |
|------|-------------|-----------|------------------------|
| /24 | 255.255.255.0 | 254 | 1 |
| /25 | 255.255.255.128 | 126 | 2 |
| /26 | 255.255.255.192 | 62 | 4 |
| /27 | 255.255.255.224 | 30 | 8 |
| /28 | 255.255.255.240 | 14 | 16 |
| /29 | 255.255.255.248 | 6 | 32 |
| /30 | 255.255.255.252 | 2 | 64 |
| /32 | 255.255.255.255 | 1 (host route) | — |

### Subnet Components
For any subnet (e.g., `192.168.1.0/24`):
- **Network address** — `192.168.1.0` (first address, not assignable)
- **Broadcast address** — `192.168.1.255` (last address, not assignable)
- **Usable hosts** — `192.168.1.1` to `192.168.1.254`
- **Gateway** — typically `192.168.1.1` (first usable, by convention)

---

## 8. MAC Addresses

A **MAC (Media Access Control) address** is a unique physical identifier assigned to a network interface card (NIC).

- 48-bit address written in hexadecimal
- Format: `AA:BB:CC:DD:EE:FF` or `AA-BB-CC-DD-EE-FF`
- First 3 bytes = **OUI (Organizationally Unique Identifier)** — identifies the manufacturer
- Last 3 bytes = device-specific identifier

### Example
```
00:1A:2B:3C:4D:5E
└──────┘ └──────┘
  OUI     Device ID
(manufacturer)
```

### MAC vs IP
| | MAC Address | IP Address |
|--|------------|-----------|
| Layer | Layer 2 (Data Link) | Layer 3 (Network) |
| Scope | Local network only | Global (internet) |
| Assigned by | Manufacturer | Network admin / DHCP |
| Changes? | Mostly static (can be spoofed) | Changes with network |
| Format | 48-bit hex | 32-bit decimal (IPv4) |

### ARP — Address Resolution Protocol
ARP resolves IP addresses to MAC addresses on a local network.

```
Device A wants to send to 192.168.1.5:
  1. A broadcasts: "Who has 192.168.1.5?"
  2. Device B replies: "I have it — my MAC is 00:1A:2B:..."
  3. A caches the MAC and sends the frame
```

> **Security note:** ARP has no authentication — ARP spoofing/poisoning allows a man-in-the-middle attack by associating an attacker's MAC with a legitimate IP.

---

## 9. Protocols

A **protocol** is a set of rules that defines how data is transmitted and received across a network.

### Application Layer Protocols

| Protocol | Port | Purpose |
|----------|------|---------|
| **HTTP** | 80 | Web traffic (unencrypted) |
| **HTTPS** | 443 | Web traffic (encrypted with TLS) |
| **FTP** | 20/21 | File transfer (unencrypted) |
| **SFTP** | 22 | Secure file transfer (over SSH) |
| **SSH** | 22 | Secure remote shell access |
| **Telnet** | 23 | Remote shell (unencrypted — avoid) |
| **SMTP** | 25 / 587 | Sending email |
| **POP3** | 110 | Receiving email (downloads) |
| **IMAP** | 143 | Receiving email (syncs) |
| **DNS** | 53 | Domain name resolution |
| **DHCP** | 67/68 | Automatic IP assignment |
| **SNMP** | 161/162 | Network device management |
| **RDP** | 3389 | Windows remote desktop |
| **SMB** | 445 | Windows file sharing |
| **NTP** | 123 | Time synchronization |
| **LDAP** | 389 | Directory services |
| **LDAPS** | 636 | LDAP over TLS |

### Transport Layer Protocols
- **TCP** — Reliable, ordered, connection-oriented
- **UDP** — Fast, connectionless, no guarantee

### Network Layer Protocols
- **IP (IPv4/IPv6)** — Addressing and routing
- **ICMP** — Error reporting and diagnostics (`ping`, `traceroute`)
- **ARP** — IP-to-MAC address resolution

### Tunnelling & VPN Protocols
- **IPSec** — Encryption at the network layer
- **GRE** — Generic Routing Encapsulation
- **OpenVPN** — Open-source VPN
- **WireGuard** — Modern, fast VPN protocol
- **L2TP/PPTP** — Legacy VPN protocols

---

## 10. DNS — Domain Name System

DNS is the internet's "phone book" — it translates human-readable domain names into IP addresses.

### How DNS Works (Step by Step)
```
You type: www.example.com

1. Your browser checks its local DNS cache
2. If not found → asks your OS (checks hosts file)
3. If not found → asks your configured DNS resolver (e.g., 8.8.8.8)
4. Resolver checks its cache
5. If not found → asks a Root Name Server (.)
6. Root refers to the TLD Name Server (.com)
7. TLD refers to the Authoritative Name Server for example.com
8. Authoritative NS returns the IP: 93.184.216.34
9. Resolver caches the result and returns it to you
10. Your browser connects to 93.184.216.34
```

### DNS Record Types
| Record | Purpose | Example |
|--------|---------|---------|
| **A** | Maps domain → IPv4 address | `example.com → 93.184.216.34` |
| **AAAA** | Maps domain → IPv6 address | `example.com → 2606:2800::` |
| **CNAME** | Alias for another domain | `www → example.com` |
| **MX** | Mail server for domain | `example.com → mail.example.com` |
| **TXT** | Text info (SPF, DKIM, verification) | `"v=spf1 include:..."` |
| **NS** | Name servers for domain | `example.com → ns1.dns.com` |
| **PTR** | Reverse DNS (IP → domain) | `93.184.216.34 → example.com` |
| **SOA** | Start of Authority — zone info | Admin email, serial number |
| **SRV** | Service location records | Used by SIP, XMPP, etc. |

### Common DNS Servers
| DNS Server | IP | Provider |
|-----------|----|---------|
| Google | 8.8.8.8 / 8.8.4.4 | Google |
| Cloudflare | 1.1.1.1 / 1.0.0.1 | Cloudflare |
| OpenDNS | 208.67.222.222 | Cisco |
| Quad9 | 9.9.9.9 | Quad9 (privacy-focused) |

### DNS Security Issues
- **DNS Spoofing / Cache Poisoning** — Attacker injects fake DNS records
- **DNS Hijacking** — Redirecting DNS queries to malicious servers
- **DNS Tunnelling** — Encoding data in DNS queries to exfiltrate data
- **DNSSEC** — DNS Security Extensions add cryptographic signatures to prevent spoofing

---

## 11. DHCP

**DHCP (Dynamic Host Configuration Protocol)** automatically assigns IP addresses and network configuration to devices when they join a network.

### What DHCP Provides
- IP address
- Subnet mask
- Default gateway
- DNS server addresses
- Lease duration

### DHCP DORA Process
```
Client              DHCP Server
  |                     |
  |── DISCOVER ────────►|   "Is there a DHCP server? I need an IP"
  |                     |
  |◄── OFFER ───────────|   "Here, take 192.168.1.50"
  |                     |
  |── REQUEST ─────────►|   "I'll take 192.168.1.50 please"
  |                     |
  |◄── ACKNOWLEDGE ─────|   "It's yours for 24 hours"
```

### DHCP Lease
- IP addresses are leased for a set time (e.g., 24 hours)
- Devices renew leases before they expire
- Released when a device disconnects

### Static vs Dynamic IPs
| | Static | Dynamic (DHCP) |
|--|--------|----------------|
| Assignment | Manual | Automatic |
| Changes | Never | Can change on reconnect |
| Use case | Servers, printers, routers | Laptops, phones, guests |

> **Security note:** A **DHCP starvation attack** floods the server with requests, exhausting the IP pool. A **rogue DHCP server** attack plants a fake DHCP server to redirect traffic.

---

## 12. Routing & Switching

### Switching (Layer 2)

A **switch** connects devices within the same network (LAN). It uses MAC addresses to forward frames to the correct port.

**How a switch learns:**
1. Device A sends a frame
2. Switch records: "Port 1 = MAC of Device A" in its **MAC address table**
3. Switch forwards the frame to the destination MAC
4. Over time, the MAC table fills up — switch sends frames directly, not to all ports

**Key switch concepts:**
- **VLAN (Virtual LAN)** — Logically segments a switch into separate networks
- **Trunk port** — Carries traffic for multiple VLANs between switches
- **STP (Spanning Tree Protocol)** — Prevents loops in redundant switch networks
- **Port security** — Limits which MACs can connect to a port

### Routing (Layer 3)

A **router** connects different networks and forwards packets between them using IP addresses.

**Routing table example:**
```
Destination      Subnet Mask       Gateway         Interface
0.0.0.0          0.0.0.0           192.168.1.1     eth0      ← Default route
192.168.1.0      255.255.255.0     0.0.0.0         eth0      ← Local network
10.0.0.0         255.0.0.0         10.0.0.1        eth1      ← Remote network
```

### Routing Protocols
| Protocol | Type | Use |
|----------|------|-----|
| **Static routing** | Manual | Small networks, simple setups |
| **RIP** | Dynamic (Distance Vector) | Legacy, small networks |
| **OSPF** | Dynamic (Link State) | Enterprise internal routing |
| **BGP** | Dynamic (Path Vector) | Internet backbone routing |
| **EIGRP** | Dynamic (Cisco proprietary) | Cisco enterprise networks |

### Default Gateway
- The router's IP address on your local network
- Your device sends all traffic destined outside the local subnet to the default gateway
- Typically the first usable IP in a subnet (e.g., `192.168.1.1`)

---

## 13. Firewalls & NAT

### Firewalls

A **firewall** monitors and controls incoming and outgoing network traffic based on defined rules.

**Types of firewalls:**

| Type | How it works |
|------|-------------|
| **Packet filter** | Inspects individual packets (IP, port, protocol) — stateless |
| **Stateful firewall** | Tracks connection state — knows if a packet is part of an established session |
| **Application firewall (WAF)** | Inspects application-layer data (HTTP content, payloads) |
| **Next-Gen Firewall (NGFW)** | Combines stateful + application awareness + IPS + deep packet inspection |
| **Host-based firewall** | Software firewall on individual devices (Windows Defender Firewall, iptables) |

**Firewall rule example (iptables):**
```bash
# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH from specific IP only
iptables -A INPUT -p tcp --dport 22 -s 192.168.1.10 -j ACCEPT

# Drop all other inbound traffic
iptables -A INPUT -j DROP
```

### NAT — Network Address Translation

NAT allows multiple devices on a private network to share a single public IP address.

```
Private Network              Router (NAT)         Internet
192.168.1.10  ──┐                                ┌── Web Server
192.168.1.11  ──┼──► 203.0.113.5 (Public IP) ──►│
192.168.1.12  ──┘                                └── ...
```

**Types of NAT:**
- **SNAT (Source NAT)** — Translates private source IP to public IP (most common)
- **DNAT (Destination NAT)** — Redirects inbound traffic to a specific internal host (port forwarding)
- **PAT (Port Address Translation)** — Many-to-one NAT using port numbers to track sessions

---

## 14. Wireless Networking

### Wi-Fi Standards
| Standard | Frequency | Max Speed | Notes |
|----------|-----------|-----------|-------|
| 802.11b | 2.4 GHz | 11 Mbps | Legacy |
| 802.11g | 2.4 GHz | 54 Mbps | Legacy |
| 802.11n (Wi-Fi 4) | 2.4/5 GHz | 600 Mbps | Common |
| 802.11ac (Wi-Fi 5) | 5 GHz | 3.5 Gbps | Very common |
| 802.11ax (Wi-Fi 6/6E) | 2.4/5/6 GHz | 9.6 Gbps | Current standard |

### Wireless Security Protocols
| Protocol | Status | Notes |
|----------|--------|-------|
| **WEP** | Broken — never use | RC4-based, trivially crackable |
| **WPA** | Deprecated | TKIP-based, weak |
| **WPA2** | Current minimum | AES-CCMP, strong if using strong password |
| **WPA3** | Recommended | SAE handshake, forward secrecy, stronger |

### Wireless Concepts
- **SSID** — Network name broadcast by access point
- **BSSID** — MAC address of the access point
- **Channel** — Frequency sub-band used (1–13 for 2.4GHz; 36–165 for 5GHz)
- **Hidden SSID** — SSID not broadcast (security through obscurity — not effective)
- **Access Point (AP)** — Device providing wireless connectivity
- **WPS** — Wi-Fi Protected Setup — convenient but has known vulnerabilities

### Common Wireless Attacks
- **Evil Twin** — Rogue AP mimicking a legitimate network
- **Deauthentication attack** — Forcing clients to reconnect (capturing handshakes)
- **WPA2 handshake capture + cracking** — Offline password cracking
- **PMKID attack** — Crack WPA2 without a full handshake

---

## 15. Network Devices

| Device | Layer | Function |
|--------|-------|---------|
| **Hub** | Layer 1 | Broadcasts all traffic to all ports — dumb, largely obsolete |
| **Switch** | Layer 2 | Forwards frames using MAC addresses — connects devices in a LAN |
| **Router** | Layer 3 | Routes packets between networks using IP addresses |
| **Firewall** | Layer 3–7 | Filters traffic based on rules |
| **Access Point (AP)** | Layer 2 | Provides wireless connectivity |
| **Modem** | Layer 1–2 | Modulates/demodulates signal for ISP connection |
| **Load Balancer** | Layer 4–7 | Distributes traffic across multiple servers |
| **Proxy Server** | Layer 7 | Intermediary between clients and servers |
| **IDS/IPS** | Layer 3–7 | Detects or prevents intrusions |
| **Network TAP** | Layer 1 | Passively copies network traffic for monitoring |
| **SPAN/Mirror Port** | Layer 2 | Switch port that copies traffic for analysis |

---

## 16. Network Security Basics

### Common Network Attacks

| Attack | Description |
|--------|------------|
| **ARP Spoofing** | Poisoning ARP cache to redirect Layer 2 traffic |
| **Man-in-the-Middle (MitM)** | Intercepting traffic between two parties |
| **DoS / DDoS** | Flooding a target to make it unavailable |
| **SYN Flood** | Exploiting TCP handshake to exhaust server resources |
| **DNS Spoofing** | Injecting fake DNS responses |
| **VLAN Hopping** | Bypassing VLAN segmentation |
| **Rogue DHCP** | Fake DHCP server to redirect traffic |
| **Packet Sniffing** | Capturing network traffic passively |
| **Port Scanning** | Probing open ports for services |
| **ICMP Tunnelling** | Hiding data in ping packets |

### Defense Mechanisms

| Defense | What it protects against |
|---------|------------------------|
| **Network segmentation / VLANs** | Limits blast radius of breaches |
| **Firewall rules** | Blocks unauthorized traffic |
| **IDS/IPS** | Detects/prevents known attack patterns |
| **Encryption (TLS/IPSec)** | Protects data in transit from sniffing |
| **Strong Wi-Fi passwords (WPA3)** | Wireless network access |
| **802.1X port authentication** | Only authorized devices can join |
| **DNSSEC** | DNS cache poisoning |
| **Rate limiting** | DoS and brute-force mitigation |
| **Network monitoring (SIEM)** | Visibility into traffic anomalies |
| **Zero Trust architecture** | Never trust, always verify — even inside the network |

---

## 17. Common Ports & Services

Knowing ports is essential for network troubleshooting, firewalling, and security analysis.

### Well-Known Ports (0–1023)
| Port | Protocol | Service |
|------|----------|---------|
| 20 | TCP | FTP Data |
| 21 | TCP | FTP Control |
| 22 | TCP | SSH |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 67/68 | UDP | DHCP |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 |
| 123 | UDP | NTP |
| 143 | TCP | IMAP |
| 161/162 | UDP | SNMP |
| 389 | TCP | LDAP |
| 443 | TCP | HTTPS |
| 445 | TCP | SMB |
| 465/587 | TCP | SMTP (TLS) |
| 636 | TCP | LDAPS |
| 993 | TCP | IMAPS |
| 995 | TCP | POP3S |

### Registered Ports (1024–49151)
| Port | Service |
|------|---------|
| 1433 | Microsoft SQL Server |
| 1521 | Oracle DB |
| 3306 | MySQL |
| 3389 | RDP |
| 5432 | PostgreSQL |
| 5900 | VNC |
| 6379 | Redis |
| 8080 | HTTP Alternate |
| 8443 | HTTPS Alternate |
| 27017 | MongoDB |

### Scanning Ports with Nmap
```bash
# Quick scan — top 1000 ports
nmap 192.168.1.1

# Full port scan
nmap -p- 192.168.1.1

# Version detection + OS detection
nmap -sV -O 192.168.1.1

# Stealth SYN scan
nmap -sS 192.168.1.1

# Scan entire subnet
nmap 192.168.1.0/24
```

---

## 18. Packet Analysis

Packet analysis (packet capture / pcap) is the process of capturing and inspecting network traffic.

### Wireshark
Wireshark is the most popular packet analyser — free and open source.

**Key display filters:**
```
ip.addr == 192.168.1.5          # Filter by IP address
tcp.port == 80                  # Filter by port
http                            # Show only HTTP traffic
dns                             # Show only DNS traffic
tcp.flags.syn == 1              # Show SYN packets
!(arp or dns)                   # Exclude ARP and DNS
http.request.method == "POST"  # Show HTTP POST requests
```

**Following a TCP stream:**
Right-click a packet → Follow → TCP Stream — see the full conversation in plain text.

### tcpdump (Command Line)
```bash
# Capture all traffic on eth0
tcpdump -i eth0

# Capture HTTP traffic
tcpdump -i eth0 port 80

# Save to file
tcpdump -i eth0 -w capture.pcap

# Read from file
tcpdump -r capture.pcap

# Show packet contents in ASCII
tcpdump -A -i eth0
```

### What to Look For
- **Cleartext credentials** — Telnet, FTP, HTTP basic auth
- **Unusual volumes of traffic** — possible exfiltration or DoS
- **Unexpected outbound connections** — possible malware C2
- **ARP requests to many hosts** — possible network scanning
- **Large DNS responses** — possible DNS tunnelling

---

## 19. Virtual Networks & VPNs

### VLANs (Virtual LANs)

VLANs logically segment a physical network into isolated broadcast domains.

```
Physical Switch
├── Ports 1–10  → VLAN 10 (HR)       ← can't talk to other VLANs directly
├── Ports 11–20 → VLAN 20 (Finance)  ← isolated
└── Ports 21–30 → VLAN 30 (IT)       ← isolated
```

- Traffic between VLANs requires a **Layer 3 device (router or Layer 3 switch)**
- VLANs are tagged in frames using **802.1Q** tagging
- Improves security, performance, and network management

### VPNs (Virtual Private Networks)

A VPN creates an encrypted tunnel over a public network (like the internet).

**Types of VPN:**

| Type | Use |
|------|-----|
| **Remote Access VPN** | Individual users connecting to a corporate network remotely |
| **Site-to-Site VPN** | Connects two office networks permanently over the internet |
| **Split Tunnelling** | Only specific traffic goes through the VPN — rest goes direct |

**Common VPN Protocols:**
- **OpenVPN** — Open source, widely used, TLS-based
- **WireGuard** — Modern, fast, minimal codebase
- **IPSec/IKEv2** — Native to most operating systems
- **L2TP/IPSec** — Older but common
- **PPTP** — Deprecated — broken security

### Network Virtualization
- **VMware NSX / Open vSwitch** — Software-defined networking
- **Overlay networks** — VXLAN, GENEVE (used in cloud/container networking)
- **SDN (Software-Defined Networking)** — Separates control plane from data plane

---

## 20. Cloud Networking

Cloud providers have their own networking constructs that map to traditional concepts.

### Key Cloud Networking Concepts

| Concept | AWS | Azure | GCP |
|---------|-----|-------|-----|
| Virtual Network | VPC | VNet | VPC |
| Subnet | Subnet | Subnet | Subnet |
| Internet Gateway | IGW | Internet Gateway | Cloud Router |
| Private Subnet | Private subnet + NAT Gateway | Private subnet | Private subnet |
| Firewall Rules | Security Groups + NACLs | NSG | Firewall Rules |
| Load Balancer | ALB / NLB | Azure Load Balancer | Cloud Load Balancing |
| DNS | Route 53 | Azure DNS | Cloud DNS |
| VPN | VPN Gateway | VPN Gateway | Cloud VPN |
| Private connectivity | Direct Connect | ExpressRoute | Cloud Interconnect |

### VPC (Virtual Private Cloud)
- Isolated virtual network within the cloud provider
- You define IP ranges, subnets, routing, and firewall rules
- Can be connected to on-premises via VPN or dedicated connection

### Security Groups vs NACLs (AWS)
| | Security Group | Network ACL |
|--|---------------|------------|
| Level | Instance (VM) | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only | Allow and Deny |
| Default | Deny all inbound | Allow all |

---

## 21. Networking in Cybersecurity

Networking knowledge is directly applied across nearly every area of cybersecurity.

### Reconnaissance
```bash
# Ping sweep — find live hosts
nmap -sn 192.168.1.0/24

# DNS enumeration
dig @8.8.8.8 example.com ANY
nslookup example.com

# Traceroute — map network path
traceroute example.com        # Linux
tracert example.com           # Windows
```

### Network-Based Attacks to Understand
- **Port scanning** — Identify open services (Nmap)
- **Banner grabbing** — Identify service versions (Netcat, Nmap -sV)
- **Packet sniffing** — Capture traffic on local network
- **ARP poisoning** — MitM on local network (arpspoof, Bettercap)
- **DNS poisoning** — Redirect traffic via forged DNS
- **SMB relay attacks** — Capture and replay Windows authentication
- **LLMNR/NBT-NS poisoning** — Respond to name resolution broadcasts (Responder)

### Key Tools for Network Security
| Tool | Purpose |
|------|---------|
| **Nmap** | Port scanning and service enumeration |
| **Wireshark** | Packet capture and analysis |
| **tcpdump** | CLI packet capture |
| **Netcat (nc)** | Network debugging, banners, shells |
| **Bettercap** | MitM framework |
| **Responder** | LLMNR/NBNS/WPAD poisoning |
| **Scapy** | Python-based packet crafting |
| **Masscan** | Fast large-scale port scanner |
| **Zeek** | Network traffic analysis and logging |

### Network Forensics
When investigating an incident, network data helps you:
- Trace the attacker's path through the network
- Identify what data was accessed or exfiltrated
- Determine the initial access vector
- Establish a timeline of events

**Evidence sources:**
- Firewall logs
- Router/switch logs and NetFlow data
- SIEM-correlated events
- Full packet captures (if available)
- DNS logs
- Proxy / web gateway logs

---

## 22. Glossary

| Term | Definition |
|------|-----------|
| **ACL** | Access Control List — rules controlling traffic flow |
| **ARP** | Address Resolution Protocol — maps IP to MAC |
| **AS** | Autonomous System — a collection of IP networks under one routing policy |
| **Bandwidth** | Maximum data transfer capacity of a connection |
| **BGP** | Border Gateway Protocol — routes traffic between internet ASes |
| **Broadcast** | Sending a packet to all devices on a network |
| **CIDR** | Classless Inter-Domain Routing — flexible subnetting notation |
| **Default Gateway** | Router IP used to reach networks outside the local subnet |
| **DHCP** | Dynamic Host Configuration Protocol — auto IP assignment |
| **DNS** | Domain Name System — translates names to IPs |
| **Encapsulation** | Wrapping data with protocol headers at each OSI layer |
| **Ethernet** | Wired LAN standard (IEEE 802.3) |
| **Firewall** | Network security device that filters traffic |
| **Frame** | Layer 2 data unit (includes MAC addresses) |
| **Gateway** | Device that connects networks using different protocols |
| **Hub** | Layer 1 device that broadcasts to all ports |
| **ICMP** | Internet Control Message Protocol — used for ping, traceroute |
| **IDS/IPS** | Intrusion Detection/Prevention System |
| **IP** | Internet Protocol — addressing and routing |
| **ISP** | Internet Service Provider |
| **LAN** | Local Area Network |
| **Latency** | Delay in data transmission |
| **MAC Address** | Hardware address of a network interface |
| **MTU** | Maximum Transmission Unit — largest packet size |
| **NAT** | Network Address Translation |
| **NIC** | Network Interface Card |
| **OSI Model** | 7-layer network communication framework |
| **Packet** | Layer 3 data unit (includes IP addresses) |
| **Ping** | ICMP echo test to check connectivity |
| **Port** | Logical endpoint for a specific service (0–65535) |
| **Protocol** | Rules for network communication |
| **Proxy** | Intermediary between client and server |
| **Router** | Device that forwards packets between networks |
| **Routing Table** | Database of network routes on a router |
| **SDN** | Software-Defined Networking |
| **Segment** | Layer 4 data unit (TCP) |
| **SIEM** | Security Information and Event Management |
| **SNMP** | Simple Network Management Protocol |
| **Subnet** | Subdivision of an IP network |
| **Switch** | Layer 2 device for connecting LAN devices |
| **TCP** | Transmission Control Protocol — reliable transport |
| **Topology** | Physical or logical layout of a network |
| **Traceroute** | Tool to trace the path packets take to a destination |
| **TTL** | Time To Live — limits packet lifespan (hops) |
| **UDP** | User Datagram Protocol — fast, connectionless transport |
| **VLAN** | Virtual LAN — logical network segmentation |
| **VPN** | Virtual Private Network — encrypted tunnel |
| **WAN** | Wide Area Network |
| **Wireshark** | Packet capture and analysis tool |

---

*Last updated: 2025 | For the Cybersecurity-Basics knowledge base*