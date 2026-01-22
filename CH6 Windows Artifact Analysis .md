# CH6: Windows Artifact Analysis

---

- English Summary
    
    <aside>
    <img src="https://www.notion.so/icons/graduate_lightgray.svg" alt="https://www.notion.so/icons/graduate_lightgray.svg" width="40px" />
    
    # 🌟 Chapter 6: Windows Artifact Analysis 🌟
    
    Welcome to a deep dive into *Chapter 6: Windows Artifact Analysis* from *Learn Computer Forensics*! This chapter is a goldmine for digital forensics engineers, offering a comprehensive guide to uncovering critical artifacts in Microsoft Windows systems, which dominate ~90% of the OS market. Below is a vibrant, detailed, and organized summary capturing every essential point, tailored to empower you in your forensic investigations. Let’s unlock the secrets of Windows artifacts! 🚀
    
    ---
    
    ## ✨ Overview: Why Windows Artifacts Matter
    
    Windows is the backbone of most systems you’ll investigate, far outpacing macOS and Linux. This chapter focuses on **Windows 7, 8, and 10** (with nods to XP’s legacy) and equips you to:
    
    - **Locate Evidence**: Find artifacts proving user or system actions.
    - **Reconstruct Activity**: Trace account usage, file access, program execution, and more.
    - **Analyze Context**: Use timelines and locations to support or refute allegations.
    
    **Key Topics Covered**:
    
    - User Profiles
    - Windows Registry
    - Account Usage
    - File Knowledge
    - Physical Locations
    - Program Execution
    - USB/Attached Devices
    
    Each section is packed with practical insights, registry paths, tools (commercial and open-source), and artifact locations to make your investigations precise and efficient. Let’s explore! 🔍
    
    ---
    
    ## 👤 1. Understanding User Profiles
    
    User profiles are your starting point for tracking user activity, housing critical data about preferences, files, and settings.
    
    ### 🔑 Key Points
    
    - **Purpose**: Profiles store user-specific data, created when a user first logs in.
    - **Locations**:
        - **Windows XP/2000/NT**: C:\Documents and Settings\<UserName>
        - **Windows Vista/7/8/10**: C:\Users\<UserName>
    - **Profile Types**:
        - **Local**: Stored on the local disk, changes stay local.
        - **Roaming**: Network-based, syncs with a server (Enterprise environments).
        - **Mandatory**: Locked by admins, no user changes allowed.
        - **Temporary**: Created on profile load errors, deleted on logout.
    
    ### 🗂️ Folder Structure
    
    Each profile contains:
    
    - **Core Folders**: Documents, Music, Pictures, Videos.
    - **AppData Subfolders**:
        - **Roaming**: Syncs data like browser bookmarks, cookies, recent files (\AppData\Roaming\Microsoft\Windows\[Cookies, Recent, Start Menu]).
        - **Local**: Workstation-specific, stores temp files, program data (\AppData\Local\Microsoft\Windows\History).
        - **LocalLow**: Low-access data (e.g., browser temp files in protected mode).
    
    ### ⚡ Forensic Value
    
    - **NTUSER.DAT**: Each profile has this registry hive, mapped to HKEY_CURRENT_USER at login, storing user settings.
    - **UsrClass.dat:** Located in **`AppData\Local\Microsoft\Windows`**; tracks GUI preferences.v
    - **Tip**: Examine AppData for user behavior (e.g., recent files, browser data).
    
    ---
    
    ## 🗄️ 2. Understanding Windows Registry
    
    The Windows Registry is the heart of the OS, a hierarchical database storing settings for users, hardware, and apps.
    
    ### 🔑 Key Points
    
    - **Definition**: Central storage for configuration data, constantly referenced by Windows.
    - **Location**: Hive files in \%SystemRoot%\System32\Config (e.g., SAM, SECURITY, SOFTWARE, SYSTEM).
    - **Additional Hives**:
        - **NTUSER.DAT**: In user profile root, stores user-specific settings.
        - **UsrClass.DAT**: In \AppData\Local\Microsoft\Windows, tracks UAC and GUI settings.
    
    ### 🛠️ Hive Breakdown
    
    - **SAM**: Security Accounts Manager, holds user login data.
    - **SECURITY**: Stores security and password info.
    - **SOFTWARE**: Tracks app settings and default OS configurations.
    - **SYSTEM**: Contains hardware and system config data.
    
    ### ⚡ Forensic Tools
    
    - **RegRipper**: Open-source, parses registry hives (by Harlan Carvey).
    - **Registry Explorer**: By Eric Zimmerman, simplifies hive navigation.
    - **Commercial Tools**: X-Ways Forensics, EnCase, FTK.
    
    ### 🌟 Why It Matters
    
    - The registry is a treasure trove for artifacts (e.g., user activity, USB usage).
    - **Challenge**: Raw registry data (subkeys, values) can be hard to decode—use tools to parse.
    
    ---
    
    ## 🔐 3. Determining Account Usage
    
    Identifying who was behind the keyboard is a core forensic task. Artifacts here help tie actions to specific users.
    
    ### 🔑 Key Artifacts
    
    1. **SAM Hive:**
    - **RID (Relative Identifier):**
        - **`500`**: Administrator
        - **`501`**: Guest
        - **`1000+`**: User-created accounts.
    1. **Last Login/Last Password Change**
        - **Path**: C:\Windows\System32\config\SAM\Domains\Account\Users
        - **Tool**: Registry Explorer (Eric Zimmerman).
        - **Details**:
            - Subkey Names lists accounts (e.g., jcloudy).
            - Hexadecimal subkeys (e.g., x3E9) link to accounts, showing login/password data.
            - Example: jcloudy linked to x3E9, revealing account activity.
        - **Tip**: Check for deleted accounts (e.g., defaultuser0).
    2. **Security Identifiers (SIDs)**
        - **Format**: S-1-5-21-<Domain>-<RID> (RID = Relative ID, e.g., 1005).
        - **Use**: Tracks accounts; missing RIDs (e.g., 1001–1004) suggest deleted accounts.
    3. **Event Logs**
        - **Path**: C:\Windows\System32\winevt\Logs (Vista–10).
        - **Classes**: System, Application, Security.
        - **Key Events**:
            - **4624**: Successful login (shows username, time, login type).
            - **4625**: Failed login.
            - **4634/4647**: Logoff.
            - **4648**: Explicit credentials used.
            - **4672**: Elevated permissions (e.g., admin).
            - **4720**: Account created.
            - **4778/4779**: RDP connect/reconnect, disconnect.
        - **Login Types**:
            - **Interactive (2)**: Local keyboard login.
            - **Network**: Remote access.
            - **RemoteInteractive**: RDP login.
            - Others: Batch, Service, Unlock, etc.
        - **Tool**: Event Viewer (filter by event ID).
    
    ### ⚡ Forensic Value
    
    - **Prove User Presence**: Correlate login times with incidents.
    - **Detect Unauthorized Access**: Failed logins or unusual admin rights raise flags.
    - **RDP Analysis**: Refute alibis (e.g., “someone else used my system”).
    
    ---
    
    ## 📂 4. Determining File Knowledge
    
    Proving a user knew about specific files (e.g., contraband, stolen data) is critical in many cases.
    
    ### 🔑 Key Artifacts
    
    1. **Thumbcache**
        - **Path**: \AppData\Local\Microsoft\Windows\Explorer
        - **Purpose**: Stores thumbnails from Windows Explorer (thumbnail view).
        - **Details**: Multiple databases based on thumbnail size; not definitive proof of knowledge (auto-generated).
        - **Linking to Files**:
            - Use Windows.edb (Search Indexing DB):
                - **Windows 7**: SystemIndex_0A table.
                - **Windows 8/10**: SystemIndex_PropertyStore table.
            - Example: Thumbnail ID 96 5a be bc cc 2b f2 27 (reverse to 27 f2 2b cc bc ba 5a 96) linked to C:\Users\jcloudy\Desktop\MyTiredHead.jpg.
        - **Tools**: Thumbcache Viewer (open-source), commercial suites.
    2. **Microsoft Browsers (IE/Edge/File Explorer)**
        - **Purpose**: Tracks local/remote file access and web history.
        - **Locations**:
            - **IE6-7**: \%USERPROFILE%\LocalSettings\History\History.IE5
            - **IE10-11**: \%USERPROFILE%\AppData\Local\Microsoft\Windows\WebCache\WebCacheV*.dat
        - **Details**: WebCacheV*.dat (ESE database) logs file access (e.g., PDFs, JPEGs, DOCX).
        - **Tools**: X-Ways Forensics, ESEDatabaseView (open-source, check Containers table).
    3. **Most Recently Used (MRU)**
        - **Path**: NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer
        - **Details**:
            - **OpenSavePidlMRU**: Tracks last 20 files opened/saved via Common Dialogue.
            - **RecentDocs**: Lists last 150 files opened via Explorer, with subkeys by extension (e.g., .docx, .csv).
            - **Folder Subkey**: Tracks folder access.
        - **Example**: Files like cloudy thoughts (4apr).docx, LeftusesBoycotts.pdf.
        - **Cloud Storage**: Artifacts for OneDrive, CloudLog hint at external data locations.
    4. **Recycle Bin**
        - **Path**: \$Recycle.Bin (hidden, root of fixed disks).
        - **Details**:
            - Subfolders use user SIDs (e.g., S-1-5-21-...-1001).
            - Deleted files renamed: $R<6 chars>.<ext> (data), $I<6 chars>.<ext> (metadata—deletion time, original path).
            - Example: C:\Users\jcloudy\Desktop\Larry King_...html deleted 04/05/2018.
            - Directories: $R holds subdirectories/files, $I tracks metadata.
        - **Challenge**: Emptying the Bin marks clusters as free; $I data (MFT resident) is hard to recover.
    5. **Link (LNK) Files**
        - **Path**: \AppData\Roaming\Microsoft\Windows\Recent
        - **Purpose**: Shortcuts to files/apps, created on double-click or File Open.
        - **Data**: MAC times, file size, path, volume details (persists post-deletion).
        - **Example**: C:\Users\jcloudy\Desktop\AIRPORT INFORMATION.docx (size: 172684, modified: 04/04/2018).
        - **Tools**: LECmd (Eric Zimmerman), commercial suites.
    6. **JumpLists**
        - **Purpose**: Track recent files per app (Windows 7+).
        - **Details**: Logs file access with timestamps (e.g., PDFs, HTML files viewed in Chrome).
        - **Example**: LeftusesBoycotts.pdf opened 04/06/2018.
        - **Tools**: JumpList Explorer (Eric Zimmerman), commercial suites.
    7. **Shellbags**
        - **Path**: USRCLASS.DAT (\AppData\Local\Microsoft\Windows)
        - **Purpose**: Tracks folder access (size, location) via GUI.
        - **Details**: Shows access to removable media, cloud storage (e.g., Google Drive, Box Sync, Dropbox).
        - **Example**: Google Drive accessed 03/28/2018.
        - **Tools**: Shellbag Explorer (Eric Zimmerman), commercial suites.
    8. **Prefetch Files**
    - **Purpose:** Speeds up application launches.
    - **Location:** **`%WINDOWS%\Prefetch`** (**`.pf`** files).
    - **Data Captured:**
        - Execution count.
        - Last run time.
    - **Tool:** WinPrefetchView.
    
    ### ⚡ Forensic Value
    
    - **Prove Knowledge**: MRU, LNK, JumpLists, and shellbags show deliberate access.
    - **Trace Data**: Cloud storage artifacts guide you to external evidence.
    - **Recover Deleted Files**: Recycle Bin and LNK files preserve metadata.
    - **Caution**: Thumbcache isn’t conclusive (auto-generated).
    
    ---
    
    ## 🌍 5. Identifying Physical Locations
    
    Pinpointing a system’s location can validate or refute alibis, crucial in cases like network breaches.
    
    ### 🔑 Key Artifacts
    
    1. **Time Zones**
        - **Path**: SYSTEM\CurrentControlSet\Control\TimeZoneInformation
        - **Details**: Stores bias (minutes from GMT), names (e.g., Eastern Standard Time).
        - **Example**: Bias 300 (5 hours), ActiveTimeBias 240 (4 hours).
        - **Tool**: RegRipper.
        - **Caveat**: User can manually set time zones—cross-check with other data.
    2. **Network History**
        - **Paths**:
            - **Wi-Fi Profiles**: C:\ProgramData\Microsoft\Wlansvc\Profiles\Interfaces (XML with SSIDs).
            - **Registry**: SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList
        - **Details**:
            - XML: SSID (e.g., Net 2.4), authentication (e.g., WPA2PSK).
            - Registry: MAC address, connection times (e.g., Net 2.4 connected 03/30/2018).
        - **Tool**: RegRipper.
    3. **WLAN Event Log**
        - **Path**: C:\Windows\System32\winevt\Logs\Microsoft-Windows-WLAN-AutoConfig%4Operational.evtx
        - **Details**: Logs SSIDs, MAC addresses, connection times.
        - **Event IDs**:
            - **11000**: Connection start.
            - **8001**: Connection success.
            - **8002**: Connection failure.
            - **8003**: Disconnection.
            - **6100**: Network association.
        - **Example**: Net 2.4 connected 03/27/2018, 12:15:58 GMT.
    
    ### ⚡ Forensic Value
    
    - **Locate Systems**: Wi-Fi SSIDs and logs tie devices to physical locations.
    - **Break Alibis**: Example case—suspect’s laptop wiped, but mobile Wi-Fi logs placed them at the scene.
    - **Correlate Timelines**: Time zone data aligns events with incident times.
    
    ---
    
    ## 🖥️ 6. Exploring Program Execution
    
    Tracking executed programs reveals user or system activity, from deliberate launches to autostart events.
    
    ### 🔑 Key Artifacts
    
    1. **UserAssist**
        - **Path**: NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist
        - **Details**: Tracks GUI-launched apps, encoded in ROT-13.
            - Shows executable path, run count, last run time.
            - Example: FTK Imager.exe run once on 04/06/2018.
        - **Tool**: RegRipper (auto-decodes).
    2. **Shimcache**
        - **Path**: SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache
        - **Purpose**: Tracks program compatibility issues.
        - **Details**: File path, modify time, cache update time.
            - Example: OneDriveSetup.exe run 03/27/2018.
        - **Tool**: RegRipper.
    3. **Prefetch**
        - **Path**: \%WINDOWS%\Prefetch
        - **Purpose**: Speeds app loading by preloading data to RAM.
        - **Details**: .pf files track executable, files used, run count, last run time.
            - Example: Adjust timestamps by -10 seconds for accuracy.
        - **Tools**: WinPrefetchView (NirSoft), commercial suites.
    
    ### ⚡ Forensic Value
    
    - **Identify Tools**: Reveals apps like FTK Imager, OneDrive, or malware.
    - **Hidden Storage**: Points to encrypted containers, cloud storage, or mobile devices.
    - **Support Hypotheses**: Cross-reference with MRU, JumpLists for robust evidence.
    
    ---
    
    ## 🔌 7. Understanding USB/Attached Devices
    
    USB devices pose risks (data theft, malware) and leave traceable artifacts.
    
    ### 🔑 Key Artifacts
    
    1. **USB Registry Keys**
        - **Paths**:
            - SYSTEM\CurrentControlSet\Enum\USB
            - SYSTEM\CurrentControlSet\Enum\USBSTOR
        - **Details**:
            - USB: Lists devices by Vendor/Product ID (VID/PID), serial numbers, last write times.
            - USBSTOR: Adds device names (e.g., SanDisk Extreme USB Device).
            - Example: Serial AA010215170355310594 connected 03/27/2018.
        - **Note**: Volume serials differ from physical serials; & in second char = no unique serial.
    2. **MountedDevices**
        - **Path**: SYSTEM\MountedDevices
        - **Details**: Maps USB serials to drive letters (e.g., D:, E:).
            - Example: Serial AA010603160707470215 = D:.
    3. **User Association**
        - **Path**: NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2
        - **Details**: Links USB GUIDs to user accounts.
            - Example: GUIDs 3869c27a-... and 5c3108bb-... tied to jcloudy on 03/27/2018.
        - **Tool**: RegRipper.
    
    ### ⚡ Forensic Value
    
    - **Track Devices**: Identify USBs used for data exfiltration or malware.
    - **Attribute Usage**: Tie devices to specific users (e.g., jcloudy).
    - **Map Drives**: Drive letters help locate accessed files.
    
    ---
    
    ## 🌟 Summary: Your Forensic Toolkit
    
    This chapter arms you with the skills to dissect Windows systems like a pro! Here’s what you’ve gained:
    
    - **User Profiles**: Navigate C:\Users, analyze AppData, and leverage NTUSER.DAT for user settings.
    - **Registry**: Master hives (SAM, SYSTEM, etc.) with tools like RegRipper and Registry Explorer.
    - **Account Usage**: Use event logs (e.g., 4624) and SIDs to pinpoint active users.
    - **File Knowledge**: Trace access via thumbcache, MRU, LNK, JumpLists, and Recycle Bin—prove intent or recover deleted data.
    - **Physical Locations**: Locate systems with Wi-Fi logs, time zones, and event IDs (e.g., 11000).
    - **Program Execution**: Track apps with UserAssist, Shimcache, and prefetch to uncover hidden tools.
    - **USB Devices**: Identify devices and user interactions via USBSTOR, MountedDevices, and MountPoints2.
    
    ### 🛠️ Tools to Master
    
    - **Open-Source**: RegRipper, Registry Explorer, Thumbcache Viewer, ESEDatabaseView, LECmd, JumpList Explorer, Shellbag Explorer, WinPrefetchView.
    - **Commercial**: X-Ways Forensics, EnCase, FTK.
    - **Resources**: SANS posters, Microsoft Event ID lists, Carvey’s *Windows Registry Forensics*.
    
    ### ⚡ Pro Tips
    
    - **Cross-Reference**: Combine artifacts (e.g., event logs + LNK files) for robust conclusions.
    - **Stay Updated**: Artifacts evolve with OS updates—monitor changes.
    - **Document Everything**: Timestamps, paths, and tools ensure defensible findings.
    
    ---
    
    ## **9. Key Takeaways**
    
    - **User Activity:** MRUs, JumpLists, and LNK files reveal file access.
    - **Account Tracking:** SAM hive + event logs identify logins/RDP sessions.
    - **Location Evidence:** Wi-Fi logs + time zone settings place the system geographically.
    - **Program Execution:** Prefetch/UserAssist/Shimcache track application usage.
    - **USB Forensics:** Registry keys + MountedDevices tie devices to users.
    
    ### **Essential Tools**
    
    - **Commercial:** X-Ways, EnCase, FTK.
    - **Open-Source:** RegRipper, ESEDatabaseView, LECmd, Shellbag Explorer.
    
    **Next Chapter:** RAM Memory Forensic Analysis.
    
    ---
    
    ### **Practical Applications for Digital Forensics Engineers**
    
    1. **Timeline Construction:** Correlate registry timestamps with event logs.
    2. **Data Recovery:** Recover deleted files from **`$Recycle.Bin`** or unallocated space.
    3. **Attribution:** Use RIDs/SIDs to link actions to specific accounts.
    4. **Cloud Storage:** Track OneDrive/Dropbox via Shellbags/JumpLists.
    5. **Malware Detection:** Analyze Shimcache for unusual executables.
    </aside>
    
