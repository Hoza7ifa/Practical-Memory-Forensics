# Ch9 - Malicious Activity Detection

---

# 🌟 Comprehensive Summary: Malicious Activity Detection (Chapter 9) from *Practical Memory Forensics* 🌟

This summary covers Chapter 9, "Malicious Activity Detection," from *Practical Memory Forensics* by Svetlana Ostrovskaya, focusing on identifying malicious activities on Linux-based systems during forensic investigations. 🕵️‍♂️ The chapter addresses the growing prevalence of Linux-targeted attacks, including ransomware (e.g., BlackMatter, Ransomexx, Hive) and post-exploitation frameworks (e.g., Metasploit), and provides techniques to analyze network connections, process anomalies, and kernel objects. Tools like Volatility, Bulk Extractor, and Wireshark are used to investigate network traffic, process memory, and rootkits. This detailed overview ensures you have all the necessary information for memory forensics without reading the chapter directly. 🧠

---

## 🧠 Introduction to Malicious Activity Detection

The primary goal of memory forensic investigations is to identify malicious activity, particularly on Linux-based systems, which are increasingly targeted due to their use in enterprise environments and by attackers for tools like Metasploit and network scanners. The chapter highlights the rise in Linux-specific malware (e.g., BlackMatter, Ransomexx, Hive) and common initial access techniques like vulnerability exploitation and security misconfigurations, especially in web applications. It focuses on three key areas: network connections, process injections, and access to atypical resources, with detailed analysis techniques and concrete examples.

### 💡 Key Topics Covered

- **Investigating Network Activity**: Analyze network interfaces, active connections, and traffic dumps.
- **Analyzing Malicious Activity**: Examine processes, their behavior, and associated resources for anomalies.
- **Examining Kernel Objects**: Detect rootkits and kernel-level manipulations (e.g., hooks).

### 💡 Forensic Applications

- **Victim Hosts**: Identify compromised systems and attacker activities (e.g., SSH connections to victim IPs).
- **Attacker Hosts**: Uncover tools (e.g., Metasploit), persistence mechanisms (e.g., cron jobs), and network communications.
- **Incident Context**: Reconstruct attacker actions, timelines, and motives through memory analysis.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Linux systems are critical targets, and detecting malicious activity provides evidence for attribution and response.
- **Pro Tip**: Document all findings (e.g., process IDs, IP addresses, timestamps) to maintain chain-of-custody.
- **Sensitive Note**: Attackers may hide activities by manipulating logs or using rootkits—cross-verify memory data with disk artifacts to detect tampering! 🚨

---

## 🛠️ Technical Requirements

The chapter assumes a Linux system (e.g., Ubuntu 21.04) running Volatility 2.6.1 and a Windows system for tools like Bulk Extractor and Wireshark. Volatility requires custom profiles (created in Chapter 7) placed in the profiles folder, specified with the `--plugins` option.

### 💡 Commands to Verify Profiles

- Check available Volatility profiles:
    
    ```bash
    $ vol.py --info | grep -i linux
    
    ```
    
    - **Output**: Lists available Linux profiles (e.g., `LinuxUbuntu_it-exec64`) for use with Volatility plugins.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Proper tool setup ensures accurate analysis and compatibility with the target system.
- **Pro Tip**: Store profiles in a centralized repository and verify compatibility with the target system’s kernel version.
- **Sensitive Note**: Using an incorrect profile can lead to invalid results—always match the profile to the system’s distribution and kernel! 🚨

---

## 🛠️ Investigating Network Activity

Malware often communicates with command-and-control (C2) servers, downloads modules, or exfiltrates data, making network analysis critical. The chapter covers techniques to examine network interfaces, active connections, and traffic dumps.

### ✨ Key Plugins and Tools

1. **linux_ifconfig**: Displays network interface configurations.
    
    ```bash
    $ vol.py --plugins=profiles -f /mnt/hgfs/flash/it_sec.lime --profile=LinuxUbuntu_it-exec64 linux_ifconfig
    
    ```
    
    - **Output**: Lists network interfaces, their IP addresses, and configurations (e.g., `eth0`, `lo`).
    - **Forensic Value**: Identifies interfaces used for malicious communications.
