# Windows Fundamentals Part 2

**Module:** Pre-Security (Legacy) → Windows Fundamentals  
**Difficulty:** Easy  
**Platform:** TryHackMe  
**Room Link:** https://tryhackme.com/room/windowsfundamentals2x0x  
**Date Completed:** January 2026  
**Author:** Adwait Joshi

---

## What This Room Covers

Part 2 completes Windows Fundamentals by covering the tools and subsystems that matter most in security investigation and hardening — the System Configuration utility in depth, UAC settings, resource monitoring, the Windows Registry, and the command line environment. It also introduces Windows Update and security tooling built into the OS.

Where Part 1 covered the structure of Windows, Part 2 covers the mechanisms used to configure, monitor, and secure it.

---

## Key Concepts

### 1. System Configuration (msconfig) — In Depth

`msconfig` was introduced in Part 1. Part 2 covers the Tools tab specifically — a launchpad for system administration and investigation tools directly accessible from a single interface.

```
Win+R → msconfig → Tools tab
```

**Tools tab inventory — security-relevant entries:**

| Tool | Launch Command | Purpose |
|---|---|---|
| About Windows | `winver` | OS version and build number |
| Change UAC Settings | `UserAccountControlSettings.exe` | Adjust UAC level |
| Security and Maintenance | `wscui.cpl` | Security status overview |
| Windows Troubleshooting | `control.exe /name Microsoft.Troubleshooting` | Diagnostic tools |
| Computer Management | `compmgmt.msc` | Central admin console |
| System Information | `msinfo32.exe` | Comprehensive system details |
| Event Viewer | `eventvwr.msc` | Windows log viewer |
| Programs and Features | `appwiz.cpl` | Installed software list |
| Internet Properties | `inetcpl.cpl` | IE/Edge settings, proxy config |
| Internet Protocol Configuration | `cmd` → `ipconfig` | Network configuration |
| Performance Monitor | `perfmon.exe` | Resource performance tracking |
| Resource Monitor | `resmon.exe` | Real-time resource usage |
| Task Manager | `taskmgr.exe` | Process and performance viewer |
| Command Prompt | `cmd.exe` | Command line interface |
| Registry Editor | `regedit.exe` | Registry viewer and editor |

**Real-world relevance:** During a Windows investigation, `msconfig` Tools tab provides rapid access to every diagnostic tool without navigating menus. More importantly, `msinfo32.exe` and `eventvwr.msc` are the two tools that answer the majority of "what is this system and what has happened on it" questions in an incident.

---

### 2. System Information (msinfo32)

`msinfo32` provides a comprehensive snapshot of the system — hardware, software, and running components.

```
Win+R → msinfo32
```

**Three primary sections:**

**System Summary** — OS name, version, build, architecture, domain membership, BIOS version, installed RAM, processor details.

**Hardware Resources** — IRQs, I/O ports, memory ranges, DMA channels. Relevant for hardware-level conflict investigation.

**Components** — detailed hardware inventory including network adapters (with MAC addresses), storage, display, and USB devices.

**Software Environment** — the most security-relevant section:

| Sub-section | Security Relevance |
|---|---|
| System Drivers | Loaded drivers — malicious drivers appear here |
| Environment Variables | PATH manipulation — attack surface |
| Print Jobs | Active print jobs |
| Network Connections | Active connections at query time |
| Running Tasks | Running processes with full binary paths |
| Loaded Modules | DLLs loaded into processes |
| Services | All services — running and stopped |
| Program Groups | Startup programs |
| Startup Programs | Programs that run at login |

```cmd
:: Export msinfo32 to text file for offline analysis
msinfo32 /report C:\temp\sysinfo.txt
msinfo32 /nfo C:\temp\sysinfo.nfo     :: NFO format
```

**Real-world relevance:** The Software Environment → Startup Programs section reveals persistence mechanisms — programs configured to run at login. The Running Tasks section shows processes with full binary paths, making it harder to spoof than Task Manager's default view. `msinfo32 /report` is used in incident response to capture a system snapshot for offline analysis.

---

### 3. Resource Monitor (resmon)

Resource Monitor provides real-time visibility into CPU, memory, disk, and network usage — with process-level granularity.

```
Win+R → resmon
```

**Four tabs:**

**Overview** — summary of all four resource types simultaneously. Colour-coded bars show usage levels.

