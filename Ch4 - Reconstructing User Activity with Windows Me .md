# Ch4 - Reconstructing User Activity with Windows Memory Forensics

---

# Summary of Chapter 4: Reconstructing User Activity with Windows Memory Forensics

This chapter from *Practical Memory Forensics* by Svetlana Ostrovskaya provides an in-depth guide to reconstructing user activity through Windows memory forensics, a critical skill for digital forensics investigators. It addresses scenarios involving victim or suspect devices in incidents like hacking, child pornography, human trafficking, or drug dealing. The chapter covers analyzing running processes, opened documents, browser history, communication applications, user passwords, crypto containers, and the Windows Registry in memory, using tools like Volatility, MemProcFS, RegRipper, and Passware. Below is a comprehensive and detailed summary, including all concepts, tools, procedures, commands, and notes, designed to be sufficient for investigators without reading the original chapter.

## Importance of User Activity Reconstruction

- **Purpose**:
    - **Victim’s Device**: Identifies infection methods or attacker actions during remote access, revealing how a breach occurred.
    - **Suspect’s Device**: Uncovers attack preparation, executed actions, and evidence of illegal activities, critical for criminal investigations.
    - **Criminal Cases**: Recovers private communications, browser history, and encryption keys, which are key evidence in cases like child pornography, human trafficking, or drug dealing.
- **Forensic Value**: Memory images contain volatile data unavailable in post-mortem analysis, such as:
    - Running processes (e.g., messengers, anonymous browsers, encrypted containers).
    - Cached credentials, encryption keys, and temporary file data.
- **Applications**: Essential for incident response and criminal investigations, providing insights into user behavior and system state at the time of the memory dump.

## Technical Requirements

- **Minimal Setup**: A Windows operating system (host or virtual machine) is sufficient to run the tools discussed, making them accessible for most forensic setups.

## Analyzing Launched Applications

### Overview

- **Goal**: Analyze running processes to build a suspect’s profile, identifying tools like messengers, high-anonymity browsers (e.g., Tor), or mounted encrypted containers.
- **Process Creation**: When a user launches a program, a process is created and added to the active process list, capturing the system’s state at the time of the memory dump.
- **Forensic Relevance**: Process analysis reveals user activity, such as communication tools or encryption software, which may be unavailable in disk-based forensics.

### Volatility Framework

