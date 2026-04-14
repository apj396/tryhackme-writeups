# Linux Fundamentals Part 2

**Module:** Pre-Security (Legacy) → Linux Fundamentals  
**Difficulty:** Easy  
**Platform:** TryHackMe  
**Room Link:** https://tryhackme.com/room/linuxfundamentalspart2  
**Date Completed:** January 2026  
**Author:** Adwait Joshi

---

## What This Room Covers

This room builds on Part 1 by introducing remote access via SSH, flags and switches that extend command functionality, man pages for self-directed learning, and file permissions — one of the most security-relevant topics in the entire Linux Fundamentals module. It also covers text editors and a broader set of useful utilities.

Where Part 1 established basic navigation and file operations, Part 2 introduces the depth that makes Linux genuinely useful — and genuinely dangerous when misconfigured.

---

## Key Concepts

### 1. SSH — Secure Shell

SSH is the standard protocol for remotely accessing Linux machines over a network. It creates an encrypted terminal session between your machine and a remote host.

```bash
# Basic SSH connection
ssh username@remote-ip

# SSH with specific port (default is 22)
ssh username@remote-ip -p 2222

# SSH with private key authentication
ssh -i private-key.pem username@remote-ip

# SSH with verbose output (useful for debugging connection issues)
ssh -v username@remote-ip
```

**How SSH authentication works:**

Two methods:

**Password authentication** — user provides password at prompt. Simpler but weaker — susceptible to brute force.

**Key-based authentication** — uses a cryptographic key pair:
- **Private key** — stays on the client, never shared
- **Public key** — placed on the server in `~/.ssh/authorized_keys`

On connection, the server issues a challenge that only the holder of the matching private key can answer. No password transmitted. Far more secure.

```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096

# Copy public key to remote server
ssh-copy-id username@remote-ip

# Private key location (default)
~/.ssh/id_rsa

# Public key location (default)
~/.ssh/id_rsa.pub

# Authorized keys on server
~/.ssh/authorized_keys
```

**Real-world relevance:** SSH key management is a primary attack surface in cloud environments. Misconfigured `authorized_keys` files, overly permissive private key permissions, and exposed private keys in repositories are consistent findings. In incident response, reviewing `~/.ssh/authorized_keys` on compromised servers often reveals attacker-planted public keys — a persistence mechanism that survives password changes.

---

### 2. Flags and Switches

Linux commands accept flags (also called switches or options) that modify their behaviour. Understanding how to use them — and how to discover them — is what separates basic command usage from effective command usage.

```bash
# Without flag — basic output
ls

# With flag — detailed output
ls -l

# With multiple flags
ls -la

# Long-form flags (double dash)
ls --all          # same as -a
ls --help         # show help for ls
```

**The `--help` flag** is available on virtually every command and prints a summary of available options:

```bash
ssh --help
find --help
grep --help
```

**Real-world relevance:** Security tools like `nmap`, `gobuster`, and `hydra` have extensive flag sets — knowing how to read `--help` output and man pages is how you use them effectively without memorising every option.

---

### 3. Man Pages

Man (manual) pages are the built-in documentation system for Linux commands. Every standard command has a man page — comprehensive, authoritative, and available offline.

```bash
# Open man page for a command
man ls
man ssh
man find
man grep

# Navigate man pages
# Arrow keys or j/k — scroll
# / followed by term — search within page
# n — next search result
# q — quit
```

**Man page structure:**
- **NAME** — command name and brief description
- **SYNOPSIS** — syntax with all available options
- **DESCRIPTION** — detailed explanation
- **OPTIONS** — every flag documented
- **EXAMPLES** — usage examples
- **SEE ALSO** — related commands

**Real-world relevance:** The ability to read man pages means you're never truly stuck on an unfamiliar command — the answer is always one `man` invocation away. In an assessment or incident, you'll frequently encounter commands or tools you haven't used before. Self-directed learning from documentation is a professional skill, not just a study habit.

---

### 4. File Permissions

File permissions are one of the most security-critical concepts in Linux. They determine who can read, write, and execute every file and directory on the system.

**Permission structure:**

```bash
ls -la
# Output:
-rwxr-xr-- 1 root users 1234 Jan 10 12:00 script.sh
│└──┬───┘
│   └── Three groups: owner | group | others
└── File type: - (file), d (directory), l (symlink)
```

**Breaking down `rwxr-xr--`:**

