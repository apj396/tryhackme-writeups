# Linux Fundamentals Part 3

**Module:** Pre-Security (Legacy) → Linux Fundamentals  
**Difficulty:** Easy  
**Platform:** TryHackMe  
**Room Link:** https://tryhackme.com/room/linuxfundamentalspart3  
**Date Completed:** January 2026  
**Author:** Adwait Joshi

---

## What This Room Covers

Part 3 completes the Linux Fundamentals module by covering the topics that make Linux genuinely powerful in a security context — automation via cron jobs, package management, process management, log files, and an introduction to running services. It also introduces more advanced file operations and the concept of backgrounding tasks.

Where Part 1 built navigation and Part 2 built permissions and remote access, Part 3 builds operational depth — the skills that separate someone who can use Linux from someone who can administer and investigate it.

---

## Key Concepts

### 1. Advanced File Operations

Building on Part 1's file basics with more nuanced operations:

```bash
# Copy with preservation of attributes
cp -p source.txt destination.txt    # preserve timestamps, permissions
cp -a source/ destination/          # archive mode — preserves everything

# Symbolic links (symlinks)
ln -s /path/to/target linkname      # create symlink
ls -la                              # symlinks shown with -> arrow

# File details
file document.txt                   # identify file type regardless of extension
stat file.txt                       # detailed metadata — size, permissions, timestamps

# Word, line, character count
wc file.txt                         # lines, words, characters
wc -l file.txt                      # lines only
wc -w file.txt                      # words only

# Sort and deduplicate
sort file.txt                       # alphabetical sort
sort -n numbers.txt                 # numeric sort
sort -r file.txt                    # reverse sort
sort file.txt | uniq                # remove duplicate lines
sort file.txt | uniq -c             # count occurrences of each line

# Cut and extract columns
cut -d ":" -f 1 /etc/passwd         # extract first field using : delimiter
cut -d "," -f 2,4 data.csv          # extract columns 2 and 4

# Stream editor — find and replace in files
sed 's/old/new/g' file.txt          # replace all occurrences of 'old' with 'new'
sed -i 's/old/new/g' file.txt       # in-place edit (modifies file directly)
```

**Real-world relevance:** `stat` reveals file timestamps — created, modified, accessed — critical in forensic investigations for establishing timelines. `sort | uniq -c` on log files is a quick frequency analysis technique — identifying the most common IPs, error types, or usernames in a log. `cut` extracts specific fields from structured output like `/etc/passwd` or CSV exports.

---

### 2. Process Management

Every running program on Linux is a process. Understanding how to view, manage, and interact with processes is essential for both administration and security investigation.

```bash
# View running processes
ps aux                              # all processes, all users, detailed
ps aux | grep apache                # filter for specific process
top                                 # live process monitor (q to quit)
htop                                # enhanced live monitor (if installed)

# Process identifiers
# PID  — Process ID (unique identifier)
# PPID — Parent Process ID
# UID  — User ID of process owner

# Send signals to processes
kill <PID>                          # send SIGTERM (graceful termination)
kill -9 <PID>                       # send SIGKILL (force kill — immediate)
kill -15 <PID>                      # send SIGTERM explicitly
killall apache2                     # kill all processes named apache2

# Find PID of a process
pidof apache2
pgrep apache2

# Background and foreground
command &                           # run command in background
Ctrl+Z                              # suspend current foreground process
bg                                  # resume suspended process in background
fg                                  # bring background process to foreground
jobs                                # list background/suspended jobs
```

**Common signals:**

| Signal | Number | Effect |
|---|---|---|
| SIGTERM | 15 | Graceful shutdown — process can clean up |
| SIGKILL | 9 | Immediate termination — no cleanup |
| SIGHUP | 1 | Hangup — often used to reload config |
| SIGINT | 2 | Interrupt — same as Ctrl+C |

