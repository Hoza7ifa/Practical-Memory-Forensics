# Ch7 - Linux Memory Acquisition

---

# 🌟 Comprehensive Summary: Linux Memory Acquisition (Chapter 7) from *Practical Memory Forensics* 🌟

This summary covers Chapter 7, "Linux Memory Acquisition," from *Practical Memory Forensics* by Svetlana Ostrovskaya, detailing the process of capturing volatile memory (RAM) from Linux-based systems for forensic analysis. 🕵️‍♂️ The chapter emphasizes the growing importance of Linux forensics due to increasing attacks on Linux servers and devices, driven by their widespread use in enterprise environments and the rise of Linux-targeting malware (e.g., Fancy Bear’s Droverub rootkit and ransomware variants). It provides a thorough guide to memory acquisition challenges, preparation steps, tools like LiME and AVML, and creating Volatility profiles, with practical commands and forensic applications. This detailed overview ensures you have all the necessary information without reading the chapter directly. 🧠

---

## 🧠 Introduction to Linux Memory Acquisition

Linux-based systems are critical in enterprise environments (e.g., web, mail, database, and file servers) and diverse hardware (PCs, tablets, smartphones), making them prime targets for attackers, including state-sponsored groups like Fancy Bear and ransomware operators. Memory acquisition is a vital first step in Linux memory forensics, capturing volatile RAM data to analyze user actions, detect malware, and reconstruct system activity. The chapter focuses on Linux-specific challenges, tools, and preparation, building on general acquisition issues discussed in Chapter 2.

### 💡 Key Points

- **Purpose**: Capture volatile memory to preserve evidence like running processes, network connections, and malicious code, which may not persist on disk.
- **Importance**: Linux’s role in enterprise and IoT systems, combined with increasing Linux-targeting malware (e.g., Droverub rootkit, ransomware), necessitates robust memory forensics.
- **Forensic Focus**: The chapter provides practical guidance on acquiring memory dumps using tools like LiME and AVML, preparing environments, and creating Volatility profiles for analysis.
- **Applications**:
    - **Malware Detection**: Identify rootkits, injected code, or ransomware.
    - **User Activity Reconstruction**: Recover command histories or network activity.
    - **Enterprise Investigations**: Analyze server compromises in web, database, or file servers.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Memory acquisition is critical for capturing ephemeral data in Linux systems, especially in enterprise settings where downtime or data loss is costly.
- **Pro Tip**: Always prioritize memory acquisition before disk imaging due to RAM’s volatility, following the order of volatility principle.
- **Sensitive Note**: Malware may detect or interfere with acquisition tools—use trusted, tested tools to avoid evidence tampering! 🚨

---

## 📊 Linux Memory Acquisition Issues

Linux memory acquisition faces unique challenges due to the diversity of Linux distributions and kernel configurations, unlike Windows systems. The chapter outlines these issues, building on general acquisition challenges (e.g., non-atomic operations, system instability) from Chapter 2.

### ✨ Specific Challenges

1. **Diversity of Distributions**:
    - The Linux kernel’s open-source nature (GNU General Public License) results in numerous distributions (e.g., Ubuntu, CentOS, Debian), each with unique features affecting memory acquisition.
    - **Impact**: Tools must be compatible with specific distributions and kernel versions, complicating the process.
    - **Forensic Consideration**: Identify the target system’s distribution and kernel version to ensure tool compatibility.
2. **Legacy Memory Access (Pre-Linux 2.6)**:
    - Older kernels (before 2.6) allowed memory access via `/dev/mem` (physical memory) and `/dev/kmem` (kernel memory) using tools like `cat` or `dd` to redirect output to a file.
    - **Issues**:
        - Non-sequential memory mapping from physical offset 0 could lead to system instability, memory corruption, or crashes if sensitive regions were accessed.
        - Security risks due to direct access by programs with root privileges.
    - **Forensic Value**: Understanding legacy methods helps interpret older systems or misconfigured setups still using `/dev/mem`.
