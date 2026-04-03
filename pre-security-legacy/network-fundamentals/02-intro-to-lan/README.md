# Intro to LAN

**Module:** Pre-Security (Legacy) → Network Fundamentals  
**Difficulty:** Easy  
**Platform:** TryHackMe  
**Room Link:** https://tryhackme.com/room/introtolan  
**Date Completed:** January 2026  
**Author:** Adwait Joshi

---

## What This Room Covers

This room moves from "what is a network" to "how is a network actually structured." It covers the physical and logical design of local networks — topologies, subnetting, and the two protocols (ARP and DHCP) that keep devices identified and addressed without manual intervention.

If the previous room answered *what* a network is, this one answers *how* it holds itself together.

---

## Key Concepts

### 1. LAN Topologies

A topology is the physical or logical layout of how devices are connected on a network. Three are covered here:

**Star Topology**
All devices connect individually to a central switch or hub. Most common in modern networks because it's reliable and scalable — adding a new device means one new cable to the switch, nothing else changes. The tradeoff: if the central switch fails, the entire network goes down. Also the most expensive to set up due to cabling and dedicated hardware.

**Bus Topology**
All devices share a single backbone cable. Simple and cheap to set up — but a single cable break takes everything offline, and performance degrades as more devices are added (all traffic competes on the same wire). Largely obsolete today.

**Ring Topology**
Devices are connected in a loop, with data travelling in one direction around the ring until it reaches its destination. Predictable performance, but a single break disrupts the entire loop unless redundancy is built in.

**Switches vs Routers — the distinction that matters:**
- A **switch** connects devices *within* a network (LAN). It uses MAC addresses to forward traffic only to the intended device — not to everyone, like a hub would.
- A **router** connects *different* networks together. It routes traffic between a LAN and the internet, or between two separate LANs.

**Real-world relevance:** In a SOC context, understanding topology is the prerequisite for understanding blast radius. If an attacker compromises a switch in a star topology, they're positioned to intercept traffic from every device on that segment. Topology = attack surface shape.

---

### 2. Subnetting

Subnetting is the practice of dividing a larger network into smaller, logically isolated sub-networks. The mechanism is the **subnet mask** — a 32-bit number (four octets, 0–255) that defines which part of an IP address identifies the network and which part identifies the host.

Three address types within any subnet:

| Address Type | Purpose | Example |
|---|---|---|
| Network Address | Identifies the subnet itself | `192.168.1.0` |
| Host Address | Identifies individual devices | `192.168.1.25` |
| Default Gateway | The exit point to other networks | `192.168.1.1` or `192.168.1.254` |

**Why subnetting matters beyond efficiency:**

The security angle is often underemphasised at this level. Subnetting creates isolation boundaries. A café separating its staff network from its guest Wi-Fi is subnetting for security — a compromised guest device cannot directly reach the point-of-sale system on a different subnet. The same logic applies at enterprise scale: finance, HR, and engineering on separate subnets means lateral movement requires crossing a network boundary, which is detectable.

**Real-world relevance:** Subnetting is foundational to network segmentation — a core defensive concept. When reviewing an alert, knowing whether two communicating IPs are on the same subnet or different ones changes the interpretation entirely.

---

### 3. ARP — Address Resolution Protocol

ARP solves a specific problem: IP addresses tell you *where* to send data logically, but actual transmission on a local network happens via MAC addresses. ARP is the translation layer between the two.

**How it works:**

1. Device A wants to communicate with `192.168.1.20` but doesn't know its MAC address
2. Device A broadcasts an **ARP Request** to the entire network: *"Who has `192.168.1.20`? Tell me your MAC."*
3. The device with that IP responds with an **ARP Reply**: *"That's me — here's my MAC address."*
4. Device A stores this mapping in its **ARP cache** for future use
```bash
# View ARP cache on Linux
arp -a

# View ARP cache on Windows
arp -a
```

**The security implication — ARP Poisoning:**
ARP has no authentication. Any device can send an unsolicited ARP Reply claiming to be any IP address. This is the basis of ARP poisoning (also called ARP spoofing) — an attacker sends false ARP replies to redirect traffic through their machine, enabling a man-in-the-middle position on the local network.

