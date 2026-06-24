# UI_UX_DASHBOARD_REPORTING_GUIDE.md

# UI/UX Dashboard & Reporting Guide

## SaaS Billboard Monitoring & Report Platform

Dokumen ini adalah panduan UI/UX utama untuk membangun dashboard, analytics, live view, device management, proof display, dan report center pada aplikasi SaaS Billboard Monitoring & Report Platform.

Dokumen ini wajib menjadi acuan untuk agent, developer, designer, dan reviewer agar UI tidak dibuat asal cantik tetapi salah fungsi, salah role, salah data, dan salah klaim.

Dashboard aplikasi ini harus menjadi pusat kendali data billboard, bukan sekadar halaman angka kendaraan.

---

# 1. Tujuan Dokumen

Tujuan dokumen ini:

* mengunci struktur dashboard berdasarkan role,
* mengunci layout utama dashboard,
* mengunci alur traffic analytics,
* mengunci alur live view / CCTV,
* mengunci alur report center,
* mengunci alur proof of display,
* mengunci prinsip client dashboard mobile-first,
* mengunci batasan UI berdasarkan permission,
* mencegah agent membuat UI yang keluar dari business rules,
* memastikan dashboard mendukung data quality, confidence score, audit log, dan anti-overclaim.

Dokumen ini adalah “rel UI”. Agent tidak boleh belok bebas seperti sopir angkot ngejar setoran.

---

# 2. Benchmark Dasar dari Manual ADX

Manual ADX Dashboard Traffic Counting dijadikan benchmark fitur dasar.

Fitur dasar dari benchmark:

* halaman login,
* dashboard awal setelah berhasil login,
* menu Analytics,
* rekap data traffic counting,
* tampilan data per hari / minggu / bulan,
* filter lokasi penempatan kamera,
* pengaturan periode pengukuran,
* download report PDF / CSV / Excel,
* menu Profile,
* menu Logout,
* menu Device,
* copy link streaming,
* live streaming app,
* pembatasan sesi live streaming,
* dokumentasi lokasi billboard / kamera.

Namun aplikasi kita tidak boleh meniru mentah-mentah standar lama seperti:

* credential ditulis di dokumen,
* password default sama dengan email,
* raw RTSP diekspos ke user,
* user harus copy RTSP ke VLC,
* tidak ada audit log live streaming,
* tidak ada data quality,
* tidak ada confidence score,
* tidak ada role-based dashboard kuat.

Manual ADX dipakai sebagai benchmark alur, bukan benchmark keamanan final.

Target kita:

```text
ADX Basic Dashboard
+
SaaS Multi-Tenant Architecture
+
Role-Based Dashboard
+
Secure Live View
+
Report Center
+
Proof of Display
+
Data Quality Score
+
Confidence Score
+
Audit Trail
+
Client Mobile Dashboard
```

---

# 3. Prinsip UI/UX Utama

## 3.1 Dashboard Harus Role-Based

Setiap role hanya boleh melihat menu dan aksi sesuai permission.

Role utama:

* Owner Platform SaaS,
* Admin Tenant,
* Owner Tenant,
* Sales Tenant,
* Teknisi Lapangan,
* Client / Brand.

UI tidak boleh hanya menyembunyikan tombol. Backend tetap wajib melakukan permission guard. Tapi dari sisi UI, menu yang tidak relevan juga tidak boleh muncul.

Contoh:

* Owner Tenant tidak boleh melihat tombol Create Campaign.
* Client tidak boleh melihat menu Users & Roles.
* Teknisi tidak boleh melihat budget, CPV, CPI, atau seluruh data client.
* Owner Platform SaaS tidak boleh diperlakukan seperti Admin Tenant biasa.

---

## 3.2 Dashboard Harus Mobile-First untuk Client

Client umumnya membuka dashboard dari HP.

Client dashboard harus:

* ringan,
* cepat dibaca,
* KPI jelas,
* tidak penuh tabel,
* tombol download report mudah ditemukan,
* proof display mudah dilihat,
* status campaign langsung terlihat.

Dashboard client bukan ruang cockpit pilot. Jangan dijejali semua grafik seperti panel listrik gedung tua.

---

## 3.3 Web Admin untuk Data Besar

Admin Tenant, Owner Platform SaaS, dan operator internal memakai web dashboard yang lebih lengkap.

Web admin boleh punya:

* tabel data,
* filter kompleks,
* bulk action,
* export,
* chart detail,
* device health,
* report workflow,
* approval flow.

---

## 3.4 Jangan Overclaim Exposure

UI harus menggunakan istilah aman:

* estimated exposure,
* potential exposure,
* opportunity to see,
* potensi iklan terlihat.