**Real-world relevance:** In incident response, `ps aux` identifies unexpected or suspicious processes — malware, reverse shells, cryptominers. Process names alone can be misleading — attackers name malicious processes after legitimate ones. Cross-referencing PID with the binary path (`/proc/<PID>/exe`) confirms what's actually running. `kill -9` is the immediate termination mechanism for unresponsive or malicious processes.

```bash
# Check what binary a running process actually is
ls -la /proc/<PID>/exe

# Check open files and network connections for a process
lsof -p <PID>

# Check network connections with process IDs
netstat -anp | grep <PID>
ss -anp | grep <PID>
```

---

### 3. Package Management

Linux software is managed through package managers — tools that handle installation, updating, and removal of software from centralised repositories.

**Debian/Ubuntu-based systems (apt):**

```bash
# Update package list (fetch latest available versions)
sudo apt update

# Upgrade installed packages
sudo apt upgrade

# Install a package
sudo apt install nmap
sudo apt install -y nmap            # auto-confirm prompts

# Remove a package
sudo apt remove nmap
sudo apt purge nmap                 # remove including config files

# Search for a package
apt search nmap
apt-cache search keyword

# Show package information
apt show nmap

# List installed packages
dpkg -l
dpkg -l | grep nmap
```

**Manual installation from source:**

```bash
# Download, extract, compile, install pattern
wget https://example.com/tool.tar.gz
tar -xf tool.tar.gz
cd tool/
./configure
make
sudo make install
```

**Real-world relevance:** Package managers are how tools get installed on Linux attack and defence environments. Understanding `apt` means being able to install missing tools quickly during an assessment. Package management logs (`/var/log/dpkg.log`) record installation history — in a forensic investigation, unexpected recently-installed packages are an indicator of attacker activity.

```bash
# Check recently installed packages — forensic use
cat /var/log/dpkg.log | grep "install"
grep "install" /var/log/apt/history.log
```

---

### 4. Cron Jobs — Scheduled Tasks

Cron is the Linux task scheduler. It runs commands automatically at specified times — for system maintenance, backups, monitoring scripts, or anything that needs to happen on a schedule.

**Crontab syntax:**

```
* * * * * command
│ │ │ │ │
│ │ │ │ └── Day of week (0-7, 0 and 7 = Sunday)
│ │ │ └──── Month (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)
```

**Common examples:**

```bash
# Run script every minute
* * * * * /home/user/script.sh

# Run at 2:30 AM daily
30 2 * * * /usr/bin/backup.sh

# Run every hour
0 * * * * /usr/bin/check.sh

# Run at midnight on the 1st of every month
0 0 1 * * /usr/bin/monthly.sh

# Run every 5 minutes
*/5 * * * * /usr/bin/monitor.sh
```

**Managing crontabs:**

```bash
# Edit current user's crontab
crontab -e

# List current user's crontab
crontab -l

# Edit root's crontab
sudo crontab -e

# System-wide cron directories
ls /etc/cron.d/             # drop-in cron files
ls /etc/cron.daily/         # scripts run daily
ls /etc/cron.hourly/        # scripts run hourly
ls /etc/cron.weekly/        # scripts run weekly
ls /etc/cron.monthly/       # scripts run monthly
cat /etc/crontab            # system crontab
```

**Real-world relevance:** Cron jobs are one of the most common persistence mechanisms on Linux. An attacker with write access plants a cron job that runs a reverse shell or maintains their access. During a privilege escalation assessment, world-writable scripts executed by root's crontab are a classic escalation path — modify the script, wait for cron to execute it as root.

```bash
# Enumeration — find all cron jobs on system
crontab -l
sudo crontab -l
cat /etc/crontab
ls -la /etc/cron*
find / -name "*.cron" 2>/dev/null

# Check for world-writable scripts run by cron
cat /etc/crontab | grep -v "^#"
```

---

### 5. System Logs

Linux maintains extensive logs in `/var/log/`. These are the primary evidence source in incident response and forensic investigation.

**Key log files:**

