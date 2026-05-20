# Web Application Basics

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Web Hacking |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/webapplicationbasics |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the first room in the CS101 Web Hacking section. Every web application attack — SQL injection, XSS, CSRF, broken authentication — operates within the HTTP protocol. Understanding HTTP precisely is therefore the prerequisite for understanding why web vulnerabilities exist and how they are exploited. This room covers the architecture of web applications (front end, back end, web server, database, WAF), the structure of URLs, HTTP request and response messages, request methods, status codes, request and response headers, cookies, and the security headers that modern applications should implement. The room closes with a practical exercise using a built-in HTTP request tool.

---

## Key Concepts

### Web Application Architecture

A web application has two visible layers and several hidden ones. The **front end** — HTML, CSS, and JavaScript — is what the user sees and interacts with in the browser. The **back end** is the engine underneath: the web server that handles requests, the application logic, the database that stores data, and optionally a Web Application Firewall (WAF) that sits in front of the web server to filter malicious traffic.

The component responsible for hosting and serving web content to clients is the **web server**. Common web servers include Apache, Nginx, and IIS. The WAF sits between the client and the web server, inspecting incoming requests and blocking those that match known attack patterns — SQL injection strings, XSS payloads, and so on.

### URLs — Uniform Resource Locators

A URL is the address used to access a resource on the web. Its structure:

    scheme://username:password@host:port/path?query=value#fragment

| Component | Example | Purpose |
|---|---|---|
| Scheme | `https` | Protocol to use — HTTP or HTTPS |
| Username/Password | `user:pass` | Optional credentials for authentication |
| Host | `tryhackme.com` | Domain name or IP address of the server |
| Port | `443` | Network port — defaults to 80 for HTTP, 443 for HTTPS |
| Path | `/api/users/123` | Location of the resource on the server |
| Query string | `?search=nmap` | Additional parameters passed to the server |
| Fragment | `#section2` | Client-side anchor — not sent to the server |

The URL path tells the server where to find the resource being requested. In the URL `https://tryhackme.com/api/users/123`, the path `/api/users/123` identifies a specific user resource.

### HTTP Messages — Requests and Responses

HTTP communication is a request-response cycle. The client sends a request; the server returns a response.

**HTTP Request structure:**

    GET /search?query=nmap HTTP/1.1
    Host: tryhackme.com
    User-Agent: Mozilla/5.0
    Accept: text/html

The first line is the **Request Line**: HTTP method, path with query string, and HTTP version. Following lines are headers. For methods that send data (POST, PUT), a blank line separates headers from the **request body**.

**HTTP Response structure:**

    HTTP/1.1 200 OK
    Content-Type: text/html
    Server: nginx

    <html>...</html>

The first line is the **Status Line**: HTTP version, status code, and reason phrase. Headers follow. A blank line separates headers from the **response body**.

### HTTP Methods

| Method | Purpose |
|---|---|
| GET | Retrieve a resource — parameters in the URL |
| POST | Submit data to the server — data in the request body |
| PUT | Update or replace an existing resource |
| DELETE | Remove a resource |
| PATCH | Partially update a resource |
| HEAD | Like GET but returns headers only — no body |
| OPTIONS | Returns the supported HTTP methods for a resource |

GET requests should be idempotent — repeated requests should not change server state. POST is used when the submission modifies server state (form submissions, login, data creation). The format for URL-encoded form data in a POST body uses `application/x-www-form-urlencoded` as the Content-Type.

### HTTP Status Codes

Status codes communicate the outcome of a request in three digits. The first digit indicates the category:

