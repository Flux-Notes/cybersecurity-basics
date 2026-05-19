# Intro to LAN

> A beginner-friendly introduction to Local Area Networks — what they are, how they work, and how devices communicate within them.

---

## What is a LAN?

A **Local Area Network (LAN)** is a network of devices connected together within a limited geographic area — such as a home, office, school, or building.

Every device on a LAN can communicate with every other device on that same network. The key word is *local* — a LAN doesn't span cities or countries; that's the job of WANs (Wide Area Networks).

### Everyday Examples
- Your home Wi-Fi network connecting your phone, laptop, and smart TV
- An office network connecting computers, printers, and servers
- A school computer lab where all machines share the same internet connection
- A gaming setup where multiple consoles play together locally

### Key Characteristics of a LAN
- **High speed** — typically 100 Mbps to 10 Gbps on wired connections
- **Low latency** — devices are physically close to each other
- **Private** — not directly accessible from the internet
- **Owned and managed** by the individual or organization using it
- **Uses private IP addresses** (e.g., 192.168.x.x, 10.x.x.x)

---

## How Devices Connect to a LAN

Devices join a LAN either through a **wired (Ethernet)** or **wireless (Wi-Fi)** connection.

### Wired Connection (Ethernet)
- Uses an **Ethernet cable** (Cat5e, Cat6, Cat6a) plugged into a **switch** or **router**
- Faster and more reliable than Wi-Fi
- Common in desktops, servers, printers, and gaming setups

### Wireless Connection (Wi-Fi)
- Uses radio waves to connect to a **wireless access point (WAP)** or router
- More convenient — no cables needed
- Slightly more susceptible to interference and security risks

### The Central Device
All devices on a LAN connect to a central device — typically a **switch** or a **router** (for home networks, these are often combined into one box).

```
Laptop ──────────────┐
Phone (Wi-Fi) ───────┤
Smart TV (Wi-Fi) ────┤──► Router/Switch ──► Internet
Printer ─────────────┤
Desktop ─────────────┘
```

---

## LAN Hardware

### Switch
A **switch** is the core device of a wired LAN. It connects multiple devices and forwards data only to the intended recipient — not to everyone on the network.

- Operates at **Layer 2** (Data Link layer) of the OSI model
- Uses **MAC addresses** to direct traffic
- Maintains a **MAC address table** to learn which device is on which port
- Most home routers have a built-in 4-port switch

### Router
A **router** connects your LAN to other networks — most importantly, the internet.

- Operates at **Layer 3** (Network layer)
- Uses **IP addresses** to route traffic
- Assigns IP addresses to devices via **DHCP**
- Acts as the **default gateway** — all traffic leaving the LAN goes through it
- Performs **NAT** — translating private IPs to your public internet IP

### Access Point (AP)
A **wireless access point** extends the LAN wirelessly. In home setups, the router includes a built-in AP. In larger offices, dedicated APs are deployed throughout the building and connected back to the core switch.

### Network Interface Card (NIC)
Every device that connects to a network has a **NIC** — either built into the motherboard or as an add-on card. The NIC has a unique **MAC address** burned in at the factory.

---

## IP Addressing on a LAN

Every device on a LAN is assigned an **IP address** — a logical address used to identify it on the network.

### Private IP Ranges
LANs use private IP address ranges that are **not routable on the public internet**:

| Range | CIDR | Common Use |
|-------|------|-----------|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | Large corporate networks |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | Medium networks |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | Home and small office |

Most home networks use `192.168.0.x` or `192.168.1.x`.

### Static vs Dynamic IPs

**Dynamic (DHCP)** — The router automatically assigns an IP address when a device joins the network. This is the default for most devices (laptops, phones).

**Static** — Manually configured IP that never changes. Used for devices that need a consistent address — servers, printers, and network devices.

### Typical Home LAN Address Layout
```
Network:         192.168.1.0/24
Router/Gateway:  192.168.1.1
DNS Server:      192.168.1.1  (router handles DNS forwarding)
DHCP Range:      192.168.1.100 – 192.168.1.200
Broadcast:       192.168.1.255

Devices:
  Laptop     → 192.168.1.101  (DHCP)
  Phone      → 192.168.1.102  (DHCP)
  Printer    → 192.168.1.10   (Static)
  NAS/Server → 192.168.1.20   (Static)
```

---

## MAC Addresses on a LAN