UI tidak boleh menggunakan istilah:

* pasti dilihat,
* pasti dibaca,
* pasti menghasilkan penjualan,
* guaranteed impression.

Sistem membaca kendaraan dan peluang exposure, bukan membaca mata manusia.

---

## 3.5 Data Harus Punya Status

Setiap data traffic harus dapat diberi status:

```text
Raw
Validated
Estimated
Adjusted
Anomaly
Missing
Low Quality
```

UI wajib menampilkan warning jika:

* data quality rendah,
* confidence score rendah,
* ada missing hours,
* ada downtime CCTV,
* ada estimasi data,
* ada anomaly.

Jangan menyembunyikan data jelek di balik desain cantik. Itu bukan UI/UX, itu make-up statistik.

---

# 4. Design Direction

## 4.1 Gaya Visual

Arah desain:

```text
Modern SaaS
Clean Dashboard
Premium Tech
Data-Driven
Mobile-Friendly
Professional OOH Report
```

Kesan yang dikejar:

* rapi,
* premium,
* terpercaya,
* bukan terlalu ramai,
* bukan terlalu gelap,
* bukan terlalu “template admin murahan”.

---

## 4.2 Palet Warna Rekomendasi

Palet utama:

```text
Deep Navy / Charcoal untuk header dan teks utama
Soft White / Warm White untuk background
Red Accent atau Gold Accent untuk highlight penting
Soft Blue untuk data / analytics
Green untuk status online / sehat
Yellow / Amber untuk warning
Red untuk critical / offline
Gray untuk neutral state
```

Catatan:

* Hindari putih mentah terlalu silau.
* Hindari terlalu banyak warna chart.
* Status warna harus konsisten.
* Warna merah hanya untuk alert, active menu, atau critical emphasis.

---

## 4.3 Tipografi

Rekomendasi:

```text
Font sans-serif modern
Heading tegas
Body text mudah dibaca
Angka KPI besar dan jelas
Label kecil tetapi tetap terbaca
```

Prinsip:

* angka KPI harus menonjol,
* satuan harus jelas,
* label jangan ambigu,
* chart label jangan terlalu kecil,
* tabel harus punya sticky header jika panjang.

---

# 5. Struktur Navigasi Global

## 5.1 Navigation Pattern

Gunakan pola:

```text
Sidebar kiri untuk desktop
Bottom navigation / drawer untuk mobile client
Topbar untuk tenant info, filter global, profile, notification
Breadcrumb untuk halaman detail
```

---

## 5.2 Topbar Global

Topbar desktop berisi:

```text
Tenant Name
Current Role
Active Plan
Date Range Filter
Location Filter
Campaign Filter
Notification Bell
Quick Export
User Profile
```

Untuk Client mobile:

```text
Campaign Selector
Date Range
Notification
Profile
```

---

# 6. Sidebar Berdasarkan Role

## 6.1 Owner Platform SaaS Sidebar

```text
Platform Dashboard
Tenant Management
Subscription / Tier
Platform Usage
System Health
Global Device Status
Billing
Audit Log
Feature Flags
Settings
Profile
Logout
```

Owner Platform SaaS fokus pada platform, tenant, tier, subscription, dan system health.

---

## 6.2 Admin Tenant Sidebar

```text
Dashboard
Traffic Analytics
Live View / CCTV
Billboard
Campaign
Client / Brand
Proof of Display
Report Center
Device & Camera
Alert & Maintenance
Users & Roles
Settings
Profile
Logout
```

Admin Tenant adalah role operasional paling lengkap di tenant sendiri.

---

## 6.3 Owner Tenant Sidebar

```text
Executive Dashboard
Traffic Summary
Campaign Summary
Billboard Performance
CCTV Health Summary
Report Center
Insight & Recommendation
Profile
Logout
```

Owner Tenant monitoring-only.

Tidak boleh ada menu:

```text
Users & Roles
Device Configuration
Create Campaign
Create Billboard
Approval Proof
Tenant Settings
Subscription Platform
```

---

## 6.4 Sales Tenant Sidebar

```text
Sales Dashboard
Billboard Inventory
Campaign Draft
Client Portfolio
Proposal Data
Report Access
Profile
Logout
```

Sales fokus bahan jualan, bukan konfigurasi teknis.

---

## 6.5 Teknisi Sidebar

```text
Technical Dashboard
Assigned Camera
Device Health
Maintenance Checklist
Upload Field Proof
Incident Report
Profile
Logout
```

Teknisi fokus pekerjaan lapangan.

---

## 6.6 Client Sidebar / Mobile Menu

