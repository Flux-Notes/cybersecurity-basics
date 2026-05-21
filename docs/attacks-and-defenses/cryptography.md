# Cryptography

Cryptography is the science of securing information through mathematical techniques. It is the backbone of modern cybersecurity — every secure connection, stored password, digital signature, and encrypted file relies on cryptographic primitives. Understanding cryptography — both how it works and how it breaks — is essential for any security practitioner.

---

## 1. Foundations

### Core Goals of Cryptography

| Goal | Description | Primitive |
|------|-------------|-----------|
| **Confidentiality** | Only authorized parties can read the data | Symmetric / Asymmetric encryption |
| **Integrity** | Data has not been modified | Cryptographic hash, MAC |
| **Authentication** | Verify the identity of a party | MAC, digital signature, certificates |
| **Non-repudiation** | Sender cannot deny sending | Digital signature |
| **Key exchange** | Securely establish a shared secret over an insecure channel | Diffie-Hellman, ECDH |

### Kerckhoffs's Principle

> *A cryptosystem should be secure even if everything about the system, except the key, is public knowledge.*

Security through obscurity is not security. All modern cryptographic algorithms are public — security relies entirely on the secrecy of the key, not the secrecy of the algorithm.

### Basic Terminology

| Term | Definition |
|------|-----------|
| **Plaintext** | Original, readable data |
| **Ciphertext** | Encrypted, unreadable data |
| **Encryption** | Process of converting plaintext to ciphertext |
| **Decryption** | Process of converting ciphertext back to plaintext |
| **Key** | Secret value that controls encryption/decryption |
| **Cipher** | Algorithm used for encryption/decryption |
| **Cryptanalysis** | Science of breaking cryptographic systems |
| **Cryptology** | Encompassing field of cryptography + cryptanalysis |

### Brief History

| Era | System | Notes |
|-----|--------|-------|
| ~100 BC | Caesar cipher | ROT-3 substitution; trivially broken |
| 1467 | Vigenère cipher | Polyalphabetic; "unbreakable" for 300 years |
| WWII | Enigma machine | Rotor-based; broken by Turing et al. at Bletchley Park |
| 1976 | DES | First public standard symmetric cipher; 56-bit key |
| 1976 | Diffie-Hellman | First public key exchange protocol |
| 1977 | RSA | First practical public key cryptosystem |
| 2001 | AES | Replaced DES; current standard |
| 2005+ | ECC | Elliptic curve cryptography; smaller keys, same security |
| 2020s | Post-quantum | Lattice-based, hash-based cryptography for quantum resistance |

---

## 2. Symmetric Encryption

### How It Works

Both parties share the same secret key. The same key is used to encrypt and decrypt.

```
Plaintext ──[Key + Cipher]──▶ Ciphertext ──[Key + Cipher]──▶ Plaintext
              (encrypt)                        (decrypt)
```

**Advantage:** Fast — suitable for bulk data encryption
**Disadvantage:** Key distribution problem — how do you securely share the key in the first place?

### Stream Ciphers

Encrypt one bit or byte at a time by XORing plaintext with a keystream generated from the key.

```
Keystream:   1 0 1 1 0 1 0 0 1 1 ...
Plaintext:   0 1 1 0 1 0 1 1 0 0 ...
             ─────────────────────
Ciphertext:  1 1 0 1 1 1 1 1 1 1 ...  (XOR of above)
```

| Cipher | Status | Notes |
|--------|--------|-------|
| RC4 | **Broken** | Biases in keystream; prohibited in TLS |
| ChaCha20 | Secure | Fast in software; used in TLS 1.3, WireGuard |
| Salsa20 | Secure | Predecessor to ChaCha20 |
| A5/1 | **Broken** | GSM phone encryption; cryptanalyzed |

**Critical rule:** Never reuse a keystream (nonce reuse). XORing two ciphertexts encrypted with the same keystream cancels the key, exposing the XOR of the two plaintexts.

### Block Ciphers

Encrypt fixed-size blocks of plaintext (e.g. 128 bits) at a time.

| Cipher | Block Size | Key Size | Status |
|--------|-----------|---------|--------|
| DES | 64 bits | 56 bits | **Broken** — 56-bit key exhaustively crackable |
| 3DES (Triple-DES) | 64 bits | 112/168 bits | **Deprecated** — slow; 64-bit block vulnerable to SWEET32 |
| AES-128 | 128 bits | 128 bits | Secure |
| AES-192 | 128 bits | 192 bits | Secure |
| AES-256 | 128 bits | 256 bits | Secure — recommended |
| Blowfish | 64 bits | 32–448 bits | Avoid — 64-bit block; SWEET32 |
| Twofish | 128 bits | 128–256 bits | Secure; AES finalist |
| Camellia | 128 bits | 128–256 bits | Secure; used in some TLS implementations |

### AES Internals

AES (Advanced Encryption Standard) operates on a 4×4 matrix of bytes (state) over multiple rounds:

```
Key expansion → Initial AddRoundKey
    ↓
For each round (9/11/13 rounds for 128/192/256-bit keys):
    SubBytes   → Non-linear substitution (S-Box)
    ShiftRows  → Row rotation
    MixColumns → Column mixing (not in final round)
    AddRoundKey → XOR with round key
    ↓
Final round (no MixColumns) → Ciphertext
```

### Block Cipher Modes of Operation

A block cipher encrypts a single block. Modes define how to encrypt multiple blocks.

