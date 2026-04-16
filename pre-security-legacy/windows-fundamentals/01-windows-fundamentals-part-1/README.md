# Windows Fundamentals Part 1

**Module:** Pre-Security (Legacy) → Windows Fundamentals  
**Difficulty:** Easy  
**Platform:** TryHackMe  
**Room Link:** https://tryhackme.com/room/windowsfundamentals1xbx  
**Date Completed:** January 2026  
**Author:** Adwait Joshi

---

## What This Room Covers

This room introduces Windows as an operating system from a security perspective — the desktop environment, filesystem structure, user account management, and the system configuration tools that matter most in security work. It uses a remote Windows desktop (RDP-style in-browser) as the practical environment.

Linux gets more attention in security tooling, but Windows is the dominant enterprise operating system. Understanding Windows internals is prerequisite to understanding the majority of real-world attacks — Active Directory exploitation, lateral movement, ransomware deployment, and privilege escalation all happen primarily on Windows infrastructure.

---

## Key Concepts

### 1. Windows Editions and Versions

Windows exists in multiple editions targeting different use cases:

| Edition | Target |
|---|---|
| Windows Home | Consumer — personal use |
| Windows Pro | Business users — adds BitLocker, RDP, Hyper-V |
| Windows Enterprise | Large organisations — advanced security features |
| Windows Server | Server environments — Active Directory, DNS, DHCP |

**Version history context:** Windows 10 and Windows 11 are the current desktop versions. Windows Server 2019 and 2022 are current server versions. Older versions — Windows 7, Server 2008 — still exist in enterprise environments and are prime targets due to end-of-life status and unpatched vulnerabilities.

**Real-world relevance:** EternalBlue (MS17-010) — the exploit behind WannaCry ransomware — targeted Windows 7 and Server 2008 systems. End-of-life Windows versions in production networks are consistently the highest-risk assets in a network. Identifying OS version during reconnaissance directly informs exploit selection.

---

### 2. The Windows Desktop and GUI

The Windows graphical environment has several components relevant to security investigation:

**Desktop** — the primary workspace. Shortcuts, files, and the taskbar.

**Start Menu** — application launcher and search. `Win+R` opens the Run dialog — a direct way to launch tools and access system utilities.

**Taskbar** — running applications, system tray (background processes), clock.

**File Explorer** — the GUI file manager, equivalent to the Linux file manager. Shows drives, folders, and files.

**Task Manager (`Ctrl+Shift+Esc`)** — process viewer, performance monitor, startup manager. The Windows equivalent of `ps aux` and `top`.

```
Win+R shortcuts — launch system tools directly:
cmd          → Command Prompt
powershell   → PowerShell
msconfig     → System Configuration
regedit      → Registry Editor
services.msc → Services Manager
eventvwr     → Event Viewer
taskschd.msc → Task Scheduler
compmgmt.msc → Computer Management
```

**Real-world relevance:** `Win+R` shortcuts are how experienced Windows users navigate system tools quickly — and how attackers do the same after gaining access. Knowledge of these shortcuts is practical both offensively (moving around a compromised Windows system efficiently) and defensively (investigating a Windows system during an incident).

---

### 3. The Windows Filesystem — NTFS

Windows uses NTFS (New Technology File System) as its primary filesystem. Understanding NTFS is foundational to understanding Windows security.

**Key filesystem locations:**

| Path | Purpose |
|---|---|
| `C:\` | Root of the primary drive |
| `C:\Windows\` | Operating system files |
| `C:\Windows\System32\` | Core system binaries and DLLs |
| `C:\Windows\SysWOW64\` | 32-bit binaries on 64-bit systems |
| `C:\Program Files\` | 64-bit installed applications |
| `C:\Program Files (x86)\` | 32-bit installed applications |
| `C:\Users\` | User home directories |
| `C:\Users\username\Desktop\` | User's desktop |
| `C:\Users\username\Documents\` | User's documents |
| `C:\Users\username\AppData\` | Application data (hidden by default) |
| `C:\Temp\` or `C:\Windows\Temp\` | Temporary files |
| `C:\ProgramData\` | Application data shared across users |

**NTFS-specific features:**

**Permissions** — NTFS supports granular access control via ACLs (Access Control Lists), more complex than Linux's rwx model:

| Permission | Effect |
|---|---|
| Full Control | Read, write, execute, delete, change permissions |
| Modify | Read, write, execute, delete |
| Read & Execute | Read and run files |
| Read | View file contents and attributes |
| Write | Create and modify files |

**Alternate Data Streams (ADS)** — NTFS allows files to contain multiple data streams. A file can have hidden content attached that doesn't appear in normal directory listings.

```cmd
# Create a file with an alternate data stream
echo "hidden content" > file.txt:hidden_stream

# View alternate data streams
dir /r file.txt

