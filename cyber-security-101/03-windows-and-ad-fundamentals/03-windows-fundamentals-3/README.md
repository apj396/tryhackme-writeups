# Windows Fundamentals 3

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Windows and AD Fundamentals |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/windowsfundamentals3xzx |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the third and final room in the Windows Fundamentals series within CS101. Parts 1 and 2 covered the desktop, filesystem, user accounts, administrative utilities, and the Registry — the operational layer of Windows. Part 3 shifts entirely to the security layer: the built-in tools and mechanisms Microsoft provides to protect a Windows system. This includes Windows Update and Patch Tuesday, the Windows Security dashboard, Microsoft Defender Antivirus, Windows Defender Firewall, device security and the Trusted Platform Module (TPM), BitLocker full-disk encryption, and the Volume Shadow Copy Service (VSS). The room also introduces Living Off The Land — the attacker technique of using Windows' own built-in tools to avoid detection — as a closing context for why understanding native Windows security features matters.

---

## Key Concepts

### Windows Update and Patch Tuesday

Windows Update is the Microsoft service responsible for delivering security patches, feature enhancements, and updates for Windows and other Microsoft products including Microsoft Defender. Security patches are typically released on the second Tuesday of each month — a cadence the industry calls Patch Tuesday. Critical vulnerabilities may receive out-of-band patches pushed immediately without waiting for the regular cycle.

Windows Update settings are managed through Settings → Update & Security. On the lab VM, the update settings are managed by policy — meaning a domain administrator controls when updates are applied rather than the local user. The update history — accessible via View Update History — shows all installed updates categorised by type: Quality Updates, Driver Updates, Definition Updates, and Other Updates. Two definition updates were installed in the lab VM on `5/3/2021`.

The command to open Windows Update from the Run dialog or CMD is:

    control /name Microsoft.WindowsUpdate

Unpatched systems are the most common entry point for ransomware and commodity malware campaigns. Patch Tuesday creates a predictable release cadence that attackers monitor — new patches implicitly disclose vulnerabilities, and threat actors reverse-engineer patches to develop exploits targeting organisations that have not yet applied them.

### Windows Security Dashboard

The Windows Security dashboard provides a centralised view of the security status of the device. It is opened via Settings → Update & Security → Windows Security → Open Windows Security, or by searching for it in the Start Menu.

The dashboard uses colour coding to communicate urgency:

| Colour | Meaning |
|---|---|
| Green | Protected — no recommended actions |
| Yellow | Safety recommendation to review |
| Red | Requires immediate attention |

On the lab VM, the area flagged red — requiring immediate attention — is `Virus & threat protection`. Specifically, the setting that is turned off and prompting a notification is `Real-time protection`.

The dashboard covers seven protection areas: Virus & threat protection, Account protection, Firewall & network protection, App & browser control, Device security, Device performance & health, and Family options.

### Microsoft Defender Antivirus

Microsoft Defender Antivirus is the built-in antimalware solution. It is managed through the Virus & threat protection section of the Windows Security dashboard.

Key operational areas:

**Current threats** — shows the results of the most recent scan, any active threats, and options to initiate a scan. Four scan types are available:
- Quick scan — checks the most common locations where threats reside; a useful starting point
- Full scan — examines every file and running program; thorough but time-intensive
- Custom scan — user-specified folders or files
- Offline scan (Microsoft Defender Offline) — reboots the machine and scans before the OS fully loads, targeting malware that hides from running OS-level scans

**Virus & threat protection settings:**

| Setting | Function |
|---|---|
| Real-time protection | Continuously monitors and stops malware from installing or running |
| Cloud-delivered protection | Sends data to Microsoft's cloud for faster detection using up-to-date intelligence |
| Automatic sample submission | Sends suspicious file samples to Microsoft to improve protection for all users |
| Controlled folder access | Protects designated folders against unauthorised modification — a ransomware mitigation |
| Exclusions | Files and folders excluded from scanning — reduces false positives but introduces risk if misused |

**Updates** — Defender uses definition files (signatures) to identify known malware. Keeping definitions current is critical; outdated definitions leave gaps. Definition updates are separate from Windows feature updates and can be applied independently.

### Windows Defender Firewall

The Windows Defender Firewall controls inbound and outbound network traffic based on rules. It is opened via `WF.msc` (Windows Defender Firewall with Advanced Security) or through the Windows Security dashboard. Three profiles determine which ruleset is active based on the network the device is connected to:

