# Windows Fundamentals 2

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Windows and AD Fundamentals |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/windowsfundamentals2x0x |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the second room in the CS101 Windows and AD Fundamentals section. Windows Fundamentals 1 covered the desktop, filesystem, user accounts, UAC, and Task Manager — the user-facing layer of Windows. This room goes deeper into the administrative layer: the System Configuration utility (MSConfig) as a central launchpad for Windows tools, UAC settings adjustment, Computer Management, System Information, the Resource Monitor, the Command Prompt, and the Windows Registry. The room is heavily tool-oriented — each task introduces a specific utility, explains what it does, and asks questions whose answers require actually using the tool on the attached VM.

---

## Key Concepts

### System Configuration — MSConfig

The System Configuration utility (MSConfig) is designed primarily for advanced troubleshooting, with its main purpose being to help diagnose and isolate startup issues. It is opened via the Run dialog (`Win+R`), the Start Menu search, or by typing `msconfig` in a Command Prompt.

MSConfig has five tabs:

| Tab | Purpose |
|---|---|
| General | Select startup mode: Normal (all drivers and services), Diagnostic (basic drivers only), or Selective (user-specified subset) |
| Boot | Configure boot options for the operating system — safe boot, boot logging, base video |
| Services | Lists all services configured on the system regardless of running state; allows selective enabling and disabling for troubleshooting |
| Startup | On Windows desktop systems, Microsoft directs startup application management to Task Manager; on the Windows Server VM used in this room, the Startup tab does not surface startup items the same way |
| Tools | A curated list of Windows administrative utilities with a brief description of each and the command used to launch it — this tab functions as a launchpad for every tool covered in the room |

The Tools tab is the primary focus of the room. Selecting a tool populates the "Selected command" field at the bottom of the tab with the exact command to launch that utility — either directly or from the Run prompt. This makes MSConfig a reference point for administrative tool commands without needing to memorise them.

The service that lists Sysinternals as its manufacturer, visible in the Services tab, is `PsShutdown`. The Windows version information is visible via the "About Windows" tool in the Tools tab.

### UAC Settings

UAC settings can be adjusted independently of the on/off toggle. The Change UAC Settings tool — accessible from the Tools tab in MSConfig or by running `UserAccountControlSettings.exe` — presents a slider with four levels:

| Level | Behaviour |
|---|---|
| Always notify | Notifies for all app changes and all user changes to Windows settings — most secure |
| Notify for apps only (default) | Notifies when apps attempt changes; does not notify when the user manually changes Windows settings; desktop dims (Secure Desktop) |
| Notify for apps without dimming | Same as above but the desktop does not dim — Secure Desktop is disabled, which means other running applications could potentially interact with the UAC prompt |
| Never notify | UAC is effectively disabled — not recommended |

The command to open UAC Settings is `UserAccountControlSettings.exe`.

### Computer Management — compmgmt.msc

Computer Management consolidates several administrative tools into a single console. It is opened via `compmgmt.msc`. The console has three primary sections:

**System Tools:**
- **Task Scheduler** — create and manage automated tasks that run at logon, logoff, on a schedule, or on a trigger event; the `GoogleUpdateTaskMachineUA` scheduled task visible on the VM runs on a schedule visible in the Triggers column
- **Event Viewer** — a log of all events that have occurred on the system, providing an audit trail for diagnosing problems and investigating actions; events are organised by Windows Logs (Application, Security, Setup, System) and Applications and Services Logs
- **Shared Folders** — lists active shares, open sessions, and open files; the non-standard share visible on the VM (`sh4r3dF0ld3r`) is the answer to the shared folder question
- **Performance** — resource monitoring
- **Device Manager** — hardware device status and driver management

**Storage:**
- **Disk Management** — view and manage disk volumes, partitions, and drive letters

**Services and Applications:**
- **Services** — view, start, stop, and configure all system services
- **WMI Control** — Windows Management Instrumentation configuration

### System Information — msinfo32

System Information (`msinfo32.exe`) provides a comprehensive view of the system's hardware, software components, and current configuration. It is organised into three main categories:

