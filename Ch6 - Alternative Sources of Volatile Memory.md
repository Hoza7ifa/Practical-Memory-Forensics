# Ch6 -  Alternative Sources of Volatile Memory

---

# Detailed Summary of Chapter 6: Alternative Sources of Volatile Memory

This chapter from *Practical Memory Forensics* by Svetlana Ostrovskaya explores alternative sources of volatile memory—hibernation files, pagefiles, swapfiles, and crash dumps—as critical resources for forensic investigations when full memory dumps are unavailable or fail analysis. It details how to acquire and analyze these sources to extract data like processes, files, commands, and malicious activity, complementing traditional memory forensics. The chapter emphasizes their forensic value, acquisition methods, analysis techniques, and limitations, providing practical steps and tools for digital forensics investigators. Below is a comprehensive summary of theoretical concepts, practical applications, tools, commands, and notes, designed as a standalone reference without requiring the original chapter.

## Importance of Alternative Volatile Memory Sources

- **Purpose**: Alternative sources (hibernation files, pagefiles, swapfiles, crash dumps) provide volatile data when full memory dumps are unavailable or corrupted, capturing processes, files, commands, and malicious activity (e.g., malware, attacker tools).
- **Forensic Value**:
    - **Hibernation Files**: Compressed RAM snapshots from hibernation mode, containing user files, processes, and system state at power-off.
    - **Pagefiles/Swapfiles**: Store temporary process data when RAM is insufficient, including commands, IPs, domains, or file fragments.
    - **Crash Dumps**: Capture system or application state during crashes, revealing kernel issues, malware exploitation, or suspicious processes.
- **Applications**: Essential for incident response, malware analysis, and reconstructing attacker actions (e.g., initial access via application vulnerabilities).
- **Limitations**:
    - Hibernation files lack network connection data.
    - Pagefiles/swapfiles store fragmented data, complicating structured analysis.
    - Crash dumps vary in scope (e.g., small dumps lack full memory context).

## Investigating Hibernation Files

### Theoretical Concepts

- **Definition**: A hibernation file (hiberfil.sys) is a compressed RAM copy created when a Windows system enters hibernation mode, a power-saving state that saves memory contents to non-volatile storage (C:\hiberfil.sys) before powering off.
- **Forensic Value**:
    - Captures files, processes, and system state at hibernation time, even if files are later deleted from disk.
    - Useful for recovering user activity, open documents, or malware executed in memory.
- **Limitations**: Lacks active network connection data, as connections are cleared during hibernation.
- **Acquisition Challenges**: Protected system file, hidden by default, requiring imaging tools or forensic image access.

### Practical Steps and Tools

