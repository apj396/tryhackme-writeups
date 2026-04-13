# Linux Fundamentals Part 1

**Module:** Pre-Security (Legacy) → Linux Fundamentals  
**Difficulty:** Easy  
**Platform:** TryHackMe  
**Room Link:** https://tryhackme.com/room/linuxfundamentalspart1  
**Date Completed:** January 2026  
**Author:** Adwait Joshi

---

## What This Room Covers

This room introduces Linux — its background, why it matters in cybersecurity, and the foundational command-line skills needed to navigate and interact with a Linux system. It covers basic commands, filesystem navigation, file operations, and an introduction to shell operators.

Linux fluency is non-negotiable in security work. Most servers run Linux. Most security tools run on Linux. Most CTF environments are Linux. This room is where that fluency begins.

---

## Key Concepts

### 1. What is Linux?

Linux is an open-source operating system kernel first released by Linus Torvalds in 1991. It powers the majority of the world's servers, cloud infrastructure, Android devices, and cybersecurity tooling.

**Why Linux in cybersecurity:**
- Most web servers, databases, and cloud instances run Linux
- Security tools — Nmap, Metasploit, Wireshark, Burp Suite, and virtually every offensive and defensive tool — run natively on Linux
- CTF environments and TryHackMe AttackBoxes are Linux
- Understanding Linux is understanding the environment you'll be attacking, defending, and investigating

**Linux distributions (distros):** Linux comes in many flavours — Ubuntu, Debian, Kali, CentOS, Fedora. They share the same kernel but differ in package management, default tools, and use cases. Kali Linux is the standard penetration testing distribution. Ubuntu is common in server environments.

---

### 2. Interacting with Linux — The Terminal

Linux interaction in security contexts is almost exclusively via the terminal — a text-based interface where commands are typed and executed. The shell interprets these commands. The most common shell is **Bash** (Bourne Again Shell).

```bash
# The prompt structure
username@hostname:~$

# username  — who you're logged in as
# hostname  — the machine name
# ~         — current directory (~ means home directory)
# $         — indicates a regular user (# indicates root)
```

**Why the terminal over a GUI:**
- Remote servers rarely have a graphical interface
- SSH connections are terminal-based
- Automation and scripting require command-line proficiency
- Speed — experienced users work faster in a terminal than a GUI

---

### 3. Basic Commands

The foundation of Linux interaction — these commands appear in virtually every security task:

```bash
# Print text to the terminal
echo "Hello World"
echo $USER          # print current username as variable

# Print current logged-in username
whoami

# Print current working directory (where you are in the filesystem)
pwd

# List contents of current directory
ls

# List with details — permissions, owner, size, date
ls -l

# List all files including hidden (dot files)
ls -a

# Combine flags
ls -la
```

**Real-world relevance:** `whoami` is one of the first commands run after gaining access to a system in a penetration test — confirming what user context you're operating under. `pwd` and `ls` orient you in an unfamiliar filesystem. These aren't beginner commands that get retired — they appear at every skill level.

---

### 4. Filesystem Navigation

The Linux filesystem is a hierarchical tree starting from `/` (root). Understanding the structure is prerequisite to finding files, navigating systems, and understanding where sensitive data lives.

**Key directories:**

| Directory | Purpose |
|---|---|
| `/` | Root of the entire filesystem |
| `/home` | User home directories (`/home/username`) |
| `/root` | Home directory for the root user |
| `/etc` | System configuration files |
| `/var` | Variable data — logs, databases, web files |
| `/tmp` | Temporary files — world-writable, often abused |
| `/bin` | Essential user binaries (commands) |
| `/usr` | User programs and utilities |
| `/opt` | Optional/third-party software |
| `/proc` | Virtual filesystem — running processes and system info |

**Navigation commands:**

```bash
# Change directory
cd /etc
cd ~              # go to home directory
cd ..             # go up one level
cd -              # go to previous directory

# Print working directory
pwd

# List directory contents
ls /var/log       # list specific directory without navigating to it
```

**Real-world relevance:** In an investigation or post-exploitation context, specific directories matter immediately:
- `/etc/passwd` and `/etc/shadow` — user accounts and password hashes
- `/var/log/` — system and application logs
- `/tmp/` — commonly used by attackers to stage files (world-writable, often not monitored)
- `/etc/cron*` — scheduled tasks, a persistence mechanism
- `/home/` — user files, browser history, SSH keys

