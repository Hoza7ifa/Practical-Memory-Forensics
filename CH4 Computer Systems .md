# CH4: Computer Systems

---

- English Summary
    
    <aside>
    <img src="https://www.notion.so/icons/graduate_lightgray.svg" alt="https://www.notion.so/icons/graduate_lightgray.svg" width="40px" />
    
    # 🌟 Chapter 4: Computer Systems 🌟
    
    Digital forensics demands mastery over the chaotic world of computer systems—hardware, boot processes, and filesystems. This chapter equips you with the knowledge to control the environment, preserve evidence, and navigate the tech jungle. Here’s a vibrant, thorough breakdown for your Notion page! 🌈
    
    ---
    
    ## ✨ Overview: Decoding Computer Systems
    
    To protect evidence integrity, you *must* understand how systems boot, how data is stored, and how filesystems organize it all. Missteps here can alter evidence or tank your courtroom cred. This chapter covers:
    
    - **Boot Process**: From power-on to OS.
    - **Filesystems**: How data lives on drives.
    - **NTFS Deep Dive**: The king of Windows filesystems.
    
    Let’s break it down! 🔍
    
    ---
    
    ## 🔧 1. Understanding the Boot Process
    
    ### 🔑 What’s the Boot Process?
    
    - **Definition**: Power button → running OS, step-by-step like climbing a ladder.
    - **Why It Matters**: Boot mishaps can overwrite evidence—control is everything.
    
    ### ⚡ Steps of the Boot Process
    
    1. **Power-On Self-Test (POST)**:
        - CPU tests motherboard via ROM/BIOS.
        - Beep codes signal errors (Google if no manual!).
    2. **BIOS Activation**:
        - Pre-storage phase—runs from motherboard.
        - Shows RAM, CPU, drives, date/time (snap a pic to document!).
        - Access via key combo (e.g., F2)—time’s tight, or it boots the suspect’s drive!
        - **Pro Tip**: Disconnect drives first to avoid evidence loss.
        - Sets boot order (CD/DVD → HDD, tweakable).
    3. **UEFI (Post-2010)**:
        - Replaces BIOS with perks:
            - Secure Boot (authenticates OS).
            - Faster startup, >2TB drive support, 64-bit drivers, GPT.
        - **Catch**: Secure Boot blocks unsigned forensic tools—disable it if needed.
    4. **Boot Loader**:
        - **BIOS**: Finds Master Boot Record (MBR) at sector 0—partition info + boot code.
        - **UEFI**: Uses GUID Partition Table (GPT)—protective MBR + partition entries.
        - Hands off to OS to finish booting.
    
    ### 🛡️ Forensic Control
    
    - **Goal**: Boot into *your* environment, not suspect’s.
    - **How**: Change boot order to forensic media (CD/USB).
    
    ---
    
    ## 💿 2. Forensic Boot Media
    
    ### 🔑 Why Use It?
    
    - **Scenario**: Can’t remove drive? Boot from CD/USB to image it safely.
    - **Goal**: Create a sound forensic environment—no changes to source.
    
    ### 🛠️ Options
    
    1. **Linux-Based (Paladin)**:
        - Free, USB-bootable, live OS.
        - Sumuri offers preloaded USBs + support.
    2. **Windows-Based (WinFE)**:
        - By Troy Larson (2008), Mini-WinFE by Brett Shavers.
        - Runs X-Ways, FTK Imager—not heavy suites (EnCase/FTK).
    
    ### ⚠️ Challenges
    
    - Boot order tweaks in BIOS/UEFI.
    - Short key-press window.
    - Older systems may not USB-boot.
    - Filesystem write-protection varies.
    - Secure Boot must be disabled (F2/F12 → Security → Off).
    
    ### 🛠️ Creating Boot Media
    
    - **Tools**: 8GB+ USB, Paladin ISO, Rufus (free).
    - **Steps**:
        1. Run Rufus.
        2. Select USB (Device), Paladin ISO (Boot Selection).
        3. Pick MBR (BIOS/UEFI) or GPT (UEFI-only).
        4. Default format options → Start.
    - **Result**: Bootable forensic USB—test it first!
    
    ---
    
    ## 💾 3. Hard Drives
    
    ### 🔑 Physical vs. Logical
    
    - **Physical**: The drive (platters/SSD chips).
    - **Logical**: Partitions/volumes (e.g., “C:” is a partition, not the drive).
    
    ### 🛠️ Hard Drive Anatomy
    
    - **Platters**: Metal/glass, magnetic coating, spin at 1000s RPM.
    - **Heads**: Read/write, <0.1 microns from surface.
    - **Actuator**: Moves heads.
    - **Interfaces**: SCSI (old, corporate), IDE (old, consumer), SATA/SAS (current).
    
    ### ⚡ SSDs
    
    - **No Moving Parts**: Memory chips—lighter, faster, reliable.
    - **Firmware Ops**:
        - Wear leveling (spreads writes).
        - Trim (wipes unallocated space).
        - Garbage collection (reuses blocks).
    - **Forensic Impact**: Unallocated data vanishes—firmware runs on power-up, unstoppable.
    
    ### 📏 Drive Geometry
    
    - **Structure**: Tracks (circles), sectors (smallest unit, 512B or 4KB), cylinders (vertical track sets).
    - **Addressing**:
        - **CHS**: Cylinder-Head-Sector (old, limited).
        - **LBA**: Logical Block Addressing (scales big).
    
    ---
    
    ## 🗂️ 4. Partitioning
    
    ### 🔑 MBR (Master Boot Record)
    
    - **Location**: Sector 0, 512B.
    - **Layout**:
        - 440B boot code.
        - 64B partition table (4 primaries max).
        - 2B signature.
    - **Limits**: 4 primaries—workaround with extended partitions.
    - **Table**:
        - 0x80 = active, 0x00 = inactive.
        - Filesystem IDs (e.g., 0x07 = NTFS, 0xDE = Dell utils).
    - **Extended Partitions**:
        - Replaces 1 primary, splits into logical volumes.
        - Extended Boot Record (EBR) daisy-chains logicals.
    
    ### 🌐 GPT (GUID Partition Table)
    
    - **UEFI Pair**: Replaces MBR.
    - **Structure**:
        - Sector 0: Protective MBR (0xEE = GPT).
        - Sector 1: GPT header (“EFI PART”).
        - Sector 2+: Partition entries (128B each, up to 128 partitions).
    - **GUIDs**: Unique 128-bit IDs (e.g., 00112233-4455-6677-8899-aabbccddeeff).
        - Types: V1 (time+MAC), V3/V5 (hash+namespace), V4 (random).
    
    ### 🕵️ Hidden Areas
    
    - **HPA**: Manufacturer-locked (recovery tools).
    - **DCO**: Config overlay—hides capacity (e.g., 500GB vs. 600GB).
    - **Forensic Note**: Users can hide data here with tools—check with X-Ways/FTK Imager.
    
    ---
    
    ## 📂 5. Understanding Filesystems
    
    ### 🔑 What’s a Filesystem?
    
    - **Role**: Tracks files/partitions, manages clusters (1+ sectors).
    - **Cluster**: Smallest writable unit (bigger than a sector).
    
    ### 🗄️ FAT Filesystem
    
    - **Evolution**:
        - FAT12 (1977, 4,096 clusters, floppy-era).
        - FAT16 (1984, still limited).
        - VFAT (Win95, long filenames).
        - FAT32 (28-bit, 2.2TB theoretical, 32GB/4GB limits).
    - **Layout**:
        - **System Area**: VBR, FAT tables (file locations).
        - **Data Area**: Root dir + files.
    - **Use**: Removable drives (USB, SD)—universal compatibility.
    
    ---
    
    ## 🔐 6. Understanding the NTFS Filesystem
    
    ### 🔑 NTFS Basics
    
    - **MFT (Master File Table)**: Heart of NTFS—stores file metadata + small files.
    - **File Record**: 1,024B, tracks file details via attributes.
    
    ### 🛠️ Key Components
    
    1. **VBR**: Partition sector 0—boot code + volume info.
    2. **MFT**:
        - Starts at cluster 0 or offset 0x0C.
        - Records files/folders (e.g., $MFT, $LogFile).
    3. **Attributes**: Data blocks in file records—resident (in MFT) or non-resident (elsewhere).
    
    ### 📋 MFT File Record Layout
    
    - **Header**: “FILE”, offsets, sequence numbers.
    - **Attributes**:
        - **$10 Standard Info**: Timestamps, permissions.
        - **$30 File Name**: Long/short names (255 chars max).
        - **$80 Data**: File content (resident if <1KB, else non-resident).
    
    ### ⚡ Resident vs. Non-Resident
    
    - **Resident**: Small files (e.g., 23B resident.txt) fit in MFT.
    - **Non-Resident**: Larger files link to external clusters.
    
    ### 🕵️ Example Attributes
    
    - **$10**: Created 6/20/19 1:32 PM, Security ID 264.
    - **$30**: longfilename.txt, parent dir record 5.
    - **$80**: Dropbox adds named data (com.dropbox.attributes).
    
    ### **NTFS (New Technology File System)**
    
    - **Default for Windows**.
    - **Key Components:**
        - **$MFT (Master File Table)** – Tracks all files.
        - **$Boot** – Boot record.
        - **$LogFile** – Transaction log.
    - **File Attributes:**
        - **$STANDARD_INFORMATION** (timestamps).
        - **$FILE_NAME** (filename in Unicode).
        - **$DATA** (file content).
    - **Resident vs. Non-Resident Data:**
        - **Resident** – Small files stored in MFT.
        - **Non-Resident** – Large files stored in clusters (run lists).
    
    ---
    
    ## **Summary & Key Takeaways**
    
    1. **Boot Process** – Understand BIOS/UEFI, MBR/GPT.
    2. **Forensic Boot Media** – Use Linux/WinFE for imaging.
    3. **Storage Devices** – HDD vs. SSD, wear leveling, TRIM.
    4. **Partitioning** – MBR (4 partitions) vs. GPT (128 partitions).
    5. **Filesystems** – FAT (simple, recoverable) vs. NTFS (complex, metadata-rich).
    </aside>
    
