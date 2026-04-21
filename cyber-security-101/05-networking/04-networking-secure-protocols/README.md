# Networking Secure Protocols

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Networking |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/networkingsecureprotocols |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the fourth and final room in the CS101 Networking series. The previous room established what the core application-layer protocols do — and made clear that most of them transmit everything in plaintext. This room addresses that problem directly. TLS is the cryptographic layer added to HTTP, SMTP, POP3, and IMAP to produce their secure equivalents. SSH replaces Telnet for remote access and extends into secure file transfer via SFTP. FTPS applies TLS to FTP. VPNs encrypt traffic across untrusted networks entirely. The room examines each in turn, with Wireshark captures to show what encrypted traffic looks like compared to its plaintext predecessor, and a practical component to close.

---

## Key Concepts

### TLS — Transport Layer Security

TLS is the cryptographic protocol that secures communication over networks by providing three properties: confidentiality (data cannot be read in transit), integrity (data cannot be altered in transit), and authenticity (the server is who it claims to be). It operates at the Transport layer of the OSI model.

The history matters for context. Netscape Communications developed SSL (Secure Sockets Layer) and released SSL 2.0 in 1995 as the first public version, recognising the need for secure web communication. In 1999, the IETF developed TLS 1.0 as an upgrade to SSL 3.0, incorporating improved security measures. TLS 1.3, a significant overhaul of the protocol, was released in 2018 and is the current standard. Although SSL and TLS are sometimes used interchangeably in conversation, SSL is deprecated and TLS is what is actually in use.

TLS establishment involves a negotiation — a handshake — before any application data is exchanged. In the Wireshark captures shown in the room, this negotiation completes in 8 packets. The GET request to a login page, which follows after the TLS session is established, appears at packet 10 — and unlike the same request over HTTP, its contents are encrypted and unreadable in the capture.

Two certificate-related points are operationally significant. First, TLS certificates are issued by Certificate Authorities (CAs) — trusted third parties that verify a server's identity before signing its certificate. Second, self-signed certificates — created and signed by the server itself without CA verification — should not be used to confirm server authenticity in production environments. They provide encryption but no identity assurance, leaving the connection vulnerable to man-in-the-middle attacks.

### HTTPS

HTTPS is HTTP with TLS applied. The application-layer protocol is identical — the same request methods, status codes, and headers — but the entire exchange is wrapped in a TLS session before transmission. The default port shifts from 80 to 443.

The practical implication visible in the room's Wireshark captures: over HTTP, a GET to a login endpoint shows the path, host, cookies, and any submitted credentials in plaintext. Over HTTPS, the same exchange is opaque — only the TLS handshake packets and the encrypted application data records are visible. The content is present in the capture but unreadable without the session keys.

### SMTPS, POP3S, and IMAPS

TLS is applied to the email protocol suite in the same pattern — the underlying protocol is unchanged, TLS wraps it, and the port number shifts:

| Plaintext Protocol | Port | Secure Version | Port |
|---|---|---|---|
| SMTP | 25 | SMTPS | 465 / 587 |
| POP3 | 110 | POP3S | 995 |
| IMAP | 143 | IMAPS | 993 |

An important distinction for practical packet analysis: if network traffic is captured containing SMTPS or POP3S, the credentials and message content are encrypted and cannot be extracted without the session keys. IMAP without TLS — plain IMAP on port 143 — transmits credentials in cleartext and is therefore the protocol from which login credentials can be extracted directly from a capture. This is a standard exam and interview point.

### SSH — Secure Shell

SSH was developed by Tatu Ylönen and released as SSH-1 in 1995 — the same year Netscape released SSL 2.0. A more secure version, SSH-2, was defined in 1996. In 1999, the OpenBSD developers released OpenSSH, an open-source implementation of SSH that has become the de facto standard. When an SSH client is used today, it is almost certainly built on OpenSSH libraries.

SSH replaces Telnet for remote terminal access. Where Telnet sends everything — including credentials and commands — in plaintext, SSH encrypts the entire session. Beyond remote shell access, SSH supports several authentication methods including password, public key, and two-factor authentication. It also serves as the foundation for secure file transfer protocols.

    ssh user@MACHINE_IP

The default port for SSH is 22, replacing Telnet's port 23.

### SFTP and FTPS

Two secure alternatives to FTP exist and are frequently confused with each other.

**SFTP (SSH File Transfer Protocol)** is part of the SSH protocol suite and operates over port 22. It is not FTP with encryption added — it is an entirely different protocol that runs inside an SSH session. Setting up an SFTP server requires only enabling the relevant option within an OpenSSH server configuration.

**FTPS (File Transfer Protocol Secure)** is FTP with TLS applied — the same relationship that HTTPS has to HTTP. It typically operates on port 990 and requires a proper TLS certificate to run securely. FTPS retains FTP's use of separate control and data connections, which can make it difficult to allow through strict firewalls.

    sftp user@MACHINE_IP

The key distinction: SFTP uses SSH infrastructure (port 22, no separate certificate required beyond SSH keys). FTPS uses TLS infrastructure (port 990, certificate required). They are different protocols that happen to produce a similar result.

### VPN — Virtual Private Network

A VPN creates an encrypted tunnel between a client and a VPN server, allowing the client to access a remote network as if physically connected to it. All traffic between the client and the VPN server is encrypted, regardless of what protocols are carrying the application data inside.

