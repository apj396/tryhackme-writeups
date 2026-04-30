# Linux Fundamentals Part 1

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Linux Fundamentals |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/linuxfundamentalspart1 |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the first room in the CS101 Linux Fundamentals section. Linux is the operating system that underpins most of the infrastructure a security analyst will ever interact with — web servers, cloud instances, SIEM backends, containers, and the majority of offensive and defensive tooling all run on Linux. This room builds the absolute foundation: what Linux is, how to interact with it through the terminal, how to navigate the filesystem, how to find and read files, and how to use shell operators to control where command output goes. All interaction is through an in-browser terminal — no local setup required.

---

## Key Concepts

### What Linux Is

Linux is an open-source operating system first released in 1991 by Linus Torvalds, a Finnish computer science student who began it as a personal project and released it to the public. Unlike Windows or macOS, Linux is not a single operating system — it is a family of operating systems all built on the Linux kernel. Different distributions (distros) package the kernel with different software, tools, and desktop environments for different use cases. Common distros include Ubuntu, Debian, Fedora, Kali Linux, and Arch. TryHackMe's lab machines run Ubuntu.

Linux is ubiquitous in infrastructure that is not visible to end users. Android devices use the Linux kernel. The majority of the world's web servers run Linux. Supercomputers, cloud platforms, network devices, and most cybersecurity tools are built for and run on Linux. For a security analyst, Linux is not optional background knowledge — it is the primary operating environment.

### First Commands — echo and whoami

Two commands introduce the concept of terminal interaction:

`echo` outputs whatever text is given to it as an argument. It is the simplest demonstration that the terminal takes input and produces output:

    echo TryHackMe

`whoami` returns the username of the currently logged-in user. On the TryHackMe lab machine, this returns `tryhackme`. It is useful for confirming the current user context — particularly relevant when switching between users or escalating privileges:

    whoami

### Navigating the Filesystem — ls, cd, cat, pwd

Four commands form the foundation of filesystem navigation:

| Command | Full Name | Purpose |
|---|---|---|
| `ls` | Listing | Lists files and directories in the current directory |
| `cd` | Change Directory | Moves into a specified directory |
| `cat` | Concatenate | Outputs the contents of a file to the terminal |
| `pwd` | Print Working Directory | Prints the full path of the current directory |

These four commands together allow a user to explore an entire filesystem — listing what exists, moving into directories, reading file contents, and confirming location at any point. On the lab machine, running `ls` in the home directory reveals four folders. Only `folder4` contains a file (`note.txt`), confirmed by running `ls folder4` or by entering each directory with `cd` and checking with `ls`. The file's contents (`Hello World!`) are read with `cat`. The full path after navigating into `folder4` is `/home/tryhackme/folder4`, confirmed with `pwd`.

### Searching for Files — find and grep

As filesystem complexity grows, manually navigating becomes impractical. Two commands address this:

`find` searches for files or directories matching specified criteria. The `-name` flag searches by filename, and `-type f` restricts results to files (as opposed to directories with `-type d`). Wildcard matching is supported — `find -name *.txt` finds all `.txt` files in the current directory and below:

    find -name note.txt
    find -name *.txt

`grep` searches the contents of files for a specified string. It is indispensable for extracting relevant lines from large files — logs, configuration files, wordlists. Running `grep "THM" access.log` searches `access.log` for any line containing the string `THM` and returns those lines:

    grep "THM" access.log

The flag in the room's grep task is `THM{ACCESS}`, found by running grep against the provided `access.log` file.

### Shell Operators

Shell operators control how commands interact with each other and with files:

| Operator | Behaviour |
|---|---|
| `&` | Runs the command in the background — control returns to the terminal immediately |
| `&&` | Runs the second command only if the first succeeds (exit code 0) |
| `>` | Redirects output to a file, overwriting existing contents |
| `>>` | Redirects output to a file, appending to existing contents |

The distinction between `>` and `>>` is operationally significant: `echo password123 > passwords` creates (or overwrites) the file `passwords` with only `password123`. `echo tryhackme >> passwords` appends `tryhackme` to the file, preserving `password123`. Using `>` when `>>` is intended silently destroys existing file content.

---

## Walkthrough Notes

The room runs through eight tasks, all using the in-browser terminal deployed within the room.

