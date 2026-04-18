# Networking Concepts

**Module:** Cyber Security 101 → Networking
**Difficulty:** Easy
**Platform:** TryHackMe
**Room Link:** https://tryhackme.com/room/networkingconcepts
**Date Completed:** February 2026
**Author:** Adwait Joshi

---

## What This Room Covers

This room establishes the foundational models and vocabulary that underpin all network-level analysis. It covers the OSI and TCP/IP models, IP addressing and subnetting, transport layer protocols, encapsulation, and finishes with a hands-on Telnet exercise that puts the theory directly to work. For a SOC analyst, this is ground truth — every log, alert, and packet capture is interpreted through the mental models introduced here.

---

## Key Concepts

### 1. The OSI Model

The OSI (Open Systems Interconnection) model is a conceptual framework developed by ISO to describe how network communication should be structured. It divides communication into seven distinct layers, each with a specific responsibility.

| Layer | Name | Function | Example Protocols |
|---|---|---|---|
| 7 | Application | User-facing services | HTTP, DNS, FTP, SMTP |
| 6 | Presentation | Encoding, encryption, compression | TLS, SSL |
| 5 | Session | Session establishment and management | NetBIOS, RPC |
| 4 | Transport | End-to-end delivery, segmentation | TCP, UDP |
| 3 | Network | Logical addressing and routing | IP, ICMP |
| 2 | Data Link | Physical addressing, framing | Ethernet, Wi-Fi |
| 1 | Physical | Raw bit transmission | Cables, radio signals |

The OSI model is theoretical — it describes how communication *should* work, not how it is literally implemented. Its value is precision: when something breaks or something looks suspicious, naming the layer immediately narrows the investigative search space.

**Real-world relevance:** Layer awareness is diagnostic shorthand in both troubleshooting and threat analysis. ARP poisoning lives at Layer 2. IP spoofing at Layer 3. Port scanning at Layer 4. Phishing at Layer 7. An analyst who reasons in layers moves faster than one who treats the network as a single undifferentiated system.

---

### 2. The TCP/IP Model

The TCP/IP model is the practical implementation that governs actual internet traffic. It collapses the OSI model's seven layers into four:

| TCP/IP Layer | OSI Equivalent | Function |
|---|---|---|
| Application | Layers 5, 6, 7 | User services, encoding, session management |
| Transport | Layer 4 | End-to-end delivery |
| Internet | Layer 3 | Logical addressing, routing |
| Link | Layer 2 | Physical addressing, local delivery |

The TCP/IP model was developed by the US Department of Defense in the 1970s and remains the operational framework underlying all internet communication. The OSI model is the reference model you reason with; TCP/IP is the model the network actually runs on.

**Real-world relevance:** Security advisories, CVEs, and RFCs are written with TCP/IP layer terminology. Being fluent in both models — and in the mapping between them — allows you to read technical documentation accurately and trace a described vulnerability to the correct point in the communication stack.

---

### 3. IP Addresses and Subnets

Every device on a network requires an IP address to send and receive data. Two versions are in active use.

**IPv4** uses a 32-bit address expressed in dotted decimal notation:

    192.168.1.1

**IPv6** uses a 128-bit address expressed in hexadecimal groups:

    2001:0db8:85a3:0000:0000:8a2e:0370:7334

IPv6 solves the address exhaustion problem of IPv4 and eliminates the need for NAT — each device can carry a globally unique address. Both versions coexist in most modern networks.

**Subnetting** divides an address space into network and host portions. CIDR notation expresses this compactly:

    192.168.1.0/24

The `/24` means 24 bits identify the network, leaving 8 bits for host addresses — 254 usable hosts. Subnets define traffic boundaries: communication within a subnet stays local; communication across subnets requires routing.

**Real-world relevance:** IP addressing and subnet notation appear throughout firewall rules, access control lists, and SIEM alert conditions. In alert triage, the first question is always "is this address internal or external?" — which requires knowing the organisation's private address ranges. Misreading a CIDR range in a firewall rule means misunderstanding which hosts that rule applies to.

---

### 4. TCP and UDP

Both protocols operate at Layer 4 (Transport). Their behaviour is fundamentally different and the choice between them reflects a deliberate tradeoff.

**TCP (Transmission Control Protocol)** is connection-oriented. It establishes a session via a three-way handshake before any data is exchanged:

    Client → Server    SYN
    Server → Client    SYN-ACK
    Client → Server    ACK

TCP guarantees delivery, ordering, and error-checking. Used where reliability matters: HTTP, HTTPS, SSH, FTP.

**UDP (User Datagram Protocol)** is connectionless. There is no handshake and no delivery guarantee — packets are fired and the sender moves on. Used where speed matters more than reliability: DNS queries, video streaming, VoIP.

Port numbers (0–65535) direct traffic to the correct service on a host. There are approximately 65,000 available port numbers — a figure worth remembering as shorthand for the scale of addressable services on any given host.

**Real-world relevance:** The TCP three-way handshake is the basis for SYN flood detection — an attacker sending SYN packets without completing the handshake fills the server's connection table. Half-open connections at scale are an immediate indicator. UDP's connectionless nature means DNS traffic appears in captures as isolated request-response pairs — understanding this prevents false positive triage on what is normal DNS behaviour.

---

### 5. Encapsulation

