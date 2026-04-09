# HTTP in Detail

**Module:** Pre-Security (Legacy) → How the Web Works  
**Difficulty:** Easy  
**Platform:** TryHackMe  
**Room Link:** https://tryhackme.com/room/httpindetail  
**Date Completed:** January 2026  
**Author:** Adwait Joshi

---

## What This Room Covers

HTTP (HyperText Transfer Protocol) is the foundation of data communication on the web. This room covers how HTTP works — requests, responses, methods, status codes, headers, and cookies — and introduces HTTPS as its secured counterpart. It also includes a practical element where you construct raw HTTP requests to interact with a web server directly.

If DNS in Detail explained how you find a server, this room explains how you talk to it once you get there.

---

## Key Concepts

### 1. What is HTTP?

HTTP is an application layer protocol (Layer 7) that defines how clients and servers communicate on the web. When you type a URL into a browser, HTTP is the protocol carrying your request to the server and the server's response back to you.

**HTTPS** is HTTP with TLS encryption layered on top — the same request/response structure, but the data is encrypted in transit. The padlock in your browser confirms TLS is active.

The distinction that matters in security: HTTP transmits everything in plaintext. Any device on the network path between client and server — a router, a proxy, an attacker performing a MitM — can read the full contents of an HTTP exchange. HTTPS eliminates that exposure.

**Default ports:**
- HTTP → port 80
- HTTPS → port 443

---

### 2. URLs — Uniform Resource Locators

A URL is the full address used to locate a resource on the web. Breaking one down:

```
http://user:password@tryhackme.com:80/path/to/page?query=value&page=2#section
 │         │               │        │       │              │              │
scheme  credentials      domain    port    path          query         fragment
```

| Component | Purpose |
|---|---|
| Scheme | Protocol to use — `http` or `https` |
| Credentials | Optional username/password (rare, insecure in URLs) |
| Domain | The server to connect to |
| Port | Explicit port — defaults to 80/443 if omitted |
| Path | The specific resource being requested |
| Query string | Additional parameters passed to the server (`?key=value`) |
| Fragment | Client-side anchor — points to a section within the page, never sent to server |

**Real-world relevance:** Query strings are a primary injection point. SQL injection, XSS, and parameter tampering all frequently target URL query parameters. Understanding URL structure is prerequisite to understanding how these attacks are crafted and detected.

---

### 3. HTTP Methods

HTTP methods define the *intent* of a request — what the client wants to do with the resource.

| Method | Purpose |
|---|---|
| **GET** | Retrieve a resource. Parameters passed in the URL query string. |
| **POST** | Submit data to the server. Parameters in the request body, not the URL. |
| **PUT** | Create or replace a resource at a specific location. |
| **DELETE** | Remove a resource. |
| **PATCH** | Partially update a resource. |
| **HEAD** | Same as GET but returns headers only — no body. |
| **OPTIONS** | Ask the server which methods it supports for a resource. |

**GET vs POST — the security distinction:**

GET parameters appear in the URL — they're logged in browser history, server logs, proxy logs, and are visible in the address bar. Never use GET to transmit sensitive data.

POST parameters travel in the request body — not in the URL, not in browser history. More appropriate for credentials, form submissions, and sensitive data. Still plaintext over HTTP — only HTTPS makes it confidential in transit.

**Real-world relevance:** In web application security testing, enumerating which HTTP methods a server accepts is an early reconnaissance step. An endpoint that accepts PUT or DELETE when it shouldn't is a misconfiguration. In a SOC context, unusual HTTP methods in web server logs — particularly PUT or DELETE against unexpected endpoints — are worth investigating.

---

### 4. HTTP Request Structure

A raw HTTP request has three components — request line, headers, and optional body:

```
GET /page HTTP/1.1
Host: tryhackme.com
User-Agent: Mozilla/5.0
Accept-Language: en-GB
Accept: text/html
Connection: keep-alive

[body — present for POST/PUT, absent for GET]
```

**Request line:** Method + path + HTTP version  
**Headers:** Key-value pairs providing metadata about the request  
**Body:** Data sent with the request (POST/PUT only)

**Key request headers:**

| Header | Purpose |
|---|---|
| `Host` | The domain being requested — required in HTTP/1.1 |
| `User-Agent` | Identifies the client software (browser, version, OS) |
| `Accept` | What content types the client can handle |
| `Accept-Language` | Preferred language for the response |
| `Cookie` | Session data sent back to the server |
| `Referer` | The URL of the page that initiated the request |
| `Authorization` | Credentials for authenticated requests |
| `Content-Type` | Format of the request body (for POST/PUT) |