| Range | Category | Common Examples |
|---|---|---|
| 1xx | Informational | `100 Continue` |
| 2xx | Success | `200 OK`, `201 Created`, `204 No Content` |
| 3xx | Redirection | `301 Moved Permanently`, `302 Found`, `304 Not Modified` |
| 4xx | Client Error | `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `405 Method Not Allowed` |
| 5xx | Server Error | `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable` |

`401 Unauthorized` means authentication is required and was not provided or failed. `403 Forbidden` means the server understood the request but refuses to authorise it — the user is authenticated but lacks permission.

### Request Headers

Request headers provide the server with context about the client and the request:

| Header | Example | Purpose |
|---|---|---|
| `Host` | `tryhackme.com` | Specifies the target web server — required in HTTP/1.1 |
| `User-Agent` | `Mozilla/5.0` | Identifies the browser and OS making the request |
| `Referer` | `https://www.google.com/` | Indicates the URL the request originated from |
| `Cookie` | `sessionId=abc123` | Sends stored cookies back to the server |
| `Content-Type` | `application/json` | Describes the format of data in the request body |
| `Authorization` | `Bearer eyJ...` | Carries authentication credentials |

### Response Headers

Response headers provide the client with information about the response and instructions for handling it:

| Header | Purpose |
|---|---|
| `Content-Type` | The MIME type of the response body |
| `Server` | Identifies the web server software — can reveal version information useful to attackers |
| `Set-Cookie` | Sends a cookie to the client for storage and future requests |
| `Cache-Control` | Instructs the browser how to cache the response |
| `Location` | Used with 3xx redirects to specify the new URL |

The response header that can reveal information about the web server's software and version, potentially exposing it to security risks if not removed, is `Server`.

### Cookies and Security Flags

Cookies are key-value pairs that the server sends to the browser via `Set-Cookie`, which the browser stores and sends back in subsequent requests. They are the primary mechanism for maintaining session state in HTTP's stateless model.

Two security flags on cookies are directly relevant to web application security:

**`Secure`** — the cookie is only transmitted over HTTPS connections. Without this flag, the cookie is sent in plaintext over HTTP, where it can be intercepted on the network. The flag to add to `Set-Cookie` to ensure cookies are only transmitted over HTTPS is `Secure`.

**`HttpOnly`** — the cookie cannot be accessed by JavaScript (`document.cookie` returns nothing). This prevents cross-site scripting (XSS) attacks from stealing session cookies. The flag to add to prevent cookie access via JavaScript is `HttpOnly`.

### Security Headers

Beyond cookies, several HTTP response headers harden web application security:

| Header | Purpose |
|---|---|
| `Content-Security-Policy (CSP)` | Defines approved sources for scripts, styles, images, and other resources — mitigates XSS |
| `Strict-Transport-Security (HSTS)` | Forces browsers to use HTTPS for the domain; prevents protocol downgrade attacks |
| `X-Content-Type-Options` | Prevents browsers from interpreting files as a different MIME type than declared — mitigates MIME sniffing |
| `X-Frame-Options` | Prevents the page from being embedded in iframes — mitigates clickjacking |

In a CSP configuration, the property used to define where scripts can be loaded from is `script-src`. The directive to apply HSTS to all subdomains is `includeSubDomains`. The `X-Content-Type-Options` directive used to mitigate MIME type sniffing is `nosniff`.

---

## Walkthrough Notes

The room runs through nine tasks, each building on the previous. The practical component in the final task uses an in-page HTTP request tool to send requests and observe responses.

**Task 1 (Introduction):** Frames the room as the foundational layer for all subsequent Web Hacking rooms. No answer required.

**Task 2 (Web Application Overview):** Covers front end and back end components. Questions: which component is responsible for hosting and serving web content to clients — `Web Server`; which tool is used to intercept and filter malicious traffic — `Web Application Firewall`.

**Task 3 (Uniform Resource Locators):** Covers URL structure. Questions: which part of the URL is used by the browser to determine how to access the resource — `Scheme`; what is the path component in the URL `https://tryhackme.com/room/webapplicationbasics` — `/room/webapplicationbasics`.

