# DNS (Domain Name System)

---

## What is DNS?

**DNS (Domain Name System)** is the internet's distributed naming system that translates human-readable domain names (e.g., `google.com`) into machine-readable IP addresses (e.g., `142.250.190.14`). It is often called the **"phone book of the internet"**.

Without DNS, users would need to memorize IP addresses to visit websites or send emails.

- Defined in **RFC 1034** and **RFC 1035** (1987).
- Operates on **UDP port 53** (primary) and **TCP port 53** (for large responses, zone transfers).
- DNS is a **hierarchical, distributed, and decentralized** system.

---

## Why DNS Matters in Cybersecurity

DNS is a fundamental protocol — and a high-value target:

- Almost every network action starts with a DNS query.
- Attackers abuse DNS for **C2 communication**, **data exfiltration**, **phishing**, and **traffic hijacking**.
- Defenders use DNS as a **detection and enforcement layer** (blocking malicious domains, monitoring for tunneling).

---

## DNS Hierarchy

DNS is organized as an **inverted tree** structure:

```
                        . (Root)
                       / \
                     com  org  net  io  ...  (TLD)
                    /   \
               google  amazon  ...           (Second-Level Domain)
               /
            mail.google.com                  (Subdomain / FQDN)
```

### Levels Explained

| Level | Example | Description |
|---|---|---|
| **Root** | `.` | Top of the hierarchy; 13 root server clusters worldwide |
| **TLD (Top-Level Domain)** | `.com`, `.org`, `.uk` | Managed by IANA / registries |
| **Second-Level Domain (SLD)** | `google`, `github` | Registered by organizations |
| **Subdomain** | `mail`, `www`, `api` | Configured by domain owner |
| **FQDN** | `mail.google.com.` | Fully Qualified Domain Name — absolute address (trailing dot = root) |

---

## DNS Components

### DNS Resolver (Recursive Resolver)

- The **client-side component** that does the work of resolving a query.
- Usually provided by ISP, or a public resolver (e.g., `8.8.8.8`, `1.1.1.1`).
- Caches results to speed up future queries.
- Also called a **recursive nameserver** or **full-service resolver**.

### Root Nameservers

- 13 logical root servers (labeled `a.root-servers.net` through `m.root-servers.net`).
- Operated by 12 organizations (e.g., Verisign, ICANN, NASA).
- Physically replicated via **anycast** — hundreds of instances worldwide.
- Do **not** answer queries directly; they **delegate** to TLD nameservers.

### TLD Nameservers

- Managed by **registries** (e.g., Verisign manages `.com`).
- Know which **authoritative nameservers** are responsible for each domain under their TLD.

### Authoritative Nameserver

- The **final authority** for a domain's DNS records.
- Answers queries with the actual IP addresses (or other records) for that domain.
- Configured by the **domain owner** (via their registrar or hosting provider).
- There are typically **2+ authoritative nameservers** per domain for redundancy.

### DNS Cache

- **Resolvers, OS, and browsers** all cache DNS responses.
- Reduces load on authoritative servers and speeds up resolution.
- Cached records expire based on their **TTL (Time To Live)** value.

---

## DNS Resolution Process (Step-by-Step)

```
User types: www.example.com

1. Browser Cache
   └── Not found → OS Cache (/etc/hosts, local resolver cache)
       └── Not found → Recursive Resolver (e.g., 8.8.8.8)
           └── Not cached → Root Nameserver (.)
               └── "Ask .com TLD server"
               → TLD Nameserver (.com)
                   └── "Ask ns1.example.com"
                   → Authoritative Nameserver (ns1.example.com)
                       └── Returns: www.example.com → 93.184.216.34
               ← Resolver caches the answer (TTL: 3600s)
   ← Browser connects to 93.184.216.34
```

### Query Types

| Type | Description |
|---|---|
| **Recursive query** | Client asks resolver to fully resolve — resolver does all the work |
| **Iterative query** | Resolver asks each nameserver in turn; each responds with a referral |
| **Non-recursive query** | Resolver already has the answer cached; returns immediately |

---

## DNS Record Types

### Core Records

