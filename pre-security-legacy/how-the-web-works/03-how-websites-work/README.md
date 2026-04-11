# How Websites Work

**Module:** Pre-Security (Legacy) → How the Web Works  
**Difficulty:** Easy  
**Platform:** TryHackMe  
**Room Link:** https://tryhackme.com/room/howwebsiteswork  
**Date Completed:** January 2026  
**Author:** Adwait Joshi

---

## What This Room Covers

This room covers what actually happens when a website loads in your browser — HTML structure, JavaScript behaviour, and how the browser renders a page from raw code. It also introduces two foundational web vulnerabilities: Sensitive Data Exposure and HTML Injection. The room bridges the gap between "how HTTP works" and "how web applications can be exploited."

Relevant context: the Unified Mentor internship involves web development assignments, so some of this is familiar territory approached from a security angle rather than a development one.

---

## Key Concepts

### 1. HTML — Structure of a Web Page

HTML (HyperText Markup Language) is the skeleton of every web page. It defines the structure and content — what elements exist, what they contain, and how they're organised.

**Basic HTML structure:**

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Page Title</title>
    </head>
    <body>
        <h1>Main Heading</h1>
        <p>A paragraph of text.</p>
        <img src="image.jpg" alt="Description">
        <a href="https://example.com">A link</a>
    </body>
</html>
```

**Key elements:**

| Element | Purpose |
|---|---|
| `<!DOCTYPE html>` | Declares the document as HTML5 |
| `<head>` | Metadata — title, linked stylesheets, scripts |
| `<body>` | Visible page content |
| `<h1>` to `<h6>` | Headings, descending importance |
| `<p>` | Paragraph |
| `<a href="">` | Hyperlink |
| `<img src="">` | Image |
| `<form>` | User input container |
| `<input>` | Form field |
| `<div>` | Generic container for layout |

**Real-world relevance:** Understanding HTML structure is prerequisite to understanding injection attacks. HTML Injection and XSS both exploit the browser's willingness to interpret attacker-controlled input as HTML or JavaScript rather than treating it as plain text. If you can't read HTML, you can't understand what the injected payload is doing.

---

### 2. JavaScript — Behaviour of a Web Page

JavaScript (JS) is the programming language of the web. Where HTML defines structure, JavaScript defines behaviour — dynamic content, user interaction, real-time updates without page reloads.

JavaScript runs in the browser — client-side. This has a critical security implication: the user has full access to any JavaScript running on a page. It can be viewed, modified, and in many cases, bypassed entirely.

**Basic JavaScript examples:**

```javascript
// Alert box
alert("Hello");

// Change page content dynamically
document.getElementById("demo").innerHTML = "New content";

// Respond to a button click
document.getElementById("btn").addEventListener("click", function() {
    console.log("Button clicked");
});

// Make a request to an API
fetch("/api/data")
    .then(response => response.json())
    .then(data => console.log(data));
```

**Real-world relevance:** Client-side JavaScript validation is not security. Any input validation done exclusively in JavaScript can be bypassed by disabling JS, intercepting the request in Burp Suite, or simply editing the DOM. Security controls must be enforced server-side. This is one of the most consistently exploited misconceptions in web development.

---

### 3. How the Browser Renders a Page

When a browser receives an HTML document, it follows a specific process to render it:

```
1. Parse HTML → Build DOM (Document Object Model)
2. Parse CSS → Build CSSOM (CSS Object Model)
3. Execute JavaScript → Modifies DOM/CSSOM
4. Combine DOM + CSSOM → Render Tree
5. Layout → Calculate element positions
6. Paint → Draw pixels to screen
```

**The DOM** is the browser's in-memory representation of the page — a tree structure of every HTML element. JavaScript interacts with the page by reading and modifying the DOM.

**DevTools — the security researcher's lens:**

Browser DevTools (F12) expose everything:

| DevTools Tab | What It Shows |
|---|---|
| Elements | Live DOM — HTML and CSS, editable in real time |
| Console | JavaScript execution environment |
| Network | Every HTTP request and response the page makes |
| Sources | All JavaScript files loaded by the page |
| Application | Cookies, local storage, session storage |

**Real-world relevance:** DevTools is the first tool used in manual web application testing. Viewing page source shows the static HTML sent by the server. DevTools shows the live DOM after JavaScript has modified it — which can differ significantly. Sensitive data left in HTML comments, hardcoded credentials in JavaScript files, and exposed API endpoints in network requests are all commonly discovered through DevTools inspection.

---

### 4. Sensitive Data Exposure

Sensitive Data Exposure occurs when a web application unintentionally reveals information that should not be accessible — credentials, API keys, internal paths, developer comments, or configuration details.

**Common locations:**

```html
<!-- Developer comment left in production -->
<!-- TODO: remove test credentials admin:password123 -->

<!-- Hardcoded credentials in HTML -->
<input type="hidden" name="admin_key" value="s3cr3t_k3y">

