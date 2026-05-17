# TryHackMe Writeups — Adwait Joshi

Hands-on cybersecurity lab documentation. Analyst-style writeups —
reasoning first, steps second. Two paths completed, documented progressively.

🔗 [GitHub Portfolio](https://github.com/apj396)

---

## Progress

- **Rooms Completed:** 70+ (Pre-Security Legacy + Cyber Security 101)
- **Rooms Documented:** 41
- **Currently Documenting:** Cyber Security 101
- **Focus:** SOC fundamentals, network analysis, threat detection

---

## Paths Completed

| Path | Rooms | Status | Completed |
|------|-------|--------|-----------|
| Pre-Security (Legacy) | 15 | ✅ Complete | January 2026 |
| Cyber Security 101 | 54 | ✅ Complete | February 2026 |

---

## Writeups

### Pre-Security (Legacy) — Completed January 2026

#### Network Fundamentals

| # | Room | Key Topics |
|---|------|-----------|
| 01 | [What is Networking?](./pre-security-legacy/network-fundamentals/01-what-is-networking/) | IP, MAC, ICMP, Ping |
| 02 | [Intro to LAN](./pre-security-legacy/network-fundamentals/02-intro-to-lan/) | Topologies, Subnetting, ARP, DHCP |
| 03 | [OSI Model](./pre-security-legacy/network-fundamentals/03-osi-model/) | 7 Layers, Encapsulation, Layer Attribution |
| 04 | [Packets & Frames](./pre-security-legacy/network-fundamentals/04-packets-and-frames/) | TCP, UDP, Three-Way Handshake, Ports, TTL |
| 05 | [Extending Your Network](./pre-security-legacy/network-fundamentals/05-extending-your-network/) | Port Forwarding, Firewalls, VPNs, Routing |

> ✅ Network Fundamentals — 5/5 rooms documented.

#### How the Web Works

| # | Room | Key Topics |
|---|------|-----------|
| 01 | [DNS in Detail](./pre-security-legacy/how-the-web-works/01-dns-in-detail/) | DNS Resolution, Record Types, TTL, DNS Attacks |
| 02 | [HTTP in Detail](./pre-security-legacy/how-the-web-works/02-http-in-detail/) | HTTP/S, Methods, Status Codes, Headers, Cookies |
| 03 | [How Websites Work](./pre-security-legacy/how-the-web-works/03-how-websites-work/) | HTML, JavaScript, DOM, Sensitive Data Exposure, HTML Injection |
| 04 | [Putting It All Together](./pre-security-legacy/how-the-web-works/04-putting-it-all-together/) | Full Request Journey, Load Balancers, CDN, WAF, Databases |

> ✅ How the Web Works — 4/4 rooms documented.

#### Linux Fundamentals

| # | Room | Key Topics |
|---|------|-----------|
| 01 | [Linux Fundamentals Part 1](./pre-security-legacy/linux-fundamentals/01-linux-fundamentals-part-1/) | Terminal, Navigation, File Operations, grep, find, Pipes |
| 02 | [Linux Fundamentals Part 2](./pre-security-legacy/linux-fundamentals/02-linux-fundamentals-part-2/) | SSH, File Permissions, SUID, Man Pages, Utilities |
| 03 | [Linux Fundamentals Part 3](./pre-security-legacy/linux-fundamentals/03-linux-fundamentals-part-3/) | Cron Jobs, Process Management, Logs, Package Management, Services |

> ✅ Linux Fundamentals — 3/3 rooms documented.

#### Windows Fundamentals

| # | Room | Key Topics |
|---|------|-----------|
| 01 | [Windows Fundamentals Part 1](./pre-security-legacy/windows-fundamentals/01-windows-fundamentals-part-1/) | NTFS, User Accounts, UAC, Task Scheduler, System Tools |
| 02 | [Windows Fundamentals Part 2](./pre-security-legacy/windows-fundamentals/02-windows-fundamentals-part-2/) | Registry, Resource Monitor, PowerShell, Windows Update, Defender |

> ✅ Windows Fundamentals — 2/2 rooms documented.

---
> 🎉 Pre-Security (Legacy) — fully documented. 14/15 rooms complete.
> Note: "Learning Cyber Security" intro room skipped — single-task overview room with no substantive lab content.
---

### Cyber Security 101

#### Start Your Cyber Security Journey

| # | Room | Key Topics |
|---|---|---|
| 01 | [Offensive Security Intro](./cyber-security-101/01-start-your-cyber-security-journey/01-offensive-security-intro/) | Offensive security mindset, GoBuster, directory brute-forcing, security through obscurity |
| 02 | [Defensive Security Intro](./cyber-security-101/01-start-your-cyber-security-journey/02-defensive-security-intro/) | SOC, threat intelligence, DFIR, malware analysis, SIEM triage simulation |
| 03 | [Search Skills](./cyber-security-101/01-start-your-cyber-security-journey/03-search-skills/) | Source evaluation, Google dorking, Shodan, VirusTotal, CVE, NVD, Exploit-DB, OSINT |

> ✅ Start Your Cyber Security Journey — 3/3 rooms documented.

#### Linux Fundamentals

| # | Room | Key Topics |
|---|---|---|
| 01 | [Linux Fundamentals Part 1](./cyber-security-101/02-linux-fundamentals/01-linux-fundamentals-part-1/) | echo, whoami, ls, cd, cat, pwd, find, grep, shell operators |
| 02 | [Linux Fundamentals Part 2](./cyber-security-101/02-linux-fundamentals/02-linux-fundamentals-part-2/) | SSH, flags and switches, man pages, touch, mkdir, cp, mv, rm, file, permissions, su, root directories |
| 03 | [Linux Fundamentals Part 3](./cyber-security-101/02-linux-fundamentals/03-linux-fundamentals-part-3/) | Nano, Vim, wget, scp, Python HTTPServer, processes, systemctl, crontabs, apt, log files |

> ✅ Linux Fundamentals — 3/3 rooms documented.

#### Windows and AD Fundamentals

| # | Room | Key Topics |
|---|---|---|
| 01 | [Windows Fundamentals 1](./cyber-security-101/03-windows-and-ad-fundamentals/01-windows-fundamentals-1/) | Windows desktop, NTFS, ADS, System32, user accounts, UAC, Control Panel, Task Manager |
| 02 | [Windows Fundamentals 2](./cyber-security-101/03-windows-and-ad-fundamentals/02-windows-fundamentals-2/) | MSConfig, UAC settings, Computer Management, System Information, Resource Monitor, CMD, Registry |
| 03 | [Windows Fundamentals 3](./cyber-security-101/03-windows-and-ad-fundamentals/03-windows-fundamentals-3/) | Windows Update, Defender AV, Firewall profiles, SmartScreen, TPM, BitLocker, VSS, LotL |
| 04 | [Active Directory Basics](./cyber-security-101/03-windows-and-ad-fundamentals/04-active-directory-basics/) | Domains, AD DS, OUs, GPOs, SYSVOL, Kerberos, NetNTLM, trees, forests, trust relationships |

> ✅ Windows and AD Fundamentals — 4/4 rooms documented.

#### Command Line

| # | Room | Key Topics |
|---|---|---|
| 01 | [Windows Command Line](./cyber-security-101/04-command-line/01-windows-command-line/) | ver, systeminfo, ipconfig, netstat -abon, ping, tracert, nslookup, dir, type, tasklist, taskkill |
| 02 | [Windows PowerShell](./cyber-security-101/04-command-line/02-windows-powershell/) | Cmdlets, object pipeline, Get-ChildItem, Where-Object, Sort-Object, Select-String, Get-Process, Get-Service, scripting, execution policy, modules |
| 03 | [Linux Shells](./cyber-security-101/04-command-line/03-linux-shells/) | Shell types, Bash, Fish, Zsh, /etc/shells, history, chsh, scripting, shebang, variables, loops, conditionals |

> ✅ Command Line — 3/3 rooms documented.

#### Networking

| # | Room | Key Topics |
|---|---|-----------|
| 01 | [Networking Concepts](./cyber-security-101/05-networking/01-networking-concepts/) | OSI model, TCP/IP model, IP addressing, subnets, TCP vs UDP, encapsulation, Telnet |
| 02 | [Networking Essentials](./cyber-security-101/05-networking/02-networking-essentials/) | DHCP, ARP, ICMP, routing, NAT |
| 03 | [Networking Core Protocols](./cyber-security-101/05-networking/03-networking-core-protocols/) | DNS, WHOIS, HTTP, FTP, SMTP, POP3, IMAP |
| 04 | [Networking Secure Protocols](./cyber-security-101/05-networking/04-networking-secure-protocols/) | TLS, HTTPS, SMTPS, POP3S, IMAPS, SSH, SFTP, FTPS, VPN |
| 05 | [Wireshark: The Basics](./cyber-security-101/05-networking/05-wireshark-the-basics/) | Tool overview, packet dissection, packet navigation, display filters, Export Objects |
| 06 | [Tcpdump: The Basics](./cyber-security-101/05-networking/06-tcpdump-the-basics/) | Basic capture, BPF filters, reading pcap files, advanced filtering, output control |
| 07 | [Nmap: The Basics](./cyber-security-101/05-networking/07-nmap-the-basics/) | Host discovery, port scanning, service detection, OS fingerprinting, timing, output |

> ✅ Networking — 7/7 rooms documented.

#### Cryptography

| # | Room | Key Topics |
|---|---|---|
| 01 | [Cryptography Basics](./cyber-security-101/06-cryptography/01-cryptography-basics/) | Plaintext, ciphertext, cipher, key, Caesar Cipher, symmetric encryption, DES, AES, XOR, modulo |
| 02 | [Public Key Cryptography Basics](./cyber-security-101/06-cryptography/02-public-key-cryptography-basics/) | RSA, Diffie-Hellman, SSH key auth, digital signatures, certificates, chain of trust, GPG |
| 03 | [Hashing Basics](./cyber-security-101/06-cryptography/03-hashing-basics/) | Hash functions, MD5, SHA-256, rainbow tables, salting, bcrypt, hashcat, HMAC, file integrity, encoding vs encryption |
| 04 | [John the Ripper: The Basics](./cyber-security-101/06-cryptography/04-john-the-ripper-the-basics/) | Basic cracking, NTLM, unshadow, single crack mode, custom rules, zip2john, ssh2john, *2john utilities |

> ✅ Cryptography — 4/4 rooms documented.

#### Exploitation Basics

| # | Room | Key Topics |
|---|---|---|
| 01 | [Moniker Link](./cyber-security-101/07-exploitation-basics/01-moniker-link/) | CVE-2024-21413, Outlook Protected View bypass, file:// Moniker Link, Responder, netNTLMv2, YARA detection |
| 02 | [Metasploit: Introduction](./cyber-security-101/07-exploitation-basics/02-metasploit-introduction/) | Metasploit modules, msfconsole, search, use, show options, set, payloads, auxiliary, encoders, NOPs, msfdb |
| 03 | [Metasploit: Exploitation](./cyber-security-101/07-exploitation-basics/03-metasploit-exploitation/) | db_nmap, hosts, services, auxiliary scanners, MS17-010, EternalBlue, sessions, shell_to_meterpreter |
| 04 | Metasploit: Meterpreter | 🔄 Pending |
| 05 | Blue | 🔄 Pending |
