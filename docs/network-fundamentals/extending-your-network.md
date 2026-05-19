# Extending Your Network


## Introduction

As networks grow beyond a single local segment, they require additional technologies and architecture patterns to remain manageable, secure, and performant. **Extending your network** refers to the set of practices, protocols, and devices used to expand network reach — across buildings, cities, or the internet — while maintaining control and security.

Key goals when extending a network:

- **Connectivity** — allow devices across different locations to communicate.
- **Security** — prevent unauthorized access, contain breaches, and encrypt data in transit.
- **Performance** — minimize latency, avoid bottlenecks, and balance load.
- **Scalability** — support growth without complete redesign.

---

## Network Topologies

A **topology** describes how devices in a network are physically or logically arranged and connected.

### Common Topologies

| Topology | Description | Pros | Cons |
|---|---|---|---|
| **Bus** | All devices share a single cable | Simple, cheap | Single point of failure; collisions |
| **Star** | Devices connect to a central switch/hub | Easy to manage; failure isolated | Central device = single point of failure |
| **Ring** | Devices form a closed loop | Predictable performance | One failure can break the ring |
| **Mesh** | Devices connect to many others | High redundancy | Expensive; complex cabling |
| **Tree (Hierarchical)** | Star networks connected in tiers | Scalable; organized | Root failure = network down |
| **Hybrid** | Mix of two or more topologies | Flexible | Complex to manage |

### Physical vs Logical Topology

- **Physical topology** — how cables and hardware are actually laid out.
- **Logical topology** — how data flows through the network regardless of physical layout.

> **Example:** An Ethernet network may be physically wired in a star (all cables go to a switch), but logically behave as a bus (broadcast domain).

---

## Subnetting

**Subnetting** is the process of dividing a large IP network into smaller sub-networks (subnets). It improves performance, security, and address management.

### Why Subnet?

- Reduce broadcast traffic
- Isolate departments or devices (e.g., HR vs Engineering)
- Improve security by containing breaches
- Efficient use of IP address space

### IPv4 Subnetting Basics

An IPv4 address is 32 bits, written as four octets: `192.168.1.0`

A **subnet mask** determines which part of the address is the **network** portion and which is the **host** portion.

```
IP Address:    192.168.1.0
Subnet Mask:   255.255.255.0  (/24)
Network:       192.168.1.0
Host Range:    192.168.1.1 – 192.168.1.254
Broadcast:     192.168.1.255
```

### CIDR Notation

**CIDR (Classless Inter-Domain Routing)** uses a `/prefix` to denote the subnet mask.

| CIDR | Subnet Mask | # of Hosts |
|---|---|---|
| /8 | 255.0.0.0 | 16,777,214 |
| /16 | 255.255.0.0 | 65,534 |
| /24 | 255.255.255.0 | 254 |
| /25 | 255.255.255.128 | 126 |
| /26 | 255.255.255.192 | 62 |
| /27 | 255.255.255.224 | 30 |
| /28 | 255.255.255.240 | 14 |
| /30 | 255.255.255.252 | 2 |
| /32 | 255.255.255.255 | 1 (host route) |

### Subnetting Formula

- **Number of subnets** = 2^(bits borrowed)
- **Number of hosts per subnet** = 2^(host bits) − 2

### Private IP Address Ranges (RFC 1918)

| Range | CIDR | Usage |
|---|---|---|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | Large internal networks |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | Medium networks |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | Home/small office |

---

## VLAN

A **VLAN (Virtual Local Area Network)** logically segments a physical network into separate broadcast domains without requiring separate physical infrastructure.

### How VLANs Work

- VLANs are configured on **managed switches**.
- Each VLAN acts as its own network — devices in VLAN 10 cannot directly communicate with devices in VLAN 20.
- Traffic between VLANs requires a **Layer 3 device** (router or Layer 3 switch).

### VLAN Tagging (802.1Q)

The IEEE **802.1Q** standard adds a 4-byte tag to Ethernet frames to identify which VLAN the frame belongs to.

```
[ Ethernet Header ] [ 802.1Q Tag ] [ Payload ] [ FCS ]
                      ↑
                  VLAN ID (12-bit → 4094 VLANs possible)
```

### Port Types

| Port Type | Description |
|---|---|
| **Access port** | Belongs to a single VLAN; connects end devices (PCs, servers) |
| **Trunk port** | Carries traffic for multiple VLANs; connects switches or routers |