**Task 4 (HTTP Messages):** Covers request and response structure. Questions: what is the first line of an HTTP request called — `Request Line`; what is the first line of an HTTP response called — `Status Line`.

**Task 5 (HTTP Request: Request Line and Methods):** Covers HTTP methods. The question asking for the HTTP method used in form submissions where data is sent in the body — `POST`; the Content-Type for URL-encoded form data — `application/x-www-form-urlencoded`.

**Task 6 (HTTP Request: Headers and Body):** Covers request headers. Questions: which request header specifies the domain name of the server — `Host`; which header identifies the browser making the request — `User-Agent`.

**Task 7 (HTTP Response: Status Line and Status Codes):** Covers status codes. Questions: which status code indicates a successful request — `200`; which indicates the resource does not exist — `404`; which indicates the server understood the request but refuses to authorise it — `403`.

**Task 8 (HTTP Response: Headers and Body):** Covers response headers, cookies, and security headers. Questions: which response header reveals server software version — `Server`; which cookie flag ensures transmission over HTTPS only — `Secure`; which flag prevents JavaScript access — `HttpOnly`; the CSP property for script sources — `script-src`; the HSTS directive for subdomains — `includeSubDomains`; the X-Content-Type-Options directive — `nosniff`.

**Task 9 (Practical Task):** The in-page HTTP tool sends requests to the lab server and inspects responses. Questions are answered by sending specified requests and reading the response headers and body for flag values.

---

## Tools Referenced

    Browser Developer Tools    # Inspect HTTP requests and responses in the Network tab
    curl                       # Command-line HTTP client for manual request crafting

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| `Server` response header | Reconnaissance mitigation — removing or obscuring the `Server` header prevents attackers from fingerprinting the web server version and targeting known CVEs for that version |
| `HttpOnly` and `Secure` cookie flags | Session hijacking prevention — missing `HttpOnly` enables XSS-based cookie theft; missing `Secure` enables cookie interception on unencrypted connections; both are standard findings in web application security assessments |
| Content-Security-Policy | XSS mitigation — a properly configured CSP that restricts `script-src` to known domains prevents injected scripts from loading external attacker-controlled code |
| HSTS with `includeSubDomains` | SSL stripping prevention — HSTS forces HTTPS and prevents protocol downgrade attacks; `includeSubDomains` ensures subdomain takeover cannot be used to set insecure cookies for the parent domain |
| `X-Content-Type-Options: nosniff` | MIME confusion attack prevention — without this header, browsers may execute files as scripts even if the server declared them as a different type; a common vector for bypassing upload restrictions |
| Status code 403 vs 401 | Access control investigation — distinguishing 403 (authorised but forbidden) from 401 (authentication required) during recon identifies whether an endpoint requires authentication or applies additional access controls beyond authentication |

---

## Takeaways

1. **HTTP headers are where both attack and defence live in web application security.** The `Server` header leaks version information. Missing `HttpOnly` enables cookie theft. A weak or absent CSP enables XSS payload loading from external sources. Missing HSTS enables protocol downgrade. Every security control in this room operates through a header — understanding headers precisely is the prerequisite for understanding what is and is not protected in any web application.

2. **The distinction between `Secure` and `HttpOnly` on cookies addresses two completely different threat vectors.** `Secure` protects cookies from network interception on unencrypted connections. `HttpOnly` protects cookies from JavaScript-based theft within the browser. A session cookie missing `Secure` is vulnerable on any HTTP page; missing `HttpOnly` is vulnerable to any XSS payload. Both flags are required; either one missing is a finding in a web application security review.

3. **URL structure is not just a routing detail — it is an attack surface.** Query parameters are user-controlled input passed directly to the server. Path components can be manipulated to access unintended resources (path traversal). The fragment is client-side only and never reaches the server — but everything else is. Understanding precisely what part of a URL the server sees and processes, versus what the browser handles locally, is the foundation for understanding parameter manipulation and path-based vulnerabilities.

---
