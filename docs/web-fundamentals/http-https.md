# HTTP & HTTPS

---

## What is HTTP?

**HTTP (HyperText Transfer Protocol)** is an application-layer protocol used for transmitting hypermedia documents (HTML, JSON, images, etc.) between clients and servers. It is the foundation of data communication on the World Wide Web.

- Defined in **RFC 7230–7235** (HTTP/1.1) and **RFC 7540** (HTTP/2).
- Operates on **TCP port 80** by default.
- **Stateless** — each request is independent; the server retains no memory of previous requests.
- **Text-based** in HTTP/1.x; **binary** in HTTP/2 and HTTP/3.

---

## What is HTTPS?

**HTTPS (HTTP Secure)** is HTTP transmitted over a **TLS (Transport Layer Security)** encrypted connection.

- Operates on **TCP port 443** by default.
- Provides **confidentiality** (encryption), **integrity** (tamper detection), and **authentication** (server identity via certificates).
- Defined alongside TLS in **RFC 8446** (TLS 1.3).

```
HTTP  → Data transmitted in plaintext   → Anyone on the network can read it
HTTPS → Data encrypted with TLS         → Only client and server can read it
```

---

## HTTP vs HTTPS

| Feature | HTTP | HTTPS |
|---|---|---|
| Port | 80 | 443 |
| Encryption | None | TLS (AES, ChaCha20) |
| Integrity | None | HMAC / AEAD |
| Authentication | None | Server certificate (X.509) |
| Speed | Slightly faster (no TLS overhead) | Marginally slower (negligible on modern hardware) |
| SEO | Penalized by Google | Preferred |
| Browser indicator | "Not Secure" warning | Padlock icon |
| Required for | Legacy / internal only | Everything on the public internet |

---

## HTTP Versions

### HTTP/0.9 (1991)
- Single-line protocol; only GET method.
- No headers, no status codes.

### HTTP/1.0 (1996) — RFC 1945
- Added headers, status codes, POST method.
- **One TCP connection per request** — slow due to repeated handshakes.

### HTTP/1.1 (1997) — RFC 7230
- **Persistent connections** — reuse TCP connections (`Connection: keep-alive`).
- **Pipelining** — send multiple requests without waiting for each response (poorly supported).
- **Chunked transfer encoding** — stream data without knowing content length upfront.
- **Host header** — required, enables virtual hosting.
- Still the most widely deployed version.

### HTTP/2 (2015) — RFC 7540
- **Binary framing** — replaces text-based protocol.
- **Multiplexing** — multiple requests/responses over a single TCP connection simultaneously (solves head-of-line blocking at the HTTP layer).
- **Header compression (HPACK)** — reduces overhead of repeated headers.
- **Server push** — server proactively sends resources the client will need.
- **Stream prioritization** — important resources loaded first.
- Requires TLS in practice (browsers only support HTTP/2 over HTTPS).

### HTTP/3 (2022) — RFC 9114
- Runs over **QUIC** (UDP-based transport) instead of TCP.
- Eliminates **TCP head-of-line blocking** entirely.
- Faster connection establishment (**0-RTT** reconnects).
- Built-in encryption (QUIC uses TLS 1.3 internally).
- Operates on **UDP port 443**.

| Version | Transport | Key Feature |
|---|---|---|
| HTTP/1.0 | TCP | One request per connection |
| HTTP/1.1 | TCP | Persistent connections |
| HTTP/2 | TCP + TLS | Multiplexing, binary framing |
| HTTP/3 | QUIC (UDP) + TLS 1.3 | No TCP HOL blocking, 0-RTT |

---

## HTTP Request Structure

```
METHOD /path HTTP/version\r\n
Header-Name: Header-Value\r\n
...
\r\n
[Optional Body]
```

### Example GET Request

```http
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: text/html,application/xhtml+xml
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

### Example POST Request

```http
POST /api/login HTTP/1.1
Host: www.example.com
Content-Type: application/json
Content-Length: 47
Cookie: session=abc123