### Common VLAN Use Cases

- **Management VLAN** — isolate switch/router management traffic
- **Voice VLAN** — separate VoIP traffic for QoS
- **Guest VLAN** — isolate untrusted guest devices
- **IoT VLAN** — quarantine smart devices from critical systems

### Inter-VLAN Routing

To route between VLANs you can use:

1. **Router-on-a-stick** — single router interface with sub-interfaces for each VLAN.
2. **Layer 3 switch** — switch performs routing internally (preferred for performance).

---

## VPN

A **VPN (Virtual Private Network)** creates an encrypted tunnel over a public network (e.g., the internet), allowing remote users or sites to communicate as if on the same private network.

### VPN Types

| Type | Description | Use Case |
|---|---|---|
| **Remote Access VPN** | Individual user connects to corporate network | Work from home |
| **Site-to-Site VPN** | Two networks connected over the internet | Branch offices |
| **Client-to-Site VPN** | Same as remote access; client software used | Mobile workforce |

### Common VPN Protocols

| Protocol | Port | Encryption | Notes |
|---|---|---|---|
| **OpenVPN** | UDP 1194 / TCP 443 | AES-256 | Open-source, highly configurable |
| **WireGuard** | UDP 51820 | ChaCha20 | Modern, fast, minimal codebase |
| **IPSec/IKEv2** | UDP 500, 4500 | AES-256 | Native on most OSes; stable |
| **L2TP/IPSec** | UDP 1701 | AES-256 | Older; double encapsulation overhead |
| **PPTP** | TCP 1723 | MPPE (weak) | Legacy; considered insecure — avoid |
| **SSTP** | TCP 443 | SSL/TLS | Microsoft; firewall-friendly |

### VPN Split Tunneling

- **Full tunnel** — all traffic goes through the VPN (more secure, slower).
- **Split tunnel** — only corporate traffic goes through VPN; internet traffic goes direct (faster, less secure).

### VPN Security Considerations

- Use **strong authentication** (MFA, certificates) — not just username/password.
- Prefer **WireGuard or OpenVPN** over legacy protocols.
- Patch VPN gateways regularly — they are high-value targets.
- Log and monitor VPN connections for anomalies.

---

## Routing Protocols

**Routers** move packets between networks using routing tables. Routing protocols are used to automatically build and maintain these tables.

### Static vs Dynamic Routing

| | Static | Dynamic |
|---|---|---|
| Configuration | Manual | Automatic |
| Scalability | Poor (large networks) | Good |
| Resource usage | Low | Higher (CPU/memory) |
| Adaptability | None | Recalculates on change |
| Use case | Small networks, specific routes | Enterprise, ISPs |

### Dynamic Routing Protocols

#### Distance Vector Protocols

- Routers share their **entire routing table** with neighbors periodically.
- Choose paths based on **hop count** or metric.
- **RIP (Routing Information Protocol)** — max 15 hops; slow convergence; outdated.
- **EIGRP (Enhanced Interior Gateway Routing Protocol)** — Cisco proprietary; uses bandwidth + delay as metrics; fast convergence.

#### Link State Protocols

- Routers share **link state information** about their direct connections.
- Each router builds a **complete map** of the network (SPF tree).
- **OSPF (Open Shortest Path First)** — most common enterprise IGP; uses Dijkstra's algorithm; organizes routers into **areas**.
- **IS-IS** — similar to OSPF; common in ISP backbones.

#### Path Vector Protocols

- **BGP (Border Gateway Protocol)** — the **routing protocol of the internet**.
  - Used between **autonomous systems (AS)**.
  - Routes based on **policies** and **AS path** attributes.
  - Slow convergence by design; prioritizes stability.

### Administrative Distance (AD)

When multiple routing sources provide a route to the same destination, the router prefers the source with the **lowest AD**.

| Source | AD |
|---|---|
| Connected interface | 0 |
| Static route | 1 |
| EIGRP (internal) | 90 |
| OSPF | 110 |
| RIP | 120 |
| Unknown / untrusted | 255 |

---

## NAT

**NAT (Network Address Translation)** maps private IP addresses to one or more public IP addresses, allowing many internal devices to share a single public IP.

### Types of NAT

