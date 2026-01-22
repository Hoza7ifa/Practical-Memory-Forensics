# Ch8 - User Activity Reconstruction

---

# 🌟 Comprehensive Summary: User Activity Reconstruction (Chapter 8) from *Practical Memory Forensics* 🌟

This summary covers Chapter 8, "User Activity Reconstruction," from *Practical Memory Forensics* by Svetlana Ostrovskaya, detailing the process of analyzing user activity on Linux-based systems during forensic investigations and incident responses. 🕵️‍♂️ The chapter emphasizes the importance of Linux systems in attacker activities, as they are commonly used for network scanning, vulnerability testing, and post-exploitation frameworks. It provides a thorough guide to investigating launched programs, Bash history, recent files, filesystem recovery, browsing history, communication applications, mounted devices, and crypto containers, using tools like Volatility, Bulk Extractor, and PhotoRec. This detailed overview ensures you have all the necessary information for memory forensics without reading the chapter directly. 🧠

---

## 🧠 Introduction to User Activity Reconstruction

Reconstructing user activity is critical in forensic investigations to gather evidence from both victim and attacker hosts, particularly on Linux-based systems, which are frequently used by attackers for their activities. Linux’s popularity in enterprise environments and its support for tools like network scanners, web application security testers, and post-exploitation frameworks (e.g., Metasploit) make it a prime target for malicious activity. Analyzing user activity reveals details about tools, techniques, attack preparation, affiliations, and forum activity, aiding in incident response and threat attribution.

### 💡 Key Topics Covered

- **Investigating Launched Programs**: Identify running processes and their details.
- **Analyzing Bash History**: Recover executed commands to understand user actions.
- **Searching for Recent Files**: Locate files accessed or opened by users.
- **Recovering Filesystem from Memory**: Extract filesystem structures and files.
- **Checking Browsing History**: Analyze browser activity and visited URLs.
- **Investigating Communication Applications**: Examine messengers and email clients.
- **Looking for Mounted Devices**: Identify connected devices and their timelines.
- **Detecting Crypto Containers**: Find encryption tools and keys in memory.

### 💡 Forensic Applications

- **Victim Hosts**: Reveal files, systems, or tools targeted by attackers.
- **Attacker Hosts**: Uncover tools (e.g., Metasploit, Nmap), techniques, and artifacts like password dictionaries (e.g., rockyou.txt).
- **Incident Context**: Reconstruct the sequence of events, user actions, and attacker motives.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Linux systems are critical in enterprise environments and attacker operations, making user activity analysis essential for evidence collection.
- **Pro Tip**: Document all findings (e.g., process IDs, file paths, timestamps) to maintain chain-of-custody and support legal proceedings.
- **Sensitive Note**: Attackers may manipulate logs or history files to hide their tracks—cross-check memory data with disk artifacts to detect tampering! 🚨

---

## 🛠️ Technical Requirements

The chapter uses both Linux and Windows systems for analysis, with specific tools and configurations:

- **Linux System**: Ubuntu 21.04 (Hirsute Hippo) running Volatility 2.6.1 with built-in utilities.
- **Windows System**: Runs tools like Bulk Extractor and PhotoRec for file recovery and data parsing.
- **Volatility Setup**: Requires custom profiles (created in Chapter 7) placed in the Volatility profiles folder, specified using the `-plugins` option.

### 💡 Commands to Verify Profiles

- Check available Volatility profiles:
    
    ```bash
    $ vol.py --info | grep -i linux
    
    ```
    
    - **Output**: Lists available Linux profiles (e.g., Ubuntu, Debian) for use with Volatility plugins.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Proper tool setup ensures compatibility and accurate analysis.
- **Pro Tip**: Verify profile availability before analysis to avoid errors—store profiles in a centralized repository for reuse.
- **Sensitive Note**: Incorrect profiles can lead to invalid results—always match the profile to the target system’s distribution and kernel version! 🚨