**Task 1 (Welcome to Linux Fundamentals):** Introduces the room and the in-browser Linux machine. No answer required.

**Task 2 (A Bit of Background on Linux):** Covers Linux history and distributions. The key question asks for the year of the first Linux release — the answer is `1991`.

**Task 3 (Interacting with Your First Linux Machine):** Deploys the in-browser Ubuntu machine. No answer required — proceed to the next task once the machine is running.

**Task 4 (Running Your First Few Commands):** Introduces `echo` and `whoami`. The question asks what command outputs the text "TryHackMe" — the answer is `echo`. The question asking for the logged-in username requires running `whoami` on the deployed machine, returning `tryhackme`.

**Task 5 (Interacting with the Filesystem):** Covers `ls`, `cd`, `cat`, and `pwd`. Running `ls` in the home directory reveals 4 folders. Checking each folder with `ls` or `cd` and `ls` shows only `folder4` contains a file. Using `cat folder4/note.txt` reveals the content `Hello World!`. Navigating into `folder4` with `cd` and running `pwd` confirms the path `/home/tryhackme/folder4`.

**Task 6 (Searching for Files):** Covers `find` and `grep`. The grep question asks for the flag with prefix "THM" in `access.log` — the command is `grep "THM" access.log` and the flag is `THM{ACCESS}`. No answer required for the find demonstration questions.

**Task 7 (An Introduction to Shell Operators):** Covers `&`, `&&`, `>`, and `>>`. The question asking which operator runs a command in the background is answered by `&`. The question asking for the command to replace the contents of `passwords` with `password123` is answered by `echo password123 > passwords`. The question asking how to add `tryhackme` while keeping `password123` is answered by `echo tryhackme >> passwords`.

**Task 8 (Conclusions and Summaries):** Recaps all commands and operators covered — `echo`, `whoami`, `ls`, `cd`, `cat`, `pwd`, `find`, `grep`, and the four shell operators. Points to Linux Fundamentals Part 2. No answer required.

---

## Commands Used

    echo TryHackMe
    whoami
    ls
    ls folder4
    cd folder4
    cat note.txt
    pwd
    find -name note.txt
    find -name *.txt
    grep "THM" access.log
    echo password123 > passwords
    echo tryhackme >> passwords

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| Terminal navigation (ls, cd, pwd) | Incident response on Linux hosts — analysts SSH into compromised machines and navigate the filesystem manually; comfortable terminal navigation is a prerequisite for any host-based investigation |
| cat for file reading | Log review — reading configuration files, log entries, and dropped files during investigation all rely on `cat` and similar file-reading commands |
| grep for content search | Log analysis at scale — filtering large log files for specific IPs, usernames, error codes, or IOCs is one of the most frequent analyst tasks; grep is the primary tool for doing this without loading files into a GUI |
| find for file location | Malware hunting — searching for files by name, extension, or modification time is a standard technique for locating malware persistence files or attacker-dropped tools on a compromised system |
| `>` vs `>>` operators | Script and automation awareness — understanding output redirection is necessary for reading and writing scripts used in automation, log rotation, and persistence mechanisms |
| `&` background operator | Process management — malware and attacker tools commonly use background execution to persist without blocking the terminal session; recognising this pattern in shell history is a detection indicator |

---

## Takeaways

1. **The terminal is not an obstacle to overcome — it is the primary interface for security work on Linux.** GUI tools exist for many tasks, but the terminal is faster, scriptable, remote-accessible, and available on every Linux system regardless of whether a desktop environment is installed. Building terminal fluency early means every subsequent tool — Wireshark, tcpdump, nmap, grep pipelines — is easier to use and easier to understand.

2. **grep is disproportionately powerful relative to its simplicity.** A single `grep` command against a log file can answer questions that would take minutes to answer manually — which IP triggered the most failed logins, which user account appeared in an error message, whether a specific string appears anywhere in a directory tree. Combined with `find` to locate files and `cat` to read them, these three commands handle the majority of filesystem investigation tasks on a compromised Linux host.

3. **The `>` operator's destructive behaviour is a detail that matters in operational contexts.** Accidentally overwriting a log file or configuration file with `>` instead of `>>` during an investigation can destroy evidence or break a service. The instinct to double-check which operator is correct before redirecting output — especially when working on a live system — is a habit worth forming at the foundation.

---