```text
My Campaign
Traffic Performance
Proof of Display
Reports
Live View if Allowed
Profile
Logout
```

Client hanya melihat data campaign miliknya.

---

# 7. Login Page

## 7.1 Fungsi

Login page digunakan semua role.

Input:

```text
Email
Password
Remember Me
Sign In
Forgot Password
```

Opsional tahap lanjut:

```text
2FA Code
Tenant Subdomain Detection
Captcha after failed attempts
```

---

## 7.2 UI Rules

* Jangan tampilkan credential demo di halaman login.
* Jangan gunakan placeholder password asli.
* Jangan tampilkan pesan error terlalu detail.
* Setelah login, redirect berdasarkan role.
* Session harus punya timeout.
* Failed login harus dicatat.
* Login berhasil harus dicatat di audit log.

---

## 7.3 Redirect Setelah Login

```text
Owner Platform SaaS -> /platform/dashboard
Admin Tenant        -> /app/dashboard
Owner Tenant        -> /owner/dashboard
Sales Tenant        -> /sales/dashboard
Teknisi             -> /technical/dashboard
Client              -> /client/dashboard
```

---

# 8. Dashboard Home — Admin Tenant

Admin Tenant dashboard adalah pusat operasional tenant.

## 8.1 Tujuan

Memberikan ringkasan cepat tentang:

* traffic hari ini,
* traffic bulan ini,
* campaign aktif,
* billboard aktif,
* CCTV online / offline,
* proof pending,
* report pending,
* data quality,
* alert maintenance.

---

## 8.2 Layout Desktop

```text
┌──────────────────────────────────────────────────────────────┐
│ Topbar: Tenant | Date Range | Location | Campaign | Profile   │
├──────────────────────────────────────────────────────────────┤
│ KPI Cards: Vehicle | Exposure | Campaign | CCTV | Quality     │
├───────────────────────────────┬──────────────────────────────┤
│ Traffic Trend Chart           │ Vehicle Composition Donut     │
├───────────────────────────────┴──────────────────────────────┤
│ Hourly Heatmap                                                │
├───────────────────────────────┬──────────────────────────────┤
│ Top Billboard Performance     │ CCTV / Device Health          │
├───────────────────────────────┴──────────────────────────────┤
│ Alert | Proof Pending | Report Status | Latest Activity        │
└──────────────────────────────────────────────────────────────┘
```

---

## 8.3 KPI Cards

KPI utama:

```text
Total Vehicle Today
Total Vehicle This Month
Estimated Exposure This Month
Active Campaign
Active Billboard
CCTV Online
CCTV Offline
Data Quality Score
Pending Proof Approval
Report Waiting Review
```

Breakdown vehicle:

```text
Car
Motorcycle
Bus
Truck
Total Vehicle
```

Insight ringkas:

```text
Peak Hour
Average Speed
Congestion Level
Dominant Vehicle Type
Highest Traffic Location
Lowest Data Quality Location
```

---

## 8.4 Admin Tenant Quick Action

Admin Tenant boleh melihat tombol:

```text
Add Billboard
Create Campaign
Add Client
Register Camera
Generate Report
Upload Proof
Assign Technician
View Alert
```

Tombol harus mengikuti permission, bukan hanya role label.

---

# 9. Owner Tenant Executive Dashboard

## 9.1 Tujuan

Owner Tenant hanya membutuhkan ringkasan bisnis dan monitoring.

Dashboard harus bersih, eksekutif, ringkas, dan tidak penuh tombol operasional.

---

## 9.2 KPI Cards

```text
Total Billboard
Active Billboard
Active Campaign
Active Client
CCTV Online
CCTV Offline
Estimated Exposure This Month
Highest Traffic Billboard
Campaign Ending Soon
Latest Report
Performance Trend
```

---

## 9.3 Executive Insight

```text
Best Performing Location
Lowest Performing Location
Highest Traffic Growth
Best CPI Location
Camera With Risk
Campaign Need Attention
Report Ready to Review
```

---

## 9.4 Larangan UI

Owner Tenant tidak boleh melihat tombol:

```text
Create
Edit
Delete
Approve
Manage User
Manage Role
Manage Camera
Change Subscription
Change Counting Rule
```

Kalau Owner Tenant bisa menekan tombol operasional, berarti UI salah dan permission sedang bocor halus.

---

# 10. Client Dashboard — Mobile First

## 10.1 Tujuan

Client dashboard menjawab pertanyaan cepat:

* campaign saya aktif atau tidak,
* billboard saya di mana,
* traffic berapa,
* kendaraan dominan apa,
* proof tayang ada atau tidak,
* report bisa diunduh atau tidak.

---

## 10.2 Mobile Layout

