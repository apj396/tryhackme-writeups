# Search Skills

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Start Your Cyber Security Journey |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/searchskills |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the third and final room in the Start Your Cyber Security Journey section. The first two rooms established the two sides of the field — offensive and defensive. This room steps back and addresses something that underpins both: the ability to find reliable, accurate, and relevant information efficiently. In cybersecurity, research is not a supplementary skill — it is an operational one. Every investigation, every assessment, every malware triage session involves looking something up. This room covers how to evaluate sources critically, how to use advanced search operators, how to use specialised security search engines and databases, how to navigate technical documentation, and how social media factors into both intelligence gathering and operational security. The practical questions throughout the room are answered by actually using these tools against real data — not by recalling theory.

---

## Key Concepts

### Evaluating Search Results

The volume of information available on any security topic is not the problem. The quality of that information is. A Google search for "learn hacking" returns over a billion results. The skill is not finding results — it is evaluating them.

Key evaluation criteria for any source:

| Criterion | Questions to Ask |
|---|---|
| Source authority | Who wrote this? Are they a recognised expert or institution? |
| Currency | When was this published or last updated? Is it still accurate? |
| Accuracy | Is it consistent with other reputable sources? Are claims supported by evidence? |
| Purpose | Is this informational, commercial, or promotional? Does the author have an incentive to mislead? |

A cryptographic method or product considered bogus or fraudulent — one that claims security properties it cannot deliver — is called snake oil. The term is used in security circles to flag tools or techniques that sound credible but have no legitimate basis.

### Advanced Search Operators (Google Dorking)

Standard search queries return broad results. Advanced operators narrow them to what is actually needed. This technique is commonly called Google dorking — using search engine operators to extract specific information that would otherwise be buried in general results.

Practically useful operators:

| Operator | Effect | Example |
|---|---|---|
| `"phrase"` | Exact phrase match | `"security operations center"` |
| `site:` | Restrict results to a specific domain | `site:gov cybersecurity framework` |
| `filetype:` | Restrict to a specific file type | `filetype:pdf cyber warfare report` |
| `-term` | Exclude a term from results | `nmap tutorial -youtube` |
| `intitle:` | Match term in page title only | `intitle:admin login` |

The room's question on restricting a Google search to PDF files containing the terms "cyber warfare report" is answered with: `filetype:pdf cyber warfare report`. The `filetype:` operator is directly relevant to security research — finding published threat reports, government advisories, and technical whitepapers in PDF format rather than wading through blog posts and forum threads.

### Specialised Security Search Engines and Databases

Standard search engines index web pages. Specialised tools index different things entirely — and for security work, these specialised sources are often more useful than Google.

**Shodan** is a search engine for internet-connected devices. Where Google indexes web page content, Shodan indexes banners returned by servers, routers, cameras, industrial control systems, and any other networked device that responds to probes. Searching for a specific server software on Shodan returns a global map and count of exposed instances, organised by country. A Shodan search for `lighttpd` reveals the United States as the country with the most lighttpd servers.

**Censys** is similar to Shodan but more technically oriented — it focuses on internet-wide scanning and certificate transparency data, making it particularly useful for tracking infrastructure and TLS certificate relationships.

**VirusTotal** accepts file hashes, URLs, and IP addresses and runs them against dozens of antivirus engines simultaneously, returning a verdict from each engine alongside community notes and behavioral analysis. Given the hash `2de70ca737c1f4602517c555ddd54165432cf231ffc0e21fb2e23b9dd14e7fb4`, a VirusTotal lookup shows that BitDefenderFalx detects the file as `Android.Riskware.Agent.LHH`.

**Have I Been Pwned** checks whether an email address or password has appeared in a known data breach. It is useful both for personal operational security and for investigating whether credentials associated with an incident were previously exposed.

### CVE and Vulnerability Databases

The CVE (Common Vulnerabilities and Exposures) program acts as a standardised dictionary for publicly known vulnerabilities. Each confirmed vulnerability receives a unique identifier in the format `CVE-YEAR-IDNUMBER`. This standardisation means that a vendor, a researcher, a defender, and a scanner tool can all refer to the same flaw unambiguously using one identifier.

Two primary resources for CVE research:

**NVD (National Vulnerability Database)** — maintained by NIST, it hosts the authoritative record for every CVE including CVSS severity scores, affected software versions, and remediation guidance. CVE-2024-3094 refers to a backdoor discovered in the xz compression utility — a supply chain compromise embedded in upstream source tarballs that could allow SSH authentication bypass on affected Linux distributions.

**Exploit-DB** — maintains a database of verified, tested exploit code indexed by software name, version, and CVE. Where NVD describes the vulnerability, Exploit-DB provides the working exploit. In penetration testing and authorised assessments, Exploit-DB is used to find proof-of-concept code after a vulnerable version has been identified. Exploit-DB is maintained by Offensive Security (offsec).

### Technical Documentation

Official documentation is the most accurate and most current source of information about how a tool or system works. Community blog posts, tutorials, and forum answers are useful for guidance, but they can be outdated, incomplete, or simply wrong. When accuracy matters — and in security, it usually does — the primary source is the documentation.

Key documentation resources by platform:

- **Linux/Unix:** the `man` command provides the system reference manual for any installed command. `man cat` explains what `cat` does: concatenate and display file content.
- **Windows:** Microsoft Docs is the authoritative reference for Windows commands and APIs. The `netstat` parameter that displays the executable associated with each active connection and listening port is `-b`. The `ss` command (Socket Statistics) is the Linux equivalent — faster than `netstat` and the modern standard on Linux systems.
- **Security tools:** Snort, Apache, PHP, Nmap, and most major security tools maintain official documentation that should be the first reference, not a last resort.

### Social Media as an Intelligence Source