| Record | Full Name | Description | Example |
|---|---|---|---|
| **A** | Address | Maps hostname → IPv4 address | `example.com → 93.184.216.34` |
| **AAAA** | IPv6 Address | Maps hostname → IPv6 address | `example.com → 2606:2800::1` |
| **CNAME** | Canonical Name | Alias of another domain name | `www → example.com` |
| **MX** | Mail Exchange | Specifies mail server for the domain | `@ → mail.example.com (priority 10)` |
| **NS** | Name Server | Lists authoritative nameservers | `example.com → ns1.example.com` |
| **PTR** | Pointer | Reverse DNS — IP → hostname | `34.216.184.93.in-addr.arpa → example.com` |
| **SOA** | Start of Authority | Zone metadata (serial, refresh, retry, expire, TTL) | Primary NS, admin email |
| **TXT** | Text | Arbitrary text; used for SPF, DKIM, DMARC, verification | `v=spf1 include:_spf.google.com ~all` |
| **SRV** | Service | Service location (host + port) | `_sip._tcp → sip.example.com:5060` |
| **CAA** | Cert Authority Authorization | Which CAs can issue TLS certs for this domain | `0 issue "letsencrypt.org"` |
| **NAPTR** | Naming Authority Pointer | Used in VoIP / ENUM / URI mapping | — |
| **DS** | Delegation Signer | DNSSEC — links child zone to parent zone | — |
| **DNSKEY** | DNS Key | DNSSEC public key for a zone | — |
| **RRSIG** | Resource Record Signature | DNSSEC cryptographic signature | — |
| **NSEC / NSEC3** | Next Secure | DNSSEC authenticated denial of existence | — |

### Email-Related TXT Records

| Record | Purpose | Example value |
|---|---|---|
| **SPF** | Specifies authorized mail senders | `v=spf1 mx include:sendgrid.net ~all` |
| **DKIM** | Public key for email signature verification | `v=DKIM1; k=rsa; p=MIGfMA0G...` |
| **DMARC** | Policy for SPF/DKIM failures | `v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com` |

---

## DNS Zone

A **DNS zone** is a portion of the DNS namespace managed by a specific organization or administrator.

- Defined in a **zone file** — a text file containing DNS records.
- Every zone has exactly one **SOA record** at its apex.

### Zone File Example

```zone
$ORIGIN example.com.
$TTL 3600

@   IN  SOA   ns1.example.com. admin.example.com. (
              2024010101 ; Serial
              3600       ; Refresh
              900        ; Retry
              604800     ; Expire
              300 )      ; Minimum TTL

; Nameservers
@       IN  NS    ns1.example.com.
@       IN  NS    ns2.example.com.

; A Records
@       IN  A     93.184.216.34
www     IN  A     93.184.216.34
mail    IN  A     93.184.216.100

; MX Record
@       IN  MX    10 mail.example.com.

; TXT Record (SPF)
@       IN  TXT   "v=spf1 mx ~all"
```

### Zone Transfer

A **zone transfer** replicates DNS zone data from a **primary** nameserver to **secondary** nameservers.

- **AXFR** — full zone transfer (all records).
- **IXFR** — incremental zone transfer (only changes since last serial).

> **Security risk:** If zone transfers are not restricted, attackers can dump the entire DNS zone and map your infrastructure.

```bash
# Test for open zone transfer (authorized testing only)
dig axfr example.com @ns1.example.com
```

**Mitigation:** Restrict AXFR to trusted secondary IPs only (`allow-transfer` in BIND).

---

## TTL (Time To Live)

**TTL** controls how long a DNS record is cached before it must be refreshed.

| TTL Value | Seconds | Use Case |
|---|---|---|
| 60 | 1 minute | Rapid failover, migrations |
| 300 | 5 minutes | Frequently changing records |
| 3600 | 1 hour | Standard web/email records |
| 86400 | 24 hours | Stable records (NS, MX) |
| 604800 | 1 week | Very static records |

- **Low TTL** → faster propagation of changes, more DNS queries, higher server load.
- **High TTL** → slower propagation, better caching performance.

> **Tip before migrations:** Lower TTL to 300 or 60 several days before making changes, then raise it back after confirming everything works.

---

## Reverse DNS (rDNS)

**Reverse DNS** maps an IP address back to a hostname using **PTR records** stored in special zones.

- IPv4 reverse zone: `in-addr.arpa.`
- IPv6 reverse zone: `ip6.arpa.`

```
Forward: example.com → 93.184.216.34  (A record)
Reverse: 93.184.216.34 → example.com  (PTR record)

Zone: 34.216.184.93.in-addr.arpa → example.com.
```

### Uses of rDNS