- Arabic Summary
    
    <aside>
    <img src="https://www.notion.so/icons/graduate_lightgray.svg" alt="https://www.notion.so/icons/graduate_lightgray.svg" width="40px" />
    
    ### **الفصل السادس: تحليل آثار نظام التشغيل Windows - الدليل الشامل**
    
    ### **1. مقدمة شاملة لنظام Windows الجنائي**
    
    - **نظرة عامة**:
        - نظام Windows هو النظام الأكثر انتشارًا في التحقيقات الرقمية (حصة سوقية تصل إلى 75% حسب إحصاءات 2023)
        - الإصدارات المشمولة: Windows 7/8/10 مع الإشارة إلى الاختلافات الرئيسية بينها
        - أهمية فهم البنية التحتية للنظام في التحقيقات الجنائية
    
    ### **2. تحليل ملفات تعريف المستخدمين (User Profiles)**
    
    **أ. البنية الأساسية:**
    
    ```
    graph TD
        A[ملفات تعريف المستخدم] --> B[Windows XP]
        A --> C[Windows 7+]
        B --> D[المسار: C:\Documents and Settings]
        C --> E[المسار: C:\Users]
        E --> F[المجلدات الرئيسية]
        F --> G[Documents]
        F --> H[AppData]
        H --> I[Roaming]
        H --> J[Local]
        H --> K[LocalLow]
    ```
    
    **ب. أنواع ملفات التعريف:**
    
    | **النوع** | **الموقع** | **الميزات** | **الاستخدام الجنائي** |
    | --- | --- | --- | --- |
    | **محلي** | القرص الصلب | تغييرات دائمة | تحليل نشاط مستخدم معين |
    | **متجول** | خادم الشبكة | مزامنة الإعدادات | تتبع نشاط المستخدم عبر أجهزة متعددة |
    | **إلزامي** | خادم الشبكة | إعدادات مقفلة | كشف محاولات التعديل غير المصرح بها |
    | **مؤقت** | الذاكرة | يحذف عند الخروج | كشف محاولات الدخول الفاشلة |
    
    **ج. الملفات الحيوية:**
    
    - **NTUSER.DAT**: يحتوي على:
        - إعدادات المستخدم الشخصية
        - سجل أنشطة المستخدم
        - مفاتيح التسجيل الخاصة بالمستخدم
    - **مسار الوصول**: **`C:\Users\<Username>\NTUSER.DAT`**
    
    ### **3. سجل Windows (Registry) - كنز المعلومات الجنائية**
    
    **أ. المكونات الرئيسية:**
    
    ```
    # هيكل السجل الأساسي
    registry = {
        "HKEY_CURRENT_USER": "إعدادات المستخدم الحالي",
        "HKEY_LOCAL_MACHINE": "إعدادات النظام العامة",
        "SAM": "معلومات حسابات المستخدمين",
        "SECURITY": "إعدادات الأمان",
        "SOFTWARE": "تطبيقات مثبتة",
        "SYSTEM": "إعدادات النظام"
    }
    ```
    
    **ب. أهم المفاتيح للتحقيق:**
    
    1. **آخر تسجيل دخول**:
        - المسار: **`HKLM\SAM\Domains\Account\Users`**
        - أدوات التحليل: Registry Explorer، RegRipper
        - البيانات المستخرجة:
            - وقت إنشاء الحساب
            - آخر تسجيل دخول
            - عدد مرات الدخول
    2. **سجلات الأحداث (Event Logs)**:
        - الأحداث الرئيسية:
            
            ```
            EventID,Description,Importance
            4624,تسجيل دخول ناجح,High
            4625,محاولة دخول فاشلة,High
            4648,تسجيل دخول بصلاحيات,Critical
            4720,إنشاء حساب جديد,Critical
            ```
            
        - مسار السجلات: **`C:\Windows\System32\winevt\Logs`**
    
    ### **4. تحليل معرفة الملفات (File Knowledge)**
    
    **أ. الذاكرة المؤقتة للصور المصغرة (Thumbcache):**
    
    - **المسار**: **`C:\Users\<User>\AppData\Local\Microsoft\Windows\Explorer`**
    - **الأدوات**:
        - Thumbcache Viewer (مفتوح المصدر)
        - FTK Imager (تجاري)
    - **مثال عملي**:
        
        ```
        thumbcache_viewer.exe -o output.html -i thumbcache_*.db
        ```
        
    
    **ب. ملفات الاختصار (LNK):**
    
    - **المعلومات المستخرجة**:
        - أوقات MAC (التعديل، الوصول، الإنشاء)
        - حجم الملف
        - المسار الكامل
    - **أداة التحليل**: LECmd (Eric Zimmerman)
    
    **ج. قوائم الانتقال (JumpLists):**
    
    - **الأنواع**:
        - تلقائية (**`.automaticDestinations-ms`**)
        - مخصصة (**`.customDestinations-ms`**)
    - **مثال تحليل**:
        
        ```
        jumplist_explorer.exe -f .\5f7b5f1e01b83767.automaticDestinations-ms
        ```
        
    
    ### **5. تحليل المواقع الفعلية**
    
    **أ. المناطق الزمنية:**
    
    - **المفتاح**: **`HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation`**
    - **البيانات الرئيسية**:
        - TimeZoneKeyName
        - ActiveTimeBias (دقائق من GMT)
    
    **ب. سجل الشبكات اللاسلكية:**
    
    - **مسار XML**: **`C:\ProgramData\Microsoft\Wlansvc\Profiles\Interfaces`**
    - **مثال بيانات**:Run HTML
        
        ```
        <WLANProfile><name>شبكة_المنزل</name><SSIDConfig><SSID><hex>486F6D655F4E6574</hex><name>شبكة_المنزل</name></SSID></SSIDConfig></WLANProfile>
        ```
        
    
    ### **6. تحليل تنفيذ البرامج**
    
    **أ. مفتاح UserAssist:**
    
    - **المسار**: **`HKCU\Software\Microsoft\Windows\Currentversion\Explorer\UserAssist`**
    - **فك التشفير (ROT13)**:
        
        ```
        import codecs
        decoded = codecs.encode("Grfg", 'rot13')  # Returns "Test"
        ```
        
    
    **ب. سجلات Prefetch:**
    
    - **المسار**: **`C:\Windows\Prefetch`**
    - **المعلومات المستخرجة**:
        - عدد مرات التنفيذ
        - آخر وقت تشغيل
        - الملفات المستخدمة
    
    ### **7. تحليل أجهزة USB المتصلة**
    
    **أ. المفاتيح الرئيسية:**
    
    1. **`HKLM\SYSTEM\CurrentControlSet\Enum\USB`**
        - VID/PID
        - الأرقام التسلسلية
    2. **`HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR`**
        - أسماء الأجهزة
        - تواريخ الاتصال
    
    **ب. مثال تحليل بأداة RegRipper:**
    
    ```
    rip.exe -r SYSTEM -p usbstor
    ```
    
    ### **8. أدوات التحليل الشاملة**
    
    **قائمة مقارنة الأدوات:**
    
    | **الأداة** | **النوع** | **الاستخدام الرئيسي** | **المميزات** |
    | --- | --- | --- | --- |
    | **RegRipper** | مفتوح | تحليل السجل | دعم أكثر من 200 مكون |
    | **FTK Imager** | تجاري | تصوير وتحليل | واجهة رسومية سهلة |
    | **Eric Zimmerman Tools** | مفتوح | تحليل متقدم | مجموعة أدوات متكاملة |
    
    ### **9. دراسة حالة عملية**
    
    **سيناريو التحقيق:**
    
    1. **الواقعة**: تسريب بيانات حساسة
    2. **خطوات التحليل**:
        - فحص سجلات RDP (EventID 4624)
        - تحليل مفاتيح USBSTOR
        - تتبع ملفات LNK في مجلد Recent
    3. **النتيجة**: تحديد أن الموظف استخدم فلاشة في وقت التسريب
    
    ### **10. نصائح احترافية**
    
    1. **توثيق السلسلة الزمنية**: تسجيل جميع التوقيتات بالتوقيت العالمي (UTC)
    2. **التحقق المتعدد**: استخدام أداتين مختلفتين للنتائج الحرجة
    3. **الفهم العميق**: دراسة بنية السجل قبل التحليل
    4. **التحديث المستمر**: متابعة تغيرات النظام بين إصدارات Windows
    
    ### **11. المراجع الأساسية**
    
    1. كتاب "Windows Forensic Analysis" لهارلان كارفي
    2. موقع ForensicArtifacts.com
    3. أدوات Eric Zimmerman على GitHub
    4. وثائق Microsoft الرسمية لأنظمة الملفات
    
    ### **12. الملحق التقني**
    
    **أوامر مفيدة للتحليل:**
    
    ```
    # استخراج سجلات الأحداث
    Get-WinEvent -Path 'C:\Windows\System32\winevt\Logs\Security.evtx' | Where-Object {$_.Id -eq 4624}
    
    # تحليل Prefetch
    Get-ChildItem "C:\Windows\Prefetch\" | Select-Object Name, LastWriteTime
    ```
    
    **نموذج تقرير التحليل:**
    
    ```
    ## تقرير التحليل الجنائي-**النظام**: Windows 10 Pro
    -**المستخدم**: Admin
    -**النتائج**:
      - 5 أجهزة USB متصلة
      - آخر تسجيل دخول: 2023-05-15 14:22 UTC
      - 12 ملفًا محذوفًا في سلة المحذوفات
    ```
    
    هذا الهيكل يوفر تغطية شاملة لكل جوانب الفصل مع الحفاظ على:
    
    - التسلسل المنطقي
    - الأمثلة العملية
    - التنسيق الجاهز للنسخ
    - التوثيق الدقيق للمسارات والأدوات
    </aside>