3. **Modern Kernel Restrictions**:
    - Newer kernels disable `/dev/mem` and `/dev/kmem` for security, requiring a loadable kernel module to access physical memory.
    - **Challenge**: The kernel module must be built on a system matching the target’s distribution and kernel version to function correctly.
    - **Impact**: Building modules on the target system risks overwriting evidence due to dependency installations.
    - **Forensic Consideration**: Build modules in a controlled test environment to avoid altering the target system.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Distribution diversity and kernel restrictions demand careful tool selection and preparation to ensure successful acquisition.
- **Pro Tip**: Always verify tool compatibility with the target system’s kernel version to avoid acquisition failures.
- **Sensitive Note**: Avoid running untested kernel modules on live systems, as they can cause crashes or evidence loss—test in a virtual environment first! 🚨

---

## 🛠️ Preparing for Linux Memory Acquisition

Proper preparation is critical to ensure compatibility and minimize evidence alteration. The chapter details steps to set up a virtual machine (VM) and prepare storage for memory dumps.

### ✨ Preparation Steps

1. **Identify Target System Configuration**:
    - **Distribution**: Run the following command on the target host to identify the distribution:
        
        ```bash
        $ cat /etc/*release
        
        ```
        
        - **Example Output** (for Ubuntu 21.04):
            
            ```
            DISTRIB_ID=Ubuntu
            DISTRIB_RELEASE=21.04
            DISTRIB_CODENAME=hirsute
            DISTRIB_DESCRIPTION="Ubuntu 21.04"
            DISTRIB_BUILDTIME="2021-04-21 08:44"
            VERSION="2.6.32-24 (hirsute Hippo)"
            ID=ubuntu
            ID_GA=hirsute
            PRETTY_NAME="Ubuntu 21.04"
            NAME="Ubuntu"
            HOME_URL="https://www.ubuntu.com/"
            SUPPORT_URL="https://www.ubuntu.com/"
            BUG_REPORT_URL="https://bugs.ubuntu.com/"
            PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
            VERSION_CODENAME=hirsute
            DISTRIB_CODENAME=hirsute
            
            ```
            
    - **Kernel Version**: Run the following command to get the exact kernel version:
        
        ```bash
        $ uname -r
        
        ```
        
        - **Example Output**: `5.11.0-34-generic`
    - **Forensic Value**: Ensures the VM matches the target system for building compatible kernel modules.
2. **Set Up a Virtual Machine**:
    - Create a VM (using VMware, VirtualBox, etc.) with the same distribution and kernel version as the target (e.g., Ubuntu 21.04, kernel 5.11.0-34-generic).
    - **Alternative**: If a VM with the correct distribution exists, downgrade the kernel to match the target version.
    - **Forensic Consideration**: Avoid building modules on the target system to prevent overwriting evidence.
3. **Prepare Storage**:
    - **Removable Media**: Use a USB drive or external storage to save memory dumps, as discussed in Chapter 3 (Windows Memory Acquisition).
    - **Network Share**: Set up a network share accessible by the target host for remote dump transfer.
    - **Forensic Value**: Ensures dumps are stored securely without altering the target system’s disk.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Matching the VM to the target system ensures kernel module compatibility and preserves evidence integrity.
- **Pro Tip**: Save the output of `/etc/*release` and `uname -r` in your case documentation for reference during analysis.
- **Sensitive Note**: Installing dependencies on the target system can overwrite critical data—always use a separate VM for module building! 🚨

---

## 🛠️ Acquiring Memory with LiME

The Linux Memory Extractor (LiME) is a loadable kernel module designed for memory acquisition on Linux and Linux-based systems (e.g., Android). It is widely used due to its minimal footprint and ability to calculate dump hashes for integrity verification.

### ✨ LiME Overview

