# Linux Fundamentals Part 2

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Linux Fundamentals |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/linuxfundamentalspart2 |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the second room in the CS101 Linux Fundamentals section. Part 1 established terminal interaction through an in-browser machine — a controlled, self-contained environment. Part 2 makes a significant step up: the primary machine is now accessed remotely over SSH, which is how Linux machines are interacted with in every real-world context. Beyond remote access, the room covers flags and switches that extend command behaviour, a new set of filesystem commands for creating and moving files, an introduction to file permissions and user switching, and an overview of the key root directories on a standard Ubuntu installation.

---

## Key Concepts

### SSH — Secure Shell for Remote Access

SSH (Secure Shell) is the standard protocol for remotely accessing and controlling Linux machines over a network. It encrypts all data transmitted between the client and the server — credentials, commands, and output — so interception yields only ciphertext. In Part 1, the terminal was embedded in the browser. In Part 2, two machines are deployed: the target Linux machine and the TryHackMe AttackBox, and SSH is used from the AttackBox to connect to the target.

The SSH connection command follows the pattern:

    ssh tryhackme@MACHINE_IP

After running this command, SSH prompts to confirm trust in the host (type `yes` on first connection), then prompts for the password. An important operational detail: when typing a password into an SSH login prompt, no characters or symbols appear on screen. The input is still being registered — type the password and press Enter. For this room the credentials are `tryhackme:tryhackme`.

Once connected, every command executed runs on the remote machine, not on the local AttackBox. The prompt changes to reflect this — the hostname in the prompt switches to the remote machine's name.

### Flags and Switches

Most Linux commands accept additional arguments called flags or switches. These are preceded by a hyphen (`-` for short form, `--` for long form) and modify or extend the default behaviour of the command. Without flags, a command performs its baseline function. With the appropriate flag, the same command can reveal significantly more information.

The `ls` command without flags lists visible files and directories. Adding `-a` (or `--all`) reveals hidden files and directories — those whose names begin with a dot (`.`). Adding `-l` switches to a long listing format showing permissions, ownership, size, and modification date. Adding `-h` alongside `-l` presents file sizes in human-readable format (KB, MB, GB) rather than raw bytes.

Finding the available flags for any command is done in two ways:

**`--help`** — most commands support this flag and print a summary of available options directly to the terminal:

    ls --help

**`man` (manual pages)** — the `man` command opens the full reference manual for a command, navigable with arrow keys and exited with `q`. Man pages are more comprehensive than `--help` and include detailed explanations of every option:

    man ls

The room's question asking for the flag that displays output in human-readable format is answered by `-h`, found by scrolling through `man ls`.

### File System Interaction — New Commands

Part 2 introduces six commands that extend filesystem capability beyond navigation and reading:

| Command | Purpose |
|---|---|
| `touch` | Creates a new, empty file with the specified name |
| `mkdir` | Creates a new directory |
| `cp` | Copies a file to a specified destination |
| `mv` | Moves a file to a specified destination — also used to rename files |
| `rm` | Removes a file; requires `-R` flag to remove a directory and its contents |
| `file` | Determines the type of a file regardless of its extension |

The `file` command is particularly useful in security contexts — file extensions can be changed or omitted to disguise a file's true nature. Running `file unknown1` on the lab machine returns `ASCII text`, confirming the file type regardless of its name. Moving a file with `mv myfile myfolder` places `myfile` inside `myfolder`. The contents of the moved file, read with `cat ./myfolder/myfile`, reveal the flag `THM{FILESYSTEM}`.

### File Permissions

Linux controls access to files and directories through a permissions model that specifies what actions are allowed and for which users. Running `ls -lh` displays permissions as a string of characters at the start of each row:

    -rw-r--r-- 1 cmnatic cmnatic 0 Feb 19 10:37 file1

The permission string breaks into four parts:

| Segment | Meaning |
|---|---|
| `-` (first character) | File type: `-` for file, `d` for directory |
| `rw-` (characters 2–4) | Owner permissions: read, write, no execute |
| `r--` (characters 5–7) | Group permissions: read only |
| `r--` (characters 8–10) | Other permissions: read only |

The three permission types are read (`r`), write (`w`), and execute (`x`). A dash (`-`) in any position means that permission is not granted.

### Switching Users — su

The `su` (substitute user) command allows switching to another user account within the current terminal session. It requires the password of the user being switched to:

    su user2

Once switched, commands execute with the permissions of the new user. Files accessible only to `user2` become readable; files restricted from `user2` become inaccessible. The room's practical question requires switching to `user2` and reading the `important` file, which yields the flag `THM{SU_USER2}`.

### Common Root Directories

A standard Ubuntu Linux installation organises its filesystem under several root-level directories. Understanding what lives where is essential for both system administration and security investigation:

| Directory | Contents and Purpose |
|---|---|
| `/etc` | System configuration files used by the operating system — password files, network configuration, service configuration |
| `/var` | Variable data — files that change frequently during normal operation, including logs (`/var/log`) and application data |
| `/root` | Home directory of the root user — distinct from `/home`, accessible only by root |
| `/tmp` | Temporary files — volatile, cleared on reboot; commonly used by attackers to store tools because it is world-writable |