{"username": "alice", "password": "secret123"}
```

---

## HTTP Response Structure

```
HTTP/version STATUS_CODE Reason-Phrase\r\n
Header-Name: Header-Value\r\n
...
\r\n
[Optional Body]
```

### Example Response

```http
HTTP/1.1 200 OK
Date: Mon, 19 May 2026 10:00:00 GMT
Server: nginx/1.24.0
Content-Type: text/html; charset=UTF-8
Content-Length: 1024
Cache-Control: max-age=3600
Set-Cookie: session=xyz789; HttpOnly; Secure; SameSite=Strict
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Strict-Transport-Security: max-age=31536000; includeSubDomains

<!DOCTYPE html>
<html>...
```

---

## HTTP Methods

| Method | Description | Has Body | Idempotent | Safe |
|---|---|---|---|---|
| **GET** | Retrieve a resource | No | Yes | Yes |
| **POST** | Submit data to the server | Yes | No | No |
| **PUT** | Replace a resource entirely | Yes | Yes | No |
| **PATCH** | Partially update a resource | Yes | No | No |
| **DELETE** | Remove a resource | Optional | Yes | No |
| **HEAD** | Like GET but returns headers only | No | Yes | Yes |
| **OPTIONS** | List supported methods for a resource | No | Yes | Yes |
| **TRACE** | Echo the request back (debugging) | No | Yes | Yes |
| **CONNECT** | Establish a tunnel (used for HTTPS proxying) | No | No | No |

- **Safe** — does not modify server state.
- **Idempotent** — making the same request multiple times has the same effect as making it once.

> **Security note:** `TRACE` can be used in **XST (Cross-Site Tracing)** attacks. Disable it on production servers.

---

## HTTP Status Codes

### 1xx — Informational

| Code | Name | Description |
|---|---|---|
| 100 | Continue | Server received request headers; client should proceed |
| 101 | Switching Protocols | Server is switching protocols (e.g., to WebSocket) |

### 2xx — Success

| Code | Name | Description |
|---|---|---|
| 200 | OK | Request succeeded |
| 201 | Created | Resource successfully created (POST/PUT) |
| 204 | No Content | Success but no body returned (DELETE) |
| 206 | Partial Content | Range request fulfilled |

### 3xx — Redirection

| Code | Name | Description |
|---|---|---|
| 301 | Moved Permanently | Resource has a new permanent URL |
| 302 | Found | Temporary redirect |
| 303 | See Other | Redirect to a different URL (after POST) |
| 304 | Not Modified | Client cache is still valid |
| 307 | Temporary Redirect | Same as 302 but method must not change |
| 308 | Permanent Redirect | Same as 301 but method must not change |

### 4xx — Client Errors

| Code | Name | Description |
|---|---|---|
| 400 | Bad Request | Malformed request syntax |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Authenticated but not authorized |
| 404 | Not Found | Resource does not exist |
| 405 | Method Not Allowed | HTTP method not supported for this resource |
| 408 | Request Timeout | Client took too long to send request |
| 409 | Conflict | Request conflicts with current state |
| 410 | Gone | Resource permanently removed |
| 413 | Payload Too Large | Request body exceeds server limit |
| 414 | URI Too Long | URL too long |
| 415 | Unsupported Media Type | Content-Type not accepted |
| 422 | Unprocessable Entity | Validation errors (common in APIs) |
| 429 | Too Many Requests | Rate limit exceeded |

### 5xx — Server Errors

| Code | Name | Description |
|---|---|---|
| 500 | Internal Server Error | Generic server-side error |
| 501 | Not Implemented | Server doesn't support the request method |
| 502 | Bad Gateway | Upstream server returned an invalid response |
| 503 | Service Unavailable | Server overloaded or down for maintenance |
| 504 | Gateway Timeout | Upstream server timed out |

---

## HTTP Headers

### Request Headers

| Header | Purpose | Example |
|---|---|---|
| `Host` | Target hostname (required in HTTP/1.1) | `Host: www.example.com` |
| `User-Agent` | Client software info | `User-Agent: Mozilla/5.0 ...` |
| `Accept` | Acceptable response content types | `Accept: text/html, application/json` |
| `Accept-Encoding` | Acceptable compression formats | `Accept-Encoding: gzip, deflate, br` |
| `Accept-Language` | Preferred languages | `Accept-Language: en-US,en;q=0.9` |
| `Authorization` | Authentication credentials | `Authorization: Bearer <token>` |
| `Cookie` | Send cookies to server | `Cookie: session=abc123` |
| `Content-Type` | Body format | `Content-Type: application/json` |
| `Content-Length` | Body size in bytes | `Content-Length: 128` |
| `Referer` | URL of referring page | `Referer: https://example.com/page` |
| `Origin` | Origin of a CORS request | `Origin: https://app.example.com` |
| `If-None-Match` | Conditional request (ETag) | `If-None-Match: "abc123"` |
| `If-Modified-Since` | Conditional request (date) | `If-Modified-Since: Mon, 01 Jan 2026 00:00:00 GMT` |
| `X-Forwarded-For` | Client IP (set by proxies/LBs) | `X-Forwarded-For: 192.168.1.1` |

