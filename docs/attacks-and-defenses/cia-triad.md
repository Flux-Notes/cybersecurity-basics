# CIA Triad

The CIA Triad is the foundational model of information security. Every security control, policy, and decision ultimately maps back to protecting one or more of its three properties: **Confidentiality**, **Integrity**, and **Availability**. Understanding the triad deeply — including how its components conflict, how they are attacked, and how they are defended — is essential for any security practitioner.

---

## 1. Overview

```
              Confidentiality
                    ▲
                   / \
                  /   \
                 /     \
                /  CIA  \
               /  Triad  \
              /___________\
    Integrity               Availability
```

| Property | Core Question | Violated By |
|----------|--------------|-------------|
| **Confidentiality** | Is information accessible only to those authorized to see it? | Unauthorized disclosure, eavesdropping, data leaks |
| **Integrity** | Is information accurate, complete, and unaltered? | Unauthorized modification, corruption, tampering |
| **Availability** | Is information and its systems accessible when needed? | Denial of service, ransomware, hardware failure |

Each property is equally important. A system that is confidential and available but lacks integrity is dangerous — users cannot trust the data they receive. A system that is confidential and has integrity but is unavailable is useless in an emergency.

---

## 2. Confidentiality

### Definition

Confidentiality ensures that information is accessible only to those who are authorized to access it. It protects data from unauthorized disclosure — whether intentional (theft) or accidental (misconfiguration).

### What Confidentiality Protects Against

- Unauthorized users reading sensitive files
- Network eavesdropping on unencrypted traffic
- Data exfiltration by attackers or malicious insiders
- Accidental exposure through misconfigured permissions or cloud storage
- Shoulder surfing, social engineering, and physical access

### Mechanisms That Enforce Confidentiality

**Encryption**

Transforms data into an unreadable form; only parties with the correct key can decrypt it.

| Type | Description | Examples |
|------|-------------|---------|
| **Symmetric** | Same key encrypts and decrypts | AES-256, ChaCha20 |
| **Asymmetric** | Public key encrypts; private key decrypts | RSA, ECC |
| **Hashing** | One-way transformation; no decryption | SHA-256, bcrypt |
| **At rest** | Encrypts stored data | BitLocker, LUKS, AES-encrypted databases |
| **In transit** | Encrypts data over a network | TLS 1.3, SSH, IPsec |
| **End-to-end** | Encrypted from sender to recipient; intermediaries cannot read | Signal, PGP, WhatsApp |

**Access Controls**

Restrict who can read data:
- File system permissions (Linux `chmod`, Windows ACLs)
- Role-Based Access Control (RBAC) — users access only what their role requires
- Need-to-know principle — access granted only for job-relevant data
- Database column/row-level security

**Authentication & Authorization**

- Strong authentication (MFA) verifies identity before granting access
- Authorization systems (IAM, ABAC) enforce what authenticated users can read

**Data Classification**

Organize data by sensitivity to apply appropriate controls:

| Level | Description | Examples |
|-------|-------------|---------|
| **Public** | Intentionally available to anyone | Press releases, marketing |
| **Internal** | For employees only; not sensitive | Internal wiki, org charts |
| **Confidential** | Sensitive; limited distribution | Financial reports, HR data |
| **Restricted / Secret** | Highly sensitive; strictly controlled | Trade secrets, PII, credentials |

**Other Controls**
- **DLP (Data Loss Prevention)** – Detect and block unauthorized data transfers
- **Data masking** – Replace sensitive values with realistic but fake data (e.g. in dev/test environments)
- **Tokenization** – Replace sensitive data with a non-sensitive token (common in payment processing)
- **Physical security** – Locked server rooms, clean-desk policy, screen privacy filters
- **Network segmentation** – Isolate sensitive systems; limit who can reach them

### Attacks Against Confidentiality

| Attack | Description |
|--------|-------------|
| **Eavesdropping / Sniffing** | Capturing network traffic (e.g. Wireshark on unencrypted network) |
| **Man-in-the-Middle (MitM)** | Intercepting communication between two parties |
| **Credential theft** | Stealing passwords, tokens, or keys to impersonate authorized users |
| **Data exfiltration** | Copying and transmitting sensitive data to an attacker-controlled location |
| **Insecure storage** | Plaintext passwords in config files, unencrypted databases |
| **Side-channel attacks** | Inferring data from timing, power consumption, electromagnetic emissions |
| **Social engineering** | Manipulating people into disclosing sensitive information |
| **Broken access control** | Accessing data via misconfigured permissions or application logic flaws |
| **Cryptographic attacks** | Exploiting weak algorithms, short keys, or poor implementation (e.g. RC4, MD5) |
| **Physical theft** | Stealing an unencrypted device to access its data |

