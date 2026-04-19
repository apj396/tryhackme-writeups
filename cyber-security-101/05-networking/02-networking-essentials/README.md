# Networking Essentials

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Networking |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/networkingessentials |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the second room in the CS101 Networking series. Where Networking Concepts established the theoretical framework — OSI, TCP/IP, encapsulation — this room zooms in on the protocols that make a network functional in practice. The question it answers is deceptively simple: how does a device join a network, find other devices on it, test whether they are reachable, and send packets beyond its own segment? The answer involves four protocols most users never consciously interact with — DHCP, ARP, ICMP, and the routing infrastructure underneath all of them — plus a practical discussion of NAT, which is why the internet did not run out of addresses a decade ago.

---

## Key Concepts

### DHCP — Dynamic Host Configuration Protocol

When a device connects to a network, it needs at minimum three things: an IP address, a subnet mask, and a default gateway. Configuring these manually on every device is feasible for servers — which are static and expected to be at predictable addresses — but impractical for user devices that move between networks constantly. DHCP automates this process.

The exchange follows a four-step sequence, sometimes called DORA:

| Step | Direction | Purpose |
|---|---|---|
| Discover | Client → Broadcast | Client announces itself, requests configuration |
| Offer | Server → Client | Server proposes an IP address and configuration |
| Request | Client → Broadcast | Client formally requests the offered address |
| ACK | Server → Client | Server confirms the lease |

The Discover and Request messages are sent to the broadcast address because the client does not yet have an IP — it cannot send a directed packet to a specific server. Once the ACK is received, the client has a leased IP address valid for a defined period. When that lease expires or the device leaves the network, the address returns to the pool.

In a packet capture, DHCP traffic is immediately recognisable: source address `0.0.0.0`, destination `255.255.255.255`. Anomalous DHCP behaviour — rogue DHCP servers, DHCP exhaustion attacks — is a detection category in its own right.

### ARP — Address Resolution Protocol

IP addresses identify devices logically. MAC addresses identify them physically at the data link layer. For two devices on the same network segment to actually exchange Ethernet frames, the sender needs to know the MAC address of the destination — and ARP is how it finds out.

The exchange is straightforward: the requesting device broadcasts an ARP Request to `ff:ff:ff:ff:ff:ff` asking "who has IP `x.x.x.x`? Tell `y.y.y.y`". Every device on the segment receives this. The device that owns the requested IP responds directly with an ARP Reply containing its MAC address. The requester caches this mapping in its ARP table for future frames.

    tshark -r arp.pcapng -Nn

A capture showing ARP traffic reveals this exchange clearly — one broadcast from the requester, one unicast reply from the target. ARP sits at an interesting boundary: it deals with MAC addresses (Layer 2) but exists to support IP operations (Layer 3). The practical answer is that it bridges both, and that ambiguity is why it is a favourite for certain attacks. ARP has no authentication — an attacker can send unsolicited ARP Replies claiming any IP maps to their MAC, poisoning the ARP caches of other devices on the segment. ARP poisoning is the basis for man-in-the-middle attacks on local networks.

### ICMP — Internet Control Message Protocol

ICMP is the network's diagnostic and error-reporting layer. It does not carry application data — it carries information *about* the network. Two tools that every analyst uses daily are built on ICMP.

**Ping** sends an ICMP Echo Request (Type 8) and expects an ICMP Echo Reply (Type 0) in return. The round-trip time and packet loss statistics tell you whether a host is reachable and how the path is performing. A host that does not respond to ping is not necessarily down — firewalls commonly block ICMP — but unexpected ping responses or the absence of expected ones are both detection-relevant signals.

**Traceroute** exploits the TTL (Time To Live) field in the IP header. Every router that forwards a packet decrements the TTL by one. When TTL reaches zero, the router drops the packet and sends an ICMP Time Exceeded message back to the sender. Traceroute sends packets with incrementally increasing TTL values — starting at 1 — so each successive router along the path is forced to reveal itself. The result is a map of the hops between source and destination, with latency at each hop.

### Routing

Routing is the process of deciding which path a packet should take to reach its destination. Routers maintain routing tables — lists of known networks and the next hop toward each. When a packet arrives, the router consults its table and forwards accordingly.

Two categories of routing are relevant here. Static routing involves manually configured routes — reliable, predictable, but operationally expensive in large networks. Dynamic routing uses protocols that allow routers to share information with each other and update their tables automatically. The room covers OSPF (Open Shortest Path First), a link-state protocol where each router builds a complete map of the network topology and calculates the shortest path to every destination. OSPF is widely deployed in enterprise networks. EIGRP (Enhanced Interior Gateway Routing Protocol) is also mentioned — it is a Cisco proprietary protocol.

### NAT — Network Address Translation