**ECB (Electronic Codebook) — DO NOT USE**
```
Each block encrypted independently with the same key.
Identical plaintext blocks → identical ciphertext blocks.
Pattern leakage makes ECB catastrophically insecure for most data.
```

ECB mode famously reveals structure in images (the "ECB penguin" — a bitmap of Tux encrypted with ECB retains the image outline).

**CBC (Cipher Block Chaining)**
```
IV ──XOR──[Encrypt]──▶ C1
C1 ──XOR──[Encrypt]──▶ C2
C2 ──XOR──[Encrypt]──▶ C3
```
- Each block XORed with previous ciphertext before encryption
- Requires an IV (Initialization Vector); must be random and unpredictable
- Vulnerable to **padding oracle attacks** (POODLE, BEAST)
- Decryption is parallelizable; encryption is sequential

**CTR (Counter Mode)**
```
Encrypt(Nonce||Counter_1) ──XOR──▶ C1
Encrypt(Nonce||Counter_2) ──XOR──▶ C2
Encrypt(Nonce||Counter_3) ──XOR──▶ C3
```
- Converts block cipher into stream cipher
- Parallelizable both directions
- Nonce must never be reused with the same key

**GCM (Galois/Counter Mode) — Recommended**
```
CTR mode encryption + GHASH authentication tag
```
- Provides both encryption and authentication (AEAD — Authenticated Encryption with Associated Data)
- Detects tampering — if ciphertext is modified, authentication tag verification fails
- Used in TLS 1.3, AES-GCM-SIV
- Nonce must be unique per encryption operation

**CCM, SIV, OCB** — Other AEAD modes; each with specific use cases and tradeoffs.

**Key insight:** Always use an AEAD mode (GCM, CCM, ChaCha20-Poly1305). Non-AEAD modes provide confidentiality but not integrity — ciphertext can be modified without detection.

---

## 3. Asymmetric (Public Key) Cryptography

### How It Works

Each party has a **key pair**: a public key (shared freely) and a private key (kept secret).

```
Encryption:  Plaintext ──[Recipient's Public Key]──▶ Ciphertext
Decryption:  Ciphertext ──[Recipient's Private Key]──▶ Plaintext

Signing:     Message ──[Sender's Private Key]──▶ Signature
Verification: Message + Signature ──[Sender's Public Key]──▶ Valid / Invalid
```

**Advantage:** Solves the key distribution problem — public key can be shared openly
**Disadvantage:** ~1000× slower than symmetric encryption; not suitable for bulk data

**In practice:** Asymmetric crypto establishes a shared secret; symmetric crypto encrypts the data (hybrid encryption).

### RSA

Based on the mathematical difficulty of factoring the product of two large prime numbers.

**Key generation:**
```
1. Choose two large primes p and q
2. n = p × q  (modulus; public)
3. φ(n) = (p-1)(q-1)
4. Choose e (public exponent; typically 65537)
5. d = modular inverse of e mod φ(n)  (private exponent)

Public key:  (n, e)
Private key: (n, d)
```

**Encryption/Decryption:**
```
Encrypt:  C = M^e mod n
Decrypt:  M = C^d mod n
```

**Security requirements:**
- Minimum **2048-bit** key for current security; **4096-bit** for long-term security
- RSA-1024 is considered broken (factored with sufficient resources)
- Must use **OAEP padding** for encryption (raw RSA / PKCS#1 v1.5 is vulnerable to padding oracle attacks)
- Must use **PSS padding** for signatures (PKCS#1 v1.5 for signatures has weaknesses)

### Diffie-Hellman Key Exchange

Allows two parties to establish a shared secret over a public channel without ever transmitting the secret itself.

```
Public parameters: prime p, generator g (shared openly)

Alice:              Bob:
Private: a          Private: b
Public:  A = g^a mod p     Public: B = g^b mod p

Exchange A and B publicly.

Alice computes: S = B^a mod p = g^(ab) mod p
Bob computes:   S = A^b mod p = g^(ab) mod p

Both arrive at the same S without transmitting it.
```

An eavesdropper sees g, p, A, B — computing `a` from `g^a mod p` requires solving the **discrete logarithm problem** (computationally infeasible for large parameters).

**DHE vs DH:**
- **Static DH** — Fixed key pairs; compromise of private key exposes all past sessions
- **DHE (Ephemeral DH)** — New key pair per session; provides **Perfect Forward Secrecy (PFS)**
- **ECDHE** — Elliptic curve version; same security with much smaller keys

### Elliptic Curve Cryptography (ECC)

Based on the elliptic curve discrete logarithm problem (ECDLP) — harder than integer factoring, so smaller keys provide equivalent security.

```
Security comparison:
RSA-2048  ≈  ECC-224 bits
RSA-3072  ≈  ECC-256 bits
RSA-7680  ≈  ECC-384 bits
```

**Common curves:**

| Curve | Bits | Usage | Notes |
|-------|------|-------|-------|
| P-256 (secp256r1) | 256 | TLS, certificates | NIST curve; widely supported |
| P-384 (secp384r1) | 384 | High-security TLS | NSA Suite B |
| Curve25519 | 255 | TLS, SSH, Signal | Fast; considered safer than NIST curves |
| secp256k1 | 256 | Bitcoin | Not recommended for general use |
| Brainpool curves | 256–512 | EU standards | Alternative to NIST curves |

**ECDSA** — Elliptic Curve Digital Signature Algorithm; requires a unique, cryptographically random nonce per signature. Reusing the nonce reveals the private key (PlayStation 3 hack).

**EdDSA / Ed25519** — Deterministic signatures (no random nonce); faster; more resistant to implementation errors. Preferred over ECDSA when supported.

### ElGamal

Asymmetric encryption based on Diffie-Hellman. Used as the basis for some PGP implementations and DSA.

---

## 4. Cryptographic Hash Functions

### Properties

A cryptographic hash function H maps arbitrary-length input to a fixed-length digest:

| Property | Description |
|----------|-------------|
| **Deterministic** | Same input always produces same output |
| **Pre-image resistance** | Given hash H(x), infeasible to find x |
| **Second pre-image resistance** | Given x, infeasible to find x' ≠ x where H(x) = H(x') |
| **Collision resistance** | Infeasible to find any x, x' where H(x) = H(x') |
| **Avalanche effect** | Small change in input → completely different output |