| Type | Description |
|---|---|
| **Static NAT** | One-to-one mapping of private to public IP |
| **Dynamic NAT** | Maps private IPs to a pool of public IPs |
| **PAT / NAT Overload** | Many private IPs → single public IP using different ports (most common) |

### PAT (Port Address Translation)

Also called **NAPT** or **NAT Overload**. This is what home routers use.

```
Device A: 192.168.1.10:50001  →  203.0.113.1:10001  →  Internet
Device B: 192.168.1.11:50002  →  203.0.113.1:10002  →  Internet
```

The router tracks connections using a **NAT translation table**.

### Security Implications of NAT

- Provides **basic obscurity** — internal IPs are hidden.
- **NOT a firewall** — NAT alone does not block inbound attacks.
- Can complicate **IPSec VPNs**, **SIP/VoIP**, and **peer-to-peer** applications.
- **NAT traversal (NAT-T)** techniques (STUN, TURN, hole punching) are used to work around NAT in such applications.

---

## DHCP

**DHCP (Dynamic Host Configuration Protocol)** automatically assigns IP configuration to devices on a network, eliminating the need for manual configuration.

### DORA Process

```
Client          Server
  |─── Discover ──→|   Broadcast: "I need an IP"
  |←─── Offer ────|   Server: "Here's 192.168.1.50"
  |─── Request ──→|   Client: "I'll take 192.168.1.50"
  |←── Acknowledge|   Server: "It's yours for [lease time]"
```

### DHCP Lease Components

A DHCP server assigns:

- **IP address**
- **Subnet mask**
- **Default gateway**
- **DNS servers**
- **Lease duration**

### DHCP in Extended Networks

- **DHCP relay agent** — forwards DHCP broadcasts across subnets (a router can act as a relay).
- This allows a single DHCP server to serve multiple subnets.

### DHCP Security Threats

| Threat | Description | Mitigation |
|---|---|---|
| **DHCP Starvation** | Attacker floods server with fake requests, exhausting the IP pool | DHCP snooping, rate limiting |
| **Rogue DHCP Server** | Attacker runs a fake DHCP server to hand out malicious gateway/DNS | DHCP snooping (trust only authorized ports) |

---

## DNS

**DNS (Domain Name System)** resolves human-readable hostnames (e.g., `example.com`) to IP addresses.

### DNS Resolution Process

```
1. Browser asks: "What is the IP of example.com?"
2. OS checks local cache → /etc/hosts → not found
3. Query goes to Recursive Resolver (ISP or 8.8.8.8)
4. Recursive Resolver asks Root Server (.) → "ask .com TLD server"
5. Resolver asks .com TLD server → "ask example.com nameserver"
6. Resolver asks example.com nameserver → returns 93.184.216.34
7. Resolver caches and returns answer to client
```

### Common DNS Record Types

| Record | Purpose | Example |
|---|---|---|
| **A** | IPv4 address | `example.com → 93.184.216.34` |
| **AAAA** | IPv6 address | `example.com → 2606:2800::1` |
| **CNAME** | Alias to another name | `www → example.com` |
| **MX** | Mail server | `@ → mail.example.com` |
| **TXT** | Arbitrary text (SPF, DKIM, etc.) | `v=spf1 include:...` |
| **NS** | Authoritative nameserver | `example.com → ns1.example.com` |
| **PTR** | Reverse DNS (IP → name) | `34.216.184.93 → example.com` |
| **SOA** | Start of Authority (zone info) | Zone serial, refresh times |
| **SRV** | Service location | `_sip._tcp → sip.example.com:5060` |

### DNS Security Threats

| Threat | Description |
|---|---|
| **DNS Spoofing / Cache Poisoning** | Attacker injects false DNS records into a resolver's cache |
| **DNS Hijacking** | Attacker redirects DNS queries to malicious servers |
| **DNS Tunneling** | Data exfiltration encoded in DNS queries (bypasses firewalls) |
| **DDoS via DNS Amplification** | Attacker uses open resolvers to amplify traffic toward victim |
| **NXDOMAIN Attack** | Flood of queries for non-existent domains to exhaust resolver |

### DNS Security Mitigations

- **DNSSEC** — digitally signs DNS records; prevents spoofing.
- **DoH (DNS over HTTPS)** — encrypts DNS queries; prevents eavesdropping.
- **DoT (DNS over TLS)** — encrypts DNS queries at the transport layer.
- **RPZ (Response Policy Zones)** — block malicious domains at the resolver level.

