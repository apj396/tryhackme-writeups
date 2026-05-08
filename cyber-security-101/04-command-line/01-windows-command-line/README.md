# Windows Command Line

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Command Line |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/windowscommandline |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the first room in the CS101 Command Line section. The Windows Fundamentals series introduced `cmd.exe` as a tool and demonstrated `hostname` and `ipconfig`. This room builds on that foundation comprehensively — covering basic system information, network diagnostics, filesystem management, and task management from the command line. The lab machine runs Windows Server Core, a version of Windows without a GUI, accessed entirely via SSH from the AttackBox. This mirrors the real-world scenario of administering or investigating a Windows server that has no desktop environment — a configuration that is common in enterprise infrastructure and requires full command-line fluency to navigate.

---

## Key Concepts

### The Windows Command Prompt

`cmd.exe` is the default command-line interpreter in Windows. While PowerShell has largely superseded it for scripting and administration, `cmd.exe` is present on every Windows system and remains in wide use — particularly for quick ad hoc commands, legacy scripts, and in environments where PowerShell execution policy restrictions apply. Understanding it is also directly relevant to attacker tradecraft: many malware payloads and post-exploitation commands run through `cmd.exe`.

The lab machine is accessed via SSH from the AttackBox:

    ssh user@MACHINE_IP

Password: `Tryhackme123!`. Note that the password does not appear as you type it.

Before issuing commands, the Windows Path determines which directories are searched when a command is run. The `set` command displays all environment variables, including the `Path` variable showing where Windows looks for executables.

### Basic System Information

Four commands provide an immediate picture of a Windows system:

| Command | Output |
|---|---|
| `ver` | Displays the Windows version string |
| `set` | Displays all environment variables including `Path`, `OS`, `USERNAME`, `COMPUTERNAME` |
| `systeminfo` | Comprehensive system information — OS version and build, hostname, registered owner, install date, last boot time, system locale, hardware details, hotfix list, and network adapter configuration |
| `hostname` | Returns the machine's hostname |

`systeminfo` is particularly valuable during incident response and post-exploitation enumeration — it surfaces patch level (hotfix list), boot time, and network adapter configuration in a single command.

### Network Information

Five commands cover the full range of Windows command-line network diagnostics:

**`ipconfig`** returns IP address, subnet mask, and default gateway for all network adapters. **`ipconfig /all`** extends this to include MAC address (Physical Address), DNS servers, DHCP status, and lease information. The MAC address lookup question in the room is answered with `ipconfig /all`.

**`ping target`** sends four ICMP Echo Requests to the target and reports round-trip time and packet loss. It verifies reachability and basic network path health.

**`tracert target`** traces the route to a destination by exploiting the IP TTL field — each router that decrements TTL to zero returns an ICMP Time Exceeded message, revealing itself. This maps the network hops between source and destination with per-hop latency.

**`nslookup domain`** queries the DNS resolver for the IP address of a hostname. It can also be used to query specific record types and specific DNS servers.

**`netstat`** displays current network connections and listening ports. The most operationally useful combination of flags is:

    netstat -abon

Breaking down the flags:

| Flag | Effect |
|---|---|
| `-a` | All established connections and listening ports |
| `-b` | Shows the executable associated with each connection or listening port |
| `-o` | Shows the PID associated with each connection |
| `-n` | Displays addresses and port numbers in numerical form — no DNS resolution |

Running `netstat -abon` and looking for port 3389 reveals the process listening for Remote Desktop Protocol connections — `TermService` running via `svchost.exe`.

### File and Disk Management

Windows CMD provides a complete set of filesystem navigation and management commands:

| Command | Purpose |
|---|---|
| `cd` | Without arguments, displays the current drive and directory. With a path, changes to that directory |
| `dir` | Lists files and directories in the current directory; `dir /a` shows hidden files |
| `mkdir` | Creates a new directory |
| `rmdir` | Removes an empty directory; `rmdir /s` removes a directory and all its contents |
| `tree` | Displays a visual representation of the directory structure from the current location |
| `type` | Displays the contents of a text file (equivalent of `cat` in Linux) |
| `copy` | Copies a file; supports wildcards (`copy *.txt C:\Destination`) |
| `move` | Moves a file to a new location — also used to rename files |
| `del` / `erase` | Deletes a file |

The practical filesystem question requires navigating to `C:\Treasure\Hunt`, listing contents with `dir /a` to reveal all files including hidden ones, and reading `flag.txt` with `type flag.txt`.

### Task Management

Two commands manage running processes from CMD:

**`tasklist`** lists all running processes with their PID, session name, session number, and memory usage. The `/FI` flag applies a filter — to find all tasks related to a specific executable:

    tasklist /FI "imagename eq sshd.exe"

