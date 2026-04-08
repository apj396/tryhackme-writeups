# DNS in Detail

**Module:** Pre-Security (Legacy) → How the Web Works  
**Difficulty:** Easy  
**Platform:** TryHackMe  
**Room Link:** https://tryhackme.com/room/dnsindetail  
**Date Completed:** January 2026  
**Author:** Adwait Joshi

---

## What This Room Covers

DNS (Domain Name System) is the protocol that translates human-readable domain names into IP addresses machines can route to. This room covers the full DNS resolution process — record types, the hierarchy of DNS servers involved in a lookup, and the step-by-step journey a query takes from your browser to an answer.

DNS is infrastructure. It runs quietly underneath almost every internet interaction — which is exactly why attackers target it and why defenders monitor it.

---

## Key Concepts

### 1. What DNS Does

Every device on the internet is reachable by an IP address. Humans don't use IP addresses — they use domain names. DNS is the translation layer between the two.

Without DNS, accessing a website would require knowing its IP address. With DNS, `tryhackme.com` resolves to an IP address automatically, transparently, every time.

The analogy that holds: DNS is the internet's phone book. You look up a name, you get a number.

---

### 2. Domain Hierarchy

Domains are structured in a hierarchy, read right to left:

```
subdomain.domain.tld
www.tryhackme.com

. (root)
└── com (TLD)
    └── tryhackme (Second-Level Domain)
        └── www (Subdomain)
```

**TLD (Top-Level Domain)** — the rightmost part of a domain. Two types:
- **gTLD** (generic) — `.com`, `.org`, `.net`, `.edu` — indicate purpose
- **ccTLD** (country code) — `.uk`, `.in`, `.de` — indicate geography

**Second-Level Domain** — the registered, human-chosen name. Limited to 63 characters, letters, numbers, and hyphens only.

**Subdomain** — sits to the left of the second-level domain. Can be chained (`one.two.tryhackme.com`) up to a 253-character total domain length limit.

**Real-world relevance:** Attackers register lookalike domains by manipulating these components — `paypa1.com` (character substitution), `tryhackme.co` (different TLD), `secure.tryhackme-login.com` (legitimate subdomain, malicious second-level domain). Understanding domain hierarchy is the foundation of domain spoofing detection.

---

### 3. DNS Record Types

DNS isn't just for IP address lookups. Different record types serve different purposes:

| Record Type | Purpose | Example |
|---|---|---|
| **A** | Maps domain to IPv4 address | `tryhackme.com → 104.26.10.229` |
| **AAAA** | Maps domain to IPv6 address | `tryhackme.com → 2606:4700::...` |
| **CNAME** | Maps domain to another domain name | `www.tryhackme.com → tryhackme.com` |
| **MX** | Specifies mail servers for the domain | `tryhackme.com → alt1.aspmx.l.google.com` |
| **TXT** | Free-text records — used for verification and policy | SPF, DKIM, DMARC records |
| **NS** | Specifies authoritative nameservers for the domain | `tryhackme.com → ns1.cloudflare.com` |

**TXT records deserve specific attention from a security perspective:**
- **SPF (Sender Policy Framework)** — lists which mail servers are authorised to send email for the domain. Helps prevent email spoofing.
- **DKIM** — provides a cryptographic signature for outbound email, allowing receivers to verify the message wasn't tampered with.
- **DMARC** — policy record that tells receiving mail servers what to do when SPF or DKIM checks fail (quarantine, reject, or report).

**Real-world relevance:** Missing or misconfigured SPF, DKIM, and DMARC records are a primary enabler of phishing and business email compromise (BEC). In a SOC investigation involving a phishing campaign, checking these TXT records is one of the first steps in determining whether a sender domain is legitimately configured.

---

### 4. The DNS Resolution Process

When you type a domain into your browser, a multi-step lookup process happens — typically in milliseconds:

```
Browser → Local Cache → Recursive DNS Resolver → Root Nameserver
                                                         ↓
                                               TLD Nameserver (.com)
                                                         ↓
                                            Authoritative Nameserver
                                                         ↓
                                                   IP Address returned
                                                         ↓
                                               Cached + Browser connects
```

**Step by step:**

**1. Local Cache Check**
The browser first checks its own DNS cache. If it has a recent, valid record for the domain, the lookup stops here. No external query needed.

**2. Recursive DNS Resolver**
If not cached, the query goes to a recursive resolver — typically provided by your ISP or a public DNS service (Google `8.8.8.8`, Cloudflare `1.1.1.1`). The resolver does the legwork of finding the answer on your behalf.

**3. Root Nameserver**
The resolver queries a root nameserver — the top of the DNS hierarchy. The root doesn't know the IP for `tryhackme.com`, but it knows which TLD nameserver handles `.com`. There are 13 sets of root nameservers globally, operated by different organisations.

**4. TLD Nameserver**
The `.com` TLD nameserver doesn't know the IP either, but it knows which authoritative nameserver is responsible for `tryhackme.com`. It returns that information.

**5. Authoritative Nameserver**
This is the final authority — the nameserver that holds the actual DNS records for `tryhackme.com`. It returns the A record (IP address) to the recursive resolver.

**6. Cache and Connect**
The resolver caches the result for the duration of the record's **TTL (Time to Live)** and returns the IP address to the browser. The browser connects to the IP. The cached record means subsequent lookups for the same domain skip straight to the cached answer.

