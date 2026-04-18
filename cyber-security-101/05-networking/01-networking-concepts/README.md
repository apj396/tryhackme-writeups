# Networking Concepts

**Path:** Cyber Security 101
**Module:** Networking (05)
**Room:** Networking Concepts (01)
**Completed:** February 2026
**Author:** Adwait Joshi

---

## What This Room Covers

This room establishes the foundational models and vocabulary that underpin all
network-level analysis. It covers the OSI and TCP/IP models, IP addressing and
subnetting, transport layer protocols, encapsulation, and finishes with a hands-on
Telnet exercise that puts the theory to work. For a SOC analyst, this is ground
truth — every log, alert, and packet capture is interpreted through the mental
models introduced here.

---

## Key Concepts

### The OSI Model

The OSI (Open Systems Interconnection) model is a conceptual framework developed
by ISO to describe how network communication should be structured. It divides
communication into seven distinct layers, each with a specific responsibility.

| Layer | Name | Function | Example Protocols |
|---|---|---|---|
| 7 | Application | User-facing services | HTTP, DNS, FTP, SMTP |
| 6 | Presentation | Encoding, encryption, compression | TLS, SSL |
| 5 | Session | Session establishment and management | NetBIOS, RPC |
| 4 | Transport | End-to-end delivery, segmentation | TCP, UDP |
| 3 | Network | Logical addressing and routing | IP, ICMP |
| 2 | Data Link | Physical addressing, framing | Ethernet, Wi-Fi |
| 1 | Physical | Raw bit transmission | Cables, radio signals |

The OSI model is theoretical — it describes *how* communication should work.
Understanding which layer an event belongs to is a core analyst skill: it
immediately narrows where to look when something goes wrong or something looks
suspicious.

### The TCP/IP Model

The TCP/IP model is the practical implementation that governs actual internet
traffic. It collapses the OSI model's seven layers into four:

| TCP/IP Layer | OSI Equivalent | Function |
|---|---|---|
| Application | Layers 5, 6, 7 | User services, encoding, session |
| Transport | Layer 4 | End-to-end delivery |
| Internet | Layer 3 | Logical addressing, routing |
| Link | Layer 2 | Physical addressing, local delivery |

The key distinction: OSI is the reference model you reason with; TCP/IP is the
model the network actually runs on.

### IP Addresses and Subnets

Every device on a network requires an IP address to send and receive data.

**IPv4** uses a 32-bit address in dotted decimal notation:
192.168.1.1

**IPv6** uses a 128-bit address in hexadecimal:
2001:0db8:85a3:0000:0000:8a2e:0370:7334

IPv6 solves the address exhaustion problem of IPv4 and removes the need for NAT
— each device can carry a globally unique address. Both versions coexist in most
modern networks.

**Subnetting** divides an address space into network and host portions using a
subnet mask. CIDR notation expresses this compactly:
192.168.1.0/24

The `/24` means 24 bits identify the network — leaving 8 bits for host addresses
(254 usable hosts). Subnets define traffic boundaries: communication within a
subnet stays local; communication across subnets requires routing.

### TCP and UDP

Both operate at Layer 4 (Transport). Their behaviour is fundamentally different.

**TCP (Transmission Control Protocol)**
- Connection-oriented — establishes a session via three-way handshake:
  SYN → SYN-ACK → ACK
- Guarantees delivery, ordering, and error-checking
- Higher overhead; used where reliability matters: HTTP, HTTPS, SSH

**UDP (User Datagram Protocol)**
- Connectionless — no handshake, no delivery guarantee
- Lower overhead, higher speed
- Used where latency matters more than reliability: DNS, video streaming, VoIP

Port numbers (0–65535) direct traffic to the correct service on a host. There
are approximately 65,000 available port numbers — a figure worth remembering as
shorthand for the scale of addressable services.

### Encapsulation

Encapsulation is the process by which each layer of the network model adds its
own header (and sometimes a trailer) to the data received from the layer above
before passing it downward. In reverse, each layer on the receiving end strips
its header before passing the data upward — this is de-encapsulation.