---

## Proxy Servers

A **proxy server** sits between clients and servers, forwarding requests on the client's behalf.

### Types of Proxies

| Type | Description | Use Case |
|---|---|---|
| **Forward Proxy** | Client → Proxy → Internet | Corporate web filtering, anonymity |
| **Reverse Proxy** | Internet → Proxy → Backend Servers | Load balancing, SSL termination, WAF |
| **Transparent Proxy** | Client unaware; traffic intercepted | ISP monitoring, caching |
| **SOCKS Proxy** | Works at Layer 5; protocol-agnostic | Bypassing restrictions |
| **Web Proxy (HTTP)** | Only HTTP/HTTPS traffic | Browser proxying |

### Forward Proxy Use Cases

- **Content filtering** — block malicious or inappropriate sites.
- **Caching** — cache frequently accessed content to save bandwidth.
- **Anonymity** — hide client's IP from external servers.
- **Logging/Monitoring** — inspect outbound traffic.

### Reverse Proxy Use Cases

- **Load balancing** — distribute traffic across backend servers.
- **SSL offloading** — handle TLS termination before passing to backend.
- **WAF (Web Application Firewall)** — inspect and filter HTTP traffic.
- **Caching** — serve static content without hitting the backend.
- **Hide backend infrastructure** — external users never see real server IPs.

> **Common reverse proxies:** Nginx, HAProxy, Apache, Cloudflare, Traefik.

---

## Load Balancers

A **load balancer** distributes incoming network traffic across multiple servers to ensure no single server is overwhelmed.

### Load Balancing Algorithms

| Algorithm | Description |
|---|---|
| **Round Robin** | Requests distributed equally in rotation |
| **Weighted Round Robin** | Servers with higher weight receive more requests |
| **Least Connections** | New request goes to server with fewest active connections |
| **IP Hash** | Client IP determines which server handles their requests (session persistence) |
| **Random** | Randomly assigned |
| **Resource-based** | Based on actual server load (CPU, memory) |

### Layer 4 vs Layer 7 Load Balancing

| | Layer 4 (Transport) | Layer 7 (Application) |
|---|---|---|
| Works on | IP + TCP/UDP | HTTP headers, URLs, cookies |
| Speed | Faster | Slightly slower |
| Awareness | No content awareness | Content-aware routing |
| SSL offload | No | Yes |
| Use case | High throughput, any protocol | Web apps, microservices |

### Health Checks

Load balancers continuously probe backend servers (ping, HTTP GET, TCP connect) and **remove unhealthy nodes** from the pool automatically.

---

## DMZ

A **DMZ (Demilitarized Zone)** is a network segment that sits between the internal trusted network and the external untrusted internet. It hosts publicly accessible services while isolating the internal network.

### DMZ Architecture

```
Internet ──── [Firewall 1] ──── DMZ ──── [Firewall 2] ──── Internal Network

DMZ contains:
  - Web servers
  - Mail servers (MTA)
  - DNS servers (authoritative)
  - FTP servers
  - Reverse proxies
```

### Why Use a DMZ?

- Publicly accessible servers are **attack targets** — a compromise should not directly expose the internal network.
- The inner firewall blocks DMZ hosts from initiating connections to internal systems.
- Limits **lateral movement** after a breach.

### Single vs Dual Firewall DMZ

| Setup | Description |
|---|---|
| **Single Firewall (3-legged)** | One firewall with 3 interfaces: WAN, DMZ, LAN | Simpler; less secure |
| **Dual Firewall** | Two separate firewalls; DMZ sits between them | More secure; defense in depth |

---

## Wireless Networking Extensions

Wireless networks can be extended using several techniques and technologies.

### Wireless Extension Methods

| Method | Description |
|---|---|
| **Wireless Repeater/Extender** | Receives Wi-Fi signal and rebroadcasts it; extends range but halves bandwidth |
| **Wireless Access Point (WAP)** | Wired back to the network; provides full bandwidth wireless extension |
| **Mesh Network** | Multiple nodes create a self-healing wireless network with smart roaming |
| **WDS (Wireless Distribution System)** | Links APs wirelessly without a wired backbone; older approach |

### Wi-Fi Security Standards (Evolution)