```text
┌──────────────────────────────┐
│ Campaign Name                │
│ Active | 12 Days Remaining   │
├──────────────────────────────┤
│ Total Vehicle Today          │
│ 117,303                      │
├──────────────────────────────┤
│ Estimated Exposure Month     │
│ 4,875,559                    │
├──────────────────────────────┤
│ Vehicle Composition          │
│ Car | Motorcycle | Bus | Truck│
├──────────────────────────────┤
│ Traffic Trend                │
├──────────────────────────────┤
│ Proof Display Approved       │
├──────────────────────────────┤
│ Download Monthly Report      │
└──────────────────────────────┘
```

---

## 10.3 Client KPI

```text
Campaign Status
Days Remaining
Total Vehicle Today
Total Vehicle This Week
Total Vehicle This Month
Estimated Exposure
Peak Hour
Dominant Vehicle Type
Proof Display Status
Latest Report
```

---

## 10.4 Client Access Rules

Client boleh melihat:

```text
Campaign miliknya
Billboard yang terkait campaign miliknya
Approved proof display
Report campaign miliknya
Traffic summary campaign miliknya
Live view jika diizinkan
```

Client tidak boleh melihat:

```text
Campaign client lain
Data tenant penuh
User tenant
Role tenant
Raw RTSP
Camera config
Billing tenant
Data quality internal yang terlalu teknis, kecuali summary
```

---

# 11. Traffic Analytics Page

Traffic Analytics adalah versi naik kelas dari menu Analytics pada benchmark ADX.

## 11.1 Tujuan

Menampilkan data traffic secara periodik dan bisa difilter.

Fungsi minimum:

```text
Rekap traffic counting
Tampilan Day / Week / Month / Custom
Filter lokasi kamera
Filter periode
Filter campaign
Export PDF / CSV / Excel
Chart traffic
Table traffic
```

Fungsi tambahan aplikasi kita:

```text
Hourly heatmap
Vehicle composition
Vehicle class trend
Weekday vs weekend comparison
CPV / CPI comparison
Cumulative exposure
Data quality score
Confidence score
Anomaly flag
Raw / Validated / Estimated mode
```

---

## 11.2 Filter Bar

```text
Period Type: Day | Week | Month | Custom
Date Range
Location
Campaign
Client
Vehicle Class
Direction
Data Status: Raw | Validated | Estimated
Quality Status: All | Good | Warning | Low
Button: Apply
Button: Reset
Button: Export
```

---

## 11.3 KPI Rekap

```text
Total Vehicle
Total Car
Total Motorcycle
Total Bus
Total Truck
Estimated Exposure
Average Traffic / Day
Peak Hour
Peak Day
Average Speed
Congestion Level
Data Quality Score
Average Confidence
Valid Hours
Missing Hours
```

---

## 11.4 Chart Area

Wajib ada:

```text
Daily Traffic Line Chart
Hourly Traffic Heatmap
Vehicle Composition Donut
Vehicle Class Trend
Weekday vs Weekend Bar Chart
CPV / CPI Comparison
Cumulative Exposure Chart
```

---

## 11.5 Table Area

Tabel utama:

```text
Date
Hour
Location
Camera
Direction
Car
Motorcycle
Bus
Truck
Total Vehicle
Estimated Exposure
Average Confidence
Data Quality
Anomaly Flag
Data Status
```

Rules tabel:

* Gunakan pagination.
* Gunakan sticky header.
* Jangan tampilkan 31 hari x 24 jam x banyak lokasi dalam satu layar penuh tanpa struktur.
* Data besar masuk appendix/export Excel/CSV.
* PDF fokus executive summary dan insight.

Tabel terlalu padat itu bukan transparansi. Itu jebakan mata.

---

# 12. Location Detail Dashboard

## 12.1 Tujuan

Menampilkan performa satu billboard / satu titik lokasi.

---

## 12.2 Header Lokasi

```text
Location Name
Billboard Code
Address
City
Road Type
Direction Captured
Campaign Active
Camera Status
Data Quality Status
```

---

## 12.3 Visual Lokasi

Komponen:

```text
Map Preview
Billboard Photo
CCTV Snapshot
Campaign Creative
Camera Angle Preview
Counting Line Preview
Exposure Zone Preview
```

---

## 12.4 KPI Lokasi

```text
Total Vehicle Today
Total Vehicle This Month
Estimated Exposure
Average Daily Traffic
CPV
CPI
Camera Uptime
Data Quality Score
Average AI Confidence
Peak Day
Peak Hour
Dominant Vehicle
Valid Hours
Missing Data Hours
Anomaly Count
```

---

## 12.5 Tab Lokasi