# Read an alternate data stream
more < file.txt:hidden_stream
```

**Real-world relevance:** ADS is used by malware to hide payloads — a malicious executable stored in an ADS of an innocuous file won't appear in normal file listings. `dir /r` reveals streams. Windows Defender and most EDR solutions now detect ADS-based hiding, but the technique appears in CTFs and older malware samples. NTFS permissions misconfigurations — writable `C:\Windows\System32\` paths, weak ACLs on service binaries — are privilege escalation vectors.

---

### 4. User Accounts and Groups

Windows manages access through user accounts and groups, similar in concept to Linux but with different terminology and tooling.

**Account types:**

| Type | Description |
|---|---|
| Administrator | Full system control |
| Standard User | Limited — cannot install software or modify system settings |
| Guest | Minimal access — largely deprecated |
| System | Built-in account for OS processes — highest privilege |
| SYSTEM | The most privileged account — above Administrator |

**Built-in accounts:**

- **Administrator** — the built-in local admin account (disabled by default in modern Windows)
- **SYSTEM** — used by the OS itself; target of privilege escalation
- **Local Service / Network Service** — limited privilege accounts for services

**User management — GUI:**

```
Control Panel → User Accounts
Computer Management → Local Users and Groups
Settings → Accounts
```

**User management — command line:**

```cmd
# List local users
net user

# List local groups
net localgroup

# Show specific user details
net user username

# List members of Administrators group
net localgroup Administrators

# Add user (requires admin)
net user newuser password123 /add

# Add user to Administrators group
net localgroup Administrators newuser /add
```

**Real-world relevance:** `net user` and `net localgroup Administrators` are among the first commands run after gaining access to a Windows system — confirming the current user context and enumerating who has administrative access. Finding unexpected accounts in the Administrators group is a significant finding in both assessments and incident response. The built-in Administrator account being enabled with a weak or default password is a consistent finding in internal network assessments.

---

### 5. System Configuration — msconfig

`msconfig` (System Configuration) is a utility for managing Windows startup behaviour, boot options, and services.

```
Win+R → msconfig
```

**Tabs:**

| Tab | Purpose |
|---|---|
| General | Startup selection — Normal, Diagnostic, Selective |
| Boot | Boot options — Safe Mode, boot logging |
| Services | List and enable/disable Windows services |
| Startup | Manage startup programs (redirects to Task Manager in Win 10/11) |
| Tools | Shortcuts to system administration tools |

**The Tools tab** is particularly useful — it provides direct launch buttons for:
- Event Viewer
- Performance Monitor
- Registry Editor
- Command Prompt
- System Information
- Internet Options

**Real-world relevance:** `msconfig` Services tab shows all registered Windows services including their state. In a post-access enumeration context, unexpected services — especially those running as SYSTEM with unusual binary paths — are privilege escalation candidates. In incident response, services installed by malware frequently appear here.

---

### 6. Computer Management

Computer Management (`compmgmt.msc`) is a central administrative console consolidating multiple management tools:

```
Win+R → compmgmt.msc
```

**Key sections:**

**Task Scheduler** — Windows equivalent of cron. Scheduled tasks run commands automatically at specified times or triggers.

```cmd
# List scheduled tasks from command line
schtasks /query /fo LIST /v
schtasks /query /fo LIST /v | findstr "Task Name\|Run As User\|Task To Run"
```

**Event Viewer** — Windows centralised logging system. Every security-relevant event is logged here.

**Shared Folders** — network shares, active sessions, and open files.

**Local Users and Groups** — user and group management.

**Device Manager** — hardware management.

**Disk Management** — partition and volume management.

**Real-world relevance:** Task Scheduler is the Windows equivalent of cron — and the same persistence mechanism. Attackers create scheduled tasks to maintain access. Enumerating scheduled tasks during an investigation or assessment is mandatory. Event Viewer is the primary Windows log source — the Windows equivalent of `/var/log/`.

---

### 7. System Information

Multiple tools provide system information useful for both administration and enumeration:

```cmd
# System information — comprehensive
systeminfo

# Key output fields:
# OS Name, OS Version — operating system and patch level
# System Type — x64 or x86
# Domain — whether machine is domain-joined
# Hotfix(s) — installed patches

# Environment variables
set                             # list all environment variables
echo %USERNAME%                 # current user
echo %COMPUTERNAME%             # machine name
echo %OS%                       # operating system
echo %PATH%                     # executable search path
echo %TEMP%                     # temp directory path

