<div dir="rtl" markdown="1">

# 📚 جزوه SQL Server DBA - جلسه دوم

<p dir="rtl"><strong>تاریخ:</strong> مرداد ۱۴۰۴ (بر اساس کلاس آنلاین)</p>  
<p dir="rtl"><strong>توضیح:</strong> این جزوه بر اساس نکات خام جلسه دوم کلاس SQL Server DBA دسته‌بندی، ویرایش و تکمیل شده. نگارش روان و حرفه‌ای با نکات بهتر و عملی (مثل مثال‌های اضافی، بهترین شیوه‌ها و هشدارها) بر اساس استانداردهای SQL Server. منابع رسمی Microsoft برای دقت ذکر شده. هدف: یادگیری سریع، جذاب و بدون خستگی! 🚀</p>  

<p dir="rtl">اگر مفید بود، به ریپو استار بده <span style="color: #007bff;">⭐</span> و نظرت رو بنویس. لینک ریپو: [اینجا لینک گیت‌هابت رو بذار، مثلاً github.com/yourusername/sql-server-dba-notes]. (نکته: از خاطرات قبلی‌مون، می‌دونم دوست داری لینک‌های زیبا اضافه کنی – اگر بخوای، کمک می‌کنم!)</p>

</div>

---

<div dir="rtl" markdown="1">

## 🔄 بخش ۱: وضعیت دیتابیس و Recovery
<p dir="rtl">دیتابیس‌ها ممکنه به حالات مختلفی برن که نشون‌دهنده مشکل باشه. درک این حالات برای DBA حیاتیه تا از از دست رفتن داده جلوگیری کنه.</p>

<ul dir="rtl">
  <li><strong>حالت Recovery یا Recovery Pending:</strong></li>
  <li><p dir="rtl"><strong>دلیل ورود به Recovery:</strong> معمولاً بعد از ریستارت ناگهانی (مثل crash یا قطع برق)، SQL Server transaction log رو پردازش می‌کنه تا داده‌ها رو consistent کنه. این شامل فازهای Redo (اعمال تغییرات) و Undo (rollback ناقص‌ها) می‌شه.</p></li>
  <li><p dir="rtl"><strong>نکته کلیدی و بهترین شیوه:</strong> هرگز سرور رو دوباره ریست نکنید! حداقل ۵-۱۰ دقیقه (یا بیشتر بسته به اندازه log) صبر کنید تا خودش کامل بشه. اگر طولانی شد، از Error Log چک کنید و اگر لازم باشه، restore کنید.</p></li>
  <li><p dir="rtl"><strong>نکته اضافی:</strong> برای جلوگیری، همیشه از recovery model مناسب (مثل Full برای HA) استفاده کنید و log backup منظم بگیرید.</p></li>
  <li><p dir="rtl"><strong>دلیل Recovery Pending:</strong> یکی از فایل‌ها (مثل MDF, NDF یا LDF) مشکل داره، مثلاً آسیب‌دیده، گم‌شده یا دسترسی نداره (مثل مجوز فایل).</p></li>
  <li><p dir="rtl"><strong>راه‌حل عملی:</strong> فایل‌ها رو چک کنید و سعی کنید آنلاین کنید: <pre dir="ltr"><code>ALTER DATABASE [DBName] SET ONLINE;</code></pre> اگر نشد، restore از بک‌آپ.</p></li>
  <li><p dir="rtl"><strong>مثال:</strong> برای چک log: <pre dir="ltr"><code>DBCC OPENTRAN([DBName]);</code></pre> تا تراکنش‌های باز رو ببینید.</p></li>
  <li><p dir="rtl"><strong>منبع:</strong> <a href="https://learn.microsoft.com/en-us/sql/relational-databases/databases/database-states?view=sql-server-ver16">Microsoft Docs - Database States</a>.</p></li>
</ul>

</div>

---

<div dir="rtl" markdown="1">

## 🖥️ بخش ۲: مدیریت RAM و مانیتورینگ مصرف
<p dir="rtl">RAM یکی از کلیدی‌ترین منابع برای عملکرد SQL Serverه. تنظیم اشتباه می‌تونه باعث کندی یا crash بشه.</p>

