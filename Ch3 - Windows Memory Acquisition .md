# Ch3 - Windows Memory Acquisition

---

# Summary of Chapter 3:

This chapter from *Practical Memory Forensics* by Svetlana Ostrovskaya focuses on the practical aspects of acquiring memory dumps from Windows systems, a critical skill for digital forensics investigators given Windows' prevalence as a target for threat actors. It covers Windows-specific memory acquisition challenges, preparation steps, and detailed instructions for using four popular tools: FTK Imager, WinPmem, Belkasoft Live RAM Capturer, and Magnet RAM Capture. Below is a comprehensive summary of the key concepts, tools, procedures, and notes, tailored for a memory forensics investigator.

## Understanding Windows Memory-Acquisition Issues

### Device Memory Challenges

- **Definition**: Device memory is a portion of physical RAM reserved for devices (e.g., video cards, audio cards, PCI cards) to ensure efficient operation.
- **Risks**: Accessing or writing to device memory can lead to unpredictable consequences, such as:
    - Alteration of critical device data, potentially affecting functionality.
    - Loss of forensically significant evidence.
    - System freezing or shutdown in worst-case scenarios.
- **Access Mechanism**: Physical memory access in Windows is facilitated through the `\Device\PhysicalMemory` kernel object.
    - **Pre-Windows Server 2003 SP2**: User-space programs had full read/write access, which was unsafe.
    - **Post-Windows Server 2003 SP2**: Write access is restricted to kernel space, requiring acquisition tools to use kernel-level drivers or special drivers.

### Virtualization Challenges

- **Issue**: Tools may cause system crashes on Windows systems with Virtual Secure Mode (VSM) enabled.
- **Solution**: Modern versions of acquisition tools have addressed this issue, ensuring compatibility with virtualized environments.

## Preparing for Windows Memory Acquisition

### Storage Preparation

- **Requirement**: A sanitized flash drive with sufficient capacity to store both the acquisition tool and the memory dump.
- **Sanitization Importance**:
    - Standard deletion or formatting does not securely erase data; file contents remain recoverable.
    - Forensic wiping is necessary to overwrite the drive with zeros or random data.
- **Wiping Methods**:
    - Write a pre-prepared file matching the drive’s capacity.
    - Use the **Secure Erase** option (if supported by the drive vendor; check the vendor’s official website).

### Important Note

- **Sanitization Rationale**: Standard deletion only marks file space as available, leaving data recoverable. Formatting rewrites master files but does not erase content. Secure wiping ensures no residual data compromises the investigation.

## Acquiring Memory with FTK Imager

### Overview