```text
Overview
Traffic
Live View
Proof Display
Report
Data Quality
Maintenance
```

---

# 13. Live View / CCTV Dashboard

## 13.1 Prinsip

Live View wajib lebih modern dan aman dari benchmark lama.

Manual lama memakai pola copy RTSP dan buka di VLC. Di aplikasi kita, user tidak boleh melihat raw RTSP.

Live View wajib memakai:

```text
Secure web player
Streaming proxy
Signed token
Session expiry
Role permission
Audit log
Viewer tracking
```

---

## 13.2 Live View Page

Informasi utama:

```text
Camera Name
Location
Billboard
Campaign Related
Status Online / Offline
Last Heartbeat
Stream Quality
Resolution
FPS
Latency
Current Viewer
Session Expiry Timer
```

Action:

```text
Start Live View
Stop Session
Capture Snapshot
Report Issue
Toggle Detection Overlay
Toggle Counting Line
Toggle Exposure Zone
```

---

## 13.3 Live View Layout

```text
┌──────────────────────────────────────────────────────────────┐
│ Live View: JPO TangCity Camera 01                            │
├──────────────────────────────────────────────────────────────┤
│ Video Player                                                  │
│ Timestamp Overlay | Detection Overlay | Counting Line         │
├──────────────────────────────────────────────────────────────┤
│ Status: Online | Quality: Good | Uptime: 98.7% | Token: 09:58 │
├──────────────────────────────────────────────────────────────┤
│ Vehicle Now: Car 14 | Motorcycle 52 | Bus 1 | Truck 3         │
└──────────────────────────────────────────────────────────────┘
```

---

## 13.4 Live View Rules

* Jangan expose raw RTSP.
* Jangan simpan RTSP plain text.
* Jangan izinkan semua role membuka live stream.
* Client hanya boleh live view jika campaign dan permission mengizinkan.
* Akses live stream harus masuk audit log.
* Session harus otomatis expire.
* Satu kamera bisa dibatasi satu viewer aktif jika paket/aturan mengharuskan.
* Admin Tenant bisa melihat siapa sedang membuka stream.
* Teknisi bisa membuka stream untuk troubleshooting kamera assigned.
* Owner Tenant hanya summary, live view opsional sesuai permission.
* Owner Platform SaaS hanya support/audit, bukan nonton bebas semua tenant.

---

# 14. Device & Camera Management UI

## 14.1 Tujuan

Mengelola kamera, status, health, heartbeat, dan relasi ke billboard/campaign.

---

## 14.2 Device Table

Kolom:

```text
Camera ID
Alias
Billboard
Location
Tenant
Status
Last Heartbeat
Stream Status
AI Status
Storage Status
FPS
Resolution
Latency
Health Score
Action
```

---

## 14.3 Status Badge

```text
Online
Offline
Unstable
No Heartbeat
Low Quality
AI Error
Maintenance
Disabled
```

---

## 14.4 Action Berdasarkan Role

Admin Tenant:

```text
View Detail
Open Live View
Edit Camera
Assign Billboard
Set Counting Rule
View Health Log
Create Maintenance Ticket
Disable Camera
```

Teknisi:

```text
View Assigned
Open Live View for Maintenance
Check Camera
Upload Maintenance Photo
Report Issue
Update Field Status
```

Owner Tenant:

```text
View Summary
View Health Status
```

Client:

```text
View Status Only
Open Live View if Allowed
```

---

# 15. Proof of Display UI

## 15.1 Tujuan

Membuktikan billboard benar-benar tayang.

Karena kamera utama sering menghadap jalan, proof display harus memiliki modul tersendiri.

---

## 15.2 Proof List

Kolom:

```text
Proof ID
Campaign
Client
Billboard
Uploaded By
Uploaded At
GPS
Timestamp
Status
Approved By
Client Visibility
Action
```

---

## 15.3 Proof Detail

Isi:

```text
Campaign
Billboard
Photo Before
Photo After
Technician Name
GPS Location
Timestamp
Checklist
Admin Notes
Approval Status
Client Visibility
Audit Trail
```

---

## 15.4 Status Proof

```text
Waiting Upload
Uploaded
Waiting Approval
Approved
Rejected
Visible to Client
Archived
```

---

## 15.5 Role Rules

Teknisi:

```text
Upload field proof
Add notes
Update checklist
```

Admin Tenant:

```text
Review proof
Approve proof
Reject proof
Set client visibility
```

Owner Tenant:

```text
View summary
View approved/rejected status
```

Client:

```text
View approved proof only
Download proof if allowed
```

---

# 16. Report Center UI

## 16.1 Tujuan