| Profile | When Active |
|---|---|
| Domain | When the device can authenticate to a domain controller — typically on a managed corporate network |
| Private | When connected to a network the user has designated as private — home or trusted networks |
| Public | When connected to a public network — Wi-Fi at airports, coffee shops, hotels |

If connected to airport Wi-Fi, the active firewall profile would be `Public`. Each profile has independent rules for which applications and ports can communicate. The Domain profile typically allows more permissive communication for managed applications; the Public profile should be the most restrictive.

The command to open Windows Defender Firewall is `WF.msc`.

### App and Browser Control — SmartScreen and Exploit Protection

**Microsoft Defender SmartScreen** protects against phishing websites, malicious downloads, and unrecognised applications. It produces a warning when a user attempts to run a file or visit a site that has not been seen before or has been reported as malicious. The "Check apps and files" toggle controls whether SmartScreen evaluates unrecognised executables before they run.

**Exploit Protection** is a set of mitigations built into Windows that guard against specific exploit techniques — buffer overflows, heap sprays, ROP chains, and others. It applies both system-wide settings and per-application settings. This feature partially replaces the functionality that was previously available only through third-party tools like EMET (Enhanced Mitigation Experience Toolkit).

### Device Security — Core Isolation and TPM

**Core Isolation** uses hardware virtualisation (Hyper-V) to isolate critical processes from the rest of the operating system in a virtual machine that remains protected in memory. The primary setting within Core Isolation is **Memory Integrity** — when enabled, it prevents malicious code from being inserted into high-security processes by verifying the integrity of kernel code before it executes.

**The Trusted Platform Module (TPM)** is a dedicated cryptographic processor — a chip on the motherboard — designed to provide hardware-based security functions. It stores cryptographic keys in a way that is resistant to software-based extraction, ensures the system has not been tampered with offline, and provides the hardware root of trust that BitLocker and Secure Boot depend on. Systems without a TPM chip (or with TPM version below 1.2) cannot use BitLocker in its standard configuration.

### BitLocker

BitLocker is Windows' full-volume encryption feature, available on Pro, Enterprise, and Education editions. It encrypts the entire drive — operating system, application files, and user data — so that the contents are unreadable without the correct authentication. BitLocker integrates with TPM to verify system integrity before decrypting — if the boot configuration has changed (firmware modification, boot device change), BitLocker will prompt for a recovery key before granting access.

On systems without TPM version 1.2 or later, BitLocker can still be used, but the user must insert a removable drive containing a **startup key** at boot time or when resuming from hibernation. The startup key substitutes for the TPM-based integrity check.

The BitLocker feature is not available on the lab VM, which runs Windows Server without the feature enabled.

### Volume Shadow Copy Service (VSS)

The Volume Shadow Copy Service (VSS) coordinates the creation of consistent point-in-time snapshots of data — called shadow copies or volume snapshots — without taking applications offline. Shadow copies are stored in the `System Volume Information` folder on each protected drive. When System Protection is enabled, VSS can be used to restore previous versions of files or perform a full system restore to a prior snapshot.

From a security perspective, VSS is a double-edged feature. For defenders, shadow copies are a recovery mechanism — if ransomware encrypts files, shadow copies may allow recovery without paying a ransom. For attackers, this is well-known: ransomware routinely includes code specifically designed to delete shadow copies before encrypting files, using commands such as `vssadmin delete shadows /all /quiet`. This makes offline and off-site backups the only reliable recovery option when ransomware is involved.

### Living Off The Land (LotL)

The room closes with a note on Living Off The Land — an attacker technique where built-in Windows tools and utilities are used to perform malicious actions rather than deploying custom tools. Because the activity uses legitimate Windows executables — `cmd.exe`, `powershell.exe`, `wmic.exe`, `certutil.exe`, `mshta.exe` — it blends with normal system activity and evades many detection tools that focus on known malicious files. Understanding the legitimate use of every tool covered in the Windows Fundamentals series is a prerequisite for detecting when those same tools are being abused.

---

## Walkthrough Notes

The room runs through ten tasks. All work is done within the in-browser Windows VM.

**Task 1 (Introduction):** Notes that this room focuses on the security features built into Windows and builds on Parts 1 and 2. No answer required.

**Task 2 (Windows Updates):** Navigating to Settings → Update & Security → View Update History → Definition Updates reveals the date the two definition updates were installed — the answer is `5/3/2021`.

**Task 3 (Windows Security):** Opening the Windows Security dashboard shows the protection area flagged red — the answer is `Virus & threat protection`.

