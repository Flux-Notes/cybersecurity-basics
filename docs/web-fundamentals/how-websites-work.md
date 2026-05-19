# How Websites Work

---

## What is a Website?

A **website** is a collection of web pages and related resources (images, stylesheets, scripts, fonts, etc.) accessible via the internet or an intranet through a **web browser**. It is identified by a **domain name** and hosted on one or more **web servers**.

When you visit a website, your browser performs a complex series of steps — DNS resolution, TCP connection, TLS handshake, HTTP request/response, and rendering — all typically completing in under a second.

---

## The Big Picture

```
User types: https://www.example.com/products

1. DNS Resolution       → "What IP is www.example.com?"
2. TCP Connection       → 3-way handshake to port 443
3. TLS Handshake        → Negotiate encryption, verify certificate
4. HTTP Request         → GET /products HTTP/1.1
5. Server Processing    → Web server + application + database
6. HTTP Response        → 200 OK + HTML/CSS/JS
7. Browser Rendering    → Parse HTML, load assets, execute JS, paint pixels
```

---

## Step 1 — DNS Resolution

Before any connection is made, the browser must resolve the domain name to an IP address.

```
Browser Cache → OS Cache → /etc/hosts → Recursive Resolver
  → Root NS → TLD NS → Authoritative NS → IP returned
```

- Result: `www.example.com → 93.184.216.34`
- Cached for the record's **TTL** duration.
- On repeat visits, the IP is served from cache — no DNS query needed.

> See `DNS.md` for the full DNS resolution deep dive.

---

## Step 2 — TCP Connection

With an IP address in hand, the browser establishes a **TCP connection** to the server on **port 443** (HTTPS) or **port 80** (HTTP).

### TCP 3-Way Handshake

```
Client                    Server
  |──── SYN ────────────→|   "I want to connect"
  |←─── SYN-ACK ─────────|   "OK, I'm ready"
  |──── ACK ────────────→|   "Great, let's go"
  |═══ Connection Open ══|
```

- **SYN** — synchronize sequence numbers.
- **SYN-ACK** — server acknowledges and sends its own sequence number.
- **ACK** — client acknowledges server's sequence number.

This adds **1 round-trip time (RTT)** before any data is sent.

---

## Step 3 — TLS Handshake (HTTPS Only)

After the TCP connection, a **TLS handshake** negotiates encryption parameters and authenticates the server.

### TLS 1.3 Handshake (1-RTT)

```
Client                              Server
  |── ClientHello + KeyShare ──────→|
  |←── ServerHello + KeyShare ───────|
  |←── {Certificate + Finished} ─────|
  |── {Finished} ───────────────────→|
  |══════ Encrypted data flows ══════|
```

After the handshake:
- A **symmetric session key** is established (e.g., AES-256-GCM).
- All further communication is encrypted.
- The server's identity has been verified via its **X.509 certificate**.

> See `HTTP_HTTPS.md` for TLS cipher suites, certificate types, and forward secrecy.

---

## Step 4 — HTTP Request

The browser sends an **HTTP request** for the resource.

```http
GET /products HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Connection: keep-alive
Cookie: session=abc123; preferences=dark-mode
```

---

## Step 5 — Server Processing

### Web Server

The **web server** is the software that receives HTTP requests and returns responses.

| Web Server | Notes |
|---|---|
| **Nginx** | High-performance; reverse proxy, load balancer, static files |
| **Apache HTTP Server** | Widely used; modular; `.htaccess` support |
| **IIS (Internet Information Services)** | Windows-native; integrates with .NET |
| **Caddy** | Automatic HTTPS; simple config |
| **LiteSpeed** | High performance; cPanel compatible |

The web server either:
- Serves a **static file** directly (HTML, image, CSS) from disk.
- Passes the request to an **application server** for dynamic processing.

### Application Server

The **application server** executes business logic and generates dynamic responses.

| Language / Framework | Example Servers |
|---|---|
| Python | Django, Flask (Gunicorn, uWSGI) |
| Node.js | Express, Fastify (runs natively) |
| PHP | Laravel, WordPress (PHP-FPM) |
| Ruby | Ruby on Rails (Puma, Unicorn) |
| Java | Spring Boot (Tomcat, Jetty) |
| Go | Gin, Echo (compiled binary) |
| .NET | ASP.NET Core (Kestrel) |

### Database

Most dynamic websites query a **database** to retrieve or store data.

