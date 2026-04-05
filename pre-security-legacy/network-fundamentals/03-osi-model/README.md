# OSI Model

**Module:** Pre-Security (Legacy) → Network Fundamentals  
**Difficulty:** Easy  
**Platform:** TryHackMe  
**Room Link:** https://tryhackme.com/room/osimodelzi  
**Date Completed:** January 2026  
**Author:** Adwait Joshi

---

## What This Room Covers

The OSI (Open Systems Interconnection) model is a seven-layer framework that standardises how different network systems communicate with each other. It doesn't describe how networks *actually* work in practice — that's the TCP/IP model — but it gives you a precise vocabulary for isolating where in the communication stack a problem or attack is occurring.

In security, OSI isn't just academic. Every tool, every attack, and every defensive control operates at a specific layer. Knowing the model means knowing *where* something is happening — and that shapes everything from tool selection to incident scoping.

---

## Key Concepts

### The 7 Layers — Top to Bottom

| Layer | Name | Primary Role |
|---|---|---|
| 7 | Application | Interface between network and user-facing software |
| 6 | Presentation | Data formatting, encryption, compression |
| 5 | Session | Managing and maintaining communication sessions |
| 4 | Transport | End-to-end delivery, segmentation, error checking |
| 3 | Network | Logical addressing and routing between networks |
| 2 | Data Link | Physical addressing (MAC), frame delivery on a LAN |
| 1 | Physical | Raw bit transmission over physical media |

**Memory aid:** *"All People Seem To Need Data Processing"* (top to bottom, 7→1)  
Or reversed: *"Please Do Not Throw Sausage Pizza Away"* (bottom to top, 1→7)

---

### Layer 7 — Application

The layer closest to the end user. It provides network services directly to applications — not the applications themselves, but the protocols they use to communicate.

Protocols at this layer: HTTP, HTTPS, FTP, SMTP, DNS, SSH

**What this means in practice:** When your browser makes a web request, it's operating at Layer 7. When an attacker sends a phishing email or performs a SQL injection, they're exploiting Layer 7 behaviour. Web Application Firewalls (WAFs) operate here.

---

### Layer 6 — Presentation

Responsible for translating data into a format the application layer can use. Handles encryption, decryption, and compression.

SSL/TLS encryption operates at this layer — or more precisely, at the boundary between Layer 6 and Layer 4 depending on the implementation. The point is that data format and confidentiality transformations happen here before reaching the application.

**What this means in practice:** When HTTPS encrypts your traffic, that transformation happens at Layer 6. Attackers targeting encryption weaknesses — outdated TLS versions, weak cipher suites — are attacking this layer.

---

### Layer 5 — Session

Manages the establishment, maintenance, and termination of communication sessions between two devices. It ensures that a session stays open long enough to transfer data and closes cleanly when done.

Authentication and session tokens operate conceptually at this layer. If a session is hijacked — an attacker stealing a valid session cookie to impersonate an authenticated user — that's a Layer 5 attack.

---

### Layer 4 — Transport

Where data is broken into **segments** (TCP) or **datagrams** (UDP), and end-to-end delivery is managed.

Two protocols dominate here:

**TCP (Transmission Control Protocol)**
- Connection-oriented — establishes a three-way handshake before data flows
- Guarantees delivery, ordering, and error checking
- Slower but reliable
- Used for: HTTP, HTTPS, SSH, FTP

**UDP (User Datagram Protocol)**
- Connectionless — sends data without establishing a session first
- No delivery guarantee, no ordering, no error recovery
- Faster but unreliable
- Used for: DNS, VoIP, video streaming, online gaming

**Port numbers live at Layer 4.** Port 80 (HTTP), 443 (HTTPS), 22 (SSH), 53 (DNS) — these are all Transport layer identifiers that tell the receiving system which application should handle the incoming data.

**Real-world relevance:** SYN flood attacks target the TCP three-way handshake at Layer 4 — flooding a server with SYN packets without completing the handshake, exhausting connection resources. Port scanning (nmap) also operates here, probing which Layer 4 ports are open on a target.
```bash
# nmap default scan — operates at Layer 4 (TCP SYN scan)
nmap -sS 192.168.1.1

# Scan specific ports
nmap -p 22,80,443 192.168.1.1
```

---

### Layer 3 — Network

Handles logical addressing and routing — determining the best path for data to travel between networks.

**IP addresses live at Layer 3.** Routers operate at this layer, reading destination IP addresses to decide where to forward packets next.

**Data unit at this layer:** Packet

**Real-world relevance:** IP spoofing attacks operate at Layer 3 — forging the source IP address in a packet header. Firewalls that filter by IP address or subnet are Layer 3 controls. In a SOC alert, the source and destination IPs you're investigating are Layer 3 identifiers.

---

### Layer 2 — Data Link

Handles physical addressing and frame delivery within a single network segment. This is where MAC addresses live.

Two sub-layers:
- **MAC (Media Access Control)** — controls how devices on a shared medium access the network
- **LLC (Logical Link Control)** — handles error checking and flow control

