# CH3: Acquisition of Evidence

---

- English Summary
    
    <aside>
    <img src="https://www.notion.so/icons/graduate_lightgray.svg" alt="https://www.notion.so/icons/graduate_lightgray.svg" width="40px" />
    
    # Chapter 3: Acquisition of Evidence 🌟
    
    # Chapter 3: Acquisition of Evidence 🌟
    
    Digital evidence is fragile—handle it wrong, and it’s gone forever or tainted beyond use. This chapter is your guide to getting it right, from understanding evidence to creating pristine forensic images. Here’s a colorful, thorough breakdown you can pop straight into Notion! 🚀
    
    ---
    
    ## ✨ Overview: Mastering Evidence Acquisition
    
    Mistakes can ruin investigations—lost data, questioned integrity, or courtroom flops. This chapter arms you with tools and processes to:
    
    - **Explore Evidence**: What counts and why it’s tricky.
    - **Control the Environment**: Keep it forensically sound.
    - **Validate Tools**: Ensure accuracy.
    - **Create Sterile Media**: Avoid contamination.
    - **Master Forensic Imaging**: Protect and copy evidence right.
    
    Let’s dive in step-by-step! 🌈
    
    ---
    
    ## 🔍 1. Exploring Evidence
    
    ### 🔑 What Is Evidence?
    
    - **Definition**: Facts proving or disproving something—simple, right? Not quite.
    - **Reality**: Laws, regulations, and jurisdictions complicate it. The *trier of fact* (judge/jury) decides what’s valid.
    - **Challenge**: Even solid evidence (e.g., blood, gloves) can fail if challenged well (e.g., OJ Simpson case).
    
    ### ⚔️ Evidence Attacks
    
    - **How**: Opponents target the evidence itself or the collection/analysis process.
    - **Examples**:
        - **Thumb Cache Case**: Examiner assumed similar URIs (e.g., file:///media/bob/Picture Drive/New) meant the same drive—oops, wrong user (bob vs. bobby). Honest mistake, but it hurt credibility.
        - **Child Luring Case**: Confession, chats, images—slam dunk? Nope. Deleted texts and edited confession video led to acquittal.
    - **Lesson**: Sloppy handling or shortcuts kill evidence in court.
    
    ### 🛡️ Mitigation
    
    - **Chain of Custody**: Track every move—field to lab.
    - **Best Practices**: No shortcuts—validate every step yourself.
    - **Mindset**: Assume every action will be scrutinized.
    
    ---
    
    ## 🏢 2. Understanding the Forensic Examination Environment
    
    ### 🔑 Forensically Sound Examination Environment
    
    - **Concept**: You control everything—no unintended actions, predictable outcomes.
    - **Where**: Lab, office, or field—it’s a *mindset*, not just a place.
    - **Why**: Prevents blunders (e.g., imaging your own laptop instead of the suspect’s—yikes!).
    
    ### 😱 Real-Life Oops
    
    - **Story**: Two colleagues imaged workstations remotely. One accidentally imaged their forensic laptop (found their own files!). Lucky backup saved the day—but imagine if it hadn’t?
    - **Takeaway**: Stay methodical—details matter.
    
    ---
    
    ## 🛠️ 3. Tool Validation
    
    ### 🔑 Why Validate?
    
    - **Risk**: Faulty tools (e.g., Casey Anthony’s chloroform search error—84 vs. 1) taint findings.
    - **Defense**: Opposing counsel will recreate your exam—same tools, different tools, any discrepancy hurts you.
    - **You, Not Them**: Validate personally—don’t trust third-party claims.
    
    ### 🌐 How to Validate
    
    - **Tools**: Test with known datasets (e.g., NIST’s CFReDS: http://www.cfreds.nist.gov).
    - **Example**: NIST DCFL control image expects two files:
        - Scientific_control.mp3 (MD5: e73a608dfb422a206ce7a62deb90ff9b)
        - Export_me.JPG (MD5: c0c3892606849fd76a8534ef80956705)
    - **Test**:
        - **Autopsy**: Finds both—matches doc.
        - **X-Ways**: Confirms filenames, hashes—spot on.
    - **Policy**: Log tests (when, how)—unlogged validation = questionable in court.
    
    ---
    
    ## 🧼 4. Creating Sterile Media
    
    ### 🔑 What’s Sterile Media?
    
    - **Definition**: Storage wiped to all 00 bytes—no leftover data.
    - **Why**: Prevents cross-contamination (old case data mixing with new).
    - **When**: Before imaging (destination) and after (disposal).
    
    ### 🕰️ Why Still Relevant?
    
    - **Past**: Forensic copies (bit-for-bit, no wrapper) needed sterile media—unallocated space could hold old data.
    - **Now**: Forensic images (e.g., E01) wrap data, but sterile media still ensures no surprises for opposing counsel.
    
    ### 🧹 How to Sterilize
    
    - **Tool**: Paladin (Ubuntu live boot—USB/CD).
    - **Steps**:
        1. Open Disk Manager—pick device (e.g., /dev/sdb).
        2. Click “Wipe”—overwrites with 00.
        3. Save log (shows sectors wiped, completion time).
    - **Verify**: X-Ways—compute 64-bit checksum (all zeros = success). MD5/SHA-1 won’t cut it here.
    
    ### 🚮 Old Drives?
    
    - **Wipe**: Before disposal—protects confidential/contraband data.
    
    ---
    
    ## 🔒 5. Understanding Write Blocking
    
    ### 🔑 Why Block Writes?
    
    - **Fragility**: Plugging into Windows? It writes, altering evidence.
    - **Goal**: Zero changes to source device.
    
    ### 🛠️ Hardware Write Blocker
    
    - **What**: Physical device (e.g., Tableau T35u) between computer and source—blocks writes.
    - **Pros**: Reliable, standalone options exist.
    - **Tested**: NIST’s CFTT validates (e.g., T35u report).
    
    ### 💻 Software Write Blocker
    
    - **What**: OS tweak (e.g., Windows registry) or bootable OS (e.g., Paladin).
    - **Paladin**: Mounts nothing by default—choose “read-only” (green) vs. “read/write” (red).
    - **Caution**: Never mount read/write unless intentional.
    
    ---
    
    ## 📸 6. Defining Forensic Imaging
    
    ### 🔑 Why Image?
    
    - **Rule**: Never analyze originals—use copies. Copies = evidence too!
    - **What’s Copied**: Everything—allocated, deleted, slack, unallocated, unpartitioned.
    
    ### 📋 Copy Types (Recap)
    
    - **Forensic Copy**: Bit-for-bit, no wrapper—rare now.
    - **Forensic Image**: Bit-for-bit in a format (DD, E01, AFF)—protective wrapper.
    - **Logical Image**: Specific files only (e.g., server logs)—no deleted data.
    - **Not Backups**: Commercial backups skip forensic detail.
    
    ### 🖼️ Common Formats
    
    - **DD**:
        - UNIX tool—raw, no compression.
        - Variants: dc3dd (error logs, splitting), dcfldd (misaligns faulty drives—beware).
        - Size: Matches source or segmented (e.g., .001, .dd).
    - **E01 (EnCase)**:
        - Adds metadata (case info, dates, notes).
        - Security: CRC every 64 sectors, MD5 of data blocks.
        - Compresses (optional).
    
    ### ⚡ SSD Challenges
    
    - **Firmware**: Wear leveling, garbage collection (trim)—data moves or wipes post-imaging.
    - **Hash Shift**: Pre/post hashes may differ—explainable, not a flaw.
    
    ### 🛠️ Imaging Tools
    
    - **FTK Imager**:
        1. Pre-hash source (e.g., 2GB USB).
        2. Pick source (Physical Drive = all data).
        3. Choose format (DD, E01, SMART, AFF).
        4. Set destination, filename (e.g., usb001), fragment size (0 = no split).
        5. Verify post-creation—hashes match? Done!
    - **Paladin**:
        1. Boot live—Disk Manager lists devices (e.g., /dev/sdc).
        2. Pre-hash via Verify.
        3. Select source, format (DD, E01, Ex01, DMG, VMDK, VHD), destination (mounted read/write).
        4. Label (e.g., usb001)—create and log (via dc3dd).
    
    Digital evidence is fragile—handle it wrong, and it’s gone forever or tainted beyond use. This chapter is your guide to getting it right, from understanding evidence to creating pristine forensic images. Here’s a colorful, thorough breakdown you can pop straight into Notion! 🚀
    
    ---
    
    ## ✨ Overview: Mastering Evidence Acquisition
    
    Mistakes can ruin investigations—lost data, questioned integrity, or courtroom flops. This chapter arms you with tools and processes to:
    
    - **Explore Evidence**: What counts and why it’s tricky.
    - **Control the Environment**: Keep it forensically sound.
    - **Validate Tools**: Ensure accuracy.
    - **Create Sterile Media**: Avoid contamination.
    - **Master Forensic Imaging**: Protect and copy evidence right.
    
    Let’s dive in step-by-step! 🌈
    
    ---
    
    ## 🔍 1. Exploring Evidence
    
    ### 🔑 What Is Evidence?
    
    - **Definition**: Facts proving or disproving something—simple, right? Not quite.
    - **Reality**: Laws, regulations, and jurisdictions complicate it. The *trier of fact* (judge/jury) decides what’s valid.
    - **Challenge**: Even solid evidence (e.g., blood, gloves) can fail if challenged well (e.g., OJ Simpson case).
    
    ### ⚔️ Evidence Attacks
    
    - **How**: Opponents target the evidence itself or the collection/analysis process.
    - **Examples**:
        - **Thumb Cache Case**: Examiner assumed similar URIs (e.g., file:///media/bob/Picture Drive/New) meant the same drive—oops, wrong user (bob vs. bobby). Honest mistake, but it hurt credibility.
        - **Child Luring Case**: Confession, chats, images—slam dunk? Nope. Deleted texts and edited confession video led to acquittal.
    - **Lesson**: Sloppy handling or shortcuts kill evidence in court.
    
    ### 🛡️ Mitigation
    
    - **Chain of Custody**: Track every move—field to lab.
    - **Best Practices**: No shortcuts—validate every step yourself.
    - **Mindset**: Assume every action will be scrutinized.
    
    ---
    
    ## 🏢 2. Understanding the Forensic Examination Environment
    
    ### 🔑 Forensically Sound Examination Environment
    
    - **Concept**: You control everything—no unintended actions, predictable outcomes.
    - **Where**: Lab, office, or field—it’s a *mindset*, not just a place.
    - **Why**: Prevents blunders (e.g., imaging your own laptop instead of the suspect’s—yikes!).
    
    ### 😱 Real-Life Oops
    
    - **Story**: Two colleagues imaged workstations remotely. One accidentally imaged their forensic laptop (found their own files!). Lucky backup saved the day—but imagine if it hadn’t?
    - **Takeaway**: Stay methodical—details matter.
    
    ---
    
    ## 🛠️ 3. Tool Validation
    
    ### 🔑 Why Validate?
    
    - **Risk**: Faulty tools (e.g., Casey Anthony’s chloroform search error—84 vs. 1) taint findings.
    - **Defense**: Opposing counsel will recreate your exam—same tools, different tools, any discrepancy hurts you.
    - **You, Not Them**: Validate personally—don’t trust third-party claims.
    
    ### 🌐 How to Validate
    
    - **Tools**: Test with known datasets (e.g., NIST’s CFReDS: http://www.cfreds.nist.gov).
    - **Example**: NIST DCFL control image expects two files:
        - Scientific_control.mp3 (MD5: e73a608dfb422a206ce7a62deb90ff9b)
        - Export_me.JPG (MD5: c0c3892606849fd76a8534ef80956705)
    - **Test**:
        - **Autopsy**: Finds both—matches doc.
        - **X-Ways**: Confirms filenames, hashes—spot on.
    - **Policy**: Log tests (when, how)—unlogged validation = questionable in court.
    
    ---
    
    ## 🧼 4. Creating Sterile Media
    
    ### 🔑 What’s Sterile Media?
    
    - **Definition**: Storage wiped to all 00 bytes—no leftover data.
    - **Why**: Prevents cross-contamination (old case data mixing with new).
    - **When**: Before imaging (destination) and after (disposal).
    
    ### 🕰️ Why Still Relevant?
    
    - **Past**: Forensic copies (bit-for-bit, no wrapper) needed sterile media—unallocated space could hold old data.
    - **Now**: Forensic images (e.g., E01) wrap data, but sterile media still ensures no surprises for opposing counsel.
    
    ### 🧹 How to Sterilize
    
    - **Tool**: Paladin (Ubuntu live boot—USB/CD).
    - **Steps**:
        1. Open Disk Manager—pick device (e.g., /dev/sdb).
        2. Click “Wipe”—overwrites with 00.
        3. Save log (shows sectors wiped, completion time).
    - **Verify**: X-Ways—compute 64-bit checksum (all zeros = success). MD5/SHA-1 won’t cut it here.
    
    ### 🚮 Old Drives?
    
    - **Wipe**: Before disposal—protects confidential/contraband data.
    
    ---
    
    ## 🔒 5. Understanding Write Blocking
    
    ### 🔑 Why Block Writes?
    
    - **Fragility**: Plugging into Windows? It writes, altering evidence.
    - **Goal**: Zero changes to source device.
    
    ### 🛠️ Hardware Write Blocker
    
    - **What**: Physical device (e.g., Tableau T35u) between computer and source—blocks writes.
    - **Pros**: Reliable, standalone options exist.
    - **Tested**: NIST’s CFTT validates (e.g., T35u report).
    
    ### 💻 Software Write Blocker
    
    - **What**: OS tweak (e.g., Windows registry) or bootable OS (e.g., Paladin).
    - **Paladin**: Mounts nothing by default—choose “read-only” (green) vs. “read/write” (red).
    - **Caution**: Never mount read/write unless intentional.
    
    ---
    
    ## 📸 6. Defining Forensic Imaging
    
    ### 🔑 Why Image?
    
    - **Rule**: Never analyze originals—use copies. Copies = evidence too!
    - **What’s Copied**: Everything—allocated, deleted, slack, unallocated, unpartitioned.
    
    ### 📋 Copy Types (Recap)
    
    - **Forensic Copy**: Bit-for-bit, no wrapper—rare now.
    - **Forensic Image**: Bit-for-bit in a format (DD, E01, AFF)—protective wrapper.
    - **Logical Image**: Specific files only (e.g., server logs)—no deleted data.
    - **Not Backups**: Commercial backups skip forensic detail.
    
    ### 🖼️ Common Formats
    
    - **DD**:
        - UNIX tool—raw, no compression.
        - Variants: dc3dd (error logs, splitting), dcfldd (misaligns faulty drives—beware).
        - Size: Matches source or segmented (e.g., .001, .dd).
    - **E01 (EnCase)**:
        - Adds metadata (case info, dates, notes).
        - Security: CRC every 64 sectors, MD5 of data blocks.
        - Compresses (optional).
    
    ### ⚡ SSD Challenges
    
    - **Firmware**: Wear leveling, garbage collection (trim)—data moves or wipes post-imaging.
    - **Hash Shift**: Pre/post hashes may differ—explainable, not a flaw.
    
    ### 🛠️ Imaging Tools
    
    - **FTK Imager**:
        1. Pre-hash source (e.g., 2GB USB).
        2. Pick source (Physical Drive = all data).
        3. Choose format (DD, E01, SMART, AFF).
        4. Set destination, filename (e.g., usb001), fragment size (0 = no split).
        5. Verify post-creation—hashes match? Done!
    - **Paladin**:
        1. Boot live—Disk Manager lists devices (e.g., /dev/sdc).
        2. Pre-hash via Verify.
        3. Select source, format (DD, E01, Ex01, DMG, VMDK, VHD), destination (mounted read/write).
        4. Label (e.g., usb001)—create and log (via dc3dd).
    
    ---
    
    ## **📄 Summary & Key Takeaways**
    
    - **Evidence Integrity**: Follow strict protocols to avoid legal challenges.
    - **Tool Validation**: Regularly test tools with control datasets.
    - **Sterile Media**: Always wipe drives before/after use.
    - **Write Blocking**: Essential for preserving original evidence.
    - **Imaging**: Prefer **`E01`** for metadata or **`DD`** for speed; avoid logical images unless restricted.
    </aside>
    
- Arabic Summary
    
    <aside>
    <img src="https://www.notion.so/icons/graduate_lightgray.svg" alt="https://www.notion.so/icons/graduate_lightgray.svg" width="40px" />
    
    ### **الفصل الثالث: التحقق من الأدلة الرقمية وإنشاء الصور الشرعية**
    
    ### **1. أهمية التعامل الصحيح مع الأدلة الرقمية**
    
    - **طبيعة الأدلة الرقمية الحساسة**:
        - تتلف بسهولة عند التعامل غير السليم
        - أي خطأ في الإجراءات قد يؤدي إلى رفضها قضائيًا
    - **المثال التحذيري**: قضية قتل حيث تم الطعن في أدلة الهاتف المحمول بسبب عدم توثيق سلسلة الحراسة
    
    ### **2. متطلبات بيئة الفحص الشرعي السليمة**
    
    | **العنصر** | **الشرح** |
    | --- | --- |
    | **التحكم الكامل** | يجب أن يسيطر المحقق تمامًا على بيئة العمل |
    | **التوثيق الدقيق** | تسجيل كل إجراء ونتيجته المتوقعة |
    | **المنهجية** | اتباع إجراءات موحدة وقابلة للتكرار |
    
    **تحذير**: مثال عملي عن محقق قام بتصوير القرص الخاطئ بسبب عدم التحقق المزدوج
    
    ### **3. التحقق من أدوات التحليل**
    
    - **أهمية التحقق**:
        - ضمان دقة النتائج (قضية Casey Anthony)
        - التصدي للطعون القانونية
    - **طرق التحقق**:
        1. استخدام مجموعات اختبار NIST (CFReDS)
        2. المقارنة بين أدوات متعددة (Autopsy vs X-Ways)
        3. التوثيق المستمر للعمليات
    
    ### **4. معالجة وسائط التخزين**
    
    - **تعقيم الوسائط**:
        - الكتابة بالصفر (0x00) على كامل المساحة
        - التحقق عبر checksum 64-bit
    - **حالات الاستخدام**:
        - قبل جمع الأدلة (منع التلوث)
        - بعد انتهاء التحقيق (حماية البيانات الحساسة)
    
    ### **5. تقنيات حظر الكتابة**
    
    - **أنواع مانعات الكتابة**:
        
        mermaid
        
        Copy
        
        ```
        graph LR
          A[مانعات الكتابة] --> B[Hardware]
          A --> C[Software]
          B --> D[أجهزة مادية مثل Tableau]
          C --> E[تعديلات نظام التشغيل]
          C --> F[أنظمة تمهيد خاصة مثل Paladin]
        ```
        
    
    ### **6. إنشاء الصور الشرعية**
    
    - **المقارنة بين أنواع الصور**:
        
        
        | **النوع** | **المميزات** | **العيوب** | **الاستخدام الأمثل** |
        | --- | --- | --- | --- |
        | Forensic Copy | نسخة طبق الأصل | حجم كبير | حالات نادرة |
        | E01 Image | ضغط+تشفير | معالجة أبطأ | معظم التحقيقات |
        | Logical Image | حجم صغير | فقدان البيانات المحذوفة | حالات محدودة |
    - **أدوات التصوير الموصى بها**:
        - **FTK Imager** (للأجهزة الفردية)
        - **Paladin** (للبيئات المعقدة)
    
    ### **7. التحديات الخاصة بمحركات SSD**
    
    - **المخاطر**:
        - Wear Leveling
        - Garbage Collection
        - أمر TRIM
    - **إستراتيجيات المواجهة**:
        1. التصوير الفوري
        2. استخدام أدوات متخصصة
        3. توثيق حالة الجهاز الأولية
    
    ### **8. أفضل الممارسات للتقارير**
    
    1. **التوثيق الشامل**:
        - سلسلة الحراسة
        - قيم التجزئة قبل/بعد
        - إصدارات الأدوات المستخدمة
    2. **المراجعة المزدوجة**:
        - تدقيق تقني
        - مراجعة لغوية
    
    ### **الخلاصة**
    
    - **الدروس الرئيسية**:
        - الأدلة الرقمية هشة وتتطلب إجراءات صارمة
        - التحقق من الأدوات ليس خيارًا بل ضرورة
        - البيئة الشرعية تبدأ بعقلية المحقق قبل أن تكون أدوات
    
    **نصيحة عملية**: احتفظ دائمًا بـ:
    
    1. نسختين من الصور الشرعية
    2. سجلات التحقق ثلاثية النسخ
    3. أدوات متعددة للتحقق المتبادل
    </aside>