### Common Hash Functions

| Algorithm | Output | Status | Notes |
|-----------|--------|--------|-------|
| MD5 | 128 bits | **Broken** | Collisions found in seconds; never use for security |
| SHA-1 | 160 bits | **Broken** | Practical collision (SHAttered, 2017); deprecated |
| SHA-256 | 256 bits | Secure | Part of SHA-2 family; widely used |
| SHA-384 | 384 bits | Secure | SHA-2; higher security margin |
| SHA-512 | 512 bits | Secure | SHA-2; fast on 64-bit systems |
| SHA3-256 | 256 bits | Secure | SHA-3 family; different design (Keccak sponge) |
| SHA3-512 | 512 bits | Secure | SHA-3 |
| BLAKE2 | Variable | Secure | Faster than SHA-2; used in password managers |
| BLAKE3 | 256 bits | Secure | Faster than BLAKE2; parallelizable |

### Birthday Attack

Due to the birthday paradox, collisions can be found in approximately 2^(n/2) operations for an n-bit hash — not 2^n as intuition suggests.

```
SHA-1 (160-bit): 2^80 operations for collision (broken in practice — SHAttered used ~2^63)
SHA-256 (256-bit): 2^128 operations for collision (infeasible)
MD5 (128-bit): 2^64 operations — broken long ago
```

This is why a 128-bit hash provides only ~64 bits of collision resistance.

### Hash Uses in Security

**Password storage:**
```
# Bad
store plaintext "hunter2"

# Bad
store MD5("hunter2") = "2ab96390c7dbe3439de74d0c9b0b1767"
                        ↑ rainbow table precomputed

# Bad
store SHA-256("hunter2")  — still fast; GPU can compute 10B/sec

# Good
store bcrypt("hunter2", cost=12)  — intentionally slow; ~100ms per hash
store Argon2id("hunter2", memory=64MB, iterations=3)  — memory-hard
```

**File integrity:**
```bash
# Verify a download
sha256sum ubuntu-22.04.iso
# Compare against published hash on vendor website
```

**Merkle Trees:**
Each node is the hash of its children. Used in:
- Git (each commit hashes tree of file hashes)
- Blockchain (transaction Merkle root in block header)
- Certificate Transparency logs
- TLS certificate chain validation

---

## 5. Message Authentication Codes (MAC)

### Purpose

A MAC provides integrity AND authentication. It is a keyed hash — only a party with the secret key can produce or verify it.

```
MAC(key, message) → authentication tag

Sender:   tag = HMAC(key, message)  → send (message, tag)
Receiver: verify HMAC(key, message) == tag  → accept/reject
```

Without the key, an attacker cannot forge a valid tag, even knowing the message.

**MAC vs Hash:** A hash alone provides integrity but not authentication — an attacker can replace both the data AND the hash. A MAC requires knowledge of the key.

### HMAC

HMAC (Hash-based MAC) constructs a MAC from any cryptographic hash:

```
HMAC(K, m) = H((K ⊕ opad) || H((K ⊕ ipad) || m))

Where:
  K    = key (padded to block size)
  opad = 0x5C repeated
  ipad = 0x36 repeated
  H    = hash function (SHA-256, SHA-512, etc.)
```

HMAC-SHA256 and HMAC-SHA512 are the standard choices.

### Poly1305

A fast MAC designed to pair with ChaCha20 (ChaCha20-Poly1305 is an AEAD cipher). Used in TLS 1.3, WireGuard, SSH.

### CMAC / GMAC

- **CMAC** — Block cipher-based MAC (AES-CMAC); used in some protocol standards
- **GMAC** — Galois MAC; the authentication component of AES-GCM

### Timing Attacks on MAC Verification

Never compare MACs with `==` string comparison. String equality short-circuits at the first differing byte, leaking timing information that allows an attacker to forge MACs byte by byte.

```python
# Vulnerable
if mac == expected_mac:    # timing leak

# Safe — constant-time comparison
import hmac
if hmac.compare_digest(mac, expected_mac):
```

---

## 6. Key Exchange & Key Derivation

### Diffie-Hellman (revisited in context)

DH solves the **key establishment problem** — two parties who have never met can agree on a shared secret over a public channel. The secret can then be used as a symmetric key.

**Perfect Forward Secrecy (PFS):**
If long-term private keys are compromised, past session keys derived via ephemeral DH (DHE/ECDHE) remain secure — each session used a fresh key pair that is discarded after use.

```
Without PFS: Decrypt past traffic if you get the server's long-term private key
With PFS:    Past traffic protected even after key compromise
```

TLS 1.3 mandates PFS — DHE and ECDHE are the only supported key exchange methods.