**Real-world relevance:** User-Agent strings are logged by virtually every web server. In an investigation, unusual User-Agent values — automated tools, scanners, or empty strings — are a signal worth examining. `Referer` headers reveal navigation paths. Missing or spoofed headers are common in automated attacks.

---

### 5. HTTP Response Structure

A server's response follows a similar structure — status line, headers, body:

```
HTTP/1.1 200 OK
Server: nginx/1.15.8
Date: Fri, 10 Apr 2026 12:00:00 GMT
Content-Type: text/html
Content-Length: 2548
Set-Cookie: session=abc123; Secure; HttpOnly

<!DOCTYPE html>
<html>
...
```

**Status line:** HTTP version + status code + reason phrase  
**Headers:** Metadata about the response  
**Body:** The actual content — HTML, JSON, an image, etc.

**Key response headers:**

| Header | Purpose |
|---|---|
| `Server` | Identifies the web server software and version |
| `Content-Type` | Format of the response body |
| `Set-Cookie` | Instructs the client to store a cookie |
| `Cache-Control` | Caching directives |
| `Location` | Used with 3xx redirects — where to go next |
| `X-Frame-Options` | Controls whether page can be embedded in an iframe |
| `Strict-Transport-Security` | Forces HTTPS for future requests |
| `Content-Security-Policy` | Restricts sources of executable content |

**Real-world relevance:** The `Server` header is a passive information disclosure risk — it tells an attacker exactly what software version is running, enabling targeted exploit selection. Security-conscious configurations either remove this header or return a generic value. Headers like `Strict-Transport-Security`, `X-Frame-Options`, and `Content-Security-Policy` are defensive controls — their absence is a finding in web application security assessments.

---

### 6. HTTP Status Codes

Status codes are three-digit numbers in the response that tell the client what happened. Grouped by first digit:

| Range | Category | Meaning |
|---|---|---|
| 1xx | Informational | Request received, processing continues |
| 2xx | Success | Request was received and processed successfully |
| 3xx | Redirection | Further action needed to complete the request |
| 4xx | Client Error | The request had an issue |
| 5xx | Server Error | The server failed to fulfil a valid request |

**Codes worth knowing by memory:**

| Code | Meaning | Security Relevance |
|---|---|---|
| 200 | OK | Standard success |
| 201 | Created | Resource successfully created (POST/PUT) |
| 301 | Moved Permanently | Permanent redirect — cached by browser |
| 302 | Found | Temporary redirect |
| 400 | Bad Request | Malformed request syntax |
| 401 | Unauthorised | Authentication required |
| 403 | Forbidden | Authenticated but not authorised |
| 404 | Not Found | Resource doesn't exist |
| 405 | Method Not Allowed | HTTP method not permitted for this endpoint |
| 429 | Too Many Requests | Rate limiting in effect |
| 500 | Internal Server Error | Unhandled exception — often reveals stack traces |
| 503 | Service Unavailable | Server overloaded or down |

**Real-world relevance:** Status codes are signal-rich in logs. A burst of 404s from a single IP suggests directory enumeration. Repeated 401s indicate brute force attempts. A 500 after a crafted input suggests the application is failing to handle that input — a potential injection point. 403 vs 401 is also meaningful — 401 means not authenticated, 403 means authenticated but denied, which tells an attacker (and a defender) something different about the application's access control model.

---

### 7. Cookies

Cookies are small pieces of data the server instructs the browser to store via the `Set-Cookie` response header. The browser sends them back automatically with every subsequent request to the same domain via the `Cookie` request header.

**Primary use case:** Session management. HTTP is stateless — it has no memory between requests. Cookies give the server a way to recognise returning clients without re-authenticating on every request.

**Cookie anatomy:**

```
Set-Cookie: session=abc123; Expires=Fri, 10 Apr 2026 12:00:00 GMT; 
            Path=/; Domain=tryhackme.com; Secure; HttpOnly; SameSite=Strict
```

| Attribute | Purpose |
|---|---|
| `Expires` / `Max-Age` | When the cookie should be deleted |
| `Path` | Which URL paths the cookie applies to |
| `Domain` | Which domains receive the cookie |
| `Secure` | Cookie only sent over HTTPS |
| `HttpOnly` | Cookie inaccessible to JavaScript — mitigates XSS theft |
| `SameSite` | Controls cross-site cookie sending — mitigates CSRF |

**Real-world relevance:** Cookie security attributes are a primary web application security checklist item. A session cookie missing `HttpOnly` is vulnerable to theft via XSS — a script injected into the page can read `document.cookie` and exfiltrate it. A session cookie missing `Secure` can be transmitted over HTTP, exposing it to MitM interception. In a SOC investigation involving account takeover, examining cookie attributes and session token entropy is standard practice.

