# The OSI Model

> A complete guide to the Open Systems Interconnection model — the foundational framework for understanding how network communication works.

---

## What is the OSI Model?

The **OSI (Open Systems Interconnection) model** is a conceptual framework that standardises how different systems communicate over a network. It was developed by the **International Organisation for Standardisation (ISO)** in 1984.

The model breaks network communication into **7 distinct layers**, each with a specific role. When data is sent from one device to another, it travels down through all 7 layers on the sender's side, across the network, and back up through all 7 layers on the receiver's side.

### Why the OSI Model Matters
- Gives everyone a **common language** to describe network communication
- Helps **isolate and troubleshoot** network problems layer by layer
- Explains **where attacks happen** — essential knowledge in cybersecurity
- Underpins **how protocols and tools** are designed and categorised

---

## The 7 Layers at a Glance

```
┌──────────────────────────────────────────────────┐
│  Layer 7 — Application                           │
├──────────────────────────────────────────────────┤
│  Layer 6 — Presentation                          │
├──────────────────────────────────────────────────┤
│  Layer 5 — Session                               │
├──────────────────────────────────────────────────┤
│  Layer 4 — Transport                             │
├──────────────────────────────────────────────────┤
│  Layer 3 — Network                               │
├──────────────────────────────────────────────────┤
│  Layer 2 — Data Link                             │
├──────────────────────────────────────────────────┤
│  Layer 1 — Physical                              │
└──────────────────────────────────────────────────┘
```

### Mnemonic (Top to Bottom)
> **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing

### Mnemonic (Bottom to Top)
> **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way

---

## Encapsulation & Decapsulation

Before diving into each layer, it's important to understand **encapsulation** — the process of wrapping data with headers (and sometimes trailers) as it moves down the OSI layers on the sending side.

### Sending (Encapsulation — top to bottom)
```
Application data
        ↓  + Application header  → DATA
        ↓  + Presentation header → DATA
        ↓  + Session header      → DATA
        ↓  + TCP/UDP header      → SEGMENT
        ↓  + IP header           → PACKET
        ↓  + MAC header/trailer  → FRAME
        ↓  converted to bits     → BITS
```

### Receiving (Decapsulation — bottom to top)
Each layer strips its own header and passes the data up to the next layer, until the original data reaches the application.

### Protocol Data Units (PDUs)
Each layer has a name for its unit of data:

| Layer | PDU Name |
|-------|---------|
| Layer 7–5 | Data |
| Layer 4 | Segment (TCP) / Datagram (UDP) |
| Layer 3 | Packet |
| Layer 2 | Frame |
| Layer 1 | Bits |

---

## Layer 1 — Physical

The Physical layer is the lowest layer. It deals with the **actual transmission of raw bits** — 1s and 0s — over a physical medium.

### What it does
- Converts digital data into signals (electrical, optical, or radio)
- Defines the physical characteristics of the medium — cable type, connector type, voltage levels, signal timing
- Transmits and receives raw bit streams

### What it does NOT do
- It has no concept of addressing — it just sends bits
- It doesn't know if the data arrived correctly

### Examples
- **Cables** — Ethernet (Cat5e, Cat6), fibre optic, coaxial
- **Connectors** — RJ45, SFP, USB
- **Wireless** — Radio waves, infrared
- **Devices** — Hubs, repeaters, network interface cards (at the physical level), cables

### Common Issues at Layer 1
- Damaged or loose cables
- Interference on wireless signals
- Wrong cable type used
- Faulty NIC or port

---

## Layer 2 — Data Link

The Data Link layer is responsible for **node-to-node delivery** within the same network. It takes raw bits from Layer 1 and organises them into meaningful units called **frames**.