**Task 4 (Virus and Threat Protection):** Within Virus & threat protection, the specific setting that is off and generating a notification is `Real-time protection`.

**Task 5 (Firewall and Network Protection):** Covers the three firewall profiles. The question asking which profile would be active on airport Wi-Fi is answered by `Public network`.

**Task 6 (App and Browser Control):** Covers SmartScreen and Exploit Protection. No standalone question requiring interaction with the VM beyond reading the task content.

**Task 7 (Device Security):** Covers Core Isolation, Memory Integrity, and TPM. The question asking what TPM stands for is answered by `Trusted Platform Module`.

**Task 8 (BitLocker):** Covers full-disk encryption and the TPM dependency. The question asking what a user must insert on computers without TPM version 1.2 or later is answered by `startup key` — found in the linked Microsoft BitLocker documentation.

**Task 9 (Volume Shadow Copy Service):** Covers VSS, shadow copies, and the ransomware implication. The question asking what VSS stands for is answered by `Volume Shadow Copy Service`.

**Task 10 (Conclusion):** Summarises the full Windows Fundamentals series and introduces the Living Off The Land concept as a bridge to more advanced Windows security topics. No answer required.

---

## Commands and Tools Used

    WF.msc                                    # Windows Defender Firewall with Advanced Security
    control /name Microsoft.WindowsUpdate     # Opens Windows Update
    vssadmin list shadows                     # Lists existing shadow copies (CMD, elevated)
    vssadmin delete shadows /all /quiet       # Deletes all shadow copies — common ransomware command

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| Patch Tuesday and update history | Vulnerability management — correlating CVE disclosure dates with an organisation's patch deployment timeline determines exposure windows; the gap between Patch Tuesday and actual deployment is the attack window |
| Real-time protection disabled | Alert triage — a Windows Security alert for disabled real-time protection on an endpoint is a high-priority finding; it may indicate deliberate attacker disabling as a pre-ransomware step |
| Firewall profiles (Domain/Private/Public) | Network security policy — misconfigured profile assignments (e.g. a corporate laptop connecting to a public network using the Domain profile) can expose services that should only be accessible on managed networks |
| VSS shadow copy deletion | Ransomware detection — `vssadmin delete shadows` executed via `cmd.exe` or `powershell.exe` is a near-universal pre-encryption step in ransomware attacks; this command pattern is a high-fidelity detection rule in any EDR or SIEM |
| BitLocker and startup key | Device encryption posture — endpoints without TPM that use BitLocker with a startup key have a physical security dependency; losing or mishandling the USB key bypasses the encryption |
| SmartScreen blocks | Phishing and malware download prevention — SmartScreen blocks are logged and visible in Windows Defender event logs; a pattern of SmartScreen blocks on a specific endpoint indicates a user clicking through suspicious links or downloading untrusted files |
| Living Off The Land (LotL) | EDR tuning — detecting LotL requires behavioural analysis of legitimate tools rather than file hash matching; rules that flag `certutil.exe` downloading files, `mshta.exe` running remote scripts, or `wmic.exe` spawning child processes are examples of LotL detection logic |

---

## Takeaways

1. **VSS shadow copy deletion is one of the clearest pre-ransomware signals available — and it is frequently missed.** The command `vssadmin delete shadows /all /quiet` executes in seconds, produces no user-visible feedback, and completes before encryption begins. A SIEM or EDR rule that fires on this specific command string, or on `wbadmin delete catalog` (which achieves the same outcome), provides an opportunity to intervene before data loss occurs. The challenge is that legitimate backup software also calls these commands — tuning out the false positives while preserving the true positive detection is the practical SOC problem.

2. **Disabling real-time protection is a step, not a destination.** Attackers disable Defender not as an end goal but as preparation — typically before deploying a payload that would otherwise be caught. A real-time protection disabled alert without a corresponding explanation (legitimate IT change, authorised exclusion) is a leading indicator of a compromise in progress. Treating it as routine — "the user probably clicked something" — is the kind of alert fatigue that precedes a significant incident.

3. **Living Off The Land is the reason tool familiarity matters more than tool recognition.** A file hash check will not catch `powershell.exe` being used maliciously because `powershell.exe` is a legitimate, signed Microsoft binary. The detection requires knowing what normal PowerShell activity looks like on a given endpoint and flagging deviations — unusual parent processes, encoded command strings, remote download cradles, or execution from unexpected paths. That baseline awareness begins with understanding what each tool is legitimately for, which is exactly what the Windows Fundamentals series builds.

---