- **Description**: A free, open-source, Python-based tool for memory dump analysis, widely supported by forensic suites like Autopsy and Magnet AXIOM. It supports Windows (XP and later), Linux, and macOS.
- **Versions and Installation**:
    - **Volatility Standalone (2.6)**:
        - Download: `volatility_2.6_win64_standalone.exe` from [https://www.volatilityfoundation.org/26](https://www.volatilityfoundation.org/26).
        - Usage: Run via Windows PowerShell or Command Prompt. Copy to a convenient location (e.g., `D:\`).
        - Command to Verify: `.\volatility_2.6_win64_standalone.exe --info` (displays help menu and supported OS profiles; Figure 4.1).
    - **Python Scripts**:
        - Clone from: [https://github.com/volatilityfoundation/volatility](https://github.com/volatilityfoundation/volatility).
        - Advantage: More frequent updates and broader profile support compared to standalone.
    - **Volatility Workbench**:
        - Download: [https://www.osforensics.com/tools/volatility-workbench.html](https://www.osforensics.com/tools/volatility-workbench.html).
        - Features: GUI-based, supports recent Volatility versions (including Volatility 3), but lacks full parameter support.
    - **Volatility 3 Limitations**: Under development, missing or underdeveloped plugins, and complex output processing. Volatility 2.6 is preferred for reliability.
- **Support Resources**: Official website, GitHub repository, and community forums provide installation guides, plugin documentation, and community plugins.

### Profile Identification

- **Purpose**: Select the correct OS profile to ensure accurate plugin functionality.
- **Plugin**: `imageinfo` identifies suitable profiles for a memory dump.
- **Command Example**: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem imageinfo`.
- **Output**: Lists suggested profiles (e.g., `Win10x64_14393` for `win10Mem.vmem`; Figure 4.2). The first profile is typically the most suitable.
- **Troubleshooting**:
    - If plugins fail (e.g., no output, incorrect output, or errors), try alternative profiles from the list.
    - For new OS versions without profiles:
        - Search GitHub for community-created profiles.
        - Use Volatility 3 or alternative tools.
        - Write a custom profile (requires advanced programming and OS knowledge).
- **Example Profile**: `Win10x64_14393` used for `win10Mem.vmem`.

### Searching for Active Processes

- **Plugins**:
    - **pslist**: Lists active processes sorted by creation time.
    - **pstree**: Displays parent-child process relationships, mirroring Task Manager’s hierarchy.
- **Command Example**: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem --profile=Win10x64_14393 pslist`.
- **Output**: Includes:
    - Process name.
    - Process ID (PID).
    - Parent Process ID (PPID).
    - Number of handles and threads.
    - Creation time.
    - Exit time (if terminated).
- **Important Note**: Processes have at least one thread and associated handles (references to kernel objects like files, registry keys, or other processes).
- **Visual Reference**: Figure 4.3 (Volatility pslist).

### Searching for Terminated Processes

- **Plugin**: `psscan` searches for `_EPROCESS` structures, including recently terminated processes whose data persists in memory until overwritten.
- **Command Example**: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem --profile=Win10x64_14393 psscan`.
- **Output**: Similar to `pslist`, but includes terminated processes (Figure 4.4).
- **Use Case**: Identifies processes that were active but terminated before the dump, critical for reconstructing past activity.

### Searching for Frequently Run Programs

- **Plugin**: `userassist` extracts data from the `NTUSER.DAT` registry file, detailing frequently run programs.
- **Command Example**: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem --profile=Win10x64_14393 userassist`.
- **Output**:
    - Registry path (e.g., `\??\C:\Users\Ben\ntuser.dat` for user Ben).
    - Application details: full executable path, run count, focus time, last run time (Figures 4.5, 4.6).
- **Use Case**: Reveals programs run earlier, even if not active during the dump.

## Searching for Opened Documents

### Overview

- **Goal**: Identify files (e.g., Microsoft Office documents, text files) opened by applications, which may contain passwords or investigative data.
- **Plugins**:
    - **filescan**: Lists all files in the memory dump.
    - **dumpfiles**: Extracts files from memory.
    - **handles**: Shows resources (e.g., files) used by a specific process.

### Process Example (MS Word)

- **Scenario**: `WINWORD.exe` (PID 1592) is active (Figure 4.7).
- **Steps**:
    1. **List File Resources**:
        - Command: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\Windows7x64.vmem --profile=Win7SP1x64 handles -p 1592 -t File`.
        - Output: Identifies files like `GOT-7_HR` (Figure 4.8).
    2. **Locate File Offset**:
        - Command: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\Windows7x64.vmem --profile=Win7SP1x64 filescan > D:\filescan.txt`.
        - Output: `filescan.txt` contains file paths, physical offsets, and attributes (Figures 4.9, 4.10).
    3. **Extract File**:
        - Command: `.\volatility_2..apache_2.6_win64_standalone.exe -f D:\user activity\Windows7x64.vmem --profile=Win7SP1x64 dumpfiles -Q <physical_offset> -D D:\user activity`.
        - Output: Files like `file.None.0xffffa80282a6b80.vacb` and `file.None.0xffffa80258625f0.dat` (Figure 4.11).
- **File Extensions**:
    - `.dat`: DataSectionObject (file content).
    - `.vacb`: ImageSectionObject (file mapping).
    - `.img`: SharedCacheMap (cached data).
- **Post-Extraction**:
    - Rename extracted files with their original extension (e.g., `.docx` for `GOT-7_HR`) to open with appropriate tools.
    - Some files may be unloaded, resulting in partial or no data.
- **Important Note**: Avoid running potentially malicious extracted files on the analysis machine. Use sandboxes or specialized tools (discussed in Chapter 5).

## Investigating Browser History

### Overview

- **Goal**: Recover visited websites, search queries, and downloaded files, even in private modes or anonymous browsers like Tor.
- **Active Browser Processes**: Identified via `pslist`, e.g., Google Chrome (PID 4236), Mozilla Firefox (PID 6380), Tor Browser (PID 4708; Figure 4.12).

### Methods

1. **Strings Utility**:
    - Tool: Part of Sysinternals Suite ([https://docs.microsoft.com/en-us/sysinternals/downloads/strings](https://docs.microsoft.com/en-us/sysinternals/downloads/strings)).
    - Process: Extracts ASCII/Unicode strings from process memory dumps.
2. **bulk_extractor**:
    - Tool: Command-line tool for extracting URLs, emails, and other data ([https://downloads.digitalcorpora.org/downloads/bulk_extractor/](https://downloads.digitalcorpora.org/downloads/bulk_extractor/)).
    - Optional GUI: BEViewer (requires additional installation).
3. **Regular Expressions/YARA Rules**:
    - Use Volatility’s `yarascan` plugin for pattern-based searches.

### Chrome Analysis with yarascan

- **Plugin**: `yarascan` searches memory using regular expressions or YARA rules.
- **YARA Note**: Originally for malware detection, YARA supports textual/binary pattern searches ([https://yara.readthedocs.io/en/v4.1.0/gettingstarted.html](https://yara.readthedocs.io/en/v4.1.0/gettingstarted.html)).
- **Command Example**: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem --profile=Win10x64_14393 yarascan -p 4236 -Y "(https?:\/\/)?([\w\.-]+)([\/\w\.-]+) /"`.
- **Output**: URLs found in Chrome process memory (PID 4236; Figure 4.13).
- **Use Case**: Quickly identifies visited websites via regular expression pattern matching.

### Firefox Analysis with bulk_extractor

- **Process**:
    1. Dump Firefox process memory (PID 6380):
        - Command: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem --profile=Win10x64_14393 memdump -p 6380 -D D:\user activity\firefox`.
        - Output: `6380.dmp` in `D:\user activity\firefox` (Figure 4.14).
    2. Run bulk_extractor:
        - Command: `bulk_extractor -o D:\user activity\firefox_output -x all -e email D:\user activity\firefox\6380.dmp`.
        - Options:
            - `o`: Output directory.
            - `x all`: Disable all scanners.
            - `e email`: Enable email/URL scanner.
    - **Output**: Files like `url_histogram.txt` (visited URLs) and `url_searches.txt` (search queries, e.g., “Tor Browser”; Figures 4.15–4.17).
- **Note**: Run `bulk_extractor.exe` without options to list available scanners for customization.

### Tor Analysis with Strings

- **Purpose**: Analyze Tor Browser (PID 4708) for deep/dark web activity, which may indicate illegal behavior.
- **Process**:
    1. Dump Tor process memory:
        - Command: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem --profile=Win10x64_14393 memdump -p 4708 -D D:\user activity`.
        - Output: `4708.dmp`.
    2. Run Strings:
        - Command: `strings D:\user activity\4708.dmp > D:\user activity\tor_strings.txt`.
        - Output: Text file with ASCII/Unicode strings, searchable for URLs or keywords (Figures 4.18, 4.19).
- **Note**: Slower and less structured than bulk_extractor but yields more raw data, requiring manual keyword or regular expression searches.

## Examining Communication Applications

### Email Analysis

- **Context**: Emails may contain malicious attachments (victim’s device) or accomplice correspondence (suspect’s device).
- **Methods**:
    1. **Email Agent Process**:
        - Use `handles` to find attachments: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem --profile=Win10x64_14393 handles -p <email_agent_PID> -t File`.
        - Dump process memory: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem --profile=Win10x64_14393 memdump -p <email_agent_PID> -D D:\user activity`.
        - Parse with Strings: `strings D:\user activity\<email_agent_PID>.dmp > D:\user activity\email_strings.txt` to find email fragments or attachments.
    2. **Full Memory Dump with bulk_extractor**:
        - Command: `bulk_extractor -o D:\user activity\email_output -x all -e email D:\user activity\win10Mem.vmem`.
        - Output: Files like `email_histogram.txt` (email addresses) and `url_histogram.txt` (mailboxes/attachments; Figures 4.20–4.22).
        - Note: Comprehensive but slower due to full dump analysis.
- **Use Case**: Identifies email-related evidence regardless of whether an email client or browser was used.

### Instant Messengers

- **Examples**: Telegram, Discord (Figure 4.23).
- **Process**:
    1. Dump process memory (e.g., Telegram PID 1234):
        - Command: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem --profile=Win10x64_14393 memdump -p <messenger_PID> -D D:\user activity`.
    2. Parse with Strings:
        - Command: `strings D:\user activity\<messenger_PID>.dmp > D:\user activity\messenger_strings.txt`.
        - Output: Searchable text file for usernames, messages, URLs, or passwords (Figures 4.24, 4.25).
- **Note**: Users may share passwords via messengers for cross-device access, making them a critical source for credential recovery.

## Recovering User Passwords

### Overview

- **Sources**: Passwords may reside in caches, text editor memory, buffers, command lines, or system processes (e.g., LSA, PsExec).
- **Volatility Plugins**:
    - **hashdump**: Dumps local user password hashes (pre-Windows 8).
    - **cachedump**: Dumps cached domain user password hashes (up to 10 recent credentials).
    - **lsadump**: Extracts Local Security Authority (LSA) data, including plaintext passwords.
    - **cmdline**: Retrieves command-line arguments, potentially containing plaintext passwords.

### hashdump

- **Command Example**: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem --profile=Win10x64_14393 hashdump`.
- **Output**: Account name, relative identifier, LM/NT hashes. Identical hashes for Administrator/Guest indicate blank passwords (Figure 4.26).
- **Limitation**: Effective only for Windows versions before 8.

### cachedump

- **Command Example**: Similar to `hashdump`.
- **Output**: Domain username, password hash, and domain (Figure 4.27).
- **Use Case**: Recovers cached domain credentials, useful for enterprise environments.

### lsadump

- **Command Example**: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem --profile=Win10x64_14393 lsadump`.
- **Output**: LSA data, including plaintext passwords from various resources (Figure 4.28).
- **Use Case**: Critical for user authentication data recovery.

### Plaintext Passwords

- **Plugin**: `cmdline` retrieves command-line arguments without additional options.
- **Command Example**: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem --profile=Win10x64_14393 cmdline`.
- **Output**: System and user command lines, e.g., PsExec with plaintext password (`psexec -u max -p <password> \\win7`; Figures 4.29, 4.30).
- **Use Case**: Identifies passwords transmitted in plaintext by tools like PsExec.

## Detecting Crypto Containers

### Overview

- **Tools**: BitLocker, TrueCrypt, VeraCrypt.
- **Purpose**: Encrypt user data, potentially hiding illegal activity.
- **Forensic Value**: If a container is mounted during dumping, memory may contain cached volume passwords, master encryption keys, or unencrypted file fragments.
- **Initial Step**: Identify encryption tools via `pslist` (e.g., `VeraCrypt.exe`; Figure 4.31).

### Volatility Plugins for TrueCrypt

- **truecryptmaster**: Searches for encryption master keys.
- **truecryptpassphrase**: Searches for passphrases.
- **truecryptsummary**: Collects TrueCrypt process, service, driver, symbolic link, and file object data.
- **Command Example**: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem --profile=Win10x64_14393 truecryptsummary` (Figure 4.32).
- **Output Example**: Identifies encrypted drives and extracts master keys (Figure 4.33).

### AES Key Extraction

- **Tool**: bulk_extractor with AES scanner.
- **Command Example**: `bulk_extractor -o D:\user activity\aes_output -x all -e aes D:\user activity\win10Mem.vmem`.
- **Output**: `aes_keys.txt` with AES256 key pairs, combinable into 512-bit master keys (Figures 4.34, 4.35).
- **Note**: AES is the default encryption for TrueCrypt and similar tools.

### Passware

- **Description**: Commercial tool for detecting encrypted files, decrypting drives, and recovering passwords (supports BitLocker, TrueCrypt, PGP, and Password Managers).
- **Download**: Demo version at [https://www.passware.com/kit-forensic/](https://www.passware.com/kit-forensic/) (Figure 4.36).
- **Use Case**: Simplifies key extraction and decryption compared to manual methods.

## Investigating Windows Registry

### Virtual Registry

- **Definition**: A memory-based registry created at system startup, combining hardware/software configurations, user settings, and more from non-volatile registry files (e.g., SAM, SYSTEM, SOFTWARE, NTUSER.DAT).
- **Forensic Value**: Stores recent data on program execution, opened documents, RDP connections, and malware persistence traces.
- **Key Files**:
    - **SAM**: User/group privileges, passwords, last login.
    - **SYSTEM**: OS data (computer name, services, USB devices, time zone, network adapters).
    - **SOFTWARE**: Installed software, scheduled tasks, autorun, compatibility.
    - **NTUSER.DAT**: User-specific data (recent documents, programs, explorer history, RDP connections).
    - **usrclass.dat**: Shellbag data (directory access, including encrypted drives).
- **Note**: Plugins like `userassist`, `hashdump`, and `cachedump` rely on registry data.

### Volatility Plugins

- **printkey**: Displays registry keys, subkeys, and values.
- **hivelist**: Lists accessible registry hives.
- **dumpregistry**: Extracts registry files from memory.
- **Specialized Plugins**:
    - **userassist**: Extracts program execution data.
    - **shutdowntime**: Retrieves system shutdown times.
    - **shellbags**: Analyzes directory access.
- **Command Example**: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem --profile=Win10x64_14393 hivelist`.
- **Limitation**: Limited compatibility with newer Windows 10 builds.

### MemProcFS

- **Description**: Mounts memory dumps as a virtual filesystem, simplifying registry access.
- **Installation**:
    - **Dependencies**:
        - LeechCore: Install via PowerShell (Figure 4.37).
        - Microsoft Visual C++ Redistributables 2019: [https://go.microsoft.com/fwlink/?LinkId=746572](https://go.microsoft.com/fwlink/?LinkId=746572).
        - Dokany (Dokansetup_redist): [https://github.com/dokan-dev/dokany/releases/latest](https://github.com/dokan-dev/dokany/releases/latest).
        - Python 3.6+: [https://www.python.org/downloads/windows/](https://www.python.org/downloads/windows/).
    - **Download**: `files_and_binaries` ZIP from [https://github.com/ufrisk/MemProcFS](https://github.com/ufrisk/MemProcFS) (Figure 4.38).
    - **Command Example**: `python memprocfs.py -device D:\user activity\win10Mem.vmem` (mounts as `M:\` drive; Figure 4.39).
- **Registry Access**: Navigate to `M:\registry\hive_files` for files like `ntuser.dat`, `usrclass.dat` (Figure 4.40).

### Registry Analysis Tools

- **Registry Explorer**:
    - Download: [https://ericzimmerman.github.io/#!index.md](https://ericzimmerman.github.io/#!index.md) (requires 7-Zip to unpack).
    - Features: View registry keys/values, use bookmarks for forensic data (Figure 4.41).
- **RegRipper**:
    - Download: [https://github.com/keydet89/RegRipper3.0](https://github.com/keydet89/RegRipper3.0).
    - Process: Run `rr.exe`, specify hive file (e.g., `M:\registry\hive_files\0xffffd48e3523000-ntuser.dat`) and output file (e.g., `D:\ntuser_report.txt`), click **Rip!** (Figure 4.42).
    - Output: `.log` (program log) and `.txt` (parsed registry data). Search for `userassist`, `opensave`, `recentdocs` (Figures 4.43, 4.44).
- **ShellbagsExplorer**:
    - Download: [https://ericzimmerman.github.io/#!index.md](https://ericzimmerman.github.io/#!index.md).
    - Process: Load `usrclass.dat` to view directory access, including encrypted drives (e.g., `S:` drive encrypted by TrueCrypt; Figure 4.45).
- **Note**: Registry forensics is complex; this chapter focuses on key files and tools for user activity reconstruction.

## Summary of Key Points

- **User Activity Reconstruction**: Critical for understanding infection vectors, attacker actions, or illegal activities, recovering volatile data like communications, browser history, and encryption keys.
- **Volatility Framework**: Primary tool with plugins for process analysis (`pslist`, `pstree`, `psscan`, `userassist`), file extraction (`filescan`, `dumpfiles`, `handles`), browser history (`yarascan`, `memdump`), password recovery (`hashdump`, `cachedump`, `lsadump`, `cmdline`), and TrueCrypt analysis (`truecryptmaster`, `truecryptpassphrase`, `truecryptsummary`).
- **Additional Tools**:
    - Strings: Extracts raw strings from process memory.
    - bulk_extractor: Extracts URLs, emails, and AES keys.
    - MemProcFS: Provides virtual filesystem for registry access.
    - Registry Explorer/RegRipper/ShellbagsExplorer: Parse registry hives for user activity.
    - Passware: Simplifies decryption and password recovery.
- **Registry Analysis**: Virtual registry in memory contains critical user activity data, accessible via MemProcFS or Volatility plugins.
- **Crypto Containers**: Identifying mounted containers and extracting keys can unlock hidden evidence.
- **Comprehensive Approach**: Combine process, file, browser, communication, password, and registry analysis for a complete user activity timeline.

## Notes for Digital Forensics Investigators

- **Critical Tools and Paths**:
    - **Volatility 2.6**: `volatility_2.6_win64_standalone.exe` from [https://www.volatilityfoundation.org/26](https://www.volatilityfoundation.org/26). Key commands:
        - Profile: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem imageinfo`.
        - Processes: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem --profile=Win10x64_14393 pslist`.
        - Files: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\Windows7x64.vmem --profile=Win7SP1x64 dumpfiles -Q <offset> -D D:\user activity`.
        - Browser: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem --profile=Win10x64_14393 yarascan -p 4236 -Y "(https?:\/\/)?([\w\.-]+)([\/\w\.-]+) /"`.
        - Passwords: `.\volatility_2.6_win64_standalone.exe -f D:\user activity\win10Mem.vmem --profile=Win10x64_14393 hashdump`.
    - **Strings**: [https://docs.microsoft.com/en-us/sysinternals/downloads/strings](https://docs.microsoft.com/en-us/sysinternals/downloads/strings). Example: `strings D:\user activity\4708.dmp > D:\user activity\tor_strings.txt`.
    - **bulk_extractor**: [https://downloads.digitalcorpora.org/downloads/bulk_extractor/](https://downloads.digitalcorpora.org/downloads/bulk_extractor/). Example: `bulk_extractor -o D:\user activity\email_output -x all -e email D:\user activity\win10Mem.vmem`.
    - **MemProcFS**: [https://github.com/ufrisk/MemProcFS](https://github.com/ufrisk/MemProcFS). Mount: `python memprocfs.py -device D:\user activity\win10Mem.vmem`.
    - **Registry Explorer/ShellbagsExplorer**: [https://ericzimmerman.github.io/#!index.md](https://ericzimmerman.github.io/#!index.md).
    - **RegRipper**: [https://github.com/keydet89/RegRipper3.0](https://github.com/keydet89/RegRipper3.0). Run: `rr.exe`.
    - **Passware**: [https://www.passware.com/kit-forensic/](https://www.passware.com/kit-forensic/).
- **Important Notes**:
    - **Profile Accuracy**: Use `imageinfo` to select the correct OS profile, retrying alternatives if plugins fail.
    - **Malicious Files**: Analyze extracted files (e.g., `file.None.0xffffa80282a6b80.vacb`) in sandboxes to prevent infection.
    - **Registry Files**: Access `SAM`, `SYSTEM`, `SOFTWARE`, `NTUSER.DAT`, `usrclass.dat` in `M:\registry\hive_files` with MemProcFS.
    - **YARA Customization**: Create rules for specific URL or malware patterns ([https://yara.readthedocs.io/en/v4.1.0/gettingstarted.html](https://yara.readthedocs.io/en/v4.1.0/gettingstarted.html)).
    - **bulk_extractor Scanners**: List available scanners with `bulk_extractor.exe` to optimize analysis.
    - **File Extraction**: Rename extracted files with correct extensions and verify integrity before analysis.
- **Sensitive Considerations**:
    - **Password Recovery**: Plaintext passwords in command lines (e.g., PsExec) or messenger memory are high-value but must be handled securely to avoid exposure.
    - **Crypto Containers**: Detecting mounted containers (e.g., VeraCrypt) and extracting keys can unlock critical evidence, but manual AES key combination is complex; consider Passware for efficiency.
    - **Tor Browser**: Deep/dark web activity may indicate illegal behavior; Strings analysis is data-rich but requires manual filtering.
    - **Registry Analysis**: `NTUSER.DAT` (recent programs/documents) and `usrclass.dat` (directory access, including encrypted drives) are critical for reconstructing user timelines.
    - **Full Dump Analysis**: bulk_extractor on entire dumps is comprehensive but time-intensive; prioritize process-specific analysis for efficiency when feasible.
    - **Evidence Preservation**: Minimize system interaction during analysis to avoid overwriting volatile data, and verify extracted file integrity.

This detailed summary provides a complete reference for reconstructing user activity in Windows memory forensics, equipping investigators with the knowledge, tools, and precautions needed to conduct thorough and secure investigations without referring to the original chapter.