---

### 5. File Operations

Creating, reading, moving, copying, and deleting files are daily operations — both in legitimate administration and in security contexts.

```bash
# Display file contents
cat /etc/passwd
cat file.txt

# Display file contents page by page (useful for long files)
less file.txt       # q to quit, arrow keys to scroll
more file.txt

# Display first N lines of a file
head file.txt       # first 10 lines (default)
head -n 20 file.txt # first 20 lines

# Display last N lines of a file
tail file.txt       # last 10 lines (default)
tail -n 20 file.txt # last 20 lines
tail -f file.txt    # follow — live updates as file grows (log monitoring)

# Create an empty file
touch newfile.txt

# Create a directory
mkdir new-directory
mkdir -p parent/child/grandchild    # create nested directories

# Copy a file
cp source.txt destination.txt
cp -r source-dir/ destination-dir/  # recursive — copy directory

# Move or rename a file
mv oldname.txt newname.txt
mv file.txt /tmp/file.txt

# Delete a file
rm file.txt
rm -r directory/    # recursive — delete directory and contents
rm -rf directory/   # force — no confirmation prompts (use carefully)
```

**Real-world relevance:** `tail -f` on log files is a standard monitoring technique — watching `/var/log/auth.log` live during an incident or test shows authentication events in real time. `cat /etc/passwd` is one of the first enumeration commands run on a compromised Linux system. Understanding `rm -rf` matters because it's irreversible — and because attackers use it to cover tracks.

---

### 6. Searching for Files

Finding specific files on a Linux system is a core skill — both for administration and for enumeration during a security assessment.

```bash
# Find files by name
find / -name "passwords.txt"
find /home -name "*.txt"            # all .txt files in /home
find / -name "*.conf" 2>/dev/null   # suppress permission errors

# Find files by type
find / -type f -name "*.log"        # files only
find / -type d -name "config"       # directories only

# Find files by permissions
find / -perm -4000 2>/dev/null      # SUID files — privilege escalation targets

# Find files modified recently
find /var/log -mtime -1             # modified in last 24 hours

# Search within files — grep
grep "password" file.txt
grep -r "password" /etc/            # recursive search through directory
grep -i "password" file.txt         # case-insensitive
grep -n "error" /var/log/syslog     # show line numbers
```

**Real-world relevance:** `find / -perm -4000` searching for SUID binaries is a standard Linux privilege escalation enumeration step — SUID files run with the owner's permissions regardless of who executes them. `grep -r` through configuration directories finds hardcoded credentials. These commands appear in virtually every Linux-based CTF and real-world assessment.

---

### 7. Shell Operators

Shell operators control how commands are chained, combined, and redirected — fundamental to writing efficient command sequences.

```bash
# & — run command in background
sleep 10 &
# Returns prompt immediately while sleep runs in background

# && — run second command only if first succeeds
mkdir new-dir && cd new-dir
# cd only executes if mkdir succeeds

# || — run second command only if first fails
cd /nonexistent || echo "Directory not found"

# > — redirect output to file (overwrites)
echo "Hello" > output.txt
ls -la > directory-listing.txt

# >> — redirect output to file (appends)
echo "New line" >> output.txt

# < — use file as input to command
sort < unsorted.txt

# | (pipe) — pass output of one command as input to next
cat /etc/passwd | grep "root"
ls -la | grep ".txt"
ps aux | grep "apache"

# 2>/dev/null — redirect error messages to null (suppress errors)
find / -name "secret" 2>/dev/null
```

**Real-world relevance:** Pipes are indispensable. Chaining `cat`, `grep`, `sort`, `uniq`, `awk`, and `cut` with pipes to extract specific data from log files or configuration files is a daily operation in security analysis. Redirecting output to files is how you capture command results during an assessment. `2>/dev/null` appears in virtually every enumeration script to suppress noise from permission-denied errors.

---

## Walkthrough Notes

### Task 1 — Introduction
Background on Linux and why it matters. The key statistic worth internalising: the overwhelming majority of internet-facing servers run Linux. That alone justifies the time investment in this module.

