# CH8: Email Forensics – Investigation Techniques

---

- English Summary
    
    <aside>
    <img src="https://www.notion.so/icons/graduate_lightgray.svg" alt="https://www.notion.so/icons/graduate_lightgray.svg" width="40px" />
    
    # 🌟 Chapter 8: Email Forensics - Investigation Techniques 🌟
    
    Welcome to a captivating journey through *Chapter 8: Email Forensics - Investigation Techniques* from *Learn Computer Forensics*! This chapter is a vital resource for digital forensics engineers, unraveling the complexities of email as a cornerstone of modern communication—and a vector for criminal activity. Below is a vibrant, comprehensive, and organized summary capturing every essential detail, tailored to empower you in tracing emails, decoding headers, and analyzing artifacts. Let’s dive into the world of email forensics! 🚀
    
    ---
    
    ## ✨ Overview: Why Email Forensics Matters
    
    Email is ubiquitous, serving as a primary communication tool for consumers and corporations alike, and a common platform for criminals to collaborate or perpetrate crimes. As a digital forensics engineer, mastering email investigations is crucial to tracing communications back to their source, identifying co-conspirators, and presenting findings clearly to non-technical audiences. This chapter equips you to:
    
    - **Understand Protocols**: Navigate the standards governing email transmission and retrieval.
    - **Decode Headers**: Extract critical metadata to trace email paths.
    - **Analyze Clients and Webmail**: Uncover artifacts in local and cloud-based email systems.
    - **Locate Evidence**: Find data on local devices, servers, mobile devices, and ISP logs.
    
    **Key Topics Covered**:
    
    - Understanding Email Protocols
    - Decoding Emails
    - Client-Based Email Analysis
    - Webmail Analysis
    
    Each section provides actionable insights, artifact locations, and tools to ensure your investigations are thorough and defensible. Let’s explore! 🔍
    
    ---
    
    ## 📧 1. Understanding Email Protocols
    
    Email protocols are standardized methods enabling hosts to exchange emails across the internet. Knowing these protocols helps you trace email paths and identify artifacts.
    
    ### 🔑 Key Protocols
    
    1. **SMTP (Simple Mail Transfer Protocol)**
        - **Purpose**: Sends emails between servers and from clients to servers.
        - **Standard**: RFC 821, updated to RFC 3207, 5321/5322 (Request for Comments documents define internet standards).
        - **Port**: TCP port 25.
        - **Process**:
            - Sender’s host sends email to an SMTP server.
            - Email relays through multiple SMTP servers to the recipient’s server.
            - Recipient uses a different protocol to retrieve it (see Figure 8.1).
        - **Forensic Value**: SMTP logs on servers reveal transmission paths and timestamps.
    2. **POP3 (Post Office Protocol, Version 3)**
        - **Purpose**: Retrieves emails from a server to a client; designed for receiving only.
        - **Port**: 110.
        - **Features**:
            - Downloads emails, optionally deleting them from the server to save space.
            - Supports offline drafting/reading/replying, syncing at user-defined intervals.
        - **Process**: Sender’s email reaches the server via SMTP; recipient downloads via POP3, choosing to keep or delete server copies (see Figure 8.2).
        - **Forensic Value**: Local client may hold the only email copy if deleted from the server—handle with care.
    3. **IMAP (Internet Message Access Protocol)**
        - **Purpose**: Manages emails on a server, allowing multiple clients to access the same inbox.
        - **Port**: 143.
        - **Features**:
            - Keeps emails on the server until explicitly deleted.
            - Newer than POP3, designed for remote mailbox access (vs. POP3’s download focus).
        - **Process**: Sender’s email reaches the server via SMTP; recipient accesses via IMAP, with server copies retained (see Figure 8.3).
        - **Forensic Value**: Server retention increases chances of recovering deleted emails; client artifacts show access patterns.
    4. **Web-Based Email**
        - **Purpose**: Accesses email via a browser, bypassing traditional clients.
        - **Examples**: Gmail, Yahoo Mail, Outlook/Hotmail, ISP-provided webmail.
        - **Features**:
            - Emails remain on the provider’s server.
            - Deleted emails move to a Trash/Deleted folder, recoverable until permanently purged (timeframe varies by provider).
        - **Forensic Value**: Artifacts in browser cache/history; server data requires legal process (e.g., search warrants).
    
    ### ⚡ Why It Matters
    
    - **Traceability**: Protocols reveal email paths—SMTP for sending, POP3/IMAP for retrieval, webmail for browser access.
    - **Artifact Locations**: Local clients (POP3/IMAP), servers (IMAP/webmail), or browser caches (webmail).
    - **Tip**: Cross-reference protocol artifacts with server logs for robust timelines.
    
    ---
    
    ## 🔍 2. Decoding Emails
    
    Email headers are a goldmine of metadata, revealing the journey from sender to recipient. Decoding them helps you trace origins, identify servers, and detect tampering.
    
    ### 🔑 Key Components
    
    1. **Email Message Format**
        - **User-Visible Fields**:
            - **Subject**: e.g., “background checks.”
            - **Date**: e.g., 07/19/2008 23:39:57 (user-set system time, manipulable).
            - **Sender/Recipient**: e.g., alison@ns7.biz to jean@ns7.biz.
            - **Content**: e.g., Alison requests Jean for a spreadsheet with employee SSNs, asking secrecy.
        - **Forensic Note**: User-created fields (To, From, Subject, Content) are editable and unreliable without header verification.
    2. **Email Header**
        - **Access**: Hidden by default; e.g., Gmail requires “Show original.”
        - **Example Header** (for Jean’s email):
            
            ```
            Return-Path: <simsong@xy.dreamhostps.com>
            X-Original-To: jean@ns7.biz
            Delivered-To: x2789967@spunkymail-mx8.g.dreamhost.com
            Received: from smarty.dreamhost.com (sd-green-bigip-81.dreamhost.com [208.97.132.81]) by spunkymail-mx8.g.dreamhost.com (Postfix) with ESMTP id E32634D80F for <jean@ns7.biz>; Sat, 19 Jul 2008 16:39:57 -0700 (PDT)
            Received: from xy.dreamhostps.com (apache2-xy.xy.dreamhostps.com [208.97.188.9]) by smarty.dreamhost.com (Postfix) with ESMTP id 6E408EE23D for <jean@ns7.biz>; Sat, 19 Jul 2008 16:39:57 -0700 (PDT)
            Received: by xy.dreamhostps.com (Postfix, from userid 558838) id 64C683B1DAE; Sat, 19 Jul 2008 16:39:57 -0700 (PDT)
            To: jean@ns7.biz
            From: alison@ns7.biz
            Subject: background checks
            Message-Id: <20080719233957.64C683B1DAE@xy.dreamhostps.com>
            Date: Sat, 19 Jul 2008 16:39:57 -0700 (PDT)
            ```
            
        - **Key Fields**:
            - **Message-Id**: Globally unique identifier, e.g., <20080719233957.64C683B1DAE@xy.dreamhostps.com>.
                - Format: Alphanumeric string + domain, often with timestamp (e.g., 20080719233957 = 07/19/2008 23:39:57 GMT).
                - Issues: Duplicate IDs indicate non-compliant servers or tampering.
                - Use: Subpoena ISPs for sender details.
            - **Received**: Tracks each server hop, bottom to top (source to destination).
                - Example: First hop at xy.dreamhostps.com (Postfix, userid 558838).
                - Details: Server names, IPs (e.g., 208.97.188.9), timestamps, protocols (ESMTP).
                - Use: Subpoena ISP for userid 558838; trace IPs for server locations.
            - **Return-Path**: Undeliverable message destination, e.g., simsong@xy.dreamhostps.com.
                - Overrides user-visible From field; common in mailing lists.
            - **Optional Fields (X-)**: Non-standard, e.g., X-Priority, X-Mailer (PHPMailer), X-Report-Abuse.
                - Provide server details, spam/virus scan info, or abuse contacts.
                - Example: X-Originating-IP may show sender’s IP (often stripped by providers like Gmail).
        - **IP Addresses**:
            - **Public**: Traceable to ISPs (e.g., 208.97.132.81).
            - **Private**: Untraceable externally (e.g., 10.x.x.x, 192.168.x.x).
    3. **Email Attachments (MIME)**
        - **Standard**: Multipurpose Internet Mail Extensions (RFC for non-ASCII text, binaries, multi-part messages).
        - **Header Indicators**:
            - MIME-Version: 1.0
            - Content-Type: e.g., text/html; charset=us-ascii.
            - Content-Transfer-Encoding: e.g., 7bit (text), Base64 (binaries like JPEGs).
        - **Structure**:
            - Separates email body by data type (e.g., text, images).
            - Each segment has a MIME header with PART keyword.
        - **Forensic Value**: Base64 decoding reveals attachments; MIME headers show file types.
    
    ### ⚡ Forensic Value
    
    - **Trace Origins**: Headers reveal server paths, IPs, and user IDs for subpoenas.
    - **Detect Fraud**: Discrepancies between user fields (From) and headers (Return-Path) suggest spoofing.
    - **Recover Attachments**: MIME decoding extracts files like PDFs or images.
    - **Tip**: Use tools like Postfix logs or header parsers to automate tracing.
    
    ---
    
    ## 💻 3. Understanding Client-Based Email Analysis
    
    Client-based email systems store data locally or sync with servers, leaving rich artifacts for forensic analysis.
    
    ### 🔑 Key Clients
    
    1. **Microsoft Outlook/Outlook Express**
        - **File Types**:
            - **PST**: Personal Storage Table, stores emails, contacts, calendars.
            - **OST**: Offline Storage Table, syncs with servers, used offline.
            - **MDB**: Server-based, common in corporate environments.
        - **Location**: C:\Users\<USER>\AppData\Local\Microsoft\Outlook.
        - **Details**:
            - User can change default paths/names.
            - No login required to access PST/OST files.
            - Large files may fragment in unallocated space, complicating carving.
        - **Forensic Value**: PST/OST hold all email data; carve unallocated space for deleted files.
    2. **Microsoft Windows Live Mail**
        - **Background**: Default in Windows Vista/7, replaced by Windows Mail in Windows 10.
        - **Location**: Stores emails as .eml files in user-defined paths, e.g., under Windows Live Mail.
        - **Features**:
            - Downloads webmail (e.g., Hotmail) to local folders.
            - Example: 54 emails for littlejo@hotmail.com in user folders.
        - **Forensic Value**: .eml files are standard text, readable by forensic tools or text editors.
    3. **Mozilla Thunderbird**
        - **File Type**: MBOX (generic format for email storage).
        - **Location**: C:\Users\<USERNAME>\AppData\Roaming\Thunderbird\Profiles\<PROFILE>.default-release.
        - **Structure**:
            - MBOX files (no extension) store emails by folder (e.g., INBOX, Sent).
            - MSF files (Mail Summary Files) index headers/summaries.
            - Example Files: INBOX, INBOX.msf, Trash.msf, msgFilterRules.dat.
            - Folders: Crash data (minidumps), calendar data, IMAP mailboxes (e.g., imap.mail.yahoo.com).
        - **Forensic Value**:
            - MBOX stores all emails; MSF aids navigation.
            - Parsed emails appear as .eml in tools like X-Ways (see Figure 8.4).
            - Supports other clients (Apple Mail, Opera Mail).
    
    ### ⚡ Forensic Techniques
    
    - **Export Containers**: Open PST/OST/MBOX in a forensic client (e.g., Outlook, Thunderbird).
    - **Forensic Tools**: Commercial suites (X-Ways, EnCase, FTK) parse common formats; open-source options for MBOX.
    - **Challenges**:
        - Fragmentation in large PST/OST files.
        - User-altered paths require thorough searches.
    - **Tip**: Check IMAP folders for server-synced data; carve .eml from unallocated space.
    
    ---
    
    ## 🌐 4. Understanding Webmail Analysis
    
    Webmail, accessed via browsers, is increasingly popular for its ease and accessibility, but poses unique forensic challenges due to server-hosted data.
    
    ### 🔑 Key Points
    
    - **Providers**: Gmail, Yahoo Mail, Outlook/Hotmail, ISP webmail.
    - **Features**:
        - Minimal client configuration; accessible anywhere.
        - Emails stored on provider servers, with services like calendars/address books.
        - Deleted emails recoverable from Trash until permanently purged.
    - **Challenges**:
        - Local artifacts limited to browser cache/history.
        - Content access requires legal process (e.g., U.S. search warrants).
        - Deleted emails may be unrecoverable, depending on provider policies.
    
    ### 🛠️ Artifact Locations
    
    1. **Browser History**
        - **Chrome**:
            - Path: C:\Users\<USER>\AppData\Local\Google\Chrome\User Data\Default\History (SQLite database).
            - Example: Access to badguyneedslove@gmail.com on 08/28/2019, showing 2 unread emails (Figure 8.5).
        - **Firefox**: Path under user profile (e.g., <PROFILE>.default-release).
        - **Forensic Value**: Timestamps, email addresses, and inbox states (e.g., unread counts).
    2. **Browser Cache**
        - **Chrome**:
            - Path: C:\Users\<USER>\AppData\Local\Google\Chrome\User Data\Default\History Provider Cache.
            - Example: Fragmented data, hard to decipher (Figure 8.6).
            - Findings: JSON artifacts with email addresses (e.g., badguyneedslove@gmail.com, badguy76@yahoo.com), names, and SMTP endpoints.
        - **Firefox**: Similar path under profile; yields sparse data (e.g., lookupId: badguyneedslove@gmail.com).
        - **Forensic Value**: Breadcrumbs like email addresses or IDs, but rarely full email content.
    3. **Other Sources**:
        - **RAM**: Captures live webmail sessions (Chapter 7).
        - **Pagefile**: May hold email fragments.
        - **Keyword Searches**: Target email addresses or case-specific terms in cache/RAM.
    
    ### ⚡ Forensic Challenges
    
    - **Dynamic Content**: Gmail’s AJAX/XML (Asynchronous JavaScript) avoids static page storage, complicating reconstruction.
    - **Fragmentation**: Cache data is incomplete, requiring creative searches.
    - **Legal Barriers**: Server data access demands judicial approval, varying by jurisdiction.
    - **Tip**: Serve providers with preservation orders to freeze accounts; combine cache/history with RAM for leads.
    
    ---
    
    ## 🌟 Summary: Your Email Forensics Toolkit
    
    This chapter arms you with the skills to conquer email investigations! Here’s your arsenal:
    
    - **Protocols**:
        - **SMTP**: Sends emails (port 25); trace server logs.
        - **POP3**: Downloads emails (port 110), may delete server copies.
        - **IMAP**: Manages server-based inboxes (port 143), retains copies.
        - **Webmail**: Browser-based, server-hosted, recoverable via legal process.
    - **Decoding Headers**:
        - **Message-Id**: Unique ID for subpoenas.
        - **Received**: Maps server hops (IPs, user IDs).
        - **Return-Path**: True sender address.
        - **MIME**: Decodes attachments (Base64 for binaries).
        - **Caution**: Verify user fields against headers for spoofing.
    - **Client-Based Analysis**:
        - **Outlook**: PST/OST at AppData\Local\Microsoft\Outlook; carve for deleted files.
        - **Windows Live Mail**: .eml files, readable by text editors.
        - **Thunderbird**: MBOX/MSF at AppData\Roaming\Thunderbird; supports IMAP.
    - **Webmail Analysis**:
        - Artifacts in browser history (Chrome\History, Firefox profiles) and cache.
        - Use RAM/pagefile for live data; serve warrants for server content.
        - Gmail’s AJAX limits cache recovery—focus on keywords.
    
    ### 🛠️ Tools to Master
    
    - **Commercial**: X-Ways Forensics, EnCase, FTK (parse PST, MBOX, .eml).
    - **Open-Source**: Thunderbird (MBOX viewer), text editors (.eml), SQLite browsers (Chrome history).
    - **Resources**: *Internet Forensics* by R. Jones (2006).
    
    ### ⚡ Pro Tips
    
    - **Trace IPs**: Public IPs in headers lead to ISPs; private IPs are internal.
    - *Automate Parsing
    </aside>
    