**`taskkill`** terminates a process by PID or by image name:

    taskkill /PID 1234
    taskkill /IM notepad.exe

The `/F` flag forces termination without waiting for a graceful exit — equivalent to SIGKILL on Linux.

---

## Walkthrough Notes

The room runs through six tasks. All work is conducted via SSH to the Windows Server Core VM from the AttackBox. Credentials: `user:Tryhackme123!`.

**Task 1 (Introduction):** Frames the room — CLI efficiency, fewer resource requirements than GUI, easier automation. Notes that the default command-line interpreter in Windows is `cmd.exe`. The question asks for the default interpreter — the answer is `cmd.exe`.

**Task 2 (Basic System Information):** Covers `ver`, `set`, `systeminfo`, and `hostname`. Questions: the command to look up the OS version is `ver`; running `systeminfo` on the VM reveals the OS name, hotfix count, and boot time. The question asking which command provides the system's detailed configuration is `systeminfo`.

**Task 3 (Network Information):** Covers `ipconfig`, `ipconfig /all`, `ping`, `tracert`, `nslookup`, and `netstat`. The question asking which command looks up the MAC address is answered by `ipconfig /all`. Running `netstat -abon` and filtering for port 3389 reveals the associated process — `TermService`. The subnet mask of the VM is found via `ipconfig /all`.

**Task 4 (File and Disk Management):** Covers `cd`, `dir`, `mkdir`, `rmdir`, `tree`, `type`, `copy`, `move`, and `del`. The practical question requires navigating to `C:\Treasure\Hunt` and reading `flag.txt` with `type flag.txt` to obtain the flag.

**Task 5 (Task Management):** Covers `tasklist` and `taskkill`. The question asks how to filter `tasklist` for a specific process image name — the answer uses the `/FI` flag with `"imagename eq processname.exe"`. A question on terminating a process by PID uses `taskkill /PID [pid]`.

**Task 6 (Conclusion):** Summarises the room and points to Windows PowerShell as the next room. No answer required.

---

## Commands Used

    ssh user@MACHINE_IP
    ver
    set
    systeminfo
    hostname
    ipconfig
    ipconfig /all
    ping example.com
    tracert example.com
    nslookup example.com
    netstat
    netstat -abon
    cd
    cd C:\Treasure\Hunt
    dir
    dir /a
    type flag.txt
    mkdir newfolder
    rmdir /s oldfolder
    copy file.txt C:\Destination
    move file.txt C:\Destination
    del file.txt
    tree
    tasklist
    tasklist /FI "imagename eq sshd.exe"
    taskkill /PID 1234
    taskkill /IM notepad.exe /F

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| `systeminfo` | Post-compromise enumeration — attackers run `systeminfo` immediately after gaining access to fingerprint the target OS, patch level, and hardware; defenders use it to establish machine baseline during investigation |
| `netstat -abon` | Connection investigation — identifying which executable is responsible for a suspicious network connection without third-party tools; process-to-port mapping is the first attribution step in connection triage |
| `tasklist /FI` | Targeted process investigation — filtering for a specific process by image name verifies whether a known malicious executable is running without scrolling through the full process list |
| `ipconfig /all` | Network configuration verification — confirming IP address, MAC address, DNS configuration, and DHCP lease status during incident response establishes network context for the compromised host |
| `taskkill /PID` | Remote process termination — killing a malicious process by PID over an SSH session when no GUI is available; using `/F` ensures termination even if the process is unresponsive |
| `dir /a` | Hidden file discovery — many dropped malware files are hidden using the Windows hidden attribute; `dir /a` reveals them in directory listings |
| `type` | Quick file reading — reading dropped files, configuration files, or log entries during investigation without launching a text editor |

---

## Takeaways

1. **`netstat -abon` is one of the highest signal-to-noise commands available on a Windows system during incident response.** In a single output it shows every listening port, every established connection, the executable responsible for each, and the PID — enough to identify unexpected services, active C2 connections, and processes that should not be communicating over the network. Knowing this command and being able to read its output fluently is a baseline competency for Windows host investigation.

2. **Windows Server Core — no GUI — is not an edge case.** It is a deliberate, security-conscious deployment choice used across enterprise server fleets. Analysts and administrators who can only work on Windows through a graphical interface are locked out of these environments. The SSH-to-CMD workflow in this room is the actual workflow for investigating, administering, and hardening headless Windows servers. Building comfort with it now saves significant friction later.

3. **`systeminfo` reveals the patch level, and patch level determines exploitability.** The hotfix list in `systeminfo` output shows every installed Windows update. Cross-referencing that list against known CVEs for the running OS version immediately surfaces which publicly disclosed vulnerabilities have not been patched. This is trivially automatable and is a standard first step in both offensive and defensive Windows assessments.

---