| Type | Examples | Use Case |
|---|---|---|
| **Relational (SQL)** | MySQL, PostgreSQL, SQLite, MSSQL | Structured data, transactions |
| **Document (NoSQL)** | MongoDB, CouchDB | Flexible schemas, JSON data |
| **Key-Value** | Redis, Memcached | Caching, sessions, leaderboards |
| **Search** | Elasticsearch, Solr | Full-text search |
| **Graph** | Neo4j | Relationships, social networks |

### Request Flow (Dynamic Website)

```
Browser
  │
  ▼
Web Server (Nginx)           ← handles SSL, static files, routing
  │
  ▼
Application Server (Gunicorn/Node) ← runs business logic
  │
  ├──→ Database (PostgreSQL)  ← query/store data
  ├──→ Cache (Redis)          ← check cache first
  ├──→ External APIs          ← payment, auth, maps, etc.
  │
  ▼
HTTP Response → Browser
```

---

## Step 6 — HTTP Response

The server sends back an HTTP response.

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Encoding: gzip
Content-Length: 4821
Cache-Control: public, max-age=3600
Set-Cookie: session=xyz789; HttpOnly; Secure; SameSite=Strict
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff

<!DOCTYPE html>
<html lang="en">
<head>
  <title>Products – Example</title>
  <link rel="stylesheet" href="/static/css/main.css">
</head>
<body>
  <h1>Our Products</h1>
  <script src="/static/js/app.js"></script>
</body>
</html>
```

---

## Step 7 — Browser Rendering

Rendering is how the browser turns raw HTML/CSS/JS into pixels on screen. It is a multi-stage pipeline.

### The Critical Rendering Path

```
HTML bytes received
  │
  ▼
1. Parse HTML → DOM (Document Object Model)
  │
  ▼
2. Parse CSS → CSSOM (CSS Object Model)
  │
  ▼
3. DOM + CSSOM → Render Tree (only visible elements)
  │
  ▼
4. Layout (Reflow) → Calculate position and size of every element
  │
  ▼
5. Paint → Draw pixels (text, colors, images, borders)
  │
  ▼
6. Composite → Layers merged and displayed on screen
```

### DOM (Document Object Model)

- The browser parses HTML into a **tree of nodes** (elements, text, attributes).
- JavaScript can read and modify the DOM via `document.getElementById()`, `querySelector()`, etc.
- DOM manipulation is how modern web apps dynamically update the page without reloading.

```
Document
└── html
    ├── head
    │   ├── title → "Products"
    │   └── link  → main.css
    └── body
        ├── h1 → "Our Products"
        └── script → app.js
```

### CSSOM (CSS Object Model)

- CSS is parsed into a separate object model.
- Combined with the DOM to produce the **Render Tree**.
- CSS is **render-blocking** — the browser won't render until CSS is parsed.

### JavaScript Execution

- `<script>` tags without `async`/`defer` are **parser-blocking** — HTML parsing stops until the script is downloaded and executed.
- `async` — download in parallel, execute as soon as available.
- `defer` — download in parallel, execute after HTML is fully parsed.
- Modern JS frameworks (React, Vue, Angular) manipulate the DOM extensively via JavaScript.

### Sub-resource Loading

After parsing HTML, the browser identifies and fetches additional resources:

```
HTML parsed → discover:
  ├── CSS files   → <link rel="stylesheet">
  ├── JS files    → <script src="...">
  ├── Images      → <img src="...">
  ├── Fonts       → @font-face in CSS
  ├── Iframes     → <iframe src="...">
  └── Fetch/XHR   → JavaScript API calls
```

Each sub-resource triggers its own DNS + TCP + TLS + HTTP cycle (connections are reused via `keep-alive` or HTTP/2 multiplexing).

---

## Static vs Dynamic Websites

### Static Website

- Pre-built HTML/CSS/JS files served directly by the web server.
- No server-side processing per request.
- Fast, cheap to host, highly cacheable.
- **Examples:** Landing pages, blogs (Hugo, Jekyll), documentation sites.

```
Browser → Web Server → File on disk → Response
```

### Dynamic Website

- Content generated on the fly by an application server, often from a database.
- Personalized content per user (dashboard, shopping cart, social feeds).
- More complex infrastructure.
- **Examples:** E-commerce, social media, SaaS apps.

```
Browser → Web Server → App Server → Database → Response
```

### JAMstack / SSG (Static Site Generation)

Modern hybrid approach:
- Site is **pre-built** at deploy time (not per request).
- Dynamic features handled by **APIs** and client-side JavaScript.
- Hosted on CDN edges — extremely fast globally.
- **Examples:** Next.js (SSG mode), Gatsby, Astro, Vercel, Netlify.

---

## Web Architecture Components

### CDN (Content Delivery Network)

A CDN caches copies of website assets at **edge servers** distributed globally. Users are served from the nearest edge node.

```
User in Chennai
  │
  ▼
