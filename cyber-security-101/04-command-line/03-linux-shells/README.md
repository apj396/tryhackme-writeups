# Linux Shells

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Command Line |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/linuxshells |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the third and final room in the CS101 Command Line section. The previous two rooms covered Windows command-line environments — `cmd.exe` and PowerShell. This room returns to Linux and addresses the layer above individual commands: the shell itself. A shell is the program that acts as the interface between the user and the operating system — it interprets commands and passes them to the OS for execution. The room covers what shells are, the different types of Linux shells and their distinguishing features, how to view and switch between shells, and how to write shell scripts using Bash. The practical component involves writing a working authentication script from scratch and modifying an existing log-searching script — two tasks that reflect real scripting work rather than just running pre-built commands.

---

## Key Concepts

### What a Shell Is

A shell is a program that provides an interface between the user and the operating system kernel. When a command is typed in a terminal, the shell interprets it and passes it to the OS for execution, then displays the output back to the user. Every terminal session runs inside a shell.

Shells come in two broad categories: command-line shells, which accept text input and produce text output, and graphical shells, which provide a GUI layer over the OS — Windows Explorer is an example of a graphical shell. The room focuses on Linux command-line shells.

The file `/etc/shells` lists all valid login shells installed on the system. Viewing it on the lab machine:

    cat /etc/shells

This returns: `/bin/sh`, `/bin/bash`, `/usr/bin/bash`, `/bin/rbash`, `/usr/bin/rbash`, `/bin/dash`, `/usr/bin/dash`, `/usr/bin/tmux`, `/usr/bin/screen`, `/bin/zsh`, `/usr/bin/zsh`.

### Types of Linux Shells

**Bash (Bourne Again Shell)** is the default shell in most Linux distributions. It was developed as an enhanced replacement for earlier shells — `sh`, `ksh`, and `csh` — borrowing capabilities from all of them. Bash provides powerful scripting capabilities, command history, tab completion, and job control. It is the shell used for all scripting examples in this room and the one most security tools and automation scripts target. Bash does not come with out-of-the-box syntax highlighting.

**Fish (Friendly Interactive Shell)** focuses on ease of use and interactivity. Its distinguishing features are syntax highlighting as an out-of-the-box feature (no configuration required), auto spell correction, and auto-suggestions based on command history. It is available on Windows, macOS, and Linux. Fish does not auto spell correct when compared to Zsh, but it does provide syntax highlighting which Bash does not.

**Zsh (Z Shell)** is a modern shell that combines features from several predecessors. It is not installed by default in most Linux distributions. Its distinguishing features are advanced tab completion, auto spell correction, and extensive customisation options — including themes and plugins via frameworks like Oh My Zsh. The depth of customisation available in Zsh can make it slower than Bash or Fish. Zsh provides syntax highlighting through plugins rather than out-of-the-box.

A comparison of the three on the features most relevant to the room's questions:

| Feature | Bash | Fish | Zsh |
|---|---|---|---|
| Default in most Linux distros | Yes | No | No |
| Syntax highlighting (out of box) | No | Yes | No (plugin required) |
| Auto spell correction | No | Yes | Yes |
| Advanced tab completion | Basic | Yes | Yes |
| Scripting capability | Excellent | Limited | Excellent |

The command to display all previously executed commands in the current session is `history`.

### Switching and Changing Shells

To switch to a different shell for the current session, type its name at the prompt:

    zsh
    fish

To permanently change the default shell, use `chsh` (change shell):

    chsh -s /usr/bin/zsh

This change takes effect on the next login. The shell must be listed in `/etc/shells` for `chsh` to accept it.

### Bash Scripting

A shell script is a file containing a sequence of commands that the shell executes in order. Instead of typing commands one by one, a script combines them into a single executable file. Bash scripts use the `.sh` extension by convention.

**Script structure — the four fundamental components:**

**Shebang (`#!`)** — the first line of every script. It specifies which interpreter should execute the file. Without it, the OS does not know which shell to use:

    #!/bin/bash

**Variables** — store values for use throughout the script. No spaces around the `=` operator. String values are enclosed in quotes:

    username=""
    directory="/var/log"

To read user input into a variable:

    read username

**Conditional statements** — execute different code based on conditions. Comparison operators in Bash use flags: `-eq` (equal, numeric), `-ne` (not equal), `=` (equal, string), `-gt` (greater than), `-lt` (less than):

    if [ "$username" = "John" ]; then
        echo "Access granted"
    else
        echo "Access denied"
    fi

**Loops** — execute a block of commands repeatedly. A `for` loop iterating a fixed range:

    for i in {1..3}; do
        echo "Iteration $i"
    done

