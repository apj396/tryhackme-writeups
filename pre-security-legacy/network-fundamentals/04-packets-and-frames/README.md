# Packets & Frames

**Module:** Pre-Security (Legacy) → Network Fundamentals  
**Difficulty:** Easy  
**Platform:** TryHackMe  
**Room Link:** https://tryhackme.com/room/packetsframes  
**Date Completed:** January 2026  
**Author:** Adwait Joshi

---

## What This Room Covers

This room moves from the abstract (OSI layers) to the concrete — how data is actually broken down, addressed, transmitted, and reassembled across a network. It covers packets and frames as distinct data units, the TCP three-way handshake, TCP vs UDP as competing philosophies of data delivery, and practical port number knowledge.

If the OSI model was the map, this room is the vehicle.

---

## Key Concepts

### 1. Packets vs Frames — The Distinction That Matters

These terms are often used interchangeably in casual conversation. They shouldn't be.

**Frame** — Layer 2 (Data Link) data unit. Contains MAC addresses. Handles delivery *within* a local network segment. A frame doesn't survive crossing a router — the router strips the frame, reads the packet inside, and wraps it in a new frame for the next hop.

**Packet** — Layer 3 (Network) data unit. Contains IP addresses. Handles delivery *between* networks. A packet persists across the entire journey from source to destination, hopping through routers along the way.

The relationship: a packet is encapsulated inside a frame for each individual hop. Same packet, different frame at each leg of the journey.

**Why this matters in security:** When you capture traffic in Wireshark, you're capturing frames. The packet (and everything above it) is nested inside. Understanding the boundary between frame and packet is what makes reading captures structured rather than chaotic.

---

### 2. Packet Structure

An IP packet has two components — the **header** and the **payload**.

The header carries metadata the network needs to deliver the packet:

| Field | Purpose |
|---|---|
| Source IP | Where the packet came from |
| Destination IP | Where the packet is going |
| TTL (Time to Live) | Max hops before the packet is discarded |
| Protocol | What's inside the payload (TCP=6, UDP=17, ICMP=1) |
| Checksum | Error detection for the header |

**TTL is worth understanding specifically:** Every router a packet passes through decrements the TTL by 1. When TTL hits zero, the packet is dropped and an ICMP "Time Exceeded" message is sent back to the source. This is exactly how `traceroute` works — it deliberately sends packets with low TTLs to map each hop along the path.

**Real-world relevance:** Abnormally low or inconsistent TTL values in packet captures can indicate spoofed traffic or OS fingerprinting attempts. TTL values also vary by OS (Windows default: 128, Linux default: 64), making them a passive fingerprinting signal.

---

### 3. TCP — Transmission Control Protocol

TCP is connection-oriented. Before a single byte of application data moves, TCP establishes a connection through the **three-way handshake**:
```
Client                          Server
  |                               |
  |——— SYN ———————————————————————>|   "I want to connect"
  |                               |
  |<—— SYN-ACK ———————————————————|   "Acknowledged, I'm ready"
  |                               |
  |——— ACK ————————————————————————>|   "Connection established"
  |                               |
  |====== Data flows both ways ===|
  |                               |
  |——— FIN ————————————————————————>|   "I'm done"
  |<—— FIN-ACK ———————————————————|   "Acknowledged, closing"
```

After the handshake, TCP guarantees:
- **Delivery** — lost packets are retransmitted
- **Ordering** — segments are reassembled in the correct sequence
- **Error checking** — checksums validate data integrity

This reliability comes at the cost of overhead — the handshake, acknowledgements, and retransmission logic all add latency.

**TCP headers carry sequence numbers** — each segment is numbered so the receiver can reassemble out-of-order data and request retransmission of missing segments.

**Real-world relevance:**

The three-way handshake is one of the most exploited mechanisms in networking:

- **SYN flood** — attacker sends massive volumes of SYN packets without completing the handshake, exhausting the server's connection table
- **SYN scan (nmap -sS)** — sends SYN, waits for SYN-ACK to confirm a port is open, then sends RST instead of ACK to avoid completing the connection. Stealthier than a full connect scan
- **Session hijacking** — predicting or stealing TCP sequence numbers to inject into an established session
```bash
# TCP SYN scan — most common nmap default
nmap -sS 192.168.1.1

# Full TCP connect scan — completes the handshake
nmap -sT 192.168.1.1
```

---

### 4. UDP — User Datagram Protocol

UDP is connectionless. It sends data without establishing a session, without guaranteeing delivery, and without caring whether the receiver got it.

That sounds like a flaw. It's a deliberate tradeoff.

**Where UDP wins:**
- No handshake overhead — data starts flowing immediately
- No retransmission — dropped packets are simply gone, which is fine for real-time data where a late packet is worse than a missing one
- Lower latency — critical for applications where speed matters more than perfection

