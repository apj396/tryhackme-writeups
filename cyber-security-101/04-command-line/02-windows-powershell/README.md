# Windows PowerShell

| Field | Details |
|---|---|
| **Module** | Cyber Security 101 — Command Line |
| **Difficulty** | Easy |
| **Platform** | TryHackMe |
| **Room Link** | https://tryhackme.com/room/windowspowershell |
| **Date Completed** | February 2026 |
| **Author** | Adwait Joshi |

---

## What This Room Covers

This is the second room in the CS101 Command Line section. Windows Command Line covered `cmd.exe` — the legacy interpreter. This room introduces PowerShell: Microsoft's modern, object-oriented shell and scripting framework built on the .NET runtime. PowerShell was designed to address the fundamental limitations of `cmd.exe` — its inability to work with structured data, its poor scripting capabilities, and its restricted integration with Windows internals. The room covers what PowerShell is and why it works differently from text-based shells, core cmdlets, filesystem navigation, filtering and sorting output, retrieving system and network information, scripting basics, and how to extend PowerShell with external modules. Because PowerShell is simultaneously the most capable administration tool on Windows and one of the most abused by attackers, understanding it is directly relevant to both offensive and defensive security work.

---

## Key Concepts

### What PowerShell Is

PowerShell is a task automation and configuration management framework developed by Microsoft, first released in 2006. It integrates a command-line interface with a full scripting language and is built on the .NET framework. The advanced approach used to develop PowerShell is object-oriented — unlike `cmd.exe` and traditional Unix shells which pass text between commands, PowerShell passes .NET objects through its pipeline. This means the output of one cmdlet is not a string to be parsed, but a structured object with properties and methods that the next cmdlet can work with directly.

PowerShell commands follow a consistent `Verb-Noun` naming convention. This makes commands predictable and discoverable — even for an unfamiliar cmdlet, knowing the verb gives a reliable indication of what it does.

### Core Cmdlets and Help

**`Get-Command`** lists all available cmdlets, functions, aliases, and scripts in the current session. It supports filtering — to retrieve a list of commands that start with the verb `Remove`:

    Get-Command -Name Remove*

**`Get-Help`** is the PowerShell equivalent of `man` — it displays documentation for any cmdlet including description, syntax, parameters, and examples. To retrieve example usage for a cmdlet:

    Get-Help New-LocalUser -Examples

**`Get-Alias`** lists all aliases defined in the current session — shortcuts that map familiar commands to their PowerShell equivalents. The cmdlet whose traditional counterpart `echo` is an alias for is `Write-Output`. Other common aliases: `ls` → `Get-ChildItem`, `cd` → `Set-Location`, `cat` → `Get-Content`, `ps` → `Get-Process`, `pwd` → `Get-Location`.

### Filesystem Navigation and File Operations

PowerShell's filesystem cmdlets mirror Linux commands in function while following the `Verb-Noun` pattern:

| Cmdlet | Alias | Purpose |
|---|---|---|
| `Get-ChildItem` | `ls`, `dir` | Lists files and directories |
| `Set-Location` | `cd` | Changes the current directory |
| `Get-Location` | `pwd` | Returns the current directory path |
| `Get-Content` | `cat`, `type` | Reads and outputs the contents of a file |
| `New-Item` | — | Creates a new file or directory |
| `Remove-Item` | `rm`, `del` | Deletes a file or directory |
| `Copy-Item` | `cp`, `copy` | Copies a file or directory |
| `Move-Item` | `mv`, `move` | Moves a file or directory |

**`Get-ChildItem`** is more powerful than `ls` or `dir` alone. The `-Recurse` flag searches subdirectories. The `-Filter` or `-Include` parameter filters by file extension. The `-Hidden` flag reveals hidden files. To find all `.txt` files recursively from the root:

    Get-ChildItem -Path C:\ -Include *.txt -File -Recurse -ErrorAction SilentlyContinue

### Filtering, Sorting, and Searching

PowerShell's object pipeline enables powerful data manipulation without string parsing:

**`Where-Object`** filters objects in the pipeline based on a condition. It is the PowerShell equivalent of `grep` for structured data:

    Get-ChildItem | Where-Object {$_.Extension -eq ".txt"}

**`Sort-Object`** sorts pipeline output by a specified property. To find the largest file in a directory — sort by Length descending and take the first result:

    Get-ChildItem -Path C:\Users\captain\Documents\captain-cabin | Sort-Object Length -Descending | Select-Object -First 1

**`Select-Object`** selects specific properties from objects or limits the number of results returned.

