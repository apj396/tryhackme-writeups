# John the Ripper: The Basics

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Cryptography |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/johntheripperbasics |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the fourth and final room in the CS101 Cryptography section. Hashing Basics introduced hashcat as a GPU-accelerated cracking tool. This room introduces John the Ripper (JtR) — the other primary password cracking tool in the security professional's arsenal, and the one with significantly broader format support. While hashcat excels at raw GPU-powered dictionary and brute-force attacks, John the Ripper specialises in versatility: it handles dozens of hash types natively, cracks Linux shadow file hashes, Windows NTLM hashes, SSH private key passphrases, encrypted archives, and PDF passwords — all through format-conversion utilities bundled with the tool. The room covers basic hash cracking, Windows authentication hash cracking, Linux shadow file cracking, single crack mode, custom rules, and cracking protected files.

---

## Key Concepts

### What John the Ripper Is

John the Ripper (JtR, or simply `john`) is a free and open-source password cracking tool, first released in 1996 and originally developed for Unix-based systems. It is maintained by Openwall and is one of the most widely used tools in both penetration testing and defensive security assessments. Its primary strength is breadth — it supports hundreds of hash and cipher formats and includes a suite of conversion utilities (`*2john` tools) that transform protected files into crackable hash format.

John operates in several modes, each suited to a different attack scenario. It also includes automatic hash type detection, though the reliability of auto-detection varies and explicitly specifying the format is recommended when the hash type is known.

Cracked passwords are stored in `~/.john/john.pot`. Running `john --show` on a hash file displays any previously cracked results from the pot file without re-running the crack.

### Basic Syntax and Hash Cracking

The fundamental John syntax:

    john <options> <file_containing_hash>

With wordlist and format specified:

    john --format=<format> --wordlist=<wordlist> <hash_file>

Example — cracking an MD5 hash with rockyou.txt:

    john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash_to_crack.txt

The `raw-` prefix distinguishes between raw hash formats (the hash alone) and crypt-style formats (hash with embedded salt and parameters). To check whether a format prefix is needed, or to find the exact format identifier John uses:

    john --list=formats | grep -iF "md5"

To identify an unknown hash type before selecting the format, the `hash-identifier` tool is used:

    python3 /usr/share/hash-identifier/hash-id.py

Paste the hash and it returns the most probable formats ranked by likelihood.

### Cracking Windows Authentication Hashes — NTLM

Windows stores user and service account passwords as NTHash (also called NTLM). The SAM (Security Account Manager) database holds these hashes on local Windows systems. They can be extracted using tools like Mimikatz or by accessing the NTDS.dit file on a Domain Controller. Once extracted, the hash is cracked with John using the `nt` format:

    john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt ntlm_hash.txt

When a Windows NTLM hash is obtained during an assessment, two approaches are possible. The first is Pass-the-Hash — using the hash directly to authenticate to Windows services without cracking it. The second is cracking the hash to recover the plaintext password, which allows access to services that do not accept hash-based authentication and reveals whether the user reuses the password elsewhere.

### Cracking Linux Shadow File Hashes — unshadow

Linux stores password hashes in `/etc/shadow`, which is readable only by root. The `/etc/passwd` file contains usernames and account information but no passwords. To crack Linux password hashes with John, the two files must first be combined using the `unshadow` utility:

    unshadow /etc/passwd /etc/shadow > unshadowed.txt

`unshadow` merges the user account information from passwd with the password hash from shadow, producing a single file in a format John can parse directly. The combined file is then cracked:

    john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt unshadowed.txt

The root password in the room's lab is recovered using this method.

### Single Crack Mode

Single crack mode is John's most targeted approach. Rather than iterating through an external wordlist, it generates password candidates from the information already associated with the account — the username, the GECOS field (the fifth field in `/etc/passwd`, which may contain the user's full name, office, and phone number), and the home directory name. John applies mangling rules to these values — capitalising, appending numbers, substituting characters — to generate a targeted set of candidates.