**Data unit at this layer:** Frame

Switches operate at Layer 2 — they read MAC addresses to forward frames to the correct port within a LAN.

**Real-world relevance:** ARP poisoning (from the previous room) is a Layer 2 attack. So is MAC spoofing. Network access controls based on MAC address filtering are Layer 2 controls — and as noted earlier, MAC addresses can be spoofed, making this a fragile trust anchor.

---

### Layer 1 — Physical

The raw bit stream — electrical signals, light pulses, radio waves. Cables, switches' physical ports, NICs, Wi-Fi antennas. No addressing, no logic — just signal transmission.

**Real-world relevance:** Physical security is a Layer 1 concern. An attacker with physical access to network infrastructure can bypass every Layer 2–7 control. Wiretapping, hardware keyloggers, and rogue network taps all operate at Layer 1.

---

### Encapsulation — How Data Moves Down the Stack

As data travels from Layer 7 down to Layer 1 on the sending side, each layer adds its own header (and sometimes trailer) to the data — a process called **encapsulation**. On the receiving side, each layer strips its header as data moves back up — **de-encapsulation**.
```
Layer 7-5:  Data
Layer 4:    Segment (Data + TCP/UDP header)
Layer 3:    Packet  (Segment + IP header)
Layer 2:    Frame   (Packet + MAC header + trailer)
Layer 1:    Bits    (Frame converted to signal)
```

**Why this matters in security:** When you capture network traffic in Wireshark, you're looking at frames (Layer 2). Wireshark then de-encapsulates them to show you the packet (Layer 3), segment (Layer 4), and application data (Layer 7) within. Understanding encapsulation is what makes reading packet captures intuitive rather than confusing.

---

## Walkthrough Notes

### Tasks 1–7 — One Layer Per Task

The room dedicates one task to each layer, working top-down from Application to Physical. Each task is primarily reading-based with a question or two to confirm comprehension. The value isn't in the interactivity — it's in building the habit of thinking in layers.

The question that's worth pausing on: *"What layer does X attack/protocol/tool operate at?"* The room asks this repeatedly in different forms, and it's the right mental habit to develop. Not "what is Layer 4" but "where does nmap operate, and why?"

### Task 8 — Interactive Practical

A drag-and-drop exercise mapping protocols and attacks to their respective layers. Straightforward once the conceptual layer is solid. The exercise reinforces the mapping rather than testing new knowledge.

---

## Layer → Protocol → Attack Mapping

| Layer | Key Protocols | Common Attacks / Controls |
|---|---|---|
| 7 — Application | HTTP, HTTPS, DNS, FTP, SMTP, SSH | SQL injection, XSS, phishing, WAF |
| 6 — Presentation | SSL/TLS, JPEG, ASCII | Weak cipher exploitation, downgrade attacks |
| 5 — Session | NetBIOS, RPC, session tokens | Session hijacking, cookie theft |
| 4 — Transport | TCP, UDP, port numbers | SYN flood, port scanning, firewalls |
| 3 — Network | IP, ICMP, routing protocols | IP spoofing, ICMP flood, ACLs |
| 2 — Data Link | Ethernet, ARP, MAC addresses | ARP poisoning, MAC spoofing, DAI |
| 1 — Physical | Cables, Wi-Fi, NICs | Wiretapping, hardware taps, physical access |

---

## Real-World Mapping

The OSI model's primary value in a SOC context is **layer attribution** — identifying at which layer an attack or anomaly is occurring, because that determines your response.

| Observation | Layer | Implication |
|---|---|---|
| Unusual HTTP POST requests to `/login` | Layer 7 | Brute force or credential stuffing |
| Expired or mismatched TLS certificate | Layer 6 | Possible MitM or misconfiguration |
| Repeated failed session authentications | Layer 5 | Session hijacking attempt |
| High volume of SYN packets, no ACK | Layer 4 | SYN flood DDoS in progress |
| Traffic from spoofed source IPs | Layer 3 | IP spoofing, likely DDoS amplification |
| ARP table changes, new MAC for known IP | Layer 2 | ARP poisoning / MitM on LAN |
| Physical port newly active on switch | Layer 1 | Unauthorised device plugged in |

---

## Takeaways

Three things the OSI model is actually for, beyond the exam question:

1. **It's a diagnostic framework.** When something goes wrong or looks suspicious on a network, the first question is "what layer is this happening at?" The answer narrows your toolset and your response. A Layer 2 ARP issue and a Layer 7 application exploit require completely different investigation approaches.

2. **Attacks respect layers.** Every attack in your future studies will have a layer. SYN floods are Layer 4. SQL injection is Layer 7. ARP poisoning is Layer 2. Understanding the model means you can categorise new attacks immediately rather than treating each one as isolated.

3. **Encapsulation explains packet captures.** Once you understand that a frame contains a packet contains a segment contains application data — reading a Wireshark capture stops being a wall of hex and starts being a structured, readable artefact. That shift in perception is worth the time spent on this room.

---