**CPU** — per-process CPU usage. Includes Services section showing which services are consuming CPU.

**Memory** — physical memory usage per process. Hard faults (page faults requiring disk access) are a performance and investigation indicator.

**Disk** — per-process read/write activity. High disk I/O from unexpected processes — especially writes to `%TEMP%` or unusual directories — is a behavioural indicator.

**Network** — the most security-relevant tab:
- **Processes with Network Activity** — every process making network connections, with PID
- **Network Activity** — active connections with local/remote addresses and ports
- **TCP Connections** — established TCP connections with state
- **Listening Ports** — all ports currently listening, with owning process

**Real-world relevance:** Resource Monitor's Network tab answers "what process is responsible for this network connection?" in real time — without needing `netstat -ano` cross-referenced against `tasklist`. A process making unexpected outbound connections to unusual IPs, or a listening port owned by an unexpected binary, are actionable indicators. During a live incident on a Windows system, Resource Monitor's Network tab is frequently the first tool opened.

---

### 4. Command Prompt and PowerShell

Windows provides two command-line environments with different capabilities and security implications.

**Command Prompt (cmd.exe):**

```cmd
:: Basic navigation
cd C:\Users\username\Desktop
dir
dir /a                          :: show hidden files
dir /s /b *.txt                 :: recursive search for .txt files

:: System information
ipconfig
ipconfig /all                   :: full network config including MAC
ipconfig /displaydns            :: cached DNS entries
ipconfig /flushdns              :: flush DNS cache

:: Network
netstat -ano                    :: all connections with PIDs
netstat -ano | findstr LISTENING :: listening ports only
netstat -ano | findstr :443     :: connections on port 443

:: Processes
tasklist
tasklist /v                     :: verbose — includes window title
tasklist /svc                   :: services hosted in each process
taskkill /PID 1234 /F

:: User enumeration
net user
net localgroup Administrators
whoami
whoami /priv                    :: current user privileges
whoami /groups                  :: group memberships

:: File operations
type file.txt                   :: equivalent of cat
copy source.txt dest.txt
move source.txt dest.txt
del file.txt
mkdir new-folder
rmdir /s /q folder              :: delete folder recursively

:: Search
findstr "password" file.txt     :: equivalent of grep
findstr /s /i "password" *.txt  :: recursive, case-insensitive
where program.exe               :: locate executable in PATH
```

**PowerShell:**

PowerShell is a significantly more powerful scripting environment built on .NET. It operates on objects rather than text, making output manipulation more structured than CMD.

```powershell
# Navigation and files
Get-ChildItem                   # equivalent of dir/ls
Get-ChildItem -Hidden           # show hidden files
Get-Content file.txt            # equivalent of type/cat
Set-Location C:\Users           # equivalent of cd

# System information
Get-ComputerInfo
Get-WmiObject Win32_OperatingSystem
systeminfo

# Processes
Get-Process
Get-Process | Sort-Object CPU -Descending
Stop-Process -Id 1234 -Force

# Services
Get-Service
Get-Service | Where-Object {$_.Status -eq "Running"}
Start-Service -Name "servicename"
Stop-Service -Name "servicename"

# Network
Get-NetTCPConnection
Get-NetTCPConnection | Where-Object {$_.State -eq "Listen"}
Resolve-DnsName google.com

# Users and groups
Get-LocalUser
Get-LocalGroupMember -Group "Administrators"

# Event logs
Get-WinEvent -LogName Security -MaxEvents 100
Get-WinEvent -LogName System -MaxEvents 50

# Scheduled tasks
Get-ScheduledTask
Get-ScheduledTask | Where-Object {$_.State -eq "Ready"}

# File search
Get-ChildItem -Path C:\ -Recurse -Filter "*.txt" -ErrorAction SilentlyContinue
Select-String -Path C:\*.txt -Pattern "password"    # equivalent of grep
```

**Real-world relevance:** PowerShell is the primary tool for both Windows administration and Windows-based attacks. PowerShell execution policy, script block logging, and transcript logging are key defensive controls — and key attacker targets. PowerShell download cradles (`IEX (New-Object Net.WebClient).DownloadString(url)`) are a primary malware delivery mechanism. In a SOC context, PowerShell command-line logging in Event Viewer (Event ID 4104) is a high-signal detection source.

