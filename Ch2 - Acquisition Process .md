# Ch2 - Acquisition Process

---

# Summary of Memory Forensics Chapter 2

This chapter from *Practical Memory Forensics* by Svetlana Ostrovskaya provides a comprehensive overview of memory acquisition and analysis, covering memory management concepts, live memory analysis, partial versus full memory acquisition, and popular tools and techniques for memory forensics. Below is a detailed summary of the key information, concepts, tools, and notes relevant for a digital forensics investigator specializing in memory forensics.

## Key Concepts in Memory Management

### Address Space

- **Definition**: A set of addresses used to access memory, providing an abstraction to isolate processes from physical memory and the operating system.
- **Purpose**: Enhances security and isolation by preventing processes from directly accessing physical memory, which could crash the OS or interfere with other processes.
- **Isolation**: Each process operates within its own address space, ensuring safe and simultaneous program execution.

### Virtual Memory

- **Definition**: An abstraction separating logical memory (used by processes) from physical memory.
- **Architecture Details**:
    - **x86 Systems**: Each process is allocated 4 GB of virtual memory by default, split into 2 GB for user space (0x00000000 to 0x7FFFFFFF) and 2 GB for kernel space. Windows allows a 3:1 split (3 GB user, 1 GB kernel).
    - **x64 Systems**: User space ranges from 0x0000000000000000 to 0x00007FFFFFFFFFFF, with kernel space starting at 0xFFFF080000000000.
- **Purpose**: Allows processes to operate as if they have exclusive memory access, improving security and efficiency.

### Paging

- **Definition**: Divides the process address space into fixed-size blocks called pages, mapped to physical memory by the memory manager.
- **Functionality**:
    - The memory manager translates virtual addresses to physical addresses using hardware support.
    - Only necessary pages are loaded into RAM, with others stored on disk (in a pagefile or swap partition).
- **Page Replacement Algorithms**: Examples include FIFO, LRU, Clock, and WSClock, designed to optimize stability and performance.
- **Implication for Forensics**: Memory dumps capture only pages in RAM. To obtain a complete picture, combine memory dump analysis with non-memory-resident data (e.g., pagefile or swap).

### Shared Memory

- **Definition**: A memory region accessible to multiple processes simultaneously.
- **Uses**:
    - Facilitates data exchange or shared code execution among processes.
    - Enhances efficiency by mapping multiple processes to a single instance of a dynamic library in physical memory.
- **Forensic Relevance**: Shared memory regions may contain critical data shared across processes, such as library code or inter-process communication.

### Stack and Heap

- **Stack**:
    - Stores static data related to executable code, including stack frames for function calls (parameters, variables, return addresses).
    - Data exists only during function execution, useful for tracing executed functions.
- **Heap**:
    - Stores dynamically allocated data (e.g., text in editors, clipboard content, or chat messages in messengers).
    - Persists for the process's lifetime, making it critical for forensic analysis (e.g., recovering passwords or user data).

## Live Memory Analysis

### Overview

- **Definition**: Manual examination of running processes, memory contents, network connections, and system state without creating a full memory dump.
- **Use Cases**:
    - When memory dumps are impractical (e.g., remote systems or systems with >32 GB RAM).
    - For quick analysis during incident response.
- **Disadvantages**:
    - Cannot access terminated processes or closed network connections.
    - Limited interaction with kernel objects.
    - System interaction may overwrite forensic evidence.
    - Memory contents change constantly, risking oversight of critical data.

### Tools for Live Memory Analysis

- **Windows**:
    - **Process Hacker**:
        - Provides a list of running processes, services, network connections, and disk usage.
        - Allows inspection of process memory (stack, heap, address space) and searching with regular expressions.
        - Useful for identifying rogue processes (e.g., malware-injected explorer.exe with unusual network activity).
    - **Built-in Tools**:
        - Windows Command Shell, PowerShell, and Windows Management Instrumentation (WMI).
        - Example Command: `wmic process list full` (displays active processes, command lines, and executable paths).
        - Example Output:
            
            ```
            CommandLine=powershell.exe -nop -w hidden -enc SQBmACg<edited>
            CSName=DESKTOP-1J4LKT5
            Description=powershell.exe
            
            ```
            
- **Linux and macOS**:
    - Apple Terminal and Linux Terminal provide similar functionality to view processes, network connections, and resource usage.
    - Example: Linux command to list active processes (not specified but implied, e.g., `ps aux`).

### Important Note

- **Privilege Risks**: Live analysis often requires administrator rights. If a threat actor has access, logging in as a privileged user risks credential theft via carving tools.

## Partial vs. Full Memory Acquisition

### Partial Memory Acquisition

- **Definition**: Captures memory of specific processes rather than the entire RAM.
- **Tools**:
    - **ProcDump (Windows)**: Part of Sysinternals Suite, creates dumps of specific processes (e.g., Telegram messenger).
    - **Linux**: Analogous tools create core dumps for applications.
    - **macOS**: GDB (GNU Debugger) can create process dumps but requires manual specification of memory addresses, making it complex.
- **Use Case**: Ideal for quick extraction of specific data (e.g., IP addresses, executable code) during incident response.
- **Analysis**: Process dumps can be analyzed using debuggers like WinDbg (e.g., analyzing a Telegram process dump).

### Full Memory Acquisition

- **Purpose**: Captures the entire RAM for comprehensive analysis, including user data, encryption keys, or RAM-based timelines.
- **Advantages**: Provides a complete snapshot for in-depth investigations, unlike partial dumps, which limit analysis scope.