### What it does
- Adds **MAC addresses** (source and destination) to frames
- Handles **error detection** — detects (but doesn't always correct) corrupted frames using CRC (Cyclic Redundancy Check)
- Controls **how devices share the physical medium** (Media Access Control)
- Manages **flow control** between directly connected nodes

### Two Sub-layers
- **MAC (Media Access Control)** — controls how devices access the shared medium, handles MAC addressing
- **LLC (Logical Link Control)** — provides flow control and error notification to Layer 3

### Examples
- **Protocols** — Ethernet (IEEE 802.3), Wi-Fi (IEEE 802.11), PPP, ARP
- **Devices** — Switches, bridges, wireless access points
- **Addressing** — MAC addresses

### Frame Structure (Ethernet)
```
┌──────────────┬────────────┬────────────┬──────┬──────────┬─────┐
│  Preamble    │ Dest MAC   │ Source MAC │ Type │  Data    │ FCS │
│  (7 bytes)   │ (6 bytes)  │ (6 bytes)  │(2 B) │(46–1500B)│(4B) │
└──────────────┴────────────┴────────────┴──────┴──────────┴─────┘
FCS = Frame Check Sequence (error detection)
```

### Security at Layer 2
- **ARP Spoofing** — Sending fake ARP replies to poison devices' ARP caches
- **MAC Flooding** — Overwhelming a switch's MAC table, forcing it to broadcast all traffic
- **VLAN Hopping** — Exploiting trunk port misconfigurations to access other VLANs

---

## Layer 3 — Network

The Network layer is responsible for **routing data between different networks**. This is where IP addresses live.

### What it does
- Assigns logical **IP addresses** to devices
- **Routes packets** from source to destination across multiple networks
- Determines the best path for data to travel (routing)
- Handles **fragmentation** — breaking large packets into smaller ones if needed

### Examples
- **Protocols** — IP (IPv4, IPv6), ICMP, ARP (debated — often placed at L2/L3 boundary)
- **Routing protocols** — OSPF, BGP, RIP, EIGRP
- **Devices** — Routers, Layer 3 switches
- **Addressing** — IP addresses

### IPv4 Packet Header (simplified)
```
┌─────────────┬────────────┬───────────┬──────────────────┐
│  Version    │  TTL       │ Protocol  │  Header Checksum │
├─────────────┴────────────┴───────────┴──────────────────┤
│             Source IP Address (32 bits)                  │
├──────────────────────────────────────────────────────────┤
│           Destination IP Address (32 bits)               │
└──────────────────────────────────────────────────────────┘
TTL = Time To Live (decremented at each router hop; dropped at 0)
```

### ICMP — Internet Control Message Protocol
ICMP lives at Layer 3. It's used for diagnostics and error reporting.
- `ping` sends ICMP Echo Requests and waits for Echo Replies
- `traceroute` / `tracert` uses ICMP (or UDP) to map the path packets take
- ICMP is also used to report errors — "Destination Unreachable", "TTL Exceeded"

### Security at Layer 3
- **IP Spoofing** — Forging the source IP address in packets
- **ICMP Flood (Ping Flood)** — Overwhelming a target with ICMP requests (DoS)
- **Smurf Attack** — Amplified ICMP flood using broadcast addresses
- **Routing attacks** — BGP hijacking, OSPF route injection

---

## Layer 4 — Transport

The Transport layer is responsible for **end-to-end communication** between applications on different hosts. It manages how data is broken up, sent, and reassembled.

### What it does
- **Segmentation** — Breaks large data into smaller segments
- **Reassembly** — Puts segments back together in the correct order at the destination
- **Port numbers** — Identifies which application the data belongs to
- **Flow control** — Ensures the sender doesn't overwhelm the receiver
- **Error recovery** — TCP retransmits lost segments

### TCP vs UDP

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Requires handshake | Connectionless |
| Reliability | Guaranteed delivery | No guarantee |
| Order | Maintains order | No ordering |
| Speed | Slower | Faster |
| Use cases | Web, email, file transfer | DNS, VoIP, streaming, gaming |

### The TCP 3-Way Handshake
```
Client                        Server
  │                              │
  │──────── SYN ────────────────►│   "I want to connect, seq=100"
  │                              │
  │◄─────── SYN-ACK ─────────────│   "OK, seq=300, ack=101"
  │                              │
  │──────── ACK ────────────────►│   "Acknowledged, ack=301"
  │                              │
  │════════ Data Transfer ═══════│
```

### Port Numbers
Ports identify specific services/applications on a device. Combined with an IP address, they form a **socket** (e.g., `192.168.1.5:443`).

| Range | Type | Description |
|-------|------|------------|
| 0–1023 | Well-known ports | Reserved for standard services (HTTP=80, HTTPS=443, SSH=22) |
| 1024–49151 | Registered ports | Assigned to specific applications (MySQL=3306, RDP=3389) |
| 49152–65535 | Dynamic/ephemeral ports | Temporarily assigned to client connections |

### Security at Layer 4
- **SYN Flood** — Sends many SYN packets without completing the handshake, exhausting server resources
- **Port Scanning** — Probing ports to discover open services (Nmap)
- **Session Hijacking** — Taking over an established TCP session by predicting sequence numbers
- **UDP Flood** — Overwhelming a target with UDP packets (DoS)

---

## Layer 5 — Session

The Session layer manages **sessions** — the opening, maintaining, and closing of communication sessions between two applications.

### What it does
- **Establishes sessions** — Sets up a communication channel between two applications
- **Maintains sessions** — Keeps the session alive during data exchange, handles interruptions
- **Terminates sessions** — Cleanly closes sessions when communication is complete
- **Synchronisation** — Uses checkpoints so a long transfer can resume from where it left off if interrupted, rather than restarting from scratch

### Examples
- **Protocols** — NetBIOS, RPC (Remote Procedure Call), SMB session management, PPTP
- **Real-world analogy** — Like a phone call: you dial (establish), talk (maintain), and hang up (terminate)

### Session vs Connection
A *connection* (Layer 4) is about the reliable transport of data. A *session* (Layer 5) is about the logical conversation between two applications — it can persist across multiple connections.

### Security at Layer 5
- **Session Hijacking** — Stealing an authenticated session token to impersonate a user
- **Session Fixation** — Forcing a user to use a known session ID

---

## Layer 6 — Presentation

The Presentation layer is responsible for **how data is formatted, encoded, and encrypted** so that both sides can understand each other.

### What it does
- **Translation** — Converts data between different formats (e.g., EBCDIC to ASCII)
- **Encryption / Decryption** — Encrypts data before transmission and decrypts on receipt (TLS/SSL operates here)
- **Compression / Decompression** — Reduces data size for faster transmission

### Examples
- **Encryption** — TLS, SSL
- **Data formats** — JPEG, PNG, GIF, MP4, MP3
- **Encoding standards** — ASCII, UTF-8, EBCDIC
- **Serialisation** — JSON, XML, protobuf (converting data structures to a transmissible format)

### Real-World Analogy
Think of Layer 6 as a **translator**. If Device A speaks French and Device B speaks English, the Presentation layer handles the translation so both understand the same message.

### Security at Layer 6
- **SSL Stripping** — Downgrading an HTTPS connection to HTTP by stripping encryption
- **Weak cipher suites** — Using outdated encryption algorithms (RC4, MD5, DES) that can be broken
- **Certificate spoofing** — Presenting a fake TLS certificate to intercept encrypted traffic

---

## Layer 7 — Application

The Application layer is the topmost layer and the one users and applications interact with directly. It provides **network services to end-user applications**.

### What it does
- Provides the interface between the network and the application
- Defines protocols that applications use to communicate
- Handles things like **authentication, data exchange, and resource access**

### Note
Layer 7 does *not* refer to the application itself (e.g., Chrome, Outlook). It refers to the **protocols and services** those applications use to communicate over the network.

### Examples
- **Web** — HTTP, HTTPS
- **Email** — SMTP, POP3, IMAP
- **File transfer** — FTP, SFTP
- **Remote access** — SSH, Telnet, RDP
- **Name resolution** — DNS
- **Network management** — SNMP
- **Directory services** — LDAP

### Security at Layer 7
- **Cross-Site Scripting (XSS)** — Injecting malicious scripts into web pages
- **SQL Injection** — Injecting SQL code into application inputs
- **Cross-Site Request Forgery (CSRF)** — Tricking users into executing unwanted actions
- **Directory Traversal** — Accessing files outside the intended directory
- **DDoS at Layer 7** — HTTP flood attacks targeting web application resources
- **DNS Spoofing** — Returning fake DNS records to redirect users

---

## Summary Table

| Layer | Name | PDU | Key Protocols | Key Devices | Security Threats |
|-------|------|-----|--------------|-------------|-----------------|
| 7 | Application | Data | HTTP, DNS, FTP, SSH, SMTP | — | XSS, SQLi, DNS spoofing |
| 6 | Presentation | Data | TLS/SSL, JPEG, ASCII | — | SSL stripping, weak ciphers |
| 5 | Session | Data | NetBIOS, RPC, SMB | — | Session hijacking |
| 4 | Transport | Segment | TCP, UDP | — | SYN flood, port scanning |
| 3 | Network | Packet | IP, ICMP, ARP | Routers, L3 switches | IP spoofing, ICMP flood |
| 2 | Data Link | Frame | Ethernet, Wi-Fi, ARP | Switches, APs | ARP spoofing, MAC flooding |
| 1 | Physical | Bits | Ethernet (physical), Wi-Fi | Cables, hubs, NICs | Cable tapping, jamming |

---

## OSI vs TCP/IP Model

The TCP/IP model is the practical implementation the internet actually uses. It condenses the 7 OSI layers into 4.

```
OSI Model                   TCP/IP Model
─────────────────           ──────────────────
Layer 7 — Application  ─┐
Layer 6 — Presentation  ├─► Application Layer
Layer 5 — Session      ─┘
Layer 4 — Transport    ────► Transport Layer
Layer 3 — Network      ────► Internet Layer
Layer 2 — Data Link    ─┐
Layer 1 — Physical     ─┘──► Network Access Layer
```

The OSI model is used as a **teaching and troubleshooting framework**. The TCP/IP model is what's actually implemented in real-world systems.

---

## OSI in Troubleshooting

When something isn't working on a network, the OSI model gives you a structured way to diagnose it. Start at Layer 1 and work up.

```
Layer 1 — Is the cable plugged in? Are the lights on?
Layer 2 — Is the switch seeing the MAC address? Is VLAN configured correctly?
Layer 3 — Is there a valid IP? Can you ping the default gateway?
Layer 4 — Is the correct port open? Is a firewall blocking it?
Layer 5 — Is the session being established? Any timeout issues?
Layer 6 — Is there a certificate error? Encoding mismatch?
Layer 7 — Is the application responding? Is the service running?
```

### Example — "I can't access the website"
```
✓ Layer 1: Cable is connected, NIC lights are on
✓ Layer 2: Switch port is active, correct VLAN
✓ Layer 3: IP assigned, can ping default gateway
✓ Layer 4: Port 443 is open, no firewall block
✓ Layer 5: Session establishes successfully
✗ Layer 6: TLS certificate has expired → FOUND THE ISSUE
```

---

## OSI in Cybersecurity

Understanding which layer an attack operates at helps you select the right defensive tool.

| Attack | OSI Layer | Defence |
|--------|-----------|---------|
| Cutting a cable / physical intrusion | Layer 1 | Physical security, locked comms rooms |
| ARP spoofing | Layer 2 | Dynamic ARP Inspection, static ARP entries |
| IP spoofing | Layer 3 | Ingress filtering, uRPF |
| SYN flood | Layer 4 | SYN cookies, rate limiting, firewall |
| Session hijacking | Layer 5 | Secure session tokens, re-authentication |
| SSL stripping | Layer 6 | HSTS, enforcing TLS |
| SQL injection / XSS | Layer 7 | WAF, input validation, prepared statements |

Security tools also map to layers:
- **Firewall (stateful)** — Layer 3/4
- **IDS/IPS** — Layer 3–7
- **WAF (Web Application Firewall)** — Layer 7
- **Wireshark** — Captures at Layer 2, analyses up to Layer 7
- **Switches (managed)** — Layer 2
- **Routers** — Layer 3

---

## Glossary

| Term | Definition |
|------|-----------|
| **ARP** | Address Resolution Protocol — resolves IPs to MAC addresses (Layer 2/3) |
| **Checksum** | Error-detection value appended to data |
| **CRC** | Cyclic Redundancy Check — error detection in frames |
| **Decapsulation** | Stripping headers at each layer as data moves up the stack |
| **Encapsulation** | Adding headers at each layer as data moves down the stack |
| **Frame** | Layer 2 PDU containing MAC addresses |
| **Header** | Control information prepended to data at each layer |
| **ICMP** | Internet Control Message Protocol — diagnostics at Layer 3 |
| **IP** | Internet Protocol — logical addressing and routing (Layer 3) |
| **MAC Address** | Hardware address used at Layer 2 |
| **OSI** | Open Systems Interconnection — 7-layer networking model |
| **Packet** | Layer 3 PDU containing IP addresses |
| **PDU** | Protocol Data Unit — the data format at each OSI layer |
| **Port** | Logical endpoint for a service, used at Layer 4 |
| **Segment** | Layer 4 PDU (TCP) |
| **Session** | Logical communication channel managed at Layer 5 |
| **Socket** | Combination of IP address and port (e.g., 192.168.1.1:443) |
| **TCP** | Transmission Control Protocol — reliable transport at Layer 4 |
| **TLS** | Transport Layer Security — encryption at Layer 6 |
| **TTL** | Time To Live — limits packet lifespan, decremented at each router |
| **UDP** | User Datagram Protocol — fast, connectionless transport at Layer 4 |

---

*Last updated: 2025 | For the Cybersecurity-Basics knowledge base*