The primary use cases are two: enterprise remote access (employees connecting to internal company resources from outside the office) and site-to-site connectivity (linking geographically separated offices so devices across locations can communicate as though on the same local network). VPNs are also used to mask the client's public IP address from external services, since traffic appears to originate from the VPN server's address.

---

## Walkthrough Notes

The room runs through eight tasks covering TLS theory, each secure protocol, and a practical exercise.

**Task 1 (Introduction):** Sets out the learning objectives and notes that this room assumes completion of Networking Core Protocols at minimum. The framing is direct — the previous room showed what plaintext protocols expose; this room shows the fixes. No questions.

**Task 2 (TLS):** Covers the history from SSL 2.0 through TLS 1.3, the handshake process, and the role of Certificate Authorities. Key questions: the protocol TLS upgraded and built upon is SSL; the certificate type that should not be used to confirm server authenticity is a self-signed certificate. The Wireshark screenshots show the TLS negotiation completing in 8 packets.

**Task 3 (HTTPS):** Compares Wireshark captures of HTTP and HTTPS for the same login page request. Over HTTP, the GET request and its contents are visible in plaintext. Over HTTPS, the same request is at packet 10 in the capture — present but encrypted. The contrast between the two captures is the core learning point of this task.

**Task 4 (SMTPS, POP3S, and IMAPS):** Covers the port number mappings for all three secure email protocols and the effect of TLS on captured traffic. The key question asks which protocol from the set — SMTPS, POP3S, or IMAP — would expose login credentials in a packet capture. The answer is IMAP — the plaintext version on port 143, not its secure equivalent IMAPS on port 993.

**Task 5 (SSH):** Covers the SSH development history, OpenSSH as the open-source implementation, and the shift from Telnet (port 23) to SSH (port 22). The question on the open-source implementation of SSH is answered by OpenSSH. Authentication methods and the practical use of SSH for remote access are discussed.

**Task 6 (SFTP and FTPS):** Distinguishes SFTP from FTPS — SFTP runs over SSH on port 22, FTPS uses TLS on port 990. The practical component uses the `sftp` command to establish a secure file transfer session with the lab machine. The room notes that FTPS can be difficult to configure through strict firewalls due to its separate control and data connections.

**Task 7 (VPN):** Covers VPN use cases — remote access and site-to-site connectivity — and the encryption model. The practical interactive site is accessed via the View Site button and walks through a simulated scenario applying the secure protocols covered in the room to obtain the flag.

**Task 8 (Conclusion):** Summarises the full four-room Networking series and notes that Networking Secure Protocols is the endpoint of this module. No answer required.

---

## Commands Used

    ssh user@MACHINE_IP
    sftp user@MACHINE_IP

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| TLS handshake packet count | Baseline awareness for TLS anomaly detection — unexpected handshake failures or unusually high TLS negotiation volume can indicate scanning or TLS-based attack activity |
| Self-signed certificates | Certificate validation alerts — browsers and security tools flag self-signed certificates; their presence on internal services may indicate misconfiguration or a rogue server |
| HTTPS vs HTTP in captures | Incident investigation — determining whether credentials or session tokens were transmitted over HTTP rather than HTTPS is a standard step in data exposure assessments |
| IMAP plaintext vs IMAPS | Mail client audit — detecting port 143 traffic on the network identifies mail clients operating without TLS, flagging a credential interception risk on the same segment |
| SSH replacing Telnet | Hardening verification — presence of Telnet (port 23) traffic or open Telnet services on internal hosts is a finding in any security review; SSH on port 22 is the expected replacement |
| SFTP vs FTPS | File transfer security review — SFTP requires only SSH infrastructure and is lower friction to deploy securely; FTPS requires certificate management and firewall consideration, making misconfigurations more likely |
| VPN encrypted tunnel | Remote access security — VPN traffic in captures shows encrypted payloads; confirming that remote workers connect via VPN before accessing internal resources is a standard access control verification |

---

## Takeaways

1. **TLS is not a protocol — it is a layer applied to existing protocols, and understanding which version of a protocol is in use (plaintext vs TLS-wrapped) determines what an attacker can see.** IMAP and IMAPS carry the same email. HTTP and HTTPS carry the same web content. The difference is entirely in whether TLS is present. In a SOC context, identifying plaintext protocol traffic on the network is not just a configuration note — it is an active risk that defines what an attacker with network access can read.

2. **SFTP and FTPS are not the same protocol and the distinction has operational consequences.** SFTP inherits SSH's infrastructure — keys, port 22, no separate certificate management. FTPS requires TLS certificates, uses separate ports for control and data, and interacts poorly with strict firewalls. Choosing between them is an architectural decision that affects both security posture and operational complexity. Conflating them in documentation or configuration is a common source of misconfiguration.

3. **The progression across this four-room series mirrors how network security actually developed historically — function first, security retrofitted later.** TCP/IP was built for reliability, not confidentiality. DNS, FTP, SMTP, and HTTP were built to work, not to resist interception. TLS, SSH, SFTP, and VPNs are all answers to the same question asked too late: what happens when the network cannot be trusted? That sequence — build, deploy at scale, discover the threat model, retrofit security — is a pattern that recurs throughout the history of technology, and understanding it is as important as knowing the port numbers.

---
