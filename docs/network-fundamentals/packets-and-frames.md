# Packets and Frames

> A detailed guide to understanding how data is structured, wrapped, and transmitted across networks — and the difference between packets and frames.

---

## What is a Packet?

When you send data over a network — a file, a web request, a message — that data doesn't travel as one continuous stream. It gets broken into small, manageable chunks called **packets**.

A **packet** is a unit of data at **Layer 3 (Network layer)** of the OSI model. It contains:
- The actual data being sent (or a piece of it)
- A **header** with routing information — most importantly, the source and destination **IP addresses**

### Why Break Data into Packets?

- **Efficiency** — Multiple devices can share the same network links simultaneously
- **Resilience** — If one packet is lost, only that packet needs to be retransmitted, not the entire file
- **Flexibility** — Different packets from the same transmission can take different routes to the destination and be reassembled at the other end
- **Fairness** — No single transfer can monopolise the network

### Analogy
Think of sending a large book through the post. Instead of sending it as one heavy parcel, you tear out each chapter, put each one in a separate envelope with the destination address on it, and send them all. They may arrive out of order, via different routes — but the recipient reassembles them into the full book.

---

## What is a Frame?

A **frame** is a unit of data at **Layer 2 (Data Link layer)** of the OSI model. It wraps a packet for delivery across a **single network link** — from one device to the next hop.

Where packets use IP addresses to route data across networks, frames use **MAC addresses** to deliver data between directly connected devices.

### The Key Difference

| | Packet | Frame |
|--|--------|-------|
| OSI Layer | Layer 3 — Network | Layer 2 — Data Link |
| Address type | IP addresses | MAC addresses |
| Scope | End-to-end across networks | Single hop (one link at a time) |
| Created by | Router / OS network stack | Switch / NIC |
| Protocol | IP | Ethernet, Wi-Fi |

### Analogy
If a packet is a letter with the final destination address written on it, a frame is the van that physically carries it from one post office to the next. The letter's destination address doesn't change — but a different van (frame) handles each leg of the journey.

---

## Encapsulation — How Packets and Frames are Built

As data travels **down** the OSI layers on the sending device, each layer wraps the data with its own header. This process is called **encapsulation**.

```
Application data: "GET / HTTP/1.1"
        │
        ▼  Layer 4 adds TCP header (ports, sequence numbers)
   [ TCP Header | Data ]          ← Segment
        │
        ▼  Layer 3 adds IP header (source/dest IP)
   [ IP Header | TCP Header | Data ]     ← Packet
        │
        ▼  Layer 2 adds Ethernet header + trailer (source/dest MAC)
   [ ETH Header | IP Header | TCP Header | Data | ETH Trailer ]  ← Frame
        │
        ▼  Layer 1 converts to bits and transmits
   10110001 01001101 ...           ← Bits on the wire
```

On the **receiving** side, each layer strips its own header as data moves up — this is called **decapsulation**.

---

## Anatomy of a Packet

A packet consists of a **header** and a **payload**.

