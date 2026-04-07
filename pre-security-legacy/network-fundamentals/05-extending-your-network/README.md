# Extending Your Network

**Module:** Pre-Security (Legacy) → Network Fundamentals  
**Difficulty:** Easy  
**Platform:** TryHackMe  
**Room Link:** https://tryhackme.com/room/extendingyournetwork  
**Date Completed:** January 2026  
**Author:** Adwait Joshi

---

## What This Room Covers

This room covers the technologies that extend networks beyond their local boundaries — port forwarding, firewalls, VPNs, and the network devices (routers, switches) that make it all work. It's the bridge between understanding what a LAN is and understanding how LANs connect to the wider internet securely.

The previous four rooms built the foundation. This room is where that foundation meets the real world.

---

## Key Concepts

### 1. Port Forwarding

By default, devices on a private network are not reachable from the internet. Port forwarding changes that — it instructs a router to redirect incoming traffic on a specific port to a specific internal device.

**How it works:**

```
Internet → Router (Public IP: 82.34.12.1) → Internal Server (192.168.1.10)

External request to 82.34.12.1:80
       ↓
Router rule: forward port 80 → 192.168.1.10:80
       ↓
Internal web server receives the request
```

Port forwarding is configured on the router, not the destination device. The internal device simply listens on its port — it has no awareness of the forwarding happening upstream.

**Common legitimate uses:**
- Hosting a web server at home
- Remote access to a NAS or security camera
- Game server hosting

**Real-world relevance:** Port forwarding is one of the most common misconfigurations found in small business and home networks. An exposed RDP port (3389) via port forwarding is one of the primary ransomware entry points documented in incident reports. From an attacker's perspective, scanning for open forwarded ports is basic external reconnaissance. From a defender's perspective, knowing which ports are forwarded on a perimeter router is fundamental network hygiene.

---

### 2. Firewalls

A firewall is a network security device — hardware, software, or both — that monitors and controls incoming and outgoing network traffic based on a defined set of rules.

**Two primary types covered in this room:**

**Stateless Firewall**
Evaluates each packet in isolation against a static ruleset. Fast, but limited — it has no memory of previous packets. A stateless firewall can't distinguish between a legitimate response to an outbound request and unsolicited inbound traffic with the same characteristics.

**Stateful Firewall**
Tracks the state of active connections. It knows whether an inbound packet is part of an established, legitimate session or an unsolicited connection attempt. Far more effective at blocking attacks that exploit the stateless model.

**Firewall rule anatomy:**

| Field | Description |
|---|---|
| Source IP | Where the traffic originates |
| Destination IP | Where the traffic is going |
| Port | Which port the traffic targets |
| Protocol | TCP, UDP, ICMP |
| Action | Allow or Deny |

Rules are evaluated top-to-bottom. The first matching rule wins — order matters significantly.

**Example ruleset logic:**

```
Rule 1: Allow TCP from 192.168.1.0/24 to ANY on port 443  → ALLOW
Rule 2: Allow TCP from ANY to 192.168.1.10 on port 80     → ALLOW
Rule 3: Deny ALL                                           → DENY
```

Anything not explicitly allowed by Rule 1 or 2 hits Rule 3 and is dropped. This is the principle of **default deny** — the most secure baseline posture.

**Real-world relevance:** Misconfigured firewall rules are among the most common findings in penetration tests and security audits. Overly permissive rules ("allow any to any"), rules in the wrong order, or missing default-deny at the end of a ruleset are all recurring issues. In a SOC context, firewall logs are a primary data source — blocked connection attempts often signal reconnaissance or exploitation attempts.

---

### 3. VPNs — Virtual Private Networks

A VPN creates an encrypted tunnel between two endpoints over an untrusted network (typically the internet), allowing devices to communicate as if they were on the same private network.

**How it works:**

```
Device A (Home)  ←——— Encrypted Tunnel ———→  VPN Server  ←——→  Private Network
192.168.1.50                                  10.0.0.1          10.0.0.0/24
```

Device A gets assigned a VPN IP address and can reach resources on the private network as if physically present. All traffic between Device A and the VPN server is encrypted in transit.

**Three VPN technologies mentioned in this room:**

| Technology | Description |
|---|---|
| PPP | Point-to-Point Protocol — handles authentication and encryption within a VPN tunnel |
| PPTP | Point-to-Point Tunneling Protocol — encapsulates PPP packets for transmission; older, largely deprecated due to weak encryption |
| IPSec | Internet Protocol Security — encrypts IP packets directly; stronger and widely used in enterprise VPNs |

**Two VPN architectures:**

**Remote Access VPN** — individual users connect to a corporate network from remote locations. The standard work-from-home setup.

**Site-to-Site VPN** — connects two entire networks (e.g., two office locations) through a persistent encrypted tunnel. Traffic between the sites flows securely without individual users needing to authenticate to the VPN separately.