<ul dir="rtl">
  <li><strong>چطور ببینیم SQL Server چقدر RAM داره و مصرف می‌کنه؟</strong></li>
  <li><p dir="rtl">از query جامع: <pre dir="ltr"><code>SELECT physical_memory_in_use_kb / 1024 AS RAM_Used_MB, total_physical_memory_kb / 1024 AS Total_RAM_MB FROM sys.dm_os_process_memory;</code></pre> این لحظه‌ای نشون می‌ده.</p></li>
</ul>

<ul dir="rtl">
  <li><strong>چقدر RAM به SQL Server اختصاص بدیم و چطور تنظیم کنیم؟</strong></li>
  <li><p dir="rtl"><strong>تنظیم Max Memory:</strong> در SSMS، راست‌کلیک روی سرور > Properties > Memory > Max Server Memory (واحد MB). حتماً ادمین باشید و سرور رو ریستارت کنید تا اعمال بشه. همیشه محدود کنید تا OS حداقل رم داشته باشه (برای جلوگیری از paging).</p></li>
  <li><p dir="rtl"><strong>بهترین شیوه:</strong> "۸۰% رم به SQL" علمی نیست – بسته به workload تنظیم کنید. برای سرور با ۱.۲ ترابایت RAM، ۲۰% (۲۴۰ GB) به OS بدید، اما اگر فقط SQL باشه، حداقل ۴ GB کافیه (۱ GB هم کار می‌کنه، اما ۴ GB ایمن‌تره برای پروسس‌های background). برای رم تا ۳۲ GB، ۲۰% قابل قبوله. بعد لود، رم رو مانیتور کنید – اگر SQL رم رو پر نکرد (به خاطر کمبود داده)، مشکلی نیست.</p></li>
  <li><p dir="rtl"><strong>نکته اضافی:</strong> از فرمول: Max Memory = Total RAM - (OS + دیگر اپ‌ها). مثلاً برای ۶۴ GB، ۵۲ GB به SQL بدید. همیشه بعد تنظیم، performance رو تست کنید.</p></li>
  <li><p dir="rtl"><strong>مثال:</strong> Properties > Memory > Max Server Memory = 53248 (برای ۵۲ GB).</p></li>
</ul>

<ul dir="rtl">
  <li><strong>چطور مصرف RAM رو چک کنیم؟</strong></li>
  <li><p dir="rtl">Query اصلی: <pre dir="ltr"><code>SELECT * FROM sys.dm_os_sys_info;</code></pre> (خروجی: cpu_count برای هسته‌ها، physical_memory_kb برای کل رم، committed_kb برای مصرف فعلی، committed_target_kb برای مقدار مطلوب، sqlserver_start_time برای زمان استارت).</p></li>
  <li><p dir="rtl"><strong>روش‌های جایگزین:</strong> Properties از tempdb (Date Created نشان‌دهنده ریستارته، چون tempdb هر بار بازسازی می‌شه) یا از Error Log فعلی.</p></li>
  <li><p dir="rtl"><strong>نکته کلیدی:</strong> اگر committed_kb نزدیک به Max Memory باشه، رم رو افزایش بدید. همه DMVها (dynamic management views با پیشوند dm_) لحظه‌ای هستن و برای مانیتورینگ realtime عالی‌ان.</p></li>
  <li><p dir="rtl"><strong>منبع:</strong> <a href="https://learn.microsoft.com/en-us/sql/database-engine/configure-windows/server-memory-server-configuration-options?view=sql-server-ver16">Microsoft Docs - Server Memory Configuration</a>.</p></li>
</ul>

</div>

---

<div dir="rtl" markdown="1">

## 📈 بخش ۳: Performance Tuning و Buffer Manager
<p dir="rtl">مانیتورینگ عملکرد کمک می‌کنه bottlenecks رو زود پیدا کنید.</p>