**Real-world relevance:** ARP poisoning is a classic LAN-based attack. In a SOC context, unexpected changes in ARP tables — especially mapping a known IP to a new MAC — are an indicator of compromise worth investigating.

---

### 4. DHCP — Dynamic Host Configuration Protocol

DHCP automates IP address assignment. Without it, every device on every network would need a manually configured, unique IP address — unmanageable at scale.

**The four-step handshake (DORA):**

| Step | Packet | Direction | Purpose |
|---|---|---|---|
| 1 | **DHCP Discover** | Client → Broadcast | *"Is there a DHCP server? I need an IP."* |
| 2 | **DHCP Offer** | Server → Client | *"I have one available — here it is."* |
| 3 | **DHCP Request** | Client → Server | *"I'll take that IP, please confirm."* |
| 4 | **DHCP ACK** | Server → Client | *"Confirmed. It's yours for the lease period."* |

IP addresses assigned by DHCP are leased, not permanent. When the lease expires, the device must renew or be assigned a new address. This is why the same device can have different IPs on different days on a home network.

**Real-world relevance:** DHCP logs are gold in an investigation. They map IP addresses to MAC addresses with timestamps — letting you answer "what device had this IP at this time?" even on dynamic networks. Rogue DHCP servers (an attacker's machine responding to Discover packets first) are also a known attack vector for traffic redirection.

---

## Walkthrough Notes

### Task 1 — LAN Topologies
The interactive lab here lets you physically break each topology to observe failure modes. The star topology failing when the switch is taken out, versus the ring topology failing at a single cable break, makes the theoretical tradeoffs concrete. The flag `THM{TOPOLOGY_FLAWS}` is earned by completing the interactive exercise.

### Task 2 — Subnetting
No hands-on lab here, but the conceptual framing — network address, host address, default gateway — is worth internalising precisely because it comes up constantly. The subnet mask as a 32-bit number across four octets (0–255) is the same structure as an IP address, which makes reading them intuitive with practice.

### Task 3 — ARP Protocol
The request/reply cycle is straightforward, but the cache behaviour is worth noting — devices don't broadcast ARP for every communication, only when the mapping isn't already in cache. This is exactly what ARP poisoning exploits: poisoning the cache means the device won't re-request.

### Task 4 — DHCP Protocol
DORA is one of those acronyms worth memorising (Discover, Offer, Request, ACK). In packet captures, these four packets have a very recognisable pattern — broadcast source, specific destination ports (67/68 UDP). Recognising them in Wireshark is a practical skill that comes up quickly.

---

## Commands Referenced
```bash
# View ARP table — works on both Linux and Windows
arp -a

# Release and renew DHCP lease on Linux
sudo dhclient -r        # release
sudo dhclient           # request new lease

# Release and renew DHCP lease on Windows
ipconfig /release
ipconfig /renew

# View current IP configuration (includes subnet mask and gateway)
ipconfig              # Windows
ip addr               # Linux
```

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| Star Topology | Most enterprise networks — understanding single points of failure aids incident scoping |
| Subnetting | Network segmentation for containment; lateral movement detection across subnet boundaries |
| ARP Cache | First place to look for MAC address conflicts or spoofing indicators on a LAN |
| ARP Poisoning | MitM attack vector on local networks; detectable via unexpected ARP table changes |
| DHCP Logs | IP-to-MAC-to-timestamp mapping — essential for attribution during investigations |
| Rogue DHCP | Attacker-controlled DHCP server redirecting traffic; detected via unauthorised DHCP offers |
| DORA Handshake | Recognisable in packet captures (UDP 67/68); baseline behaviour for anomaly detection |

---

## Takeaways

Three things this room makes clear that stay relevant well beyond the basics:

1. **Topology defines blast radius.** The way a network is physically laid out determines how far an attacker can move and what they can see from any given position. Knowing topology isn't just infrastructure knowledge — it's threat modelling.

2. **ARP's lack of authentication is a feature turned vulnerability.** The protocol was designed for trusted local networks. The moment you put adversarial actors on the same LAN segment, ARP's stateless, unauthenticated design becomes an attack surface. Dynamic ARP Inspection (DAI) on managed switches exists specifically to address this.

3. **DHCP logs are underrated in investigations.** They're often overlooked in favour of firewall or IDS logs, but for answering "who was using this IP at this time," DHCP lease logs are frequently the most direct source of truth.

---
