# Defensive Security Intro

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Start Your Cyber Security Journey |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/defensivesecurityintro |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the second room in the CS101 path and the second in the Start Your Cyber Security Journey section. Where Offensive Security Intro demonstrated how attackers find and exploit weaknesses, this room introduces the other side of the field — the people, teams, and disciplines responsible for preventing, detecting, and responding to those attacks. The room covers the structure and responsibilities of a Security Operations Center (SOC), Threat Intelligence, Digital Forensics and Incident Response (DFIR), and Malware Analysis. It closes with a hands-on simulation of a SOC analyst's workflow — triaging alerts in a simplified SIEM dashboard, investigating a suspicious IP, escalating the finding, and blocking the threat.

---

## Key Concepts

### Offensive vs Defensive Security

Offensive security focuses on one thing: breaking into systems. It achieves this through exploiting bugs, abusing insecure configurations, and taking advantage of insufficient access controls. Red teams and penetration testers specialise in offensive security.

Defensive security is concerned with two primary objectives: preventing intrusions before they occur, and detecting and responding to intrusions when prevention fails. The professionals operating in this space are collectively known as the blue team. Their day-to-day responsibilities span a wide range — user security awareness training, asset management, patch and vulnerability management, security monitoring, and incident response.

The two sides are not opposites in the way the names imply. Understanding how attackers operate is a prerequisite for building effective defences. The best defenders think like attackers. The best attackers understand what defenders monitor. The distinction is primarily one of role and mandate, not of knowledge or mindset.

### Security Operations Center (SOC)

A SOC is a centralised team of cyber security professionals whose primary function is monitoring an organisation's networks and systems around the clock for malicious activity. When the room asks what you call a team that monitors a network for malicious events, the answer is Security Operations Center.

The SOC's responsibilities extend beyond monitoring:

| Responsibility | Description |
|---|---|
| Vulnerability management | Identifying unpatched systems and ensuring known vulnerabilities are remediated before they are exploited |
| Policy enforcement | Detecting and responding to violations of security policy — unauthorised data sharing, unapproved software, access to restricted resources |
| Threat detection | Identifying unauthorised access attempts, credential misuse, and anomalous network behaviour |
| Threat intelligence | Consuming information about adversary tactics, techniques, and procedures to inform detection logic and response priorities |

The SIEM — Security Information and Event Management — is the primary tool that makes SOC operations scalable. A SIEM aggregates security-related logs and events from across the environment — endpoints, network devices, authentication systems, applications — and presents them in a unified dashboard. Detection rules run against this aggregated data and generate alerts when suspicious patterns are identified. Not every alert represents a genuine threat; analyst judgment is required to triage, investigate, and determine whether an alert warrants escalation or can be dismissed.

### Threat Intelligence

Threat intelligence is the process of collecting, analysing, and acting on information about adversaries — their motivations, capabilities, infrastructure, and tactics. The goal is not just to understand what happened in the past but to anticipate what is likely to happen next and to build detections accordingly.

Sources of threat intelligence include: open-source databases such as AbuseIPDB and Cisco Talos Intelligence for IP reputation data; commercial threat feeds providing indicators of compromise (IOCs); government advisories; and information sharing communities within specific industries. In the practical component of this room, checking a suspicious IP against AbuseIPDB and Cisco Talos Intelligence is the investigation step — these tools allow analysts to quickly establish whether an IP has been previously associated with malicious activity.

### Digital Forensics and Incident Response (DFIR)

DFIR is the discipline that takes over when a security incident has been confirmed. DFIR stands for Digital Forensics and Incident Response — two related but distinct functions that are commonly combined in practice.

**Digital Forensics** is the application of scientific investigation methods to digital systems. In a security context, forensic analysis focuses on understanding the scope and nature of an attack after it has occurred. Key areas of investigation include:

| Area | What It Reveals |
|---|---|
| File system analysis | Installed programs, created files, modified files, deleted files recoverable from unallocated space |
| System memory analysis | Running processes at the time of incident, loaded malware, encryption keys held in memory |
| System and network logs | Timeline of attacker actions, lateral movement, data accessed or exfiltrated |

