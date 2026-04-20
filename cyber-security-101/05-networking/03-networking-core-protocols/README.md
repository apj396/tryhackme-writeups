# Networking Core Protocols

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Networking |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/networkingcoreprotocols |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the third room in the CS101 Networking series. The previous two rooms covered how networks are structured and how devices find and communicate with each other. This room moves up the stack to the application layer — the protocols that users and services actually interact with. DNS resolves names to addresses. HTTP moves web content. FTP moves files. SMTP, POP3, and IMAP handle email at different stages of its journey. WHOIS provides domain registration lookups. Each protocol is examined at the command level using telnet and purpose-built clients, making the underlying exchanges visible rather than hidden behind a GUI.

---

## Key Concepts

### DNS — Domain Name System

DNS is the internet's naming infrastructure. Every time a domain name is typed into a browser, a DNS query runs in the background to resolve it to an IP address. Without DNS, every connection would require knowing the destination's IP directly — functional in theory, unusable in practice at scale.

DNS operates at the Application layer and uses UDP port 53 for standard queries. It uses several record types, the most operationally significant being:

| Record Type | Purpose |
|---|---|
| A | Maps a domain name to an IPv4 address |
| AAAA | Maps a domain name to an IPv6 address |
| MX | Specifies the mail server responsible for a domain |
| CNAME | Canonical name — aliases one domain to another |
| TXT | Arbitrary text — used for SPF, DKIM, and domain verification |

A packet capture of a DNS query shows a two-packet exchange: a Standard query from the client to the resolver, and a Standard query response returning the resolved address. Both A and AAAA records may be requested in separate queries for the same domain.

    tshark -r dns-query.pcapng -Nn

DNS is a high-value target in both offensive and defensive contexts. Attackers use DNS tunnelling to exfiltrate data through what appears to be normal DNS traffic. DNS cache poisoning injects false records to redirect users to attacker-controlled infrastructure. Monitoring DNS query volume, unusual record types, and abnormally long domain names are all standard detection approaches in a SOC environment.

### WHOIS

WHOIS is a query protocol used to look up registration information for domain names. Given a domain, a WHOIS lookup returns details such as the registrar, registration and expiry dates, name servers, and in some cases registrant contact information — though privacy protection services increasingly mask this.

    whois example.com

For defenders, WHOIS is useful for quickly characterising a suspicious domain — registration date (newly registered domains are higher risk), registrar, and whether registrant details are hidden behind a proxy service. For threat intelligence work, patterns in registrar choice, registration timing, and name server infrastructure can link apparently unrelated domains to the same actor.

### HTTP — HyperText Transfer Protocol

HTTP is the protocol that moves web content between servers and browsers. It operates at the Application layer over TCP port 80. Every browser interaction — loading a page, submitting a form, calling an API — is fundamentally an HTTP request and response exchange.

The core request methods:

| Method | Purpose |
|---|---|
| GET | Retrieve a resource |
| POST | Submit data to the server |
| PUT | Update a resource |
| DELETE | Remove a resource |

HTTP is stateless — each request is independent. The server responds with a status code indicating the outcome: `200 OK` for success, `301`/`302` for redirects, `404` for not found, `500` for server errors. HTTP sends everything in plaintext, including credentials and cookies. This is why HTTPS — HTTP over TLS — is the expected standard for any site handling sensitive data, and is covered in the following room.

### FTP — File Transfer Protocol

FTP is a dedicated file transfer protocol operating over TCP. It uses two separate connections: a control connection on port 21 for commands and responses, and a data connection negotiated dynamically in passive mode for the actual file transfer.

Anonymous FTP — where the username is `anonymous` and the password is typically blank — is a legacy configuration that allows unauthenticated file access. It was common for public software distribution and remains a misconfiguration worth checking for during assessments.

    ftp MACHINE_IP

Once connected, standard FTP commands include `ls` to list files, `type ascii` to switch transfer mode for text files, `get filename` to download, and `quit` to close the session. Like HTTP, FTP transmits in plaintext — credentials and file contents are both visible in a packet capture. The secure replacements are SFTP and FTPS, covered in the following room.

### SMTP — Simple Mail Transfer Protocol

SMTP handles the sending and relaying of email. It operates over TCP port 25. The protocol exchange is conversational — the client identifies itself, specifies sender and recipient, and transmits the message body.

The basic SMTP exchange sequence:

    HELO / EHLO    — client identifies itself to the server
    MAIL FROM:     — specifies the sender address
    RCPT TO:       — specifies the recipient address
    DATA           — begins the message body
    .              — signals end of message body (dot on a line by itself)
    QUIT           — closes the connection

SMTP has no native authentication or encryption in its base form, which is why email spoofing — sending mail that appears to come from a legitimate address — is trivially easy without additional controls like SPF, DKIM, and DMARC records in DNS.

### POP3 and IMAP — Email Retrieval Protocols

Once email has been delivered to a mail server via SMTP, the recipient retrieves it using either POP3 or IMAP.

**POP3 (Post Office Protocol 3)** operates on TCP port 110. It downloads messages to the client and typically deletes them from the server. Simple and bandwidth-efficient, but the mail exists on only one device after retrieval.

**IMAP (Internet Message Access Protocol)** operates on TCP port 143. It synchronises mail between server and client — messages remain on the server and are accessible from multiple devices simultaneously. This is the model that modern webmail and mobile clients use.

