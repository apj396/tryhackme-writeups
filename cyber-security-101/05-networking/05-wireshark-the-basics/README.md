# Wireshark: The Basics

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Networking |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/wiresharkthebasics |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the fifth room in the CS101 Networking section. The previous four rooms built the conceptual and protocol foundation — OSI model, TCP/IP, encapsulation, DHCP, ARP, DNS, TLS. This room introduces the primary tool for actually seeing that traffic: Wireshark. It covers the interface layout, how to read and navigate a packet capture file, how to dissect packets layer by layer, how to search within a capture, and how to apply display filters to isolate traffic of interest. The room is hands-on throughout — all tasks after the introduction involve working directly inside Wireshark with provided `.pcapng` files.

---

## Key Concepts

### What Wireshark Is and Is Not

Wireshark is an open-source, cross-platform network packet analyser capable of capturing live traffic and inspecting pre-captured packet capture (PCAP) files. It decodes hundreds of protocols and presents each packet's contents in a structured, layered view that maps directly to the OSI model.

It is important to be clear on what Wireshark does not do: it is not an Intrusion Detection System. It does not alert, block, or take action on traffic. It only allows analysts to observe and investigate packets. Detection logic and alerting live elsewhere — Wireshark is the investigation layer, used after something has already been flagged or to understand protocol behaviour during analysis.

Three primary use cases for Wireshark in a SOC or networking context:

- Detecting and troubleshooting network problems — congestion, load failure, connectivity drops
- Detecting security anomalies — rogue hosts, abnormal port usage, suspicious traffic patterns
- Investigating and learning protocol details — response codes, payload data, handshake behaviour

### The Wireshark Interface

The interface is divided into three main panes, visible when a capture file is open:

| Pane | Purpose |
|---|---|
| Packet List | Top pane — one row per packet, with columns for number, timestamp, source, destination, protocol, length, and info |
| Packet Details | Middle pane — layered breakdown of the selected packet, expandable by OSI layer |
| Packet Bytes | Bottom pane — raw hex and ASCII representation of the selected packet; highlights the bytes corresponding to the selected detail |

The status bar at the bottom shows packet count and capture statistics. The display filter bar sits above the Packet List pane and accepts Wireshark display filter syntax.

### Capture File Properties

The Statistics menu provides metadata about the open capture file via Capture File Properties. This includes: the total number of packets, file size, capture duration, first and last packet timestamps, and the SHA256 hash of the capture file. The capture file comments section — also accessible here — can contain embedded notes or, in CTF and training contexts, embedded flags.

### Packet Dissection

Clicking a packet in the Packet List pane opens its layered breakdown in the Packet Details pane. Packets contain between 5 and 7 layers depending on the protocol. Each layer maps to the OSI model and is individually expandable. Clicking on a specific field in the Packet Details pane highlights the corresponding bytes in the Packet Bytes pane — making explicit the relationship between the human-readable protocol interpretation and the raw data on the wire.

For an HTTP packet, the layers visible in the details pane are: Frame (physical layer metadata), Ethernet II (data link — source and destination MAC), Internet Protocol (network — source and destination IP), Transmission Control Protocol (transport — ports, flags, sequence numbers), and Hypertext Transfer Protocol (application — method, host, path, headers). This is encapsulation made visible.

### Packet Navigation

Wireshark provides several tools for navigating within a large capture file:

- **Go to Packet** (`Ctrl+G`) — jump directly to a packet by number
- **Find Packet** (`Ctrl+F`) — search across packet list, packet details, or packet bytes using a string, hex value, or display filter
- **Mark Packets** — visually tag packets of interest for reference during analysis; marked packets appear highlighted but the marking does not persist after the session
- **Packet Comments** — notes attached to specific packets; these do persist in the capture file and are visible to anyone who opens it
- **Expert Info** — a summary view under Analyse that categorises Wireshark's automatic annotations by severity: Errors, Warnings, Notes, and Chats. The count of Warnings is a useful quick indicator of potential issues in a capture
- **Export Objects** — under File, allows extraction of files transferred over HTTP, SMB, TFTP, IMF, and DICOM protocols directly from the capture. This is the mechanism for recovering files from network captures during incident investigation

### Packet Filtering

Wireshark has two distinct filtering mechanisms with different purposes:

**Capture filters** are applied during live capture — they limit which packets are recorded to the capture file in the first place. They use Berkeley Packet Filter (BPF) syntax. The room notes an important caution: misusing capture filters can cause relevant packets to be missed entirely. For investigation work, the recommended approach is to capture without filters and apply display filters afterwards.

**Display filters** are applied to an already-captured file and control which packets are shown — they do not discard packets, only hide them. Display filters use Wireshark's own filter syntax and support a wide range of protocol fields, comparison operators, and logical operators. Common examples:

    ip                        — show only IP packets
    http                      — show only HTTP packets
    tcp.port == 80            — show traffic on port 80
    ip.addr == 192.168.1.1    — show traffic to or from a specific IP
    dns                       — show only DNS queries and responses