Report Center bukan sekadar tombol download. Report Center adalah pusat produksi, review, approval, versioning, dan export report.

---

## 16.2 Jenis Report

```text
Daily Report
Weekly Report
Monthly Report
Campaign Report
Billboard Report
Client Report
Custom Period Report
```

---

## 16.3 Format Export

```text
PDF Executive Report
Excel Raw Data
CSV Hourly Data
JSON/API Export if Needed
PNG Chart Export if Needed
```

---

## 16.4 Report Table

Kolom:

```text
Report Name
Client
Campaign
Period
Location
Generated By
Generated At
Status
Version
PDF
Excel
CSV
Action
```

---

## 16.5 Report Status

```text
Draft
Generated
Waiting Review
Approved
Sent to Client
Failed
Archived
```

---

## 16.6 Report Action

Admin Tenant:

```text
Generate Report
Preview Report
Approve Report
Send to Client
Download PDF
Download Excel
Download CSV
Regenerate
Archive
```

Owner Tenant:

```text
View Report
Download Report if Allowed
```

Client:

```text
View Final Report
Download Final Report
```

Owner Platform SaaS:

```text
View Report Usage
View Export Usage
Support/Audit Only
```

---

# 17. Report PDF UX Structure

Report PDF harus minimal setara benchmark ADX, tetapi lebih transparan dan audit-able.

Struktur ideal:

```text
1. Cover
2. Executive Summary
3. Campaign Overview
4. Measurement Methodology
5. Network Map
6. Overall KPI Recap
7. Location Summary
8. Location Traffic Analysis
9. Hourly Heatmap
10. Vehicle Composition
11. Cost Efficiency Analysis
12. Data Quality & Integrity
13. Proof of Display
14. Cross-location Comparison
15. Recommendation
16. Raw Data Appendix
17. Disclaimer & Formula Notes
18. Closing Page
```

---

## 17.1 Cover Report

Isi:

```text
Client Name
Campaign Name
Brand
City / Area
Period
Report Type
Platform Logo
Generated Date
Report Version
```

---

## 17.2 Executive Summary

Isi:

```text
Total Locations
Total Vehicle Traffic
Total Estimated Exposure
Average Daily Traffic
Best CPV Location
Best CPI Location
Highest Traffic Location
Key Campaign Insight
```

---

## 17.3 Measurement Methodology

Isi:

```text
Data Source: CVC / AI CCTV
Vehicle Classes: Car, Motorcycle, Bus, Truck
Counting Line
Direction Rule
Exposure Zone
Impression Multiplier
Data Quality Policy
Confidence Policy
Exclusion Rule
Downtime Handling
```

---

## 17.4 Location KPI

Minimal:

```text
Estimated OOH Budget
Total Vehicle Traffic
Total Estimated Exposure
Average Traffic / Day
CPV
CPI
Vehicle Composition
```

Tambahan wajib aplikasi kita:

```text
Camera Uptime
Data Quality Score
Average Confidence Score
Peak Day
Peak Hour
Dominant Vehicle Type
Valid Hours / Expected Hours
Missing Data Hours
Anomaly Count
```

---

## 17.5 Raw Data Appendix

Isi:

```text
Hourly Table
Daily Table
Vehicle Class Table
Export CSV/Excel Reference
API Trace ID
Generated Timestamp
Formula Version
Methodology Version
```

Tabel besar masuk appendix, bukan halaman utama. Halaman utama harus bicara insight, bukan membuat client tenggelam di angka.

---

# 18. Alert & Maintenance UI

## 18.1 Alert List

Kolom:

```text
Alert ID
Severity
Camera
Billboard
Location
Issue Type
Detected At
Assigned To
Status
Action
```

---

## 18.2 Alert Type

```text
CCTV Offline
Stream Down
No Traffic Data
Low Data Quality
High Missing Hours
Report Failed
Proof Not Uploaded
Device No Heartbeat
Storage Warning
AI Processing Error
```

---

## 18.3 Severity

```text
Critical
High
Medium
Low
Info
```

---

## 18.4 Maintenance Ticket

Isi:

```text
Ticket ID
Camera
Location
Issue
Assigned Technician
Status
Created At
Resolved At
Evidence Photo
Technician Notes
Admin Notes
```

---

# 19. Data Quality UI

## 19.1 Tujuan

Data Quality UI memastikan angka traffic tidak menjadi angka sulap.

---

## 19.2 Data Quality Card

```text
Camera Uptime
Expected Hours
Valid Captured Hours
Missing Hours
Average AI Confidence
Frame Quality
Occlusion Risk
Weather/Night Risk
Anomaly Count
Data Quality Score
```

---

## 19.3 Data Quality Status

