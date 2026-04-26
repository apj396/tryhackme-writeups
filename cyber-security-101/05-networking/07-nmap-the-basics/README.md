# Nmap: The Basics

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Networking |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/nmap |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the seventh and final room in the CS101 Networking section. The previous two rooms covered passive observation — capturing and reading traffic that already exists on the network. This room introduces active reconnaissance: using Nmap to discover what is live on a network, which ports are open on those hosts, and what services are running behind those ports. Nmap has been the standard network scanner since its first release in 1997. The room covers host discovery, port scanning techniques, service and version detection, timing control, output verbosity, and saving scan results — the core workflow for any network enumeration task.

---

## Key Concepts

### What Nmap Is

Nmap (Network Mapper) is an open-source network scanner first published in 1997. It answers two fundamental questions about a network: which hosts are live, and what services are running on those hosts. It does this through a combination of ICMP probes, TCP and UDP packet crafting, and banner grabbing — adapting its approach based on the scan type specified and the responses (or lack of responses) it receives.

Nmap runs most effectively with root privileges. Without `sudo`, certain scan types — particularly SYN scans — are unavailable because they require raw socket access. Throughout this room, Nmap is run as root or with `sudo`.

### Specifying Targets

Nmap supports multiple target specification formats:

| Format | Example | Meaning |
|---|---|---|
| Single IP | `192.168.0.1` | One host |
| IP range | `192.168.0.1-10` | All IPs from .1 to .10 |
| CIDR subnet | `192.168.0.1/24` | All 256 addresses in the /24 |
| Hostname | `example.thm` | Resolved via DNS |

A `/27` subnet covers 32 addresses — the last scannable IP in `192.168.0.1/27` is `192.168.0.31`. Understanding subnet math is a prerequisite for using Nmap's target specification correctly.

### Host Discovery

Before scanning ports, Nmap determines which hosts are actually live — there is no point sending port probes to an offline host. The ping scan option `-sn` performs host discovery without port scanning:

    sudo nmap -sn 192.168.0.0/24

Despite the name, `-sn` is not limited to ICMP ping. On a local network, Nmap sends: two ICMP echo requests, two ICMP timestamp requests, one TCP SYN to port 443, and one TCP ACK to port 80. The combination maximises the chance of detecting a host even if ICMP is blocked by a firewall. A host that responds to any of these probes is marked as up.

### Port Scanning

Once live hosts are identified, Nmap scans their ports to determine which are open, closed, or filtered. Nmap recognises six port states:

| State | Meaning |
|---|---|
| Open | A service is actively accepting connections on this port |
| Closed | The port is accessible but no service is listening |
| Filtered | A firewall or filter is preventing Nmap from determining the state |
| Unfiltered | The port is accessible but state cannot be determined (ACK scan specific) |
| Open\|Filtered | Nmap cannot determine if open or filtered |
| Closed\|Filtered | Nmap cannot determine if closed or filtered |

For a pentester or SOC analyst, `open` is the most significant state — it identifies attack surface and running services.

**TCP Connect Scan (`-sT`)** completes the full three-way handshake with each port. It is the default when Nmap runs without root privileges. It is reliable but loud — the complete connection is logged by the target system.

**TCP SYN Scan (`-sS`)** sends a SYN packet and waits for a response. If a SYN-ACK is received, the port is open — Nmap then sends a RST to tear down the connection without completing the handshake. This is the default scan when running as root. It is faster than `-sT` and less likely to be logged because the connection is never fully established. It is sometimes called a stealth scan or half-open scan.

**UDP Scan (`-sU`)** probes UDP ports. UDP has no handshake, so detection is less reliable — an open UDP port may simply not respond, while a closed one typically returns an ICMP port unreachable message. UDP scans are slower than TCP scans as a result.

### Service and Version Detection

Knowing a port is open tells you the port number — it does not confirm what is actually running on it. The `-sV` flag instructs Nmap to probe open ports and attempt to identify the service and its version:

    sudo nmap -sV MACHINE_IP