<ul dir="rtl">
  <li><strong>استفاده از Performance Monitor در ویندوز:</strong></li>
  <li><p dir="rtl">Counter: SQL Server: Buffer Manager (برای default instance؛ اگر named، نام instance اضافه می‌شه).</p></li>
  <li><p dir="rtl"><strong>Buffer Cache Hit Ratio:</strong> درصد صفحاتی که از RAM خونده شدن (نه دیسک). اگر زیر ۹۰% بره، رم کمه – یعنی SQL زیاد از دیسک می‌خونه.</p></li>
  <li><p dir="rtl"><strong>نکته اضافی:</strong> از counterهای دیگه مثل Page Life Expectancy (اگر زیر ۳۰۰ ثانیه باشه، رم کمه) استفاده کنید.</p></li>
  <li><p dir="rtl">برای خالی کردن RAM (فقط برای تست): <pre dir="ltr"><code>DBCC DROPCLEANBUFFERS;</code></pre> (clean pages رو پاک می‌کنه؛ dirty pages منتظر checkpoint می‌مونن تا به دیسک برن). هرگز روی سرور تولید نزنید – مثل ریستارت عمل می‌کنه و بار اضافه می‌آره!</p></li>
  <li><p dir="rtl"><strong>Clean vs Dirty Pages:</strong> Clean = داده‌های بدون تغییر؛ Dirty = تغییرکرده (مثل پس از UPDATE) که با checkpoint ذخیره می‌شن.</p></li>
  <li><p dir="rtl"><strong>مثال:</strong> کوئری با RAM خالی = Cold Cache (برای تست بدترین سناریو). بعد، warm cache رو چک کنید تا تفاوت عملکرد رو ببینید.</p></li>
  <li><p dir="rtl"><strong>نکته بهترین شیوه:</strong> ریستارت سرور مسکن موقتیه – مشکلات ریشه‌ای مثل پر شدن tempdb یا lockها رو حل کنید. از ابزارهایی مثل Query Store برای tuning طولانی‌مدت استفاده کنید.</p></li>
  <li><p dir="rtl"><strong>منبع:</strong> <a href="https://learn.microsoft.com/en-us/sql/relational-databases/performance/monitor-and-tune-for-performance?view=sql-server-ver16">Microsoft Docs - Performance Monitoring</a>.</p></li>
</ul>

</div>

---

<div dir="rtl" markdown="1">

## ⚙️ بخش ۴: Virtual Memory و Query Execution
<p dir="rtl">Virtual Memory جایگزین RAM نیست و می‌تونه کندی ایجاد کنه.</p>

<ul dir="rtl">
  <li><strong>تغییر Virtual Memory:</strong></li>
  <li><p dir="rtl">مسیر: System Settings > Performance Options > Advanced > Virtual Memory. اندازه رو دستی تنظیم کنید (مثلاً ۱.۵ برابر RAM).</p></li>
  <li><p dir="rtl"><strong>بهترین شیوه:</strong> SQL رو طوری configure کنید که از virtual memory کمتر استفاده کنه (با Max Memory مناسب). کندی paging می‌تونه queryها رو ۱۰ برابر کندتر کنه – اولویت رم فیزیکی باشه.</p></li>
  <li><p dir="rtl"><strong>نکته اضافی:</strong> اگر paging زیاد باشه، از sys.dm_os_performance_counters چک کنید و رم اضافه کنید.</p></li>
  <li><p dir="rtl"><strong>منبع:</strong> <a href="https://learn.microsoft.com/en-us/sql/database-engine/configure-windows/server-memory-server-configuration-options?view=sql-server-ver16">Microsoft Docs - Virtual Memory in SQL</a>.</p></li>
</ul>

</div>

---

<div dir="rtl" markdown="1">

## 🔌 بخش ۵: مدیریت کانکشن‌ها و Sessions
<p dir="rtl">کانکشن‌های باز می‌تونن منابع رو هدر بدن و سرور رو کند کنن.</p>

<ul dir="rtl">
  <li><strong>بررسی Dormant Connections:</strong></li>
  <li><p dir="rtl">Query: <pre dir="ltr"><code>SELECT * FROM sys.dm_exec_sessions WHERE is_user_process = 1 ORDER BY last_request_end_time DESC;</code></pre> (سشن‌های کاربر؛ ID < ۵۰ سیستمیه). Status='Sleeping' یعنی بیکاره و آخرین درخواست رو نشون می‌ده.</p></li>
  <li><p dir="rtl"><strong>نکته کلیدی:</strong> همیشه کانکشن‌ها رو بعد استفاده close کنید (dispose فقط object رو پاک می‌کنه، نه اتصال). اگر سشن‌ها > max threads بشن، سرور جدیدها رو رد می‌کنه و مثل هنگ عمل می‌کنه.</p></li>
  <li><p dir="rtl"><strong>نکته اضافی:</strong> از connection pooling در اپ‌ها استفاده کنید تا باز/بستن مکرر نداشته باشید.</p></li>
  <li><p dir="rtl">جاب برای kill سشن‌های قدیمی (بیش از ۶۰ دقیقه بیکار):</p></li>
  <li><pre dir="ltr"><code>DECLARE @SQL nvarchar(4000) = N'';