**`Select-String`** searches file contents for a text pattern — the PowerShell equivalent of `grep` for file content. It fully supports regular expressions:

    Select-String -Path "C:\Logs\*.log" -Pattern "failed"

### System and Network Information

PowerShell provides rich system interrogation through dedicated cmdlets:

**`Get-Process`** lists all running processes with detailed information — PID, CPU time, memory usage, process name, and more. It is the PowerShell equivalent of `tasklist` but returns structured objects rather than formatted text, making it composable with other cmdlets.

**`Get-Service`** lists all services on the system with their status (Running, Stopped) and display name. Combined with `Where-Object`, it can filter for running or stopped services specifically:

    Get-Service | Where-Object {$_.Status -eq "Running"}

**`Get-NetIPConfiguration`** returns network adapter configuration — IP address, default gateway, and DNS server information. It is the PowerShell equivalent of `ipconfig`.

**`Get-NetIPAddress`** returns IP address details for all adapters including IPv4 and IPv6 addresses and their prefix lengths.

### Scripting

PowerShell scripts are plain text files with a `.ps1` extension. They combine cmdlets, variables, loops, conditionals, and functions into reusable automation. The key scripting constructs:

**Variables** are declared with a `$` prefix and assigned with `=`:

    $username = "John"

**Conditionals** use `if`, `elseif`, and `else` with comparison operators (`-eq`, `-ne`, `-gt`, `-lt`, `-like`, `-match`):

    if ($username -eq "John") { Write-Output "Access granted" }

**Loops** use `foreach`, `for`, `while`, and `do-while`:

    foreach ($file in Get-ChildItem) { Write-Output $file.Name }

**Script execution policy** controls which scripts are permitted to run. The default policy on most systems is `Restricted` — no scripts can run. To check the current policy:

    Get-ExecutionPolicy

To change it (requires Administrator):

    Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

`RemoteSigned` allows locally created scripts to run but requires downloaded scripts to be signed by a trusted publisher. This is the most commonly used permissive policy in managed environments.

Scripts are executed by specifying their path:

    .\script.ps1