With `-sV`, Nmap establishes a full connection to each open port and reads the service banner. The output gains a version column — instead of `22/tcp open ssh`, it shows `22/tcp open ssh OpenSSH 8.2p1`. Version information is directly relevant to vulnerability assessment — a specific version string can be matched against known CVEs.

Version detection intensity can be controlled:

    --version-light    # intensity 2 — faster, less thorough
    --version-all      # intensity 9 — most complete

OS detection adds the `-O` flag and attempts to fingerprint the target operating system based on TCP/IP stack behaviour. Nmap compares observed responses against a database of known OS fingerprints.

### Timing Control

Nmap's timing templates control the speed and aggressiveness of a scan. They are specified with `-T` followed by a number 0–5, or their equivalent names:

| Template | Name | Behaviour |
|---|---|---|
| `-T0` | Paranoid | Extremely slow — evades most IDS systems |
| `-T1` | Sneaky | Very slow — reduced IDS detection risk |
| `-T2` | Polite | Slower than default — reduces network load |
| `-T3` | Normal | Default — balanced speed and reliability |
| `-T4` | Aggressive | Faster — assumes a reliable, fast network |
| `-T5` | Insane | Maximum speed — may miss results on slow networks |

`-T4` is the equivalent of `-T aggressive`. In controlled lab environments and authorised assessments on reliable networks, `-T4` is commonly used. In real-world engagements where stealth matters, `-T1` or `-T2` reduces the footprint.

### Verbosity, Debugging, and Output

By default, Nmap prints results after the scan completes. Verbosity with `-v` causes Nmap to print open ports as they are discovered rather than waiting — useful for long scans.

Debugging with `-d` increases the detail of Nmap's internal output. Multiple `-d` flags increase verbosity further.

Nmap can save scan results in multiple formats:

| Flag | Format |
|---|---|
| `-oN FILE` | Normal — human-readable text |
| `-oX FILE` | XML — machine-parseable |
| `-oG FILE` | Grepable — one line per host, suitable for shell processing |
| `-oA FILE` | All three formats simultaneously |

Saving results is essential for documentation, for comparing scans over time, and for feeding Nmap output into other tools.

---

## Walkthrough Notes

The room runs through several tasks covering host discovery, port scanning, service detection, timing, and output. All practical tasks use an attached VM as the scan target.

**Task 1 (Introduction):** Frames the problem Nmap solves — manual host discovery and port checking do not scale. Introduces Nmap as the tool that automates both. Notes that throughout the room, Nmap is run as root or with `sudo` to avoid restricting available scan types. No answer required.

**Task 2 (Host Discovery):** Covers the `-sn` ping scan and the multiple probe types Nmap uses beyond ICMP. The key question asks for the last IP address scanned when the target is `192.168.0.1/27` — the answer is `192.168.0.31`, derived from the /27 subnet covering 32 addresses starting at .0. A live scan against `MACHINE_IP`'s subnet using `-sn` identifies the online hosts for follow-on port scanning tasks.

**Task 3 (Port Scanning):** Covers TCP Connect (`-sT`), TCP SYN (`-sS`), and UDP (`-sU`) scans. The question asks how many TCP ports are open on `MACHINE_IP` — the answer is 6, found by running a default TCP scan. A follow-up question asks for the flag on the web server found during the scan — accessing the open HTTP port in a browser reveals the flag `THM{SECRET_PAGE_38B9P6}` on the main page. The question on the number of port states Nmap recognises is answered by 6 — open, closed, filtered, unfiltered, open|filtered, closed|filtered.

**Task 4 (Version Detection and OS Fingerprinting):** Covers `-sV` and `-O`. Running `nmap -sV MACHINE_IP` against the lab machine reveals the web server name and version — `lighttpd 1.4.74`. Running with `-O` identifies the target OS as Linux. The `--version-light` flag is introduced as the lower-intensity variant; the room notes that `rpcbind` does not return a version with `--version-light` — a practical illustration of how intensity affects detection completeness.