---

### 5. Windows Registry

The Registry is Windows' centralised configuration database — storing settings for the OS, hardware, applications, and users. It is hierarchical, organised into keys and values.

```
Win+R → regedit
```

**Five root keys (hives):**

| Hive | Abbreviation | Contents |
|---|---|---|
| HKEY_CLASSES_ROOT | HKCR | File associations and COM objects |
| HKEY_CURRENT_USER | HKCU | Settings for the currently logged-in user |
| HKEY_LOCAL_MACHINE | HKLM | System-wide settings — hardware, software, security |
| HKEY_USERS | HKU | Settings for all user profiles |
| HKEY_CURRENT_CONFIG | HKCC | Current hardware profile |

**Security-critical registry locations:**

```cmd
:: Autorun keys — programs that run at startup
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce

:: Services registry entries
HKLM\SYSTEM\CurrentControlSet\Services\

:: UAC settings
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System

:: Installed software
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\

:: Recent files and commands (user activity)
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
```

**Registry command-line interaction:**

```cmd
:: Query a registry key
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run

:: Query specific value
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run /v ProgramName

:: Add a registry value
reg add HKCU\Software\Microsoft\Windows\CurrentVersion\Run /v MyApp /t REG_SZ /d "C:\path\app.exe"

:: Delete a registry value
reg delete HKCU\Software\Microsoft\Windows\CurrentVersion\Run /v MyApp /f

:: Export registry key to file
reg export HKLM\Software\Microsoft\Windows output.reg
```

**Real-world relevance:** The Run and RunOnce keys are the most commonly abused persistence locations in Windows malware. Any executable listed under these keys launches automatically at login. During incident response, querying these keys is a standard persistence-hunting step. The Registry also records user activity — recently accessed files, recently run commands, recently connected USB devices — making it a rich forensic artefact.

---

### 6. Windows Update and Security Features

**Windows Update** keeps the OS and Microsoft software patched against known vulnerabilities. From a security perspective, patch status directly determines exploit exposure.

```
Settings → Windows Update
Win+R → ms-settings:windowsupdate
```

**Checking patch status from command line:**

```cmd
:: List installed updates
wmic qfe list brief /format:table

:: PowerShell
Get-HotFix
Get-HotFix | Sort-Object InstalledOn -Descending
```

**Windows Security (Defender):**

Built-in security tooling available on all modern Windows versions:

| Component | Purpose |
|---|---|
| Virus & Threat Protection | Antivirus — real-time scanning |
| Account Protection | Windows Hello, Dynamic Lock |
| Firewall & Network Protection | Windows Firewall management |
| App & Browser Control | SmartScreen, exploit protection |
| Device Security | Secure Boot, TPM, kernel isolation |
| Device Performance & Health | System health reporting |

```cmd
:: Check Windows Defender status
sc query WinDefend
Get-MpComputerStatus                    :: PowerShell

:: Check firewall status
netsh advfirewall show allprofiles
Get-NetFirewallProfile                  :: PowerShell

:: Check firewall rules
netsh advfirewall firewall show rule name=all
Get-NetFirewallRule | Where-Object {$_.Enabled -eq "True"}   :: PowerShell
```

**Real-world relevance:** `wmic qfe list` and `Get-HotFix` are standard enumeration commands — missing patches map directly to CVEs. An unpatched system with a known remote code execution vulnerability is an immediate critical finding. Windows Defender's real-time protection status is also relevant — malware frequently disables Defender as a first action, so checking its status during incident response confirms whether the system had active protection.

---

## Walkthrough Notes

### Task 1 — Introduction
Brief recap of Part 1, setting the scope for Part 2 — configuration, monitoring, and built-in security tools.

### Task 2 — System Configuration
`msconfig` Tools tab in depth. The exercise involves launching specific tools from the Tools tab and answering questions about system configuration — building familiarity with the tool inventory.

### Task 3 — UAC Settings
UAC levels — from "Always Notify" (highest) to "Never Notify" (disabled). The security implication: lower UAC settings reduce friction for legitimate users and attackers equally. UAC at "Never Notify" means no prompt for any administrative action — effectively no UAC protection.

### Task 4 — Computer Management
`compmgmt.msc` revisited with focus on the Shared Folders section — Active Sessions and Open Files reveal who is currently connected to the machine over the network, and what files they have open. Relevant in incident response for identifying active attacker sessions.