---

## Walkthrough Notes

### Task 1 — What is HTTP(S)?
Establishes the plaintext vs encrypted distinction between HTTP and HTTPS. The key point: HTTPS doesn't change the request/response structure — it wraps it in TLS. Same protocol, encrypted channel.

### Task 2 — Requests and Responses
The raw request/response format is introduced here. Worth getting comfortable with reading these directly — they appear constantly in Burp Suite, browser DevTools, and packet captures.

### Task 3 — HTTP Methods
GET vs POST is the core distinction. The room demonstrates both — GET parameters visible in the URL, POST parameters in the body. The security implication of GET parameters being logged everywhere is the key takeaway.

### Task 4 — HTTP Status Codes
The categorisation by range (2xx success, 4xx client error, 5xx server error) is more useful than memorising every code. Understanding the category tells you immediately what type of outcome occurred even for unfamiliar codes.

### Task 5 — Headers
Request and response headers. The security headers on the response side — `Strict-Transport-Security`, `X-Frame-Options`, `Content-Security-Policy` — are worth noting specifically as they appear in web application security assessments.

### Task 6 — Cookies
The practical element here — viewing and manipulating cookies in browser DevTools — is where the abstract becomes concrete. Opening DevTools → Application → Cookies and seeing session tokens, their attributes, and their values is a foundational web security skill.

### Task 7 — Making Requests (Practical)
The room provides an in-browser HTTP request tool to construct and send raw requests. The exercise involves crafting GET and POST requests with specific parameters and headers to retrieve flags from the server.

```bash
# Construct HTTP requests from the command line
curl http://target-ip/page
curl -X POST http://target-ip/login -d "username=admin&password=test"
curl -v http://target-ip/page          # verbose — shows headers
curl -I http://target-ip/page          # HEAD request — headers only
curl -H "Cookie: session=abc123" http://target-ip/dashboard
curl -b "session=abc123" http://target-ip/dashboard
```

---

## Commands Used

```bash
# Basic GET request
curl http://<target-ip>/

# POST request with data
curl -X POST http://<target-ip>/login \
     -d "username=admin&password=password123"

# Verbose output — shows full request and response headers
curl -v http://<target-ip>/page

# Send custom header
curl -H "Authorization: Bearer token123" http://<target-ip>/api

# Send cookie
curl -b "session=abc123" http://<target-ip>/dashboard

# HEAD request — response headers only
curl -I http://<target-ip>/

# Follow redirects
curl -L http://<target-ip>/redirect
```

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| HTTP vs HTTPS | Plaintext HTTP in logs = credentials/session tokens visible to anyone on path |
| GET parameters in URL | Logged by servers, proxies, browsers — sensitive data in GET = exposure risk |
| User-Agent header | Scanner/tool fingerprinting in web logs; unusual values = automated activity |
| Server header | Version disclosure — maps to CVEs; remove or genericise in hardened configs |
| 404 spike from single IP | Directory/endpoint enumeration in progress |
| 401/403 spike | Brute force or authorisation bypass attempts |
| 500 after crafted input | Application error — potential injection point |
| Missing HttpOnly on session cookie | XSS can steal session token — account takeover risk |
| Missing Secure on session cookie | Cookie transmitted over HTTP — MitM interception risk |
| Missing SameSite on session cookie | CSRF attack vector |
| Security response headers absent | Finding in web app security assessment — missing defence-in-depth |
| Cookie manipulation | Session hijacking via stolen or forged tokens |

---

## Takeaways

Three things this room makes concrete that apply immediately in security work:

1. **HTTP logs are a narrative.** Status codes, methods, User-Agent strings, and URL paths together tell the story of what a client was doing. A sequence of 404s followed by a 200 on a sensitive endpoint is a recognisable attack pattern once you know what each component means. Reading HTTP logs fluently is a core SOC skill.

2. **Cookie attributes are security controls, not optional configuration.** `HttpOnly`, `Secure`, and `SameSite` each mitigate a specific attack class. Their presence or absence on a session cookie directly determines the application's exposure to XSS session theft, MitM interception, and CSRF respectively. Auditing cookie attributes is one of the fastest ways to assess a web application's security posture.

3. **The `Server` header is an unnecessary gift to attackers.** Disclosing the exact web server software and version in every response maps directly to public CVE databases. Removing or obscuring this header costs nothing and removes a passive reconnaissance advantage. Small configuration detail — meaningful security impact.

---