## Popular Acquisition Tools and Techniques

### Virtual vs. Physical Environments

- **Virtual Machines**:
    - Memory is stored in files with specific extensions, simplifying acquisition:
        - **VMware**: `.vmem` (raw memory), `.vmss` (suspended state), `.vmsn` (snapshot).
        - **Microsoft Hyper-V**: `.vsv` (metadata), `.bin` (memory chunks).
        - **VirtualBox**: `.sav` (partial memory image).
    - **Methods**:
        - **Suspend**: Saves memory to disk but may close network connections (e.g., VMware), risking data loss.
        - **Snapshot**: Captures the current state without changes, preferred for preserving original data.
- **Physical (Bare Metal) Systems**:
    - Require additional tools for memory dumping.
    - Physical access is critical for tool deployment and data storage.

### Remote vs. Local Acquisition

- **Remote Acquisition Plan**:
    1. Create a temporary user with administrator privileges to avoid credential theft.
    2. Create a network share (e.g., `$C` or `$ADMIN`) and copy the dumping tool.
    3. Use remote-control tools, service creation, or task scheduling to run the tool and send the dump to the network share.
    4. Delete the temporary account.
    - **Important Note**: Calculate checksums (e.g., MD5, SHA) before and after network transfer to verify dump integrity.
- **Local Acquisition**:
    - **Storage**: Avoid saving dumps on the target system to prevent overwriting forensic data. Use pre-prepared removable devices.
    - **Risks**: Avoid using the same device across multiple infected hosts or connecting directly to the investigator’s computer due to malware risks (e.g., USBferry, Crimson.USBWorm, USBCulprit).
    - **Solution**: Use an intermediate host to transfer data to the investigator’s system over the network.

### Hardware vs. Software Solutions

- **Hardware Solutions**:
    - **Example**: Direct Memory Access (DMA) via FireWire.
    - **Limitations**:
        - Disabled on locked systems (Windows 10, macOS 10.7.2+).
        - Requires additional drivers, increasing complexity.
- **Software Solutions**:
    - Numerous free and commercial tools map physical addresses to a process’s virtual address space and write data to a file.
    - **Device Memory**: Reserved for firmware, accessing it can cause unpredictable results. Modern tools skip these regions.

### Choosing a Tool

- **Factors to Consider**:
    - Supported OS and hardware architecture.
    - Remote dumping capability.
    - Impact on the target system (minimize changes).
    - Reliability (test tools in similar conditions before use).
- **Recommendation**: Avoid untested tools on target systems to prevent unpredictable behavior.

### Timing of Memory Dumps

- **Victim’s Device**: Create dumps when the attacker is not visibly active to avoid tampering.
- **Suspect’s Device**: Capture during active use to find evidence of illegal activity.
- **General Advice**: Avoid dumping during startup, shutdown, reboot, system updates, or high-activity periods to ensure data stability.

## Summary of Key Points

- A solid understanding of memory management (address space, virtual memory, paging, shared memory, stack, and heap) is essential for effective memory forensics.
- Live memory analysis is useful for quick assessments but risks altering evidence and missing terminated processes or connections.
- Partial memory acquisition (e.g., ProcDump) is suited for targeted data extraction, while full dumps are necessary for comprehensive investigations.
- Virtual machine memory acquisition is simpler (via snapshots or suspend), but physical systems require careful tool selection and storage planning.
- Remote and local acquisition strategies differ, with precautions to prevent data tampering or malware spread.
- Tool selection should prioritize compatibility, minimal system impact, and reliability.

## Notes for Digital Forensics Investigators

- **Critical Tools**:
    - **Process Hacker (Windows)**: Versatile for live analysis, offering process, network, and memory inspection.
    - **ProcDump (Windows)**: Efficient for partial process dumps, part of Sysinternals Suite.
    - **WinDbg (Windows)**: Debugger for analyzing process dumps.
    - **GDB (macOS/Linux)**: For process dumps, though complex on macOS.
    - **WMI (Windows)**: Use commands like `wmic process list full` for process details.
- **Paths and Commands**:
    - Example WMI command: `C:\WINDOWS\system32> wmic process list full`.
    - Network shares for remote dumping: `$C`, `$ADMIN`.
- **Important Notes**:
    - **Credential Security**: Avoid using privileged accounts for live analysis on compromised systems to prevent credential theft.
    - **Dump Integrity**: Always verify dump file checksums during remote transfers.
    - **Malware Risks**: Use intermediate hosts and avoid reusing removable devices across infected systems to prevent malware spread (e.g., USBferry, Crimson.USBWorm, USBCulprit).
    - **Virtual Machine Snapshots**: Preferred over suspending to preserve original memory state.
    - **Device Memory**: Avoid accessing to prevent system instability.
    - **Tool Testing**: Test tools in controlled environments matching the target system’s OS and architecture.
- **Sensitive Considerations**:
    - **Timing**: Strategic timing of dumps (e.g., during suspect activity or attacker inactivity) can significantly impact evidence quality.
    - **Data Completeness**: Combine memory dumps with pagefile/swap analysis for a comprehensive investigation.
    - **Shared Memory**: Investigate shared memory regions for inter-process communication or library data, which may contain critical evidence.
    - **Heap Analysis**: Prioritize heap inspection for dynamic data like passwords or chat contents, which persist throughout a process’s lifetime.

This summary and notes provide a robust foundation for conducting memory forensics investigations, emphasizing both technical understanding and practical precautions to ensure evidence integrity and investigative success.