**Incident Response** is the structured process of managing a confirmed security incident to minimise damage, reduce recovery time, and prevent recurrence. The four key phases of the incident response lifecycle are: Preparation (building the team and tooling before an incident occurs), Detection and Analysis (identifying that an incident has occurred and understanding its scope), Containment and Eradication (stopping the spread and removing the attacker's access), and Recovery (restoring systems to normal operation).

### Malware Analysis

Malware analysis is the discipline of understanding what a malicious program does — its capabilities, persistence mechanisms, command and control infrastructure, and impact on infected systems. Two primary approaches exist:

**Static analysis** examines the malicious program without executing it. This requires knowledge of assembly language and binary formats — the analyst inspects the code and structure of the file to infer its behaviour without running it. Static analysis is safe but requires significant expertise.

**Dynamic analysis** executes the malware in a controlled, isolated environment — a sandbox — and observes its behaviour in real time: what files it creates, what registry keys it modifies, what network connections it attempts to establish, what processes it spawns. Dynamic analysis is more accessible but requires careful environment isolation to prevent the malware from escaping the controlled environment. The malware type that requires the victim to pay money to regain access to their files is ransomware.

---

## Walkthrough Notes

The room runs through four tasks covering defensive security concepts and one practical SIEM simulation task.

**Task 1 (Introduction to Defensive Security):** Defines defensive security and frames its two primary objectives — prevention and detection and response. Lists the day-to-day responsibilities of blue team professionals. The key question asks what you call a team that monitors a network for malicious events — the answer is Security Operations Center.

**Task 2 (Areas of Defensive Security):** Covers the four sub-disciplines — SOC, Threat Intelligence, DFIR, and Malware Analysis — at an introductory level. Key questions: DFIR stands for Digital Forensics and Incident Response. The malware type that requires payment to regain access to files is ransomware.

**Task 3 (Practical Example of Defensive Security — SIEM):** The interactive SIEM simulation is launched via the View Site button. The scenario: you are a SOC analyst at a bank. Five alert logs appear in the SIEM dashboard. One is highlighted in red — an unauthorised connection attempt from IP `143.110.250.149` to port 22 (SSH). The workflow follows a realistic triage process:

Step 1 — Identify the alert: locate the red-highlighted alert in the dashboard and note the source IP `143.110.250.149`.

Step 2 — Investigate the IP: check the IP against AbuseIPDB or Cisco Talos Intelligence. Both confirm the IP is flagged as malicious and associated with previous attack activity.

Step 3 — Escalate: escalate the confirmed malicious alert to the appropriate SOC staff member for approval to act.

Step 4 — Block: add the malicious IP to the firewall block list to prevent further connection attempts.

The flag returned upon completing all four steps is `THM{THREAT-BLOCKED}`.

**Task 4 (Conclusion):** Summarises the room and points to follow-on rooms for each sub-discipline — SOC Fundamentals, DFIR Introduction, Intro to Malware Analysis. No answer required.

---

## Commands Used

No command-line tools used in this room. The practical component operates entirely through the interactive SIEM simulation accessed via the View Site button.

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| SIEM alert triage | Core SOC analyst daily workflow — the simulation mirrors the genuine process of reviewing a dashboard, identifying anomalous events, and making triage decisions under time pressure |
| IP reputation lookup (AbuseIPDB, Cisco Talos) | Alert investigation — cross-referencing source IPs against threat intelligence databases is a standard first step in assessing whether an alert represents a genuine threat |
| Alert escalation workflow | Incident management — SOC analysts rarely act alone; confirmed findings are escalated to senior analysts or incident response teams for approval before remediation actions are taken |
| Firewall block as remediation | Tactical containment — adding a malicious IP to a block list is a first-line containment measure; it stops the immediate threat but does not address how the attacker obtained the target, what else they may have done, or whether they have other infrastructure |
| Ransomware as a malware category | Threat landscape awareness — ransomware remains the dominant monetisation mechanism for cybercriminal organisations; understanding its classification within malware analysis is foundational for SOC alert categorisation |
| DFIR four-phase lifecycle | Incident response planning — Preparation, Detection and Analysis, Containment and Eradication, Recovery maps directly to how organisations structure their IR playbooks and tabletop exercises |

---

## Takeaways

1. **The SIEM simulation compresses a real workflow into four steps — and that compression is instructive.** In a genuine SOC, each of those steps has significant depth: alert review involves prioritisation across dozens of concurrent events, IP investigation involves correlating multiple intelligence sources, escalation involves documenting findings clearly for a senior analyst, and blocking involves verifying the action will not disrupt legitimate traffic. The simulation teaches the shape of the workflow. The depth comes from experience. Recognising the difference between knowing the steps and being able to execute them under operational conditions is the starting point for taking defensive security seriously.

2. **Threat intelligence transforms reactive defence into proactive defence.** Without intelligence, a SOC can only respond to what it sees. With intelligence — knowledge of adversary infrastructure, common attack patterns, and emerging campaigns — it can build detections before the attack arrives, identify low-signal indicators that would otherwise be dismissed as noise, and prioritise patching against vulnerabilities that are actively being exploited in the wild. The IP lookup step in the simulation is the simplest possible version of this. At scale, threat intelligence is one of the most complex and highest-value functions in a mature security programme.

3. **Blocking a malicious IP is containment, not remediation.** The simulation ends with `THM{THREAT-BLOCKED}` — the alert is resolved. In a real environment, that is where the investigation begins. How did the attacker identify the target? Was the SSH connection attempt successful before it was detected? Are there other IPs in the same infrastructure that have not yet triggered an alert? What does the attacker want? Treating containment as the end of the process is a common gap in immature SOC operations — and the gap attackers exploit to maintain persistence after their initial indicators are blocked.

---