- **Email spam filtering** — mail servers verify that the sending IP has a matching PTR record.
- **Network troubleshooting** — tools like `traceroute` display hostnames.
- **Logging** — logs show hostnames instead of raw IPs.
- **Access control** — some services grant access based on rDNS.

---

## DNS Caching

### Where Caching Occurs

| Location | Cache Type | Notes |
|---|---|---|
| **Browser** | Application cache | Short TTL; browser-specific |
| **OS** | System resolver cache | `nscd`, `systemd-resolved` on Linux; Windows DNS Client |
| **Recursive Resolver** | Shared cache | ISP or public resolver (8.8.8.8, 1.1.1.1) |
| **Authoritative Server** | No caching | Serves authoritative answers only |

### Flushing DNS Cache

```bash
# Linux (systemd-resolved)
sudo systemd-resolve --flush-caches

# Linux (nscd)
sudo service nscd restart

# macOS
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder

# Windows
ipconfig /flushdns
```

---

## DNS over HTTPS (DoH)

**DoH** encrypts DNS queries inside HTTPS (port 443), preventing eavesdropping and tampering by ISPs or on-path attackers.

- Defined in **RFC 8484**.
- Queries look like regular HTTPS traffic — hard to block or monitor.
- Supported by Firefox, Chrome, Windows 11, Android.

```
Traditional DNS:  Client → UDP/TCP 53 → Resolver  (plaintext, visible)
DoH:              Client → HTTPS 443 → DoH Resolver  (encrypted)
```

**Providers:** Cloudflare (`1.1.1.1`), Google (`8.8.8.8`), NextDNS, Quad9.

**Security trade-off:** DoH improves privacy from local network observers but moves trust to the DoH provider. Enterprises may block DoH to maintain DNS visibility.

---

## DNS over TLS (DoT)

**DoT** encrypts DNS queries using **TLS** on a dedicated port **853**.

- Defined in **RFC 7858**.
- Easier for network admins to monitor/block (specific port) compared to DoH.
- Provides same encryption benefits as DoH.

| | DoH | DoT |
|---|---|---|
| Port | 443 | 853 |
| Protocol | HTTPS | TLS |
| Detectability | Hard (blends with HTTPS) | Easy (dedicated port) |
| Enterprise use | Problematic for monitoring | Preferred |

---

## DNSSEC (DNS Security Extensions)

**DNSSEC** adds **cryptographic signatures** to DNS records, allowing resolvers to verify that responses are authentic and have not been tampered with.

- Does **not** encrypt DNS queries (use DoH/DoT for that).
- Prevents **DNS spoofing / cache poisoning**.
- Defined in **RFC 4033–4035**.

### How DNSSEC Works

```
Zone signs all records with a private key (ZSK - Zone Signing Key)
↓
Resolver verifies signature using public key (DNSKEY record)
↓
Chain of trust established from root → TLD → domain
```

### DNSSEC Record Chain

```
Root (.) → signed DS record → TLD (.com)
TLD (.com) → signed DS record → example.com
example.com → DNSKEY + RRSIG → individual records
```

### DNSSEC Record Types

| Record | Purpose |
|---|---|
| **DNSKEY** | Public key for the zone |
| **RRSIG** | Signature over a resource record set |
| **DS** | Hash of child zone's DNSKEY (in parent zone) |
| **NSEC** | Authenticated denial of existence (enumeratable) |
| **NSEC3** | Hashed denial of existence (prevents zone enumeration) |

### DNSSEC Limitations

- Complex to deploy and manage (key rollovers, misconfigurations break DNS).
- Does not protect against all attacks (DDoS, resolver hijacking).
- Low adoption compared to what it could be.

---

## Public DNS Resolvers

| Provider | IPv4 | IPv6 | Features |
|---|---|---|---|
| **Google** | `8.8.8.8`, `8.8.4.4` | `2001:4860:4860::8888` | Fast, reliable |
| **Cloudflare** | `1.1.1.1`, `1.0.0.1` | `2606:4700:4700::1111` | Privacy-focused, fast |
| **Quad9** | `9.9.9.9`, `149.112.112.112` | `2620:fe::fe` | Malware blocking |
| **OpenDNS** | `208.67.222.222`, `208.67.220.220` | — | Filtering, parental controls |
| **NextDNS** | Customizable | Customizable | Custom blocklists, logging |
| **AdGuard DNS** | `94.140.14.14` | `2a10:50c0::ad1:ff` | Ad/tracker blocking |