---

## 🛠️ Investigating Launched Programs

Analyzing running processes reveals what programs a user was executing, providing insights into their activities. Volatility plugins like `linux_pslist`, `linux_pstree`, and `linux_psaux` are used to list processes and their details.

### ✨ Key Plugins and Usage

1. **linux_pslist**: Lists all active processes and kernel threads.
    
    ```bash
    $ vol.py --profile=LinuxUbuntu-5_11_0-34-genericx64 -f mem.lime linux_pslist
    
    ```
    
    - **Output**: Includes process name, PID, UID, and DTB (Directory Table Base, the physical offset of the process address space).
    - **Note**: Kernel threads lack a DTB as they use the kernel address space.
    - **Forensic Value**: Identifies all running processes, including user and system activities.
2. **linux_pstree**: Displays processes in a tree structure, showing parent-child relationships.
    - **Forensic Value**: Helps identify process hierarchies, useful for detecting spawned malicious processes.
3. **linux_psaux**: Provides detailed process information, including program locations and command-line arguments.
    
    ```bash
    $ vol.py --profile=LinuxUbuntu-5_11_0-34-genericx64 -f mem.lime linux_psaux | grep 1000
    
    ```
    
    - **Output**: Lists processes for a specific user (e.g., UID 1000), including paths and arguments (e.g., `libreoffice --calc c1.txt`, `nano passwords.txt`).
    - **Forensic Value**: Reveals specific files opened (e.g., `c1.txt` in LibreOffice Calc, `passwords.txt` in nano) and program locations.
4. **linux_psenv**: Displays environment variables for a specific process.
    
    ```bash
    $ vol.py --profile=LinuxUbuntu-5_11_0-34-genericx64 -f mem.lime linux_psenv -p 23333
    
    ```
    
    - **Output**: Shows variables like `USER=itsupport`, confirming the user account (e.g., `itsupport` for PID 23333, a bash process).
    - **Forensic Value**: Identifies the user running a process when `/etc/passwd` is unavailable in memory.

### 💡 Example Findings

- **User Activity**: UID 1000 (e.g., `itsupport`) ran `bash`, `nano`, `thunderbird`, and `libreoffice`.
- **File Interactions**: `libreoffice --calc c1.txt` and `nano passwords.txt` indicate specific files opened on the Desktop.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Process analysis reveals user and attacker activities, including GUI and terminal-based programs.
- **Pro Tip**: Filter processes by UID using `grep` to focus on a specific user’s activities.
- **Sensitive Note**: Attackers may mask processes—cross-reference with `linux_psaux` to ensure all arguments and paths are captured! 🚨

---

## 🛠️ Analyzing Bash History

The Bash shell, preinstalled on most Linux distributions, is widely used for executing commands and scripts. Bash history, stored in `~/.bash_history`, logs user commands but can be manipulated by attackers to hide their tracks.

### ✨ Key Plugin: linux_bash

- **Command**:
    
    ```bash
    $ vol.py --profile=LinuxUbuntu-5_11_0-34-genericx64 -f mem.lime linux_bash
    
    ```
    
- **Output**: Lists commands executed in Bash, e.g.:
    - `cat passwords` (attempt to view file contents).
    - `touch passwords` (create the file).
    - `nano passwords` (edit the file).
    - `ping` (network check).
    - `apt install gui` (install a GUI package).
- **Forensic Value**:
    - **Victim Hosts**: Reveals files or systems targeted by attackers.
    - **Attacker Hosts**: Shows tools (e.g., `msfconsole` for Metasploit, `nmap` for scanning) and artifacts (e.g., `rockyou.txt` for brute-forcing).

### 💡 Example Findings (Attacker Host)

- **Commands**:
    - `msfconsole` (Metasploit post-exploitation framework).
    - `nmap` (network scanning).
    - `rockyou.txt` (password dictionary for brute-forcing).