**Real-world relevance:** Every step in this process is a potential attack surface. DNS cache poisoning injects false records into a resolver's cache. DNS hijacking redirects queries by compromising nameservers or resolver configurations. DNS tunnelling encodes data inside DNS queries to exfiltrate information through firewalls that allow DNS traffic. Monitoring DNS query logs is a core SOC capability — unusual query volumes, lookups for known-malicious domains, or queries using non-standard resolvers are all actionable signals.

---

### 5. TTL — Time to Live (DNS Context)

Every DNS record has a TTL value — the number of seconds a resolver or browser should cache the record before querying again.

- Short TTL (e.g., 300 seconds) — changes propagate quickly, but generates more DNS traffic
- Long TTL (e.g., 86400 seconds / 24 hours) — efficient caching, but changes take longer to propagate globally

**Real-world relevance:** Attackers using **fast flux DNS** rapidly rotate the IP addresses behind a domain — with very short TTLs — to evade blocklists. The domain name stays constant, the IP changes constantly. Detecting fast flux requires monitoring TTL values and the rate of IP changes for a given domain.

---

## Walkthrough Notes

### Task 1 — What is DNS?
Establishes the core premise: domain names are for humans, IP addresses are for machines, DNS bridges the two. The phone book analogy the room uses is accurate and sufficient.

### Task 2 — Domain Hierarchy
The breakdown of TLD, second-level domain, and subdomain is the structural foundation. Worth internalising precisely because phishing domain analysis depends on reading domain hierarchy correctly — identifying which part of a suspicious domain is the legitimate-looking component and which part is the attacker's.

### Task 3 — Record Types
The record type table is the practical reference output of this task. A, AAAA, CNAME, MX, TXT, NS — each has a specific purpose, and mixing them up in an investigation creates errors. The TXT record security applications (SPF, DKIM, DMARC) are the most operationally relevant.

### Task 4 — Making a Request
The step-by-step resolution process walkthrough. The interactive element maps each step visually — which server responds at which stage. The key detail: the authoritative nameserver is the source of truth, and everything upstream is a caching intermediary.

### Task 5 — Practical
The room provides a DNS lookup tool to query specific record types for given domains. Commands worth knowing:

```bash
# Query A record
nslookup tryhackme.com

# Query specific record type
nslookup -type=MX tryhackme.com
nslookup -type=TXT tryhackme.com
nslookup -type=CNAME www.tryhackme.com

# Using dig (Linux) — more detailed output
dig tryhackme.com
dig MX tryhackme.com
dig TXT tryhackme.com +short

# Specify a DNS resolver explicitly
dig @8.8.8.8 tryhackme.com
```

---

## Commands Used

```bash
# Basic DNS lookup
nslookup <domain>

# Query specific record type (nslookup)
nslookup -type=A <domain>
nslookup -type=AAAA <domain>
nslookup -type=MX <domain>
nslookup -type=TXT <domain>
nslookup -type=NS <domain>
nslookup -type=CNAME <domain>

# dig — detailed DNS query tool
dig <domain>
dig <domain> +short          # clean output, IP only
dig MX <domain>
dig TXT <domain>
dig @1.1.1.1 <domain>        # query against Cloudflare resolver

# Check DNS cache on Windows
ipconfig /displaydns

# Flush DNS cache on Windows
ipconfig /flushdns

# Flush DNS cache on Linux (systemd)
sudo systemd-resolve --flush-caches
```

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| DNS resolution process | Understanding the full chain helps identify where DNS-based attacks intercept queries |
| DNS cache poisoning | False records injected into resolver cache; users directed to attacker-controlled IPs |
| DNS hijacking | Compromised nameserver or resolver redirects all queries for a domain |
| DNS tunnelling | Data exfiltration via DNS queries; bypasses firewalls that permit DNS traffic |
| Fast flux DNS | Rapid IP rotation behind a domain to evade blocklists; detected via TTL monitoring |
| TXT records (SPF/DKIM/DMARC) | Missing or misconfigured = phishing enabler; first check in email-based incident triage |
| MX records | Targeted in email infrastructure attacks; unexpected MX changes = red flag |
| Non-standard resolvers | DNS queries to unexpected resolvers (not corporate/ISP) may indicate compromise or tunnelling |
| DNS query logs | Core SOC data source — volume anomalies, known-bad domains, unusual query patterns |

---

## Takeaways

Three things DNS in Detail makes clear that compound in value over time:

1. **DNS is trusted infrastructure, which makes it a high-value target.** Almost all internet traffic depends on DNS working correctly. Attacks that compromise DNS — cache poisoning, hijacking, tunnelling — operate at a layer that most users and many security tools implicitly trust. Monitoring DNS is non-negotiable in a mature SOC.

2. **Reading a domain correctly is a practical security skill.** The hierarchy — TLD, second-level domain, subdomain — determines what's legitimate and what's spoofed. `login.microsoft.com` and `microsoft.com.login.attacker.com` look similar at a glance. Parsed correctly, the second one's second-level domain is `attacker` — nothing to do with Microsoft. That parsing happens in seconds once the hierarchy is internalised.

3. **TTL values tell a story.** Unusually short TTLs, combined with rapidly changing IP addresses, are a technical fingerprint of fast flux infrastructure — a technique used by botnets and malware C2 operators to stay ahead of blocklists. TTL isn't just a caching parameter; it's a behavioural signal.

---