### Task 2 — A Bit of Background on Linux
Linux history and distribution overview. Kali Linux as the penetration testing standard, Ubuntu as the common server/lab environment. TryHackMe's AttackBox runs Ubuntu.

### Task 3 — Interacting With Your First Linux Machine
The in-browser Linux terminal is deployed here. First interaction with a live Linux machine — running `echo`, `whoami`, and basic orientation commands.

### Task 4 — Running Your First Few Commands
`echo` and `whoami` are the entry point. Simple but significant — confirming you can interact with the machine and establishing who you are on it.

### Task 5 — Interacting With the Filesystem
`ls`, `cd`, `cat`, `pwd` in sequence. The exercise builds the muscle memory of navigation — changing directories, listing contents, reading files.

### Task 6 — Searching for Files
`find` and `grep` introduced. The practical exercise involves locating specific files on the system — simulating the enumeration workflow used in real assessments.

### Task 7 — An Introduction to Shell Operators
Pipes and redirects. The exercise chains commands together — the first exposure to writing compound command sequences rather than single commands.

---

## Commands Reference

```bash
# Orientation
whoami                          # current user
pwd                             # current directory
hostname                        # machine name
id                              # user ID and group memberships

# Navigation
ls / ls -la / ls -a             # list directory contents
cd <path> / cd ~ / cd .. / cd - # change directory

# Reading files
cat <file>                      # print file contents
less <file>                     # paginated view
head -n <N> <file>              # first N lines
tail -n <N> <file>              # last N lines
tail -f <file>                  # follow live updates

# File operations
touch <file>                    # create empty file
mkdir <dir> / mkdir -p          # create directory
cp <src> <dst>                  # copy
mv <src> <dst>                  # move/rename
rm <file> / rm -rf <dir>        # delete

# Searching
find <path> -name <pattern>     # find by name
find <path> -perm -4000         # find SUID files
grep <pattern> <file>           # search within file
grep -r <pattern> <dir>         # recursive search
grep -i / -n / -v               # case-insensitive / line numbers / invert

# Shell operators
command &                       # background
cmd1 && cmd2                    # chain on success
cmd1 || cmd2                    # chain on failure
> / >>                          # redirect output (overwrite/append)
|                               # pipe output to next command
2>/dev/null                     # suppress errors
```

---

## Real-World Mapping

| Command / Concept | Real-World Application |
|---|---|
| `whoami` / `id` | First command after gaining access — confirm user context and group memberships |
| `/etc/passwd` | User enumeration — accounts, shells, home directories |
| `/var/log/` | Primary log location — auth.log, syslog, application logs |
| `/tmp/` | Attacker staging ground — world-writable, check for anomalous files |
| `tail -f /var/log/auth.log` | Live authentication monitoring during incident or test |
| `find / -perm -4000` | SUID binary enumeration — standard privilege escalation step |
| `grep -r "password" /etc/` | Credential hunting in configuration files |
| `2>/dev/null` | Noise suppression in enumeration scripts |
| Pipe chains | Log analysis, data extraction, output filtering in investigations |
| `/etc/cron*` | Persistence mechanism — check for attacker-planted scheduled tasks |

---

## Takeaways

Three things Linux Fundamentals Part 1 establishes that compound throughout every subsequent security topic:

1. **The terminal is the environment.** Remote servers, AttackBoxes, compromised machines, cloud instances — interaction with all of them is command-line. Comfort in the terminal isn't a nice-to-have; it's the prerequisite for everything that follows. The commands in this room — `ls`, `cat`, `find`, `grep` — appear in every security workflow at every skill level.

2. **The filesystem layout is a map.** Knowing that `/etc` holds configuration, `/var/log` holds logs, `/tmp` is world-writable, and `/home` holds user data means you know where to look first on an unfamiliar system. Enumeration isn't random exploration — it's a structured walkthrough of known-interesting locations.

3. **Pipes transform single commands into analysis tools.** `cat /var/log/auth.log` returns thousands of lines. `cat /var/log/auth.log | grep "Failed" | grep "root" | tail -n 50` returns the last 50 failed root login attempts. The pipe is what makes the command line genuinely powerful for investigation and analysis — not individual commands in isolation, but chains of them.

---
