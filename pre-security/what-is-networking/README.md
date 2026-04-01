# What is Networking?

**Module:** Pre-Security → Network Fundamentals  
**Difficulty:** Easy  
**Platform:** TryHackMe  
**Room Link:** https://tryhackme.com/room/whatisnetworking  
**Date Completed:** February 2026  
**Author:** Adwait Joshi

---

## What This Room Covers

This room answers the foundational question that underpins all of cybersecurity: what actually *is* a network, and how do devices find and talk to each other? It covers the concept of networks, IP addressing, MAC addressing, and introduces `ping` as a basic diagnostic tool.

Simple on the surface. Load-bearing in practice.

---

## Key Concepts

### 1. What is a Network?
A network is simply a collection of connected devices that can communicate with each other. That connection can be wired or wireless, local or global. The internet itself is just a very large network of networks.

The key insight here: every security concept — firewalls, intrusion detection, packet analysis — only makes sense *in the context of a network*. You can't defend what you don't understand structurally.

### 2. IP Addresses
An IP (Internet Protocol) address is a logical identifier assigned to a device on a network. It's how data knows *where to go*.

- **IPv4** format: `192.168.1.1` — four octets, each ranging from 0–255
- **IPv6** was introduced to solve IPv4 exhaustion (~4.3 billion addresses wasn't enough)
- IP addresses can change (dynamic) or stay fixed (static)

**Real-world relevance:** In a SOC context, IP addresses are the first thing you look at in an alert. Is this IP internal or external? Is it known-malicious? Has it communicated before? The whole mental model starts here.

### 3. MAC Addresses
A MAC (Media Access Control) address is a *physical* identifier hardcoded into a device's network interface card (NIC). Unlike IP addresses, MACs don't change (under normal circumstances).

- Format: `a4:c3:f0:85:ac:2d` — six pairs of hexadecimal values
- The first three pairs identify the *manufacturer*, the last three are device-specific
- MAC addresses operate at Layer 2 (Data Link) of the OSI model

**The nuance worth noting:** MAC addresses can be *spoofed*. An attacker can change their device's MAC to impersonate another device on a local network — a technique relevant to ARP poisoning and network-based attacks. "Hardcoded" doesn't mean "immutable in practice."

> For a deeper look at how MAC and IP addressing fit into the broader communication model, see my [OSI Model writeup on Medium](https://adwaitjoshi1.medium.com).

### 4. Ping (ICMP)
`ping` is a command-line tool that uses ICMP (Internet Control Message Protocol) to test whether a device is reachable on a network.
```bash
ping 192.168.1.1
ping tryhackme.com
```

It sends an ICMP Echo Request and waits for an Echo Reply. The round-trip time (RTT) tells you not just *if* the host is up, but how far away it effectively is in network terms.

**What ping actually tells you in an investigation:**
- Host is alive and reachable → continue enumeration
- No response → host is down, blocking ICMP, or behind a firewall
- High RTT or packet loss → potential network issue or rate-limiting

ICMP is also the protocol behind `traceroute` — the tool that maps the path packets take across a network, hop by hop.

---

## Walkthrough Notes

### Task 1 & 2 — What is Networking + What is the Internet?
These tasks establish the mental model: devices form networks, networks connect to form the internet. The interesting framing here is thinking of the internet not as a cloud or an abstraction, but as a *physical infrastructure* — cables, routers, data centers — that someone has to secure.

### Task 3 — Identifying Devices on a Network
This is where IP and MAC addressing are introduced side by side. The distinction that matters: IP is *logical and changeable* (used for routing across networks), MAC is *physical and local* (used for communication within a network segment).

Think of it this way — an IP address is like your current mailing address. A MAC address is like your fingerprint. One tells the network where to send data. The other identifies the hardware sending it.

### Task 4 — Ping
Executing `ping` in the AttackBox to verify connectivity. The task is simple, but the underlying concept is foundational — you're sending a packet and waiting for a response. That stimulus-response model is how most network diagnostic thinking works.

---

## Commands Used
```bash
# Basic ping to an IP
ping 192.168.1.1

# Ping a domain (tests DNS resolution too)
ping tryhackme.com

# Limit ping to 4 packets (Windows default behavior)
ping -c 4 192.168.1.1
```

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| IP Addressing | First data point in any alert triage — internal vs. external, geolocation, reputation |
| MAC Addressing | Used in DHCP logs to track device identity on a LAN; relevant to rogue device detection |
| MAC Spoofing | Evasion technique in network-based attacks; flagged in Layer 2 security monitoring |
| Ping / ICMP | Used in host discovery during recon; often blocked by defenders — absence of response ≠ absence of host |

---

## Takeaways

Three things this room quietly establishes that matter far more than the tasks suggest:

1. **Every alert has an IP address attached to it.** Understanding addressing isn't academic — it's the first filter in every investigation workflow.

2. **"Hardcoded" is a relative term in security.** MAC addresses feel permanent until you realize they can be spoofed in seconds. Security models that rely on physical identifiers as trust anchors are fragile.

3. **Ping is both a tool and a concept.** The underlying ICMP model — send a packet, expect a response, measure the result — is the mental template for dozens of diagnostic and recon techniques you'll use later.

---