SELECT @SQL = CONCAT(@SQL, 'KILL ', session_id, '; ') 
FROM sys.dm_exec_sessions 
WHERE is_user_process = 1 AND DATEDIFF(minute, last_request_start_time, GETDATE()) > 60 
AND last_request_end_time IS NULL AND status = 'Sleeping';
PRINT @SQL;
EXEC (@SQL);</code></pre></li>
  <li><p dir="rtl"><strong>هشدار:</strong> Kill اگر در حال اجرا باشه، rollback می‌کنه – فقط برای sleeping استفاده کنید.</p></li>
  <li><p dir="rtl"><strong>منبع:</strong> <a href="https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-sessions-transact-sql?view=sql-server-ver16">Microsoft Docs - sys.dm_exec_sessions</a>.</p></li>
</ul>

</div>

---

<div dir="rtl" markdown="1">

## 💾 بخش ۶: TempDB و تنظیمات
<p dir="rtl">TempDB برای عملیات موقتی حیاتیه و باید بهینه باشه.</p>

<ul dir="rtl">
  <li><strong>نکات TempDB:</strong></li>
  <li><p dir="rtl">دیفالت در C: هست – حتماً به درایو پرحجم و سریع منتقل کنید (مثل SSD). اگر پر بشه، سرویس stop می‌شه (با ریست حل می‌شه، اما پیشگیری بهتره).</p></li>
  <li><p dir="rtl"><strong>بهترین شیوه:</strong> چند فایل data (مثل ۴-۸ تا بسته به CPU) اضافه کنید تا contention کم بشه. اندازه اولیه رو بزرگ تنظیم کنید تا auto-grow کمتر بشه.</p></li>
  <li><p dir="rtl"><strong>نکته اضافی:</strong> از trace flag 1117 برای رشد یکنواخت فایل‌ها استفاده کنید. (منبع اضافی: <a href="https://learn.microsoft.com/en-us/sql/relational-databases/databases/tempdb-database?view=sql-server-ver16">Microsoft Docs - TempDB Best Practices</a>).</p></li>
  <li><p dir="rtl"><strong>منبع:</strong> <a href="https://learn.microsoft.com/en-us/sql/relational-databases/databases/tempdb-database?view=sql-server-ver16">Microsoft Docs - TempDB Best Practices</a>.</p></li>
</ul>

</div>

---

<div dir="rtl" markdown="1">

## 🧵 بخش ۷: Max Worker Threads
<p dir="rtl">Threadها برای مدیریت concurrent tasks مهمن.</p>

<ul dir="rtl">
  <li><strong>اضافه کردن Thread:</strong></li>
  <li><p dir="rtl">Properties > Advanced > Max Worker Threads (۰ = خودکار؛ بر اساس فرمول: برای ۳۲-bit تا ۲۵۶، برای ۶۴-bit بیشتر).</p></li>
  <li><p dir="rtl"><strong>نکته اضافی:</strong> اگر workload سنگین باشه، به ۵۱۲+ افزایش بدید، اما تست کنید – بیش از حد می‌تونه overhead بسازه.</p></li>
  <li><p dir="rtl"><strong>منبع:</strong> <a href="https://learn.microsoft.com/en-us/sql/database-engine/configure-windows/configure-the-max-worker-threads-server-configuration-option?view=sql-server-ver16">Microsoft Docs - Max Worker Threads</a>.</p></li>
</ul>

</div>

---

<div dir="rtl" markdown="1">

## 🔒 بخش ۸: Locks و Plan Cache
<p dir="rtl">Lockها برای consistency ضرورین، اما می‌تونن بلاکینگ ایجاد کنن.</p>