IPv4's address space tops out at approximately 4.3 billion addresses — exhausted in practice well before the number of internet-connected devices reached that figure. NAT is one of the primary mechanisms that has kept IPv4 functional despite this constraint.

NAT allows a router to represent an entire private network behind a single public IP address. Devices inside the network use private address ranges (such as `192.168.x.x` or `10.x.x.x`) that are not routable on the public internet. When a device inside the network initiates an outbound connection, the router replaces the private source IP with its own public IP and records the mapping in a translation table. When the response arrives, the router reverses the translation and delivers the packet to the correct internal device.

The implication for port numbers is significant: since many internal devices share one public IP, the router uses port numbers to distinguish between simultaneous connections. This is why a router with standard hardware can theoretically maintain around 65,000 simultaneous TCP connections — one per port number.

---

## Walkthrough Notes

The room runs through seven tasks. Tasks 2 through 6 cover DHCP, ARP, ICMP, routing, and NAT respectively, each with inline comprehension questions. Task 7 is a practical exercise accessed via the "View Site" button — an interactive simulation that tests understanding of the protocols covered.

**Task 2 (DHCP):** The key question is which protocol to use when you want to automatically obtain an IP address, subnet mask, and default gateway — the answer is DHCP. Understanding that the Discover and Request steps use broadcast because the client has no IP yet is the conceptual anchor.

**Task 3 (ARP):** The broadcast MAC address used in an ARP Request is `ff:ff:ff:ff:ff:ff`. The packet captures shown in the room use both `tshark` and `tcpdump` to display the same ARP exchange — worth noting that the two tools format output differently but show identical underlying data.

**Task 4 (ICMP):** Traceroute's dependence on the TTL field is the key concept. The IP header field that traceroute requires to reach zero is TTL — each router decrements it, and the ICMP Time Exceeded response at zero is what reveals the hop.

**Task 5 (Routing):** The Cisco proprietary routing protocol discussed is EIGRP. OSPF is the open standard equivalent.

**Task 6 (NAT):** The practical question on NAT asks what public IP a device will appear to use — the answer is the router's public IP, which all internal devices share for outbound traffic. The 65,000 simultaneous TCP connection figure comes directly from the port number space.

**Task 7 (Practical):** The interactive site walks through a simulated network scenario. Following the on-screen instructions and applying the DHCP, ARP, and ICMP concepts from the room yields the flag.

---

## Commands Used

    tshark -r arp.pcapng -Nn
    tshark -r DHCP-G5000.pcap -n
    tcpdump -r arp.pcapng -n -v
    ping <target-ip> -c 4
    traceroute <target>

---

## SOC / Real-World Mapping

| Concept | Real-World Application |
|---|---|
| DHCP DORA sequence | Rogue DHCP server detection — unexpected DHCP Offer packets from an unknown source on the segment indicate a potential MitM setup |
| ARP broadcast and reply | ARP poisoning detection — gratuitous ARP replies (unsolicited, mapping a known IP to a new MAC) are a primary indicator of ARP spoofing in IDS rules |
| ICMP Echo Request/Reply | Host discovery in reconnaissance — ping sweeps across a subnet are a standard first step in network enumeration; blocking ICMP at the perimeter limits attacker visibility |
| TTL field in IP header | TTL manipulation detection — unusually low TTL values on inbound packets can indicate spoofing or traceroute-based reconnaissance |
| NAT translation table | Source attribution in investigations — traffic logs showing the router's public IP must be correlated with NAT logs to identify the actual internal source device |
| Routing protocols (OSPF) | Route injection attacks — a compromised router advertising false OSPF routes can redirect traffic through an attacker-controlled path; detecting unexpected route changes is a network monitoring use case |

---

## Takeaways

1. **The protocols in this room are invisible in normal operation — which is exactly what makes anomalies in them significant.** Users never see a DHCP exchange or an ARP request. When these protocols behave unexpectedly — a rogue DHCP server, an unsolicited ARP reply, an ICMP flood — it is because something deliberate is happening. Knowing the normal baseline is what makes the anomaly detectable.

2. **NAT is an architectural constraint with direct forensic implications.** A public IP address in a log does not identify a device — it identifies a router. Everything behind that router shares the address. Tracing an incident to a specific internal host requires the NAT translation logs from the router at the relevant timestamp. This is a practical reality in almost every corporate network investigation.

3. **ARP's complete absence of authentication is a deliberate design trade-off that has never been fixed at the protocol level.** The only defences against ARP poisoning are network-level controls — dynamic ARP inspection on managed switches, static ARP entries for critical hosts, and detection via monitoring tools. Understanding *why* ARP is vulnerable (no sender verification, cache updates accepted from any device) is the foundation for understanding why those controls matter.

---
