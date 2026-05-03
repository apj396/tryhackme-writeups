# Linux Fundamentals Part 3

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Linux Fundamentals |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/linuxfundamentalspart3 |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the third and final room in the CS101 Linux Fundamentals section. Parts 1 and 2 built the foundation — terminal navigation, filesystem commands, permissions, and remote access via SSH. Part 3 moves into operational Linux: text editors for modifying files directly in the terminal, file transfer methods between machines, process management and signal handling, task automation via crontabs, package management with `apt`, and log file analysis. These are the capabilities that separate basic Linux literacy from the ability to actually work on a Linux system in a security context — configuring tools, investigating running processes, reviewing logs, and scheduling automated tasks.

---

## Key Concepts

### Terminal Text Editors — Nano and Vim

When working on a remote Linux machine over SSH, GUI applications are unavailable. Files must be created and edited directly in the terminal using a text editor. Two editors are introduced in this room.

**Nano** is the beginner-friendly option. Launching it with a filename either opens an existing file or creates a new one:

    nano filename

Once inside Nano, text can be typed or edited immediately. Navigation uses arrow keys. Common keyboard shortcuts are displayed at the bottom of the screen — `^` represents the `Ctrl` key. Saving is done with `Ctrl+O` (write out), followed by Enter to confirm the filename. Exiting is done with `Ctrl+X`.

**Vim** is a more powerful and highly customisable editor that has been in use since 1991. It supports syntax highlighting, operates in multiple modes (normal, insert, visual), and is available on virtually every Unix-based system. Its learning curve is steeper than Nano's, but proficiency in Vim is a meaningful skill signal in Linux-heavy environments. The room introduces Vim as an alternative without requiring deep practice — that depth comes from dedicated rooms.

The practical question in this task requires editing `task3` in the home directory using Nano and reading its contents to obtain the flag `THM{TEXT_EDITORS}`.

### File Transfer — wget, scp, and Python HTTPServer

Moving files between machines is a routine operation in both administration and security work. Three methods are covered:

**wget** downloads files from a web server using HTTP or HTTPS. It requires only the URL of the file:

    wget https://assets.tryhackme.com/additional/linux-fundamentals/part3/myfile.txt

**scp** (Secure Copy) transfers files between two machines over SSH, providing both encryption and authentication. It works in both directions — from local to remote or from remote to local:

    scp important.txt tryhackme@MACHINE_IP:/home/tryhackme/transferred.txt
    scp tryhackme@MACHINE_IP:/home/tryhackme/myRemoteFile myCopiedRemoteFile

**Python HTTPServer** turns the current directory into a lightweight web server that other machines can download files from using `wget` or `curl`. It requires no configuration beyond running the command:

    python3 -m http.server

This starts a server on port 8000 serving files from the current directory. A second machine can then download a file using:

    wget http://MACHINE_IP:8000/.flag.txt

The practical question requires starting the HTTPServer on the deployed machine, then downloading `.flag.txt` from it using `wget` on the AttackBox. Because the filename begins with a dot, it is a hidden file — `ls -a` is needed to confirm it was downloaded, and `cat .flag.txt` reads its contents.

### Processes — ps, top, kill, fg, and systemctl

Every program running on a Linux system is a process. The kernel manages processes and assigns each a unique Process ID (PID). PIDs are assigned sequentially in the order processes start — if the previous process had PID 300, the next will have PID 301.

**Viewing processes:**

`ps` displays the processes running in the current user session with their PID, status, CPU time, and command name:

    ps

`ps aux` extends this to show all processes from all users, including system processes not attached to a terminal session. This is the command used to locate a specific running process — the flag embedded in the running process on the lab machine (`THM{PROCESSES}`) is found by running `ps aux` and scanning the output.

`top` provides a real-time, continuously updating view of all running processes, refreshing every 10 seconds. It is the Linux equivalent of Task Manager.

**Managing processes:**

`kill` sends a signal to a process to terminate it. The cleanly terminating signal is `SIGTERM` — it asks the process to clean up and exit gracefully. The hard kill signal `SIGKILL` forces immediate termination without cleanup. `kill` takes the PID of the target process:

    kill 1234

`fg` brings a backgrounded process back to the foreground of the current terminal session.

**Managing services at boot:**

`systemctl` is the interface to systemd — the init system responsible for starting, stopping, and managing services. Key commands:

    systemctl stop myservice       # stop a running service
    systemctl start myservice      # start a stopped service
    systemctl enable myservice     # configure service to start automatically at boot
    systemctl disable myservice    # remove service from boot startup

