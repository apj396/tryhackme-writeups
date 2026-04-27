# Offensive Security Intro

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Start Your Cyber Security Journey |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/offensivesecurityintro |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the first room in the CS101 path and the first in the Start Your Cyber Security Journey section. It answers a question that every newcomer to the field asks: what does offensive security actually look like in practice? Rather than describing it in the abstract, the room puts you directly into a simulated scenario — a fake banking web application with a hidden administrative page — and walks through how an attacker would find and exploit it using a directory brute-forcing tool called GoBuster. The lesson is deliberately simple and deliberately concrete. The point is not the exploit. The point is the mindset: finding what is not meant to be found, understanding why it matters, and recognising that the same skills that make an attacker dangerous are what make a defender effective.

---

## Key Concepts

### What Offensive Security Is

Offensive security is the practice of thinking and acting like an attacker — proactively seeking vulnerabilities in systems, applications, and networks before malicious actors do. It operates under authorisation: every legitimate offensive security engagement requires explicit written permission from the system owner. Without that permission, the same actions constitute criminal activity in most jurisdictions.

The primary roles within offensive security:

| Role | Description |
|---|---|
| Penetration Tester | Hired to legally test systems for vulnerabilities and report findings for remediation |
| Red Teamer | Simulates a full adversarial campaign — not just technical exploitation but also social engineering, physical access, and evasion — to test an organisation's overall detection and response capability |
| Security Engineer | Designs and maintains secure systems, informed by an understanding of how attackers think and operate |

The common thread across all three is the ability to identify weakness before it is weaponised. That ability begins with knowing how attackers find targets and what they look for — which is exactly what this room demonstrates.

### Directory Brute-Forcing with GoBuster

Web applications expose content through URLs. Most of what a site contains is visible through its navigation — the pages it links to. But not all pages are linked. Admin portals, configuration panels, legacy endpoints, and development interfaces may exist on a server without appearing anywhere in the public-facing interface. They are hidden by obscurity, not by access control. If an attacker knows the URL, they can access the page — so finding the URL is the attack.

GoBuster automates this discovery. It takes a wordlist — a file containing hundreds or thousands of common directory and page names — and attempts to access each one by appending it to the target URL. If the server responds with a `200 OK` status code, the page exists. If it responds with `404 Not Found`, it does not.

The command used in the room:

    gobuster -u http://fakebank.thm -w wordlist.txt dir

Breaking down the flags:

| Flag | Purpose |
|---|---|
| `-u` | Specifies the target URL |
| `-w` | Specifies the wordlist file to iterate through |
| `dir` | Sets the mode to directory and file enumeration |

GoBuster returns status codes alongside each discovered path:

| Status Code | Meaning |
|---|---|
| `200` | Page exists and is accessible |
| `301` | Redirect — often means a directory exists |
| `403` | Forbidden — exists but access is denied |
| `404` | Not found |

In the FakeBank scan, GoBuster discovers two paths: `/images` returning `301` and `/bank-transfer` returning `200`. The `/bank-transfer` path is the target — a bank administration portal that allows money transfers between accounts with no authentication check.

### The Security Implication

The FakeBank scenario demonstrates a class of vulnerability called security through obscurity — the assumption that keeping a resource's location secret is sufficient protection. It is not. A hidden URL with no access control is effectively a public URL that most users have not thought to look for yet. Any automated scanner, any attacker with a wordlist, and any curious user who notices the URL pattern can find it. The correct fix is not to hide the page better — it is to require authentication before displaying it.

This is the core lesson of offensive security at the introductory level: security assumptions that rely on attackers not knowing something are fragile. Proper controls — authentication, authorisation, input validation — do not depend on the attacker being ignorant.

---

## Walkthrough Notes

The room runs through three tasks. Task 2 is the primary hands-on component.

**Task 1 (Introduction):** Defines offensive security, contrasts it with defensive security, and frames the career paths that offensive security skills enable — penetration tester, red teamer, security engineer. Notes that all offensive work covered in TryHackMe rooms is conducted in legal, controlled environments. No answer required.

**Task 2 (Hacking Your First Machine):** The FakeBank VM is deployed in split-screen view. A terminal is available on the machine. The GoBuster command is run against `http://fakebank.thm` using the `wordlist.txt` file available on the Desktop:

    gobuster -u http://fakebank.thm -w wordlist.txt dir

GoBuster output shows two discovered paths:

    /images        (Status: 301)
    /bank-transfer (Status: 200)

Navigating to `http://fakebank.thm/bank-transfer` in the browser reveals the admin money transfer portal. The task instructs transferring `$2000` from account `2276` to account `8881`. After submitting the transfer, returning to the account dashboard shows the updated balance and the room flag: `BANK-HACKED`.

**Task 3 (Careers in Cyber Security):** A brief career-mapping task that connects the skills demonstrated — reconnaissance, tool usage, exploitation — to real-world roles. Notes that offensive security professionals are among the highest-paid in the industry. No answer required.

---

## Commands Used

    gobuster -u http://fakebank.thm -w wordlist.txt dir

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| Directory brute-forcing (GoBuster) | Web application assessment — scanning for unlinked or forgotten endpoints is a mandatory step in any web penetration test; hidden admin portals without authentication controls are a critical finding |
| Status code 200 vs 301 | Recon triage — distinguishing directly accessible pages (200) from redirects (301) determines which discovered paths warrant immediate investigation |
| Security through obscurity | Architecture review — any control that relies solely on an endpoint being unknown is a design flaw; defenders use automated scanning internally to find such pages before attackers do |
| Hidden admin portal, no auth | Access control assessment — unauthenticated administrative interfaces are a critical-severity finding in any web application security review; they represent direct business impact (financial, data integrity) |
| Wordlist-based enumeration | Threat modelling — understanding that attackers use wordlists to enumerate endpoints informs decisions about URL structure, access control placement, and WAF rules |

---

## Takeaways

1. **Offensive security is not about breaking things — it is about finding what is already broken before someone else does.** The FakeBank admin portal was always accessible. GoBuster did not create the vulnerability; it revealed it. This distinction matters for how defenders think about their own systems: the question is not whether an attacker could find a weakness, but whether the defender found it first.

2. **Tools amplify intent — they do not replace it.** GoBuster is effective because it automates what would otherwise be manual, impractical URL guessing. But knowing which tool to reach for, what flags to use, and how to interpret the output requires understanding what the tool is doing and why. Using security tools as black boxes produces unreliable results. Understanding them produces findings.

3. **Every offensive technique has a direct defensive countermeasure.** Directory brute-forcing is countered by proper authentication on every sensitive endpoint — not by hiding the URL. Rate limiting and WAF rules reduce the efficiency of automated scanners. Web application firewalls can detect and block directory enumeration patterns. The offensive and defensive skill sets are mirrors of each other — which is why understanding one develops the other.

---