- **Acquiring Hibernation Files**:
    - **Live System**:
        1. Use **FTK Imager** (https://www.accessdata.com/product-download/ftk-imager):
            - Open FTK Imager, select *File -> Add Evidence Item -> Logical Drive*.
            - Choose root (C:\), locate hiberfil.sys.
            - Right-click hiberfil.sys, select *Export Files*, choose removable media (e.g., D:\), and save (Figures 6.1–6.6).
        2. If no hibernation file exists, enable and create one:
            - Open PowerShell as administrator.
            - Check hibernation status: .\powercfg.exe /availablesleepstates.
            - Enable hibernation: .\powercfg.exe /hibernate on.
            - Create hibernation file: .\shutdown.exe /h (enters hibernation mode, creating hiberfil.sys).
    - **Forensic Image**:
        - Open FTK Imager, select *File -> Add Evidence Item -> Image File*.
        - Specify forensic image path, locate hiberfil.sys, and export as above.
- **Analyzing Hibernation Files**:
    - **Conversion to Raw Format**:
        - **Volatility (imagecopy plugin)**:
            - Command: .\volatility_2.6_win64_standalone.exe -f D:\hiberfil.sys --profile=Win7SP1x64 imagecopy -O D:\hiberfil.raw.
            - Purpose: Uncompresses hiberfil.sys into a raw memory image for analysis.
            - Parameters: -f (input file), -O (output file), --profile (OS version, e.g., Win7SP1x64).
        - **Comae Toolkit (Hibr2Bin)**:
            - Download: Register for beta at https://www.comae.com/.
            - Command: Hibr2Bin.exe -i D:\hiberfil.sys -o D:\hiberfil.raw /MAJOR 6 /MINOR 1 (e.g., for Windows 7).
            - Supported OS versions:
                - Windows XP: /MAJOR 5 /MINOR 1
                - Windows XP x64/2003 R2: /MAJOR 5 /MINOR 2
                - Windows Vista/Server 2008: /MAJOR 6 /MINOR 0
                - Windows 7/Server 2008 R2: /MAJOR 6 /MINOR 1
                - Windows 8/Server 2012: /MAJOR 6 /MINOR 2
                - Windows 8.1/Server 2012 R2: /MAJOR 6 /MINOR 3
                - Windows 10/Server 2017: /MAJOR 10 /MINOR 0
    - **Analysis with Volatility**:
        - **pslist Plugin**:
            - Command: .\volatility_2.6_win64_standalone.exe -f D:\hiberfil.raw --profile=Win7SP1x64 pslist.
            - Output: Lists active processes at hibernation time (Figure 6.10).
        - **filescan and dumpfiles Plugins**:
            - Command (list files): .\volatility_2.6_win64_standalone.exe -f D:\hiberfil.raw --profile=Win7SP1x64 filescan > D:\files.txt.
            - Command (extract file): .\volatility_2.6_win64_standalone.exe -f D:\hiberfil.raw --profile=Win7SP1x64 dumpfiles -Q <file_offset> -D D:\output.
            - Output: Lists files in memory, extracts specified files (Figures 6.11–6.12).
        - **Other Plugins**: Apply techniques from Chapters 4–5 (e.g., cmdline, yarascan) for malware or command analysis.
    - **Automated Tools**:
        - **Hibernation Recon** (Arsenal Recon): Paid tool for automated hibernation file analysis.
        - **Magnet AXIOM** / **Belkasoft Evidence Center**: Comprehensive forensic suites for processing hibernation files.
- **Important Notes**:
    - Hibernation files are system-protected and hidden; ensure proper access permissions.
    - Network connection data is absent; focus on processes, files, and commands.
    - Analysis mirrors full memory dump techniques, but data is limited to hibernation time.

## Examining Pagefiles and Swapfiles

### Theoretical Concepts

- **Definition**:
    - **Pagefile (**pagefile.sys**)**: Stores temporary process data when RAM is insufficient, using 4 KB blocks, enabled by default.
    - **Swapfile (**swapfile.sys**)**: Stores data for Microsoft Store apps, functioning like a hibernation mechanism for app switching.
- **Forensic Value**:
    - Contains fragments of processes, commands, IPs, domains, emails, or files (e.g., malicious executables or scripts).
    - Useful for recovering data not in RAM, such as attacker commands or network artifacts.
- **Differences**:
    - Pagefiles store general process data; swapfiles focus on app-specific data.
    - Both are fragmented, complicating structured analysis; relationships to processes are lost after runtime.
- **Challenges**:
    - Data is stored in non-contiguous 4 KB blocks, requiring string searching or file carving.
    - Windows 10 (build 10525+) compresses pagefiles, requiring decompression.
    - No direct process-to-data mapping in analysis.

### Practical Steps and Tools

- **Acquiring Pagefiles**:
    - **Locating Pagefiles**:
        - Check registry: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management for ExistingPageFiles and PagingFiles values (Figure 6.13).
        - Default location: C:\pagefile.sys, but multiple files may exist elsewhere.
        - Tools: Use **Registry Editor** (live system) or **Registry Explorer** (forensic image, https://ericzimmerman.github.io/) to view registry.
    - **Extracting Pagefiles**:
        - **Live System**:
            - Use FTK Imager: *File -> Add Evidence Item -> Logical Drive*, select drive (e.g., C:\), right-click pagefile.sys, select *Export Files* to removable media (Figure 6.14).
        - **Forensic Image**:
            - Use FTK Imager: *File -> Add Evidence Item -> Image File*, select image, locate pagefile.sys, and export.
        - **With Memory Dump**:
            - FTK Imager: Enable *Include pagefile* during memory capture, creating pagefile.sys alongside dump (Figure 6.15).
- **Analyzing Pagefiles**:
    - **Decompression (Windows 10+)**:
        - Tool: **winmem_decompress** (https://github.com/msuhanov/winmem_decompress).
        - Purpose: Decompresses compressed pagefiles for analysis.
    - **Joint Analysis with MemProcFS**:
        - Command: MemProcFS.exe -device D:\memdump.mem -pagefile0 D:\pagefile.sys.
        - Purpose: Combines memory dump and pagefile analysis (default: -pagefile0 for pagefile, -pagefile9 for swapfile).
        - Note: Complements memory dump but may obscure specific pagefile data; analyze separately for focus.
    - **String Searching**:
        - **Strings Utility**:
            - Command: .\strings64.exe D:\pagefile.sys > D:\output.txt.
            - Output: ASCII/Unicode strings (e.g., HTTPS reverse shell commands; Figure 6.17).
            - Steps: Search output.txt for keywords (e.g., cmd, powershell, https).
        - **YARA**:
            - Download: https://github.com/VirusTotal/yara/.
            - Command: .\yara64.exe -r <rule_file> D:\pagefile.sys.
            - Output: Matches URLs, scripts, or malicious patterns (Figure 6.20).
            - Steps: Use public YARA rules or create custom rules for URLs, domains, or malware signatures.
        - **bulk_extractor**:
            - Command: .\bulk_extractor.exe -o D:\output D:\pagefile.sys.
            - Output: Extracts IPs, domains, URLs, emails, SQL queries (Figures 6.21–6.22).
            - Steps: Check IPs on **VirusTotal** (https://www.virustotal.com/gui/home/search) for malicious associations (Figure 6.23).
    - **File Carving**:
        - **PhotoRec** (https://www.cgsecurity.org/wiki/PhotoRec):
            - Command: .\photorec_win.exe D:\pagefile.sys.
            - Steps:
                1. Select media (pagefile.sys), press *Enter* (Figure 6.24).
                2. Confirm NTFS filesystem, press *Enter* (Figure 6.25).
                3. Choose output directory (e.g., D:\output), press *C* to start (Figure 6.26).
            - Output: Recovered files (e.g., DLLs, executables, text files; Figures 6.27–6.28).
            - Analysis: Search carved files for IPs or keywords using PowerShell: Select-String "<IP>" D:\output\* (Figure 6.29).
        - **VirusTotal**: Check carved files (e.g., DLLs) for maliciousness (Figure 6.30).
- **Important Notes**:
    - Pagefile data is fragmented; string searching and file carving are primary methods.
    - Swapfile analysis mirrors pagefile methods but focuses on Microsoft Store app data.
    - Windows 10 pagefile compression requires winmem_decompress.
    - Carved files (e.g., DLLs) may resemble antivirus signatures; verify context to avoid false positives.
    - VirusTotal checks are critical for IPs or files linked to malicious activity.

## Analyzing Crash Dumps

### Theoretical Concepts

- **Definition**: Crash dumps capture system or application state during crashes, caused by kernel issues (system crashes) or application vulnerabilities (process crashes).
- **Types of System Crash Dumps**:
    - **Small Memory Dump** (64 KB/128 KB for 32-bit/64-bit): Lists processes, drivers, and bug check messages.
    - **Kernel Memory Dump**: Captures kernel-mode memory (~1/3 of physical RAM).
    - **Complete Memory Dump**: Full physical memory snapshot, excluding unmapped memory.
    - **Automatic Memory Dump** (Windows 8+ default): Kernel dump with system-managed paging file size.
    - **Active Memory Dump** (Windows 10+): Filters out irrelevant pages, capturing active user/kernel memory.
- **Process Dumps**: Capture application state during crashes (e.g., malware exploiting vulnerabilities), stored in %LocalAppData%\CrashDumps (user processes) or C:\Windows\System32\config\systemprofile\AppData\Local\CrashDumps (system processes).
- **Forensic Value**:
    - System dumps reveal kernel issues or driver-related malware.
    - Process dumps identify malware techniques (e.g., initial access via vulnerabilities).
    - Dumps provide process lists, commands, IPs, or file fragments.
- **Configuration**:
    - Check: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CrashControl for CrashDumpEnabled (Figure 6.32):
        - 0: None
        - 1: Complete/Active
        - 2: Kernel
        - 3: Small
        - 7: Automatic
    - Default path: %SystemRoot%\MEMORY.DMP.
    - Process dumps: Configured via HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\Windows Error Reporting\LocalDumps (DumpType=2 for full dumps).

### Practical Steps and Tools

- **Acquiring Crash Dumps**:
    - **System Crash Dumps**:
        - **Live System**:
            - Verify dump settings: *My Computer -> System and Security -> System -> Advanced Settings -> Startup and Recovery* (Figure 6.31).
            - Simulate crash using **NotMyFault** (https://docs.microsoft.com/en-us/sysinternals/downloads/notmyfault):
                - Run notmyfault.exe as administrator, select crash type (e.g., *High IRQL fault*), click *Crash* (Figure 6.33).
                - Output: MEMORY.DMP created on reboot.
        - **Forensic Image**:
            - Use FTK Imager: *File -> Add Evidence Item -> Image File*, locate MEMORY.DMP, export as with hibernation files.
    - **Process Dumps**:
        - **Task Manager**:
            - Open: *Ctrl+Alt+Delete -> Task Manager*.
            - Right-click suspicious process (e.g., explorer.exe), select *Create dump file* (Figure 6.34).
            - Output: .DMP file (e.g., C:\Users\<user>\AppData\Local\Temp\powershell.DMP; Figure 6.35).
        - **Process Hacker** (https://processhacker.sourceforge.io/downloads.php):
            - Similar to Task Manager, right-click process, create dump (Figure 6.36).
        - **ProcDump** (https://docs.microsoft.com/en-us/sysinternals/downloads/procdump):
            - Command (mini-dump): .\procdump.exe -mm <PID> D:\output.dmp.
            - Command (full dump): .\procdump.exe -ma <PID> D:\output.dmp.
            - Steps: Identify PID in Task Manager (Figure 6.38), run command (Figure 6.39).
- **Analyzing System Crash Dumps**:
    - **WinDbg** (https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools):
        - Install: Download *WinDbg Preview* from Microsoft Store.
        - Steps:
            1. Open WinDbg, select *File -> Open dump file*, choose MEMORY.DMP (Figure 6.40).
            2. Run: !analyze -v (Figure 6.41).
            - Output: Faulty drivers, exception codes, IPs, failure IDs (Figure 6.42).
    - **BlueScreenView** (https://www.nirsoft.net/utils/blue_screen_view.html):
        - Purpose: Analyzes mini-dumps on live systems (Figure 6.43).
        - Limitation: Less effective for postmortem analysis.
    - **SuperDump** (https://github.com/Dynatrace/superdump):
        - Purpose: Automated analysis with graphical reports, supports Windows/Linux dumps (Figure 6.44).
        - Requirements: Docker installation.
        - Steps: Upload dump via web/REST interface, review report.
- **Analyzing Process Dumps**:
    - **Strings**:
        - Command: .\strings64.exe D:\explorer.exe_210813_000718.dmp > D:\explorer.txt.
        - Output: Commands (e.g., malicious cmd execution; Figure 6.45).
        - Steps: Search for keywords (e.g., cmd, exe, yrpaykg) to find malware artifacts (Figures 6.48–6.49).
    - **bulk_extractor**:
        - Command: .\bulk_extractor.exe -o D:\output D:\explorer.exe_210813_000718.dmp.
        - Output: IPs, domains, URLs (Figure 6.46).
        - Steps: Check IPs on VirusTotal for malicious associations (Figure 6.47).
    - **YARA**:
        - Command: .\yara64.exe -r <rule_file> D:\explorer.exe_210813_000718.dmp.
        - Purpose: Detects malicious patterns or scripts.
- **Important Notes**:
    - System dumps require debugging tools (e.g., WinDbg) for kernel/driver analysis.
    - Process dumps are critical for malware analysis, revealing commands or network artifacts.
    - Verify dump settings in CrashControl registry before creating dumps.
    - Carved files or IPs may match antivirus signatures; confirm context to avoid false positives.

## Summary of Key Points

- **Sources**:
    - **Hibernation Files**: Compressed RAM snapshots, capturing processes and files but not network connections.
    - **Pagefiles/Swapfiles**: Fragmented process data, useful for commands, IPs, or files.
    - **Crash Dumps**: System (kernel/driver issues) or process (application crashes) dumps, revealing malware techniques.
- **Acquisition**:
    - Hibernation files: Use FTK Imager or create via powercfg.exe and shutdown.exe.
    - Pagefiles: Locate via registry, extract with FTK Imager.
    - Crash dumps: Simulate system crashes with NotMyFault, create process dumps with Task Manager, Process Hacker, or ProcDump.
- **Analysis**:
    - Hibernation files: Convert to raw with Volatility (imagecopy) or Hibr2Bin, analyze with pslist, filescan, etc.
    - Pagefiles: Decompress (Windows 10+), use Strings, YARA, bulk_extractor, or PhotoRec for carving.
    - Crash dumps: Use WinDbg, BlueScreenView, or SuperDump for system dumps; Strings, bulk_extractor, or YARA for process dumps.
- **Tools and Commands**:
    - **FTK Imager**: Extract files from live systems or forensic images.
    - **Volatility**: imagecopy, pslist, filescan, dumpfiles for hibernation files.
    - **Hibr2Bin**: Convert hibernation files (https://www.comae.com/).
    - **winmem_decompress**: Decompress pagefiles (https://github.com/msuhanov/winmem_decompress).
    - **Strings**: .\strings64.exe <file> > output.txt for text extraction.
    - **YARA**: .\yara64.exe -r <rule> <file> for pattern matching (https://github.com/VirusTotal/yara).
    - **bulk_extractor**: .\bulk_extractor.exe -o <output> <file> for IPs/URLs.
    - **PhotoRec**: .\photorec_win.exe <file> for file carving (https://www.cgsecurity.org/wiki/PhotoRec).
    - **WinDbg**: !analyze -v for system dump analysis (https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools).
    - **BlueScreenView**: Mini-dump analysis (https://www.nirsoft.net/utils/blue_screen_view.html).
    - **SuperDump**: Automated dump analysis (https://github.com/Dynatrace/superdump).
    - **NotMyFault**: Simulate system crashes (https://docs.microsoft.com/en-us/sysinternals/downloads/notmyfault).
    - **ProcDump**: Create process dumps (https://docs.microsoft.com/en-us/sysinternals/downloads/procdump).
    - **Process Hacker**: Process dump creation (https://processhacker.sourceforge.io/downloads.php).
    - **Registry Explorer**: Analyze registry for pagefile/crash dump settings (https://ericzimmerman.github.io/).

## Notes for Digital Forensics Investigators

- **Critical Tools and Paths**:
    - **FTK Imager**: Extract hiberfil.sys, pagefile.sys, or MEMORY.DMP from live systems or forensic images.
    - **Volatility**: Convert hibernation files (imagecopy -f D:\hiberfil.sys -O D:\hiberfil.raw), analyze with pslist, filescan.
    - **Hibr2Bin**: Hibr2Bin.exe -i D:\hiberfil.sys -o D:\hiberfil.raw /MAJOR <X> /MINOR <Y> for specific OS versions.
    - **Strings**: .\strings64.exe D:\pagefile.sys > D:\output.txt or .\strings64.exe D:\explorer.exe_*.dmp > D:\explorer.txt.
    - **YARA**: .\yara64.exe -r rules.yar D:\pagefile.sys for malicious patterns.
    - **bulk_extractor**: .\bulk_extractor.exe -o D:\output D:\pagefile.sys for network artifacts.
    - **PhotoRec**: .\photorec_win.exe D:\pagefile.sys for file recovery.
    - **WinDbg**: !analyze -v on MEMORY.DMP for crash analysis.
    - **ProcDump**: .\procdump.exe -ma <PID> D:\output.dmp for full process dumps.
- **Sensitive Considerations**:
    - **Hibernation Files**: Lack network data; focus on processes and files. Use sandboxes for extracted files to avoid executing malware.
    - **Pagefiles/Swapfiles**: Fragmented data requires string searching or carving. Decompress Windows 10+ pagefiles with winmem_decompress. Verify carved files (e.g., DLLs) on VirusTotal, as they may mimic antivirus signatures (Figure 6.30).
    - **Crash Dumps**: Process dumps are critical for malware analysis (e.g., explorer.exe with malicious commands; Figure 6.45). Check IPs from bulk_extractor on VirusTotal (Figure 6.47). System dumps may reveal kernel-level malware.
    - **Malware Indicators**: Look for HTTPS reverse shells (Figure 6.17), suspicious IPs/domains (Figure 6.23), or executable names (e.g., yrpaykg; Figure 6.49).
    - **Evidence Preservation**: Extracted files may be malicious or corrupted; analyze in isolated environments.
    - **Tool Setup**: Ensure YARA rules are sourced from reputable repositories or customized for specific threats. Docker is required for SuperDump.
    - **Registry Analysis**: Verify CrashControl and Memory Management keys for dump/pagefile settings to ensure proper acquisition.

This summary provides a complete, detailed reference for analyzing alternative volatile memory sources in Windows memory forensics, equipping investigators with the knowledge, tools, and precautions needed to conduct thorough investigations without referring to the original chapter.