| Standard | Notes |
|---|---|
| **WEP** | Broken; do not use |
| **WPA** | Improved over WEP but has vulnerabilities (TKIP weakness) |
| **WPA2-Personal (PSK)** | AES-CCMP encryption; secure for home use |
| **WPA2-Enterprise (802.1X)** | Uses RADIUS for per-user authentication; preferred for business |
| **WPA3** | Latest standard; SAE (Dragonfly) handshake; resistant to offline brute-force |
| **WPA3-Enterprise** | 192-bit security suite; for high-security environments |

### Wireless Threats

| Threat | Description |
|---|---|
| **Evil Twin / Rogue AP** | Attacker creates AP with the same SSID to intercept traffic |
| **Deauthentication Attack** | Attacker sends spoofed deauth frames to disconnect clients |
| **PMKID Attack** | Capture PMKID from beacon to perform offline password cracking |
| **KRACK (WPA2)** | Key Reinstallation Attack; patched on most modern devices |
| **Wardriving** | Driving around scanning for unsecured Wi-Fi networks |

### Enterprise Wireless Best Practices

- Deploy **WPA3-Enterprise with 802.1X** and a **RADIUS server**.
- Segment wireless traffic into VLANs (corporate, guest, IoT).
- Enable **rogue AP detection** on wireless controllers.
- Disable **WPS** — vulnerable to brute-force.
- Use **certificate-based authentication** to prevent credential harvesting.

---

## SD-WAN

**SD-WAN (Software-Defined Wide Area Network)** uses software to intelligently route traffic across multiple WAN connections (MPLS, broadband, 4G/5G, etc.).

### Traditional WAN vs SD-WAN

| | Traditional WAN | SD-WAN |
|---|---|---|
| Traffic routing | Hardware-defined, manual | Policy-based, dynamic |
| Link types | Primarily MPLS | MPLS, broadband, LTE, etc. |
| Management | Per-device CLI | Centralized controller |
| Cost | High (MPLS) | Lower (can replace or supplement MPLS) |
| Agility | Low | High |

### SD-WAN Key Features

- **Dynamic path selection** — route traffic based on latency, jitter, or packet loss.
- **Centralized management** — single pane of glass for all branch WAN links.
- **Application-aware routing** — send critical apps (VoIP) over best-quality link.
- **Zero Touch Provisioning (ZTP)** — branches configured automatically on startup.
- **Integrated security** — some SD-WAN platforms include firewall, IPS, and DNS filtering.

### SD-WAN Security Considerations

- Encrypted overlays (IPSec tunnels) between sites.
- Enforce **segmentation policies** consistently across all sites.
- Integrate with **SASE (Secure Access Service Edge)** for cloud-delivered security.

---

## Network Segmentation

**Network segmentation** divides a network into smaller zones to limit the blast radius of a security incident.

### Segmentation Approaches

| Approach | Method |
|---|---|
| **Physical segmentation** | Separate hardware (switches, routers, cables) per zone |
| **VLAN segmentation** | Logical separation on shared infrastructure |
| **Firewall-based segmentation** | Enforce policies between zones with firewalls |
| **Micro-segmentation** | Granular policy per workload/VM (used in data centers) |

### Segmentation Zones (Example)

```
[Internet]
    │
[Edge Firewall]
    │
[DMZ] ── Web Servers, Mail Relay, Public DNS
    │
[Internal Firewall]
    ├── [Corporate LAN] ── Workstations, Printers
    ├── [Server Zone]   ── File Servers, AD, Databases
    ├── [OT/IoT Zone]   ── Industrial controls, IoT devices
    └── [Management Zone] ── Network devices (out-of-band)
```

### Benefits of Segmentation

- **Contain breaches** — attacker in one zone cannot easily reach others.
- **Compliance** — PCI-DSS, HIPAA require isolation of sensitive data environments.
- **Reduce attack surface** — devices only communicate with what they need.
- **Easier monitoring** — traffic between zones crosses chokepoints (logged by firewall).

---

## Zero Trust Architecture

**Zero Trust** is a security model based on the principle: **"Never trust, always verify."** No user, device, or network segment is trusted by default — even if inside the perimeter.

### Core Principles

1. **Verify explicitly** — always authenticate and authorize using all available data (identity, device, location, behavior).
2. **Use least privilege access** — grant minimum necessary permissions; use JIT (Just-In-Time) access.
3. **Assume breach** — design as if the attacker is already inside; limit blast radius, encrypt everything, monitor constantly.