| Category | Contents |
|---|---|
| System Summary | OS name and version, system manufacturer, processor, BIOS version, installed RAM, locale, time zone |
| Hardware Resources | IRQ assignments, DMA channels, I/O addresses, memory ranges |
| Components | Display, multimedia, input, network, storage, and other hardware component details |
| Software Environment | Drivers, environment variables, print jobs, network connections, running tasks, loaded modules, services, startup programs |

The Software Environment section within System Information lists environment variables — including the value of `%windir%` and the system path. The command to open System Information is `msinfo32.exe`.

### Resource Monitor — resmon

The Resource Monitor (`resmon.exe`) provides real-time data on CPU, memory, disk, and network resource usage at a granular level — far beyond what Task Manager displays. It shows exactly which processes are using which resources, which files are locked by which processes, and which network connections are active per process. The command to open Resource Monitor is `resmon.exe`.

### Command Prompt — cmd

The Windows Command Prompt (`cmd.exe`) is the command-line interpreter for Windows. Despite PowerShell's emergence as the more capable shell, `cmd.exe` remains present on all Windows systems and is frequently used by both administrators and attackers. The command to open Command Prompt found in the MSConfig Tools tab is `C:\Windows\System32\cmd.exe`.

Two commands are covered in this task:

**`hostname`** returns the computer's hostname — the name of the machine on the network. On the lab VM, this returns the machine's name.

**`ipconfig`** returns the network configuration of all adapters on the machine — IP address, subnet mask, default gateway. The flag that shows detailed information (including MAC address, DNS server, DHCP status, and lease information) is `/all`:

    ipconfig /all

The full command path for ipconfig as shown in MSConfig's Tools tab is:

    C:\Windows\System32\cmd.exe /k %windir%\system32\ipconfig.exe

### The Windows Registry — regedit

The Windows Registry is a central hierarchical database that stores configuration settings for the operating system, installed applications, hardware devices, and user preferences. It is critical infrastructure — modifying it incorrectly can cause system instability or render the OS unbootable. The Registry is organised into five root keys:

| Root Key | Purpose |
|---|---|
| HKEY_CLASSES_ROOT (HKCR) | File type associations and COM object registrations |
| HKEY_CURRENT_USER (HKCU) | Settings for the currently logged-in user |
| HKEY_LOCAL_MACHINE (HKLM) | System-wide settings — hardware configuration, installed software, security settings |
| HKEY_USERS (HKU) | Settings for all user profiles loaded on the system |
| HKEY_CURRENT_CONFIG (HKCC) | Hardware profile used at startup |

The Registry is the primary location where malware establishes persistence on Windows. Common persistence keys — such as `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` and `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` — cause entries to execute at every user login. Registry analysis is a core step in any Windows host forensic investigation. The command to open the Registry Editor is `regedit.exe`.

---

## Walkthrough Notes

The room runs through nine tasks. All work is done within the in-browser Windows VM (or via RDP with credentials `administrator:letmein123!`).

**Task 1 (Introduction):** Notes that this room builds on Windows Fundamentals 1 and covers administrative utilities accessible via MSConfig. No answer required.

**Task 2 (System Configuration — MSConfig):** The VM is deployed and MSConfig is opened. Questions: the service with Sysinternals as manufacturer is found in the Services tab — the answer is `PsShutdown`; the Windows version is found via Tools → About Windows; the command to open Windows Troubleshooting (found in Tools → Windows Troubleshooting → Selected command) is `C:\Windows\System32\control.exe /name Microsoft.Troubleshooting`; the command to open the Control Panel (found in Tools → System Properties → Selected command) is `control.exe`.

**Task 3 (Change UAC Settings):** UAC settings are found via Tools → Change UAC Settings → Launch, or by running `UserAccountControlSettings.exe` directly. The question asks for the command to open UAC Settings — the answer is `UserAccountControlSettings.exe`.

**Task 4 (Computer Management):** Opened via `compmgmt.msc` or Tools → Computer Management. Questions: the command to open Computer Management is `compmgmt.msc`; the scheduled task `GoogleUpdateTaskMachineUA` trigger details are found in the Task Scheduler section under Triggers; the non-standard share is found in Shared Folders → Shares — the answer is `sh4r3dF0ld3r`.