<ul dir="rtl">
  <li><strong>مدیریت Lock:</strong></li>
  <li><p dir="rtl">تراکنش طولانی رکوردها رو Exclusive Lock (EX) می‌کنه. برای خواندن بدون بلاک: <pre dir="ltr"><code>SELECT * FROM Table WITH (NOLOCK);</code></pre> (اما dirty read ممکنه و write همچنان بلاک می‌شه).</p></li>
  <li><p dir="rtl"><strong>بهترین شیوه:</strong> سشن لاک‌کننده رو پیدا و kill کنید (با sys.dm_tran_locks)، نه ریست سرور! برای جلوگیری از deadlock، ایندکس مناسب بسازید.</p></li>
  <li><p dir="rtl"><strong>نکته اضافی:</strong> از isolation levels مثل READ COMMITTED SNAPSHOT برای کم کردن lock استفاده کنید.</p></li>
  <li><p dir="rtl"><strong>Plan Cache:</strong> بعد ریست خالی می‌شه – SQL پلن‌ها رو کش می‌کنه تا کامپایل دوباره نکنه. ریستارت بار اضافه می‌آره (مثل بازخوانی داده‌ها به RAM).</p></li>
  <li><p dir="rtl"><strong>نکته:</strong> واحد شبکه نباید به SQL دست بزنه! (به فرموده استاد پرویز آقاصادقی 😄).</p></li>
  <li><p dir="rtl"><strong>منبع:</strong> <a href="https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide?view=sql-server-ver16">Microsoft Docs - Locking Guide</a>.</p></li>
</ul>

</div>

---

<div dir="rtl" markdown="1">

## 🗄️ بخش ۹: Storage و Disk Management
<p dir="rtl">دیسک کندترین بخش سروره – انتخاب درستش عملکرد رو چند برابر می‌کنه.</p>

<ul dir="rtl">
  <li><strong>انواع Storage:</strong></li>
  <li><p dir="rtl"><strong>DAS:</strong> مستقیم به مادربرد (ساده اما محدود).</p></li>
  <li><p dir="rtl"><strong>SAN:</strong> باکس بزرگ با شبکه (برای HA و SQL عالی؛ تست با ATTO Disk Benchmark: IO Size ۶۴KB, File Size ۱GB).</p></li>
  <li><p dir="rtl"><strong>NAS:</strong> برای فایل/بک‌آپ (نه SQL – latency بالا).</p></li>
  <li><p dir="rtl"><strong>نکته اضافی:</strong> برای SQL، SAN با fibre channel اولویت بدید.</p></li>
</ul>

<ul dir="rtl">
  <li><strong>مدیریت Data/Log:</strong></li>
  <li><p dir="rtl">Data می‌تونه چند NDF داشته باشه، اما Log فقط یک LDF. اگر RAM کافی باشه، نوشتن Asynchronousه (با checkpoint ذخیره می‌شه). اگر نه، از الگوریتم LRU (Least Recently Used) برای eviction استفاده می‌کنه. Log synchronousه – سریع‌ترین دیسک (SSD) رو بدید و رشدش رو پیش‌بینی کنید.</p></li>
  <li><p dir="rtl"><strong>مدیریت Log Size:</strong> در Simple، log رو دور می‌ریزه. در Full، نگه می‌داره تا log backup (هر ۳۰-۶۰ دقیقه). Reindex log زیاد می‌کنه. برای Always On، Full الزامیه – قطع ارتباط log primary رو بزرگ می‌کنه (تا ۴۸ ساعت پیش‌بینی کنید، چون backup هم کمک نمی‌کنه).</p></li>
  <li><p dir="rtl"><strong>نکته اضافی:</strong> از VLF (Virtual Log Files) کم برای log بهینه استفاده کنید – با shrink و regrow مدیریت کنید.</p></li>
  <li><p dir="rtl"><strong>SSD vs SAS:</strong> SSD ۵-۱۰ برابر سریع‌تر (برای random IO عالی).</p></li>
  <li><p dir="rtl"><strong>منبع:</strong> <a href="https://learn.microsoft.com/en-us/sql/relational-databases/policy-based-management/place-data-and-log-files-on-separate-drives?view=sql-server-ver16">Microsoft Docs - Storage Best Practices</a>.</p></li>
</ul>

</div>

---

<div dir="rtl" markdown="1">

## 🛡️ بخش ۱۰: RAID و سخت‌افزار سرور
<p dir="rtl">RAID برای reliability و speed ضروریه.</p>