- **Forensic Insight**: Indicates attacker techniques and tools used in the attack.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Bash history provides a direct view of user and attacker command-line activity.
- **Pro Tip**: Cross-check `linux_bash` output with disk-based `~/.bash_history` to detect tampering.
- **Sensitive Note**: Attackers may disable history logging or edit `~/.bash_history`—rely on memory analysis to recover commands! 🚨

---

## 🛠️ Searching for Opened Documents

Linux systems have less detailed file access logging than Windows, but Volatility plugins can identify and extract files opened by processes.

### ✨ Key Plugins

1. **linux_psaux**: Lists files opened at process startup.
    - **Example**: `libreoffice --calc c1.txt`, `nano passwords.txt`.
    - **Limitation**: Does not show files opened during runtime.
2. **linux_lsof**: Lists open file descriptors for a specific process.
    
    ```bash
    $ vol.py --profile=LinuxUbuntu-5_11_0-34-genericx64 -f mem.lime linux_lsof -p 23333 | grep xls
    
    ```
    
    - **Output**: Shows `c1.xls` opened by LibreOffice (PID 23333).
    - **Forensic Value**: Identifies files accessed during program execution.
3. **linux_find_file**: Lists or extracts cached files in memory.
    - **List Files**:
        
        ```bash
        $ vol.py --profile=LinuxUbuntu-5_11_0-34-genericx64 -f mem.lime linux_find_file -L > cached_files.txt
        
        ```
        
        - **Output**: Lists directories, files, inode numbers, and memory addresses.
        - **Note**: Inodes store metadata (e.g., file type, permissions) but not file content or names.
    - **Search for a File**:
        
        ```bash
        $ vol.py --profile=LinuxUbuntu-5_11_0-34-genericx64 -f mem.lime linux_find_file -F c1.xls
        
        ```
        
        - **Output**: Shows inode and location if found.
    - **Extract a File**:
        
        ```bash
        $ vol.py --profile=LinuxUbuntu-5_11_0-34-genericx64 -f mem.lime linux_find_file -i <inode> -O /path/to/output/c1.xls
        
        ```
        
        - **Output**: Saves the file to the specified path.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Identifying opened files reveals user and attacker interactions with critical data.
- **Pro Tip**: Save `linux_find_file -L` output to a file to manage large lists of cached files.
- **Sensitive Note**: File extraction may fail if data is not fully cached in memory—use multiple plugins to confirm findings! 🚨

---

## 🛠️ Recovering the Filesystem

Volatility can recover portions of the filesystem cached in memory, leveraging inode metadata to reconstruct directories and files.

### ✨ Key Plugin: linux_recover_filesystem

- **Command**:
    
    ```bash
    $ vol.py --profile=LinuxUbuntu-5_11_0-34-genericx64 -f mem.lime linux_recover_filesystem -D recovered_fs
    
    ```
    
- **Output**: Saves recovered filesystem to the `recovered_fs` directory, including:
    - Standard directories (e.g., `/bin`, `/etc`, `/home`).
    - Swap file (Linux equivalent of Windows’ pagefile).
- **Forensic Value**: Recovers critical files like `/etc/passwd`, `~/.bash_history`, and system logs.

### ✨ Filesystem Hierarchy Standard (FHS)

- **Definition**: Maintained by the Linux Foundation, FHS defines the directory structure and contents in Linux distributions.
- **Key Directories**:
    - `/bin/`: Essential user command binaries.
    - `/boot/`: Boot loader files.
    - `/dev/`: Device files.
    - `/etc/`: System configuration (e.g., `/etc/passwd`, `/etc/os-release`).
    - `/home/`: User home directories (e.g., `/home/itsupport/.bash_history`).
    - `/lib/`: Shared libraries and kernel modules.
    - `/media/`: Mount point for removable media.
    - `/mnt/`: Temporary mount point.
    - `/opt/`: Add-on software packages.
    - `/sbin/`: System binaries.
    - `/srv/`: Service data.
    - `/tmp/`: Temporary files.
    - `/usr/`: User utilities and applications.
    - `/var/`: Variable files (e.g., `/var/log/syslog`).
    - `/root/`: Root user’s home directory.
    - `/proc/`: Virtual filesystem for kernel and process status.