### IPv4 Packet Header

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
┌─────────┬─────────┬───────────────────┬───────────────────────┐
│ Version │   IHL   │   DSCP / ECN      │     Total Length      │
├─────────┴─────────┴───────────────────┼───────────────────────┤
│         Identification                │ Flags │ Fragment Offset│
├───────────────────┬───────────────────┼───────────────────────┤
│       TTL         │     Protocol      │    Header Checksum    │
├───────────────────┴───────────────────┴───────────────────────┤
│                    Source IP Address                           │
├───────────────────────────────────────────────────────────────┤
│                  Destination IP Address                        │
└───────────────────────────────────────────────────────────────┘
```

### IPv4 Header Fields Explained

| Field | Size | Description |
|-------|------|------------|
| **Version** | 4 bits | IP version — `4` for IPv4, `6` for IPv6 |
| **IHL** | 4 bits | Internet Header Length — size of the header in 32-bit words |
| **DSCP / ECN** | 8 bits | Quality of service markings and congestion notification |
| **Total Length** | 16 bits | Total size of the packet (header + data), max 65,535 bytes |
| **Identification** | 16 bits | ID for reassembling fragmented packets |
| **Flags** | 3 bits | Controls fragmentation (Don't Fragment, More Fragments) |
| **Fragment Offset** | 13 bits | Position of this fragment in the original packet |
| **TTL** | 8 bits | Time To Live — decremented by 1 at each router; dropped at 0 |
| **Protocol** | 8 bits | Protocol of the payload — `6`=TCP, `17`=UDP, `1`=ICMP |
| **Header Checksum** | 16 bits | Error detection for the header |
| **Source IP** | 32 bits | IP address of the sender |
| **Destination IP** | 32 bits | IP address of the intended recipient |

### TTL — Time To Live

TTL prevents packets from looping forever on a network. Each router that forwards a packet decrements the TTL by 1. When it hits 0, the router discards the packet and sends an ICMP "Time Exceeded" message back to the sender.

```
PC ──(TTL=64)──► Router1 ──(TTL=63)──► Router2 ──(TTL=62)──► Destination
```

TTL is also used by `traceroute` to map network paths — it sends packets with TTL=1, 2, 3... and collects the ICMP responses from each router.

### Protocol Field
The Protocol field tells the receiving device what's inside the packet:

| Value | Protocol |
|-------|---------|
| 1 | ICMP |
| 6 | TCP |
| 17 | UDP |
| 47 | GRE |
| 50 | ESP (IPSec) |
| 89 | OSPF |

---

## Anatomy of a Frame

A frame wraps a packet for delivery across a single network link. The most common frame type is an **Ethernet frame**.

### Ethernet Frame Structure

```
┌───────────┬──────────┬──────────────┬──────────────┬──────┬──────────────┬─────┐
│ Preamble  │   SFD    │  Dest MAC    │  Source MAC  │ Type │   Payload    │ FCS │
│ (7 bytes) │ (1 byte) │  (6 bytes)   │  (6 bytes)   │(2 B) │ (46–1500 B)  │(4 B)│
└───────────┴──────────┴──────────────┴──────────────┴──────┴──────────────┴─────┘
```

### Ethernet Frame Fields Explained

| Field | Size | Description |
|-------|------|------------|
| **Preamble** | 7 bytes | Alternating 1s and 0s — synchronises the receiver's clock |
| **SFD** | 1 byte | Start Frame Delimiter — signals the actual frame is beginning |
| **Destination MAC** | 6 bytes | MAC address of the intended recipient on this link |
| **Source MAC** | 6 bytes | MAC address of the sending device |
| **EtherType / Length** | 2 bytes | Identifies the Layer 3 protocol (`0x0800`=IPv4, `0x0806`=ARP, `0x86DD`=IPv6) |
| **Payload** | 46–1500 bytes | The data being carried — typically an IP packet |
| **FCS** | 4 bytes | Frame Check Sequence — CRC value for error detection |

### MTU — Maximum Transmission Unit

The **MTU** is the largest payload an Ethernet frame can carry — **1500 bytes** by default. If a packet is larger than the MTU, it must be **fragmented** into smaller pieces.

- **Jumbo frames** — Some networks support up to 9000 bytes (used in data centres)
- **IP fragmentation** — Handled at Layer 3; fragments are reassembled at the destination
- **Path MTU Discovery** — Mechanism to find the smallest MTU along a path to avoid fragmentation

---

## How Frames Change Hop by Hop

A crucial concept: **frames change at every hop, but packets don't.**

The IP header (packet) keeps the same source and destination IP addresses from end to end. But the Ethernet frame (with MAC addresses) is stripped and rebuilt at every router along the path.

### Example — PC to Web Server

```
PC (192.168.1.10) → Router → ISP Router → Web Server (93.184.216.34)
```

**Hop 1: PC → Router**
```
Frame:  Src MAC: PC's MAC      | Dst MAC: Router's MAC
Packet: Src IP:  192.168.1.10  | Dst IP:  93.184.216.34
```

**Hop 2: Router → ISP Router**
```
Frame:  Src MAC: Router's WAN MAC  | Dst MAC: ISP Router's MAC  ← Frame rebuilt
Packet: Src IP:  192.168.1.10      | Dst IP:  93.184.216.34     ← Packet unchanged
```

**Hop 3: ISP Router → Web Server**
```
Frame:  Src MAC: ISP Router's MAC  | Dst MAC: Web Server's MAC  ← Frame rebuilt again
Packet: Src IP:  192.168.1.10      | Dst IP:  93.184.216.34     ← Still unchanged
```

Each router decapsulates the frame, reads the destination IP, looks up its routing table, and creates a **new frame** for the next hop.

---

## Packet Fragmentation

When a packet is too large for the MTU of a network link, it gets **fragmented** — split into smaller packets, each with its own IP header.

### How Fragmentation Works

Original packet: 4000 bytes (too large for 1500 byte MTU)

```
Fragment 1: Bytes 0–1479     (1480 bytes data + 20 bytes IP header = 1500)
Fragment 2: Bytes 1480–2959  (1480 bytes data + 20 bytes IP header = 1500)
Fragment 3: Bytes 2960–3999  (1040 bytes data + 20 bytes IP header = 1060)
```

Each fragment carries:
- The same **Identification** value (so the receiver knows they belong together)
- A **Fragment Offset** (position in the original packet)
- The **More Fragments (MF) flag** set to 1 (except the last fragment)

Fragments are reassembled only at the **final destination** — not at intermediate routers.

### Fragmentation and Security
- **Teardrop attack** — Sends overlapping fragments that crash systems during reassembly
- **Fragmentation evasion** — Attackers split malicious payloads across fragments to evade IDS/IPS
- Many modern firewalls reassemble fragments before inspection to counter this

---

## ARP — Connecting Packets to Frames

Before a device can wrap a packet in a frame, it needs to know the **MAC address** of the next hop. This is where **ARP (Address Resolution Protocol)** comes in.

### ARP Process

```
PC wants to send a packet to 192.168.1.1 (the router):