CDN Edge (Mumbai) ← cache hit → response in ~5ms
  │ (cache miss)
  ▼
Origin Server (US) → ~200ms
```

**Benefits:** Reduced latency, DDoS absorption, reduced origin load.

**Examples:** Cloudflare, AWS CloudFront, Fastly, Akamai.

### Load Balancer

Distributes incoming requests across multiple backend servers.

```
                    ┌── App Server 1
Internet → LB ──────┼── App Server 2
                    └── App Server 3
```

**Algorithms:** Round-robin, least connections, IP hash.

**Types:** L4 (TCP/UDP), L7 (HTTP-aware — can route by URL, headers, cookies).

### Reverse Proxy

Sits in front of web servers, forwarding client requests to appropriate backends.

**Functions:** SSL termination, caching, rate limiting, WAF, compression, routing.

**Examples:** Nginx, HAProxy, Traefik, Cloudflare.

### Cache Layers

| Layer | What's Cached | Tool |
|---|---|---|
| Browser cache | Static assets (CSS, JS, images) | Browser built-in |
| CDN cache | Static + some dynamic content | Cloudflare, CloudFront |
| Application cache | DB query results, computed data | Redis, Memcached |
| Database cache | Query result sets | Built-in (InnoDB buffer pool) |
| Object storage | Large files, media | S3, GCS |

### Message Queues / Background Jobs

Asynchronous tasks are offloaded to queues so the HTTP response isn't delayed:

```
User uploads image
  → HTTP 202 Accepted (instant)
  → Job queued (RabbitMQ / SQS / Redis)
  → Worker processes image resize, thumbnail generation
  → Notify user when done
```

**Tools:** Celery, Sidekiq, BullMQ, AWS SQS.

---

## Frontend Technologies

### HTML (HyperText Markup Language)

The **structure** of a web page. Defines elements like headings, paragraphs, links, forms, images.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Example</title>
</head>
<body>
  <h1>Hello World</h1>
  <a href="/about">About</a>
  <form action="/login" method="POST">
    <input type="text" name="username">
    <input type="password" name="password">
    <button type="submit">Login</button>
  </form>
</body>
</html>
```

### CSS (Cascading Style Sheets)

The **presentation** of a web page. Controls layout, colors, fonts, animations.

```css
body {
  font-family: sans-serif;
  background: #f5f5f5;
}

h1 {
  color: #333;
  font-size: 2rem;
}
```

### JavaScript

The **behaviour** of a web page. Makes pages interactive, fetches data, manipulates the DOM.

```javascript
// Fetch data from an API
fetch('/api/products')
  .then(res => res.json())
  .then(data => {
    document.getElementById('products').innerHTML =
      data.map(p => `<li>${p.name}</li>`).join('');
  });
```

### JavaScript Frameworks

| Framework | Type | Notes |
|---|---|---|
| **React** | Library (UI) | Component-based; virtual DOM; Meta |
| **Vue.js** | Framework | Progressive; gentle learning curve |
| **Angular** | Full framework | TypeScript; opinionated; Google |
| **Svelte** | Compiler | No virtual DOM; compiles to vanilla JS |
| **Next.js** | React meta-framework | SSR + SSG + API routes |
| **Nuxt.js** | Vue meta-framework | SSR + SSG for Vue |

### Browser Storage

| Type | Size | Persists | Accessible by JS | Notes |
|---|---|---|---|---|
| **Cookies** | ~4 KB | Yes (configurable) | Yes (unless HttpOnly) | Sent with every request |
| **localStorage** | ~5–10 MB | Yes (until cleared) | Yes | Same-origin only |
| **sessionStorage** | ~5–10 MB | No (tab session) | Yes | Cleared on tab close |
| **IndexedDB** | Hundreds of MB | Yes | Yes | Structured async DB in browser |
| **Cache API** | Large | Yes | Yes | Service worker caching |