---

## DNS Tools

### Query & Lookup

```bash
# Basic lookup
dig example.com
dig example.com A
dig example.com MX
dig example.com TXT

# Query a specific nameserver
dig @8.8.8.8 example.com

# Reverse DNS lookup
dig -x 93.184.216.34

# Get all records
dig example.com ANY

# Short output
dig +short example.com

# Trace full resolution path
dig +trace example.com

# nslookup (interactive)
nslookup example.com
nslookup example.com 8.8.8.8

# host (simple)
host example.com
host 93.184.216.34
```

### DNS Enumeration (Authorized Testing)

```bash
# Subdomain brute-forcing
dnsx -d example.com -w wordlist.txt
amass enum -d example.com
subfinder -d example.com

# Zone transfer test
dig axfr example.com @ns1.example.com

# DNS reconnaissance
dnsrecon -d example.com
fierce --domain example.com

# Certificate transparency (passive subdomain discovery)
curl "https://crt.sh/?q=%.example.com&output=json" | jq '.[].name_value'
```

### DNS Monitoring & Security

```bash
# Capture DNS traffic
tcpdump -i eth0 port 53

# Analyze DNS in Wireshark
# Filter: dns

# Check DNSSEC validation
dig +dnssec example.com
delv @8.8.8.8 example.com

# Check SPF record
dig example.com TXT | grep spf

# Check DMARC
dig _dmarc.example.com TXT

# Check DKIM
dig selector._domainkey.example.com TXT
```

---

## DNS Attacks

### 1. DNS Cache Poisoning / Spoofing

**What it is:** Attacker injects forged DNS records into a resolver's cache, redirecting users to malicious IPs.

**How it works:**
```
1. Attacker sends a flood of forged DNS responses to a resolver
2. Forged response matches a pending query (correct TxID guessed)
3. Resolver caches the malicious record
4. All users of that resolver are redirected to attacker's IP
```

**Impact:** Credential theft, malware delivery, traffic interception.

**Mitigations:**
- DNSSEC (cryptographic validation)
- DNS resolvers use **random source ports** + **random transaction IDs** (makes spoofing harder)
- **0x20 encoding** (randomize case in query to make matching harder)

---

### 2. DNS Hijacking

**What it is:** Attacker redirects DNS queries to a malicious resolver.

**Variants:**
| Type | Method |
|---|---|
| **Local hijacking** | Malware modifies `/etc/resolv.conf` or registry DNS settings |
| **Router hijacking** | Attacker changes DNS settings on a compromised router |
| **Rogue DNS server** | ISP or government redirects all DNS traffic |
| **Registrar hijacking** | Attacker takes over domain registration and changes NS records |

**Mitigations:** DoH/DoT, monitor DNS settings, use registrar lock, MFA on registrar accounts.

---

### 3. DNS Amplification Attack (DDoS)

**What it is:** Attacker uses open DNS resolvers to amplify DDoS traffic toward a victim.

**How it works:**
```
Attacker (spoofed src IP = victim) → Open Resolver: "give me ALL records for example.com"
Open Resolver → Victim: [large DNS response, 50-100x amplification]
```

**Amplification factor:** A small query (~40 bytes) can generate a response of 3,000–4,000 bytes (ANY query).

**Mitigations:**
- Disable open recursion (only allow queries from trusted clients)
- Rate limit DNS responses
- BCP38 — ISPs should block spoofed source IPs

---

### 4. DNS Tunneling

**What it is:** Attacker encodes data (C2 commands, exfiltrated data) inside DNS queries and responses to bypass firewalls.

**How it works:**
```
Malware encodes data → subdomain of attacker's domain
e.g., aGVsbG8gd29ybGQ.evil.com TXT query
Attacker's authoritative server decodes the data
Response encodes reply in TXT/CNAME answer
```

**Tools used by attackers:** `iodine`, `dnscat2`, `dns2tcp`

**Detection indicators:**
- Unusually **long subdomain names**
- High volume of DNS queries to a single domain
- High entropy in subdomain labels
- Unusual record types (TXT, NULL, CNAME used for data)
- DNS queries with no matching HTTP/HTTPS traffic

**Mitigations:**
- DNS traffic monitoring and anomaly detection
- Block known tunneling tools' signatures
- RPZ (Response Policy Zones) to block malicious domains
- Limit DNS to internal resolvers only