1. PC checks ARP cache — is 192.168.1.1 already resolved?
   - If yes → use the cached MAC address
   - If no  → send an ARP request

2. ARP Request (broadcast to FF:FF:FF:FF:FF:FF):
   "Who has 192.168.1.1? Tell 192.168.1.10"

3. Router replies (unicast):
   "192.168.1.1 is at 00:1A:2B:3C:4D:5E"

4. PC caches the result and builds the Ethernet frame:
   Dst MAC: 00:1A:2B:3C:4D:5E
   Src MAC: PC's MAC
```

### ARP Cache
```bash
# View ARP cache on Linux/macOS
arp -n

# View ARP cache on Windows
arp -a
```

### ARP Packet Structure
| Field | Size | Description |
|-------|------|------------|
| Hardware type | 2 bytes | Network type (1 = Ethernet) |
| Protocol type | 2 bytes | Protocol (0x0800 = IPv4) |
| Hardware address length | 1 byte | MAC address length (6) |
| Protocol address length | 1 byte | IP address length (4) |
| Operation | 2 bytes | 1 = Request, 2 = Reply |
| Sender MAC | 6 bytes | MAC of the sender |
| Sender IP | 4 bytes | IP of the sender |
| Target MAC | 6 bytes | MAC of the target (zeros in request) |
| Target IP | 4 bytes | IP of the target |

---

## Viewing Packets and Frames in Practice

### Wireshark

Wireshark lets you capture and inspect frames and packets in real time.

**Useful display filters:**
```
# Show all ARP traffic
arp

# Show all ICMP (ping) traffic
icmp

# Show packets to/from a specific IP
ip.addr == 192.168.1.1

# Show only TCP traffic on port 80
tcp.port == 80

# Show only frames with a specific destination MAC
eth.dst == 00:1a:2b:3c:4d:5e

# Show fragmented packets
ip.flags.mf == 1 or ip.frag_offset > 0
```

**What Wireshark shows you:**
```
Frame 1: 74 bytes on wire
├── Ethernet II
│   ├── Destination: 00:1a:2b:3c:4d:5e
│   ├── Source: aa:bb:cc:dd:ee:ff
│   └── Type: IPv4 (0x0800)
├── Internet Protocol Version 4
│   ├── Source: 192.168.1.10
│   ├── Destination: 93.184.216.34
│   ├── TTL: 64
│   └── Protocol: TCP (6)
└── Transmission Control Protocol
    ├── Source Port: 54321
    ├── Destination Port: 80
    └── Flags: SYN
```

### tcpdump
```bash
# Capture all traffic
tcpdump -i eth0

# Show Ethernet frame headers
tcpdump -e -i eth0

# Show packet contents in hex and ASCII
tcpdump -XX -i eth0

# Capture only ICMP
tcpdump -i eth0 icmp