# Check Windows version
winver                          # GUI version dialog
ver                             # command line version
```

**Real-world relevance:** `systeminfo` output is the Windows equivalent of `uname -a` combined with patch enumeration. OS version and installed hotfixes directly inform exploit research — a system missing specific patches is vulnerable to specific CVEs. Domain membership (`Domain:` field) indicates whether the machine is part of an Active Directory environment — significantly expanding the attack surface and investigation scope.

---

## Walkthrough Notes

### Task 1 — Introduction to Windows
Historical context and edition overview. The key takeaway: Windows dominates enterprise environments, making it the primary target environment for the majority of real-world attacks.

### Task 2 — Windows Editions
Edition differences and version history. The security implication of end-of-life versions in production is the carry-forward point.

### Task 3 — The Desktop (GUI)
Orientation to the Windows desktop environment. The `Win+R` shortcut inventory is the practical output — direct access to system tools without navigating menus.

### Task 4 — The File System
NTFS structure, key directory paths, and Alternate Data Streams. The `dir /r` technique for revealing ADS is the security-specific detail worth retaining.

### Task 5 — The Windows\System32 Folder
`C:\Windows\System32\` as the home of core Windows binaries — `cmd.exe`, `powershell.exe`, `notepad.exe`, and thousands of DLLs. Also home to many built-in security tools: `sfc.exe` (System File Checker), `cipher.exe` (encryption), `netstat.exe`.

### Task 6 — User Accounts, Profiles, and Permissions
Account types, built-in accounts, and NTFS permissions. The SYSTEM account being above Administrator in the privilege hierarchy is the key detail — privilege escalation often targets SYSTEM, not just Administrator.

### Task 7 — User Account Control (UAC)
UAC is the Windows mechanism that prompts for confirmation or credentials when administrative actions are requested — the popup that asks "Do you want to allow this app to make changes?"

UAC bypass is a well-documented attack class — techniques that trigger administrative actions without generating the UAC prompt, silently elevating privileges. Understanding what UAC is explaining why bypassing it matters.

### Task 8 — Settings and the Control Panel
Two parallel configuration interfaces in Windows — the modern Settings app and the legacy Control Panel. Both remain relevant; many administrative functions still only exist in Control Panel.

### Task 9 — Task Manager
`Ctrl+Shift+Esc` — the essential Windows process viewer. Tabs: Processes, Performance, App History, Startup, Users, Details, Services. The Details tab shows PIDs and the full binary path — the Windows equivalent of `/proc/<PID>/exe` verification.

---

## Commands Reference

```cmd
:: User enumeration
net user
net user <username>
net localgroup
net localgroup Administrators

:: System information
systeminfo
ver
winver
set
echo %USERNAME%
echo %COMPUTERNAME%

:: Scheduled tasks
schtasks /query /fo LIST /v

:: Alternate Data Streams
dir /r <file>
more < <file>:<stream>

:: Process management (CMD)
tasklist
tasklist /v
taskkill /PID <pid>
taskkill /IM process.exe /F

:: Network
netstat -ano
netstat -ano | findstr LISTENING

:: Services
sc query
sc query type= all
net start
```

```powershell
# PowerShell equivalents
Get-LocalUser
Get-LocalGroupMember Administrators
Get-Process
Get-Service | Where-Object {$_.Status -eq "Running"}
Get-ScheduledTask
Get-WinEvent -LogName Security -MaxEvents 50
systeminfo
```

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| End-of-life Windows versions | Highest-risk assets — unpatched CVEs, no vendor support |
| SYSTEM account | Primary privilege escalation target — above Administrator |
| UAC bypass | Silent privilege escalation — no user prompt generated |
| Alternate Data Streams | Malware hiding technique — `dir /r` for detection |
| `net localgroup Administrators` | Unexpected members = backdoor accounts or compromise indicator |
| Task Scheduler | Persistence mechanism — attacker-created scheduled tasks |
| `systeminfo` hotfixes | Missing patches map directly to exploitable CVEs |
| Domain membership | Domain-joined = Active Directory environment = expanded attack surface |
| NTFS ACL misconfigurations | Writable service binaries = privilege escalation path |
| `netstat -ano` | Active connection enumeration — identify C2 connections by PID |

---

## Takeaways

Three things Windows Fundamentals Part 1 establishes that carry forward into every Windows-focused security topic:

1. **SYSTEM is the goal, not Administrator.** In Windows privilege escalation, the target is usually SYSTEM — the OS-level account that runs core processes and is above even the local Administrator in the privilege hierarchy. Understanding that this account exists, what it's used for, and why attackers target it frames every Windows privilege escalation technique that follows.

2. **Scheduled tasks are the Windows cron — and the Windows persistence mechanism.** Just as cron job enumeration is mandatory on Linux, scheduled task enumeration is mandatory on Windows. `schtasks /query` reveals every scheduled task, its trigger, and the account it runs as. Attacker-created tasks running malicious binaries as SYSTEM are a recurring finding in incident reports.

3. **The Windows filesystem has attack surface the Linux filesystem doesn't.** Alternate Data Streams, DLL search order hijacking (exploiting how Windows finds DLLs), writable paths in `%PATH%`, and weak NTFS ACLs on service binaries are Windows-specific attack surfaces. Understanding the filesystem layout — where binaries live, where temp files go, what `System32` contains — is the foundation for understanding these attack classes.

---