The `.\` prefix is required because PowerShell does not search the current directory by default — unlike `cmd.exe`.

### Extending PowerShell — Modules

PowerShell's functionality is extended through modules — packages of cmdlets, functions, and providers. Modules are distributed via the PowerShell Gallery, a public repository. The `PowerShellGet` module provides the tooling to find, install, and manage modules:

    Find-Module -Name ModuleName
    Install-Module -Name ModuleName

Modules must be imported into the current session before their cmdlets are available:

    Import-Module ModuleName

---

## Walkthrough Notes

The room runs through eight tasks. All practical work is conducted on the VM deployed within the room, accessed via the in-browser terminal or SSH.

**Task 1 (Introduction):** Introduces PowerShell as the second command-line utility built for Windows. Notes that it was historically the second — after `cmd.exe`. No answer required.

**Task 2 (What Is PowerShell):** Covers the object-oriented design and .NET foundation. The question asks what advanced approach was used to develop PowerShell — the answer is `Object-Oriented`.

**Task 3 (PowerShell Basics):** Covers `Get-Command`, `Get-Help`, and `Get-Alias`. Questions: to retrieve commands starting with `Remove`, the answer is `Get-Command -Name Remove*`; the cmdlet aliased to `echo` is `Write-Output`; the command to retrieve example usage for `New-LocalUser` is `Get-Help New-LocalUser -Examples`.

**Task 4 (Navigating the File System and Working with Files):** Covers `Get-ChildItem`, `Set-Location`, `Get-Content`, `New-Item`, `Copy-Item`, and `Move-Item`. Questions require navigating to a specified directory, listing contents, reading a file, and using `Get-ChildItem` with `-Recurse` to find files across subdirectories. The practical question asks for the content of a specific file in the captain's directory — navigated to with `Set-Location` and read with `Get-Content`.

**Task 5 (Piping, Filtering, and Sorting Data):** Covers `Where-Object`, `Sort-Object`, `Select-Object`, and `Select-String`. The pipeline exercise requires sorting files by size descending and selecting the first result to find the largest file in the `captain-cabin` directory. Questions on `Select-String` ask which cmdlet searches for text patterns — the answer is `Select-String`.

**Task 6 (System and Network Information):** Covers `Get-Process`, `Get-Service`, `Get-NetIPConfiguration`, and `Get-NetIPAddress`. Questions require running these cmdlets on the VM and reading the output — the listening port, running services, and IP configuration. The question asking which cmdlet retrieves all services is answered by `Get-Service`.

**Task 7 (Scripting):** Covers `.ps1` files, execution policy, variables, conditionals, and loops. Questions: the file extension for PowerShell scripts is `.ps1`; the cmdlet to check the current execution policy is `Get-ExecutionPolicy`; the default execution policy that prevents all scripts from running is `Restricted`.

**Task 8 (Conclusion):** Summarises the room — cmdlets, pipeline, filesystem operations, system information, scripting, and modules — and points to Linux Shells as the next room. No answer required.

---

## Commands Used

    Get-Command -Name Remove*
    Get-Help New-LocalUser -Examples
    Get-Alias
    Get-ChildItem
    Get-ChildItem -Path C:\ -Include *.txt -File -Recurse -ErrorAction SilentlyContinue
    Set-Location C:\Users\captain\Documents
    Get-Location
    Get-Content filename.txt
    New-Item -ItemType File -Name newfile.txt
    Copy-Item source.txt destination.txt
    Move-Item source.txt C:\Destination
    Get-ChildItem | Where-Object {$_.Extension -eq ".txt"}
    Get-ChildItem -Path C:\Users\captain\Documents\captain-cabin | Sort-Object Length -Descending | Select-Object -First 1
    Select-String -Path "C:\Logs\*.log" -Pattern "failed"
    Get-Process
    Get-Service
    Get-Service | Where-Object {$_.Status -eq "Running"}
    Get-NetIPConfiguration
    Get-NetIPAddress
    Get-ExecutionPolicy
    Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
    .\script.ps1
    Find-Module -Name ModuleName
    Install-Module -Name ModuleName
    Import-Module ModuleName

---

## Real-World Mapping

| Concept | Real-World Application |
|---|---|
| `Get-Process` with pipeline filtering | Malware hunting — filtering processes by unusual names, paths outside standard directories, or unexpected parent processes; composing with `Where-Object` to isolate anomalies without third-party tools |
| `Get-Service` | Persistence review — listing all services and filtering for those in unexpected states or with unusual names is a standard post-compromise check; attacker-installed services appear here |
| `Select-String` on log files | Log analysis — searching `.log` or `.evtx` files for specific strings (IP addresses, usernames, error codes) across multiple files recursively, equivalent to `grep -r` in Linux investigations |
| Execution policy (`Get-ExecutionPolicy`) | Policy verification and bypass awareness — `Restricted` prevents scripts but does not prevent `cmd.exe` from launching PowerShell with the `-EncodedCommand` flag to run base64-encoded commands; understanding policy scope is necessary for both enforcement and detection |
| `Get-NetIPConfiguration` | Network baseline verification — confirming adapter configuration during investigation; unexpected IP addresses, gateway changes, or DNS server modifications are indicators of network tampering |
| PowerShell module installation | Attacker tooling delivery — `Install-Module` from the PowerShell Gallery or a custom repository can be used to deliver attacker tools that bypass signature-based detection since they are not traditional executables; monitoring for unexpected module installation activity is a detection category |
| Object pipeline | Scripted automation and Living Off The Land — PowerShell's object pipeline allows sophisticated data manipulation without writing to disk, making it attractive for fileless attacks; defenders detect this through script block logging and module logging in PowerShell's operational event logs |

---

## Takeaways

1. **PowerShell's object pipeline is not just a convenience feature — it changes what is possible from the command line.** In `cmd.exe`, chaining commands requires parsing text output with tools like `findstr`, which is fragile and error-prone. In PowerShell, the output of `Get-Process` is a collection of Process objects. Piping that to `Where-Object`, `Sort-Object`, or `Select-Object` filters, sorts, and selects based on structured properties — no parsing required. This composability is why PowerShell replaced `cmd.exe` for serious Windows administration and why it is also the preferred environment for sophisticated attackers.

2. **Execution policy is a convenience setting, not a security boundary.** Microsoft explicitly states this. `Restricted` stops accidental script execution but does not prevent a motivated user or attacker from running PowerShell code — they can bypass policy entirely using `-ExecutionPolicy Bypass` as a command-line parameter, running base64-encoded commands with `-EncodedCommand`, or piping script content directly into PowerShell via `cmd.exe`. Treating execution policy as a defence leads to a false sense of security. The real controls are script block logging, constrained language mode, and application control policies like AppLocker.

3. **PowerShell event logging is one of the most valuable sources of attacker activity on Windows — and it requires deliberate configuration to be useful.** Script block logging (Event ID 4104) records the content of every script block executed, including deobfuscated code. Module logging records all pipeline execution. Transcription logging creates a full session transcript. None of these are enabled by default. An environment without PowerShell logging configured has a significant blind spot — one that attackers actively exploit by preferring PowerShell for post-exploitation precisely because it leaves minimal traces on unmonitored systems.

---