GECOS stands for General Electric Comprehensive Operating System — a legacy field that survives in modern Unix systems. Its presence in `/etc/passwd` gives John material to work with even when no wordlist is provided. Single crack mode is the correct first attempt when passwords are likely to be based on personal information related to the account.

    john --single --format=<format> <hash_file>

To use single crack mode against a specific hash, the file must contain the username prepended to the hash in the format `username:hash`. This gives John the account context it needs for mangling:

    john --single --format=raw-md5 hash_with_username.txt

The `Joker` account password in the room's single crack lab is recovered using this mode.

### Custom Rules

John's rule system allows defining precise transformations applied to wordlist entries to generate password candidates. Rules are stored in `/etc/john/john.conf` under headers in the format `[List.Rules:RuleName]`. Custom rules are defined by adding a new section to this file.

Rule syntax uses single characters to represent transformations:

| Character | Transformation |
|---|---|
| `c` | Capitalise the first letter |
| `u` | Uppercase the entire word |
| `l` | Lowercase the entire word |
| `r` | Reverse the word |
| `d` | Duplicate the word |
| `$X` | Append character X to the end |
| `^X` | Prepend character X to the beginning |
| `Az"[0-9]"` | Append a character from the set `[0-9]` |

A rule that capitalises the first letter and appends a number would be defined as:

    [List.Rules:CustomRule]
    cAz"[0-9]"

And invoked with:

    john --wordlist=wordlist.txt --rules=CustomRule hash_file.txt

Custom rules allow targeting password policies — for example, if an organisation requires passwords to start with a capital letter and end with a number, a custom rule applying exactly those transformations to a base wordlist will crack policy-compliant passwords far more efficiently than a generic dictionary attack.

### Cracking Protected Files — the *2john Utilities

John's `*2john` conversion utilities extract the crackable hash material from password-protected files. The general workflow is: locate the appropriate utility, run it on the file to produce a hash, then crack the hash with John.

    locate *2john

Common conversion utilities:

| Utility | Input File Type |
|---|---|
| `ssh2john` | SSH private key with passphrase |
| `pdf2john` | Password-protected PDF |
| `zip2john` | Password-protected ZIP archive |
| `rar2john` | Password-protected RAR archive |
| `office2john` | Microsoft Office documents |

The workflow for a protected ZIP file:

    zip2john protected.zip > zip_hash.txt
    john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt

The workflow for a passphrase-protected SSH private key:

    ssh2john id_rsa > ssh_hash.txt
    john --wordlist=/usr/share/wordlists/rockyou.txt ssh_hash.txt

After cracking, the passphrase is displayed and the private key can be used for SSH authentication with:

    ssh -i id_rsa user@MACHINE_IP

---

## Walkthrough Notes

The room runs through eight tasks. A VM is deployed for all practical cracking tasks, accessible via SSH from the AttackBox.

**Task 1 (Introduction):** Introduces John the Ripper — history, purpose, and scope. Notes that cracked passwords are stored in `~/.john/john.pot`. No answer required.

**Task 2 (Setting Up John on Your System):** Covers installation and the automatic hash detection capability. John is pre-installed on the lab VM. No standalone answer required beyond reading the task.

**Task 3 (Cracking Basic Hashes):** Practical hash cracking using wordlist mode. Hash files are in the lab VM. The task requires running John against multiple hash formats — MD5, SHA-1, SHA-256, whirlpool — each specified with the `--format` flag. Answers are recovered by running John and reading the cracked output.

**Task 4 (Cracking Windows Authentication Hashes):** Covers NTLM hash cracking. The hash file contains an NTLM hash. The command uses `--format=nt`. The question also asks about the two approaches when an NTLM hash is obtained — Pass-the-Hash and cracking the hash for the plaintext.

**Task 5 (Cracking /etc/shadow Hashes):** Covers `unshadow` and shadow file cracking. The task requires running `unshadow` on the provided passwd and shadow files to produce `unshadowed.txt`, then cracking with John. The root user's password is recovered.