### ECDH Key Exchange

```
Alice:                          Bob:
Private key: a                  Private key: b
Public key: A = a × G           Public key: B = b × G

Exchange public keys.

Alice: S = a × B = a × b × G
Bob:   S = b × A = b × a × G
Same shared secret S.
```

Where G is the base point of the elliptic curve.

### Key Derivation Functions (KDF)

A KDF derives one or more cryptographic keys from a source of entropy (a password, shared secret, or master key).

**Why not use a password directly as a key?**
- Passwords have low entropy
- Keys need specific lengths
- Need to derive multiple keys from one master secret

**HKDF (HMAC-based Extract-and-Expand KDF):**
```
Extract: PRK = HMAC(salt, input keying material)
Expand:  key = HMAC(PRK, info || counter)
```
Used in TLS 1.3, Signal Protocol, noise protocol framework.

**Password-Based KDFs (slow/memory-hard — for password storage/derivation):**

| KDF | Properties | Status |
|-----|-----------|--------|
| PBKDF2 | Iterations; uses HMAC internally | Acceptable; prefer Argon2 |
| bcrypt | Work factor; 72-byte password limit | Good |
| scrypt | Memory-hard; CPU and memory cost | Good |
| Argon2id | Memory-hard; parallelism; winner of PHC | **Recommended** |

```
Argon2id parameters (2024 recommendations):
  Memory: 64 MB minimum (higher = better)
  Iterations: 3 minimum
  Parallelism: 4
  Output length: 32 bytes
  Salt: 16 random bytes (unique per password)
```

---

## 7. Digital Signatures & PKI

### Digital Signatures

A digital signature scheme consists of three algorithms:
- **KeyGen** — Generate public/private key pair
- **Sign** — Produce signature using private key
- **Verify** — Verify signature using public key

```
Sign:   σ = Sign(private_key, H(message))
Verify: Verify(public_key, H(message), σ) → {valid, invalid}
```

Properties guaranteed:
- **Authenticity** — Only the holder of the private key could produce the signature
- **Integrity** — Any modification to the message invalidates the signature
- **Non-repudiation** — Signer cannot deny signing (assuming private key was secure)

**Common signature algorithms:**

| Algorithm | Based On | Recommendations |
|-----------|---------|----------------|
| RSA-PSS | RSA | Secure; use PSS padding, not PKCS#1 v1.5 |
| ECDSA | Elliptic curve | Secure; require random nonce per signature |
| Ed25519 | EdDSA / Curve25519 | **Preferred** — deterministic; fast; safe |
| Ed448 | EdDSA / Curve448 | Higher security margin than Ed25519 |
| DSA | Discrete log | Deprecated; avoid |

### X.509 Certificates

A certificate binds a public key to an identity, signed by a trusted Certificate Authority (CA).

```
Certificate contents:
  Version         : 3
  Serial Number   : 03:AB:7F:...
  Issuer          : CN=DigiCert TLS RSA SHA256 2020 CA1, O=DigiCert Inc
  Subject         : CN=www.example.com, O=Example Corp
  Valid From      : 2024-01-01
  Valid To        : 2025-01-01
  Public Key      : RSA 2048-bit / EC P-256 (subjectPublicKeyInfo)
  Extensions:
    Subject Alternative Names (SANs): www.example.com, example.com
    Key Usage: Digital Signature, Key Encipherment
    Extended Key Usage: TLS Web Server Authentication
    CRL Distribution Points: http://crl.digicert.com/...
    OCSP: http://ocsp.digicert.com
  Signature       : [CA's signature over all the above]
```

### Public Key Infrastructure (PKI)

PKI is the system of policies, processes, hardware, software, and roles needed to create, manage, distribute, use, store, and revoke digital certificates.

```
Root CA (self-signed; kept offline)
    │  signs
    ▼
Intermediate CA (online; issues end-entity certs)
    │  signs
    ▼
End-Entity Certificate (server, client, code signing)
```

**Trust anchor:** The root CA certificate is pre-installed in OS/browser trust stores. Any certificate signed by a trusted root (directly or through intermediates) is trusted.

**Certificate chain validation:**
1. Is the signature on the end-entity cert valid, signed by intermediate CA?
2. Is the intermediate CA cert signed by the root CA?
3. Is the root CA in the trust store?
4. Is each certificate within its validity period?
5. Has any certificate been revoked?
6. Does the subject name / SAN match the intended domain?

### Certificate Revocation

If a certificate's private key is compromised, the certificate must be revoked before expiry:

| Mechanism | How It Works | Limitations |
|-----------|-------------|------------|
| **CRL (Certificate Revocation List)** | CA publishes a list of revoked serial numbers | Large files; periodic updates; latency |
| **OCSP (Online Certificate Status Protocol)** | Real-time query to CA for revocation status | Privacy concern; CA availability required |
| **OCSP Stapling** | Server fetches and caches its own OCSP response; includes it in TLS handshake | Solves privacy and availability; must be enforced |
| **CRLite / OCSP Must-Staple** | Browser-side mechanisms for enforcing revocation | Newer; not universally supported |

### Certificate Transparency (CT)