> **Security note:** Never store sensitive data (tokens, passwords, PII) in `localStorage` or `sessionStorage` — accessible to any JavaScript on the page (including XSS).

---

## Backend Technologies

### Server-Side Rendering (SSR)

HTML is generated **on the server** per request and sent to the browser.

```
Browser → Request → Server generates HTML → Response → Browser renders
```

**Pros:** Better SEO, faster first paint on slow devices.
**Cons:** Higher server load, slower time-to-first-byte on complex pages.

### Client-Side Rendering (CSR)

A minimal HTML shell is returned; JavaScript **builds the page in the browser**.

```
Browser → Request → Minimal HTML + JS bundle → Browser runs JS → Renders page
```

**Pros:** Rich interactivity, fast subsequent navigation (SPA).
**Cons:** Poor SEO (without extra config), slow initial load, heavier on client.

### APIs (Application Programming Interfaces)

Modern websites communicate with backends via APIs.

#### REST (Representational State Transfer)

- Resources identified by **URLs**.
- Standard HTTP methods (GET, POST, PUT, DELETE).
- Stateless — each request is self-contained.
- Returns **JSON** (or XML).

```
GET    /api/users          → list users
POST   /api/users          → create user
GET    /api/users/42       → get user 42
PUT    /api/users/42       → replace user 42
PATCH  /api/users/42       → update user 42 partially
DELETE /api/users/42       → delete user 42
```

#### GraphQL

- Single endpoint (`/graphql`).
- Client specifies exactly what data it needs.
- Reduces over-fetching and under-fetching.

```graphql
query {
  user(id: 42) {
    name
    email
    posts {
      title
    }
  }
}
```

#### WebSockets

- Full-duplex persistent connection.
- Used for real-time features: chat, live dashboards, notifications, multiplayer games.

---

## Sessions & Authentication

### Session-Based Authentication

```
1. User submits credentials (POST /login)
2. Server validates, creates a session in DB/Redis
3. Server sends back Set-Cookie: session_id=abc123
4. Browser includes Cookie: session_id=abc123 on every request
5. Server looks up session in store to identify user
```

### Token-Based Authentication (JWT)

```
1. User submits credentials
2. Server validates, returns a signed JWT
3. Client stores JWT (memory, cookie, or localStorage)
4. Client sends JWT in Authorization: Bearer <token> header
5. Server verifies signature — no DB lookup needed (stateless)
```

### OAuth 2.0 / OpenID Connect

Used for **"Login with Google/GitHub/Facebook"** flows.

```
User clicks "Login with Google"
  │
  ▼
Redirect to Google authorization server
  │ (user logs in + grants permission)
  ▼
Google redirects back with authorization code
  │
  ▼
App exchanges code for access token (server-to-server)
  │
  ▼
App uses access token to fetch user profile
```

---

## How Forms Work

HTML forms are how users submit data to a server.

```html
<form action="/login" method="POST" enctype="application/x-www-form-urlencoded">
  <input type="text"     name="username" value="alice">
  <input type="password" name="password" value="secret">
  <button type="submit">Login</button>
</form>
```

Submission sends:

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 33

username=alice&password=secret
```

### Form Encoding Types

| `enctype` | Format | Use Case |
|---|---|---|
| `application/x-www-form-urlencoded` | `key=value&key2=value2` | Default; text data |
| `multipart/form-data` | MIME multipart | File uploads |
| `application/json` | JSON (via JS fetch) | API calls |

---

## Security Considerations

### Input Validation & Sanitization

Every piece of user input is a potential attack vector.

| Attack | Unsanitized Input Example | Mitigation |
|---|---|---|
| **SQL Injection** | `' OR '1'='1` in login form | Parameterized queries / ORM |
| **XSS** | `<script>alert(1)</script>` in comment | HTML encode output; CSP |
| **Path Traversal** | `../../etc/passwd` as filename | Validate and sanitize file paths |
| **Command Injection** | `; rm -rf /` in search field | Never pass user input to shell |
| **SSRF** | Internal URL in image fetch field | Allowlist external URLs |

### HTTPS Everywhere

- All websites should use HTTPS — even static sites.
- Free certificates via **Let's Encrypt**.
- Use **HSTS** to prevent downgrade attacks.
- Redirect all HTTP traffic to HTTPS.

### Same-Origin Policy

Browsers enforce the **Same-Origin Policy (SOP)**: JavaScript on `https://a.com` cannot read responses from `https://b.com` without explicit CORS permission.