---

### 5. NXDOMAIN Attack

**What it is:** Flood of queries for non-existent domains to exhaust resolver resources.

**Impact:** Resolver CPU/memory exhaustion; legitimate queries time out.

**Mitigations:** Rate limiting (RRL — Response Rate Limiting), anycast, filtering.

---

### 6. Fast Flux DNS

**What it is:** Attacker rapidly rotates IP addresses (and sometimes nameservers) in DNS records with very low TTLs to evade blocklists.

**Types:**
- **Single flux** — A records change rapidly (many IPs for the same domain).
- **Double flux** — Both A records **and NS records** rotate.

**Use cases for attackers:** Phishing sites, botnet C2, spam infrastructure.

**Detection:** Very low TTL (< 300s), large number of IPs per domain, IPs in many different ASNs.

---

### 7. Domain Generation Algorithms (DGA)

**What it is:** Malware generates **pseudorandom domain names** algorithmically. Only the attacker knows which domain is valid on a given day.

```
Day 1: aklsdjfh3.com
Day 2: mnvbqwer7.com
Day 3: zxcvpoiu2.com
```

**Why it works:** Defenders can't easily blocklist thousands of future domains.

**Detection:** High-entropy domain names, NXDOMAINs in bulk, domains with no web content or history.

**Mitigations:** ML-based DNS analytics, threat intel feeds, sinkholing known DGA families.

---

### 8. BGP Hijacking (DNS Impact)

When BGP routes are hijacked, DNS traffic destined for legitimate resolvers can be intercepted and answered by attacker-controlled servers. This is a high-sophistication attack typically performed by nation-state actors or malicious ISPs.

---

### 9. Subdomain Takeover

**What it is:** A DNS record (usually CNAME) points to an external service (e.g., AWS S3, GitHub Pages) that has been **deleted**. Attacker claims the resource and hosts malicious content under the victim's subdomain.

```
CNAME: blog.victim.com → victim.github.io  (GitHub Pages deleted)
Attacker claims victim.github.io → hosts phishing page at blog.victim.com
```

**Mitigations:** Audit DNS records regularly, remove stale CNAMEs pointing to deprovisioned services.

---

## DNS Security Controls

### Response Policy Zones (RPZ)

**RPZ** allows DNS resolvers to **override responses** for specific domains based on threat intelligence feeds — effectively a DNS-layer firewall.

```
Query: malware-c2.example.com
RPZ rule: NXDOMAIN (blocked)
Response: NXDOMAIN — domain does not exist
```

**RPZ actions:** NXDOMAIN, NODATA, PASSTHRU, DROP, redirect to sinkhole.

**Sources:** Infoblox, Spamhaus, ISC, commercial threat intel feeds.

---

### DNS Sinkholing

A **sinkhole** redirects malicious domain queries to a controlled IP (the sinkhole server) instead of the real C2 infrastructure.

- Breaks malware communication.
- Provides **visibility** — which hosts queried the malicious domain.
- Used by security researchers, ISPs, and law enforcement.

---

### DANE (DNS-based Authentication of Named Entities)

**DANE** uses DNSSEC-signed **TLSA records** to publish TLS certificate information in DNS, allowing clients to verify certificates without relying solely on Certificate Authorities.

```
TLSA record: _443._tcp.example.com → SHA-256 hash of cert
Client checks: does the presented TLS cert match the TLSA record?
```

---

### DNS Firewall

A DNS firewall (offered by Cloudflare Gateway, Cisco Umbrella, NextDNS, etc.) blocks DNS queries to:

- Known malware/C2 domains
- Phishing sites
- Newly registered domains
- Domains with bad reputation

It acts as a **first line of defense** — stopping threats before a TCP connection is even made.

---

## Split-Horizon DNS (Split-Brain DNS)

**Split-horizon DNS** returns **different answers** depending on where the query originates (internal vs external).

```
Internal query:  app.example.com → 10.0.1.50   (private IP)
External query:  app.example.com → 203.0.113.10 (public IP)
```

**Use cases:**
- Hide internal infrastructure from the internet.
- Serve internal users via private IPs for performance.
- Different content for internal vs external users.

**Implementation:** Two separate DNS zones with the same name — one served to internal resolvers, one to external.

---

## DNS in Active Directory

Windows **Active Directory** relies heavily on DNS:

- Domain controllers register **SRV records** so clients can find them.
- AD-integrated zones store DNS in Active Directory itself (replicated automatically).
- Clients use DNS to locate **LDAP, Kerberos, and Global Catalog** services.

```
_ldap._tcp.dc._msdcs.domain.local  SRV  → dc1.domain.local:389
_kerberos._tcp.domain.local        SRV  → dc1.domain.local:88
```

**Security note:** Compromising DNS in an AD environment can redirect authentication traffic and facilitate **MitM attacks** on Kerberos/LDAP.

---

## DNS Configuration Files (Linux/BIND)

### `/etc/resolv.conf` — Resolver configuration

```
nameserver 8.8.8.8
nameserver 8.8.4.4
search example.com
options ndots:5
```

### `/etc/hosts` — Local static DNS

```
127.0.0.1   localhost
192.168.1.10  server1.local server1
10.0.0.5    db.internal
```

### BIND (`named.conf`) — Authoritative nameserver

```
options {
    directory "/var/named";
    allow-query { any; };
    allow-transfer { 192.168.1.2; };  // Restrict zone transfers
    recursion no;                      // Authoritative only
};

zone "example.com" IN {
    type master;
    file "example.com.zone";
    allow-transfer { 192.168.1.2; };
};
```

---

## DNS Troubleshooting Checklist

```
1. Can you reach the DNS server?
   → ping 8.8.8.8

2. Is DNS resolving anything?
   → dig google.com @8.8.8.8

3. Is your specific domain resolving?
   → dig yourdomain.com

4. Is the record correct?
   → dig yourdomain.com A / MX / TXT

5. Is DNS propagation complete?
   → dig @ns1.yourdomain.com yourdomain.com (check authoritative)
   → https://dnschecker.org

6. Is DNSSEC valid?
   → dig +dnssec yourdomain.com
   → delv yourdomain.com

7. Is email configured correctly?
   → dig yourdomain.com MX
   → dig yourdomain.com TXT (SPF)
   → dig _dmarc.yourdomain.com TXT

8. Check TTL / caching
   → dig +nocmd +noall +answer yourdomain.com (shows TTL remaining)

9. Flush local cache and retry
   → sudo systemd-resolve --flush-caches (Linux)
   → ipconfig /flushdns (Windows)

10. Check /etc/hosts for overrides
    → cat /etc/hosts
```

---

## DNS Quick Reference

### Common Record Types

| Record | Maps | Example |
|---|---|---|
| A | Hostname → IPv4 | `example.com → 93.184.216.34` |
| AAAA | Hostname → IPv6 | `example.com → 2606:2800::1` |
| CNAME | Alias → Hostname | `www → example.com` |
| MX | Domain → Mail server | `@ → mail.example.com (10)` |
| TXT | Domain → Text | SPF, DKIM, DMARC, verification |
| NS | Zone → Nameserver | `example.com → ns1.example.com` |
| PTR | IP → Hostname | `34.216.184.93... → example.com` |
| SOA | Zone metadata | Serial, refresh, retry, expire |
| SRV | Service → Host:Port | `_sip._tcp → sip.example.com:5060` |
| CAA | Domain → Allowed CA | `0 issue "letsencrypt.org"` |

### Ports

| Port | Protocol | Use |
|---|---|---|
| 53 | UDP | Standard DNS queries |
| 53 | TCP | Large responses, zone transfers |
| 853 | TCP | DNS over TLS (DoT) |
| 443 | TCP | DNS over HTTPS (DoH) |

### Attack Summary

| Attack | What Happens | Key Defense |
|---|---|---|
| Cache Poisoning | Fake records in resolver cache | DNSSEC |
| DNS Hijacking | Queries redirected to malicious resolver | DoH/DoT, registry lock |
| Amplification DDoS | Open resolver used to flood victim | Disable open recursion |
| DNS Tunneling | Data exfiltrated via DNS | Traffic monitoring, RPZ |
| Fast Flux | IPs rotate to evade blocklists | ML-based analytics |
| DGA | Random domains for C2 | DNS analytics, sinkholing |
| Subdomain Takeover | Stale CNAME claimed by attacker | Audit DNS records |
| Zone Transfer Leak | Full zone dumped by attacker | Restrict AXFR |

---

> **Note:** This document is part of the [Cybersecurity-Basics](../README.md) notes repository. It is intended for learning purposes. Always practice in a legal, authorized environment.

*Last updated: 2026*