```text
Good
Warning
Low
Critical
Insufficient Data
```

---

## 19.4 Data Quality Warning

Contoh warning:

```text
Data quality is below recommended threshold.
Some hours are missing and have been marked as estimated.
Traffic data for this period should be reviewed before final report approval.
```

---

# 20. Audit UI Behavior

UI wajib mendukung audit trail.

Event yang harus tercatat:

```text
Login
Logout
Failed Login
Open Live Stream
Stop Live Stream
Generate Report
Download PDF
Download Excel
Download CSV
Upload Proof
Approve Proof
Reject Proof
Create Campaign
Edit Campaign
Delete Campaign
Create Billboard
Edit Billboard
Update Camera
Update Counting Rule
Change User Role
```

Audit log minimal menyimpan:

```text
User
Role
Tenant
Action
Target Object
Before Value
After Value
IP Address
User Agent
Timestamp
Status
```

---

# 21. Profile & Account UI

## 21.1 Profile Page

Isi:

```text
Photo
Name
Email
Phone
Role
Tenant
Password Change
Session History
Active Login Device
Logout All Devices
```

---

## 21.2 Security UX

Rules:

* Password harus kuat.
* Tidak boleh default password sama dengan email.
* Profile update harus masuk audit log.
* Password change harus meminta current password.
* Failed password change harus dicatat.
* Session history harus bisa dilihat user.
* Logout all devices tersedia untuk keamanan.

---

# 22. Responsive Rules

## 22.1 Desktop

Desktop digunakan untuk:

```text
Admin Tenant
Owner Platform SaaS
Owner Tenant
Sales
Teknisi office mode
```

Desktop layout:

* sidebar fixed,
* topbar sticky,
* card grid 4 kolom,
* chart 2 kolom,
* table full width,
* filter horizontal.

---

## 22.2 Tablet

Tablet layout:

* sidebar collapsible,
* card grid 2 kolom,
* chart 1–2 kolom,
* table horizontal scroll.

---

## 22.3 Mobile

Mobile terutama untuk Client dan Teknisi.

Mobile layout:

* bottom navigation atau drawer,
* KPI cards 1 kolom,
* chart sederhana,
* proof display mudah dibuka,
* tombol download report jelas,
* tabel detail disembunyikan di halaman khusus,
* live view full width.

---

# 23. Empty State

Setiap halaman harus punya empty state.

Contoh:

```text
No campaign found.
No traffic data for selected period.
No report generated yet.
No proof display uploaded.
No camera assigned.
No alert at this time.
```

Empty state harus memberi arahan aksi sesuai role.

Contoh Admin Tenant:

```text
No camera registered yet. Add your first camera to start traffic monitoring.
```

Contoh Client:

```text
Your campaign report is not available yet. Please check again later.
```

---

# 24. Loading State

Gunakan loading state:

```text
Skeleton card
Chart loading shimmer
Table loading state
Button disabled while processing
Report generation progress
Live stream connecting state
```

Jangan biarkan user menekan tombol Generate Report berkali-kali saat proses masih berjalan.

---

# 25. Error State

Error harus jelas, tapi tidak membocorkan detail teknis.

Contoh:

```text
Failed to load traffic data.
Please try again or contact support if the issue continues.
```

Untuk Admin Tenant boleh lebih teknis sedikit:

```text
Camera heartbeat not received for 15 minutes.
```

Untuk Client cukup:

```text
Live view is temporarily unavailable.
```

---

# 26. Frontend Route Structure

## 26.1 Auth

```text
/auth/login
/auth/forgot-password
/auth/reset-password
```

---

## 26.2 Tenant App

```text
/app/dashboard
/app/analytics
/app/analytics/location/:locationId
/app/live-view
/app/live-view/:cameraId
/app/billboards
/app/billboards/:billboardId
/app/campaigns
/app/campaigns/:campaignId
/app/clients
/app/clients/:clientId
/app/proof
/app/proof/:proofId
/app/reports
/app/reports/:reportId
/app/devices
/app/devices/:cameraId
/app/maintenance
/app/alerts
/app/users
/app/settings
/app/profile
```

---

## 26.3 Client App

```text
/client/dashboard
/client/campaign/:campaignId
/client/traffic
/client/proof
/client/reports
/client/reports/:reportId
/client/live-view/:cameraId
/client/profile
```

---

## 26.4 Owner Platform App

```text
/platform/dashboard
/platform/tenants
/platform/tenants/:tenantId
/platform/plans
/platform/subscriptions
/platform/system-health
/platform/global-devices
/platform/audit-log
/platform/feature-flags
/platform/settings
/platform/profile
```

---

## 26.5 Owner Tenant App