Social media platforms are intelligence sources in both directions — they can be used to gather information about a target, and they expose information about the user that can be used against them.

**LinkedIn** is the primary platform for gathering professional information about technical staff — job titles, skills, certifications, employers, and career history. For OSINT purposes, LinkedIn is where you learn about a company's technical background and identify employees who might be targets for social engineering.

**Facebook** tends to contain personal details — hometown, school attended, family relationships, interests — that are frequently used in social engineering and as answers to security questions. Where LinkedIn answers "who works there and what do they know," Facebook can answer "what are the password reset question answers."

Twitter/X and specialised security communities are valuable for following current threat intelligence, newly disclosed CVEs, and tool releases in near real time.

---

## Walkthrough Notes

The room runs through eight tasks, each focused on a different category of search skill. All questions in the room require actually using the tools described — not recalling memorised answers.

**Task 1 (Introduction):** Frames the problem — information overload and the need to filter, evaluate, and apply information efficiently. Demonstrates the scale of unfiltered search results. No answer required.

**Task 2 (Evaluation of Search Results):** Covers source evaluation criteria. The key question asks for the term describing a cryptographic method or product considered bogus or fraudulent — the answer is snake oil.

**Task 3 (Search Operators):** Covers Google dorking operators. The question asks how to limit a Google search to PDF files containing "cyber warfare report" — the answer is `filetype:pdf cyber warfare report`.

**Task 4 (Specialised Search Engines):** Covers Shodan, Censys, VirusTotal, and Have I Been Pwned. The Shodan question asks which country has the most lighttpd servers — the answer, found by searching `lighttpd` on Shodan and reading the country breakdown on the left panel, is the United States. The VirusTotal question asks what BitDefenderFalx detects the file with hash `2de70ca737c1f4602517c555ddd54165432cf231ffc0e21fb2e23b9dd14e7fb4` as — the answer is `Android.Riskware.Agent.LHH`, found by pasting the hash into VirusTotal's search bar and reading the BitDefenderFalx row in the detection results.

**Task 5 (Vulnerability Databases):** Covers CVE identifiers, NVD, and Exploit-DB. The question asks what utility CVE-2024-3094 refers to — the answer is `xz`, the compression utility in which a supply chain backdoor was discovered. Verified via NVD or cve.org.

**Task 6 (Technical Documentation):** Covers man pages and official documentation. The Linux `cat` command documentation confirms its purpose: concatenate and display file content. The Windows `netstat` question asks which parameter displays the executable associated with each active connection — the answer is `-b`.

**Task 7 (Social Media):** Covers LinkedIn and Facebook as OSINT sources. The question asking which platform is useful for learning about a company's technical staff is answered by LinkedIn. The question about which platform might help find answers to security questions is answered by Facebook.

**Task 8 (Conclusion):** Summarises the room — source evaluation, search operators, specialised tools, vulnerability databases, documentation, and social media — and frames these as the research foundation that all other security work builds on. No answer required.

---

## Tools Referenced

    filetype:pdf cyber warfare report          # Google dorking — PDF file type filter
    site:example.com search term               # Google dorking — domain restriction

External tools used:

    Shodan        — https://www.shodan.io
    Censys        — https://search.censys.io
    VirusTotal    — https://www.virustotal.com
    Have I Been Pwned — https://haveibeenpwned.com
    NVD           — https://nvd.nist.gov
    Exploit-DB    — https://www.exploit-db.com

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| Source evaluation | Threat intelligence quality control — not all IOC feeds, threat reports, or vulnerability disclosures are equally reliable; analyst judgment about source credibility directly affects decision quality |
| Google dorking (`filetype:`, `site:`) | Reconnaissance — the same operators used to find research PDFs are used by attackers to find exposed configuration files, login portals, and sensitive documents indexed by search engines |
| Shodan | Asset exposure monitoring — defenders use Shodan to check whether their own infrastructure is inadvertently exposed; an unexpected Shodan result for a company's IP range is an immediate finding |
| VirusTotal hash lookup | Malware triage — submitting a file hash to VirusTotal is a standard first step when investigating a suspicious file; a clean result does not guarantee safety, but a detected result across multiple engines confirms malicious classification |
| CVE lookup (NVD, cve.org) | Patch prioritisation — CVSS scores on NVD allow defenders to rank unpatched vulnerabilities by severity and exploitability; CVEs with public exploit code on Exploit-DB are higher priority than those without |
| LinkedIn OSINT | Social engineering awareness — the same information that makes LinkedIn useful for legitimate research makes it a targeting resource for attackers conducting spear-phishing campaigns against specific employees |

---

## Takeaways

1. **Research is an operational skill, not a background activity.** In the middle of an investigation, when an unknown IP appears in a log or an unfamiliar hash shows up in an alert, the ability to quickly find accurate, relevant information determines how fast and how accurately the analyst can respond. The tools in this room — Shodan, VirusTotal, NVD — are not reference resources to consult occasionally. They are part of the active investigation workflow.

2. **The same techniques used to gather intelligence on attackers are used by attackers to gather intelligence on targets.** Google dorking finds sensitive documents exposed by accident. Shodan finds devices that should not be internet-facing. LinkedIn identifies people who can be socially engineered. Defenders who understand these tools understand what attackers see when they look at an organisation from the outside — and that perspective is what drives effective hardening.

3. **CVE-2024-3094 is a useful reminder that supply chain attacks bypass most traditional defences.** The xz backdoor was embedded in source code, distributed through legitimate channels, and would have been present in the dependencies of systems that never ran a malicious executable directly. No perimeter control stops a trusted package manager delivering compromised code. Staying current with CVE publications, monitoring dependency trees, and following security advisories are the detection mechanisms that matter here — all of which depend on the search and research skills this room covers.

---
