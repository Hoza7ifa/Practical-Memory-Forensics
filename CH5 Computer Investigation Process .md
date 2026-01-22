# CH5: Computer Investigation Process

---

- English Summary
    
    <aside>
    <img src="https://www.notion.so/icons/graduate_lightgray.svg" alt="https://www.notion.so/icons/graduate_lightgray.svg" width="40px" />
    
    # 🌟 Chapter 5: Computer Investigation Process 🌟
    
    Dive into the art of digital forensics with *Chapter 5: Computer Investigation Process*! This chapter unveils a structured approach to uncovering digital evidence, ensuring you can navigate complex cases with precision and clarity. From timeline analysis to recovering deleted data, here’s a vibrant, comprehensive summary designed for your Notion page, capturing every critical detail with flair! 🚀
    
    ---
    
    ## ✨ Overview: Mastering the Investigation
    
    Being a digital forensic examiner means having a plan—not just grabbing everything (the "kitchen sink" approach). This chapter equips you to:
    
    - **Identify Evidence**: Pinpoint artifacts that prove or disprove allegations.
    - **Analyze Context**: Place evidence in the user’s activity timeline.
    - **Draw Conclusions**: Determine if a crime occurred, backed by solid data.
    
    Key techniques covered:
    
    - **Timeline Analysis**: Tracking user and system actions chronologically.
    - **Media Analysis**: Examining storage devices for artifacts.
    - **String Search**: Finding keywords across data spaces.
    - **Recovering Deleted Data**: Restoring files from FAT and NTFS systems.
    
    Let’s break it down! 🔍
    
    ---
    
    ## ⏳ 1. Timeline Analysis: The Heart of Context
    
    ### 🔑 Why It Matters
    
    - **Purpose**: Places artifacts in context—presence alone isn’t guilt!
    - **Example**: A case where Google searches for injury treatment were wrongly tied to a father. Timeline analysis revealed the mother’s activity, leading to a not guilty verdict.
    
    ### 🛠️ How It Works
    
    - **MAC Times**: Modified, Accessed, Created timestamps from filesystems—useful but manipulable.
    - **Super Timeline**: Combines multiple sources (logs, browser history, emails) for robust context, pioneered by Rob Lee (SANS Institute).
    - **Sources**: Event logs, filesystem logs, internet history, registry hives, prefetch files, and more.
    - **Time Zone Note**: Convert all times to UTC/GMT for consistency—know your dataset’s zone!
    
    ### ⚡ Tools for Timeline Creation
    
    1. **X-Ways Forensics**
        - **Event List**: Pulls timestamps from filesystem, browser histories, emails, etc.
        - **Case Example**: Investigating a data leak (M57 Jean scenario):
            - File: `m57plan.xls` (MD5: `e23a4eb7f2562f53e88c9dca8b26a153`).
            - Timeline: July 20, 2008, 01:27–01:28 GMT.
            - Findings: Excel launched (prefetch file), spreadsheet opened (link file), email sent to `tuckgorge@gmail.com` (phishing attack).
            - **Strength**: Granular, combining artifacts like emails for deeper insight.
        - **Workflow**: Start broad (1M+ entries), filter (e.g., 4,052 events on July 20), drill down (file-specific activity).
    2. **Plaso Framework (log2timeline)**
        - **Open Source**: Python-based, CLI-driven, creates super timelines.
        - **Tools**:
            - **image_export**: Extracts files (e.g., `image_export --names 'm57plan.xls' jean.001 -w export/files`).
            - **log2timeline**: Builds `.plaso` database from forensic images.
            - **pinfo**: Shows database metadata (execution time, events extracted).
            - **psort**: Filters/sorts events, exports to CSV/XLSX (e.g., `psort -o l2tcsv jean.plaso -w jean.csv`).
            - **psteal**: Combines log2timeline + psort for quick processing (e.g., `psteal --source jean.001 -o l2tcsv -w jean.csv`).
        - **Filters**: Use premade filters (e.g., `windows.txt`) to target data (download from GitHub).
        - **Output Options**: CSV, JSON, XLSX, Elasticsearch, KML, etc.
        - **Tip**: Avoid huge CSVs—XLSX shrinks files (1GB → 35MB for a 20GB image).
    3. **Other Tools**
        - **Commercial**: Belkasoft Evidence Center, Autopsy, Recon Lab, Paladin.
        - **Analysis Platforms**: ELK Stack (Elasticsearch, Logstash, Kibana), TimelineMaker Pro, TimeSketch, Aeon Timeline, Timeline Explorer.
    
    ### 🛡️ Best Practices
    
    - **Validate Tools**: Test against known datasets (e.g., Digital Corpora’s M57 Jean image).
    - **Targeted vs. Kitchen Sink**: Focus on relevant events to avoid data overload.
    - **Context is King**: Correlate user logins, app launches, and file access to confirm timelines.
    
    ---
    
    ## 💾 2. Media Analysis: Digging into Storage
    
    ### 🔑 What’s Media Analysis?
    
    - **Focus**: Physical storage (HDDs, SSDs, USBs, optical discs, smartphones).
    - **Goal**: Find artifacts in forensic images (bit-for-bit copies, not backups).
    
    ### 🛠️ Data Types
    
    - **Allocated Space**: Used by files, recognized by filesystem.
    - **Unallocated Space**: Free for use, may hold deleted data.
    - **Slack Space**: Unused cluster space after a file—potential evidence hideout.
    - **Bad Blocks/Sectors**: Marked defective, can hide data intentionally.
    
    ### 📏 Structure (per Brian Carrier)
    
    - **Disk**: Physical device (HDD, SSD, flash).
    - **Volume**: Container (single/multiple disks), aka partition (single disk).
    - **Filesystem**: Tracks file allocation/clusters within a volume.
    - **Data Unit**: Smallest allocation (clusters in Windows, blocks in UNIX).
    - **Metadata**: Timestamps, file details tracked by filesystem.
    
    ### ⚡ Forensic Approach
    
    - **Objective**: Prove/disprove allegations via artifacts.
    - **Vectors**: Also includes network, software, hardware analysis (media is primary).
    - **Process**: Examine content, allocated/unallocated/slack spaces for relevant data.
    
    ---
    
    ## 🔍 3. String Search: Finding the Needle
    
    ### 🔑 Why Use It?
    
    - **Purpose**: Search for keywords across allocated, unallocated, and slack spaces.
    - **Tools**: Most forensic suites (commercial/open source) support string searches.
    
    ### 🛠️ Keyword Lists
    
    1. **Generic**: Used in all cases, categorized by crime (e.g., fraud vs. illicit images).
    2. **Case-Specific**: Tailored to participants, locations, slang (e.g., usernames, emails, phone numbers).
        - **Tip**: Avoid ambiguous terms (e.g., “kill” in homicide cases—also a programming term).
    
    ### ⚡ Encoding Schemes
    
    - **ASCII**: 256-character limit, US-English based.
    - **Unicode**: 2-byte, supports 65,000+ characters.
    
    ### 🌟 Regular Expressions (Regex)
    
    - **Power**: Matches patterns, not just literal strings (e.g., `ally` catches `allie`).
    - **Common Symbols**:
        - : Zero or more (e.g., `ca*t` → `ct`, `cat`, `caat`).
        - `#`: Number (0-9).
        - `\\`: Literal character (e.g., `\\.` for period).
        - `^`: Start of text.
        - `$`: End of text.
        - `+`: One or more.
        - `{n}`: Repeat n times.
        - `[abc]`: Match single char (b, c, or d).
        - `[^abc]`: Exclude chars.
        - `[0-9]`: Range (digits).
        - `.`: Any char.
        - `?`: Optional char.
        - `|`: OR (e.g., `br(ead|ake)` → `bread`, `brake`).
    - **Examples**:
        - **IP Address**: `\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}`.
        - **US Phone**: `(\\(\\d{3}\\))?[- ]?\\d{3}-\\d{4}`.
    - **Resource**: Use [regexlib.com](http://regexlib.com/) for crafting expressions.
    
    ### 🛡️ Challenges
    
    - **Literal Limitation**: Misses misspellings without regex.
    - **False Positives**: Generic terms inflate results—refine lists carefully.
    
    ---
    
    ## 🗑️ 4. Recovering Deleted Data: Bringing Files Back
    
    ### 🔑 Why Recover?
    
    - **Goal**: Retrieve files marked deleted but not overwritten.
    - **Filesystems**: FAT and NTFS handle deletion differently.
    
    ### 🛠️ FAT Filesystem Recovery
    
    - **Deletion Process**:
        - Directory entry’s first char → `0xE5` (skipped by filesystem).
        - File Allocation Table (FAT) entries → `0x00` (clusters freed).
    - **Recovery Steps**:
        1. **Find Entry**: Locate `0xE5` in directory (hex editor needed).
            - Example: File `HORT TXT`, starting cluster `0x08`, size 39 bytes.
        2. **Check Cluster Size**: Boot record shows 512 bytes/sector, 8 sectors/cluster → 4,096 bytes.
        3. **Restore FAT**: Change cluster 8’s `0x00` to `0xFFFFFFF8` (end of file).
        4. **Fix Directory**: Replace `0xE5` with `_` or  (avoid guessing original char).
        5. **Long Filenames**: Relink short/long entries using same replacement char—short entry holds metadata (timestamps, cluster, size).
    - **Challenges**:
        - Fragmented files complicate cluster chaining.
        - Overwritten clusters lose data.
    
    ### ⚡ NTFS Filesystem Recovery
    
    - **Deletion Process**:
        - MFT record’s sequence count increments.
        - Allocation status → unallocated.
        - Non-resident data: `$Bitmap` marks clusters unallocated.
    - **Recovery Steps**:
        1. **Locate MFT Record**: Search for “FILE” signature in unallocated space.
        2. **Resident Data**: Recover directly from MFT if small (<1KB).
        3. **Non-Resident Data**: Decode MFT for data runs/clusters—recover if not overwritten.
        4. **Reverse Allocation**: Mark record/clusters as allocated (if not reused).
    - **Challenges**:
        - Overwritten MFT records lose metadata, making non-resident recovery tough.
        - Fragmentation increases complexity.
    
    ### 🛡️ Best Practices
    
    - **Act Fast**: Unallocated data risks being overwritten.
    - **Use Hex Editors**: For precise FAT/NTFS edits.
    - **Document Changes**: Avoid bias (e.g., neutral char for `0xE5`).
    
    ---
    
    ## 🌍 Summary
    
    - **Timeline Analysis:** Combines MAC times with logs/artifacts for accuracy.
    - **Plaso Framework:** CLI tools (**`log2timeline`**, **`psort`**) for super timelines.
    - **Media Analysis:** Targets allocated/unallocated space for hidden evidence.
    - **String Search:** Uses keywords/regex to filter data.
    - **Data Recovery:** Reconstructs FAT/NTFS metadata to restore deleted files.
    
    **Next Chapter:** Windows Artifact Analysis (examining file contents).
    
    ---
    
    ### **Tools Mentioned**
    
    - **Commercial:** X-Ways, Belkasoft, Autopsy.
    - **Open-Source:** Plaso, TimeSketch, ELK Stack.
    - **Visualization:** TimelineMaker Pro, Aeon Timeline.
    </aside>
    
- Arabic Summary
    
    <aside>
    <img src="https://www.notion.so/icons/graduate_lightgray.svg" alt="https://www.notion.so/icons/graduate_lightgray.svg" width="40px" />
    
    ### **الفصل الخامس: تحليل الخط الزمني والتحقيق الرقمي المتقدم**
    
    ### **1. أساسيات تحليل الخط الزمني**
    
    **أ. المفهوم والأهمية:**
    
    - **التعريف**: إعادة بناء تسلسل الأحداث الرقمية زمنياً
    - **القيمة الإثباتية**:
        - تحديد أنماط النشاط (مثال: توقيت تحميل الملفات المشبوهة)
        - ربط الأحداث بالسياق البشري (مثل تحديد المستخدم الفعلي)
    
    **ب. مصادر البيانات الرئيسية:**
    
    mermaid
    
    Copy
    
    ```
    pie
        title مصادر بيانات الخط الزمني
        "سجلات نظام الملفات (MAC)" : 35
        "سجلات أحداث النظام" : 25
        "سجل المتصفحات" : 20
        "السجلات الأمنية" : 15
        "البيانات الوصفية للملفات" : 5
    ```
    
    ### **2. أدوات التحليل الزمني المتقدمة**
    
    **أ. مقارنة بين الأدوات الرئيسية:**
    
    | **الأداة** | **النوع** | **المميزات** | **العيوب** |
    | --- | --- | --- | --- |
    | **X-Ways Forensics** | تجارية | - دمج مصادر متعددة- واجهة متكاملة | تكلفة عالية |
    | **Plaso (log2timeline)** | مفتوح المصدر | - مرونة عالية- دعم تنسيقات متعددة | يحتاج خبرة تقنية |
    | **ELK Stack** | مفتوح المصدر | - تحليل كميات كبيرة- تصور بيانات | تعقيد في الإعداد |
    
    **ب. دراسة حالة عملية باستخدام Plaso:**
    
    bash
    
    Copy
    
    ```
    # مثال لاستخدام Plaso
    log2timeline.py --parsers "filestat,chrome_history" case.plaso /evidence/image.dd
    psort.py -o l2tcsv -w timeline.csv case.plaso
    ```
    
    ### **3. تحليل الوسائط المتقدم**
    
    **أ. استراتيجيات استعادة البيانات:**
    
    | **التقنية** | **التطبيق** | **الفعالية** |
    | --- | --- | --- |
    | **تحليل $MFT** | NTFS | عالية جداً |
    | **مسح FAT** | أنظمة قديمة | متوسطة |
    | **استعادة التجزئات** | SSD | محدودة |
    
    **ب. دليل استعادة الملفات المحذوفة:**
    
    1. تحديد نظام الملفات (FAT/NTFS)
    2. البحث عن آثار الإدخالات المحذوفة
    3. إعادة بناء سلاسل البيانات
    4. التحقق من السلامة عبر التجزئات
    
    ### **4. تقنيات البحث المتقدمة**
    
    **أ. التعبيرات النمطية للتحقيق:**
    
    regex
    
    Copy
    
    ```
    # أمثلة عملية:
    \d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}  # عناوين IP
    [0-9]{3}-[0-9]{2}-[0-9]{4}          # أرقام ضمان اجتماعي أمريكي
    ```
    
    **ب. إدارة الكلمات الرئيسية:**
    
    - **قوائم موحدة**: إنشاء مكتبة مصطلحات للجرائم الإلكترونية
    - **السياق القانوني**: تضمين مصطلحات خاصة بكل قضية
    - **التحسين**: تجنب المصطلحات العامة لتقليل الإيجابيات الكاذبة
    
    ### **5. تحليل متكامل للخط الزمني**
    
    **أ. منهجية التحقيق:**
    
    1. **التجميع**: جمع البيانات من مصادر متعددة
    2. **التطبيع**: تحويل كل الطوابع الزمنية لـ UTC
    3. **الربط**: تحديد العلاقات بين الأحداث
    4. **التحقق**: التأكد من اتساق الروايات
    
    **ب. نموذج تفسير الأحداث:**
    
    | **الوقت (UTC)** | **المصدر** | **الحدث** | **السياق** |
    | --- | --- | --- | --- |
    | 2023-05-15 14:22 | $MFT | إنشاء document.pdf | قبل تسريب البيانات |
    | 2023-05-15 14:25 | Browser | تحميل إلى Google Drive | نشاط مشبوه |
    
    ### **6. التحديات والحلول العملية**
    
    **أ. مشكلات شائعة:**
    
    - **تضارب التوقيت**: اختلاف مناطق الزمنية
    - **تلاعب بالطوابع**: أدوات تعديل التواريخ
    - **بيانات SSD**: تأثير TRIM و Wear Leveling
    
    **ب. أفضل الممارسات:**
    
    1. **التوثيق**: تسجيل إعدادات المنطقة الزمنية الأصلية
    2. **التحقق المتبادل**: مقارنة مصادر متعددة لنفس الحدث
    3. **التحديث المستمر**: مواكبة تغيرات أنظمة الملفات
    
    ### **7. دراسات حالة متقدمة**
    
    **أ. تحليل تسريب البيانات:**
    
    - **السيناريو**: موظف يسرق ملفات قبل الاستقالة
    - **الأدلة**:
        - طفرات في نشاط USB
        - عمليات نسخ غير معتادة
        - توافق زمني مع إشعار الاستقالة
    
    **ب. التحقيق في الاختراقات:**
    
    mermaid
    
    Copy
    
    ```
    timeline
        title خط زمني للاختراق
        section المرحلة الأولية
        2023-01-01 : مسح الشبكة
        2023-01-03 : استغلال الثغرة
        section التصعيد
        2023-01-05 : تثبيت backdoor
        2023-01-07 : سرقة بيانات
    ```
    
    ### **8. الأدوات المساعدة**
    
    **أ. سلسلة أدوات مفتوحة المصدر:**
    
    1. **Autopsy**: للتحليل الأولي
    2. **RegRipper**: لسجلات التسجيل
    3. **Bulk Extractor**: لاستخراج البيانات
    
    **ب. نصائح لاختيار الأدوات:**
    
    - **التكامل**: قدرة الأدوات على العمل معاً
    - **القابلية للتكرار**: نتائج متسقة عبر التحليلات
    - **الدعم**: مجتمع نشط للأسئلة الفنية
    
    ### **الخلاصة والتوصيات**
    
    **أ. الدروس المستفادة:**
    
    - تحليل الخط الزمني هو عملية تكرارية تحتاج للصبر
    - الجمع بين الأدلة الرقمية والشهادات البشرية يعزز النتائج
    - التحديث المستمر للمهارات ضروري لمجاراة التطورات
    
    **ب. قائمة مرجعية للتحقيق:**
    
    1. تحديد نطاق التحقيق
    2. جمع البيانات من مصادر متعددة
    3. إنشاء خط زمني موحد
    4. التحقق من النتائج بأدوات مختلفة
    5. توثيق كل الخطوات
    </aside>