# Putting It All Together

**Module:** Pre-Security (Legacy) → How the Web Works  
**Difficulty:** Easy  
**Platform:** TryHackMe  
**Room Link:** https://tryhackme.com/room/puttingitalltogether  
**Date Completed:** January 2026  
**Author:** Adwait Joshi

---

## What This Room Covers

This room synthesises everything covered in the How the Web Works module — DNS, HTTP, HTML, and JavaScript — into a single end-to-end picture of what happens from the moment you type a URL into a browser to the moment the page appears on screen. It also introduces supporting infrastructure components: load balancers, CDNs, databases, and WAFs.

No new protocols. No new attack surface. Just the full picture assembled in one place — which turns out to be more valuable than any individual piece.

---

## Key Concepts

### 1. The Complete Request Journey

When you type `https://tryhackme.com` into a browser and press Enter, a specific sequence of events unfolds:

```
Step 1: Browser checks local DNS cache
        ↓ (not cached)
Step 2: Recursive DNS resolver queried
        ↓
Step 3: Root nameserver → TLD nameserver → Authoritative nameserver
        ↓
Step 4: IP address returned, cached with TTL
        ↓
Step 5: Browser initiates TCP connection to server IP on port 443
        ↓
Step 6: TLS handshake — encrypted channel established
        ↓
Step 7: HTTP GET request sent over encrypted connection
        ↓
Step 8: Request hits load balancer → routed to available web server
        ↓
Step 9: Web server processes request → queries database if needed
        ↓
Step 10: Server returns HTTP response (HTML, CSS, JS)
         ↓
Step 11: Browser parses HTML → builds DOM
         ↓
Step 12: Browser requests additional assets (CSS, JS, images)
         ↓
Step 13: JavaScript executes → DOM modified
         ↓
Step 14: Page rendered and visible to user
```

Each step is a potential failure point — and a potential attack surface. Mapping the full journey makes both failure diagnosis and threat modelling systematic rather than guesswork.

---

### 2. Load Balancers

A load balancer sits in front of multiple web servers and distributes incoming requests across them. Two primary purposes: performance and availability.

**How it works:**

```
Client Request
      ↓
Load Balancer
   ↙    ↓    ↘
Server1 Server2 Server3
```

**Load balancing algorithms:**
- **Round robin** — requests distributed sequentially across servers, cycling back to the first
- **Weighted** — servers with more capacity receive proportionally more traffic
- **Least connections** — new requests go to the server with fewest active connections

**Health checks:** Load balancers periodically probe each server to verify it's responding. A server that fails health checks is temporarily removed from the pool until it recovers.

**Sticky sessions:** Some load balancers tie a user's session to a specific server — ensuring subsequent requests from the same user hit the same backend. Important for applications that store session state locally rather than in a shared database.

**Real-world relevance:** Load balancers are both infrastructure and security components. They can terminate TLS (handling encryption/decryption before traffic reaches backend servers), apply rate limiting, and block requests that match known attack patterns. In a SOC context, load balancer logs aggregate traffic from all backend servers into a single view — useful for correlating attack activity across a server farm.

---

### 3. CDNs — Content Delivery Networks

A CDN is a geographically distributed network of servers that cache and serve static content (images, CSS, JavaScript, videos) from locations physically close to the requesting user.

**How it works:**

```
User in Mumbai requests image from tryhackme.com
        ↓
CDN routes request to nearest edge server (Mumbai PoP)
        ↓
Edge server has cached copy → serves directly
        ↓ (if not cached)
Edge server fetches from origin server → caches → serves
```

**Benefits:** Reduced latency, reduced load on origin server, geographic redundancy.

**Real-world relevance:** CDNs complicate attribution in security investigations. The IP address you see for a CDN-hosted domain belongs to the CDN, not the origin server — the real server IP is masked. CDNs also absorb volumetric DDoS attacks by distributing the traffic across their global infrastructure. Bypassing CDN protection to find the origin server IP is a common reconnaissance technique — often done by checking historical DNS records, SSL certificate transparency logs, or email headers that may reveal the direct server IP.

---

### 4. Databases

Web applications almost universally store persistent data — user accounts, content, transactions — in databases. The web server queries the database to retrieve or store data as part of processing requests.

**Common database types:**

| Type | Examples | Structure |
|---|---|---|
| Relational (SQL) | MySQL, PostgreSQL, MSSQL | Tables, rows, columns — structured, schema-defined |
| Non-relational (NoSQL) | MongoDB, Redis, Cassandra | Flexible document/key-value structure |

**How the web server interacts:**

```
Browser → HTTP Request → Web Server → SQL Query → Database
                                            ↓
Browser ← HTTP Response ← Web Server ← Query Result
```

**Real-world relevance:** SQL Injection targets this exact interaction — if user input is incorporated into a SQL query without sanitisation, an attacker can manipulate the query to extract, modify, or delete data. Understanding that the web server acts as an intermediary between the browser and database explains why injection attacks work: the database trusts queries from the web server, and if those queries contain attacker-controlled input, the database executes them faithfully.

---

### 5. WAFs — Web Application Firewalls

A WAF inspects HTTP requests and responses, filtering out traffic that matches known attack patterns before it reaches the web server.

```
Client Request
      ↓
    WAF  ← blocks: SQLi, XSS, path traversal, scanner signatures
      ↓ (clean traffic only)
 Web Server
```