| Log File | Contents |
|---|---|
| `/var/log/syslog` | General system activity — catch-all log |
| `/var/log/auth.log` | Authentication events — logins, sudo, SSH |
| `/var/log/kern.log` | Kernel messages |
| `/var/log/dmesg` | Boot and hardware messages |
| `/var/log/dpkg.log` | Package installation/removal history |
| `/var/log/apt/history.log` | APT transaction history |
| `/var/log/apache2/access.log` | Apache web server access log |
| `/var/log/apache2/error.log` | Apache web server error log |
| `/var/log/nginx/access.log` | Nginx access log |
| `/var/log/faillog` | Failed login attempts |
| `/var/log/wtmp` | Login/logout history (binary — use `last`) |
| `/var/log/btmp` | Failed login history (binary — use `lastb`) |

```bash
# Read standard logs
cat /var/log/syslog
tail -f /var/log/auth.log           # live authentication monitoring
grep "Failed password" /var/log/auth.log    # failed SSH attempts
grep "Accepted" /var/log/auth.log           # successful SSH logins

# Read binary logs
last                                # login history from /var/log/wtmp
lastb                               # failed login history from /var/log/btmp
lastlog                             # last login per user

# Log analysis patterns
grep "Failed" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn
# Extracts IPs from failed SSH attempts, counts and sorts by frequency
```

**Real-world relevance:** Log analysis is a primary SOC skill. In an SSH brute force investigation, `/var/log/auth.log` contains every attempt — source IP, username tried, timestamp. The `grep | awk | sort | uniq -c | sort -rn` pipeline extracts and ranks attacker IPs by attempt count. Web server access logs reveal scanning activity, exploitation attempts, and successful accesses. Knowing where logs live and how to query them quickly is non-negotiable.

---

### 6. Running Services

Linux services (daemons) are background processes that run continuously — web servers, SSH, databases, cron. Managing them is fundamental to both administration and security assessment.

```bash
# systemd — modern service manager (Ubuntu 16.04+)

# Check service status
systemctl status apache2
systemctl status ssh

# Start / stop / restart service
sudo systemctl start apache2
sudo systemctl stop apache2
sudo systemctl restart apache2
sudo systemctl reload apache2       # reload config without full restart

# Enable / disable at boot
sudo systemctl enable apache2
sudo systemctl disable apache2

# List all running services
systemctl list-units --type=service --state=running

# List all services (including stopped)
systemctl list-units --type=service --all
```

**Python quick web server — commonly used in assessments:**

```bash
# Serve current directory over HTTP (Python 3)
python3 -m http.server 8080

# Serve on specific interface
python3 -m http.server 8080 --bind 0.0.0.0
```

**Real-world relevance:** `systemctl list-units --state=running` enumerates all active services on a system — a standard post-access enumeration step. Unexpected services running (unusual ports, unfamiliar names) are worth investigating. The Python HTTP server is used constantly in assessments — to serve payloads, transfer files between machines, or host a quick web interface. Knowing which services are enabled at boot (`systemctl is-enabled`) identifies persistence mechanisms.

---

## Walkthrough Notes

### Task 1 — Deploy Your Linux Machine
SSH connection to the room's machine — same workflow as Part 2. By this point the SSH connection pattern is routine.

### Task 2 — Terminal Text Editors
`nano` and `vim` revisited with practical exercises — creating and editing files. The focus here is building actual editor fluency rather than just knowing they exist.

### Task 3 — General/Useful Utilities
`wget` for downloading files, the Python HTTP server for serving them. The practical exercise involves downloading a file from a remote server to the target machine — a file transfer technique used regularly in assessments.

### Task 4 — Processes 101
`ps`, `top`, signals, backgrounding. The exercise involves identifying a specific running process and sending it the appropriate signal.

### Task 5 — Maintaining Your System: Automation
Cron jobs. The exercise involves reading and understanding an existing crontab and creating a simple scheduled task. The security angle — cron as a persistence mechanism — is the carry-forward value.

