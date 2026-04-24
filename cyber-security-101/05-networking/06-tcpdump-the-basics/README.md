# Tcpdump: The Basics

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Networking |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/tcpdump |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the sixth room in the CS101 Networking section. Where Wireshark: The Basics covered packet analysis through a GUI, this room introduces tcpdump — the command-line equivalent. tcpdump has no graphical interface. It captures and displays packets directly in the terminal, reads from saved capture files, and accepts filters using Berkeley Packet Filter syntax. The room covers basic capture, reading from files, filtering expressions, advanced filtering, and controlling how output is displayed. Because tcpdump runs entirely from the terminal, it is available in environments where a GUI is not — remote servers, headless systems, and scripted automation pipelines — making it a tool analysts encounter in real deployments far more frequently than Wireshark.

---

## Key Concepts

### What tcpdump Is

tcpdump is an open-source, command-line packet capture and analysis tool available on virtually every Unix-based system. It captures live network traffic from a specified interface, or reads previously saved `.pcap` files, and outputs packet summaries to the terminal. Because it is text-based, its output can be piped to other command-line tools — `grep`, `wc`, `awk`, `cut` — making it naturally composable in shell scripts and automated analysis pipelines.

Like Wireshark, tcpdump uses libpcap for packet capture on Linux and macOS. Unlike Wireshark, it has no protocol dissection GUI — the analyst reads and filters output directly from the command line.

Capturing live traffic requires root privileges — `sudo` is required for all live capture commands. Reading from a saved file does not require root.

### Basic Packet Capture

The minimum viable tcpdump command specifies a network interface with `-i`:

    sudo tcpdump -i ens5

This begins capturing all traffic on the `ens5` interface and prints a one-line summary per packet to the terminal until interrupted with `Ctrl+C`. To identify available interfaces before capturing:

    ip a s

Key options that modify basic capture behaviour:

| Option | Effect |
|---|---|
| `-i INTERFACE` | Specifies the interface to capture on; use `-i any` to capture on all interfaces |
| `-c COUNT` | Stops capture after COUNT packets |
| `-n` | Disables IP-to-hostname resolution — shows raw IP addresses |
| `-nn` | Disables both hostname resolution and port-to-service resolution |
| `-v` | Verbose output — more packet detail; `-vv` and `-vvv` increase verbosity further |
| `-w FILE` | Writes captured packets to a file (`.pcap` extension) instead of printing to screen |
| `-r FILE` | Reads packets from a saved file rather than a live interface |

The `-n` and `-nn` flags are particularly important in practice. Without them, tcpdump performs DNS lookups on every IP address it encounters, which adds latency and generates additional DNS traffic during the capture — potentially contaminating the capture with noise from the tool itself.

Saving to a file with `-w` suppresses the live scrolling output. The file can later be read back with `-r` or opened in Wireshark for GUI-based analysis.

### Filtering Expressions

tcpdump uses Berkeley Packet Filter (BPF) syntax for filtering — the same syntax used for capture filters in Wireshark. Filters are specified after the options on the command line and apply at the kernel level before packets reach tcpdump, making filtered captures more efficient than capturing everything and filtering later.

Core filter primitives:

| Filter | Effect |
|---|---|
| `host 192.168.1.1` | Traffic to or from the specified host |
| `src host 192.168.1.1` | Traffic from the specified source only |
| `dst host 192.168.1.1` | Traffic to the specified destination only |
| `port 80` | Traffic on port 80 (either direction) |
| `src port 443` | Traffic from source port 443 |
| `tcp` | TCP traffic only |
| `udp` | UDP traffic only |
| `icmp` | ICMP traffic only |

Filters combine using logical operators:

| Operator | Effect |
|---|---|
| `and` | Both conditions must be true |
| `or` | Either condition must be true |
| `not` | Negates the condition |

Practical examples from the room:

    sudo tcpdump -i any tcp port 22             # SSH traffic on all interfaces
    sudo tcpdump -i wlo1 udp port 123          # NTP traffic on WiFi interface
    sudo tcpdump -i eth0 host example.com and tcp port 443 -w https.pcap
    # HTTPS traffic to/from example.com, saved to file

Reading a saved file with a filter applied:

    tcpdump -r traffic.pcap src host 192.168.124.1 -n

### Advanced Filtering

Beyond the basic primitives, tcpdump supports filtering on specific byte offsets within packet headers. This allows filtering on fields that do not have named filter keywords — such as specific TCP flag combinations or ARP target IP addresses expressed in hex.

The ARP filter used in the room to find the host that sent an ARP request for a specific IP demonstrates this:

    tcpdump -r traffic.pcap -n 'arp[24:4] = 0xc0a87c89'

This reads 4 bytes starting at offset 24 in the ARP header and compares them to the hex representation of the target IP address. This level of filter precision is what makes tcpdump viable for surgical packet extraction from large capture files without loading them into a full analysis tool.

Counting filtered results by piping to `wc -l`:

    tcpdump -r traffic.pcap icmp -n | wc -l

This pattern — filter to isolate a protocol, count with `wc -l` — is a standard approach for quickly quantifying how many packets of a specific type appear in a capture.

### Displaying Packet Output

By default, tcpdump prints a compact one-line summary per packet. The verbosity flags expand this:

- `-v` adds TTL, IP ID, total length, and checksum fields
- `-vv` adds further protocol detail
- `-vvv` adds the maximum available detail for supported protocols

The `-A` flag prints the packet payload in ASCII — useful for reading plaintext protocol content such as HTTP headers or SMTP commands directly from the terminal output. The `-X` flag prints payload in both hex and ASCII.