Encapsulation is the process by which each layer adds its own header — and sometimes a trailer — to the data received from the layer above, before passing the unit downward. On the receiving end, each layer strips its header before passing the payload upward. This reverse process is de-encapsulation.

The unit of data changes name at each layer:

| Layer | Data Unit |
|---|---|
| Application / Presentation / Session | Data |
| Transport | Segment (TCP) / Datagram (UDP) |
| Network / Internet | Packet |
| Data Link | Frame |
| Physical | Bits |

On a Wi-Fi network, an IP packet is encapsulated within a Wi-Fi frame at the Data Link layer before transmission. Each header added during encapsulation contains metadata specific to that layer's function — source/destination port at Layer 4, source/destination IP at Layer 3, source/destination MAC at Layer 2.

**Real-world relevance:** Encapsulation is what makes protocol tunnelling attacks dangerous. A malicious payload wrapped inside legitimate DNS or ICMP traffic is exploiting the trust each layer places in the layer below it. In Wireshark, each encapsulation layer is visible as a distinct expandable section — knowing the layer structure tells you exactly where to look for the information you need.

---

## Walkthrough Notes

### Task 1 — Introduction
Sets the learning objectives and prerequisites for the room. The room is the first in a four-room networking series within CS101. Prior familiarity with the terms IP address and TCP port number is assumed; deep technical understanding is not required.

### Task 2 — OSI Model
Covers all seven OSI layers with function descriptions and examples. Questions test which layer handles specific functions — end-to-end application communication (Layer 4), packet routing (Layer 3), data encoding (Layer 6).

### Task 3 — TCP/IP Model
Covers the four TCP/IP layers and their mapping to OSI. Questions test layer identification — HTTP belongs to the Application layer; the Application layer in TCP/IP covers three OSI layers (5, 6, 7).

### Task 4 — IP Addresses and Subnets
Covers IPv4, IPv6, and CIDR subnetting. Questions involve identifying address types and reading subnet notation correctly.

### Task 5 — UDP and TCP
Covers both transport protocols and port number ranges. Questions test protocol behaviour — TCP requires the three-way handshake; there are approximately 65,000 port numbers.

### Task 6 — Encapsulation
Covers the encapsulation process and data unit names at each layer. Questions test the correct unit name at a given layer — IP packets are encapsulated within Wi-Fi frames on a wireless network.

### Task 7 — Telnet (Hands-On Lab)
Interactive terminal task. Telnet is used to connect directly to open TCP ports and communicate using the application-layer protocol as plain text. Three services are contacted:

- **Echo server (port 7):** Text typed is echoed back — demonstrates bidirectional TCP communication
- **Daytime server (port 13):** Returns current date and time, then closes — demonstrates a stateless single-response service
- **HTTP server (port 80):** A manual GET request is issued — demonstrates that HTTP is plain text at the application layer

The task question asks for the HTTP server name and version from the response header — this is read directly from the raw server response returned by Telnet and is specific to the live machine.

### Task 8 — Conclusion
Summarises all topics covered. Directs to the next room in the series: Networking Essentials.

---

## Commands Reference

    # Connect to echo service (port 7) — text sent is echoed back
    telnet MACHINE_IP 7

    # Connect to daytime service (port 13) — returns current datetime and closes
    telnet MACHINE_IP 13

    # Connect to web server (port 80) — then issue manual HTTP request
    telnet MACHINE_IP 80
    GET / HTTP/1.1
    Host: telnet.thm

    # View network interface and IP address (Linux)
    ifconfig
    ip a s

    # View network adapter details (Windows)
    ipconfig /all

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| OSI layer awareness | Localising attack layer — ARP poisoning (L2), IP spoofing (L3), port scanning (L4), phishing (L7) |
| TCP three-way handshake | SYN flood detection — half-open connections at scale are a DoS indicator |
| UDP connectionless behaviour | Normal DNS appears as isolated pairs in captures — prevents false positive triage |
| Encapsulation / framing | Reading Wireshark captures layer by layer; identifying protocol tunnelling attempts |
| IP addressing and CIDR | Interpreting firewall rules and ACLs; distinguishing internal from external traffic in SIEM |
| TCP/IP vs OSI models | Mapping real protocol behaviour to theoretical layers when reading CVEs and RFCs |
| Telnet HTTP interaction | Mental model for raw HTTP structure; understanding why unencrypted protocols expose data in transit |

---

## Takeaways

1. **The OSI model is a reasoning tool, not a protocol.** Its value isn't that the network literally has seven discrete layers — it's that thinking in layers lets you localise problems and attacks with precision. An analyst who can say "this is a Layer 3 issue" has already eliminated a significant portion of the investigative search space before touching a single tool.

2. **Encapsulation is what makes protocol independence possible — and protocol tunnelling attacks dangerous.** Each layer trusts that the layer below will carry its payload faithfully. An attacker wrapping a malicious payload inside legitimate DNS or ICMP traffic is exploiting exactly this trust. Knowing the encapsulation structure tells you not just how data travels, but where evasion hides.

3. **Telnet's value in 2026 is pedagogical, not operational.** Using it to hand-craft an HTTP request makes viscerally clear that application-layer protocols are formatted text over TCP — nothing more. That intuition, that protocols are conventions rather than magic, is foundational for understanding how they fail, how they leak, and how they are abused.

---