### Task 5 — System Information
`msinfo32` in depth. The Software Environment section — particularly Startup Programs and Running Tasks — is the investigation-focused output.

### Task 6 — Resource Monitor
Network tab as the primary security-relevant view. The exercise involves identifying which process owns a specific network connection — the core skill the tool provides.

### Task 7 — Command Prompt
CMD commands with security relevance. The exercise involves using CMD to retrieve specific system information — building command-line fluency in a Windows context.

### Task 8 — Registry Editor
Registry structure and the Run keys. The exercise involves navigating to specific registry locations and reading values — building familiarity with the tool and the security-critical key locations.

### Task 9 — Windows Updates
Update status and patch management. `wmic qfe list` as the command-line patch enumeration method.

### Task 10 — Windows Security
Windows Defender and built-in security features overview. The exercise involves checking the status of security components and understanding what each one provides.

---

## Commands Reference

```cmd
:: System information
msinfo32 /report C:\temp\sysinfo.txt
winver
systeminfo
wmic qfe list brief /format:table

:: Network investigation
netstat -ano
netstat -ano | findstr LISTENING
ipconfig /all
ipconfig /displaydns

:: Process investigation
tasklist /v
tasklist /svc
taskkill /PID <pid> /F

:: User enumeration
whoami /priv
whoami /groups
net user
net localgroup Administrators

:: Registry
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run

:: Security status
sc query WinDefend
netsh advfirewall show allprofiles
```

```powershell
# System and patch status
Get-ComputerInfo
Get-HotFix | Sort-Object InstalledOn -Descending

# Process and network investigation
Get-Process
Get-NetTCPConnection | Where-Object {$_.State -eq "Listen"}

# Persistence hunting
Get-ScheduledTask | Where-Object {$_.State -eq "Ready"}
Get-LocalGroupMember -Group "Administrators"

# Security status
Get-MpComputerStatus
Get-NetFirewallProfile

# Event logs
Get-WinEvent -LogName Security -MaxEvents 100
```

---

## SOC / Real-World Mapping

| Concept | Real-World Application |
|---|---|
| Registry Run keys | Primary Windows persistence location — check in every investigation |
| `msinfo32` Startup Programs | Secondary persistence enumeration — complements Run key check |
| Resource Monitor Network tab | Process-to-connection mapping — identify C2 by process in real time |
| `wmic qfe list` / `Get-HotFix` | Missing patches → exploitable CVEs → immediate critical findings |
| UAC "Never Notify" | Effectively disabled UAC — any process can elevate without prompt |
| Windows Defender disabled | Malware frequently disables AV as first action — check status in IR |
| PowerShell Event ID 4104 | Script block logging — high-signal source for PowerShell-based attacks |
| Shared Folders Active Sessions | Active attacker network sessions visible here during live incident |
| `whoami /priv` | Current privilege enumeration — identify exploitable privileges (SeImpersonatePrivilege etc.) |
| `ipconfig /displaydns` | Cached DNS reveals recent domain resolutions — useful in C2 investigation |

---

## Takeaways

Three things Windows Fundamentals Part 2 delivers that complete the Windows security foundation:

1. **The Registry is Windows' memory.** It records what runs at startup, what software is installed, what files were recently opened, what commands were recently run, and what USB devices have been connected. In forensic investigation, the Registry is a primary artefact — often more reliable than file system evidence because Registry entries survive file deletion. Knowing the Run keys, the Uninstall keys, and the user activity keys means knowing where to look for both persistence and historical activity.

2. **Resource Monitor's Network tab is a live investigation tool.** In a Windows incident, the question "what process is responsible for this suspicious outbound connection?" is answered in seconds with Resource Monitor — no cross-referencing required. The combination of process name, PID, binary path, and active network connections in one view makes it faster than any command-line equivalent for live triage.

3. **PowerShell is both the most powerful administrative tool and the most abused attack vector on Windows.** Its logging capabilities — script block logging (Event ID 4104), module logging, transcript logging — are the primary defences against PowerShell-based attacks. Whether you're investigating a PowerShell download cradle, auditing scheduled tasks, or enumerating a system post-access, PowerShell fluency determines how fast and how thoroughly you can work. It is the single highest-return Windows skill to develop.

---