**Task 6 (Single Crack Mode):** Covers how single crack mode uses username and GECOS information. The hash file must contain `username:hash` format. Running `john --single --format=raw-md5` against the provided file recovers the `Joker` account password.

**Task 7 (Custom Rules):** Covers defining rules in `/etc/john/john.conf` and invoking them. The task walks through creating a rule that capitalises the first letter and appends a digit, then using it against the provided hash. The `c` rule character capitalises the first letter.

**Task 8 (Cracking Password Protected Files):** Covers `zip2john`, `rar2john`, and `ssh2john`. The practical tasks require: running `zip2john` on a protected ZIP to produce a hash, cracking it with John and rockyou.txt; and running `ssh2john` on a passphrase-protected private key, cracking the passphrase, then using the key for SSH authentication to retrieve the flag.

---

## Commands Used

    john --list=formats | grep -iF "md5"
    john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash_to_crack.txt
    john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt ntlm.txt
    john --show hash_file.txt
    unshadow /etc/passwd /etc/shadow > unshadowed.txt
    john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt unshadowed.txt
    john --single --format=raw-md5 hash_with_username.txt
    john --wordlist=wordlist.txt --rules=CustomRule hash_file.txt
    locate *2john
    zip2john protected.zip > zip_hash.txt
    john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt
    ssh2john id_rsa > ssh_hash.txt
    john --wordlist=/usr/share/wordlists/rockyou.txt ssh_hash.txt
    ssh -i id_rsa user@MACHINE_IP
    python3 /usr/share/hash-identifier/hash-id.py

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| NTLM hash cracking | Post-compromise credential recovery — after extracting SAM or NTDS.dit hashes during a penetration test, John cracks weak passwords to demonstrate real-world impact; recovered passwords validate whether the organisation's password policy is actually enforced |
| unshadow and shadow cracking | Linux host compromise assessment — on a compromised Linux system, extracting `/etc/shadow` and cracking hashes establishes which accounts use weak passwords and which service accounts have guessable credentials |
| Single crack mode | Targeted assessment — when assessing specific accounts (help desk, executives), single mode using account-specific GECOS data tests whether users base passwords on personal information before attempting a broader dictionary attack |
| Custom rules | Password policy cracking — organisations that require passwords to meet specific patterns (capital first letter, trailing number, minimum length) create a predictable transformation space; custom rules targeting that space crack policy-compliant but weak passwords efficiently |
| ssh2john and zip2john | Encrypted file analysis — encrypted archives and passphrase-protected private keys recovered from a target system during an assessment may contain sensitive data; the *2john workflow extracts the crackable material without decrypting the file directly |
| john.pot file | Assessment documentation — the pot file persists cracked passwords across sessions; reviewing it after a full assessment provides a complete list of recovered credentials for the final report |

---

## Takeaways

1. **John the Ripper and hashcat serve different strengths — knowing when to use each reduces cracking time significantly.** Hashcat's GPU acceleration makes it faster for large wordlists and brute-force attacks against standard hash formats. John's breadth of format support and the *2john utility suite make it the correct choice when the target is a protected file, a shadow file, or an unusual format hashcat does not natively support. Experienced analysts use both — hashcat for volume and speed, John for format coverage and flexibility.

2. **Single crack mode demonstrates a fundamental truth about password security: personal information is not entropy.** Usernames, full names, birth years, and GECOS fields are not unknown to an attacker who has compromised an account database — that information is right there alongside the hash. A password derived from a username or real name provides almost no additional security over using the name directly. Password policies that prohibit common patterns but permit name-based passwords misunderstand what makes a password strong.

3. **The *2john utilities shift the boundary of what "encrypted" actually means in an assessment context.** A password-protected ZIP was never as secure as its password suggested — if the password is in rockyou.txt, `zip2john` and John crack it in seconds. A passphrase-protected SSH private key is only as secure as the passphrase. Every protected file format is ultimately a hash cracking problem once the file is in hand. Understanding this means understanding that data security does not end with encryption — it extends to key and passphrase strength, and to physical and logical access controls that prevent the encrypted file from being extracted in the first place.

---