- Arabic Summary
    
    <aside>
    <img src="https://www.notion.so/icons/graduate_lightgray.svg" alt="https://www.notion.so/icons/graduate_lightgray.svg" width="40px" />
    
    ### **الفصل الرابع: فهم أنظمة التشغيل والتخزين في التحقيقات الرقمية**
    
    ### **1. عملية الإقلاع وأنظمة التخزين**
    
    **أ. مراحل الإقلاع الأساسية:**
    
    1. **اختبار التشغيل الذاتي (POST)**
        - فحص مكونات الأجهزة الأساسية
        - رموز صوتية تشير إلى الأعطال
    2. **تنشيط BIOS/UEFI**
        - BIOS (نظام قديم):
            - واجهة نصية بسيطة
            - دعم محدود للأقراص >2TB
        - UEFI (نظام حديث):
            - واجهة رسومية
            - ميزة "الإقلاع الآمن" (Secure Boot)
            - دعم GPT للأقراص الكبيرة
    
    **ب. مقارنة بين أنظمة التقسيم:**
    
    | **الميزة** | **MBR** | **GPT** |
    | --- | --- | --- |
    | الحد الأقصى للأقسام | 4 أساسية | 128 |
    | دعم أحجام الأقراص | حتى 2TB | أكثر من 2TB |
    | التوافق | أنظمة قديمة | أنظمة حديثة |
    | الحماية | لا | MBR وقائي |
    
    ### **2. وسائط التخزين الحديثة**
    
    **أ. محركات SSD: التحديات الشرعية**
    
    - **مشكلات فريدة:**
        - تسوية التآكل (Wear Leveling)
        - أمر TRIM (يمحو البيانات نهائيًا)
        - صعوبة استعادة الملفات المحذوفة
    - **إستراتيجيات التعامل:**
        
        mermaid
        
        Copy
        
        ```
        graph TD
          A[التصوير الفوري] --> B[استخدام أدوات متخصصة]
          B --> C[توثيق حالة الجهاز الأولية]
          C --> D[التحقق من البرامج الثابتة]
        ```
        
    
    **ب. مقارنة واجهات التخزين:**
    
    | **الواجهة** | **السرعة** | **الاستخدام الشائع** |
    | --- | --- | --- |
    | SATA III | 6Gbps | أجهزة المستهلكين |
    | NVMe | 32Gbps | أجهزة عالية الأداء |
    | SAS | 12Gbps | بيئات المؤسسات |
    
    ### **3. أنظمة الملفات الرئيسية**
    
    **أ. نظام FAT (File Allocation Table)**
    
    - **الخصائص:**
        - هيكل بسيط (VBR + FAT + منطقة البيانات)
        - إدخالات الدليل (32 بايت لكل ملف)
        - علامة xE5 للملفات المحذوفة
    - **استعادة البيانات:**
        
        python
        
        Copy
        
        ```
        def recover_fat_file(entry):
            if entry[0] == 0xE5:
                entry[0] = original_first_char
                rebuild_fat_chain()
        ```
        
    
    **ب. نظام NTFS (New Technology File System)**
    
    - **المكونات الرئيسية:**
        - $MFT (جدول الملفات الرئيسي)
        - السمات الأساسية:
            - $STANDARD_INFORMATION (0x10)
            - $FILE_NAME (0x30)
            - $DATA (0x80)
    - **تخزين البيانات:**
        
        
        | **نوع الملف** | **موقع التخزين** |
        | --- | --- |
        | ملفات صغيرة (<1KB) | داخل سجل MFT |
        | ملفات كبيرة | قوائم التشغيل (Run Lists) |
    
    ### **4. أدوات العمل الشرعي**
    
    **أ. وسائط الإقلاع الشرعية:**
    
    1. **Paladin (Linux-based)**
        - دعم واسع للأجهزة
        - واجهة سهلة للمبتدئين
    2. **WinFE (Windows-based)**
        - توافق مع أدوات Windows
        - مثالي لتحليل أنظمة NTFS
    
    **ب. إعداد وسائط الإقلاع:**
    
    bash
    
    Copy
    
    ```
    # مثال باستخدام Rufus
    rufus -d /dev/sdb -f paladin.iso --partition-scheme=GPT
    ```
    
    ### **5. حالات عملية**
    
    **أ. تحليل قرص مشبوه:**
    
    1. **الخطوات الأساسية:**
        - التحقق من نظام التقسيم (MBR/GPT)
        - تحديد نظام الملفات (FAT/NTFS)
        - استخراج سجلات $MFT (لـ NTFS)
        - تحليل المساحة المتروكة (Slack Space)
    2. **أدوات مقترحة:**
        - Autopsy للتحليل الأولي
        - X-Ways للفحص المتقدم
        - FTK Imager لإنشاء الصور الشرعية
    
    ### **6. التحديات والحلول**
    
    **أ. مشكلات شائعة:**
    
    | **المشكلة** | **الحل** |
    | --- | --- |
    | أقراص كبيرة (>2TB) | استخدام GPT بدلاً من MBR |
    | مشاكل الإقلاع الآمن | تعطيله في إعدادات UEFI |
    | بيانات SSD مفقودة | التصوير قبل التوصيل بالكهرباء |
    
    **ب. أفضل الممارسات:**
    
    1. توثيق كل خطوة في سلسلة الحراسة
    2. استخدام أداتين مختلفتين للتحقق
    3. حفظ نسختين من الصور الشرعية
    
    ### **الخلاصة التقنية**
    
    text
    
    Copy
    
    ```
    ┌───────────────────────┐
    │ فهم النظام المستهدف │
    ├───────────┬───────────┤
    │   الأجهزة  │  البرمجيات │
    └───────────┴───────────┘
             ↓
    ┌───────────────────────┐
    │ تحديد إستراتيجية التحليل │
    ├───────────┬───────────┤
    │  القرص الحي  │  الصور الشرعية │
    └───────────┴───────────┘
    ```
    
    **نصيحة أخيرة:** "التحقق المزدوج من كل خطوة يمنع 90% من الأخطاء الشرعية" - خبير الطب الشرعي الرقمي
    
    </aside>