### Zero Trust vs Traditional Perimeter Security

| | Perimeter Security | Zero Trust |
|---|---|---|
| Trust model | Trust inside the network | Trust nothing by default |
| Primary control | Firewall at the edge | Identity + device posture |
| Remote access | VPN into trusted zone | Identity-aware proxy / ZTNA |
| Lateral movement | Possible once inside | Blocked by micro-segmentation |
| Data security | Perimeter = protection | Encryption + access control everywhere |

### Zero Trust Network Access (ZTNA)

**ZTNA** replaces the traditional VPN for remote access:
- Users connect to a **cloud or on-prem broker** that authenticates them.
- Access granted **per-application**, not full network access.
- Device posture (patch level, OS, endpoint agent) checked before granting access.

### Implementing Zero Trust (Phased Approach)

```
Phase 1: Identity
  → MFA everywhere, SSO, privileged access management

Phase 2: Devices
  → Device inventory, endpoint compliance (MDM/EDR)

Phase 3: Network
  → Micro-segmentation, ZTNA, encrypted east-west traffic

Phase 4: Applications
  → App-level authentication, WAF, API security

Phase 5: Data
  → Data classification, DLP, encryption at rest and in transit
```

---

## Common Attack Vectors on Extended Networks

As networks grow, the attack surface expands. Understanding how attackers target extended networks is essential.

### Man-in-the-Middle (MitM)

The attacker positions themselves between two communicating parties.

- **ARP Poisoning** — attacker sends fake ARP replies linking their MAC to a legitimate IP.
- **DNS Spoofing** — malicious DNS responses redirect traffic.
- **SSL Stripping** — downgrade HTTPS to HTTP to intercept credentials.
- **BGP Hijacking** — attacker announces false BGP routes to divert internet traffic.

**Mitigations:** ARP inspection (DAI), DNSSEC, HSTS, enforce TLS.

### Lateral Movement

After compromising one host, attackers move through the network to reach higher-value targets.

Common techniques:
- **Pass-the-Hash** — reuse captured NTLM hashes.
- **Pass-the-Ticket** — reuse captured Kerberos tickets.
- **RDP/SMB pivoting** — use compromised host as a relay.

**Mitigations:** Segmentation, micro-segmentation, disable unnecessary services, EDR, network monitoring.

### VPN / Remote Access Attacks

- **Credential stuffing** on VPN portals.
- **Exploiting unpatched VPN gateways** (e.g., Pulse Secure, Fortinet CVEs).
- **Split tunnel abuse** — attacker on compromised endpoint reaches internal network.

**Mitigations:** MFA on VPN, patch management, certificate-based auth, ZTNA.

### Wireless Attacks

- Evil twin access points.
- WPS brute force.
- Deauthentication floods.

**Mitigations:** WPA3-Enterprise, rogue AP detection, 802.1X.

### DNS-Based Attacks

- DNS tunneling for C2 and data exfiltration.
- Fast flux DNS to evade blocklists.

**Mitigations:** DNS monitoring, RPZ, DoH/DoT internally.

### Supply Chain Attacks

Attacker compromises a trusted vendor or software update mechanism to gain access.

**Mitigations:** Vendor risk assessment, software signing, network monitoring for anomalous behavior post-update.

---

## Security Best Practices

### Network Design

- Apply **defense in depth** — multiple layers of security controls.
- Use a **DMZ** for all public-facing services.
- Implement **network segmentation** and separate sensitive zones.
- Deploy **dual-homed firewalls** at segment boundaries.
- Document and maintain an **accurate network map**.

### Access Control

- Enforce **least privilege** — devices/users only access what they need.
- Use **802.1X port authentication** to prevent unauthorized LAN access.
- Disable unused switch ports and place them in an isolated VLAN.
- Enforce **MFA** for all remote access.
- Implement **PAM (Privileged Access Management)** for admin accounts.

### Encryption

- Encrypt all traffic in transit — **TLS 1.2+** minimum; prefer **TLS 1.3**.
- Use **IPSec** or **WireGuard** for site-to-site and remote access VPNs.
- Enforce **HSTS** on web applications.
- Use **MACSec** (IEEE 802.1AE) for wire-speed encryption on LAN segments.

### Monitoring & Detection