### Response Headers

| Header | Purpose | Example |
|---|---|---|
| `Content-Type` | Body format | `Content-Type: text/html; charset=UTF-8` |
| `Content-Length` | Body size | `Content-Length: 1024` |
| `Content-Encoding` | Compression applied | `Content-Encoding: gzip` |
| `Set-Cookie` | Set a cookie on the client | `Set-Cookie: id=abc; HttpOnly; Secure` |
| `Location` | Redirect target URL | `Location: https://example.com/new` |
| `Cache-Control` | Caching directives | `Cache-Control: no-store` |
| `ETag` | Resource version identifier | `ETag: "abc123"` |
| `Last-Modified` | Last modification time | `Last-Modified: Mon, 01 Jan 2026 00:00:00 GMT` |
| `WWW-Authenticate` | Auth challenge (with 401) | `WWW-Authenticate: Bearer realm="api"` |
| `Server` | Server software info | `Server: nginx/1.24.0` |
| `Vary` | Caching differentiation | `Vary: Accept-Encoding` |
| `Transfer-Encoding` | Transfer method | `Transfer-Encoding: chunked` |

---

## HTTP Security Headers

Security headers are response headers that instruct the browser to enable built-in security features. They are a critical layer of defense.

### Strict-Transport-Security (HSTS)

Forces browsers to always use HTTPS for the domain.

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

- `max-age` — how long (seconds) to enforce HTTPS.
- `includeSubDomains` — applies to all subdomains.
- `preload` — submit domain to browser HSTS preload lists (hardcoded in browsers).

**Protects against:** SSL stripping, protocol downgrade attacks.

---

### Content-Security-Policy (CSP)

Defines which sources browsers are allowed to load resources from.

```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.example.com; object-src 'none'; base-uri 'self'
```

| Directive | Controls |
|---|---|
| `default-src` | Fallback for all resource types |
| `script-src` | JavaScript sources |
| `style-src` | CSS sources |
| `img-src` | Image sources |
| `font-src` | Font sources |
| `connect-src` | XHR, Fetch, WebSocket targets |
| `frame-src` | `<iframe>` sources |
| `object-src` | `<object>`, `<embed>` — set to `'none'` |
| `base-uri` | `<base>` tag targets |
| `form-action` | Form submission targets |
| `upgrade-insecure-requests` | Auto-upgrade HTTP to HTTPS |
| `report-uri` / `report-to` | Where to send violation reports |

**Protects against:** XSS, clickjacking, data injection.

---

### X-Content-Type-Options

Prevents browsers from MIME-type sniffing (guessing content type).

```http
X-Content-Type-Options: nosniff
```