The `/tmp` directory is particularly relevant from a security perspective: because any user can write to it without elevated privileges, it is a common location for malware to drop files, for attackers to stage tools after initial access, and for exploitation scripts to write temporary payloads.

---

## Walkthrough Notes

The room runs through eight tasks. From Task 2 onward, all practical work is done via SSH from the AttackBox to the deployed target machine.

**Task 1 (Introduction):** Outlines the room's scope — SSH access, flags and switches, filesystem commands, permissions, and root directories. Notes that Part 1 is a prerequisite. No answer required.

**Task 2 (Accessing Your Linux Machine Using SSH):** Deploys both the target machine and the AttackBox. From the AttackBox terminal, the SSH command is run with the target machine's IP address. Credentials are `tryhackme:tryhackme`. The question confirms successful login — the answer is the hostname of the connected machine, visible in the terminal prompt after login: `I've logged into the Linux Fundamentals Part 2 machine using SSH!` No answer required beyond proceeding.

**Task 3 (Introduction to Flags and Switches):** Covers how flags extend commands. The question asks which flag for `ls` displays output in human-readable format — the answer is `-h`, found via `man ls`. The question asking about hidden files is answered by `-a` (or `--all`).

**Task 4 (Filesystem Interaction Continued):** Covers `touch`, `mkdir`, `cp`, `mv`, `rm`, and `file`. The question asking how to create a file named `newnote` is answered by `touch newnote`. The question asking for the file type of `unknown1` in the home directory requires running `file unknown1` on the deployed machine — the answer is `ASCII text`. The question asking how to move `myfile` to `myfolder` is answered by `mv myfile myfolder`. Reading the moved file with `cat ./myfolder/myfile` reveals the flag `THM{FILESYSTEM}`.

**Task 5 (Permissions 101):** Covers the `ls -lh` permission string and the `su` command. The question asking for the owner of the `important` file requires running `ls -la` in the home directory and reading the owner column. Switching to `user2` with `su user2` and then running `cat important` yields the flag `THM{SU_USER2}`.

**Task 6 (Common Directories):** Covers `/etc`, `/var`, `/root`, and `/tmp`. Questions ask for the purpose of each directory — conceptual answers drawn from the task description. The key point flagged is that `/tmp` is volatile and world-writable, making it a common staging location.

**Task 7 (Conclusions and Summaries):** Recaps SSH, flags, filesystem commands, permissions, and root directories. Points to Linux Fundamentals Part 3. No answer required.

**Task 8 (Linux Fundamentals Part 3):** Directs to the next room. No answer required.

---

## Commands Used

    ssh tryhackme@MACHINE_IP
    ls --help
    man ls
    ls -a
    ls -lh
    touch newnote
    mkdir newdirectory
    cp sourcefile destination
    mv myfile myfolder
    rm filename
    rm -R directoryname
    file unknown1
    cat ./myfolder/myfile
    su user2
    cat important

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| SSH remote access | Standard analyst workflow — all Linux host investigation and administration in real environments is done over SSH; comfort with SSH connection, host key verification, and password prompts is baseline |
| `ls -lh` permissions review | Privilege escalation detection — reviewing file permissions on sensitive directories and executables helps identify world-writable files, SUID binaries, or misconfigured ownership that attackers exploit |
| `file` command | Malware triage — files dropped by attackers often have misleading or missing extensions; `file` determines the true type based on magic bytes in the file header, not the name |
| `/tmp` directory | Attacker staging awareness — during incident response, `/tmp` is one of the first directories checked for dropped tools, compiled exploits, or downloaded payloads, precisely because it requires no elevated permissions to write to |
| `/etc` directory | Configuration review — password files (`/etc/passwd`, `/etc/shadow`), service configurations, and network settings all live in `/etc`; analysts review these during privilege escalation investigation and post-compromise configuration audits |
| `su` user switching | Lateral movement investigation — examining which users have been switched to, when, and from which session is a standard step in reconstructing attacker movement across a compromised Linux host |

---

## Takeaways

1. **SSH is the entry point for almost every real Linux security task — and its first-time host key prompt is a detail that matters.** The message "Are you sure you want to continue connecting (yes/no)?" appears when connecting to an SSH server for the first time. It is asking whether you trust the server's identity. In a controlled lab this is always `yes`. In a real engagement, blindly accepting this prompt without verifying the fingerprint against a known good value is a potential man-in-the-middle risk. The habit of paying attention to this prompt, not clicking past it reflexively, starts here.

2. **File permissions are not just an access control mechanism — they are an investigative signal.** A file with world-write permissions in a directory that should be restricted is a finding. An executable owned by root with the SUID bit set runs with root privileges regardless of who invokes it — a common privilege escalation vector. Reading `ls -lh` output fluently, understanding what each segment means, and recognising when a permission string looks wrong is a skill that surfaces real findings in real investigations.

3. **`/tmp` being world-writable and volatile is a design feature that attackers rely on.** Any user — including low-privilege service accounts — can write to `/tmp` without needing elevated access. This makes it the natural landing zone for files an attacker wants to create without triggering permission errors. It is also the first place to look during incident response, and because it clears on reboot, it is the first place to preserve before restarting a compromised machine. Understanding why `/tmp` works the way it does is more useful than just knowing its name.

---