### Real-World Confidentiality Failures

- **Equifax (2017)** — 147 million SSNs, birthdates, and addresses exposed due to unpatched Apache Struts and lack of encryption on internal traffic
- **Capital One (2019)** — Misconfigured WAF + SSRF allowed attacker to access 100M+ customer records from cloud storage
- **Facebook/Cambridge Analytica (2018)** — Personal data of 87M users accessed beyond what users authorized; access control failure
- **Verkada (2021)** — Hardcoded admin credentials exposed 150,000 security camera feeds

---

## 3. Integrity

### Definition

Integrity ensures that information is accurate, complete, and has not been modified in an unauthorized or undetected manner. It applies to data at rest, data in transit, and the systems themselves (software integrity).

### What Integrity Protects Against

- Unauthorized modification of files or databases
- Tampering with data in transit (e.g. injecting commands into a session)
- Malware corrupting or altering system files
- Accidental data corruption from hardware faults or software bugs
- Replay attacks (reusing legitimate data out of context)
- Repudiation (denying an action was performed)

### Mechanisms That Enforce Integrity

**Hashing**

A cryptographic hash function produces a fixed-length digest that uniquely represents input data. Any change to the data — even a single bit — produces a completely different hash.

```
SHA-256("The quick brown fox") = d7a8fbb307d7809469ca9abcb0082e4f8d5651e46d3cdb762d02d0bf37c9e592
SHA-256("The quick brown Fox") = 7d38b5cd25a2baf85ad3bb5b9311383e671a8a142eb302b324d4a5fba8748c69
                                           ↑ one character changed; completely different hash
```