Right-clicking on any field in the Packet Details pane offers an option to apply or prepare a filter based on that field's value — a practical shortcut during analysis.

---

## Walkthrough Notes

The room provides two `.pcapng` files: `http1.pcapng` is used to simulate the screenshots in the room's instructional text, and `Exercise.pcapng` is the file used to answer the task questions. Both are available in the attached VM.

**Task 1 (Introduction):** Sets out the room's scope — tool overview, packet dissection, packet navigation, and packet filtering — and introduces the two practice files. No questions beyond confirming which file is used for screenshots (`http1.pcapng`) and which for questions (`Exercise.pcapng`).

**Task 2 (Tool Overview):** Covers the three-pane interface layout and the Statistics menu. Using `Exercise.pcapng`, navigating to Statistics → Capture File Properties reveals the capture file comments (containing the flag for the first question), the total packet count (visible in the status bar or the properties window), and the SHA256 hash of the file. These three questions establish familiarity with the metadata layer before diving into individual packets.

**Task 3 (Packet Dissection):** Using `Exercise.pcapng`, navigate to packet 38 using Go to Packet (`Ctrl+G`). Expanding the HTTP layer in the Packet Details pane reveals that the markup language used is eXtensible Markup Language (XML). The arrival date of the packet is visible in the Frame layer at the top of the details. This task builds the habit of reading the packet details pane layer by layer.

**Task 4 (Packet Navigation):** Covers Find Packet, Go to Packet, Mark Packets, Packet Comments, Expert Info, and Export Objects — all via `Exercise.pcapng`. Key questions in this task: searching for the string `r4w` in packet details reveals artist names embedded in the payload; packet 12's comment is visible by right-clicking and selecting Packet Comment; the `.txt` file embedded in the capture is recovered via File → Export Objects → HTTP and contains an alien name in the file contents; the Expert Info warning count is found under Analyse → Expert Info, filtering by Warnings.

**Task 5 (Packet Filtering):** Covers the distinction between capture filters and display filters, and the display filter syntax. Navigate to packet 4 using the filter bar and Go to Packet to answer the final practical question. The task reinforces that display filters are non-destructive — they hide, not delete — and that right-clicking on packet fields offers filter shortcuts.

**Task 6 (Conclusion):** Summarises the room — tool overview, packet dissection, navigation, and filtering — and points to further Wireshark rooms. No answer required.

---

## Commands and Filters Used

    Statistics → Capture File Properties    # file metadata, comments, hash, packet count
    Ctrl+G                                  # Go to Packet by number
    Ctrl+F                                  # Find Packet by string, hex, or filter
    Analyse → Expert Info                   # Categorised annotation summary
    File → Export Objects → HTTP            # Extract transferred files from capture

Display filter examples used in the room:

    ip
    http
    tcp.port == 80

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| Capture file properties and SHA256 hash | Evidence integrity — hashing a PCAP file before analysis and recording the hash establishes chain of custody; any subsequent modification is detectable |
| Packet dissection by OSI layer | Protocol anomaly investigation — unexpected fields, malformed headers, or unusual flag combinations are identified by navigating the packet details layer by layer |
| Expert Info warnings | Triage acceleration — high warning counts in Expert Info for a capture indicate retransmissions, connection resets, or malformed packets; useful as a first-pass health check before deep analysis |
| Export Objects (HTTP) | File extraction during IR — recovering malware samples, exfiltrated documents, or dropped payloads from a capture without needing a full sandbox environment |
| Display filters by protocol and IP | Alert investigation workflow — when a SIEM alert fires on a source IP or protocol, applying a Wireshark display filter to a relevant capture immediately scopes the visible traffic to the relevant conversation |
| Packet comments | Collaborative analysis — analysts can annotate packets of interest in a shared capture file, preserving investigation notes alongside the raw evidence |

---

## Takeaways

1. **Wireshark makes the abstract concrete — every concept from the previous four rooms becomes directly visible inside a capture file.** The three-way handshake is not just a diagram; it is three packets with SYN, SYN-ACK, and ACK flags in the TCP layer of the Packet Details pane. Encapsulation is not just a theory; it is the nested layers expanding in the details view when you click on a packet. The tool and the concepts are inseparable — using one properly requires understanding the other.

2. **The distinction between capture filters and display filters is operationally significant and worth getting right from the start.** Applying an incorrect capture filter during live traffic recording can silently exclude relevant packets — there is no warning that important traffic was missed. Display filters carry no such risk. The safe habit is: capture everything, filter the view. Reserve capture filters for situations where storage or bandwidth makes promiscuous capture genuinely impractical.

3. **Export Objects is one of the most underappreciated features in Wireshark for defensive analysis.** The ability to reconstruct and extract files transmitted over HTTP, SMB, or other protocols directly from a PCAP means that identifying what data left (or entered) a network during an incident does not require re-executing the attack or having access to the endpoint. The file is already in the capture — Wireshark can pull it out.

---