### Task 6 — Maintaining Your System: Package Management
`apt` workflow — update, install, remove. The exercise involves installing a package and confirming it works.

### Task 7 — Maintaining Your System: Logs
`/var/log/` exploration. The exercise involves reading specific log files to answer questions — building the habit of going to logs first in an investigation.

---

## Commands Reference

```bash
# Advanced file operations
stat <file>                         # file metadata and timestamps
file <file>                         # identify file type
wc -l <file>                        # count lines
sort <file> | uniq -c               # frequency count
cut -d ":" -f 1 /etc/passwd         # extract field from delimited file
sed 's/old/new/g' <file>            # find and replace

# Process management
ps aux                              # all processes
top / htop                          # live process monitor
kill <PID> / kill -9 <PID>          # terminate process
jobs / bg / fg                      # job control
ls -la /proc/<PID>/exe              # verify actual binary of process
lsof -p <PID>                       # files opened by process

# Package management
sudo apt update && sudo apt upgrade
sudo apt install <package>
sudo apt remove / purge <package>
grep "install" /var/log/dpkg.log    # installation history

# Cron jobs
crontab -l / crontab -e             # list/edit user crontab
sudo crontab -l                     # root's crontab
cat /etc/crontab                    # system crontab
ls /etc/cron.*                      # cron directories

# Logs
tail -f /var/log/auth.log           # live auth monitoring
grep "Failed password" /var/log/auth.log
last / lastb / lastlog              # login history
grep "Failed" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Services
systemctl status <service>
systemctl start/stop/restart <service>
systemctl list-units --type=service --state=running
python3 -m http.server 8080         # quick file server
```

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| `ps aux` + `/proc/<PID>/exe` | Verify suspicious process identity — malware often mimics legitimate process names |
| `kill -9` | Force-terminate malicious processes during incident containment |
| Cron job enumeration | Persistence mechanism detection — attacker-planted cron jobs |
| World-writable cron scripts | Privilege escalation vector — modify script, wait for root execution |
| `/var/log/auth.log` | SSH brute force detection, successful login tracking, sudo activity |
| `grep | awk | sort | uniq -c` | Log frequency analysis — rank IPs, usernames, error types by occurrence |
| `/var/log/dpkg.log` | Forensic detection of attacker-installed packages |
| `last` / `lastb` | Login history — establish who accessed a system and when |
| `systemctl list-units --state=running` | Service enumeration post-access — detect unexpected running services |
| Python HTTP server | File transfer in assessments; also used by attackers to serve payloads |
| `stat` timestamps | Forensic timeline reconstruction — when was a file created, modified, accessed |

---

## Takeaways

Three things Part 3 delivers that complete the Linux Fundamentals picture:

1. **Logs are the ground truth.** In any Linux-based investigation, the logs in `/var/log/` are the primary evidence. Authentication events, package installations, web requests, system messages — all recorded, all queryable with standard command-line tools. The pipeline `grep | awk | sort | uniq -c | sort -rn` is not a one-off trick — it's a reusable analysis pattern that works on any structured log file. Building fluency with log analysis at the command line is one of the highest-return skills in defensive security.

2. **Cron is a persistence surface.** Every cron job on a system is a scheduled code execution. Attacker-controlled cron jobs survive reboots, survive password changes, and often survive incident response if the responder doesn't know to check them. Cron enumeration — `crontab -l`, `sudo crontab -l`, `cat /etc/crontab`, `ls /etc/cron.*` — is a standard step in both privilege escalation and forensic investigation. It takes thirty seconds and is consistently overlooked.

3. **Process verification goes beyond process names.** `ps aux` shows process names — but process names can be spoofed. `/proc/<PID>/exe` shows the actual binary a process is running. `lsof -p <PID>` shows what files it has open. `netstat -anp | grep <PID>` shows what network connections it's maintaining. Investigating a suspicious process means verifying all three — name, binary path, and network activity — not just the name.

---