---

## Walkthrough Notes

The room runs through six tasks covering basic capture, reading files, filtering, advanced filtering, and output control. All practical questions use the `traffic.pcap` file provided in the attached VM.

**Task 1 (Introduction):** Frames tcpdump as the command-line counterpart to Wireshark and emphasises that its text-based nature makes it available in server and scripted environments where GUI tools are not. Notes that the room builds on concepts from the Networking Concepts and Networking Essentials rooms — ARP, ICMP, and DHCP traffic all appear in the practical questions. No answer required.

**Task 2 (Basic Packet Capture):** Covers the `-i`, `-c`, `-n`, `-nn`, `-w`, and `-r` flags with examples. The key question asks for the name of the Linux command used to list available network interfaces — the answer is `ip address show` (or equivalently `ip a s`). The room notes that `ip a s` is the recommended first step before specifying `-i` to confirm the interface name.

**Task 3 (Filtering Expressions):** Introduces host, port, protocol, and direction filters, and the `and`, `or`, `not` logical operators. The practical question asks how many packets in `traffic.pcap` use the ICMP protocol — solved by running:

    tcpdump -r traffic.pcap icmp -n | wc -l

A second question asks for the IP address of the host that sent an ARP request for `192.168.124.137`. This requires filtering ARP traffic and reading the output to identify the sender — the filter is `arp` applied with `-r traffic.pcap -n`, and the requesting host's IP is visible in the ARP request output line.

**Task 4 (Advanced Filtering):** Covers byte-offset filtering using the `proto[offset:size] = value` syntax. The ARP target IP filter using hex conversion is the main practical example. The question asks for the IP that requested the MAC address of `192.168.124.137` — approached by filtering ARP packets and reading the source IP from the matching request line. The hex value of `192.168.124.137` converted for the byte-offset filter is `0xc0a87c89`.

**Task 5 (Displaying Packets):** Covers the `-v`, `-vv`, `-vvv`, `-A`, and `-X` flags. Questions in this task ask about specific protocol fields that become visible only with verbose output — TTL values, protocol IDs, and payload content. The `-A` flag is demonstrated as the tool for reading plaintext protocol payloads directly from the terminal.

**Task 6 (Conclusion):** Summarises the room — basic capture, reading files, filtering, advanced filtering, and output control — and notes tcpdump's role alongside Wireshark as complementary tools in a network analyst's workflow. No answer required.

---

## Commands Used

    ip a s
    sudo tcpdump -i ens5
    sudo tcpdump -i ens5 -c 5 -n
    sudo tcpdump -i any tcp port 22
    sudo tcpdump -i eth0 host example.com and tcp port 443 -w https.pcap
    tcpdump -r traffic.pcap -n
    tcpdump -r traffic.pcap icmp -n | wc -l
    tcpdump -r traffic.pcap src host 192.168.124.1 -n | wc
    tcpdump -r traffic.pcap -n 'arp[24:4] = 0xc0a87c89'
    tcpdump -r traffic.pcap -n -v
    tcpdump -r traffic.pcap -n -A

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| Live capture on a specific interface | Remote triage — SSH into a server under investigation and run tcpdump directly to capture traffic without needing physical access or GUI tools |
| `-w` to file, open in Wireshark | Analyst handoff — capture in the field with tcpdump, transfer the `.pcap` to a workstation, and analyse in Wireshark; the two tools are fully interoperable on the same file format |
| `-n` and `-nn` flags | Clean captures — suppressing DNS resolution during capture prevents the tool from generating noise traffic that contaminates the capture being analysed |
| BPF filters at capture time | Targeted capture in high-traffic environments — filtering to a specific host or port at the kernel level reduces capture file size dramatically on busy network segments |
| Pipe to `wc -l` | Quick quantification — counting packets matching a filter gives an immediate sense of scale before committing to deeper analysis; useful for comparing baselines |
| Byte-offset filtering for ARP | Precise protocol investigation — when named filter keywords do not reach deep enough into a header, byte-offset filters allow surgical extraction of packets by any field value |
| `-A` flag for ASCII payload | Plaintext protocol analysis in terminal — reading HTTP headers, SMTP commands, or FTP responses directly from tcpdump output without opening a separate tool |

---

## Takeaways

1. **tcpdump and Wireshark are complementary tools, not alternatives — and knowing when to use each is a skill in itself.** Wireshark's GUI is superior for interactive exploration of an unfamiliar capture. tcpdump's command-line interface is superior for remote capture on live systems, for automation in scripts, and for rapid targeted filtering on the command line. A network analyst who knows only one of them has a meaningful gap in their toolkit.

2. **The `-n` flag should be a default habit, not an afterthought.** Without it, tcpdump performs DNS lookups on captured IP addresses — adding latency, generating additional DNS traffic visible in the capture, and potentially revealing the investigation to a target system's DNS server. Disabling resolution with `-n` keeps the capture clean and the analysis faster. It is the kind of operational detail that distinguishes someone who has used the tool under real conditions from someone who only learned it from a room.

3. **tcpdump's composability with standard Unix tools makes it significantly more powerful than its capture-and-display surface suggests.** Piping to `wc -l` counts packets. Piping to `grep` extracts lines matching a pattern. Piping to `awk` or `cut` isolates specific fields. Writing to a file and reading it back with a different filter supports iterative analysis without re-capturing. This is the Unix philosophy — small tools that do one thing well, combined to handle complex tasks. tcpdump is built entirely on that philosophy, and using it fluently means using it as part of that ecosystem.

---