### ✨ Key Files for Forensics

- `/etc/os-release`: Operating system details.
- `/etc/passwd`: User information (UID, GID, home directory, shell).
- `/etc/group`: Group memberships.
- `/etc/sudoers`: Privilege separation details.
- `/var/log/syslog`: Program and service messages (excluding authentication).
- `/var/log/auth.log`: Authentication messages.
- `/var/log/error.log`: Error messages.
- `/var/log/dmesg`: OS event messages.
- `/home/<user>/.bash_history`: User command history.
- Application log files: Specific to installed programs.

### ✨ Extracting tmpfs

- **Definition**: A temporary file storage facility in RAM, supported since Linux kernel 2.4, used for directories like `/tmp`, `/var/lock`, `/var/run`, and `/var/tmp`.
- **Plugin**: `linux_tmpfs`
    
    ```bash
    $ vol.py --profile=LinuxUbuntu-5_11_0-34-genericx64 -f mem.lime linux_tmpfs -L
    
    ```
    
    - **Output**: Lists superblocks available for extraction.
    
    ```bash
    $ vol.py --profile=LinuxUbuntu-5_11_0-34-genericx64 -f mem.lime linux_tmpfs -S <superblock> -D /path/to/output
    
    ```
    
    - **Output**: Saves tmpfs data to the specified directory.
- **Forensic Value**: Captures temporary files and caches, including browser data.

### ✨ File Recovery with PhotoRec

- **Command (Windows)**:
    
    ```powershell
    PS D:\> .\testdisk-7.2-WIP\photorec_win.exe /path/to/ubuntu_11.05.58.lime
    
    ```
    
- **Process**:
    1. Confirm the input file.
    2. Select the partition.
    3. Choose the output directory (e.g., `photorec_output`).
    4. Press `Shift + C` to start recovery.
- **Output**: Saves recovered files to the specified directory, searchable by extension.
- **Forensic Value**: Recovers files not accessible via Volatility plugins.

### 💡 Notes

- **Timestamps**: `linux_recover_filesystem` retains existing file timestamps, but ext3 and earlier filesystems lack creation time metadata.
- **Forensic Value**: Recovered filesystems provide access to user files, logs, and configurations critical for investigations.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Filesystem recovery provides access to system and user data not directly available in memory dumps.
- **Pro Tip**: Analyze recovered logs (`/var/log/*`) and user files (`/home/<user>/*`) to reconstruct events.
- **Sensitive Note**: Incomplete filesystem recovery may miss critical files—use PhotoRec as a fallback for comprehensive recovery! 🚨

---

## 🛠️ Checking Browsing History

Browsers store history in SQLite databases (e.g., Firefox: `/home/user/.mozilla/firefox/<profile>/places.sqlite`; Chrome: `/home/user/.config/google-chrome/Default/History`). If these files are unavailable, process memory analysis can recover visited URLs.

### ✨ Key Plugin: linux_dump_map

- **Command**:
    
    ```bash
    $ vol.py --profile=LinuxUbuntu-5_11_0-34-genericx64 -f mem.lime linux_dump_map -p 1290191 -D /mnt/hgfs/flash/firefox
    
    ```
    
    - **Output**: Saves memory mappings for the Firefox process (PID 1290191) to separate files in `/mnt/hgfs/flash/firefox`.
    - **Note**: Unlike Windows, Linux does not allow direct export of entire process memory; mappings include executable files, libraries, stack, and heap.

### ✨ Processing Memory Mappings

