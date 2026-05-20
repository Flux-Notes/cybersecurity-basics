# Web Request Flow

Understanding how a web request travels from a browser to a server and back is foundational to web security. Attackers and defenders alike must know where data flows, where trust is established, and where things can go wrong.

---

## Overview

When you type a URL into a browser and press Enter, a multi-step process unfolds involving DNS resolution, TCP/TLS handshakes, HTTP messaging, and server-side processing — before a response ever reaches your screen.

```
Browser → DNS Resolver → TCP Handshake → TLS Handshake → HTTP Request → Server → HTTP Response → Browser
```

---

## Step-by-Step Breakdown

### 1. URL Parsing

The browser parses the URL into its components:

```
https://www.example.com:443/path/to/page?query=value#section
  │       │               │   │            │           │
scheme  hostname         port path        query      fragment
```

- **Scheme** – Determines the protocol (`http`, `https`, `ftp`, etc.)
- **Hostname** – The domain that needs to be resolved to an IP
- **Port** – Defaults to `80` for HTTP, `443` for HTTPS
- **Path** – The resource being requested on the server
- **Query string** – Key-value parameters sent to the server
- **Fragment** – Client-side anchor; never sent to the server

---

### 2. DNS Resolution

Before a connection can be made, the hostname must be resolved to an IP address.

```
Browser Cache → OS Cache → Recursive Resolver → Root Nameserver → TLD Nameserver → Authoritative Nameserver
```

| Step | Description |
|------|-------------|
| Browser cache | Checks locally cached DNS records |
| OS cache | Checks `/etc/hosts` and the OS DNS cache |
| Recursive resolver | Usually your ISP or a public resolver (e.g. `8.8.8.8`) |
| Root nameserver | Directs to the correct TLD nameserver (`.com`, `.org`, etc.) |
| TLD nameserver | Directs to the authoritative nameserver for the domain |
| Authoritative nameserver | Returns the final IP address for the domain |

**Security relevance:**

- **DNS spoofing / cache poisoning** – Attacker injects forged DNS records to redirect traffic
- **DNS over HTTPS (DoH)** – Encrypts DNS queries to prevent eavesdropping and tampering
- **DNSSEC** – Uses digital signatures to validate DNS responses

---

### 3. TCP Three-Way Handshake

Once the IP is known, the browser establishes a TCP connection.

```
Client                    Server
  │──── SYN ────────────────▶│
  │◀─── SYN-ACK ─────────────│
  │──── ACK ────────────────▶│
       Connection established
```

| Flag | Meaning |
|------|---------|
| `SYN` | Client initiates connection, sends sequence number |
| `SYN-ACK` | Server acknowledges, sends its own sequence number |
| `ACK` | Client confirms; connection is now open |

**Security relevance:**

- **SYN flood attack** – Attacker sends many SYN packets without completing the handshake, exhausting server resources (a form of DoS)
- **TCP session hijacking** – Attacker predicts sequence numbers to inject data into an established connection

---

### 4. TLS Handshake (HTTPS only)

For HTTPS connections, a TLS handshake occurs after the TCP connection to establish an encrypted channel.

```
Client                                Server
  │──── ClientHello ────────────────▶│  (TLS version, cipher suites, random)
  │◀─── ServerHello ─────────────────│  (chosen cipher, server random)
  │◀─── Certificate ─────────────────│  (server's X.509 cert)
  │──── Key Exchange ───────────────▶│  (pre-master secret or DH key share)
  │◀─── Finished ─────────────────── │
  │──── Finished ────────────────────▶│
       Encrypted channel established
```

Key concepts:

- **Certificate** – The server presents an X.509 certificate signed by a trusted Certificate Authority (CA)
- **Certificate validation** – Browser checks: is the cert signed by a trusted CA? Is it expired? Does the hostname match?
- **Session keys** – Both sides derive symmetric encryption keys from the handshake; all subsequent data is encrypted

**Security relevance:**

- **SSL stripping** – Attacker downgrades HTTPS to HTTP (mitigated by HSTS)
- **Man-in-the-Middle (MitM)** – Attacker intercepts TLS by presenting a forged certificate (requires controlling a trusted CA or a user accepting an invalid cert)
- **Certificate pinning** – App hard-codes the expected cert/public key to prevent MitM

---

### 5. HTTP Request

With the connection established, the browser sends an HTTP request.

```
GET /path/to/page?query=value HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 ...
Accept: text/html,application/xhtml+xml
Accept-Language: en-US,en;q=0.9
Cookie: session_id=abc123; theme=dark
Referer: https://www.example.com/previous-page
Connection: keep-alive
```

**Key headers:**

| Header | Purpose |
|--------|---------|
| `Host` | Specifies the target hostname (required in HTTP/1.1) |
| `User-Agent` | Identifies the browser/client |
| `Cookie` | Sends stored cookies to the server |
| `Authorization` | Carries credentials (e.g. `Bearer <token>`) |
| `Content-Type` | Describes the body format for POST/PUT requests |
| `Referer` | The page that triggered this request |
| `Origin` | Used in CORS preflight requests |

**HTTP Methods:**