**Task 5 (Timing and Performance):** Covers `-T0` through `-T5` and the named equivalents. The question asks for the non-numeric equivalent of `-T4` — the answer is `-T aggressive`. Timing templates are explained in the context of their trade-off between speed and stealth.

**Task 6 (Output and Verbosity):** Covers `-v`, `-d`, and the four output format flags. The question asks what option must be added to enable debugging — the answer is `-d`. The `-oA` flag is highlighted as the most practical choice in professional contexts since it produces all three output formats simultaneously from a single scan.

**Task 7 (Conclusion):** Summarises the full Networking section and notes that the tools covered — Wireshark, tcpdump, and Nmap — form the core of practical network analysis at the basics level. No answer required.

---

## Commands Used

    sudo nmap -sn 192.168.0.0/24
    sudo nmap MACHINE_IP
    sudo nmap -sT MACHINE_IP
    sudo nmap -sS MACHINE_IP
    sudo nmap -sU MACHINE_IP
    sudo nmap -sV MACHINE_IP
    sudo nmap -sV --version-light MACHINE_IP
    sudo nmap -O MACHINE_IP
    sudo nmap -sS -T4 MACHINE_IP
    sudo nmap -sS -v MACHINE_IP
    sudo nmap -sS -d MACHINE_IP
    sudo nmap -sS -oN scan.txt MACHINE_IP
    sudo nmap -sS -oA scan MACHINE_IP

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| Host discovery (`-sn`) | Asset inventory verification — scanning a network segment to confirm which hosts are live is a standard step in both security assessments and incident response scope definition |
| TCP SYN scan (`-sS`) | Authorised internal scanning — SYN scans are the default for internal network enumeration during penetration tests; their reduced logging footprint matters in real engagements |
| Six port states | Alert interpretation — firewall rules produce `filtered` states; IDS/IPS inline blocking produces similar results; distinguishing `closed` from `filtered` informs firewall rule analysis |
| Service version detection (`-sV`) | Vulnerability mapping — matching a detected version string against CVE databases identifies known exploitable services; this is the first step in patch gap analysis |
| OS detection (`-O`) | Network baseline — knowing what OS is expected on a host makes OS fingerprint anomalies detectable; an unexpected OS on a known asset is a lateral movement or rogue host indicator |
| Timing templates (`-T`) | Operational security — in real engagements, scan speed is a trade-off between time and detection risk; `-T1` or `-T2` reduces IDS trigger likelihood at the cost of scan duration |
| Output formats (`-oA`) | Documentation and toolchain integration — XML output feeds directly into vulnerability management platforms and reporting tools; grepable output supports shell-based analysis pipelines |

---

## Takeaways

1. **Nmap closes the loop between the conceptual and the operational.** The previous rooms established what a port is, what TCP and UDP are, what services run on which ports. Nmap is the tool that turns those concepts into actionable intelligence about a real network. Knowing that port 22 runs SSH is useful background. Knowing that `192.168.1.45:22` is running `OpenSSH 7.4` with a known vulnerability is an operational finding. The gap between those two things is what `-sV` covers.

2. **The difference between `-sT` and `-sS` is not just technical — it is about understanding what evidence you leave behind.** A completed TCP connection is logged. A half-open SYN scan typically is not. In an authorised assessment, this distinction affects what the target's logs will show and therefore what defenders can detect and correlate. Understanding scan types at this level of detail is what separates using Nmap as a script from using it as an analyst.

3. **Saving scan output is not optional in professional contexts — it is the job.** Network state changes. A service running on port 8080 today may be gone next week or replaced with something else. Saving scan results with `-oA` creates a dated record of network state at a point in time. Comparing that record against a future scan is how configuration drift, new exposures, and unauthorised services get detected. The scan itself is the beginning of the work; the saved output is what makes it reusable.

---