# Capture and save to file
tcpdump -i eth0 -w capture.pcap
```

---

## Packets, Frames and Security

Understanding how packets and frames work is essential for recognising and investigating attacks.

### Packet-Level Attacks

**IP Spoofing**
An attacker forges the source IP address in a packet to hide their identity or impersonate another host.
```
Attacker sends: Src IP: 10.0.0.5 (victim's IP) → Target
Target responds to 10.0.0.5 (the victim) — attacker never receives the reply
Used in: Reflected DDoS attacks, bypass IP-based ACLs
```

**Packet Fragmentation Attacks**
- **Ping of Death** — Sending a fragmented ICMP packet that exceeds 65,535 bytes when reassembled, crashing older systems
- **Teardrop** — Sending fragments with overlapping offsets, confusing the reassembly process
- **Tiny Fragment Attack** — Splitting TCP header across fragments to evade packet filters

**ICMP Tunnelling**
Data is encoded inside ICMP Echo (ping) packets to exfiltrate data or create a covert communication channel, often bypassing firewalls that only filter by port.

### Frame-Level Attacks

**ARP Spoofing / Poisoning**
An attacker sends gratuitous ARP replies, associating their MAC address with a legitimate IP (like the gateway). Devices update their ARP cache and start sending frames to the attacker instead.
```
Attacker broadcasts: "192.168.1.1 is at AA:BB:CC:DD:EE:FF" (attacker's MAC)
Victim updates ARP cache
All victim's traffic now goes to attacker → Man-in-the-Middle
```

**MAC Flooding**
An attacker floods a switch with thousands of frames using random fake source MAC addresses. The switch's MAC address table fills up, causing it to enter **fail-open mode** — broadcasting all frames to all ports like a hub.

**MAC Spoofing**
An attacker changes their NIC's MAC address to impersonate another device — bypassing MAC-based access controls or taking over another device's network identity.

---

## IPv6 Packets

IPv6 uses a different packet header format — simpler in some ways, with a fixed 40-byte header.

### IPv6 Header Structure

```
┌─────────┬──────────────┬───────────────────┬─────────────────┐
│ Version │ Traffic Class│   Flow Label      │                 │
│ (4 bits)│  (8 bits)    │   (20 bits)       │                 │
├─────────┴──────────────┴───────────────────┤  Payload Length │
│                                            │    (16 bits)    │
├────────────────────┬───────────────────────┴─────────────────┤
│  Next Header       │         Hop Limit                       │
│  (8 bits)          │         (8 bits)                        │
├────────────────────┴─────────────────────────────────────────┤
│              Source IPv6 Address (128 bits)                  │
├──────────────────────────────────────────────────────────────┤
│           Destination IPv6 Address (128 bits)                │
└──────────────────────────────────────────────────────────────┘
```

### Key Differences from IPv4

| Feature | IPv4 | IPv6 |
|---------|------|------|
| Header size | Variable (20–60 bytes) | Fixed (40 bytes) |
| Address size | 32 bits | 128 bits |
| Fragmentation | By routers and hosts | By sending host only |
| Checksum | Yes (in header) | No (removed for speed) |
| TTL equivalent | TTL | Hop Limit |
| Protocol field | Protocol | Next Header |
| Broadcast | Yes | No (uses multicast) |
| ARP | Yes | Replaced by NDP (Neighbour Discovery) |

---

## Glossary

| Term | Definition |
|------|-----------|
| **ARP** | Address Resolution Protocol — maps IP addresses to MAC addresses |
| **Broadcast** | Frame sent to all devices on a network (dst MAC: FF:FF:FF:FF:FF:FF) |
| **CRC** | Cyclic Redundancy Check — error detection algorithm used in FCS |
| **Decapsulation** | Stripping headers as data moves up the OSI stack on the receiver |
| **Encapsulation** | Adding headers as data moves down the OSI stack on the sender |
| **EtherType** | Field in an Ethernet frame identifying the Layer 3 protocol |
| **FCS** | Frame Check Sequence — error detection trailer in Ethernet frames |
| **Fragment** | A piece of a larger packet split to fit within the MTU |
| **Frame** | Layer 2 PDU — carries a packet across a single network link |
| **Gratuitous ARP** | Unsolicited ARP reply — used by attackers to poison ARP caches |
| **Header** | Control information prepended to data at each OSI layer |
| **Hop** | Each router a packet passes through on its journey |
| **IHL** | Internet Header Length — size of the IPv4 header |
| **IP Spoofing** | Forging the source IP address in a packet |
| **MAC Address** | 48-bit hardware address used in Ethernet frames |
| **MTU** | Maximum Transmission Unit — largest payload an Ethernet frame carries (1500 bytes) |
| **Packet** | Layer 3 PDU — carries data across networks using IP addresses |
| **Payload** | The actual data carried inside a packet or frame |
| **PDU** | Protocol Data Unit — the data unit at a given OSI layer |
| **Protocol Field** | IPv4 header field indicating the transport protocol (TCP=6, UDP=17) |
| **Segment** | Layer 4 PDU (TCP) — contains port numbers and sequencing |
| **SFD** | Start Frame Delimiter — marks the beginning of an Ethernet frame |
| **TTL** | Time To Live — limits a packet's lifespan by hop count |
| **Unicast** | Frame or packet sent to a single specific destination |

---

*Last updated: 2025 | For the Cybersecurity-Basics knowledge base*