A `for` loop iterating over files matching a pattern:

    for file in /var/log/*.log; do
        echo "Checking $file"
    done

**Making a script executable** — scripts require execute permission before they can be run directly:

    chmod +x script.sh
    ./script.sh

### The Locker Script — Practical Scripting Task

The room's scripting task requires writing a complete authentication script from scratch using Nano. The script prompts for a username, company name, and PIN across three iterations of a loop, then validates the inputs against hardcoded correct values. The correct PIN to authenticate successfully is `7385`:

    #!/bin/bash
    username=""
    companyname=""
    pin=""
    for i in {1..3}; do
        if [ "$i" -eq 1 ]; then
            echo "Enter your Username:"
            read username
        elif [ "$i" -eq 2 ]; then
            echo "Enter your Company name:"
            read companyname
        else
            echo "Enter your PIN:"
            read pin
        fi
    done
    if [ "$username" = "John" ] && [ "$companyname" = "Tryhackme" ] && [ "$pin" = "7385" ]; then
        echo "Authentication Successful. You can now access your locker, John."
    else
        echo "Authentication Denied!!"
    fi

### The Log Search Script — Modifying an Existing Script

The second practical task involves a pre-placed script in `/home/user` on the lab machine. The script is designed to search for a specific keyword across all `.log` files in a specified directory. The task requires switching to root with `sudo su` to access the target log directory, then editing the script with Nano to set the correct directory (`/var/log`) and the correct flag string (`thm-flag01-script`). The script uses a `for` loop over `.log` files and `grep -q` to silently search each file, printing the filename if the flag is found.

---

## Walkthrough Notes

The room runs through six tasks. All practical work is done on the in-browser Ubuntu machine deployed within the room.

**Task 1 (Introduction):** Introduces the shell as the interface between user and OS. Notes the contrast between GUI and CLI interaction and the efficiency advantage of CLI for security work. No answer required.

**Task 2 (Linux Shells):** Covers shell types — Bash, Fish, and Zsh. Questions: which shell comes with syntax highlighting as an out-of-the-box feature — the answer is `Fish`; which shell does not have auto spell correction — the answer is `Bash`; the command that displays all previously executed commands in the current session is `history`.

**Task 3 (Bash — The Default Shell):** Covers Bash specifically and how to view available shells with `cat /etc/shells`. The question asks how to permanently change the default shell — the answer uses `chsh -s /path/to/shell`. No standalone flag question.

**Task 4 (Shell Scripting):** Covers the four scripting components — shebang, variables, conditionals, and loops. Questions: the shebang line for a Bash script is `#!/bin/bash`; variables are declared without a `$` prefix on the left side of the assignment (the `$` is used when referencing the variable). The task confirms the four main components of a shell script — shebang, variables, loops, and conditional statements.

**Task 5 (Locker Script — Practical):** Requires writing the full locker authentication script in Nano and running it with `./locker_script.sh`. The correct PIN to authenticate is `7385`.

**Task 6 (Log Search Script — Practical):** Requires switching to root with `sudo su`, editing the pre-placed script to set the directory to `/var/log` and the flag string to `thm-flag01-script`, making the script executable with `chmod +x`, and running it. The script searches all `.log` files in `/var/log` and returns the flag when the matching string is found.

---

## Commands Used

    cat /etc/shells
    history
    chsh -s /usr/bin/zsh
    zsh
    fish
    sudo su
    nano locker_script.sh
    chmod +x locker_script.sh
    ./locker_script.sh
    nano log_search.sh
    chmod +x log_search.sh
    ./log_search.sh

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| Shell identification (`/etc/shells`) | Post-compromise enumeration — reviewing available shells on a compromised host reveals what scripting environments an attacker can use; unexpected shells in `/etc/shells` may indicate attacker modifications |
| `history` command | Forensic evidence collection — the shell history file (`~/.bash_history`) records commands executed by a user; reviewing it post-compromise surfaces attacker commands, tool downloads, and lateral movement attempts |
| Bash scripting for log searching | Automated log analysis — scripts that iterate over log files and search for specific strings are how analysts process large volumes of log data efficiently; the pattern in the lab task (`for file in *.log; do grep -q "$flag" "$file"`) is a production-valid approach |
| `chmod +x` and script execution | Malware deployment awareness — attackers frequently download scripts and make them executable with `chmod +x` before running them; this sequence (`wget URL`, `chmod +x script.sh`, `./script.sh`) is a common malware delivery pattern visible in shell history |
| Shell switching (`chsh`) | Persistence and evasion — an attacker modifying a user's default shell to a non-standard interpreter or a custom binary achieves persistence that survives reboots and appears in `/etc/passwd` rather than the more commonly reviewed startup locations |
| `sudo su` for root access | Privilege escalation monitoring — `sudo su` in shell history indicates a privilege escalation event; sudo logs in `/var/log/auth.log` record every `sudo` invocation with timestamp and username |

---

## Takeaways

1. **`~/.bash_history` is a forensic gift when it has not been cleared.** It records commands in the order they were executed, providing a timeline of what a user — or an attacker operating as a user — did in a session. Attackers who are aware of this either delete the file (`rm ~/.bash_history`), redirect it to `/dev/null`, or prepend commands with a space (which Bash is often configured to exclude from history). The presence or absence of history, and the presence of commands to suppress it, are both meaningful signals during investigation.

2. **Shell scripting is not a supplementary skill — it is how security automation actually works at the command line.** Log parsing, alert triage, file searching, credential rotation, system hardening checks — all of these are routinely scripted in Bash. The locker script and log search script in this room are simple, but the pattern they demonstrate — variables, loops, conditionals, file iteration, and `grep` — composes into scripts that handle real operational tasks. Every additional scripting concept mastered multiplies the analyst's ability to automate the repetitive parts of their workflow.

3. **The shell is the first thing to check and the last thing defenders think about.** Entry points, persistence mechanisms, and command execution all flow through the shell. Shell history, shell configuration files (`.bashrc`, `.bash_profile`), and modifications to `/etc/shells` or `/etc/passwd` are all places where attacker presence can be detected. Because the shell is so fundamental to how the OS works, changes to it are often overlooked in favour of more obvious indicators. Developing the habit of checking shell configuration and history as a first step in host investigation closes a gap that many checklists miss.

---