1. **Using strings**:
    
    ```bash
    $ for file in /mnt/hgfs/flash/firefox/*; do strings "$file" >> /mnt/hgfs/flash/firefox_strings.txt; done
    
    ```
    
    - **Output**: Combines strings from all mappings into `firefox_strings.txt`.
    - **Analysis**: Search for URLs or search queries using `grep` with regular expressions.
2. **Using Bulk Extractor (Windows)**:
    - **Merge Files**:
        
        ```powershell
        Get-ChildItem -Path D:\firefox -File -Recurse | ForEach-Object -Process { Get-Content -Path $_.FullName | Out-File -FilePath D:\firefox-result.vma -Append }
        
        ```
        
        - **Output**: Combines mappings into `firefox-result.vma`.
    - **Run Bulk Extractor**:
        
        ```powershell
        .\bulk_extractor -o output_folder -x all -e email D:\firefox-result.vma
        
        ```
        
        - **Output**: Extracts URLs to `url_histogram.txt` in the output folder.
        - **Forensic Value**: Captures URLs, including those from privacy-focused engines like DuckDuckGo.

### 💡 Notes

- **Obsolete Plugin**: `linux_route_cache` (used for browser history pre-kernel 3.6) is ineffective in newer kernels due to disabled routing cache.
- **Forensic Value**: Process memory analysis is versatile, recovering URLs regardless of browser or history storage format.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Browsing history reveals user interests, attacker research, or compromised sites.
- **Pro Tip**: Use Bulk Extractor for faster URL extraction from large memory dumps.
- **Sensitive Note**: Browser cache clearing by attackers may limit history recovery—rely on process memory for comprehensive results! 🚨

---

## 🛠️ Investigating Communication Applications

Communication applications (e.g., Thunderbird, messengers) store valuable data like conversations and attachments, especially on attacker hosts. Analysis mirrors browser memory analysis.

### ✨ Example: Thunderbird (PID 51825)

- **Dump Memory**:
    
    ```bash
    $ vol.py --profile=LinuxUbuntu-5_11_0-34-genericx64 -f mem.lime linux_dump_map -p 51825 -D /mnt/hgfs/flash/thunderbird
    
    ```
    
    - **Output**: Saves memory mappings to `/mnt/hgfs/flash/thunderbird`.
- **Extract Strings**:
    
    ```bash
    $ for file in /mnt/hgfs/flash/thunderbird/*; do strings "$file" >> /mnt/hgfs/flash/thunderbird_strings.txt; done
    
    ```
    
    - **Output**: Creates `thunderbird_strings.txt` with extracted strings.
    - **Analysis**: Search for emails, social media notifications, attachment names, or conversation fragments using `grep` or manual review.
- **Bulk Extractor (Windows)**:
    
    ```powershell
    .\bulk_extractor -o output_folder -x all -e email D:\thunderbird-result.vma
    
    ```
    
    - **Output**: Extracts email addresses, URLs, and other data to the output folder.

### 💡 Forensic Value

- Reveals user accounts, services, and communication content (e.g., emails, chat messages).
- Identifies attacker interactions with social networks or services.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Communication data provides insights into attacker motives and victim interactions.
- **Pro Tip**: Search for keywords (e.g., email addresses, attachment names) to prioritize relevant findings.
- **Sensitive Note**: Encryption or clearing of application data may limit recovery—analyze process memory promptly after acquisition! 🚨

---

## 🛠️ Looking for Mounted Devices

Identifying mounted devices (e.g., USB drives) and their connection timelines is crucial for tracking data transfers or external media usage.

### ✨ Key Plugin: linux_mount

- **Command**:
    
    ```bash
    $ vol.py --profile=LinuxUbuntu-5_11_0-34-genericx64 -f mem.lime linux_mount
    
    ```
    
- **Output**: Lists mounted devices, including location, mount point, filesystem type, and access rights.
- **Limitation**: Lacks timestamp information.

### ✨ Kernel Debug Buffer (dmesg)