All publicly-trusted TLS certificates must be logged in public, append-only CT logs. Allows detection of:
- Mis-issued certificates (CAs issuing certs for domains they shouldn't)
- Rogue certificates issued for your domain

Monitors (e.g. crt.sh) and clients verify CT log inclusion via Signed Certificate Timestamps (SCTs).

### Code Signing

Applying a digital signature to software to prove authenticity and integrity:

```
Developer:
  hash = SHA-256(binary)
  signature = Sign(private_key, hash)
  embed (signature, cert) in binary

User/OS:
  Extract signature + cert from binary
  Verify cert chain → trusted root
  Verify signature → matches current hash of binary
  Confirm: binary unmodified, from known publisher
```

- Windows: Authenticode (PE format)
- macOS: Apple code signing + notarization
- Linux: GPG-signed packages (apt, rpm)
- Java: JAR signing

---

## 8. TLS (Transport Layer Security)

TLS is the cryptographic protocol securing HTTPS, SMTP, IMAP, and most other encrypted network communication.

### TLS 1.3 Handshake (Current Standard)

```
Client                                          Server
  │──── ClientHello ────────────────────────────▶│
  │     (TLS 1.3, supported ciphers,              │
  │      key_share: ECDHE public key,             │
  │      random nonce)                            │
  │                                               │
  │◀─── ServerHello ──────────────────────────── │
  │     (chosen cipher, key_share: server ECDHE, │
  │      random nonce)                            │
  │◀─── {EncryptedExtensions} ────────────────── │
  │◀─── {Certificate} ────────────────────────── │
  │◀─── {CertificateVerify} ──────────────────── │
  │◀─── {Finished} ───────────────────────────── │
  │──── {Finished} ─────────────────────────────▶│
  │                                               │
  │◀══════════ Encrypted Application Data ═══════▶│
```

Key improvements in TLS 1.3 over 1.2:
- 1-RTT handshake (vs 2-RTT in TLS 1.2)
- 0-RTT resumption (with replay attack caveats)
- Removed all weak algorithms: RSA key exchange, static DH, RC4, 3DES, MD5, SHA-1
- Forward secrecy is mandatory (only ECDHE/DHE key exchange)
- Handshake is encrypted earlier

### TLS Cipher Suite Notation

**TLS 1.3 (simplified):**
```
TLS_AES_256_GCM_SHA384
    │    │       │
    │    │       └─ HKDF hash (key derivation)
    │    └─────── Bulk encryption algorithm
    └──────────── Always ECDHE key exchange in TLS 1.3
```

**TLS 1.2 (full specification):**
```
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    │      │       │         │
    │      │       │         └─ MAC/PRF hash
    │      │       └─────────── Bulk encryption + mode
    │      └─────────────────── Authentication (cert type)
    └────────────────────────── Key exchange
```

### TLS Vulnerabilities (Historical)

| Vulnerability | Protocol | Issue |
|---------------|---------|-------|
| **POODLE** | SSLv3 / TLS 1.0 CBC | Padding oracle on CBC mode |
| **BEAST** | TLS 1.0 CBC | Predictable IV in CBC mode |
| **CRIME / BREACH** | TLS compression | Compression oracle leaks secrets |
| **HEARTBLEED** | OpenSSL | Memory read beyond bounds via Heartbeat extension |
| **FREAK** | TLS | Export-grade RSA downgrade attack |
| **Logjam** | TLS DHE | Downgrade to 512-bit DH (export-grade) |
| **DROWN** | SSLv2 | Cross-protocol attack against TLS servers sharing a key with SSLv2 |
| **ROBOT** | TLS RSA | Bleichenbacher padding oracle on RSA key exchange |

**Mitigations:** Disable SSLv3, TLS 1.0, TLS 1.1; use TLS 1.2+ (TLS 1.3 preferred); disable export cipher suites; use ECDHE for key exchange.

### HSTS (HTTP Strict Transport Security)

Instructs browsers to only connect over HTTPS:
```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```
- Prevents SSL stripping attacks
- `preload` submits the domain to browser HSTS preload lists — HTTPS enforced on first visit

### Certificate Pinning

Hard-code the expected certificate or public key hash in a client application:
```
Expected pin: sha256/AAAAAABBBBBBCCCCCCDDDDDD...
If server cert doesn't match → reject connection
```
- Prevents MitM even with a rogue trusted CA
- Maintenance burden: pins must be updated before certificates expire

---

## 9. Password Hashing

### Why Password Hashing Is Different

General-purpose hash functions (SHA-256, MD5) are designed to be fast. Fast is bad for password hashing — it lets attackers try billions of candidates per second with GPUs.

```
MD5 on GPU:    ~50 billion hashes/second
SHA-256 on GPU: ~10 billion hashes/second
bcrypt (cost 12): ~200 hashes/second per GPU
Argon2id (64MB): ~2 hashes/second per GPU
```

Password hashing functions are intentionally slow and/or memory-intensive.

### Salting

A salt is a unique random value added to each password before hashing:

```
Without salt:
  H("password123") = "abc123..."  → same for all users with "password123"
  → one rainbow table cracks all

With salt:
  H("password123" + "x7k9m2") = "def456..."  → unique per user
  H("password123" + "p3q8r1") = "789abc..."  → different hash
  → attacker must crack each hash individually
```

The salt is stored alongside the hash (it's not secret; its purpose is uniqueness, not secrecy).

### bcrypt

```
$2b$12$salt22characterbase64hashhhhhhhhhhhh
 │   │   │                    │
 │   │   └── 22-char salt     └── 31-char hash
 │   └─── cost factor (2^12 = 4096 iterations)
 └──── algorithm version
```

- Cost factor is tunable; increase as hardware gets faster
- 72-byte password limit (truncation vulnerability for very long passwords)
- Widely supported; good default choice

### Argon2 (Recommended)

Winner of the Password Hashing Competition (2015). Three variants:
- **Argon2d** — GPU-resistant (data-dependent memory access); vulnerable to side channels
- **Argon2i** — Side-channel resistant; less GPU-resistant
- **Argon2id** — Hybrid; recommended for most uses

```
Argon2id parameters:
  $argon2id$v=19$m=65536,t=3,p=4$salt$hash
                  │       │   │
                  │       │   └── parallelism
                  │       └─── time cost (iterations)
                  └─── memory cost (KB)
```

### PBKDF2

```
PBKDF2(password, salt, iterations, keylen, PRF)
```
- Iterations should be ≥ 600,000 for PBKDF2-HMAC-SHA256 (NIST 2023 recommendation)
- Less memory-hard than bcrypt/Argon2; GPU attacks more effective
- FIPS-compliant; required in some regulatory contexts

---

## 10. Random Number Generation

### Importance

Cryptography depends entirely on unpredictable randomness. Weak randomness breaks everything — keys become guessable, nonces repeat, padding is predictable.

**Dual EC DRBG** — An NSA-designed PRNG standardized by NIST; later revealed to contain a backdoor allowing the designer to predict outputs. Used in RSA BSAFE (Juniper Networks breach).

### PRNG vs CSPRNG

| Type | Description | Use |
|------|-------------|-----|
| **PRNG** (Pseudorandom) | Deterministic algorithm; seeded with state; fast | Simulations, games |
| **CSPRNG** (Cryptographically Secure PRNG) | Unpredictable even knowing past outputs; seeded from entropy | All cryptographic use |

### Entropy Sources

| Source | Platform | Notes |
|--------|----------|-------|
| `/dev/urandom` | Linux | Non-blocking CSPRNG; seeded from kernel entropy pool |
| `/dev/random` | Linux | Blocking until sufficient entropy; rarely needed |
| `getrandom()` syscall | Linux 3.17+ | Preferred; fills buffer directly |
| `CryptGenRandom` / `BCryptGenRandom` | Windows | Windows CSPRNG |
| `os.urandom()` | Python | Wraps OS CSPRNG |
| `crypto.randomBytes()` | Node.js | Wraps OS CSPRNG |
| Hardware RNG (RDRAND) | x86 CPUs | CPU-level entropy; can be combined with OS entropy |
| TPM RNG | Hardware | High-quality hardware entropy |

**Never use:**
- `rand()` / `random()` / `Math.random()` for cryptographic purposes
- Time-based seeds
- User-visible values (PID, hostname) as entropy sources

---

## 11. Cryptographic Protocols

### SSH (Secure Shell)

SSH provides encrypted remote shell access and file transfer.

**SSH key exchange (simplified):**
```
Client                        Server
  │── Protocol version ──────▶│
  │◀── Protocol version ────── │
  │── Key exchange init ──────▶│  (algorithms supported)
  │◀── Key exchange init ────── │  (algorithms chosen)
  │── ECDH key share ─────────▶│
  │◀── ECDH key share ──────── │  (+ host key + signature)
  │── New Keys ───────────────▶│
  │◀── New Keys ─────────────── │
  │══════ Encrypted channel ════│
  │── User authentication ─────▶│  (password or public key)
```

**SSH key authentication:**
```
# Client proves possession of private key
Client sends: public key it wants to use
Server checks: is this key in ~/.ssh/authorized_keys?
Server sends: random challenge
Client signs:  challenge with private key
Server verifies: signature with public key
→ Authentication succeeds
```

**SSH config hardening:**
```
# /etc/ssh/sshd_config
HostKeyAlgorithms ssh-ed25519,ecdsa-sha2-nistp256
KexAlgorithms curve25519-sha256,ecdh-sha2-nistp256
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com
```

### IPsec

Network-layer encryption; secures IP packets. Used in VPNs and site-to-site tunnels.

**Modes:**
- **Transport mode** — Encrypts payload only; IP header visible
- **Tunnel mode** — Encrypts entire original IP packet; wrapped in new IP header (used in VPNs)

**Protocols:**
- **AH (Authentication Header)** — Integrity and authentication only; no encryption
- **ESP (Encapsulating Security Payload)** — Encryption + integrity
- **IKE/IKEv2** — Key exchange and SA negotiation

### PGP / GPG

Pretty Good Privacy — email encryption and signing; file encryption.

**Web of Trust:** Instead of a CA hierarchy, PGP uses a decentralized trust model where users sign each other's keys to vouch for identity.

```bash
# Encrypt a file for recipient
gpg --encrypt --recipient alice@example.com file.txt

# Sign a file
gpg --sign file.txt

# Sign and encrypt
gpg --sign --encrypt --recipient alice@example.com file.txt

# Verify a signature
gpg --verify file.txt.gpg

# Decrypt
gpg --decrypt file.txt.gpg
```

### Signal Protocol

Used by Signal, WhatsApp, and others. Combines:
- **X3DH (Extended Triple Diffie-Hellman)** — Asynchronous key agreement; allows messaging someone who is offline
- **Double Ratchet Algorithm** — Provides forward secrecy and break-in recovery; both KDF chain ratchet and DH ratchet advance on each message

The Double Ratchet means compromising a session key only exposes one or a few messages — it self-heals.

---

## 12. Cryptographic Attacks

### Brute Force

Try every possible key until the correct one is found.

```
Keyspace for n-bit key: 2^n
AES-128: 2^128 ≈ 3.4 × 10^38 keys → infeasible
DES-56:  2^56  ≈ 7.2 × 10^16 keys → cracked in 1998 by EFF DES Cracker
```

### Dictionary / Wordlist Attack

Try common passwords and variations. Highly effective against weak passwords.
Tools: Hashcat, John the Ripper

```bash
hashcat -m 1800 hashes.txt rockyou.txt   # SHA-512 crypt
hashcat -m 3200 hashes.txt rockyou.txt   # bcrypt
```

### Rainbow Table Attack

Precomputed table of hash → plaintext mappings. Trades storage for computation time.

Defeated by salting — unique per-password salt means the table would need to be recomputed for every possible salt value.

### Birthday Attack

Find two inputs with the same hash. Requires ~2^(n/2) work for n-bit hash.
Used to forge digital signatures when signatures depend on hash collisions (historical attacks on MD5, SHA-1).

### Length Extension Attack

For hash functions based on Merkle-Damgård construction (MD5, SHA-1, SHA-256), an attacker knowing H(message) can compute H(message || extension) without knowing the message itself, for arbitrary extensions.

Affects: HMAC implementations that naively use H(key || message)
Mitigation: Use HMAC properly, or use SHA-3 (sponge construction; immune to length extension)

### Padding Oracle Attack

When an application reveals whether decrypted padding is valid, an attacker can decrypt ciphertext byte by byte without knowing the key.

Famous examples:
- **POODLE** — SSLv3 CBC padding oracle
- **PKCS#1 v1.5** — RSA padding oracle (Bleichenbacher's attack, 1998; still exploitable in ROBOT, 2017)

Mitigation: Use authenticated encryption (GCM); constant-time padding validation.

### Side-Channel Attacks

Exploit information leaked by the physical implementation of cryptography:

| Attack | Side Channel | Example |
|--------|-------------|---------|
| **Timing attack** | Execution time | Key comparison time leaks key bits |
| **Power analysis** | Power consumption | SPA/DPA on smart cards |
| **Cache timing** | Cache state | AES S-box access patterns leak key |
| **EM attack** | Electromagnetic emissions | Monitor EM from CPU during crypto |
| **Fault injection** | Induced errors | Glitch power supply during signing to extract key |
| **Spectre/Meltdown** | CPU speculation | Cross-process memory read via cache timing |

Mitigation: Constant-time implementations; hardware countermeasures; power filtering.

### Cryptanalytic Attacks

| Attack | Description |
|--------|-------------|
| **Differential cryptanalysis** | Analyze how differences in input affect output; used to attack DES |
| **Linear cryptanalysis** | Find linear approximations to cipher behavior |
| **Related-key attack** | Exploit weakness when keys are related (AES-192/256 theoretical weaknesses) |
| **Meet-in-the-middle** | Attack on double encryption (why 2DES provides only 57-bit security, not 112-bit) |
| **Chosen-plaintext attack (CPA)** | Attacker can choose plaintexts and observe ciphertexts |
| **Chosen-ciphertext attack (CCA)** | Attacker can choose ciphertexts and observe decryptions |
| **Known-plaintext attack (KPA)** | Attacker has matching plaintext/ciphertext pairs |

Modern ciphers (AES, ChaCha20) are designed to resist all known cryptanalytic attacks.

---

## 13. Post-Quantum Cryptography

### The Quantum Threat

Quantum computers running **Shor's algorithm** can efficiently solve:
- Integer factorization → breaks RSA
- Discrete logarithm → breaks DH, DSA, ECDSA, ECDH

**Grover's algorithm** provides a quadratic speedup for brute force:
- AES-128 effective security reduced to ~64 bits → upgrade to AES-256
- AES-256 effective security reduced to ~128 bits → still secure

**Timeline:** Cryptographically relevant quantum computers (CRQCs) don't currently exist. NIST estimates 10–20+ years. Organizations handling long-lived secrets should prepare now (harvest now, decrypt later attacks).

### NIST Post-Quantum Standards (2024)

NIST completed standardization of post-quantum algorithms:

| Algorithm | Type | Based On | Standard |
|-----------|------|---------|---------|
| **ML-KEM** (CRYSTALS-Kyber) | Key encapsulation | Module lattices | FIPS 203 |
| **ML-DSA** (CRYSTALS-Dilithium) | Digital signature | Module lattices | FIPS 204 |
| **SLH-DSA** (SPHINCS+) | Digital signature | Hash functions | FIPS 205 |
| **FN-DSA** (FALCON) | Digital signature | NTRU lattices | Upcoming |

**Hybrid schemes:** Many implementations combine classical (ECDH) + post-quantum (ML-KEM) in a single key exchange — breaking the key exchange requires breaking both algorithms.

### Migration Strategy

```
Now:
  ✓ Inventory all cryptographic usage
  ✓ Identify data with long-term sensitivity (10+ year protection needed)
  ✓ Assess vendor cryptographic agility

Near-term:
  ✓ Upgrade to AES-256 (Grover protection)
  ✓ Upgrade to SHA-384 / SHA-512
  ✓ Enable hybrid classical + PQC in TLS where available

Long-term:
  ✓ Replace RSA/ECDSA with ML-DSA or SLH-DSA for signatures
  ✓ Replace ECDH with ML-KEM for key exchange
```

---

## 14. Common Cryptographic Mistakes

### Using Cryptography Incorrectly

| Mistake | Consequence | Fix |
|---------|-------------|-----|
| Using ECB mode | Pattern leakage; identical blocks → identical ciphertext | Use AES-GCM |
| Reusing IV/nonce | With CTR/GCM: nonce reuse = catastrophic; XOR of ciphertexts cancels key | Generate random nonce per encryption |
| Encrypting without authenticating | Ciphertext malleable; padding oracle attacks | Use AEAD (GCM, ChaCha20-Poly1305) |
| Storing passwords with SHA-256 | Fast hash; GPU-crackable in seconds | Use bcrypt or Argon2id |
| Not salting passwords | Rainbow tables; identical passwords have identical hashes | Unique random salt per password |
| Non-constant-time comparison | Timing oracle leaks secret | Use `hmac.compare_digest()` / `crypto_verify()` |
| Rolling your own crypto | Implementation flaws; side channels; subtle mathematical errors | Use vetted libraries (OpenSSL, libsodium, BouncyCastle) |
| Weak random number generation | Predictable keys, nonces, tokens | Use OS CSPRNG |
| Hardcoding keys | Keys in source code → leaked in repos, decompiled binaries | Use secrets management (Vault, AWS KMS) |
| Using MD5 / SHA-1 for security | Collision attacks; broken | Use SHA-256+ |
| Short RSA keys (1024-bit) | Factorable with sufficient resources | RSA-2048 minimum; prefer ECDSA P-256 or Ed25519 |
| Trusting self-signed certs blindly | No identity verification | Proper CA chain validation |
| Certificate not validated (hostname check skipped) | MitM undetected | Always verify hostname against certificate SANs |

### Cryptographic Library Recommendations

| Language | Library | Notes |
|----------|---------|-------|
| Python | `cryptography` | High-level hazmat; avoid `pycrypto` (unmaintained) |
| Python | `PyNaCl` | Wraps libsodium; opinionated safe defaults |
| JavaScript/Node | `Web Crypto API`, `node:crypto` | Built-in; avoid `crypto-js` for sensitive use |
| Java | `Bouncy Castle`, `JCA/JCE` | Standard for enterprise Java |
| Go | `golang.org/x/crypto`, `crypto/` stdlib | Good standard library |
| Rust | `ring`, `RustCrypto` | Memory-safe; audited |
| C/C++ | `libsodium`, `OpenSSL` | libsodium has safer API; OpenSSL for full PKI |

**Golden rule:** Use high-level, opinionated APIs (libsodium, NaCl) when possible. The more choices you make, the more opportunities for mistakes.

---

## 15. Quick Reference

### Algorithm Selection Guide (2024)

| Use Case | Recommended | Avoid |
|----------|-------------|-------|
| Symmetric encryption | AES-256-GCM, ChaCha20-Poly1305 | DES, 3DES, RC4, AES-ECB |
| Asymmetric encryption | RSA-OAEP-2048+, ECIES | RSA-PKCS1v1.5, ElGamal raw |
| Key exchange | ECDHE (X25519), DHE-2048+ | Static RSA, DH-1024 |
| Digital signature | Ed25519, ECDSA-P256, RSA-PSS-2048 | DSA, RSA-PKCS1v1.5, MD5withRSA |
| Hashing | SHA-256, SHA-512, SHA3-256, BLAKE3 | MD5, SHA-1 |
| Password hashing | Argon2id, bcrypt (cost≥12), scrypt | MD5, SHA-256 (without KDF), PBKDF2 (low iterations) |
| MAC | HMAC-SHA256, HMAC-SHA512, Poly1305 | HMAC-MD5, unauthenticated CBC |
| Key derivation | HKDF, Argon2id (from password) | Direct use of password as key |
| Random numbers | OS CSPRNG (`/dev/urandom`, `BCryptGenRandom`) | `rand()`, `Math.random()`, time-seeded |
| TLS | TLS 1.3, TLS 1.2 (strong ciphers) | TLS 1.0, TLS 1.1, SSLv3 |

### Key Length Reference

| Algorithm | Minimum | Recommended | Notes |
|-----------|---------|-------------|-------|
| AES | 128-bit | 256-bit | 128-bit still secure; 256-bit for post-quantum margin |
| RSA | 2048-bit | 4096-bit | 1024-bit broken; 2048-bit for near-term use |
| DH | 2048-bit | 3072-bit | Group size, not key size |
| ECDSA/ECDH | 256-bit (P-256) | 384-bit (P-384) | Equivalent to RSA-3072 |
| Ed25519 | 255-bit (fixed) | — | No choice needed |
| SHA-2 | SHA-256 | SHA-512 | For post-quantum, prefer SHA-384/SHA-512 |

---

## Further Reading

- [Cryptography Engineering (Ferguson, Schneier, Kohno)](https://www.schneier.com/books/cryptography-engineering/)
- [The Illustrated TLS 1.3 Connection](https://tls13.xargs.org/)
- [NIST Post-Quantum Cryptography](https://csrc.nist.gov/projects/post-quantum-cryptography)
- [An Introduction to Mathematical Cryptography (Hoffstein, Pipher, Silverman)](https://www.springer.com/book/9780387779935)
- [Crypto101 (free)](https://www.crypto101.io/)
- [Cryptopals Challenges](https://cryptopals.com/) — Hands-on cryptographic attacks
- [Applied Cryptography (Schneier)](https://www.schneier.com/books/applied-cryptography/)
- [libsodium Documentation](https://doc.libsodium.org/)
- [OWASP Cryptographic Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)
- [TLS Cipher Suite Checker (SSL Labs)](https://www.ssllabs.com/ssltest/)