### Automating Tasks — Crontabs

The cron process manages scheduled tasks on Linux. It starts during boot and reads crontab files to determine what to run and when. Crontab files use a structured format with six fields:

| Field | Meaning |
|---|---|
| MIN | Minute of execution (0–59) |
| HOUR | Hour of execution (0–23) |
| DOM | Day of the month (1–31) |
| MON | Month of the year (1–12) |
| DOW | Day of the week (0–7, where both 0 and 7 represent Sunday) |
| CMD | The command or script to execute |

A wildcard (`*`) in any field means "every" — so `* * * * * /script.sh` runs the script every minute of every hour of every day. An example backup crontab entry:

    0 */12 * * * cp -R /home/cmnatic/Documents /var/backups/

This runs the backup at minute 0 of every 12th hour, every day.

A special shorthand `@reboot` replaces the time fields entirely and instructs cron to run the command once when the system boots. The crontab on the lab machine uses `@reboot /var/opt/processes.sh` — meaning the cron job runs at system reboot. Crontabs are edited with:

    crontab -e

### Package Management — apt

Software on Ubuntu Linux is installed and managed through the `apt` package manager. Software developers submit their tools to apt repositories — remote servers that store vetted, versioned packages. The system's list of known repositories is stored in `/etc/apt/sources.list` and in files under `/etc/apt/sources.list.d/`.

Core apt commands:

    apt update                        # refresh the local list of available packages
    apt install packagename           # install a package
    apt remove packagename            # remove a package
    add-apt-repository repositoryurl  # add a new repository to the sources list

GPG keys are used to verify the authenticity of packages from added repositories. TryHackMe lab instances do not have internet access, so the package management task is reading-only — no live installation is performed.

### Log Files — /var/log

Linux applications and services write operational records to log files stored in `/var/log`. These logs are the primary source of evidence for what has happened on a system — service errors, authentication attempts, web requests, and system events all generate log entries.

Key log directories and files on a standard Ubuntu installation:

| Path | Contents |
|---|---|
| `/var/log/auth.log` | Authentication events — logins, sudo usage, SSH connections |
| `/var/log/syslog` | General system messages |
| `/var/log/apache2/access.log` | Web server access log — one line per HTTP request including source IP, timestamp, method, path, and response code |
| `/var/log/apache2/error.log` | Web server error log |

The practical question in the log task requires reading the Apache access log to find the IP address of a user who visited the site. On the lab machine, `access.log` is not readable by the `tryhackme` user, but `access.log.1` is — confirmed by running `ls -la` in `/var/log/apache2/`. Reading `access.log.1` with `cat` reveals the visitor IP `10.9.232.111`.

---

## Walkthrough Notes

The room runs through nine tasks. All practical work is done via SSH from the AttackBox to the deployed target machine, established in Task 2.

**Task 1 (Introduction):** Frames the room as the operational tier of Linux fundamentals — text editing, file transfer, process management, automation, package management, and logging. No answer required.

**Task 2 (Deploy Your Linux Machine):** Deploys both the target machine and AttackBox, then SSH connects using `tryhackme:tryhackme`. No answer required beyond confirming successful login.

**Task 3 (Terminal Text Editors):** Introduces Nano and Vim. The practical question requires opening `task3` in the home directory with Nano and reading the flag embedded in the file — the flag is `THM{TEXT_EDITORS}`.

**Task 4 (General/Useful Utilities):** Covers `wget`, `scp`, and Python HTTPServer. The practical question requires running `python3 -m http.server` on the deployed machine and using `wget http://MACHINE_IP:8000/.flag.txt` on the AttackBox to download the hidden file. Reading it with `cat .flag.txt` returns the flag. A second question asks to create and download additional files to practise the workflow — no flag required, read the instructions and proceed.

**Task 5 (Processes 101):** Covers `ps`, `ps aux`, `top`, `kill`, `fg`, and `systemctl`. Key questions: if the previous process had PID 300, the next process has PID `301`. The signal to cleanly kill a process is `SIGTERM`. Running `ps aux` on the deployed machine reveals a running process containing the flag `THM{PROCESSES}`. The command to stop `myservice` is `systemctl stop myservice`. The command to start it at boot is `systemctl enable myservice`. The command to bring a backgrounded process to the foreground is `fg`.

**Task 6 (Maintaining Your System — Automation):** Covers crontabs. Running `crontab -e` on the deployed machine opens the crontab file in an editor. The single scheduled entry uses `@reboot /var/opt/processes.sh` — meaning the cron job runs `@reboot`, i.e. every time the system reboots.