The data unit changes name at each layer:

| Layer | Data Unit |
|---|---|
| Application / Presentation / Session | Data |
| Transport | Segment (TCP) / Datagram (UDP) |
| Network / Internet | Packet |
| Data Link | Frame |
| Physical | Bits |

On a Wi-Fi network, an IP packet is encapsulated within a Wi-Fi frame at the
Data Link layer before transmission. Understanding this layered wrapping is
essential for reading packet captures — each header is visible in a tool like
Wireshark as a distinct layer that can be inspected independently.

---

## Walkthrough Notes

The room's first six tasks are conceptual — questions test comprehension of layer
functions, protocol behaviour, and addressing. The reasoning approach that holds
across these tasks:

1. Identify the layer the question operates at — OSI layer number or TCP/IP
   layer name
2. Apply the correct unit of data for that layer (segment, packet, frame)
3. For addressing questions, distinguish logical (IP) from physical (MAC) from
   service-level (port)

**Task 7 — Telnet hands-on lab**

This task uses Telnet to demonstrate that TCP communication is just text exchange
over an open connection. Three services are contacted directly:

- **Echo server (port 7):** Whatever you type is echoed back — demonstrates
  bidirectional TCP communication
- **Daytime server (port 13):** Returns current date and time, then closes the
  connection — demonstrates a stateless single-response service
- **HTTP server (port 80):** A manual HTTP GET request is issued over Telnet —
  demonstrates that HTTP is plain text at the application layer

The critical insight from this task: Telnet exposes what higher-level tools
abstract away. When you connect to port 80 and type a raw HTTP request, you are
doing exactly what a browser does — the only difference is the browser handles
the formatting automatically.

---

## Commands Reference

```bash
# Connect to echo service (port 7) — text sent is echoed back
telnet MACHINE_IP 7

# Connect to daytime service (port 13) — returns current datetime
telnet MACHINE_IP 13

# Connect to web server (port 80) — then issue manual HTTP request
telnet MACHINE_IP 80
GET / HTTP/1.1
Host: telnet.thm

# View network interface and IP address (Linux)
ifconfig
ip a s

# View network adapter information (Windows)
ipconfig /all
```

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| OSI layer awareness | Localising where an attack operates — ARP poisoning (L2), IP spoofing (L3), port scanning (L4), phishing (L7) |
| TCP three-way handshake | Recognising SYN flood patterns; interpreting incomplete connection states in IDS alerts |
| UDP behaviour | Understanding connectionless DNS traffic in captures; identifying DNS amplification attack patterns |
| Encapsulation / framing | Reading packet captures layer by layer in Wireshark; understanding what each header field contributes |
| IP addressing and subnets | Interpreting firewall rules and ACLs; distinguishing internal from external traffic in SIEM logs |
| Telnet (port 80 interaction) | Mental model for HTTP request structure; understanding why unencrypted protocols leak data |
| TCP/IP vs OSI models | Mapping real protocol behaviour to theoretical layers when reading CVEs, RFCs, and security advisories |

---

## Takeaways

1. **The OSI model is a reasoning tool, not a protocol.** Its value isn't that
   the network actually has seven discrete layers — it's that thinking in layers
   lets you localise problems and attacks quickly. An analyst who can say "this
   is a Layer 3 issue" has already eliminated a significant portion of the
   investigative search space.

2. **Encapsulation is what makes protocol independence possible.** Each layer
   trusts that the layer below will carry its payload faithfully. This is also
   why protocol tunnelling attacks work — a malicious payload can be wrapped in
   a legitimate outer layer and carried through controls that only inspect the
   envelope.

3. **Telnet's value in 2026 is pedagogical, not operational.** Using it to hand-
   craft an HTTP request makes viscerally clear that application-layer protocols
   are just formatted text over TCP. That intuition — that protocols are
   conventions, not magic — is foundational for understanding how they break.

---