2. **linux_netstat**: Lists active network connections.
    
    ```bash
    $ vol.py --plugins=profiles -f /mnt/hgfs/flash/it_sec.lime --profile=LinuxUbuntu_it-exec64 linux_netstat > netstat_output.txt
    
    ```
    
    - **Output**: Shows connections, including process names, IPs, ports, and protocols (e.g., Firefox, Postgres, Ruby connections; SSH to `192.168.3.132:22`).
    - **Example Findings**:
        - Firefox connections indicate browsing activity.
        - Postgres and Ruby connections suggest Metasploit usage (common in attacker hosts).
        - SSH connection to `192.168.3.132:22` likely indicates a victim host.
    - **Forensic Value**: Identifies C2 communications, data exfiltration, or remote access (e.g., SSH).
3. **Bulk Extractor (Network Traffic)**: Extracts network packets from memory dumps.
    
    ```powershell
    .\bulk_extractor -o output_folder -x all -e net D:\it_sec.lime
    
    ```
    
    - **Output**: Generates `packets.pcap` containing network traffic.
    - **Analysis with Wireshark**:
        - Download Wireshark from [http://www.wireshark.org](http://www.wireshark.org/).
        - Drag and drop `packets.pcap` into Wireshark.
        - View endpoint statistics (e.g., IP addresses) and protocol hierarchy (e.g., TCP, SSH).
        - Apply filters (e.g., `ip.addr == 192.168.3.132 && ssh`) to confirm specific connections.
    - **Forensic Value**: Captures detailed network traffic, including IPs, protocols, and transmitted objects.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Network analysis reveals attacker communications and compromised hosts.
- **Pro Tip**: Redirect `linux_netstat` output to a file to manage extensive data and use Wireshark filters for targeted analysis.
- **Sensitive Note**: Attackers may use encrypted protocols (e.g., SSH) to hide activity—analyze packet contents and correlate with process data! 🚨

---

## 🛠️ Analyzing Malicious Activity

Examining processes and their behavior is key to detecting anomalies like suspicious process names, child processes, or persistence mechanisms (e.g., cron jobs). The chapter uses the `rules_for_emplo` process as an example of malicious activity.

### ✨ Key Plugins and Techniques

1. **linux_dump_map**: Extracts process memory for analysis.
    
    ```bash
    $ vol.py --plugins=profiles -f /mnt/hgfs/flash/it_sec.lime --profile=LinuxUbuntu_it-exec64 linux_dump_map -p <PID> -D /mnt/hgfs/flash/rules_for_emplo
    
    ```
    
    - **Output**: Saves memory mappings for the specified process (e.g., `rules_for_emplo`) to the output directory.
    - **String Extraction**:
        
        ```bash
        $ for file in /mnt/hgfs/flash/rules_for_emplo/*; do strings "$file" >> /mnt/hgfs/flash/rules_strings.txt; done
        
        ```
        
        - **Output**: Creates `rules_strings.txt` with readable strings (e.g., `192.168.168.144`, `tcp/192.168.168.153:14444`, indicating a `reverse_tcp` connection).
        - **Forensic Value**: Reveals IP addresses, ports, and connection types (e.g., reverse TCP shells used by malware).
2. **linux_pstree**: Displays process parent-child relationships.
    
    ```bash
    $ vol.py --plugins=profiles -f /mnt/hgfs/flash/it_sec.lime --profile=LinuxUbuntu_it-exec64 linux_pstree
    
    ```
    
    - **Output**: Shows `rules_for_emplo` spawning shells (including elevated ones), Python, and `systemctl`.
    - **Forensic Value**: Identifies suspicious child processes, indicating malicious activity (e.g., privilege escalation).
3. **linux_psaux**: Lists process start arguments.
    
    ```bash
    $ vol.py --plugins=profiles -f /mnt/hgfs/flash/it_sec.lime --profile=LinuxUbuntu_it-exec64 linux_psaux
    
    ```
    
    - **Output**: Shows how child processes were started (e.g., Python spawning a TTY shell with `sudo`).
    - **Forensic Value**: Reveals execution details, such as privilege escalation attempts.
4. **linux_bash**: Recovers executed commands.
    
    ```bash
    $ vol.py --plugins=profiles -f /mnt/hgfs/flash/it_sec.lime --profile=LinuxUbuntu_it-exec64 linux_bash
    
    ```
    
    - **Output**: Lists commands, e.g.:
        - Installation of a cron job for persistence.
        - `systemctl` used to reload the cron service and check its status.
        - Use of `/tmp/cron` directory for temporary files.
    - **Forensic Value**: Reveals attacker persistence mechanisms and executed commands.
    - **Log Analysis**: Check `/var/log/cron.log` for details on created cron jobs.
5. **linux_lsof**: Lists open file descriptors for a process.
    
    ```bash
    $ vol.py --plugins=profiles -f /mnt/hgfs/flash/it_sec.lime --profile=LinuxUbuntu_it-exec64 linux_lsof -p <PID> > lsof_output.txt
    
    ```
    
    - **Output**: Shows resources used by `rules_for_emplo` and its child processes, including:
        - `/dev/null`: An empty device (writing succeeds, reading returns EOF).
        - `/dev/ptmx`: Pseudo-terminal master for TTY shells.
        - `/dev/pts/*`: Pseudo-terminal slave entries created dynamically by the kernel.
    - **Forensic Value**: Identifies files, sockets, and pipes used by malicious processes.

### 💡 Additional Analysis

- **Executable Analysis**:
    - Calculate the hash of the `rules_for_emplo` executable and check it on cyber threat intelligence platforms (e.g., VirusTotal).
    - Run the executable in a controlled environment (e.g., sandbox) or perform reverse engineering if skilled.
- **Forensic Value**: Identifies malware, its behavior, and potential affiliations with known threats.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Process analysis uncovers malicious behaviors, persistence mechanisms, and attacker tools.
- **Pro Tip**: Combine `linux_pstree`, `linux_psaux`, and `linux_bash` to reconstruct the full attack chain.
- **Sensitive Note**: Attackers may use obfuscated process names (e.g., `rules_for_emplo`) to blend in—check arguments and child processes for anomalies! 🚨

---

## 🛠️ Examining Kernel Objects

Rootkits, though less common today, historically allowed attackers to gain persistence and hide activities by manipulating kernel objects. The chapter discusses techniques to detect loaded and hidden kernel modules, as well as hooking mechanisms like IDT and syscall hooks.

### ✨ Key Plugins and Techniques

1. **linux_lsmod**: Enumerates loaded kernel modules.
    
    ```bash
    $ vol.py --plugins=profiles -f /mnt/hgfs/flash/it_sec.lime --profile=LinuxUbuntu_it-exec64 linux_lsmod
    
    ```
    
    - **Output**: Lists kernel modules by walking the global `modules` variable, including module names and base addresses.
    - **Forensic Value**: Identifies legitimate and suspicious kernel modules.
2. **linux_hidden_modules**: Detects hidden kernel modules.
    
    ```bash
    $ vol.py --plugins=profiles -f /mnt/hgfs/flash/it_sec.lime --profile=LinuxUbuntu_it-exec64 linux_hidden_modules
    
    ```
    
    - **Output**: Lists hidden modules (e.g., `RGQ4325424R24`) by carving memory and comparing with `linux_lsmod` output.
    - **Extraction**:
        
        ```bash
        $ vol.py --plugins=profiles -f /mnt/hgfs/flash/it_sec.lime --profile=LinuxUbuntu_it-exec64 linux_moddump -b 0xffffffff00521970 -D /mnt/hgfs/flash/
        
        ```
        
        - **Output**: Saves the module (e.g., `RGQ4325424R24`) to the specified directory for analysis.
    - **Forensic Value**: Detects rootkits that hide from the kernel module list.
3. **Hooking Detection**:
    - **Definition**: Hooking modifies OS or application behavior by intercepting function calls, messages, or events.
    - **Common Techniques**:
        - **Interrupt Descriptor Table (IDT)**: Stores pointers to interrupt service routines (e.g., for keyboard/mouse events).
        - **System Calls (Syscalls)**: Application requests to the kernel for operations, defined in the Linux syscall table.
    - **Plugins**:
        - **linux_check_idt**: Checks for IDT hooks.
            
            ```bash
            $ vol.py --plugins=profiles -f /mnt/hgfs/flash/it_sec.lime --profile=LinuxUbuntu_it-exec64 linux_check_idt
            
            ```
            
            - **Output**: Lists IDT entries; “HOOKED” indicates a hook (none found in the example).
        - **linux_check_syscall**: Checks for syscall hooks.
            
            ```bash
            $ vol.py --plugins=profiles -f /mnt/hgfs/flash/it_sec.lime --profile=LinuxUbuntu_it-exec64 linux_check_syscall
            
            ```
            
            - **Output**: Lists syscall hooks (multiple found, requiring manual analysis).
        - **linux_apihooks**: Checks for userland API hooks.
        - **linux_check_evt_arm**: Checks the exception vector table for syscall table hooks.
        - **linux_check_inline_kernel**: Checks for inline kernel hooks.
        - **linux_check_tty**: Checks TTY devices for hooks.
4. **File Interaction Detection**:
    - **linux_kernel_opened_files**: Lists files opened by the kernel.
        
        ```bash
        $ vol.py --plugins=profiles -f /mnt/hgfs/flash/it_sec.lime --profile=LinuxUbuntu_it-exec64 linux_kernel_opened_files
        
        ```
        
        - **Forensic Value**: Identifies kernel-level file interactions by rootkits.
    - **linux_check_fop**: Checks file operation structures for rootkit modifications.
        
        ```bash
        $ vol.py --plugins=profiles -f /mnt/hgfs/flash/it_sec.lime --profile=LinuxUbuntu_it-exec64 linux_check_fop
        
        ```
        
        - **Forensic Value**: Detects rootkit modifications to file operations.

### 💡 Notes

- **Rootkit Decline**: Rootkits are nearly obsolete, replaced by post-exploitation frameworks and dedicated malware.
- **Masquerading**: Rootkits may mimic legitimate modules—extract all modules with `linux_moddump` and compare with known legitimate versions.
- **Forensic Value**: Kernel object analysis detects advanced persistence and hiding techniques.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Rootkit detection ensures no hidden malicious activity is missed.
- **Pro Tip**: Use `linux_hidden_modules` and `linux_moddump` together to extract and analyze suspicious modules.
- **Sensitive Note**: Syscall hooks require manual analysis due to limited plugin output—correlate with process and network data for context! 🚨

---

## 📝 Summary and Next Steps

Detecting malicious activity on Linux systems involves analyzing network connections (`linux_ifconfig`, `linux_netstat`, Bulk Extractor, Wireshark), process behaviors (`linux_dump_map`, `linux_pstree`, `linux_psaux`, `linux_bash`, `linux_lsof`), and kernel objects (`linux_lsmod`, `linux_hidden_modules`, hooking plugins). The chapter uses the `rules_for_emplo` process to demonstrate a malicious attack involving reverse TCP connections, privilege escalation, and cron job persistence. While rootkits are less common, plugins like `linux_check_idt` and `linux_check_syscall` help detect kernel manipulations. Knowledge of the filesystem (e.g., `/var/log/cron.log`) and software configurations is crucial for filling investigation gaps. Tools like Volatility, Bulk Extractor, and Wireshark, combined with careful documentation, ensure comprehensive evidence collection.

**Next Steps**: Proceed to the next chapter on macOS memory forensics to learn about acquiring and analyzing memory dumps from macOS systems.