**Task 7 (Maintaining Your System — Package Management):** Covers `apt`, repositories, GPG keys, and `/etc/apt/sources.list`. Reading-only task — TryHackMe instances have no internet access. No answer required beyond reading the material.

**Task 8 (Maintaining Your System — Logs):** Covers `/var/log` and the Apache log files. Navigating to `/var/log/apache2/` and running `ls -la` shows `access.log` is not accessible by the `tryhackme` user but `access.log.1` is. Reading `access.log.1` with `cat` reveals the visitor IP address: `10.9.232.111`.

**Task 9 (Conclusions and Summaries):** Recaps the full three-part Linux Fundamentals series. Recommends exploring additional TryHackMe rooms on Bash scripting and regular expressions. No answer required.

---

## Commands Used

    ssh tryhackme@MACHINE_IP
    nano task3
    wget https://assets.tryhackme.com/additional/linux-fundamentals/part3/myfile.txt
    scp important.txt tryhackme@MACHINE_IP:/home/tryhackme/transferred.txt
    scp tryhackme@MACHINE_IP:/home/tryhackme/myRemoteFile myCopiedRemoteFile
    python3 -m http.server
    wget http://MACHINE_IP:8000/.flag.txt
    cat .flag.txt
    ps
    ps aux
    top
    kill PID
    fg
    systemctl stop myservice
    systemctl start myservice
    systemctl enable myservice
    systemctl disable myservice
    crontab -e
    apt update
    apt install packagename
    apt remove packagename
    ls -la /var/log/apache2/
    cat /var/log/apache2/access.log.1

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| Nano/Vim for file editing | On-host configuration and remediation — analysts editing firewall rules, removing malicious cron entries, or modifying service configurations on a compromised host do so in a terminal text editor over SSH |
| wget and scp for file transfer | Tool deployment and evidence collection — downloading investigation scripts to a remote host or pulling log files back to an analyst workstation uses exactly these transfer methods |
| Python HTTPServer | Quick file serving during assessments — standing up a temporary HTTP server to serve payloads, scripts, or tool binaries to a target machine is a standard penetration testing technique; defenders recognise unexpected port 8000 traffic as a potential indicator |
| ps aux and process investigation | Malware detection — listing all running processes and looking for anomalous entries (unusual names, unexpected paths, high resource consumption) is a standard first step in host-based incident response |
| SIGTERM vs SIGKILL | Forensic process termination — SIGTERM allows a process to clean up, potentially writing final log entries; SIGKILL terminates immediately without cleanup; choosing between them during an investigation affects what evidence remains |
| systemctl enable/disable | Persistence detection — malware commonly achieves persistence by registering a systemd service that starts at boot; `systemctl list-unit-files --state=enabled` enumerates all boot-start services and is a standard persistence check |
| Crontab review | Persistence hunting — reviewing crontabs for unexpected scheduled tasks is a core step in post-compromise analysis; attacker-added cron entries often run at reboot or at regular intervals to re-establish access |
| /var/log/apache2/access.log | Web server investigation — access logs provide a complete record of every HTTP request, including source IP, timestamp, method, and response code; they are the primary evidence source for web application attacks |

---

## Takeaways

1. **Crontabs are one of the most frequently abused persistence mechanisms on Linux — and reviewing them is one of the first things to check after a compromise.** An entry using `@reboot` runs every time the machine starts, re-executing attacker code even after the initial payload has been removed. An entry using a time-based schedule re-runs at defined intervals, maintaining access without any further interaction from the attacker. The presence of an unexpected crontab entry — especially one referencing a script in `/tmp` or an unusual path — is a high-confidence persistence indicator.

2. **`ps aux` is a complete snapshot of what is running on the machine at a moment in time — and that snapshot is evidence.** Running it immediately after gaining access to a potentially compromised host preserves a record of active processes before anything is terminated or cleaned up. The attacker's tool may already be running. The C2 connection may already be established. The process list captures it. This is why process review comes before remediation — you need to know what is there before you start removing it.

3. **Log files are the timeline of what happened — but only if they are preserved before the machine is touched.** `/var/log/apache2/access.log` records every web request. `/var/log/auth.log` records every authentication event. These files are evidence, and they can be overwritten, rotated, or deleted. The operational habit of copying log files before taking any remediation action is the difference between an investigation with a clear timeline and one with a gap where the critical period should be.

---