| Characters | Who | Permissions |
|---|---|---|
| `rwx` | Owner (root) | Read, Write, Execute |
| `r-x` | Group (users) | Read, Execute |
| `r--` | Others | Read only |

**Permission values:**

| Symbol | Meaning | Numeric Value |
|---|---|---|
| `r` | Read | 4 |
| `w` | Write | 2 |
| `x` | Execute | 1 |
| `-` | No permission | 0 |

**Numeric (octal) notation:**

```bash
# rwxr-xr-- = 754
# Owner: r(4)+w(2)+x(1) = 7
# Group: r(4)+w(0)+x(1) = 5
# Others: r(4)+w(0)+x(0) = 4

chmod 755 script.sh     # rwxr-xr-x
chmod 644 file.txt      # rw-r--r--
chmod 600 private.key   # rw------- (private key — owner only)
chmod 777 file.txt      # rwxrwxrwx (dangerous — avoid)
```

**Changing permissions and ownership:**

```bash
# Change permissions
chmod 755 script.sh
chmod +x script.sh      # add execute for all
chmod -w file.txt       # remove write for all
chmod u+x script.sh     # add execute for owner only

# Change owner
chown username file.txt
chown username:groupname file.txt
chown -R username directory/    # recursive

# Change group
chgrp groupname file.txt
```

**Special permissions:**

| Permission | Symbol | Effect |
|---|---|---|
| SUID | `s` in owner execute | File runs with owner's privileges |
| SGID | `s` in group execute | File runs with group's privileges |
| Sticky bit | `t` in others execute | Only owner can delete files in directory |

```bash
# SUID example — passwd command
ls -la /usr/bin/passwd
# -rwsr-xr-x  ← 's' in owner execute = SUID set

# Find all SUID files on system
find / -perm -4000 2>/dev/null

# Sticky bit example — /tmp directory
ls -la /
# drwxrwxrwt  ← 't' in others execute = sticky bit
```

**Real-world relevance:** Misconfigured permissions are a primary privilege escalation vector on Linux systems:
- World-writable scripts run by root (`chmod 777` on a cron job) — attacker modifies the script, root executes it
- SUID binaries — if a SUID binary is exploitable, the attacker gains the owner's privileges (often root)
- Weak private key permissions — SSH private keys must be `600`; if world-readable, SSH rejects them AND they're exposed to other users
- `/tmp` without sticky bit — any user could delete another user's files

---

### 5. Common Useful Utilities

A set of commands that appear frequently in security contexts:

```bash
# Download files from the web
wget https://example.com/file.txt
wget -O output-name.txt https://example.com/file.txt

# Transfer files — curl
curl https://example.com/file.txt -o file.txt
curl -O https://example.com/file.txt

# System information
uname -a                # kernel version and system info
uname -r                # kernel version only

# Running processes
ps aux                  # all running processes with details
ps aux | grep apache    # find specific process

# Current users logged in
who
w

# Disk usage
df -h                   # filesystem disk usage (human-readable)
du -sh /var/log/        # size of specific directory

# Network connections
netstat -tuln           # listening ports
netstat -anp            # all connections with process IDs
ss -tuln                # modern alternative to netstat

# Environment variables
env                     # print all environment variables
echo $PATH              # print specific variable
export MY_VAR="value"   # set environment variable
```

**Real-world relevance:** `ps aux` and `netstat -tuln` / `ss -tuln` are standard enumeration commands — identifying running processes and listening services on a system. In a post-exploitation context, these reveal what's running and what ports are open internally that weren't visible from outside. `uname -a` gives kernel version — the starting point for kernel exploit research.

---

### 6. Text Editors — nano and vim

Two terminal-based text editors are introduced. Both appear in real-world Linux work — editing configuration files, writing scripts, modifying files during assessments.

**nano — beginner-friendly:**

```bash
# Open file in nano
nano file.txt

# Key shortcuts (shown at bottom of screen)
Ctrl+O    # save (write Out)
Ctrl+X    # exit
Ctrl+W    # search
Ctrl+K    # cut line
Ctrl+U    # paste line
```

**vim — powerful, steeper learning curve:**

```bash
# Open file in vim
vim file.txt

# Modes
i           # enter Insert mode (type text)
Esc         # return to Normal mode

# Normal mode commands
:w          # save
:q          # quit
:wq         # save and quit
:q!         # quit without saving
dd          # delete current line
yy          # copy current line
p           # paste
/pattern    # search forward
n           # next search result
```