**Protects against:** MIME confusion attacks, drive-by downloads.

---

### X-Frame-Options

Controls whether the page can be loaded in an `<iframe>`.

```http
X-Frame-Options: DENY
X-Frame-Options: SAMEORIGIN
```

> **Note:** Superseded by CSP `frame-ancestors` directive, but still widely used for compatibility.

**Protects against:** Clickjacking.

---

### Referrer-Policy

Controls how much referrer information is included in requests.

```http
Referrer-Policy: strict-origin-when-cross-origin
```

| Value | Behavior |
|---|---|
| `no-referrer` | No referrer sent |
| `origin` | Only origin (no path) |
| `strict-origin-when-cross-origin` | Full URL on same-origin; origin only cross-origin; nothing on HTTP→HTTPS downgrade |
| `unsafe-url` | Full URL always (leaks data) |

**Protects against:** Sensitive URL leakage via Referer header.

---

### Permissions-Policy (formerly Feature-Policy)

Controls which browser features the page can use.

```http
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=()
```

**Protects against:** Unauthorized use of device features, supply-chain attacks via third-party scripts.

---

### Cross-Origin Headers (CORS & Beyond)

```http
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Resource-Policy: same-origin
```

- **COOP** — prevents cross-origin windows from accessing your window object.
- **COEP** — requires all sub-resources to opt in to cross-origin loading.
- **CORP** — prevents other origins from loading this resource.

---

## Cookies

Cookies are small pieces of data set by the server (`Set-Cookie`) and sent back by the client (`Cookie`) on every subsequent request.

### Cookie Attributes

| Attribute | Description |
|---|---|
| `HttpOnly` | Cookie inaccessible to JavaScript (`document.cookie`) — prevents XSS theft |
| `Secure` | Cookie only sent over HTTPS connections |
| `SameSite` | Controls cross-site cookie sending behavior |
| `Domain` | Which domains receive the cookie |
| `Path` | Which URL paths the cookie applies to |
| `Expires` / `Max-Age` | Cookie lifetime — omitting makes it a session cookie |

### SameSite Values

| Value | Behavior |
|---|---|
| `Strict` | Cookie never sent on cross-site requests |
| `Lax` | Cookie sent on top-level navigations (GET); not on AJAX, iframes (default in modern browsers) |
| `None` | Cookie always sent cross-site — **must** also set `Secure` |

### Secure Cookie Example

```http
Set-Cookie: session=abc123xyz; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=3600
```

### Cookie Security Issues

| Issue | Description | Mitigation |
|---|---|---|
| **Session hijacking** | Cookie stolen via XSS or network sniffing | `HttpOnly`, `Secure`, HTTPS |
| **CSRF** | Forged cross-site request uses victim's cookie | `SameSite=Strict/Lax`, CSRF tokens |
| **Cookie fixation** | Attacker sets a known session ID before auth | Regenerate session ID on login |
| **Overly broad scope** | Cookie sent to all subdomains | Restrict `Domain` and `Path` |

---

## CORS (Cross-Origin Resource Sharing)

**CORS** is a browser security mechanism that controls how scripts from one **origin** (scheme + host + port) can request resources from a **different origin**.

### Same-Origin Policy (SOP)

By default, browsers block cross-origin requests made by JavaScript (XMLHttpRequest, Fetch API). This is the **Same-Origin Policy**.

```
https://app.example.com → can fetch → https://api.example.com/data?
                                       ↑ Different subdomain = cross-origin
```

### CORS Headers

```http
# Server response allowing cross-origin requests
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400
```

### Preflight Request

For "complex" requests (non-simple methods, custom headers), the browser sends an `OPTIONS` preflight first:

```http
OPTIONS /api/data HTTP/1.1
Host: api.example.com
Origin: https://app.example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type, Authorization
```

### CORS Misconfigurations