- Arabic Summary
    
    <aside>
    <img src="https://www.notion.so/icons/graduate_lightgray.svg" alt="https://www.notion.so/icons/graduate_lightgray.svg" width="40px" />
    
    ### **الفصل الثامن: التحليل الجنائي للبريد الإلكتروني**
    
    ### **1. مقدمة في التحليل الجنائي للبريد الإلكتروني**
    
    **أهمية التحليل:**
    
    - يمثل البريد الإلكتروني 85% من أدوات التواصل في الجرائم الإلكترونية (حسب INTERPOL 2023)
    - مصادر الأدلة الرئيسية:
        
        mermaid
        
        Copy
        
        ```
        graph TD
          A[مصادر الأدلة] --> B[الأجهزة المحلية]
          A --> C[خوادم البريد]
          A --> D[أجهزة الهاتف]
          A --> E[سجلات مزودي الخدمة]
        ```
        
    
    ### **2. بروتوكولات البريد الإلكتروني الأساسية**
    
    **مقارنة البروتوكولات:**
    
    | **البروتوكول** | **المنفذ** | **الوظيفة** | **الاستخدام الجنائي** |
    | --- | --- | --- | --- |
    | **SMTP** | 25 | الإرسال | تتبع مسار الرسالة |
    | **POP3** | 110 | الاستقبال | استعادة الرسائل المحذوفة |
    | **IMAP** | 143 | الإدارة | تحليل صندوق الوارد الكامل |
    
    ### **3. تحليل رؤوس البريد الإلكتروني (Email Headers)**
    
    **بنية الرأس النموذجي:**
    
    python
    
    Copy
    
    ```
    email_header = {
        "From": "sender@example.com",
        "To": "recipient@domain.com",
        "Subject": "اجتماع الطوارئ",
        "Date": "Thu, 15 Jun 2023 14:22:00 +0300",
        "Message-ID": "<202306151122.ABC123@example.com>",
        "Received": [
            "from mail-server.com (192.168.1.100)",
            "by mx.google.com with SMTP"
        ],
        "X-Originating-IP": "203.0.113.45"
    }
    ```
    
    **أدوات التحليل:**
    
    1. **MXToolBox Header Analyzer**
    2. **MessageHeader في Gmail**
    3. **أداة View Source في Outlook**
    
    ### **4. تحليل عملاء البريد الإلكتروني**
    
    **أ. Microsoft Outlook:**
    
    - **ملفات التخزين**:
        - PST (تخزين شخصي)
        - OST (تخزين غير متصل)
        - **المسار**: **`C:\Users\<user>\AppData\Local\Microsoft\Outlook`**
    - **أدوات الاستخراج**:
        
        bash
        
        Copy
        
        ```
        pst2grep -f suspect.pst -o output.txt
        ```
        
    
    **ب. Mozilla Thunderbird:**
    
    - **بنية الملفات**:
        
        mermaid
        
        Copy
        
        ```
        graph LR
          A[Profile] --> B[Inbox.mbox]
          A --> C[Sent.mbox]
          A --> D[Trash.mbox]
        ```
        
    - **أداة التحليل**: **`mbox-tools`** لتحليل ملفات MBOX
    
    ### **5. تحليل البريد الإلكتروني المستند إلى الويب**
    
    **استراتيجيات التحقيق:**
    
    1. **تحليل سجلات المتصفح**:
        - Chrome: **`History`** SQLite database
        - Firefox: **`places.sqlite`**
    2. **استخراج من ذاكرة التخزين المؤقت**:
        
        bash
        
        Copy
        
        ```
        strings Cache.db | grep "@domain.com"
        ```
        
    3. **إجراءات قانونية**:
        - أوامر الحفظ (Preservation Orders)
        - مذكرات الاستدعاء (Subpoenas)
    
    ### **6. تحليل المرفقات**
    
    **تقنيات الاستخراج:**
    
    1. **فك تشفير Base64**:
        
        python
        
        Copy
        
        ```
        import base64
        decoded = base64.b64decode(encoded_attachment)
        ```
        
    2. **تحليل MIME**:
        - Content-Type: application/pdf
        - Content-Disposition: attachment
    
    ### **7. أدوات متخصصة**
    
    **قائمة مقارنة الأدوات:**
    
    | **الأداة** | **النوع** | **المميزات** | **السعر** |
    | --- | --- | --- | --- |
    | **MailXaminer** | تجاري | دعم متعدد التنسيقات | $$$ |
    | **FTK Email Analyzer** | تجاري | تكامل مع أدوات أخرى | $$$$ |
    | **MBOX Viewer** | مجاني | تحليل Thunderbird | مجاني |
    
    ### **8. دراسة حالة عملية**
    
    **سيناريو التحقيق:**
    
    1. **الواقعة**: تهديد عبر البريد الإلكتروني
    2. **خطوات التحليل**:
        - استخراج الرأس الكامل
        - تتبع سلسلة Received
        - مطابقة عنوان IP مع سجلات ISP
    3. **النتيجة**: تحديد الجهة المرسلة
    
    ### **9. تحديات وحلول**
    
    **أبرز التحديات:**
    
    1. **التشفير**: استخدام PGP/SMIME
        - الحل: البحث عن المفاتيح في الجهاز المحلي
    2. **الخدمات المؤقتة**: مثل ProtonMail
        - الحل: تتبع نشاط المستخدم المرتبط
    3. **حذف الرسائل**:
        - الحل: استعادة من النسخ الاحتياطية للخادم
    
    ### **10. أفضل الممارسات**
    
    1. **توثيق السلسلة**:
        - حفظ نسخة من الرأس الأصلي
        - تسجيل قيم التجزئة
    2. **التحليل الشامل**:
        - فحص العلاقات بين المراسلات
        - تحليل أنماط الاتصال
    3. **التحديث المستمر**:
        - متابعة تغيرات البروتوكولات
        - تحديث أدوات التحليل
    
    ### **11. المراجع الأساسية**
    
    1. RFC 5322 (تنسيق البريد الإلكتروني)
    2. دليل NIST للتحقيق الجنائي الرقمي
    3. وثائق أدوات Volatility للذاكرة
    
    ### **12. ملحق تقني**
    
    **أوامر مفيدة:**
    
    bash
    
    Copy
    
    ```
    # استخراج الرؤوس من ملف EML
    grep -E '^(From|To|Subject|Date):' email.eml
    
    # تحليل MBOX
    readpst -o output_dir inbox.mbox
    ```
    
    **نموذج تقرير التحليل:**
    
    markdown
    
    Copy
    
    ```
    ## تقرير التحليل الجنائي للبريد-**الرسالة**: EM123456
    -**المرسل**: unknown@protonmail.com
    -**المستلم**: victim@company.com
    -**النتائج**:
      - IP المصدر: 192.0.2.45 (مخفي عبر VPN)
      - مرفق خبيث: invoice.exe (SHA256: a1b2...)
    -**التوصيات**: عزل الجهاز المستهدف
    ```
    
    هذا الهيكل يوفر:
    
    - تغطية شاملة لجميع جوانب الفصل
    - أمثلة عملية وتطبيقات واقعية
    - أدوات وتقنيات محدثة
    - تنسيق جاهز للاستخدام المباشر
    </aside>