- **Plugin**: `linux_dmesg`
    
    ```bash
    $ vol.py --profile=LinuxUbuntu-5_11_0-34-genericx64 -f mem.lime linux_dmesg > dmesg_output.txt
    
    ```
    
    - **Output**: Saves kernel debug buffer to `dmesg_output.txt`, including USB device connections, serial numbers, and network activity.
- **Timestamp Analysis**:
    - Convert process start times (e.g., `mysqld` at 2021-10-05 17:05:54 UTC) to Unix timestamps using an online converter (e.g., [https://www.unixtimestamp.com](https://www.unixtimestamp.com/)).
    - **Example**: `mysqld` timestamp converts to October 5, 2021, 15:26:18 UTC, indicating a USB connection time.
- **Live Host Alternative**:
    
    ```bash
    $ dmesg -T
    
    ```
    
    - **Output**: Shows readable timestamps (e.g., USB connection on October 5, 2021, 15:25:13 UTC).
    - **Note**: Disk-based `dmesg` logs can be altered by attackers—memory analysis is more reliable.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Mounted device analysis reveals external media usage, critical for data exfiltration cases.
- **Pro Tip**: Cross-check `linux_dmesg` with live `dmesg -T` output to detect log tampering.
- **Sensitive Note**: Timestamps may have slight deviations—use multiple sources to confirm event timelines! 🚨

---

## 🛠️ Detecting Crypto Containers

Crypto containers (e.g., dm-crypt, TrueCrypt, VeraCrypt) are used to secure data, often by attackers to hide malicious tools or stolen data.

### ✨ Detection Methods

1. **Process Analysis**:
    - Use `linux_pslist` or `linux_psaux` to identify running encryption processes (e.g., `truecrypt`, `veracrypt`).
    - **Forensic Value**: Indicates active use of crypto containers.
2. **TrueCrypt Passphrase Recovery**:
    - **Plugin**: `linux_truecrypt_passphrase`
        
        ```bash
        $ vol.py --profile=LinuxUbuntu-5_11_0-34-genericx64 -f mem.lime linux_truecrypt_passphrase
        
        ```
        
        - **Output**: Recovers cached TrueCrypt passphrases.
        - **Forensic Value**: Provides access to encrypted containers.
3. **AES Key Search with Bulk Extractor**:
    - **Command (Windows)**:
        
        ```powershell
        .\bulk_extractor -o output_folder -x all -e aes D:\mem.lime
        
        ```
        
        - **Output**: Saves AES keys to `aes_keys.txt`.
        - **Forensic Value**: Identifies potential encryption keys for dm-crypt, TrueCrypt, or VeraCrypt containers.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Detecting crypto containers reveals hidden data critical to investigations.
- **Pro Tip**: Combine process analysis and key recovery to maximize decryption success.
- **Sensitive Note**: Encryption keys may be overwritten in memory—perform analysis immediately after acquisition! 🚨

---

## 📝 Summary and Next Steps

User activity reconstruction on Linux systems is vital for understanding incident context, user actions, and attacker techniques. The chapter covers techniques to analyze running processes (`linux_pslist`, `linux_psaux`, `linux_psenv`), Bash history (`linux_bash`), opened files (`linux_lsof`, `linux_find_file`), filesystem recovery (`linux_recover_filesystem`, `linux_tmpfs`), browsing history (`linux_dump_map`, Bulk Extractor), communication applications, mounted devices (`linux_mount`, `linux_dmesg`), and crypto containers (`linux_truecrypt_passphrase`, Bulk Extractor). Tools like Volatility, PhotoRec, and Bulk Extractor, combined with careful documentation and cross-checking, ensure comprehensive evidence collection. Despite Linux’s complexity compared to Windows, memory analysis provides valuable insights into user and attacker activities.

**Next Steps**: Proceed to Chapter 9 (Malicious Activity Detection) to learn how to identify malware, rootkits, and other malicious behaviors in Linux memory dumps using Volatility.