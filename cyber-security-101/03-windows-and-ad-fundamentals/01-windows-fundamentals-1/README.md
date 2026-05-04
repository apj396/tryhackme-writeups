# Windows Fundamentals 1

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Windows and AD Fundamentals |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/windowsfundamentals1xbx |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the first room in the CS101 Windows and AD Fundamentals section. Where the Linux Fundamentals section built command-line fluency for a system most security tooling runs on, this section builds equivalent familiarity with the operating system that dominates enterprise environments. The majority of corporate endpoints, file servers, Active Directory infrastructure, and the networks analysts are called to defend run Windows. This room covers the Windows desktop, the NTFS file system, user account types, User Account Control, the Control Panel and Settings, and the Task Manager — the foundations needed to navigate and understand a Windows system in a professional context.

---

## Key Concepts

### Windows — History and Editions

Windows has a long history dating to 1985, but the versions relevant to modern enterprise environments begin with Windows XP, which ran for an unusually long time and whose end-of-life announcement caused significant corporate panic as organisations scrambled to migrate. Windows Vista attempted a complete overhaul and was poorly received. Windows 7 was the widely adopted successor. Windows 10 is the current standard for desktops, available in Home and Pro editions.

The distinction between editions matters for security: the Pro edition includes BitLocker full-disk encryption, which Home does not. This difference has practical implications for device security policy in enterprise environments where drive encryption is mandated. The TryHackMe lab machine runs Windows Server, not a desktop edition, but the concepts apply across editions.

The lab machine is accessed via an in-browser virtual machine. It can also be accessed via Remote Desktop Protocol (RDP) using the credentials provided in the room (username: `administrator`, password: `letmein123!`).

### The Windows Desktop and GUI

The Windows desktop provides the graphical interface through which most users interact with the operating system. Key elements relevant to security work:

**The Start Menu** is the primary navigation entry point. Most Windows administration tools can be launched by typing their name into the Start Menu search — `msconfig`, `regedit`, `lusrmgr.msc`, `compmgmt.msc` — without needing to navigate through folder structures. This is the fastest way to access administrative utilities.

**The Taskbar** contains the system tray (Notification Area) in the bottom right corner. Beyond the standard Clock, Volume, and Network icons, a fourth icon — Action Center — is visible in the Notification Area of the lab machine. Action Center consolidates system notifications and quick settings.

**The Task View button** on the taskbar enables virtual desktop management. It can be hidden via right-clicking the taskbar. The Search box adjacent to the Start button can similarly be hidden or shown.

**Task Manager** (`Ctrl+Shift+Esc`) provides a real-time view of running applications, background processes, performance metrics (CPU, RAM, disk, network), and startup programs. It is the primary GUI tool for process inspection and termination on Windows.

### The File System — NTFS

Modern Windows installations use NTFS (New Technology File System). Before NTFS, Windows used FAT16/FAT32 (File Allocation Table) and HPFS (High-Performance File System). FAT partitions are still encountered today — USB drives and microSD cards commonly use FAT32 — but standard Windows system drives use NTFS.

NTFS addresses several limitations of FAT and adds capabilities that matter for security:

| Feature | Description |
|---|---|
| Large file and volume support | Supports volumes and files far exceeding FAT32's 4GB per-file limit |
| Permissions | Fine-grained access control on files and folders — specific users and groups can be granted or denied read, write, execute, and modify permissions independently |
| Journaling | Tracks changes to files before they are committed, enabling recovery from crashes and corruption |
| Encryption (EFS) | Encrypting File System allows file-level encryption directly within NTFS |
| Alternate Data Streams (ADS) | Allows a file to contain more than one stream of data — the primary named stream is what is normally visible; additional streams are hidden and do not appear in directory listings |

Alternate Data Streams are particularly relevant from a security perspective. Malware and attackers use ADS to hide data or executable code within what appears to be an ordinary file — the visible file size and name are unchanged, but additional content is attached. The Windows command `dir /r` reveals ADS on NTFS volumes.

### The Windows Directory and System32

The Windows operating system files are stored in `C:\Windows`. The system variable pointing to this location is `%windir%`. Within the Windows directory, the `System32` folder is the most critical — it holds the files essential for the OS to function. Many of the administrative tools covered in this series reside within System32. Deleting or modifying files in System32 can render the OS inoperable. The system variable pointing to System32 is `%SystemRoot%\system32`.

### User Accounts, Profiles, and Permissions

On a standard local Windows system, accounts are one of two types:

| Account Type | Capabilities |
|---|---|
| Administrator | Full system access — add/delete users, modify groups, install software, change system settings |
| Standard User | Limited to changes within their own user profile — cannot install system-wide software or make system-level configuration changes |

User profiles are stored in `C:\Users`. Each user has their own subdirectory containing their Desktop, Documents, Downloads, AppData, and other personal folders. Local user and group management is handled through `lusrmgr.msc` (Local Users and Groups Management). On the lab machine, running `lusrmgr.msc` reveals a second user account `tryhackmebilly`, which is a member of the `Remote Desktop Users` and `Users` groups. The built-in account for guest access to the computer is the `Guest` account.

### User Account Control (UAC)