- **Tool**: AccessData FTK Imager, a free, widely-used tool for disk imaging, live response, and memory acquisition.
- **Functionality**: Creates bit-by-bit disk images, custom content images, and memory dumps using a kernel driver to map `\Device\PhysicalMemory`.
- **Download**:
    - Access via: [https://accessdata.com/products-services/forensic-toolkit-ftk/ftkimager](https://accessdata.com/products-services/forensic-toolkit-ftk/ftkimager)
    - Navigate to **Products & Services | FTK Imager**, follow **Download FTK Imager today!**, and complete a contact form to receive a download link.
- **Installation**: Use InstallShield Wizard to install FTK Imager on the sanitized flash drive.

### Acquisition Steps

1. Connect the flash drive to the target system and launch FTK Imager.
2. Navigate to **File > Capture Memory...** or use the toolbar icon.
3. In the **Memory Capture** dialog:
    - Click **Browse** to select the storage location and specify the dump filename (default: `memdump.mem`).
    - Check **Include pagefile** to capture the pagefile for comprehensive analysis.
4. Click **Capture Memory** to start the process.
5. Monitor progress in the **Memory Progress** dialog (e.g., “Dumping RAM: 168/34GB [3%]”).
6. Output: A `.mem` file ready for analysis with tools like the Volatility Framework.

### Visual References

- **Figure 3.1**: FTK Imager main window.
- **Figure 3.2**: File menu with **Capture Memory** option.
- **Figure 3.3**: Memory Capture dialog window.
- **Figure 3.4**: Imaging progress dialog.

## Acquiring Memory with WinPmem

### Overview

- **Tool**: WinPmem, originally part of Google’s Rekall Framework, now a standalone tool by Velocidex.
- **Compatibility**: Supports Windows XP to Windows 10 (32- and 64-bit).
- **Methods**:
    - **Page Table Entry (PTE) Remapping** (default, most stable).
    - **MMMapToSpace Kernel API**.
    - **Traditional `\Device\PhysicalMemory` Mapping**.
- **Download**:
    - Access via: [https://github.com/Velocidex/winPmem](https://github.com/Velocidex/winPmem)
    - Navigate to **Releases** and download `winpmem_mini_x64.exe`.
- **Characteristics**: Self-contained executable, no additional dependencies, automatically loads the correct driver for x86/x64 systems.

### Acquisition Steps

1. Connect the flash drive to the target system.
2. Open **Command Prompt** or **PowerShell** as Administrator (e.g., search for “PowerShell” in the taskbar).
3. Navigate to the flash drive (e.g., `cd D:\`).
4. Run the command: `.\winpmem_mini_x64.exe memdump.raw`.
5. Monitor the process, which displays details like:
    - Driver extraction path: `C:\Users\<username>\AppData\Local\Temp\pme9628.tmp`.
    - System time and memory ranges (e.g., `Start 0x00001000 - Length 0x00005000`).
    - Acquisition mode: PTE Remapping.
6. Output: A raw memory dump (`memdump.raw`).

### Visual References

- **Figure 3.5**: WinPmem GitHub repository.
- **Figure 3.6**: Running PowerShell from the search box.
- **Figure 3.7**: WinPmem execution in PowerShell.
- **Figure 3.8**: Dump creation progress with WinPmem.

## Acquiring Memory with Belkasoft Live RAM Capturer

### Overview

- **Tool**: Belkasoft Live RAM Capturer, a free tool using kernel drivers for memory extraction.
- **Compatibility**: Supports all 32- and 64-bit Windows versions (XP, Vista, 7, 8, Server 2003, 2008, 10).
- **Download**:
    - Access via: [https://belkasoft.com/](https://belkasoft.com/)
    - Navigate to **Download > Belkasoft Live RAM Capturer**, provide an email, and receive a download link.
    - Extract the archive to the flash drive, which contains `x64` and `x86` folders.

### Acquisition Steps

1. Determine the system architecture:
    - Search for “system” in the taskbar and open **System Information**.
    - Check **System Type** under **System Summary** (e.g., “x64-based PC” for 64-bit).
2. Connect the flash drive and run the appropriate executable (`RamCapture.exe` from `x64` or `x86` folder).
3. Enter the output folder path in the tool’s interface.
4. Click **Capture!** to start the process.
5. Output: A `.mem` file with a default filename based on the acquisition date (customizable).

### Visual References

- **Figure 3.9**: Running System Information from the search box.
- **Figure 3.10**: System-type detection in System Information.
- **Figure 3.11**: Imaging with Belkasoft RAM Capturer.

## Acquiring Memory with Magnet RAM Capture

### Overview

- **Tool**: Magnet RAM Capture, a free tool by Magnet Forensics using a kernel-mode driver.
- **Compatibility**: Creates raw format dumps compatible with open-source tools and forensic suites.
- **Download**:
    - Access via: [https://www.magnetforensics.com/](https://www.magnetforensics.com/)
    - Navigate to **RESOURCES > FREE TOOLS > MAGNET RAM CAPTURE**, complete a form, and receive a download link.
    - Copy `MRCv120.exe` to the flash drive.

### Acquisition Steps

1. Connect the flash drive and run `MRCv120.exe` as Administrator.
2. Select **Segment size** (default: **Don’t Split**, recommended).
3. Click **Browse...** to specify the filename and storage location.
4. Click **Start** to begin imaging.
5. Monitor the progress bar until completion (100%).
6. Output: A raw memory image at the specified location.

### Visual Reference

- **Figure 3.12**: Imaging process with Magnet RAM Capture.

## Summary of Key Points

- **Windows-Specific Challenges**:
    - Device memory access risks data loss or system instability.
    - Post-Windows Server 2003 SP2, `\Device\PhysicalMemory` write access is kernel-restricted, requiring tools to use drivers.
    - Virtualization (VSM) compatibility is critical for modern tools.
- **Preparation**:
    - Use a sanitized flash drive to avoid residual data contamination.
    - Secure wiping is essential (e.g., Secure Erase or overwriting with a large file).
- **Tools**:
    - **FTK Imager**: Versatile, user-friendly, supports pagefile inclusion, outputs `.mem` files.
    - **WinPmem**: Flexible with multiple methods (PTE Remapping default), outputs raw dumps, no installation required.
    - **Belkasoft RAM Capturer**: Compatible with all Windows versions, requires architecture-specific executables, outputs `.mem` files.
    - **Magnet RAM Capture**: Simple interface, raw format output, administrator privileges required.
- **Best Practices**:
    - Test tools on systems similar to the target to ensure reliability.
    - Use raw formats for compatibility with tools like Volatility.
    - Run tools as Administrator for kernel-level access.

## Notes for Digital Forensics Investigators

- **Critical Tools and Paths**:
    - **FTK Imager**: Download from [https://accessdata.com/products-services/forensic-toolkit-ftk/ftkimager](https://accessdata.com/products-services/forensic-toolkit-ftk/ftkimager). Install on flash drive, outputs `memdump.mem`.
    - **WinPmem**: Download `winpmem_mini_x64.exe` from [https://github.com/Velocidex/winPmem](https://github.com/Velocidex/winPmem). Run via `.\winpmem_mini_x64.exe memdump.raw` in PowerShell/Command Prompt.
    - **Belkasoft RAM Capturer**: Download from [https://belkasoft.com/](https://belkasoft.com/), extract `x64` or `x86` folder, run `RamCapture.exe`.
    - **Magnet RAM Capture**: Download `MRCv120.exe` from [https://www.magnetforensics.com/](https://www.magnetforensics.com/), run as Administrator.
    - **System Information**: Access via taskbar search for “system” to check architecture (`x64-based PC` or `x86`).
- **Important Notes**:
    - **Sanitization**: Always wipe flash drives forensically to prevent data contamination. Check vendor support for Secure Erase.
    - **Administrator Privileges**: Required for WinPmem and Magnet RAM Capture to load kernel drivers.
    - **Pagefile Inclusion**: Enable in FTK Imager for comprehensive analysis, as pagefile data may contain critical evidence.
    - **Tool Testing**: Test tools in a controlled environment matching the target system’s OS and architecture to avoid unpredictable behavior.
    - **Raw Format**: Preferred for compatibility with forensic tools like Volatility Framework.
- **Sensitive Considerations**:
    - **Device Memory**: Avoid accessing to prevent system crashes or evidence loss. Modern tools skip these regions.
    - **Virtualization**: Ensure tool compatibility with VSM-enabled systems to avoid crashes.
    - **Storage Safety**: Store dumps on external drives, not the target system, to prevent overwriting forensic data.
    - **Filename Customization**: Use descriptive filenames (e.g., avoid default `memdump.mem` or date-based names) for better organization.
    - **Progress Monitoring**: Tools like FTK Imager and Magnet RAM Capture provide progress bars, while WinPmem shows detailed memory range information, aiding in verifying successful acquisition.

This summary equips memory forensics investigators with practical knowledge and precautions for acquiring Windows memory dumps, ensuring evidence integrity and effective tool usage in incident response and criminal investigations.