- **Source**: Available at [https://github.com/504ensicsLab/LiME](https://github.com/504ensicsLab/LiME).
- **Features**:
    - Minimal process footprint, reducing system impact.
    - Supports local storage or network transfer of dumps.
    - Outputs dumps in LiME format, compatible with Volatility.
    - Calculates hashes for dump verification.
- **Forensic Value**: Ideal for forensic investigations requiring reliable, verifiable memory dumps.

### ✨ Building the LiME Kernel Module

1. **Set Up VM**: Use a VM matching the target’s distribution and kernel (e.g., Ubuntu 21.04, 5.11.0-34-generic).
2. **Install Dependencies**: Run the following command to install required packages:
    
    ```bash
    $ sudo apt-get install -y linux-headers-$(uname -r) build-essential make gcc linux-forensics-dkms
    
    ```
    
    - **Output Example**: Installs kernel headers, build tools, and DKMS for module compilation.
3. **Clone LiME Repository**:
    
    ```bash
    $ git clone https://github.com/504ensicsLab/LiME.git
    $ cd LiME/src
    $ make
    
    ```
    
    - **Result**: Produces a kernel module (e.g., `lime-5.11.0-34-generic.ko`) in `/usr/src/linux-forensics-1.9.1-2`.
4. **Transfer Module**: Copy the module to removable media or a network share using `scp`:
    
    ```bash
    $ scp lime-5.11.0-34-generic.ko user@target:/path/to/module
    
    ```
    

### ✨ Acquiring the Memory Dump

1. **Load the Module**: On the target host, load the LiME module and specify output parameters:
    
    ```bash
    $ sudo insmod ./lime-5.11.0-34-generic.ko path=tcp:4444 format=lime
    
    ```
    
    - **Parameters**:
        - `path=tcp:4444`: Sends the dump over TCP to port 4444.
        - `format=lime`: Outputs in LiME format for Volatility compatibility.
    - **Alternative**: Use `path=/path/to/mem.lime` for local storage.
2. **Capture Dump (Network)**: On the investigator’s host, use `netcat` to receive the dump:
    
    ```bash
    $ nc 192.168.3.132 4444 > mem.lime
    
    ```
    
    - **Output**: Saves the dump as `mem.lime`.
3. **Unload Module**: Remove the LiME module to minimize system impact:
    
    ```bash
    $ sudo rmmod lime
    
    ```
    

### 💡 Investigator’s Toolkit

- **Why It Matters**: LiME’s minimal footprint and network transfer capability make it versatile for enterprise and remote investigations.
- **Pro Tip**: Always verify the dump’s hash (generated by LiME) to ensure integrity before analysis.
- **Sensitive Note**: Loading kernel modules can destabilize the system—test LiME on a VM first to confirm stability! 🚨

---

## 🛠️ Acquiring Memory with AVML

Acquire Volatile Memory for Linux (AVML), developed by Microsoft, is a userland tool that does not require building a kernel module, making it simpler to deploy.

### ✨ AVML Overview

- **Source**: Available at [https://github.com/microsoft/avml](https://github.com/microsoft/avml), under the Releases tab.
- **Features**:
    - Accesses memory via `/dev/crash`, `/proc/kcore`, or `/dev/mem` (tries each source until a valid one is found).
    - No kernel module compilation required.
    - Outputs dumps in LiME format, compatible with Volatility.
- **Tested Distributions**:
    - Ubuntu: 12.04, 14.04, 16.04, 18.04, 18.10, 19.04, 19.10
    - CentOS: 6.5–6.10, 7.0–7.6
    - RHEL: 6.7–6.9, 7.0, 7.2–7.5, 8
    - Debian: 8.9
    - Oracle Linux: 6.8–6.9, 7.3–7.6
- **Limitation**: Limited distribution support; test on a VM before use.
- **Forensic Value**: Simplifies acquisition for supported distributions, ideal for quick deployments.

### ✨ Using AVML

1. **Download AVML**: Obtain the binary from the GitHub Releases tab.
2. **Transfer to Target**: Copy to removable media or a network share:
    
    ```bash
    $ scp avml user@target:/mnt/ufs/linux/
    
    ```
    
3. **Set Permissions**: Make the binary executable:
    
    ```bash
    $ sudo chmod 755 /mnt/ufs/linux/avml
    
    ```
    
4. **Capture Dump**: Run AVML and specify the output file:
    
    ```bash
    $ cd /mnt/ufs/linux/
    $ ./avml mem.lime
    
    ```
    
    - **Output**: Produces `mem.lime` in LiME format.
    - **Note**: AVML automatically selects a valid memory source (`/dev/crash`, `/proc/kcore`, `/dev/mem`).

### 💡 Investigator’s Toolkit

- **Why It Matters**: AVML’s userland approach reduces preparation time, ideal for urgent investigations.
- **Pro Tip**: Verify AVML compatibility with the target distribution in a test VM to avoid runtime errors.
- **Sensitive Note**: AVML may fail on untested distributions—always have LiME as a backup tool! 🚨

---

## 🛠️ Creating a Volatility Profile

Volatility requires a custom profile for each Linux distribution and kernel version to analyze memory dumps. The chapter details the process of creating these profiles.

### ✨ Steps to Create a Volatility Profile

1. **Install Dependencies**: On the VM matching the target system, install required packages:
    
    ```bash
    $ sudo apt-get install -y zip dwarfdump
    
    ```
    
    - **Output Example**: Installs `zip` (3.0-1+bullid1) and `dwarfdump` (20180225-1).
2. **Clone Volatility Repository**:
    
    ```bash
    $ sudo apt-get install git
    $ git clone https://github.com/volatilityfoundation/volatility.git
    
    ```
    
3. **Build the Dwarf Module**: Navigate to the Linux tools directory and compile:
    
    ```bash
    $ cd volatility/tools/linux
    $ make
    
    ```
    
    - **Result**: Produces `module.dwarf` in the current directory.
    - **Note**: Compilation may fail due to dependency or license issues—search online for specific error solutions.
4. **Create the Profile Archive**: Combine the dwarf module with the system map:
    
    ```bash
    $ sudo zip -9 $(lsb_release -i -s)-$(uname -r).zip ./module.dwarf /boot/System.map-$(uname -r)
    
    ```
    
    - **Explanation**:
        - `lsb_release -i -s`: Outputs the distribution name (e.g., `Ubuntu`).
        - `uname -r`: Outputs the kernel version (e.g., `5.11.0-34-generic`).
        - **Result**: Creates a profile archive (e.g., `Ubuntu-5.11.0-34-generic.zip`).
5. **Use the Profile**: Place the profile in Volatility’s profiles folder and specify it during analysis:
    
    ```bash
    $ mv Ubuntu-5.11.0-34-generic.zip /path/to/volatility/plugins/profiles/
    $ volatility --profile=LinuxUbuntu-5_11_0-34-genericx64 -f mem.lime <plugin>
    
    ```
    
    - **Example Profile Name**: `LinuxUbuntu-5_11_0-34-genericx64`.

### ✨ Alternative Profile Sources

- **Official Repository**: Check [https://github.com/volatilityfoundation/profiles/tree/master/linux](https://github.com/volatilityfoundation/profiles/tree/master/linux) for prebuilt profiles for common distributions.
- **Forensic Value**: Saves time for popular distributions but may not cover all kernel versions.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Custom Volatility profiles ensure accurate analysis of Linux memory dumps.
- **Pro Tip**: Store profiles in a centralized repository for reuse across investigations with similar systems.
- **Sensitive Note**: Mismatched profiles can lead to incorrect analysis results—always verify the distribution and kernel version! 🚨

---

## 🛠️ Automation with Linux Memory Capturer

The Linux Memory Capturer (a fork of Linux Memory Grabber by Hal Pomeranz) automates memory dump creation and Volatility profile generation.

### ✨ Overview

- **Source**: [https://github.com/cpu4/linux](https://github.com/cpu4/linux).
- **Features**:
    - Uses LiME to create memory dumps.
    - Generates Volatility profiles and saves a copy of `/bin/bash`.
    - Outputs a folder with:
        - `<hostname>-YYYY-MM-DD_hh_mm_ss-memory.lime`: Memory dump in LiME format.
        - `<hostname>-YYYY-MM-DD_hh_mm_ss-profile.zip`: Volatility profile.
        - `<hostname>-YYYY-MM-DD_hh_mm_ss-bash`: Copy of `/bin/bash`.
        - `volatilityrc`: Prototype Volatility configuration file.
    - Kernel module stored in `/usr/src/lime-forensics`.
- **Limitation**: Requires Python 2.7, limiting compatibility with modern systems.
- **Forensic Value**: Streamlines acquisition and profile creation for supported systems.

### ✨ Usage

1. **Install and Run**: Install the tool on a VM and execute it to generate the dump and profile.
2. **Test First**: Run in a VM matching the target system to avoid `make` errors or compatibility issues.
3. **Output**: Transfer the generated dump and profile to the analysis workstation.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Automation saves time in large-scale or repetitive investigations.
- **Pro Tip**: Verify the tool’s output in a test environment, as Python 2.7 dependencies may cause issues on newer systems.
- **Sensitive Note**: Automation tools may fail on unsupported distributions—always test before deployment! 🚨

---

## 📈 Virtual Machine Snapshots

For Linux systems running in virtualized environments, memory acquisition can be simplified by leveraging hypervisor capabilities.

### ✨ Process

- **Create a Snapshot**: Use the hypervisor (e.g., VMware, VirtualBox) to create a snapshot, saving the VM’s memory state in a `.vmsn` file.
- **Analyze Snapshot**: Use the `.vmsn` file directly for analysis, though a Volatility profile is still required.
- **Forensic Value**: Bypasses kernel module complexities, ideal for enterprise environments with virtualized servers.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Snapshots provide a non-invasive way to capture memory in virtualized environments.
- **Pro Tip**: Document the snapshot creation time to correlate with system events during analysis.
- **Sensitive Note**: Ensure the hypervisor supports snapshot creation without suspending the VM to avoid downtime! 🚨

---

## ⚠️ Practical Considerations and Best Practices

The chapter emphasizes the complexity of Linux memory acquisition and provides best practices to ensure forensic soundness.

### ✨ Key Considerations

1. **Tool Selection**:
    - **LiME**: Preferred for its minimal footprint and network transfer capability, but requires kernel module compilation.
    - **AVML**: Simpler for supported distributions, no module needed, but limited compatibility.
    - **Linux Memory Capturer**: Automates acquisition and profile creation but limited by Python 2.7.
2. **Testing**: Always test tools in a VM matching the target system to avoid crashes or compatibility issues.
3. **Storage**: Use removable media or network shares to avoid altering the target system’s disk.
4. **Profile Creation**: Custom Volatility profiles are mandatory for Linux analysis, as prebuilt profiles are not always available.
5. **Documentation**: Record all commands, tool versions, and system details (e.g., distribution, kernel version) for chain-of-custody.

### 💡 Investigator’s Toolkit

- **Why It Matters**: Proper tool selection and testing ensure reliable evidence collection.
- **Pro Tip**: Maintain a library of prebuilt kernel modules and Volatility profiles for common distributions to speed up future investigations.
- **Sensitive Note**: Failure to test tools can lead to system crashes or invalid dumps, jeopardizing legal admissibility—always validate in a controlled environment! 🚨

---

## 📝 Summary and Next Steps

Linux memory acquisition is a complex but critical process for forensic investigations, driven by the increasing prevalence of Linux-targeting malware and the platform’s role in enterprise systems. The chapter covers challenges like distribution diversity and kernel restrictions, preparation steps (e.g., VM setup, storage), and tools like LiME (kernel module-based), AVML (userland), and Linux Memory Capturer (automation). Creating custom Volatility profiles is essential for analysis, and VM snapshots offer a simpler alternative for virtualized systems. By following best practices—testing tools, verifying compatibility, and documenting processes—investigators can ensure reliable memory dumps for malware detection, user activity reconstruction, and enterprise investigations.

**Next Steps**: Proceed to Chapter 8 (User Activity Reconstruction) and Chapter 9 (Malicious Activity Detection) to learn how to analyze Linux memory dumps using Volatility, focusing on extracting user actions and detecting malicious behavior.