| Misconfiguration | Risk |
|---|---|
| `Access-Control-Allow-Origin: *` with `Allow-Credentials: true` | Browsers block this combination — but misconfigured apps may try |
| Reflecting `Origin` header without validation | Any origin can make credentialed requests |
| Trusting `null` origin | Sandboxed iframes or `file://` pages can make requests |
| Overly permissive regex | `evil-example.com` matches `.*example.com` |

---

## TLS / SSL (The "S" in HTTPS)

**TLS (Transport Layer Security)** is the cryptographic protocol that secures HTTPS connections. Its predecessor was **SSL (Secure Sockets Layer)** — now deprecated and insecure.

### TLS Versions

| Version | Status | Notes |
|---|---|---|
| SSL 2.0 | Broken — disabled | Multiple critical vulnerabilities |
| SSL 3.0 | Broken — disabled | POODLE attack |
| TLS 1.0 | Deprecated (RFC 8996) | BEAST, POODLE downgrade |
| TLS 1.1 | Deprecated (RFC 8996) | Insufficient cipher support |
| TLS 1.2 | Widely used — acceptable | Safe with correct cipher configuration |
| TLS 1.3 | Current standard | Faster, fewer round trips, forward secrecy mandatory |

### TLS 1.2 Handshake

```
Client                              Server
  |── ClientHello ──────────────────→|  (TLS version, cipher suites, random)
  |←── ServerHello ─────────────────|  (chosen cipher, random, session ID)
  |←── Certificate ─────────────────|  (server's X.509 certificate)
  |←── ServerHelloDone ─────────────|
  |── ClientKeyExchange ────────────→|  (pre-master secret, encrypted with server pub key)
  |── ChangeCipherSpec ──────────────→|
  |── Finished ──────────────────────→|
  |←── ChangeCipherSpec ─────────────|
  |←── Finished ─────────────────────|
  |══════ Encrypted Application Data ══════|
```

### TLS 1.3 Handshake (Faster)

```
Client                              Server
  |── ClientHello + KeyShare ────────→|  (includes key exchange in first message)
  |←── ServerHello + KeyShare ────────|
  |←── {Certificate + Finished} ──────|
  |── {Finished} ─────────────────────→|
  |══════ Encrypted Application Data ══════|
```

TLS 1.3 requires only **1 round trip** (1-RTT) vs 2 for TLS 1.2. With session resumption: **0-RTT**.

### Cipher Suites

A cipher suite specifies the algorithms used for:
- **Key exchange** — how session keys are agreed upon.
- **Authentication** — how server identity is verified.
- **Bulk encryption** — how data is encrypted.
- **MAC / Hash** — how integrity is verified.

**TLS 1.2 example:**
```
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
 │      │    │       │    │    └─ Hash (SHA-384)
 │      │    │       │    └─ Mode (GCM = AEAD)
 │      │    │       └─ Key size (256-bit)
 │      │    └─ Bulk cipher (AES)
 │      └─ Authentication (RSA)
 └─ Key exchange (ECDHE = Elliptic Curve Diffie-Hellman Ephemeral)
```

**TLS 1.3** only allows 5 cipher suites (all AEAD):
- `TLS_AES_256_GCM_SHA384`
- `TLS_CHACHA20_POLY1305_SHA256`
- `TLS_AES_128_GCM_SHA256`

### Forward Secrecy (Perfect Forward Secrecy — PFS)

With **PFS**, session keys are generated fresh for every connection using **ephemeral key exchange (DHE/ECDHE)**. Even if the server's private key is compromised later, past sessions cannot be decrypted.

- Required in TLS 1.3.
- Look for `ECDHE` or `DHE` in cipher suites for TLS 1.2.

---

## TLS Certificates

### X.509 Certificate

A TLS certificate binds a **public key** to a **domain name**, verified by a **Certificate Authority (CA)**.

```
Certificate Fields:
  Subject: CN=example.com, O=Example Inc, C=US
  Issuer: Let's Encrypt Authority R3
  Valid From: 2026-01-01
  Valid To:   2026-04-01
  Public Key: RSA 2048-bit / EC 256-bit
  SANs: example.com, www.example.com
  Signature Algorithm: SHA256WithRSA
```