UAC was introduced with Windows Vista in response to the security risk of users operating with permanent administrative privileges. The core design: even when logged in as an administrator, the session does not run with elevated permissions by default. When an operation requiring higher privileges is initiated — installing software, modifying system settings — Windows prompts the user to confirm. This prompt is the UAC dialog.

The shield icon on an executable or shortcut indicates that UAC will prompt before running it. UAC does not apply by default to the built-in local administrator account. It can be adjusted via four slider levels from "Always notify" (most secure) to "Never notify" (least secure, not recommended). Disabling UAC entirely removes a meaningful layer of defence against malware that attempts privilege escalation through user-triggered execution.

### Control Panel and Settings

Windows provides two interfaces for system configuration. The Settings menu was introduced in Windows 8 for touch-oriented interaction and covers the most common configuration tasks. The Control Panel is the legacy interface for more complex settings. In practice, some navigation paths start in Settings and end in the Control Panel. The last setting in the Control Panel when viewed in Small icons mode is `Windows Defender Firewall`.

---

## Walkthrough Notes

The room runs through eight tasks, all conducted within the in-browser Windows VM.

**Task 1 (Introduction to Windows):** Brief history of Windows versions from 1985 through Windows 10. No answer required.

**Task 2 (Windows Editions):** Covers the Home vs Pro distinction. The question asks what encryption feature is available in Pro but not in Home — the answer is `BitLocker`.

**Task 3 (The Desktop — GUI):** Covers the desktop, taskbar, Notification Area, Task View, and Search box. The question asking which selection will hide the Search box is answered by `Hidden`. The question asking which selection will hide the Task View button is answered by `Show Task View button` being unchecked. The question asking what other icon is visible in the Notification Area beyond Clock, Volume, and Network is answered by `Action Center`.

**Task 4 (The File System):** Covers NTFS, FAT, ADS, permissions, journaling, and EFS. The question asking what NTFS stands for is answered by `New Technology File System`. The question asking for the system variable for the Windows folder is answered by `%windir%`.

**Task 5 (The Windows\System32 Folder):** Covers System32's role and the risk of modifying it. No standalone question beyond context-based understanding.

**Task 6 (User Accounts, Profiles, and Permissions):** Requires opening `lusrmgr.msc` from the Start Menu on the lab VM. Questions: the name of the other user account is `tryhackmebilly`; the groups this user is a member of are `Remote Desktop Users, Users`; the built-in account for guest access is `Guest`; the account description (visible in the user's Properties) is `window$Fun1!`.

**Task 7 (User Account Control):** Covers UAC's operation, the shield icon, and the four slider levels. The question asks what UAC stands for — the answer is `User Account Control`.

**Task 8 (Settings and the Control Panel):** Covers the Settings menu and Control Panel. The question asking for the last setting in the Control Panel in Small icons view is `Windows Defender Firewall`. The keyboard shortcut to open Task Manager is `Ctrl+Shift+Esc`.

---

## Commands and Tools Used

    lusrmgr.msc          # Local Users and Groups Management
    dir /r               # Reveals NTFS Alternate Data Streams in Command Prompt
    Ctrl+Shift+Esc       # Opens Task Manager directly

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| NTFS Alternate Data Streams | Malware detection — ADS are used to hide malicious payloads or data within seemingly normal files; `dir /r` or Sysinternals Streams can reveal hidden streams during forensic analysis |
| Administrator vs Standard User | Least privilege enforcement — ensuring users operate as Standard rather than Administrator significantly limits the blast radius of phishing or malware execution |
| UAC prompts | Privilege escalation detection — UAC bypass techniques are a common attack category; unexpected elevated process execution without a visible UAC prompt is a detection indicator |
| lusrmgr.msc | Account audit — reviewing local user accounts and group memberships is a standard step in post-compromise host analysis; unexpected accounts or group changes are high-confidence indicators of attacker persistence |
| Task Manager | Process analysis — reviewing running processes for anomalous names, unexpected parent-child relationships, or unusual resource consumption is a first-pass triage step on a potentially compromised Windows host |
| `%windir%` and System32 | Masquerading detection — malware commonly places executables with System32-mimicking names in other directories to appear legitimate; knowing what belongs in System32 and what the path variables resolve to is the baseline for detecting this |

---

## Takeaways

1. **NTFS permissions and ADS represent both a security feature and a security risk.** The permissions model allows fine-grained access control that is foundational to Windows security architecture. But Alternate Data Streams are a feature of the same file system that attackers routinely exploit to hide data in plain sight. The defensive response to both is the same: understand how the feature works at a technical level, because surface-level awareness is not enough to detect abuse.

2. **UAC is a mitigation, not a defence.** It raises the bar for privilege escalation by requiring explicit user confirmation. It does not prevent a determined attacker who has already achieved code execution from escalating further — UAC bypass techniques are well-documented and widely used. Its value is in stopping casual malware and untrained users from making unintended system-level changes. Treating it as a robust defence rather than a speed bump leads to miscalibrated risk assumptions.

3. **The distinction between Administrator and Standard User account types is one of the most impactful access control decisions in a Windows environment.** An organisation where all employees operate as local administrators is an organisation where every phishing email is one click away from full system compromise. The principle of least privilege — users operate with the minimum permissions required for their role — is not just a compliance checkbox. It is a fundamental constraint on how much damage a successful attack can cause.

---