Uses:
- File integrity verification (verify a download wasn't tampered with)
- Password storage (compare hashes, never plaintext)
- Digital signatures (sign hash of a document)
- Blockchain (each block contains hash of previous block)

**Message Authentication Codes (MAC)**

A keyed hash — like a hash, but requires knowledge of a secret key to produce and verify. Provides both integrity and authenticity.

- **HMAC-SHA256** – Most common; used in JWTs, API authentication
- Without the key, an attacker cannot forge a valid MAC even knowing the message

**Digital Signatures**

Asymmetric cryptography applied to a hash:
1. Sender hashes the message
2. Sender encrypts hash with their private key (the signature)
3. Recipient decrypts signature with sender's public key, compares to their own hash of the message
4. If they match → message is unmodified AND came from the private key holder

Uses: Code signing, email signing (S/MIME, PGP), TLS certificates, document signing

**File Integrity Monitoring (FIM)**

Continuously monitors critical files and alerts on unauthorized changes:
- Linux: AIDE, Tripwire, auditd with `-w` watches
- Windows: Sysmon (Event IDs 11, 2), Windows Defender for Endpoint FIM

**Version Control**

Every commit is cryptographically hashed (Git uses SHA-1/SHA-256). The history is tamper-evident — modifying a past commit changes all subsequent commit hashes.

**Access Controls**

Prevent unauthorized writes:
- Read-only permissions for accounts that don't need to modify data
- Separation of duties — the person who creates data cannot approve it (financial controls)
- Immutable storage — cloud write-once storage for logs and audit trails

**Input Validation & Sanitization**

Prevents integrity attacks through application logic:
- Reject malformed or unexpected input
- Parameterized queries prevent SQL injection (which can modify database integrity)
- Enforce data types, ranges, and formats

**Checksums & Error Detection**

Detect accidental corruption (not adversarial):
- CRC32, Adler-32 — fast; used in network protocols, archives
- Not cryptographically secure — cannot detect intentional tampering

### Attacks Against Integrity

| Attack | Description |
|--------|-------------|
| **SQL Injection** | Modify database contents by injecting SQL via user input |
| **Man-in-the-Middle** | Intercept and alter data in transit |
| **Replay attack** | Re-send a previously captured valid message to trigger unintended actions |
| **Rootkit** | Modify OS files, binaries, or kernel to hide attacker presence |
| **Malware / ransomware** | Encrypt, corrupt, or modify files |
| **Supply chain attack** | Compromise software at source — malicious updates distributed to users |
| **DNS cache poisoning** | Corrupt DNS cache to redirect users to malicious sites |
| **Session hijacking** | Take over a session and inject malicious data |
| **Bit-flipping attack** | Flip bits in encrypted data to change plaintext in predictable ways (CBC mode) |
| **Hash collision** | Find two inputs with the same hash; used to substitute malicious content |
| **Insider threat** | Authorized user deliberately modifies data without authorization |

### Real-World Integrity Failures

- **SolarWinds (2020)** — Attackers modified the SolarWinds Orion build system; malicious code compiled into a legitimate, signed software update distributed to 18,000+ organizations
- **NotPetya (2017)** — Distributed via a trojanized update to M.E.Doc accounting software; modified MBR to destroy data
- **SWIFT Banking Attacks (2016)** — Attackers modified SWIFT transaction files and deleted audit logs to cover theft of $81M from Bangladesh Bank
- **Stuxnet (2010)** — Modified PLC firmware to cause physical damage to centrifuges while reporting normal operation to operators

---

## 4. Availability

### Definition

Availability ensures that information systems, data, and services are accessible and functional when authorized users need them. It encompasses both preventing outages and ensuring rapid recovery when they occur.

### What Availability Protects Against

- Denial of service attacks (network and application layer)
- Ransomware encrypting data and systems
- Hardware failure (disk crash, power outage)
- Software bugs, crashes, and resource exhaustion
- Accidental deletion or corruption
- Natural disasters affecting data centers

### Mechanisms That Enforce Availability

**Redundancy**

Eliminate single points of failure at every layer:

| Layer | Redundancy Mechanism |
|-------|---------------------|
| **Power** | UPS, dual power supplies, generator backup |
| **Network** | Multiple ISPs, redundant switches/routers, link bonding |
| **Storage** | RAID arrays, replicated storage, cloud object storage |
| **Servers** | Clustering, active-passive or active-active failover |
| **Application** | Load balancers, multiple instances, auto-scaling |
| **Data center** | Geographically separated sites (hot/warm/cold standby) |

**Backups**

The last line of defense against data loss:

**3-2-1 Backup Rule:**
```
3 copies of data
2 different storage media types
1 copy offsite (geographically separated)
```

**Modern extension — 3-2-1-1-0:**
```
3 copies
2 different media
1 offsite
1 air-gapped or immutable (ransomware protection)
0 errors — backups are tested and verified
```

Backup types:
- **Full** – Complete copy of all data; slowest; most storage
- **Incremental** – Only changes since last backup; fastest; requires full + all incrementals to restore
- **Differential** – Changes since last full backup; balance between full and incremental

**Disaster Recovery (DR)**

| Metric | Definition |
|--------|-----------|
| **RTO (Recovery Time Objective)** | Maximum acceptable time to restore service after an outage |
| **RPO (Recovery Point Objective)** | Maximum acceptable data loss measured in time (e.g. restore to a point 1 hour ago) |

DR site types:
- **Hot site** – Fully operational; failover in minutes; highest cost
- **Warm site** – Hardware ready but not fully operational; failover in hours
- **Cold site** – Space and power only; hardware must be sourced; failover in days

**DDoS Protection**

- **Rate limiting** – Limit requests per IP per second
- **Anycast diffusion** – Distribute attack traffic across global PoPs (Cloudflare, Akamai)
- **Traffic scrubbing** – Route traffic through a scrubbing center; malicious traffic dropped
- **BGP blackholing** – Null-route an attacked IP to sacrifice it and protect surrounding infrastructure
- **CAPTCHAs** – Distinguish bots from humans at the application layer
- **WAF** – Block known attack patterns at the HTTP layer

**High Availability (HA)**

Measured in "nines":

| Availability | Downtime per Year |
|-------------|------------------|
| 99% (two nines) | 3.65 days |
| 99.9% (three nines) | 8.76 hours |
| 99.99% (four nines) | 52.6 minutes |
| 99.999% (five nines) | 5.26 minutes |

Achieved through: redundant hardware, automatic failover, health checks, rolling updates, chaos engineering (deliberately injecting failures to test resilience).

**Capacity Planning**

- Monitor resource utilization trends; provision ahead of demand
- Auto-scaling (cloud) — automatically add instances under load
- Resource limits and quotas — prevent one tenant/user from exhausting shared resources

**Patch & Change Management**

- Unpatched vulnerabilities lead to compromises that take systems offline
- Poor change management causes accidental outages (misconfiguration is the #1 cause of cloud outages)
- Change windows, rollback plans, and staged deployments protect availability

### Attacks Against Availability

| Attack | Description |
|--------|-------------|
| **DDoS (Volumetric)** | Flood target with traffic exceeding its capacity (UDP flood, ICMP flood, amplification) |
| **DDoS (Protocol)** | Exploit protocol weaknesses (SYN flood, Smurf attack, ping of death) |
| **DDoS (Application Layer)** | HTTP flood targeting application logic; harder to distinguish from legitimate traffic |
| **Ransomware** | Encrypt files/systems; demand payment for decryption key |
| **Wiper malware** | Destroy data with no recovery option (NotPetya, Shamoon) |
| **Resource exhaustion** | Consume CPU, memory, disk, or connections until service collapses |
| **Fork bomb** | Rapidly create processes until the system runs out of resources |
| **Logic bomb** | Malicious code that triggers on a date/condition and destroys data |
| **Physical destruction** | Damage hardware, cut cables, disable cooling |
| **DNS attack** | Take down DNS infrastructure to make services unreachable by name |
| **BGP hijacking** | Redirect network traffic away from legitimate destination |

### Real-World Availability Failures

- **Dyn DDoS (2016)** — Mirai botnet (IoT devices) launched massive DDoS against Dyn DNS; took down Twitter, Netflix, Reddit, and many others for hours
- **Colonial Pipeline (2021)** — Ransomware caused the company to shut down pipeline operations proactively; fuel shortages along US East Coast
- **AWS us-east-1 (2021)** — AWS outage took down Alexa, Ring, Disney+, and thousands of other services for hours; single-region dependency exposed
- **CrowdStrike (2024)** — Faulty content update caused 8.5 million Windows systems to BSOD; largest IT outage in history; not a cyberattack but an availability failure

---

## 5. The Tensions Between CIA Properties

The three properties often conflict. Security engineers must make deliberate tradeoffs:

### Confidentiality vs. Availability

```
High Confidentiality → stricter access controls → harder to access → lower Availability
High Availability    → easier access, more redundant copies → higher exposure risk → lower Confidentiality
```

**Example:** Encrypting a database protects confidentiality but adds latency (reduces availability under high load). An emergency medical system may need to prioritize availability so clinicians can access patient records instantly — even at the cost of some confidentiality controls.

### Confidentiality vs. Integrity

```
Homomorphic encryption allows computation on encrypted data (preserving confidentiality)
but it is difficult to verify the integrity of the results without decryption.
```

**Example:** End-to-end encrypted messaging (Signal, WhatsApp) protects confidentiality but makes it impossible for the platform to scan messages for malware or CSAM — creating a tension with the integrity/safety of the platform ecosystem.

### Integrity vs. Availability

```
Strong integrity checks (cryptographic verification on every access) add overhead
→ reduces throughput and availability under high load
```

**Example:** A content delivery network that cryptographically verifies every cached file for integrity would add significant latency. In practice, checksums are verified on ingestion, then trusted during serving — a tradeoff accepting some integrity risk for availability.

### Tradeoff Decision Framework

When designing a system, ask:
1. **What is the worst outcome if confidentiality fails?** (data exposure)
2. **What is the worst outcome if integrity fails?** (incorrect decisions, corrupted systems)
3. **What is the worst outcome if availability fails?** (service disruption, financial loss, safety risk)
4. **Rank the properties by priority for this system and user.**

**Examples of different priority orderings:**

| System | Priority Order | Rationale |
|--------|---------------|-----------|
| Military intelligence database | Confidentiality → Integrity → Availability | Exposure is catastrophic |
| Stock exchange | Integrity → Availability → Confidentiality | Wrong trade data is catastrophic |
| Emergency services (911) | Availability → Integrity → Confidentiality | Must always be reachable |
| Medical records | Integrity → Confidentiality → Availability | Wrong data kills; exposure also serious |
| Cryptocurrency ledger | Integrity → Availability → Confidentiality | Double-spend is catastrophic |
| Public website | Availability → Confidentiality → Integrity | Uptime is primary concern |

---

## 6. Extensions to the CIA Triad

The original triad is often extended to address limitations:

### Authenticity

Ensures that data or communications genuinely originate from the claimed source and have not been forged. Closely related to integrity but distinct — a message can be unmodified (integrity preserved) but come from an impersonator (authenticity violated).

Mechanisms: Digital signatures, PKI, HMAC, certificate pinning

### Non-Repudiation

Ensures that a party cannot deny having performed an action. Critical in legal, financial, and audit contexts.

Mechanisms: Digital signatures (signing proves key holder signed), audit logs with timestamps, blockchain immutability, legal agreements + authentication records

**Example:** A user digitally signs a financial transaction. Even if they later claim they didn't authorize it, the signature (tied to their private key) provides non-repudiation.

### Privacy

Broader than confidentiality — privacy covers the individual's right to control their personal information and how it is used, not just whether it is disclosed.

Regulations enforcing privacy:
- **GDPR** (EU) — Right to access, erasure, portability; breach notification within 72 hours
- **CCPA** (California) — Right to know, delete, opt-out of sale
- **HIPAA** (US healthcare) — Protected health information (PHI) safeguards
- **PCI-DSS** — Payment card data protection

### Safety

In industrial and embedded systems (ICS/SCADA, medical devices, automotive), safety is a separate concern — systems must operate correctly to avoid physical harm. A safety failure can kill people.

**Example:** Stuxnet violated the integrity of centrifuge control systems, which ultimately became a safety issue — physical destruction of equipment.

### The Parkerian Hexad

An extended model proposed by Donn Parker adding three properties to the CIA Triad:

| Property | Description |
|----------|-------------|
| Confidentiality | As above |
| Integrity | As above |
| Availability | As above |
| **Possession / Control** | Physical control of the data medium, independent of confidentiality |
| **Authenticity** | Verified origin of data |
| **Utility** | Data is useful in its current form (not just available but accessible/interpretable) |

---

## 7. CIA Triad in Security Frameworks

### NIST Cybersecurity Framework (CSF)

The five functions map to CIA:

| Function | CIA Mapping |
|----------|------------|
| **Identify** | Understand assets to protect (all three) |
| **Protect** | Implement controls (all three) |
| **Detect** | Identify CIA violations in progress (all three) |
| **Respond** | React to CIA violations (all three) |
| **Recover** | Restore availability; verify integrity |

### ISO/IEC 27001

The international standard for information security management (ISMS) explicitly defines security as preserving CIA and additionally authenticity and non-repudiation.

Control domains most directly mapped:

| Domain | Primary CIA Property |
|--------|---------------------|
| Access control (A.9) | Confidentiality |
| Cryptography (A.10) | Confidentiality + Integrity |
| Operations security (A.12) | Availability + Integrity |
| Communications security (A.13) | Confidentiality + Integrity |
| Incident management (A.16) | All three |
| Business continuity (A.17) | Availability |

### MITRE ATT&CK

ATT&CK tactics map to CIA violations:

| Tactic | CIA Violated |
|--------|-------------|
| Initial Access | — (precursor) |
| Execution | Integrity (unauthorized code runs) |
| Persistence | Integrity + Availability |
| Privilege Escalation | Confidentiality + Integrity |
| Defense Evasion | Integrity (hiding changes) |
| Credential Access | Confidentiality |
| Discovery | Confidentiality |
| Lateral Movement | Confidentiality + Integrity |
| Collection | Confidentiality |
| Exfiltration | Confidentiality |
| Command & Control | Availability + Integrity |
| Impact | Availability + Integrity |

### Risk Management

Risk is the intersection of threat, vulnerability, and impact on CIA:

```
Risk = Threat × Vulnerability × Impact

Impact is measured in terms of CIA:
  - Confidentiality impact: what data is exposed?
  - Integrity impact: what data is modified/corrupted?
  - Availability impact: what services are disrupted?
```

CVSS v3.1 scores separately rate Confidentiality, Integrity, and Availability impact (None / Low / High) for every vulnerability.

---

## 8. Applying the CIA Triad — Worked Examples

### Example 1: Web Application

| Threat | CIA Property Violated | Control |
|--------|-----------------------|---------|
| SQL injection reading user data | Confidentiality | Parameterized queries, WAF, least-privilege DB account |
| SQL injection modifying records | Integrity | Input validation, parameterized queries, audit logging |
| DDoS attack | Availability | Rate limiting, CDN, DDoS scrubbing service |
| Session token theft (XSS) | Confidentiality | HttpOnly/Secure cookies, CSP, input sanitization |
| Malicious file upload replacing legitimate files | Integrity | File type validation, upload directory separation, FIM |
| Third-party script compromise (Magecart) | Confidentiality + Integrity | Subresource Integrity (SRI) hashes, CSP |

### Example 2: Enterprise Network

| Threat | CIA Property Violated | Control |
|--------|-----------------------|---------|
| Unencrypted email intercepted | Confidentiality | TLS for SMTP, S/MIME email signing/encryption |
| Rogue DHCP server | Integrity + Availability | DHCP snooping on switches |
| Ransomware encrypting file server | Availability + Integrity | Offline backups, EDR, network segmentation |
| Insider exfiltrating data via USB | Confidentiality | DLP, USB device control policies |
| Attacker modifying firewall rules | Integrity | Change management, MFA on admin access, audit logs |
| Power outage taking down servers | Availability | UPS, generator, redundant PDUs |

### Example 3: Cloud Storage (S3 / Blob)

| Threat | CIA Property Violated | Control |
|--------|-----------------------|---------|
| Public bucket with sensitive data | Confidentiality | Block public access by default; SCPs; bucket policies |
| Object tampered in transit | Integrity | HTTPS only; enforce `aws:SecureTransport` condition |
| Object deleted by attacker | Availability | Object versioning, MFA delete, replication |
| Credentials leaked in code repo | Confidentiality | IAM roles (no long-term keys), secret scanning in CI |
| Ransomware deleting all objects | Availability + Integrity | Object Lock (WORM), versioning, cross-account backup |

### Example 4: Database Security

| Threat | CIA Property Violated | Control |
|--------|-----------------------|---------|
| Plaintext PII in database | Confidentiality | Column-level encryption, TDE |
| Direct DB access bypassing application | Confidentiality + Integrity | Network segmentation; DB accessible only from app servers |
| SQL injection modifying records | Integrity | Parameterized queries, stored procedures, audit logging |
| DB server hardware failure | Availability | Read replicas, automatic failover, backups |
| DBA modifying data without approval | Integrity | Separation of duties; database activity monitoring |

---

## 9. CIA Triad Quick Reference

### Summary Table

| Property | Goal | Key Mechanisms | Key Attacks | Key Metrics |
|----------|------|---------------|------------|-------------|
| **Confidentiality** | Only authorized parties can read data | Encryption, access control, DLP, classification | Eavesdropping, credential theft, data exfiltration, MitM | Data breach cost, records exposed |
| **Integrity** | Data is accurate and unmodified | Hashing, digital signatures, FIM, input validation | SQL injection, supply chain, rootkit, replay | Mean time to detect tampering, FIM alert rate |
| **Availability** | Systems accessible when needed | Redundancy, backups, DDoS protection, patching | DDoS, ransomware, resource exhaustion, physical destruction | Uptime %, RTO, RPO, MTTR |

### One-Line Definitions

- **Confidentiality** — Keeping secrets secret
- **Integrity** — Keeping data trustworthy
- **Availability** — Keeping systems running

### Common Exam Traps

| Scenario | CIA Property |
|----------|-------------|
| Attacker reads an encrypted file they shouldn't | Confidentiality |
| Attacker modifies a database record | Integrity |
| DDoS takes a website offline | Availability |
| Ransomware encrypts all files | **Availability** (primary) + Integrity |
| Attacker intercepts unencrypted credentials | Confidentiality |
| Malware modifies system binaries | Integrity |
| Flood of fake accounts exhausts server resources | Availability |
| An employee leaks customer data to a competitor | Confidentiality |
| A hash mismatch is detected on a downloaded file | Integrity |
| A backup is found to be unrestorable | Availability |
| Weak cipher allows decryption of old traffic | Confidentiality |
| Log files are deleted to cover an attacker's tracks | Integrity + Availability |
| An insider sells access credentials | Confidentiality |

---

## Further Reading

- [NIST SP 800-33 – Underlying Technical Models for Information Technology Security](https://csrc.nist.gov/publications/detail/sp/800-33/final)
- [ISO/IEC 27001 – Information Security Management](https://www.iso.org/isoiec-27001-information-security.html)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [CVSS v3.1 Specification](https://www.first.org/cvss/specification-document)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [The Parkerian Hexad](https://en.wikipedia.org/wiki/Parkerian_Hexad)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)