### Certificate Types

| Type | Validation Level | Use Case |
|---|---|---|
| **DV (Domain Validation)** | Domain ownership only | Blogs, personal sites |
| **OV (Organization Validation)** | Domain + organization verified | Business websites |
| **EV (Extended Validation)** | Rigorous org verification | Banks, e-commerce |
| **Wildcard** | Covers `*.example.com` | All subdomains |
| **Multi-SAN** | Multiple domains in one cert | SaaS platforms |

### Certificate Authorities (CAs)

- **Public CAs** — trusted by browsers (DigiCert, Comodo, Let's Encrypt, GlobalSign).
- **Private CAs** — internal use only (must be manually trusted by clients).
- **Let's Encrypt** — free, automated, 90-day DV certificates.

### Certificate Chain of Trust

```
Root CA (self-signed, in browser trust store)
  └── Intermediate CA (signed by Root CA)
        └── Server Certificate (signed by Intermediate CA)
```

Browsers verify the full chain up to a trusted root.

### Certificate Transparency (CT)

All publicly-trusted certificates must be logged in public **CT logs** (RFC 6962). This allows anyone to audit issued certificates and detect misissuance.

```bash
# View CT logs for a domain
curl "https://crt.sh/?q=example.com&output=json"
```

### Certificate Revocation

If a certificate is compromised, it must be revoked:

- **CRL (Certificate Revocation List)** — downloadable list of revoked certs. Slow, large.
- **OCSP (Online Certificate Status Protocol)** — real-time revocation check. Privacy concerns.
- **OCSP Stapling** — server fetches OCSP response and includes it in the TLS handshake.

---

## HTTP Authentication

### Basic Authentication

Credentials encoded in Base64 and sent in the `Authorization` header.

```http
Authorization: Basic dXNlcjpwYXNzd29yZA==
```

Decoded: `user:password`

> **Security:** Base64 is **not encryption**. Never use Basic Auth over HTTP — always HTTPS. Even over HTTPS, credentials are exposed if logs are not protected.

### Digest Authentication

Challenge-response scheme using MD5 hashing — avoids sending password in plaintext.

> **Security:** MD5 is weak; Digest Auth is rarely used today. Use token-based auth instead.

### Bearer Token (OAuth 2.0)

Token issued by auth server, included in every request.

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### API Key

```http
X-API-Key: my-secret-api-key
Authorization: ApiKey my-secret-api-key
```

### JWT (JSON Web Token)

Self-contained token with header, payload, and signature.

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ1c2VyMTIzIn0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
 └─── Header ───┘  └──── Payload ────┘  └─────── Signature ──────┘
```

**JWT Security Issues:**
- `alg: none` — algorithm confusion attack; always validate algorithm.
- Weak secret for HMAC — brute-forceable offline.
- Sensitive data in payload — payload is Base64-encoded, not encrypted (use JWE for encryption).
- No revocation — compromised JWTs valid until expiry.

---

## HTTP Caching

### Cache-Control Directives

```http
Cache-Control: max-age=3600, public
Cache-Control: no-store
Cache-Control: no-cache
Cache-Control: private, max-age=0
```

| Directive | Meaning |
|---|---|
| `max-age=N` | Cache for N seconds |
| `no-store` | Never cache — sensitive data |
| `no-cache` | Cache but revalidate before each use |
| `public` | Any cache (CDN, proxy, browser) can store |
| `private` | Only browser cache (not CDN/proxy) |
| `immutable` | Content will never change (forever cache) |
| `must-revalidate` | Must revalidate with server when stale |

### Caching Security Considerations

- Use `Cache-Control: no-store` for sensitive responses (auth pages, personal data).
- Ensure `private` is set for user-specific content to prevent CDN caching.
- Avoid caching pages with CSRF tokens.
- Beware of **cache poisoning** attacks — attackers pollute shared caches with malicious responses.

---

## HTTP/2 & HTTP/3 Security

### HTTP/2 Specific Attacks

- **HTTP/2 Rapid Reset (CVE-2023-44487)** — attacker sends and immediately cancels streams in bulk, exhausting server resources. Largest DDoS in history used this vector.
- **Header compression side-channels (CRIME/BREACH)** — if TLS compression is enabled alongside HTTP compression, secrets can be leaked byte-by-byte.

**Mitigations:** Disable TLS-layer compression; patch servers; rate-limit RST_STREAM frames.

### HTTP/3 / QUIC Considerations

- QUIC runs on **UDP** — some firewalls block it; browsers fall back to HTTP/2.
- **0-RTT replay attacks** — early data in 0-RTT can be replayed by attackers; avoid non-idempotent operations in 0-RTT.
- Network monitoring tools may not yet fully support QUIC traffic analysis.

---

## Common HTTP Attacks

### 1. HTTP Request Smuggling

**What it is:** Attacker exploits discrepancies between how a **front-end proxy** and **back-end server** interpret HTTP request boundaries.

**How it works:**
```
Front-end uses Content-Length to determine request end
Back-end uses Transfer-Encoding: chunked
Attacker crafts request that embeds a second hidden request
```

**Impact:** Bypass security controls, poison cache, hijack other users' requests, SSRF.

**Mitigations:**
- Use HTTP/2 end-to-end (eliminates the ambiguity).
- Configure front-end and back-end to use the same parsing.
- Normalize ambiguous requests at the proxy.

---

### 2. HTTP Response Splitting

**What it is:** Attacker injects CRLF (`\r\n`) characters into HTTP response headers via unsanitized user input, allowing them to inject arbitrary headers or split the response.

**Example:**
```
Input: ?redirect=https://example.com%0d%0aSet-Cookie:%20malicious=true
Injected header: Set-Cookie: malicious=true
```

**Mitigations:** Sanitize and encode all user-supplied values used in HTTP headers; never include raw user input in response headers.

---

### 3. Clickjacking

**What it is:** Victim is tricked into clicking on a transparent `<iframe>` overlay of the target site.

**Mitigations:** `X-Frame-Options: DENY`, CSP `frame-ancestors 'none'`.

---

### 4. SSL Stripping

**What it is:** Attacker (acting as MitM) intercepts HTTP traffic before it is upgraded to HTTPS, keeping the victim on HTTP while maintaining an HTTPS connection to the server.

```
Victim → HTTP → Attacker → HTTPS → Server
               ↑ Attacker sees plaintext
```

**Mitigations:** HSTS, HSTS preloading, always redirect HTTP → HTTPS.

---

### 5. MIME Sniffing

**What it is:** Browser ignores `Content-Type` header and guesses the content type — can cause a `.txt` file containing HTML/JS to be executed.

**Mitigations:** `X-Content-Type-Options: nosniff`.

---

### 6. Cache Poisoning

**What it is:** Attacker poisons a shared cache (CDN, proxy) with a malicious response, which is then served to other users.

**Mitigations:** Validate and normalize cache keys; use `Vary` headers correctly; avoid caching based on user-controllable headers.

---

### 7. HTTP Parameter Pollution (HPP)

**What it is:** Sending multiple parameters with the same name — different servers/frameworks handle duplicates differently, potentially bypassing input validation.

```
?id=1&id=2
```

**Mitigations:** Explicitly define expected parameter handling; validate inputs server-side.

---

## HTTPS Implementation Best Practices

### Server Configuration

```nginx
# Nginx HTTPS configuration example
server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate     /etc/ssl/certs/example.com.pem;
    ssl_certificate_key /etc/ssl/private/example.com.key;

    # TLS versions
    ssl_protocols TLSv1.2 TLSv1.3;

    # Cipher suites (TLS 1.2 only — 1.3 ciphers are automatic)
    ssl_ciphers ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305;
    ssl_prefer_server_ciphers off;  # Let client choose in TLS 1.3

    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;

    # Session
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;  # Disable for forward secrecy
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    return 301 https://$host$request_uri;
}
```

### Security Headers Checklist

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'; object-src 'none'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

### TLS Testing Tools

```bash
# Test SSL/TLS configuration
nmap --script ssl-enum-ciphers -p 443 example.com

