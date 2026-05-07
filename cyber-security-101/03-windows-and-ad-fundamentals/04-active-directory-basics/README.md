# Active Directory Basics

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Windows and AD Fundamentals |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/winadbasics |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the fourth and final room in the CS101 Windows and AD Fundamentals section. The previous three rooms built familiarity with Windows as a standalone operating system. This room introduces Active Directory — the identity and access management infrastructure that governs the majority of enterprise networks. Understanding Active Directory is not optional for a security analyst. It is the system that controls who can log in where, what they can access, what policies their machine enforces, and how authentication is performed across an organisation. It is also the primary target for privilege escalation and lateral movement in most real-world attacks on corporate environments. This room covers Windows domains, Active Directory structure, user and group management, Group Policy Objects, authentication protocols, and the domain topology concepts of trees, forests, and trusts.

---

## Key Concepts

### Windows Domains and Active Directory

A Windows domain is a group of users and computers under the administration of a given business, managed through a centralised repository called Active Directory (AD). Without a domain, each computer would need to be configured and managed independently — user accounts created locally, policies set per-machine, credentials stored on each device. At scale, this is unmanageable. A domain solves this by centralising identity, policy, and access control.

The server responsible for running Active Directory Domain Services (AD DS) is called a Domain Controller (DC). All credentials in a domain are stored on the Domain Controller, not on individual machines. When a user attempts to log in to any domain-joined computer, the authentication request is sent to the DC for verification.

The core Active Directory data store is the `NTDS.dit` database, stored in `%SystemRoot%\NTDS` on the Domain Controller. This file contains all AD objects — user accounts, group memberships, computer accounts, and crucially, password hashes for every domain user. It is the highest-value target on any domain-joined network; obtaining and cracking `NTDS.dit` yields the credentials of every user in the organisation.

The practical benefits of centralised AD management:

| Benefit | Description |
|---|---|
| Centralised identity management | User accounts created once in AD apply across all domain-joined machines |
| Security policy enforcement | Policies configured in AD apply to users and computers across the network without per-machine configuration |
| Access control | Resource permissions (file shares, printers, applications) are managed through AD groups rather than per-user assignments on each machine |

### AD DS Data Store and Objects

Active Directory stores information about every object in the network. Objects include users, groups, machines, printers, shares, and many others.

**Users** represent people or services. Service accounts — user objects created to run specific applications or services — are common and frequently misconfigured, often with excessive privileges. Users are the most common object type that attackers target for credential theft and impersonation.

**Machines** represent every computer that has joined the domain. Machine accounts end with a dollar sign by convention (`DESKTOP-01$`) and have their own passwords, managed automatically by Windows. Machine accounts can be used for lateral movement if an attacker compromises the machine.

**Security Groups** define collections of users and machines for the purpose of assigning permissions. Rather than granting a user direct access to a resource, access is granted to a group and the user is added to that group. Key default groups:

| Group | Scope |
|---|---|
| Domain Admins | Administrative privileges across the entire domain |
| Server Operators | Can administer Domain Controllers but cannot change group memberships |
| Backup Operators | Can bypass file permissions to perform backups; can log into DCs |
| Account Operators | Can create and modify most user accounts |
| Domain Users | All user accounts in the domain |
| Domain Computers | All workstations and servers joined to the domain |
| Domain Controllers | All Domain Controllers in the domain |

### Organisational Units (OUs)

Organisational Units are container objects within AD used to organise users, groups, computers, and other objects into logical groups. They serve two primary purposes: applying Group Policy Objects (GPOs) to subsets of domain objects, and delegating administrative control over a subset of the domain to specific users without granting domain-wide admin rights.

OUs are structured hierarchically. A GPO applied to an OU automatically applies to all objects within that OU and any sub-OUs beneath it. When a new department needs to be grouped so policies can be applied consistently, an OU is the correct container type — not a security group.

By default, OUs are protected against accidental deletion. To delete an OU in the Active Directory Users and Computers console, Advanced Features must first be enabled in the View menu, after which the "Protect object from accidental deletion" option can be unchecked in the OU's Properties before deletion proceeds.

**Delegation** is the process of granting specific users or groups administrative control over a particular OU without elevating them to Domain Admin. For example, a Helpdesk team can be delegated the ability to reset passwords in the `Sales` OU without having any permissions outside it. Delegation is the mechanism that makes the principle of least privilege practical in large AD environments.

### Group Policy Objects (GPOs)

Group Policy Objects are collections of settings that can be applied to OUs to enforce a configuration baseline on the users and computers within them. GPOs are managed through the Group Policy Management tool accessible from the Start Menu on a Domain Controller.

GPOs are created under the Group Policy Objects container and then linked to one or more OUs. A single GPO can be linked to multiple OUs; a single OU can have multiple GPOs applied. GPOs contain two configuration sections:

| Section | Applies To |
|---|---|
| Computer Configuration | Settings applied to computer objects when they start up, regardless of which user logs in |
| User Configuration | Settings applied to user objects when they log in, regardless of which computer they use |

GPOs are distributed to domain-joined machines via a network share called `SYSVOL`, stored in `C:\Windows\SYSVOL\sysvol\` on each Domain Controller. All domain users need access to SYSVOL over the network to periodically sync their GPOs. GPOs can be forced to sync immediately using:

    gpupdate /force

**Practical GPO examples from the room:**

The first GPO task requires restricting Control Panel access for all users except the IT department. A new GPO named `Restrict Control Panel Access` is created, the `Prohibit access to Control Panel and PC settings` policy is enabled under User Configuration → Administrative Templates → Control Panel, and the GPO is linked to the Management, Marketing, and Sales OUs — but not to IT.

The second GPO task requires automatically locking workstations and servers after 5 minutes of inactivity. A new GPO named `Auto Lock Screen` is created, and the Machine Inactivity Limit is set to 300 seconds under Computer Configuration → Policies → Windows Settings → Security Settings → Local Policies → Security Options. This GPO is linked to the root domain so it applies to all computers.

### Authentication — Kerberos and NetNTLM

When a user authenticates to a service in a Windows domain, one of two protocols handles the credential verification.

**Kerberos** is the default authentication protocol in all modern Windows domains. It is ticket-based: instead of transmitting the user's password over the network, Kerberos issues cryptographic tickets that prove prior authentication. The process:

1. The user sends their username and a timestamp encrypted with a key derived from their password to the Key Distribution Center (KDC), which runs on the Domain Controller
2. The KDC verifies the request and issues a **Ticket Granting Ticket (TGT)** — proof that the user has authenticated
3. When the user wants to access a specific service, they present their TGT to the KDC and request a **Ticket Granting Service (TGS)** ticket for that service
4. The TGS is presented to the target service, which verifies it and grants access

The user's password is never transmitted over the network at any point in this process. The current version of Windows does not use NetNTLM as the preferred authentication protocol by default — Kerberos is preferred.

**NetNTLM** is a legacy authentication protocol retained for compatibility with older systems and applications that do not support Kerberos. It uses a challenge-response mechanism: the server sends a random challenge, the client encrypts it using a hash derived from the user's password, and the server verifies the response against the stored hash. The user's password is never transmitted directly — but the hash used in the response can be captured and subjected to offline cracking or relay attacks. NetNTLM is the basis for Pass-the-Hash and NTLM relay attacks, which remain widely used in real-world assessments.

### Trees, Forests, and Trust Relationships

As organisations grow and acquire other companies, a single domain may not be sufficient. Active Directory supports multi-domain topologies through trees and forests.

**Trees** are collections of domains that share the same namespace. If `thm.local` expands to cover UK and US operations, the tree might consist of `thm.local` as the root with `uk.thm.local` and `us.thm.local` as subdomains. Each subdomain has its own Domain Controller and its own AD, but they share the root namespace.

**Forests** are collections of trees with different namespaces joined into the same network. If `thm.local` acquires a company running `mht.local`, neither can be a subdomain of the other — they form separate trees, and the union of those trees is a forest.

When multiple domains exist in a tree or forest, a new group becomes relevant: **Enterprise Admins**, which grants administrative privileges across all domains in the enterprise. Domain Admins retain their authority within their own domain; Enterprise Admins span everything.

**Trust Relationships** allow users in one domain to access resources in another. Without a trust relationship, domain boundaries are hard walls — a user in `thm.local` cannot authenticate to a resource in `mht.local`.

| Trust Type | Behaviour |
|---|---|
| One-way trust | Domain AAA trusts Domain BBB — users in BBB can access resources in AAA, but not vice versa |
| Two-way trust | Both domains mutually authorise users from the other |

By default, joining domains under a tree or forest creates a two-way trust. What should be configured between two domains for a user in Domain A to access a resource in Domain B is a trust relationship.

---

## Walkthrough Notes

The room runs through nine tasks. All practical work is performed on the in-browser Windows Server VM, which has AD DS installed and a pre-configured domain (`thm.local`).

**Task 1 (Introduction):** Introduces Windows domains and Active Directory. In a Windows domain, credentials are stored in a centralised repository called `Active Directory`. The server in charge of running Active Directory services is called a `Domain Controller`.

**Task 2 (Active Directory):** Covers AD DS objects, users, machines, and security groups. Questions: what type of container is used to group users so that policies can be applied consistently to a new department — the answer is `Organisational Unit (OU)`. Deleting a protected OU requires enabling Advanced Features and unchecking the accidental deletion protection.

**Task 3 (Managing Users in AD):** Practical task using Active Directory Users and Computers on the VM. The task involves creating user accounts, moving users between OUs, and delegating password reset permissions to a Helpdesk user for a specific OU. Delegation is done by right-clicking the target OU and selecting "Delegate Control", then specifying the user and the specific permission to grant.

**Task 4 (Managing Computers in AD):** Covers machine accounts and the convention of organising workstations, servers, and Domain Controllers into separate OUs for targeted GPO application.

**Task 5 (Group Policies):** Practical GPO tasks on the VM. Task 1 requires creating `Restrict Control Panel Access` GPO and linking it to Management, Marketing, and Sales OUs. Task 2 requires creating `Auto Lock Screen` GPO and setting Machine Inactivity Limit to 300 seconds, linked to the root domain. GPOs are distributed via the SYSVOL share. The question asks for the name of the network share used to distribute GPOs — the answer is `SYSVOL`. GPOs can be immediately synced with `gpupdate /force`.

**Task 6 (Authentication Methods):** Covers Kerberos and NetNTLM. Questions: will a current version of Windows use NetNTLM as the preferred protocol by default — `nay`. When referring to Kerberos, the type of ticket that allows requesting further service tickets (TGS) is a `Ticket Granting Ticket (TGT)`. When using NetNTLM, is the user's password transmitted over the network at any point — `nay`.

**Task 7 (Trees, Forests and Trust Relationships):** Covers multi-domain topologies. Questions: a group of Windows domains sharing the same namespace is called a `tree`; what should be configured between two domains for a user in Domain A to access a resource in Domain B is a `trust relationship`.

**Task 8 (Conclusion):** Summarises the room and notes that AD is one of the most attack-relevant systems in enterprise environments. No answer required.

---

## Commands and Tools Used

    gpupdate /force                    # Forces immediate GPO sync on domain-joined machine
    Active Directory Users and Computers (dsa.msc)   # AD object management console
    Group Policy Management (gpmc.msc)               # GPO creation and linking console

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| NTDS.dit database | Credential theft — obtaining NTDS.dit via volume shadow copy or DC sync attack yields password hashes for every domain user; this is the end goal of many privilege escalation chains in enterprise assessments |
| Domain Admins group | Privilege escalation target — achieving Domain Admin is typically the objective of a network penetration test; excessive Domain Admin membership is a common finding in AD security assessments |
| Kerberos TGT | Ticket-based attacks — Kerberoasting (requesting TGS tickets for service accounts and cracking them offline) and Golden Ticket attacks (forging TGTs using the `krbtgt` hash) are among the most widely used AD attack techniques |
| NetNTLM challenge-response | NTLM relay and Pass-the-Hash — capturing NetNTLM hashes via tools like Responder and relaying them to authenticate elsewhere is a standard lateral movement technique in internal assessments |
| GPO linked to OU | Policy enforcement verification — security analysts auditing AD review GPO configurations to confirm that password policies, screen lock settings, and software restriction policies are applied to the correct OUs and have not been tampered with |
| SYSVOL share | Persistence and credential exposure — SYSVOL is accessible to all domain users; Group Policy Preferences stored in SYSVOL historically contained encrypted but easily decryptable credentials (MS14-025); attackers also use SYSVOL for script-based persistence |
| Trust relationships | Cross-domain lateral movement — a compromised user in a trusted domain can potentially access resources in the trusting domain; misconfigured trust relationships are a common finding in large enterprise AD environments |
| Delegation in AD | Privilege escalation via misconfigurations — overly broad delegation (e.g. a Helpdesk user delegated `GenericAll` on an OU containing admin accounts) allows privilege escalation; reviewing delegation settings is a standard AD security assessment step |

---

## Takeaways

1. **Active Directory is the backbone of enterprise identity — and attacking it is the objective of most internal network penetration tests for good reason.** Every user account, every machine account, every access control decision, and every authentication event flows through AD. Compromising a Domain Controller means compromising every credential, every policy, and every resource in the domain. Understanding AD architecture is not advanced knowledge for a security analyst — it is the minimum required context for understanding what attackers are doing and where they are trying to get to.

2. **Kerberos is designed to avoid transmitting passwords over the network — but the tickets it issues are themselves attack surface.** TGTs and TGS tickets are cryptographically protected, but they can be forged if the right keys are obtained (`krbtgt` hash for Golden Tickets, service account hash for Silver Tickets). Kerberoasting requests valid TGS tickets for service principal names and cracks the service account's password hash offline, without any exploitation — just a legitimate Kerberos request. The protocol is sound; the risk lies in how service accounts are configured and how strong their passwords are.

3. **GPO misconfiguration is a security finding, not just an operational one.** A GPO that applies a permissive password policy, that fails to enforce screen locking, or that inadvertently grants elevated permissions to the wrong OU creates exploitable conditions at scale across every machine the GPO reaches. Reviewing GPO configuration — what is applied, to which OUs, in which order, with which security filtering — is a standard component of an AD security assessment. The same Group Policy Management console used to create policies is used to audit them.

---