- Collect logs from **firewalls, routers, switches, DNS servers, proxies**.
- Deploy a **SIEM** (Security Information and Event Management) to correlate events.
- Implement **NetFlow / IPFIX** analysis to detect anomalous traffic patterns.
- Deploy **IDS/IPS** at network boundaries and on critical segments.
- Set up **alerting** for: unusual outbound connections, DNS tunneling indicators, large data transfers, failed authentication spikes.

### Patch Management

- Maintain an asset inventory.
- Apply patches to **network devices** (routers, switches, firewalls, VPN gateways) promptly — these are high-value targets.
- Subscribe to vendor security advisories.

---

## Useful Tools

### Network Discovery & Mapping

| Tool | Use |
|---|---|
| `nmap` | Port scanning, OS detection, service enumeration |
| `netdiscover` | ARP-based host discovery |
| `masscan` | Ultra-fast port scanner |
| `Angry IP Scanner` | GUI-based network scanner |

### Traffic Analysis

| Tool | Use |
|---|---|
| `Wireshark` | Packet capture and deep protocol analysis |
| `tcpdump` | CLI packet capture |
| `ntopng` | Network traffic monitoring and flow analysis |
| `Zeek (Bro)` | Network security monitoring framework |

### VPN & Tunneling

| Tool | Use |
|---|---|
| `OpenVPN` | Flexible open-source VPN |
| `WireGuard` | Fast, modern VPN |
| `strongSwan` | IPSec VPN implementation |
| `Stunnel` | SSL/TLS tunnel wrapper |

### DNS Tools

| Tool | Use |
|---|---|
| `dig` | DNS query tool |
| `nslookup` | Basic DNS lookup |
| `dnsx` | Fast DNS toolkit |
| `dnsrecon` | DNS enumeration and recon |

### Wireless Tools

| Tool | Use |
|---|---|
| `aircrack-ng` | Wireless security auditing suite |
| `Kismet` | Wireless network detector and analyzer |
| `Wireshark` | Capture and analyze 802.11 frames |
| `hcxdumptool` | Capture WPA PMKID/handshakes |

### Proxy & Web

| Tool | Use |
|---|---|
| `Burp Suite` | Web proxy and security testing |
| `OWASP ZAP` | Open-source web application scanner |
| `mitmproxy` | Interactive HTTPS proxy |
| `Nginx / HAProxy` | Production reverse proxy / load balancer |

---

## Quick Reference

### Key Protocols & Ports

| Protocol | Port | Description |
|---|---|---|
| DNS | UDP/TCP 53 | Name resolution |
| DHCP | UDP 67/68 | IP assignment (server/client) |
| HTTP | TCP 80 | Web (unencrypted) |
| HTTPS | TCP 443 | Web (encrypted) |
| BGP | TCP 179 | Internet routing |
| OSPF | IP Protocol 89 | Link-state routing |
| OpenVPN | UDP 1194 | VPN |
| WireGuard | UDP 51820 | VPN |
| IKEv2 (IPSec) | UDP 500 / 4500 | VPN key exchange |
| RADIUS | UDP 1812/1813 | Network authentication |
| SNMP | UDP 161/162 | Network management |
| NTP | UDP 123 | Time synchronization |
| LDAP | TCP 389 | Directory services |
| LDAPS | TCP 636 | LDAP over TLS |

### Cheat Sheet: Subnetting

```
/24 → 254 hosts      /25 → 126 hosts     /26 → 62 hosts
/27 → 30 hosts       /28 → 14 hosts      /29 → 6 hosts
/30 → 2 hosts        /32 → 1 host (single IP)
```

### Cheat Sheet: OSI Layer Mapping

| Layer | Name | Devices | Protocols |
|---|---|---|---|
| 7 | Application | — | HTTP, DNS, SMTP, FTP |
| 6 | Presentation | — | SSL/TLS, JPEG, ASCII |
| 5 | Session | — | SMB, RPC, NetBIOS |
| 4 | Transport | Firewall (L4) | TCP, UDP |
| 3 | Network | Router, L3 Switch | IP, OSPF, BGP |
| 2 | Data Link | Switch, Bridge | Ethernet, 802.1Q, ARP |
| 1 | Physical | Hub, Cable, NIC | Electrical/optical signals |

---

> **Note:** This document is part of the [Cybersecurity-Basics](../README.md) notes repository. It is intended for learning purposes. Always practice in a legal, authorized environment (home lab, CTF, or with explicit permission).

*Last updated: 2026*