**Real-world relevance:** On minimal server installations and CTF environments, `nano` and `vim` are often the only editors available. Being unable to use either means being unable to edit configuration files, write scripts, or modify files during an assessment. `vim` proficiency specifically is worth developing — it's available on virtually every Unix-like system by default.

---

## Walkthrough Notes

### Task 1 — Introduction
Brief recap of Part 1, setting expectations for Part 2. SSH is introduced as the mechanism for the rest of the module — the TryHackMe machine is accessed via SSH rather than the in-browser terminal.

### Task 2 — Accessing Your Linux Machine Using SSH
First live SSH connection in the module. The workflow: deploy the machine, get the IP, connect via `ssh username@ip`. Confirms the fundamental remote access pattern used throughout security work.

### Task 3 — Introduction to Flags and Switches
The `--help` convention and man pages. The exercise involves using flags to modify command output — building the habit of checking available options before assuming a command can't do what you need.

### Task 4 — Filesystem & File Permissions
The permission string breakdown — `rwxr-xr--` parsed character by character. The exercise involves reading permissions from `ls -la` output and understanding what each group (owner, group, others) can do.

### Task 5 — Common Directories
Revisits the filesystem layout with more depth — specifically `/etc`, `/var`, `/root`, `/tmp`, and `/usr`. Each directory's security significance is worth noting here.

### Task 6 — Conclusion
Wraps up with a pointer to Part 3. The SSH workflow, permissions model, and man page habit established here are the foundation for everything in Part 3.

---

## Commands Reference

```bash
# SSH
ssh username@ip
ssh -i key.pem username@ip
ssh-keygen -t rsa -b 4096
ssh-copy-id username@ip

# Flags and documentation
command --help
man <command>

# Permissions
ls -la                          # view permissions
chmod 755 file / chmod +x file  # change permissions
chown user:group file           # change ownership
find / -perm -4000 2>/dev/null  # find SUID files

# System info
uname -a                        # kernel and system info
ps aux                          # running processes
netstat -tuln / ss -tuln        # listening ports
df -h / du -sh                  # disk usage
env / echo $PATH                # environment variables

# File download
wget <url>
curl <url> -o <filename>

# Text editors
nano <file>                     # beginner-friendly editor
vim <file>                      # powerful modal editor
```

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| SSH key-based auth | Gold standard for server access; password auth is a finding in hardening assessments |
| `~/.ssh/authorized_keys` | Attacker persistence mechanism — planted public keys survive password resets |
| Weak file permissions | Primary privilege escalation vector — world-writable scripts, SUID misuse |
| `chmod 777` | Critical misconfiguration — any user can modify; any script = code execution risk |
| `chmod 600` on private keys | Required by SSH; world-readable private key = immediate credential exposure |
| SUID binaries | Standard privilege escalation enumeration — `find / -perm -4000` |
| `ps aux` | Process enumeration post-access — identify running services and their owners |
| `ss -tuln` / `netstat -tuln` | Internal port enumeration — reveals services not exposed externally |
| `uname -a` | Kernel version → kernel exploit research starting point |
| `/tmp/` sticky bit | Absence allows cross-user file deletion; presence is expected hardening |

---

## Takeaways

Three things Part 2 introduces that stay relevant throughout a security career:

1. **Permissions are the Linux security model.** Every privilege escalation technique on Linux — SUID abuse, writable cron jobs, path hijacking, sudo misconfigurations — is ultimately a permissions issue. Understanding the read/write/execute model for owner/group/others, and understanding special bits like SUID, is the foundation for understanding every Linux privilege escalation technique you'll encounter later.

2. **SSH key hygiene is non-negotiable.** Private keys must be `600`. Public keys in `authorized_keys` must be audited. Key pairs left in repositories or with weak permissions are a consistently exploited vulnerability. In a cloud environment where SSH keys are the primary access mechanism, key management is the security model. Treating it as an afterthought is how breaches happen.

3. **Man pages are the answer to "I don't know this flag."** In an assessment or investigation, you'll regularly encounter commands and tools you haven't memorised completely. The man page is always there. The ability to quickly find and apply the right flag from documentation — rather than defaulting to Google — is a meaningful professional skill in environments without internet access, and a speed advantage in environments that have it.

---