- **Origin** = scheme + host + port.
- Prevents malicious sites from reading your banking data.
- Does not prevent requests from being sent — only the response from being read.

### Content Security Policy (CSP)

Defines where resources can be loaded from, reducing XSS impact.

```http
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-abc123'
```

### CSRF (Cross-Site Request Forgery)

Attacker tricks a logged-in user into unknowingly submitting a malicious request.

```html
<!-- Malicious page visits bank.com as the logged-in user -->
<img src="https://bank.com/transfer?to=attacker&amount=10000">
```

**Mitigations:** `SameSite` cookies, **CSRF tokens** in forms, `Origin`/`Referer` header validation.

### Clickjacking

Victim clicks on a transparent `<iframe>` overlay of a legitimate site.

```html
<!-- Attacker's page -->
<iframe src="https://bank.com/transfer" style="opacity:0; position:absolute;"></iframe>
<button>Click me to win a prize!</button>
```

**Mitigations:** `X-Frame-Options: DENY`, CSP `frame-ancestors 'none'`.

---

## Browser Developer Tools

Every modern browser includes **DevTools** — essential for understanding and testing websites.

| Panel | What You Can Do |
|---|---|
| **Elements** | Inspect and modify DOM/CSS live |
| **Console** | Run JavaScript, view errors and logs |
| **Network** | View all HTTP requests/responses, headers, timing |
| **Sources** | View JavaScript source, set breakpoints |
| **Application** | Inspect cookies, localStorage, sessionStorage, IndexedDB, Service Workers |
| **Security** | View TLS certificate, connection details |
| **Performance** | Profile rendering, JS execution, paint events |
| **Lighthouse** | Automated audit for performance, SEO, accessibility, security |

### Network Tab Tips (Security Testing)

```
1. Open DevTools (F12) → Network tab
2. Reload page or interact
3. Click any request to see:
   - Request URL, method, status code
   - Request headers (including cookies, auth tokens)
   - Response headers (security headers)
   - Response body
   - Timing (DNS, TCP, TLS, TTFB, download)
```

---

## Performance Concepts

### Key Metrics

| Metric | Description | Good Target |
|---|---|---|
| **TTFB** (Time to First Byte) | Time from request to first byte of response | < 200ms |
| **FCP** (First Contentful Paint) | When first content appears on screen | < 1.8s |
| **LCP** (Largest Contentful Paint) | When main content loads | < 2.5s |
| **TTI** (Time to Interactive) | When page is fully interactive | < 3.8s |
| **CLS** (Cumulative Layout Shift) | Visual stability (elements jumping around) | < 0.1 |

### Performance Optimizations

- **Minify** CSS/JS/HTML — remove whitespace and comments.
- **Compress** responses — gzip or Brotli.
- **Cache** static assets aggressively — long `max-age`, content-addressed filenames.
- **Use a CDN** — serve assets from edge servers near the user.
- **Lazy load** images and off-screen content.
- **Preconnect** to critical third-party origins.
- **HTTP/2 or HTTP/3** — multiplexing reduces connection overhead.

---

## How a Modern SPA (Single Page Application) Works

A **SPA** loads once and dynamically updates content without full page reloads.

```
Initial visit:
  Browser → Server → HTML shell + large JS bundle
  Browser runs JS → Framework boots → Renders initial view

Navigation (e.g., click "About"):
  JavaScript intercepts click → No new HTTP request for HTML
  JS updates URL (History API) → Fetches /api/about-data
  Re-renders only what changed → Instant feel

Deep link (direct URL to /about):
  Browser → Server → Server must return HTML shell for any path
  JS boots → Reads URL → Renders /about view
```

**Routing:**
- Client-side router (React Router, Vue Router) handles URL changes.
- Server must be configured to return the SPA shell for all routes (catch-all).

---

## Web Server Configuration Basics

### Virtual Hosting

Multiple websites on a single server using the `Host` header to distinguish them.

```nginx
server {
    server_name www.site-a.com;
    root /var/www/site-a;
}

server {
    server_name www.site-b.com;
    root /var/www/site-b;
}
```

### URL Routing

```nginx
location / {
    try_files $uri $uri/ /index.html;  # SPA catch-all
}

location /api/ {
    proxy_pass http://localhost:3000;  # Proxy to app server
}

location /static/ {
    root /var/www;
    expires 1y;                        # Long cache for static files
    add_header Cache-Control "public, immutable";
}
```

### Common Security Misconfigurations