# Comprehensive TLS audit
testssl.sh example.com

# Check for weak ciphers / protocols
sslscan example.com

# Online tools
# https://www.ssllabs.com/ssltest/
# https://securityheaders.com/
# https://observatory.mozilla.org/
```

---

## WebSockets & HTTP

**WebSockets** provide full-duplex communication over a single TCP connection, starting with an HTTP upgrade:

```http
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13

HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

**WebSocket Security:**
- Use `wss://` (WebSocket Secure) — never `ws://` in production.
- Validate `Origin` header to prevent cross-site WebSocket hijacking.
- Authenticate the WebSocket connection (token in URL or initial message).
- Apply rate limiting and message size limits.

---

## HTTP Proxy & Interception

### Forward Proxy

Client → Proxy → Server

Used for:
- Corporate web filtering
- Anonymization
- Caching
- Security testing (Burp Suite, OWASP ZAP, mitmproxy)

### Intercepting HTTPS with a Proxy

Tools like **Burp Suite** act as a MitM proxy:
1. Install proxy's CA certificate in browser.
2. Proxy terminates TLS with the browser, re-encrypts to the server.
3. Proxy can now read and modify all HTTPS traffic.

**This only works if the proxy CA is trusted by the browser** — why pinning and certificate validation matter.

### Certificate Pinning