**Task 5 (System Information):** Opened via `msinfo32.exe` or Tools → System Information. The question asks for the command to open System Information — the answer is `msinfo32.exe`. Additional questions on system hardware details are answered by reading the System Summary panel.

**Task 6 (Resource Monitor):** Opened via `resmon.exe` or Tools → Resource Monitor. The question asks for the command to open Resource Monitor — the answer is `resmon.exe`.

**Task 7 (Command Prompt):** Opens Command Prompt from MSConfig or directly. Questions: the full command for Internet Protocol Configuration found in Tools is `C:\Windows\System32\cmd.exe /k %windir%\system32\ipconfig.exe`; the flag for detailed ipconfig output is `/all`.

**Task 8 (Windows Registry):** Opened via `regedit.exe` or Tools → Registry Editor. The question asks for the command to open the Registry Editor — the answer is `regedit.exe`.

**Task 9 (Conclusion):** Notes that all tools covered are also accessible directly without going through MSConfig. No answer required.

---

## Commands and Tools Used

    msconfig                           # System Configuration utility
    UserAccountControlSettings.exe     # UAC Settings
    compmgmt.msc                       # Computer Management
    msinfo32.exe                       # System Information
    resmon.exe                         # Resource Monitor
    cmd.exe                            # Command Prompt
    hostname                           # Returns computer name
    ipconfig                           # Network configuration
    ipconfig /all                      # Detailed network configuration
    regedit.exe                        # Registry Editor

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| MSConfig Services tab | Persistence review — disabling all non-Microsoft services in MSConfig to determine if a problem disappears isolates third-party service interference; also used to identify unexpected services during triage |
| Event Viewer | Incident investigation — Security event logs record logon events, privilege use, object access, and policy changes; Event ID 4624 (successful logon) and 4625 (failed logon) are among the most commonly reviewed events in investigations |
| Shared Folders (compmgmt) | Network exposure audit — unexpected file shares are a common finding in network assessments; shares accessible to `Everyone` with write permissions are a lateral movement enabler |
| Registry Run keys | Persistence hunting — `HKLM\...\CurrentVersion\Run` and `HKCU\...\CurrentVersion\Run` are the first registry locations checked for attacker persistence; automated tools like Autoruns (Sysinternals) enumerate these comprehensively |
| Resource Monitor per-process network | Connection investigation — identifying which process is responsible for a specific network connection is possible in Resource Monitor under the Network tab, providing process-level attribution without requiring a dedicated tool |
| ipconfig /all | Network reconnaissance baseline — MAC address, DNS server, DHCP lease information, and default gateway are all visible; during incident response, confirming network configuration helps establish whether a machine has been reconfigured |

---

## Takeaways

1. **MSConfig's Tools tab is a map of the Windows administrative landscape — and knowing it reduces investigation time.** Rather than searching for where a utility lives or what command launches it, the Selected command field in MSConfig gives the exact executable and any required parameters. For a new analyst working on an unfamiliar Windows environment, MSConfig is a reliable starting point for accessing any administrative tool without needing to memorise dozens of paths.

2. **The Windows Registry is where persistence lives — and understanding its structure is non-negotiable for host forensics.** The Run keys under HKLM and HKCU are where most commodity malware establishes boot persistence. Services registered under HKLM\SYSTEM\CurrentControlSet\Services represent another persistence tier. Registry forensics — reading, comparing, and differencing registry hives — is a core skill in Windows incident response, and it starts with knowing what the five root keys contain and why HKLM versus HKCU matters.

3. **Event Viewer contains the closest thing Windows has to a complete audit trail — but only if logging is configured to capture what matters.** By default, Windows logs many events but not all. Security audit policy determines which events generate Security log entries. An environment where audit policy has not been deliberately configured may have gaps — successful process creation may not be logged, object access may not be tracked, or logon events may be missing. Knowing what Event Viewer can and cannot show, and why, is as important as knowing how to read it.

---