While IP addresses identify devices logically, **MAC addresses** identify them physically at the hardware level.

- Every NIC has a unique 48-bit MAC address (e.g., `00:1A:2B:3C:4D:5E`)
- MAC addresses are used by switches to forward frames within the LAN
- They don't leave the local network — routers strip and replace MAC addresses at each hop

### How a Switch Uses MAC Addresses

When a switch receives a frame, it records the source MAC address and the port it arrived on. Over time it builds a complete **MAC address table**:

```
Port  |  MAC Address
------|------------------
  1   |  00:1A:2B:3C:4D:5E  (Laptop)
  2   |  AA:BB:CC:DD:EE:FF  (Desktop)
  3   |  11:22:33:44:55:66  (Printer)
```

When a frame arrives destined for `AA:BB:CC:DD:EE:FF`, the switch sends it only to Port 2 — not to every device. This is far more efficient than a hub (which broadcasts everything to everyone).

---

## How Devices Communicate on a LAN

When Device A wants to send data to Device B on the same LAN, a specific process happens:

### Step 1 — Does A know B's MAC address?
A knows B's IP address but needs its MAC address to actually send the frame. It checks its **ARP cache** first.

### Step 2 — ARP Request (if MAC unknown)
A broadcasts an **ARP (Address Resolution Protocol)** request to the entire LAN:
> "Who has IP 192.168.1.102? Tell 192.168.1.101"

### Step 3 — ARP Reply
Device B responds directly to A:
> "192.168.1.102 is at AA:BB:CC:DD:EE:FF"

### Step 4 — Frame is sent
A now sends the Ethernet frame directly to B's MAC address. The switch forwards it to the correct port.

```
Device A                  Switch                  Device B
192.168.1.101             (forwards)              192.168.1.102
00:1A:...                                         AA:BB:...

  |── ARP broadcast ─────────────────────────────► (all ports)
  |◄── ARP reply ──────────────────────────────── |
  |── Ethernet frame (to AA:BB:...) ─────────────►|
```

---

## VLANs — Virtual LANs

A **VLAN** logically divides a single physical LAN into multiple isolated networks. Devices in different VLANs can't communicate directly — even if they're plugged into the same switch.

### Why Use VLANs?
- **Security** — Separate sensitive departments (Finance, HR) from the general network
- **Performance** — Reduce unnecessary broadcast traffic
- **Management** — Group devices by function, not physical location

### Example VLAN Setup
```
Same Physical Switch
├── Ports 1–8   → VLAN 10: Staff Network     (192.168.10.0/24)
├── Ports 9–16  → VLAN 20: Guest Wi-Fi       (192.168.20.0/24)
└── Ports 17–24 → VLAN 30: Security Cameras  (192.168.30.0/24)
```

Guests can browse the internet but can't reach internal staff devices or cameras.

### Inter-VLAN Routing
To allow traffic between VLANs, you need a **Layer 3 device** — either a router (router-on-a-stick setup) or a Layer 3 switch.

---

## DHCP on a LAN

**DHCP (Dynamic Host Configuration Protocol)** is what automatically gives every device an IP address when it joins the network. On a home LAN, the router is the DHCP server.

### What DHCP Assigns
- IP address
- Subnet mask
- Default gateway
- DNS server

### The DORA Process
```
Device joins LAN:

1. DISCOVER  → "Is there a DHCP server? I need an address"
2. OFFER     ← "Here, take 192.168.1.105 — it's yours for 24 hours"
3. REQUEST   → "I'll take 192.168.1.105 please"
4. ACK       ← "Confirmed. Lease granted."
```

### DHCP Lease
IP addresses are leased for a set duration. When the lease expires, the device either renews it or gets a new address. This is why your laptop might have a different IP after being away for a while.

---

## DNS on a LAN

When you type `google.com` into a browser on your LAN, your device needs to resolve that name to an IP address. This is handled by **DNS**.

On most home networks, the router acts as a **DNS forwarder** — it receives DNS queries from local devices and forwards them to an upstream DNS server (like `8.8.8.8` or `1.1.1.1`).

### Local DNS Resolution
Some networks run a **local DNS server** (like Pi-hole) to:
- Block ads and tracking at the DNS level
- Resolve internal hostnames (e.g., `printer.local → 192.168.1.10`)
- Cache frequent lookups for faster responses

---

## LAN Security