| Misconfiguration | Risk | Fix |
|---|---|---|
| Directory listing enabled | Exposes file structure | `autoindex off` |
| Server version disclosure | Helps attacker fingerprint | `server_tokens off` |
| Default error pages | Leak framework/version info | Custom error pages |
| Unnecessary HTTP methods enabled | TRACE/PUT misuse | Restrict allowed methods |
| Open redirects | Phishing, token theft | Validate redirect URLs |
| Sensitive files exposed | `.git`, `.env`, `backup.zip` | Block in web server config |
| Missing security headers | XSS, clickjacking, etc. | Add HSTS, CSP, etc. |

---

## Useful Tools

### Browser-Based

| Tool | Use |
|---|---|
| **Browser DevTools** | Inspect requests, DOM, cookies, storage |
| **Wappalyzer** | Fingerprint technologies used by a site |
| **BuiltWith** | Technology profiler |

### CLI Tools

```bash
# Fetch a page and see headers
curl -I https://example.com
curl -v https://example.com

# Follow redirects
curl -L https://example.com

# Send a POST request
curl -X POST https://example.com/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"secret"}'

# Check response headers
curl -s -D - https://example.com -o /dev/null

# Trace DNS + connection timing
curl -w "\n\nDNS: %{time_namelookup}s\nTCP: %{time_connect}s\nTLS: %{time_appconnect}s\nTTFB: %{time_starttransfer}s\nTotal: %{time_total}s\n" \
  -s -o /dev/null https://example.com
```

### Security Testing Tools

| Tool | Use |
|---|---|
| **Burp Suite** | Intercept, modify, and replay HTTP requests |
| **OWASP ZAP** | Automated and manual web app scanning |
| **Nikto** | Web server vulnerability scanner |
| **Gobuster / feroxbuster** | Directory and file brute-forcing |
| **Waybackurls** | Find old URLs via Wayback Machine |
| **whatweb** | Web technology fingerprinting |

---

## Quick Reference

### Website Request Lifecycle

```
1. DNS   → Resolve domain to IP
2. TCP   → 3-way handshake (SYN, SYN-ACK, ACK)
3. TLS   → Handshake, negotiate ciphers, verify cert
4. HTTP  → Send request (method, headers, body)
5. Server → Web server → App server → DB → Response
6. HTTP  → Receive response (status, headers, body)
7. Render → Parse HTML → Build DOM → Apply CSS → Execute JS → Paint
```

### Key Technologies by Layer

| Layer | Technology |
|---|---|
| Network | TCP/IP, DNS, TLS |
| Protocol | HTTP/1.1, HTTP/2, HTTP/3 |
| Web Server | Nginx, Apache, IIS, Caddy |
| App Server | Node.js, Django, Rails, Spring |
| Database | PostgreSQL, MySQL, MongoDB, Redis |
| Frontend | HTML, CSS, JavaScript, React/Vue |
| Infrastructure | CDN, Load Balancer, Cache, Queue |

### Common Status Codes

| Code | Meaning |
|---|---|
| 200 | OK — success |
| 301 | Moved permanently |
| 302 | Temporary redirect |
| 304 | Not modified (use cache) |
| 400 | Bad request |
| 401 | Unauthenticated |
| 403 | Forbidden |
| 404 | Not found |
| 429 | Rate limited |
| 500 | Server error |
| 503 | Service unavailable |

### Security Checklist for Websites

```
[ ] HTTPS enforced with valid TLS certificate
[ ] HTTP redirects to HTTPS (301)
[ ] HSTS header set (with preload)
[ ] Content-Security-Policy configured
[ ] X-Content-Type-Options: nosniff
[ ] X-Frame-Options: DENY (or CSP frame-ancestors)
[ ] Referrer-Policy set
[ ] Cookies: HttpOnly + Secure + SameSite=Strict
[ ] Input validation on all user-supplied data
[ ] Parameterized queries (no SQL injection)
[ ] No sensitive data in localStorage
[ ] CSRF protection on state-changing requests
[ ] Rate limiting on login / sensitive endpoints
[ ] No sensitive files exposed (.env, .git, backups)
[ ] Security headers checked via securityheaders.com
[ ] TLS config graded A+ on ssllabs.com
```

---

> **Note:** This document is part of the [Cybersecurity-Basics](../README.md) notes repository. It is intended for learning purposes. Always practice in a legal, authorized environment.

*Last updated: 2026*