**Real-world relevance:** VPN infrastructure is a high-value target. Exploiting vulnerabilities in VPN concentrators (Pulse Secure, Fortinet, Citrix — all have had critical CVEs) gives an attacker direct access to an internal network without needing to breach the perimeter otherwise. Monitoring VPN authentication logs — unusual login times, impossible travel, new device types — is a core SOC activity.

---

### 4. Network Devices — Routers and Switches Revisited

This room revisits routers and switches with more depth than the earlier topology-focused view.

**Router**
Operates at Layer 3. Connects different networks and determines the optimal path for packets to travel between them. Maintains a **routing table** — a map of known networks and the paths to reach them.

Routing can be:
- **Static** — manually configured routes, predictable but not adaptive
- **Dynamic** — routes learned automatically via protocols like OSPF or BGP, adaptive but more complex

**Switch**
Operates at Layer 2. Connects devices within the same network. Maintains a **MAC address table** (also called a CAM table) — mapping which MAC address is reachable via which physical port.

When a frame arrives, the switch checks its MAC table:
- MAC known → forward only to the correct port
- MAC unknown → flood to all ports, then learn from the response

**The security implication of flooding:** A **MAC flood attack** overwhelms a switch's CAM table with fake MAC addresses until it runs out of memory. The switch then falls back to flooding all traffic to all ports — effectively turning it into a hub, making all traffic visible to any device on the segment. This is a precursor to sniffing attacks on switched networks.

---

## Walkthrough Notes

### Task 1 — Introduction to Port Forwarding
The interactive element here shows traffic flow from an external IP through a router to an internal server. The key insight: the public IP is the router's, not the server's. The server is entirely hidden behind it until port forwarding exposes a specific service.

### Task 2 — Firewalls 101
The rule-ordering exercise is the practical centrepiece. Understanding that rules are evaluated sequentially and the first match wins is foundational for both configuring firewalls and for understanding why misconfigured ones fail. The default-deny rule at the end of a ruleset is the safety net — without it, unmatched traffic is permitted by default.

### Task 3 — VPN Basics
The distinction between remote access and site-to-site VPNs maps directly to real enterprise environments. Remote access is what most people think of as "the VPN" — site-to-site is what quietly connects office locations without user involvement.

### Task 4 — LAN Networking Devices
The MAC address table behaviour — forward when known, flood when unknown, learn from responses — explains both normal switch operation and the mechanism behind MAC flooding attacks.

---

## Commands Referenced

```bash
# View routing table on Linux
ip route
route -n

# View routing table on Windows
route print

# View MAC address table on a Cisco switch (privileged mode)
show mac address-table

# Test port forwarding / connectivity to a specific port
nc -zv <public-ip> <port>
nmap -p <port> <public-ip>

# Check active firewall rules on Linux (iptables)
sudo iptables -L -v -n

# Check firewall status on Windows
netsh advfirewall show allprofiles
```

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| Port forwarding | External attack surface; exposed RDP/SMB via forwarding = critical finding |
| Stateless vs stateful firewall | Stateless rules bypassable via fragmentation and spoofing; stateful is baseline expectation |
| Default deny | Gold standard firewall posture; absence = implicit allow = risk |
| Firewall logs | Primary source for blocked connection attempts; key in perimeter monitoring |
| VPN authentication logs | Impossible travel, off-hours logins, new devices = indicators of credential compromise |
| VPN CVEs | High-value targets; Pulse Secure, Fortinet, Citrix all had critical exploited vulnerabilities |
| MAC flood attack | CAM table exhaustion turns switch into hub; enables passive sniffing on switched networks |
| Routing tables | Understanding routing is prerequisite for detecting traffic redirection attacks |

---

## Takeaways

Three things this room establishes that stay relevant at every level of security work:

1. **Default deny is a mindset, not just a firewall rule.** The principle — block everything unless explicitly permitted — applies to firewall configuration, network access control, cloud IAM policies, and application authorisation. Whenever you see "allow all" as a baseline with exceptions carved out, that's a weaker posture than "deny all" with exceptions allowed in.

2. **VPNs are trusted pathways, which makes them targets.** A VPN isn't just a privacy tool — in an enterprise context, it's the front door to the internal network. Attackers who compromise VPN credentials or exploit VPN software vulnerabilities bypass almost all perimeter controls in one step. VPN infrastructure deserves the same scrutiny as any other critical system.

3. **Understanding normal device behaviour is how you detect abnormal behaviour.** Knowing how a switch builds its MAC table, how a router consults its routing table, how a firewall evaluates rules — all of this creates a mental baseline. Anomalies (unexpected routes, MAC table flooding, firewall rule changes) only register as suspicious when you know what normal looks like.

---