LANs are often assumed to be safe because they're "internal" — but this assumption is dangerous. Attackers who gain access to your LAN can cause significant damage.

### Common LAN Attacks

**ARP Spoofing / Poisoning**
An attacker sends fake ARP replies, associating their MAC address with another device's IP. This lets them intercept traffic — a man-in-the-middle attack.
```
Attacker tells Device A: "I'm the router (192.168.1.1)"
Attacker tells Router:   "I'm Device A (192.168.1.101)"
All of A's traffic now flows through the attacker.
```

**Rogue DHCP Server**
An attacker sets up a fake DHCP server. Devices that connect get a malicious gateway or DNS server — redirecting all their traffic.

**MAC Flooding**
An attacker floods a switch with fake MAC addresses, filling the MAC table. The switch falls back to broadcasting all traffic (like a hub), allowing the attacker to capture it.

**VLAN Hopping**
Exploiting misconfigured trunk ports to send traffic into a VLAN the attacker shouldn't have access to.

**Packet Sniffing**
On older hubs (or after a successful ARP poisoning attack), all traffic on the LAN can be captured and read — including unencrypted passwords and data.

### LAN Security Best Practices

| Practice | What it prevents |
|----------|-----------------|
| Use a managed switch with port security | Limits which MACs can use each port |
| Enable Dynamic ARP Inspection (DAI) | Blocks ARP spoofing |
| Use DHCP Snooping | Prevents rogue DHCP servers |
| Segment with VLANs | Limits lateral movement |
| Use WPA3 for Wi-Fi | Protects wireless access |
| Disable unused switch ports | Reduces physical attack surface |
| Use 802.1X authentication | Only authorized devices can join |
| Encrypt sensitive traffic (TLS/HTTPS) | Protects data from sniffing |
| Monitor with a SIEM or IDS | Detects anomalous LAN behaviour |

---

## LAN vs Other Network Types

| Feature | LAN | WAN | MAN |
|---------|-----|-----|-----|
| Geographic scope | Building / campus | Country / global | City-wide |
| Speed | Very high (1–10 Gbps) | Moderate (varies) | Moderate–high |
| Latency | Very low | Higher | Low–moderate |
| Ownership | Private | ISP / leased | Mixed |
| Example | Home network | The internet | City Wi-Fi |

---

## Useful Commands

### Windows
```cmd
# View IP address, subnet, gateway, DNS
ipconfig /all

# Show ARP cache
arp -a

# Ping a device on the LAN
ping 192.168.1.1

# Trace route to a host
tracert 192.168.1.1

# View open ports and connections
netstat -ano
```

### Linux / macOS
```bash
# View network interfaces and IPs
ip a
ifconfig        # older systems

# Show routing table
ip route
route -n

# Show ARP cache
arp -n

# Ping a device
ping 192.168.1.1

# Scan the LAN for live hosts (requires nmap)
nmap -sn 192.168.1.0/24

# Capture LAN traffic (requires tcpdump)
tcpdump -i eth0
```

---

## Glossary

| Term | Definition |
|------|-----------|
| **ARP** | Address Resolution Protocol — resolves IP to MAC on a LAN |
| **Broadcast** | Sending a packet to all devices on the network |
| **DHCP** | Protocol that automatically assigns IP addresses to devices |
| **Default Gateway** | The router's IP — where traffic goes when leaving the LAN |
| **DNS** | Resolves domain names to IP addresses |
| **Ethernet** | Wired LAN standard using cables and switches |
| **Frame** | Layer 2 unit of data — includes source/destination MAC |
| **IP Address** | Logical address identifying a device on a network |
| **LAN** | Local Area Network — devices connected in a small area |
| **MAC Address** | Physical hardware address of a network interface |
| **NAT** | Network Address Translation — maps private IPs to a public IP |
| **NIC** | Network Interface Card — hardware that connects a device to a network |
| **Packet** | Layer 3 unit of data — includes source/destination IP |
| **Router** | Connects the LAN to other networks / the internet |
| **Subnet Mask** | Defines which part of an IP is the network vs host |
| **Switch** | Connects devices within a LAN using MAC addresses |
| **VLAN** | Virtual LAN — logical segmentation of a physical network |
| **WAN** | Wide Area Network — connects LANs across large distances |
| **Wi-Fi** | Wireless LAN technology using radio waves |

---

*Last updated: 2025 | For the Cybersecurity-Basics knowledge base*