<ul dir="rtl">
  <li><strong>انواع RAID:</strong></li>
  <li><p dir="rtl"><strong>RAID-0:</strong> سریع اما بدون redundancy (ریسک بالا).</p></li>
  <li><p dir="rtl"><strong>RAID-1:</strong> میرورینگ ساده (تحمل یک خرابی).</p></li>
  <li><p dir="rtl"><strong>RAID-10:</strong> بهترین برای SQL (خواندن/نوشتن سریع؛ ۵۰% فضا برای میرور – ایده‌آل برای OLTP).</p></li>
  <li><p dir="rtl"><strong>RAID-5:</strong> کم‌هزینه اما نوشتن کند (بدترین برای SQL؛ اگر هارد بسوزه، rebuild کند می‌شه).</p></li>
  <li><p dir="rtl"><strong>RAID-50/6/60:</strong> بهبودیافته (RAID-6 دو خرابی تحمل می‌کنه؛ RAID-60 برای ظرفیت بالا).</p></li>
  <li><p dir="rtl"><strong>دلیل RAID:</strong> Fault Tolerance (دیسک‌ها ۳-۵ سال عمر دارن) + Performance. RAID Controller با Battery Backup حیاتیه (جلوگیری از data loss در قطع برق – داده اول در cache controller نوشته می‌شه).</p></li>
  <li><p dir="rtl"><strong>نکته اضافی:</strong> برای SQL، RAID-10 رو برای data/log اولویت بدید و stripe size رو ۶۴KB تنظیم کنید.</p></li>
</ul>

<ul dir="rtl">
  <li><strong>سخت‌افزار سرور:</strong></li>
  <li><p dir="rtl">CPU/RAM/Disk کلیدی. Case: Rack Mountable با Dual Power Supply و Thermal Solution. Motherboard: Dual Socket, زیاد RAM slots, Dual NIC (۱۰G), VGA onboard (بدون sound). لوازم: Monitor, KVM, DVD-ROM.</p></li>
  <li><p dir="rtl"><strong>نرم‌افزار نصب:</strong> Win Server ۲۰۱۶+، WSUS برای آپدیت، آخرین CU برای SQL ۲۰۲۲، Antivirus (با exclusion برای SQL فایل‌ها).</p></li>
  <li><p dir="rtl"><strong>نکته:</strong> سه‌شنبه آخر ماه، پچ امنیتی Microsoft – بعد نصب ریست کنید، اما در maintenance window.</p></li>
</ul>

<ul dir="rtl">
  <li><strong>نکات ارتقا SQL:</strong></li>
  <li><p dir="rtl">یک‌طرفه – بک‌آپ روی نسخه قدیمی restore نمی‌شه. فقط script جدول‌ها و داده‌ها. ریسک بالا برای سرورهای باز به اینترنت، پورت‌های باز (مثل ۱۴۳۳)، file shares یا Database Mail. Antivirus رو configure کنید تا SQL رو اسکن نکنه.</p></li>
  <li><p dir="rtl"><strong>نکته اضافی:</strong> قبل ارتقا، compatibility level رو چک کنید و از Upgrade Advisor استفاده کنید.</p></li>
  <li><p dir="rtl"><strong>منبع:</strong> <a href="https://learn.microsoft.com/en-us/sql/sql-server/install/hardware-and-software-requirements-for-installing-sql-server-2022?view=sql-server-ver16">Microsoft Docs - Hardware Requirements</a> و <a href="https://learn.microsoft.com/en-us/troubleshoot/sql/database-engine/security/antivirus-and-sql-server">Antivirus with SQL</a>.</p></li>
</ul>

</div>

---

<div dir="rtl" markdown="1">

## ❓ چطور این جزوه رو استفاده کنید؟
<ul dir="rtl">
  <li><p dir="rtl">هر بخش رو به عنوان مرجع سریع بخونید – ایموجی‌ها برای ناوبری آسونن. 📌</p></li>
  <li><p dir="rtl">برای تمرین: queryها رو در SSMS تست کنید، سرور مجازی بسازید و سناریوها رو شبیه‌سازی کنید (مثل پر کردن log).</p></li>
  <li><p dir="rtl">اگر سؤالی داری، در issues ریپو بنویس یا از Microsoft Docs/فروم‌ها کمک بگیر.</p></li>
</ul>

<p dir="rtl"><strong>گیت‌هاب خانه ماست!</strong> این جزوه رو فورک کن، pull request بفرست برای بهبود، یا لینک پروژه‌های دیگه‌ات اضافه کن. نگارش رو کامل‌تر کردم – اگر نیاز به تغییر داری، بگو! <span style="color: #007bff;">⭐🚀</span></p>


  <li><p dir="rtl"><strong>بازگشت به صفحه اصلی:</strong> <a href="https://github.com/LaneZero/sql-server-dba-notes">صفحه اصلی</a>.</p></li>


</div>