Applications can **pin** a specific certificate or public key, refusing connections that present any other certificate — even from a trusted CA.

- Prevents MitM by malicious CAs or proxy tools.
- Common in mobile apps.
- Bypass techniques exist for rooted/jailbroken devices (`Frida`, `objection`).

---

## Quick Reference

### Ports

| Protocol | Port |
|---|---|
| HTTP | 80 |
| HTTPS | 443 |
| HTTP/3 (QUIC) | UDP 443 |
| HTTP proxy | 8080, 3128 |
| HTTPS proxy | 8443 |

### HTTP Methods Summary

| Method | Purpose | Safe | Idempotent |
|---|---|---|---|
| GET | Read | Yes | Yes |
| POST | Create | No | No |
| PUT | Replace | No | Yes |
| PATCH | Update | No | No |
| DELETE | Remove | No | Yes |
| HEAD | Headers only | Yes | Yes |
| OPTIONS | Capabilities | Yes | Yes |

### Security Headers Cheatsheet

| Header | Protects Against |
|---|---|
| HSTS | SSL stripping, downgrade |
| CSP | XSS, data injection |
| X-Content-Type-Options | MIME sniffing |
| X-Frame-Options | Clickjacking |
| Referrer-Policy | Data leakage via Referer |
| Permissions-Policy | Feature abuse |

### Common Attacks Summary

| Attack | Vector | Defense |
|---|---|---|
| SSL Stripping | MitM on HTTP→HTTPS upgrade | HSTS, preload |
| Clickjacking | Transparent iframe overlay | X-Frame-Options, CSP |
| Request Smuggling | CL/TE header desync | HTTP/2 end-to-end, normalize |
| Cache Poisoning | Malicious cache entry | Validate cache keys, Vary headers |
| MIME Sniffing | Browser guesses content type | X-Content-Type-Options: nosniff |
| Cookie Theft | XSS reads document.cookie | HttpOnly, Secure, SameSite |
| CSRF | Forged cross-site request | SameSite, CSRF tokens |
| JWT Confusion | alg:none / weak secret | Validate algorithm, strong secrets |

---

> **Note:** This document is part of the [Cybersecurity-Basics](../README.md) notes repository. It is intended for learning purposes. Always practice in a legal, authorized environment.

*Last updated: 2026*