| Method | Typical Use | Safe? | Idempotent? |
|--------|-------------|-------|-------------|
| `GET` | Retrieve a resource | ✅ | ✅ |
| `POST` | Submit data / create resource | ❌ | ❌ |
| `PUT` | Replace a resource | ❌ | ✅ |
| `PATCH` | Partially update a resource | ❌ | ❌ |
| `DELETE` | Remove a resource | ❌ | ✅ |
| `HEAD` | Like GET, but no body | ✅ | ✅ |
| `OPTIONS` | Query allowed methods (CORS preflight) | ✅ | ✅ |

**Security relevance:**

- **Header injection** – Attacker injects newlines into headers to manipulate responses
- **Cookie theft** – Stolen session cookies allow session hijacking (mitigated by `HttpOnly` and `Secure` flags)
- **CSRF** – Attacker tricks a user's browser into sending an authenticated request to another site

---

### 6. Server-Side Processing

The server receives the request and processes it:

```
Incoming Request
      │
      ▼
  Web Server (e.g. Nginx, Apache)
      │
      ├── Static file? ──▶ Serve directly
      │
      └── Dynamic request? ──▶ Application Server (e.g. Node.js, Django, Rails)
                                      │
                                      ├── Business logic
                                      ├── Authentication / Authorization check
                                      ├── Input validation & sanitization
                                      └── Database query
                                              │
                                              ▼
                                         Database (e.g. PostgreSQL, MongoDB)
```

**Security relevance at each layer:**

| Layer | Common Attack |
|-------|--------------|
| Web server | Directory traversal, misconfigured permissions |
| Application | SQL injection, XSS, command injection, broken auth |
| Database | SQL injection, excessive privileges, unencrypted data at rest |

---

### 7. HTTP Response

The server sends back an HTTP response:

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 4523
Set-Cookie: session_id=xyz789; HttpOnly; Secure; SameSite=Strict
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Content-Security-Policy: default-src 'self'
Cache-Control: no-store

<!DOCTYPE html>
<html>...
```

**Common status codes:**

| Code | Meaning |
|------|---------|
| `200 OK` | Request succeeded |
| `301 Moved Permanently` | Resource has a new permanent URL |
| `302 Found` | Temporary redirect |
| `400 Bad Request` | Malformed request syntax |
| `401 Unauthorized` | Authentication required |
| `403 Forbidden` | Authenticated but not authorized |
| `404 Not Found` | Resource doesn't exist |
| `500 Internal Server Error` | Server-side error |

**Security response headers:**

| Header | Purpose |
|--------|---------|
| `Strict-Transport-Security` (HSTS) | Forces HTTPS for future requests |
| `Content-Security-Policy` (CSP) | Restricts sources of scripts, styles, images, etc. |
| `X-Frame-Options` | Prevents the page from being embedded in iframes (clickjacking) |
| `X-Content-Type-Options: nosniff` | Prevents MIME-type sniffing |
| `Set-Cookie: HttpOnly; Secure; SameSite` | Hardens cookie security |

---

### 8. Browser Rendering & Same-Origin Policy

After the response is received, the browser parses and renders the page. During this stage, the **Same-Origin Policy (SOP)** enforces critical security boundaries.

Two URLs share the same origin if they have the same **scheme + hostname + port**.

| URL | Same origin as `https://example.com`? |
|-----|---------------------------------------|
| `https://example.com/other-page` | ✅ Yes |
| `http://example.com` | ❌ No (different scheme) |
| `https://sub.example.com` | ❌ No (different hostname) |
| `https://example.com:8080` | ❌ No (different port) |

SOP prevents a malicious page from reading responses from a different origin. **CORS (Cross-Origin Resource Sharing)** relaxes this restriction in a controlled way using headers like `Access-Control-Allow-Origin`.

---

## Full Flow Diagram

```
User types URL
      │
      ▼
Browser parses URL
      │
      ▼
DNS resolution ──────────────────────────── (Attack surface: DNS spoofing)
      │
      ▼
TCP three-way handshake ─────────────────── (Attack surface: SYN flood, session hijack)
      │
      ▼
TLS handshake (HTTPS) ───────────────────── (Attack surface: MitM, SSL stripping)
      │
      ▼
HTTP request sent ───────────────────────── (Attack surface: CSRF, header injection)
      │
      ▼
Server processes request ────────────────── (Attack surface: SQLi, XSS, broken auth)
      │
      ▼
HTTP response returned ──────────────────── (Attack surface: missing security headers)
      │
      ▼
Browser renders page ────────────────────── (Attack surface: XSS, clickjacking)
      │
      ▼
Same-Origin Policy enforced ─────────────── (Mitigates cross-origin data theft)
```

---

## Key Takeaways

- Every layer of the web request flow has distinct attack surfaces and corresponding defenses
- **Encryption (TLS)** protects data in transit but does not guarantee the server is trustworthy — certificate validation matters
- **DNS** is often overlooked but is a critical control point; always validate DNS security in assessments
- **HTTP headers** are both an attack vector (injection) and a defense mechanism (CSP, HSTS, SameSite cookies)
- **Same-Origin Policy** is the browser's primary isolation mechanism; understanding it is essential for web security testing

---

## Further Reading

- [MDN – How browsers work](https://developer.mozilla.org/en-US/docs/Web/Performance/How_browsers_work)
- [MDN – HTTP overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)
- [OWASP – Transport Layer Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html)
- [OWASP – Content Security Policy Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)