**Use cases:** DNS lookups, VoIP, video streaming, online gaming, DHCP

**The mental model:** TCP is a recorded delivery letter — tracked, confirmed, redelivered if lost. UDP is a flyer through a letterbox — sent without knowing or caring if anyone picked it up.

**Real-world relevance:** UDP is commonly used in amplification DDoS attacks. DNS and NTP — both UDP-based — can be abused to reflect and amplify traffic toward a target. A small spoofed UDP request generates a large response directed at the victim's IP.

---

### 5. Port Numbers

Ports are Layer 4 identifiers that tell the receiving system which application or service should handle incoming data. Combined with an IP address, a port number uniquely identifies a specific communication endpoint — called a **socket**.

**Three ranges:**

| Range | Name | Description |
|---|---|---|
| 0–1023 | Well-known ports | Reserved for standard services |
| 1024–49151 | Registered ports | Assigned to specific applications |
| 49152–65535 | Dynamic/ephemeral ports | Temporarily assigned by the OS for outbound connections |

**Ports worth knowing by memory:**

| Port | Protocol | Service |
|---|---|---|
| 21 | TCP | FTP |
| 22 | TCP | SSH |
| 23 | TCP | Telnet |
| 25 | TCP | SMTP |
| 53 | TCP/UDP | DNS |
| 80 | TCP | HTTP |
| 443 | TCP | HTTPS |
| 445 | TCP | SMB |
| 3389 | TCP | RDP |
| 3306 | TCP | MySQL |
| 8080 | TCP | HTTP (alternate) |

**Real-world relevance:** Port scanning is the first phase of reconnaissance — mapping which ports are open tells an attacker (or a penetration tester) which services are running. In a SOC context, unexpected open ports or traffic on unusual ports are high-priority anomalies. RDP (3389) and SMB (445) open to the internet are near-universal findings in incident reports.

---

## Walkthrough Notes

### Task 1 — What are Packets and Frames?
The core distinction — frames are Layer 2, packets are Layer 3 — is established here. The key detail worth holding: frames change at every hop (new MAC addresses for each segment), while the packet (IP addresses) stays consistent end-to-end.

### Task 2 — TCP/IP (Three-Way Handshake)
The handshake diagram is the centrepiece of this task. Worth understanding not just *what* each step is, but *why* each step exists — SYN establishes intent, SYN-ACK confirms the server is available and listening, ACK confirms the client received that confirmation. Each step is a verification.

### Task 3 — UDP/IP
The contrast with TCP is the point here. UDP's lack of handshake isn't a bug — it's the feature that makes it appropriate for real-time applications where latency matters more than guaranteed delivery.

### Task 4 — Ports 101
The practical exercise involves connecting to services on specific ports to retrieve flags. The mechanism is straightforward:
```bash
# Connect to a service on a specific port
nc 10.10.x.x 8080

# Or using telnet
telnet 10.10.x.x 22
```

The exercise makes port numbers tangible — instead of memorising a table, you're actually knocking on specific doors and seeing what answers.

---

## Commands Used
```bash
# TCP SYN scan (half-open) — stealthy port scan
nmap -sS 192.168.1.1

# Full connect scan — completes handshake
nmap -sT 192.168.1.1

# Scan specific ports
nmap -p 21,22,80,443,3389 192.168.1.1

# Connect to a port with netcat
nc <target-ip> <port>

# Traceroute — uses TTL to map network hops
traceroute 8.8.8.8        # Linux
tracert 8.8.8.8           # Windows
```

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| TTL values | Passive OS fingerprinting; spoofed traffic detection |
| Three-way handshake | Baseline for SYN flood detection; nmap scan identification |
| SYN without ACK (half-open) | SYN scan or SYN flood in progress |
| UDP traffic spikes | Potential amplification DDoS; check source/destination ports |
| Unexpected open ports | Key finding in vulnerability assessments and incident triage |
| Port 3389 / 445 exposed | Critical misconfiguration — RDP and SMB are primary ransomware entry points |
| Ephemeral ports | High-numbered source ports in captures indicate outbound connections |

---

## Takeaways

Three things this room makes concrete that stay relevant throughout your career:

1. **Packets and frames are not the same thing, and the difference matters when reading captures.** A frame is local. A packet travels the full journey. Mixing them up when describing traffic in an incident report signals imprecision — getting them right signals clarity.

2. **The three-way handshake is both a protocol and an attack surface.** Understanding its mechanics — SYN, SYN-ACK, ACK — immediately explains SYN floods, SYN scans, and session hijacking without needing to study each attack separately. The attack follows directly from the protocol.

3. **Port numbers are not just a memorisation exercise.** Knowing that 445 is SMB and 3389 is RDP tells you, at a glance, what a connection *means* in context — whether it's expected traffic or a red flag. That pattern recognition is what separates fast triage from slow triage.

---