Both protocols transmit credentials in plaintext in their unencrypted forms. Using telnet to connect directly to port 110 or 143 demonstrates this — the login exchange is fully readable in the session output, exactly as it would appear in a packet capture on the same network segment.

    telnet MACHINE_IP 110    # POP3
    telnet MACHINE_IP 143    # IMAP

---

## Walkthrough Notes

The room runs through several tasks, each covering one protocol with inline comprehension questions and hands-on components using an AttackBox connected to the lab machine. The task names below are based on available research — verify against your dashboard if any title appears off.

**Task 1 (Introduction):** Sets out the learning objectives and lists the protocols covered. Notes that this is the third room in the four-part Networking series and recommends completing Networking Essentials first. No questions — read and proceed.

**Task 2 (DNS and WHOIS):** The tshark command reads a pre-captured DNS pcap file showing both the A record query and the AAAA record query for `www.example.com` in sequence. The resolved IPv4 address and IPv6 address are visible in the response packets. The WHOIS component uses the `whois` command on the AttackBox — the key question asks for the creation date of `x.com`, readable directly from the WHOIS output in `YYYY-MM-DD` format.

**Task 3 (HTTP):** Conceptual task covering request methods, status codes, and the distinction between HTTP and HTTPS. The detail that HTTPS is HTTP secured with TLS is introduced here but examined in depth in the next room. No live practical in this task.

**Task 4 (FTP):** Anonymous login to the lab machine using the `ftp` client. After connecting, `ls` reveals the directory listing including a `flag.txt` file alongside `coffee.txt` and `tea.txt`. Switching to ASCII mode before downloading text files avoids binary transfer warnings. The flag is retrieved using `get flag.txt`.

    ftp MACHINE_IP
    # username: anonymous
    # password: (blank — press Enter)
    ls
    type ascii
    get flag.txt
    quit

**Task 5 (SMTP):** Using telnet on port 25, the full SMTP exchange is carried out manually — `HELO`, `MAIL FROM`, `RCPT TO`, `DATA`, message body, `.` on a line by itself to end, `QUIT`. The server queues the message and confirms with a `250 OK` response. This makes visible what every email client does silently on every send.

    telnet MACHINE_IP 25

**Task 6 (POP3 and IMAP):** Telnet connections to ports 110 and 143 respectively. The POP3 exchange uses `USER`, `PASS`, `STAT`, `LIST`, `RETR 1`, and `QUIT` to authenticate and retrieve the first message. The IMAP connection banner on port 143 confirms the server software and version — in the room this is Dovecot running on Ubuntu — which is the answer to the banner-grabbing question and a practical demonstration of why IMAP exposes server identity on connection.

    telnet MACHINE_IP 110
    telnet MACHINE_IP 143

**Task 7 (Conclusion):** Summarises the protocols covered and directs to Networking Secure Protocols as the next room. No answer required.

---

## Commands Used

    tshark -r dns-query.pcapng -Nn
    whois <domain>
    ftp MACHINE_IP
    telnet MACHINE_IP 25
    telnet MACHINE_IP 110
    telnet MACHINE_IP 143

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| DNS A and AAAA queries | DNS-based threat detection — high query volume to a single domain, unusually long subdomains, or queries for newly registered domains are IOC categories in SIEM rules |
| DNS TXT records | Email authentication verification — SPF and DKIM records live in DNS TXT entries; absent or misconfigured records are a phishing enabler |
| WHOIS registration data | Threat intelligence — newly registered domains (under 30 days old) correlate strongly with phishing infrastructure; registration date is a standard field in domain reputation scoring |
| FTP anonymous access | Misconfiguration detection — anonymous FTP on an internet-facing server indicates unintended public file exposure and is a finding in any network assessment |
| SMTP plaintext exchange | Phishing triage — understanding the raw SMTP command sequence is necessary for reading email headers and tracing spoofed sender paths during incident response |
| POP3/IMAP plaintext credentials | Credential interception risk — unencrypted mail retrieval (port 110 or 143) on internal networks is a lateral movement enabler; detecting these ports in traffic versus their secure equivalents (995/993) flags unencrypted mail clients |

---

## Takeaways

1. **Every protocol in this room was designed for function, not security — and that design decision is now a permanent constraint.** DNS, FTP, SMTP, POP3, and IMAP were all built in an era when network trust was assumed. The result is that credentials, mail content, and file transfers move in plaintext by default. The security layer was added later — DNSSEC, FTPS, SMTPS, IMAPS — and adoption remains incomplete. Knowing what the insecure baseline looks like is the prerequisite for recognising when the secure version is absent.

2. **Using telnet to manually speak these protocols is not an exercise in obsolete tools — it is a method for internalising what every client application conceals.** When an email client sends a message, it is executing the SMTP exchange carried out by hand in this room. When a mail server delivers to a POP3 inbox, it is the command sequence demonstrated on port 110. Understanding the exchange at this level is what makes phishing header analysis, protocol anomaly detection, and misconfiguration review interpretable rather than opaque.

3. **DNS is simultaneously the most relied-upon protocol and one of the most abused.** It underpins every other application-layer interaction — a page load, an API call, an email send — and it is also the channel through which data exfiltration, C2 communication, and domain hijacking operate. Treating DNS traffic as background noise is a detection gap. Treating it as signal — with the right tooling and baseline — surfaces threats that bypass every other control.

---