```text
/owner/dashboard
/owner/traffic-summary
/owner/campaign-summary
/owner/billboard-performance
/owner/cctv-health
/owner/reports
/owner/insights
/owner/profile
```

---

## 26.6 Technician App

```text
/technical/dashboard
/technical/assigned-camera
/technical/device-health
/technical/maintenance-checklist
/technical/upload-proof
/technical/incident-report
/technical/profile
```

---

# 27. MVP UI Phase

## Phase 1 — UI Foundation

* Login page.
* Role-based redirect.
* Base layout.
* Sidebar per role.
* Topbar.
* Profile.
* Logout.
* Empty state.
* Loading state.
* Error state.

---

## Phase 2 — Dashboard Core

* Admin Tenant dashboard.
* Owner Tenant dashboard read-only.
* Client mobile dashboard.
* Owner Platform dashboard basic.
* Teknisi dashboard basic.

---

## Phase 3 — Traffic Analytics

* KPI traffic.
* Day / Week / Month / Custom filter.
* Location filter.
* Campaign filter.
* Vehicle breakdown.
* Daily trend chart.
* Vehicle composition chart.
* Export PDF / CSV / Excel button.

---

## Phase 4 — Live View & Device UI

* Device list.
* Camera health.
* Camera detail.
* Secure live view page.
* Session expiry timer.
* Snapshot action.
* Report issue action.
* Audit event trigger.

---

## Phase 5 — Report Center

* Report list.
* Generate report.
* Preview report.
* Download PDF.
* Download Excel.
* Download CSV.
* Report status.
* Report history.

---

## Phase 6 — Proof Display

* Proof list.
* Upload proof.
* Proof detail.
* Approval UI.
* Client proof view.

---

## Phase 7 — Data Quality UI

* Data quality card.
* Confidence score.
* Missing hours.
* Anomaly flag.
* Data status label.
* Methodology note.

---

## Phase 8 — Polish & Hardening

* Mobile optimization.
* Permission-based UI hiding.
* Accessibility pass.
* Table optimization.
* Chart optimization.
* Report print styling.
* Final QA.

---

# 28. UI Acceptance Criteria

Sebuah halaman dianggap lolos jika:

* sesuai role,
* tidak menampilkan menu terlarang,
* tidak menampilkan tombol terlarang,
* semua data tenant terfilter,
* state loading jelas,
* state error jelas,
* empty state jelas,
* responsive,
* tidak expose raw RTSP,
* export button sesuai permission,
* action penting memicu audit log,
* data estimated diberi label,
* data quality rendah diberi warning,
* Client tidak bisa melihat data client lain,
* Owner Tenant tidak punya tombol operasional.

---

# 29. Larangan untuk Agent / Developer

Agent tidak boleh:

* membuat satu dashboard untuk semua role,
* membuat Owner Tenant bisa create/edit/delete,
* membuat Client bisa melihat semua campaign,
* membuat Teknisi melihat data budget dan CPI,
* menampilkan raw RTSP di frontend,
* membuat live streaming tanpa session expiry,
* membuat export report tanpa audit log,
* membuat table hourly raksasa sebagai halaman utama,
* menampilkan impression sebagai pasti dilihat,
* menyembunyikan data quality rendah,
* menghilangkan confidence score dari UI analytics,
* membuat route frontend tanpa permission guard,
* membuat menu sidebar statis tanpa role check,
* membuat chart dummy tanpa label simulation,
* membuat UI cantik tapi data tidak bisa dijelaskan.

---

# 30. Kesimpulan

Dashboard aplikasi ini harus menjadi gabungan antara:

```text
Traffic Counting Dashboard
+
Billboard Campaign Dashboard
+
CCTV Monitoring Dashboard
+
Client Report Dashboard
+
SaaS Admin Dashboard
+
Data Quality Dashboard
```

Benchmark ADX memberi kita baseline:

```text
Login
Dashboard
Analytics
Filter Day/Week/Month
Location Filter
Date Range
Export PDF/CSV/Excel
Device
Live Streaming
Profile
Logout
```

Aplikasi kita harus naik kelas menjadi:

```text
Role-Based SaaS Dashboard
Secure Live View
Report Center
Proof of Display
Data Quality
Confidence Score
Audit Trail
Client Mobile Dashboard
Executive Owner Dashboard
```

Dashboard harus cantik, tetapi yang lebih penting: dashboard harus jujur, aman, dan bisa dipertanggungjawabkan.

Kalau angka ditanya, dashboard harus bisa menjawab. Kalau client minta bukti, dashboard harus punya. Kalau role mencoba melewati batas, dashboard harus mengunci pintu.