**What WAFs detect and block:**
- SQL Injection patterns (`' OR 1=1 --`)
- XSS payloads (`<script>alert(1)</script>`)
- Path traversal (`../../etc/passwd`)
- Known scanner signatures (Nikto, sqlmap User-Agent strings)
- Rate limiting violations

**WAF limitations:**
- Rule-based WAFs can be bypassed with encoding, obfuscation, or novel payloads
- They add latency
- They generate false positives — blocking legitimate traffic
- They're a control, not a fix — they don't resolve the underlying vulnerability

**Real-world relevance:** WAFs are a defence-in-depth control, not a silver bullet. In a penetration test, WAF bypass is a standard technique — encoding payloads in URL encoding, Unicode, or case variation to evade signature matching. In a SOC context, WAF logs are a high-signal source — blocked requests reveal active attack attempts and attacker tooling. A spike in WAF blocks followed by a successful server-side error (500) suggests a payload got through.

---

### 6. How It All Fits Together — The Full Stack

```
┌─────────────────────────────────────────────┐
│                   CLIENT                    │
│  Browser → DNS → TCP → TLS → HTTP Request  │
└─────────────────────┬───────────────────────┘
                      │
              ┌───────▼────────┐
              │   CDN / WAF    │  ← Static content served here
              └───────┬────────┘    Attack patterns blocked here
                      │
              ┌───────▼────────┐
              │ Load Balancer  │  ← Distributes requests
              └───┬────────────┘    Health checks backends
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
  Web Server  Web Server  Web Server  ← Processes requests
       │          │          │
       └──────────┼──────────┘
                  │
          ┌───────▼────────┐
          │    Database    │  ← Persistent data storage
          └────────────────┘
```

Every component in this stack has a security role, a failure mode, and a logging surface. Knowing where each sits — and what it's responsible for — is what allows you to answer "where did this attack come from and what did it touch?" in an incident.

---

## Walkthrough Notes

### Task 1 — Putting It All Together
The room's primary exercise is a drag-and-drop ordering task — placing the steps of a web request in the correct sequence. It sounds trivial. It's actually a useful self-check: if any step feels uncertain, that's a signal to revisit the corresponding room.

The sequence that matters: DNS resolution → TCP connection → TLS handshake → HTTP request → load balancer → web server → database query → HTTP response → browser rendering.

### Task 2 — Other Components
Load balancers, CDNs, databases, and WAFs are introduced here as the supporting cast — the infrastructure that sits around the web server and shapes how requests are handled before and after the server processes them.

### Task 3 — How Web Servers Work
Web servers (Apache, Nginx, IIS) receive HTTP requests and determine how to respond — serving static files directly or passing dynamic requests to application code (PHP, Python, Node.js). Virtual hosting allows a single web server to serve multiple domains by reading the `Host` header from the request.

**Real-world relevance:** Virtual host misconfiguration is a known vulnerability — a server that doesn't validate the `Host` header correctly may serve content intended for one domain when requested with a different `Host` value. Host header injection is a real attack class.

---

## Commands Used

```bash
# Trace the full journey — DNS + connectivity
nslookup tryhackme.com
ping tryhackme.com
traceroute tryhackme.com       # Linux
tracert tryhackme.com          # Windows

# Full HTTP request with verbose output — see every step
curl -v https://tryhackme.com

# Check what IP a CDN-hosted domain resolves to
dig tryhackme.com +short

# View response headers — identify WAF, server, CDN signatures
curl -I https://tryhackme.com

# Test virtual host routing
curl -H "Host: admin.internal" http://<target-ip>/
```

---

## Real-World Mapping

| Component | Security Role | SOC Relevance |
|---|---|---|
| DNS | Resolution layer — first step of every request | DNS logs reveal C2 beaconing, tunnelling, malicious domain lookups |
| TLS | Encrypts HTTP traffic in transit | Certificate anomalies, expired certs, weak cipher suites |
| Load Balancer | Distributes traffic, can terminate TLS, rate limit | Aggregated access logs across server farm |
| CDN | Serves static content, absorbs DDoS | Masks origin IP; bypass = direct server exposure |
| WAF | Filters known attack patterns | High-signal log source; block spikes = active attack |
| Web Server | Processes requests, serves responses | Access and error logs — primary investigation source |
| Database | Stores persistent data | Target of SQLi; breach scope = data classification of DB contents |
| Virtual Hosting | Multiple domains on one server | Host header injection if `Host` not validated |

---

## Takeaways

Three things this room delivers that the individual rooms couldn't:

1. **The full picture changes how you think about attacks.** Knowing DNS, HTTP, and HTML individually is useful. Knowing how they chain together — DNS resolution feeds the TCP connection, the TLS handshake secures the HTTP exchange, the HTTP request reaches a WAF before a load balancer before a web server before a database — tells you what an attacker has to navigate and where defenders have visibility. Threat modelling requires the full stack view, not just individual components.

2. **Every component logs.** DNS resolvers log queries. Load balancers log requests. WAFs log blocked and allowed traffic. Web servers log every HTTP request. Databases log queries. Each log source answers different questions in an investigation. Knowing the full stack means knowing which log to look at first — and what it will and won't tell you.

3. **Defence in depth is the full stack working together.** A WAF without a hardened web server is one layer. A hardened web server with a misconfigured load balancer exposes the backend directly. CDN protection bypassed by a known origin IP removes the DDoS shield. The components are individually useful; their security value compounds when they're correctly layered — and degrades when any layer is misconfigured.

---