<!-- Internal path disclosed in error message -->
Error: file not found at /var/www/html/admin/config.php
```

**In JavaScript files:**

```javascript
// API key hardcoded in client-side JS — visible to anyone
const API_KEY = "AIzaSyD-9tSrke72...";
const ADMIN_ENDPOINT = "/internal/admin/panel";
```

**Real-world relevance:** Sensitive data exposure through HTML comments and JavaScript files is a remarkably consistent finding in real web application assessments. Developers leave debug comments, staging credentials, and internal endpoint references in code that gets shipped to production. These are trivially discoverable — no exploitation required, just reading the source. In a bug bounty context, this category yields consistent findings. In a SOC context, discovering that an internal admin panel URL was exposed in client-side code changes the scope of an incident significantly.

---

### 5. HTML Injection

HTML Injection occurs when user-supplied input is rendered directly in a page without sanitisation, allowing an attacker to inject arbitrary HTML elements.

**Example — vulnerable code:**

```html
<!-- Server renders user input directly into the page -->
<p>Hello, [USER_INPUT]!</p>

<!-- If user inputs: <h1>Injected Heading</h1> -->
<p>Hello, <h1>Injected Heading</h1>!</p>
```

The browser renders the injected HTML as legitimate page content. The attacker controls what appears on the page for any user who loads it.

**The escalation path — HTML Injection → XSS:**

HTML Injection becomes Cross-Site Scripting (XSS) when the injected content includes JavaScript:

```html
<!-- Injected input: -->
<script>alert('XSS')</script>

<!-- Rendered in page as: -->
<p>Hello, <script>alert('XSS')</script>!</p>
```

The browser executes the script. From here, the attacker can steal cookies, redirect users, log keystrokes, or perform actions on behalf of the victim.

**The root cause in both cases:** Trusting user input. Input that reaches a page without being sanitised or encoded is a vulnerability waiting to be exploited.

**Real-world relevance:** XSS is consistently in the OWASP Top 10. Stored XSS — where the injected payload is saved to a database and served to every subsequent visitor — is particularly impactful. A stored XSS on a banking application could silently exfiltrate session tokens from every authenticated user who loads the affected page. The fix is output encoding — treating all user input as plain text, never as HTML, when rendering to the page.

---

## Walkthrough Notes

### Task 1 — How Websites Work
Establishes the client-server model in web context: browser sends HTTP request, server returns HTML, browser renders it. The distinction between what the server sends and what the browser renders (after JavaScript execution) is the key detail.

### Task 2 — HTML
The basic page structure exercise. Viewing page source (`Ctrl+U` in most browsers) versus DevTools Elements tab is the practical skill here — source shows what the server sent, Elements shows what the DOM currently looks like after JS execution.

### Task 3 — JavaScript
The `document.getElementById` and `innerHTML` interaction demonstrates how JavaScript modifies the DOM dynamically. The security implication — client-side code is fully visible and modifiable by the user — is the takeaway that carries forward into every subsequent web security topic.

### Task 4 — Sensitive Data Exposure
The exercise involves viewing page source to find credentials or sensitive information left in HTML comments. The action: `Ctrl+U` to view source → `Ctrl+F` to search for keywords like `password`, `key`, `secret`, `TODO`, `admin`. A simple technique with consistently high yield in real assessments.

### Task 5 — HTML Injection
The practical input field demonstrates injection — submitting HTML tags as input and observing them render in the page. The mental model to take forward: any place user input appears in a page is a potential injection point until proven otherwise.

---

## Commands Used

```bash
# View page source from terminal
curl http://<target-ip>/

# View with headers
curl -v http://<target-ip>/

# Search source for sensitive keywords
curl -s http://<target-ip>/ | grep -i "password\|secret\|key\|admin\|todo"

# Download JS file for offline analysis
curl -s http://<target-ip>/script.js -o script.js
grep -i "key\|secret\|password\|endpoint" script.js
```

**Browser shortcuts:**
```
Ctrl+U          → View page source
F12             → Open DevTools
Ctrl+Shift+I    → Open DevTools (alternative)
Ctrl+F          → Search within source/DevTools
```

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| Client-side JS validation | Bypassable — never a security control; server-side validation is mandatory |
| HTML comments in production | Common source of credential and path disclosure; check in recon phase |
| Hardcoded credentials in JS | Exposed to any user who views source; immediate critical finding |
| HTML Injection | Entry point for UI redressing and XSS escalation |
| XSS (stored) | Session token theft at scale; affects every user loading the compromised page |
| XSS (reflected) | Requires social engineering to deliver; common in phishing chains |
| DevTools Network tab | Reveals all API calls a page makes — often exposes undocumented endpoints |
| DOM manipulation | Understanding JS-modified DOM is essential for testing single-page applications |
| Output encoding missing | Root cause of HTML injection and XSS; fix is encoding user input on render |

---

## Takeaways

Three things this room establishes that compound as web security knowledge builds:

1. **Client-side is not secure-side.** Everything in the browser — HTML, JavaScript, cookies, local storage — is accessible to the user. Any security logic that exists only client-side can be bypassed. This single principle explains the root cause of an enormous proportion of web vulnerabilities.

2. **Viewing source is reconnaissance.** Before any tool is run, manually reading a page's HTML source and JavaScript files often reveals more than automated scanners catch — developer comments, hardcoded values, internal endpoints, error messages. It takes two keystrokes and requires no special access. In an assessment or investigation, it's always the first step.

3. **Injection follows trust.** HTML Injection, SQL Injection, Command Injection — all stem from the same root: the application trusts user-supplied input and processes it as code or markup rather than data. The specific injection type depends on where the untrusted input lands. Understanding this unified root cause means recognising injection vulnerabilities across contexts, not just the specific variants covered in any single room.

---
