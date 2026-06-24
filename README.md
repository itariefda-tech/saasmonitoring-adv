# README.md

# SaaS Billboard Monitoring & Report Platform

Aplikasi ini adalah platform monitoring dan reporting untuk perusahaan billboard / outdoor advertising. Sistem menggunakan CCTV atau Computer Vision Camera untuk membaca kondisi lalu lintas di sekitar titik billboard, mengolah data kendaraan menjadi analytics, lalu menyajikannya dalam dashboard, KPI card, proof of display, dan report harian, mingguan, serta bulanan.

Produk ini dirancang sebagai **multi-tenant SaaS**, bukan aplikasi custom satu perusahaan. Satu platform pusat dapat dipakai oleh banyak perusahaan billboard, dengan data yang wajib terpisah aman antar-tenant.

Tujuan akhirnya sederhana tetapi besar:

> Membantu perusahaan billboard menjual titik iklan dengan bukti data, bukan hanya klaim “lokasi ramai dan strategis”.

Billboard bukan lagi hanya papan besar di pinggir jalan. Billboard harus menjadi aset media yang dapat dimonitor, diukur, dilaporkan, dan dipertanggungjawabkan.

---

# 1. Latar Belakang Produk

Perusahaan billboard sering menjual titik iklan dengan narasi:

* lokasi strategis,
* jalan ramai,
* dekat pusat bisnis,
* dekat mall,
* dekat perkantoran,
* dekat pemukiman,
* potensi dilihat banyak orang.

Masalahnya, klaim seperti itu sering tidak punya data operasional yang kuat.

Aplikasi ini dibuat agar perusahaan billboard dapat menunjukkan data seperti:

* jumlah kendaraan yang lewat,
* jenis kendaraan yang dominan,
* jam paling ramai,
* hari paling ramai,
* jam macet,
* rata-rata kecepatan kendaraan,
* congestion level,
* estimated exposure,
* proof billboard sedang tayang,
* performa campaign,
* report siap kirim ke client.

Dengan sistem ini, perusahaan billboard dapat berkata:

> “Ini bukan cuma lokasi ramai. Ini data traffic-nya, ini komposisi kendaraan, ini jam puncaknya, ini report-nya, ini bukti tayangnya.”

Itu baru jualan yang punya tulang punggung. Bukan sekadar brosur manis beraroma Excel.

---

# 2. Benchmark Fitur dari Manual ADX

Manual ADX Traffic Counting menjadi benchmark awal untuk alur dashboard. Dari manual tersebut, fitur dasar yang wajib dijadikan acuan adalah:

* login dashboard,
* tampilan awal setelah login,
* menu Analytics,
* rekap data traffic counting,
* filter tampilan per hari / minggu / bulan,
* filter lokasi kamera,
* filter periode pengukuran,
* export report PDF / CSV / Excel,
* menu Profile,
* menu Logout,
* menu Device,
* akses live streaming traffic counting,
* pembatasan sesi live streaming,
* dokumentasi lokasi kamera / billboard.

Namun, aplikasi ini tidak boleh meniru mentah-mentah standar keamanan lama seperti:

* credential ditulis di dokumen,
* password default sama dengan email,
* raw RTSP link diekspos langsung,
* user diminta copy link streaming ke VLC,
* akses live streaming tanpa audit yang kuat.

Manual ADX dipakai sebagai **benchmark fitur**, bukan benchmark keamanan final.

Aplikasi kita harus naik kelas:

```text
ADX Basic Dashboard
+
SaaS Multi-Tenant
+
Role-Based Access Control
+
Secure Live Streaming
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

# 3. Target Produk

Platform ini harus mampu menghasilkan:

* dashboard realtime,
* dashboard executive,
* dashboard client mobile-first,
* monitoring CCTV,
* analytics traffic,
* vehicle counting,
* vehicle classification,
* campaign monitoring,
* proof of display,
* report PDF,
* export Excel,
* export CSV,
* report history,
* report approval,
* data quality monitoring,
* audit log.

Target aplikasi bukan cuma menampilkan angka.

Target aplikasi adalah membuat angka itu:

* jelas sumbernya,
* jelas rumusnya,
* jelas validitasnya,
* jelas siapa yang boleh melihat,
* jelas siapa yang membuat report,
* jelas kapan data digenerate,
* jelas apakah data raw, validated, atau estimated.

Kalau angka tidak bisa dijelaskan, angka itu bukan insight. Itu cuma hiasan dashboard.

---

# 4. Prinsip Utama Produk

Prinsip yang wajib dipegang:

* Multi-tenant sejak awal.
* Tenant isolation wajib keras.
* Role dan permission harus jelas.
* Client hanya melihat campaign miliknya.
* Owner Tenant hanya monitoring.
* Admin Tenant mengelola operasional tenant.
* Owner Platform SaaS mengelola tenant, tier, subscription, dan platform.
* CCTV dan aplikasi dianggap satu ekosistem.
* Dashboard client wajib mobile-first.
* Web admin dipakai untuk pengelolaan data besar.
* Report harus mudah dipahami client.
* Report harus punya metodologi.
* Jangan overclaim data exposure.
* Semua aksi penting wajib masuk audit log.
* Jangan menyimpan credential mentah.
* Jangan expose raw RTSP ke client.
* Jangan membangun fitur cantik tetapi permission bocor.

Prinsip galak:

> Dashboard boleh cantik, tapi kalau role kacau, itu bukan SaaS. Itu pasar malam digital.

---

# 5. Role Utama

## 5.1 Owner Platform SaaS

Owner Platform SaaS adalah pemilik aplikasi / platform SaaS secara keseluruhan.

Role ini menggantikan istilah lama **Super Admin Platform**.

Owner Platform SaaS bukan bagian dari tenant billboard tertentu. Ia adalah pemilik dan pengendali utama platform.

Tugas utama:

* mengelola semua tenant / perusahaan billboard,
* membuat dan mengatur paket berlangganan / tier,
* mengatur limit paket,
* mengatur jumlah billboard per paket,
* mengatur jumlah CCTV per paket,
* mengatur jumlah user per paket,
* mengatur fitur report per paket,
* mengatur fitur analytics per paket,
* melakukan approval tenant baru jika diperlukan,
* memonitor seluruh sistem,
* melihat status subscription tenant,
* mengontrol aktif / nonaktif tenant,
* mengatur billing dan subscription,
* melihat audit log platform,
* mengatur konfigurasi global aplikasi,
* memonitor kesehatan sistem global,
* mengontrol feature flags / fitur per tier.

Owner Platform SaaS adalah level tertinggi dalam platform.

Owner Platform SaaS tidak boleh diperlakukan sebagai Admin Tenant biasa.

---

## 5.2 Admin Tenant

Admin Tenant adalah admin utama di perusahaan billboard yang memakai platform.

Role ini adalah role paling kuat di dalam tenant.

Tugas utama:

* mengelola data perusahaan billboard miliknya,
* mengelola user internal tenant,
* mengelola billboard,
* mengelola sisi / muka billboard,
* mengelola CCTV,
* mengelola client / brand penyewa billboard,
* membuat dan mengelola campaign,
* mengatur akses dashboard client,
* upload dan approval proof of display,
* melihat traffic analytics,
* generate dan download report,
* mengelola teknisi,
* melihat alert dan maintenance,
* mengatur konfigurasi operasional tenant.

Admin Tenant hanya berkuasa di tenant sendiri.

Admin Tenant tidak boleh:

* melihat data tenant lain,
* mengubah data tenant lain,
* mengatur paket SaaS global,
* mengubah subscription global platform,
* mengubah konfigurasi global platform.

Admin Tenant adalah “raja operasional” di kerajaannya sendiri, tapi tetap tidak boleh lompat pagar ke kerajaan tenant lain.

---

## 5.3 Owner Tenant

Owner Tenant adalah pemilik atau pimpinan perusahaan billboard yang menggunakan platform.

Role ini bukan admin operasional.

Owner Tenant hanya membutuhkan akses:

* monitoring,
* KPI card,
* rekap,
* insight bisnis,
* report,
* performa traffic,
* status campaign,
* status CCTV.

Owner Tenant boleh melihat:

* dashboard ringkasan perusahaan,
* KPI utama,
* total billboard,
* billboard aktif,
* campaign aktif,
* client aktif,
* CCTV online / offline,
* performa traffic ringkas,
* report harian / mingguan / bulanan,
* campaign hampir selesai,
* billboard traffic tertinggi,
* estimated exposure,
* insight bisnis,
* report download jika diizinkan.

Owner Tenant tidak boleh:

* membuat billboard,
* mengubah billboard,
* menghapus billboard,
* membuat campaign,
* mengubah campaign,
* menghapus campaign,
* mengatur user,
* mengatur role,
* mengatur CCTV,
* approval proof display,
* mengubah data client,
* mengubah subscription,
* mengakses data tenant lain.

Owner Tenant adalah executive monitoring.

Bahasa sederhananya:

> Boleh melihat ruang komando, tapi tidak boleh menekan tombol nuklir.

---

## 5.4 Sales Tenant

Sales adalah user tenant yang fokus pada aktivitas penjualan billboard dan campaign.

Tugas:

* melihat data billboard yang tersedia,
* melihat performa ringkas billboard untuk bahan jualan,
* melihat data client miliknya,
* membuat draft campaign jika diberi izin,
* melihat report campaign miliknya,
* membantu client memahami performa campaign.

Sales tidak boleh punya akses penuh seperti Admin Tenant.

---

## 5.5 Teknisi Lapangan

Teknisi adalah user untuk pemasangan, pengecekan, dan maintenance perangkat lapangan.

Tugas:

* memasang CCTV,
* upload foto pemasangan,
* cek status kamera,
* update maintenance,
* melaporkan kendala lapangan,
* upload foto lokasi,
* mengisi checklist teknis,
* mencatat hasil perbaikan,
* upload proof lapangan jika diberi akses.

Teknisi tidak boleh mengakses data bisnis tenant secara luas.

---

## 5.6 Client / Brand

Client adalah pihak penyewa billboard yang ingin melihat performa campaign miliknya.

Client boleh melihat:

* campaign miliknya,
* KPI campaign,
* grafik traffic,
* proof display yang sudah approved,
* report harian,
* report mingguan,
* report bulanan,
* download report jika diizinkan,
* live view jika permission diaktifkan.

Client tidak boleh melihat:

* campaign client lain,
* data tenant penuh,
* billing tenant,
* konfigurasi CCTV,
* raw RTSP,
* user internal tenant,
* data report client lain.

Client dashboard wajib mobile-first.

---

# 6. Modul Utama Aplikasi

## 6.1 Tenant Management

Untuk mengelola perusahaan billboard yang menjadi pelanggan SaaS.

Akses:

* Owner Platform SaaS.

Fitur:

* tambah tenant,
* edit tenant,
* aktif / nonaktif tenant,
* subscription tenant,
* paket / tier,
* batas jumlah billboard,
* batas jumlah CCTV,
* batas jumlah user,
* batas fitur analytics,
* batas fitur report,
* tenant branding,
* audit tenant,
* status pembayaran,
* suspend tenant jika diperlukan.

---

## 6.2 Subscription / Tier Management

Untuk mengatur paket SaaS.

Contoh tier:

* Starter,
* Growth,
* Pro,
* Enterprise,
* Custom Premium.

Parameter paket:

* harga,
* masa aktif,
* jumlah billboard,
* jumlah CCTV,
* jumlah user,
* jumlah client access,
* report PDF,
* export Excel,
* export CSV,
* live view,
* data retention,
* advanced analytics,
* data quality score,
* anomaly detection,
* white label report.

---

## 6.3 User & Role Management

Untuk mengatur user dan hak akses.

Role utama:

* Owner Platform SaaS,
* Admin Tenant,
* Owner Tenant,
* Sales,
* Teknisi,
* Client,
* Viewer / Report Viewer jika dibutuhkan.

Prinsip:

* permission tidak boleh hanya mengandalkan nama role,
* semua endpoint harus dicek permission,
* semua menu sidebar harus role-based,
* semua aksi penting masuk audit log,
* semua query tenant wajib difilter tenant_id.

---

## 6.4 Billboard Management

Untuk mengelola data billboard.

Data utama:

* billboard ID,
* nama billboard,
* kode billboard,
* alamat,
* koordinat GPS,
* kota,
* provinsi,
* ukuran,
* tipe billboard,
* road type,
* sisi / muka billboard,
* arah hadap,
* visibility angle,
* foto lokasi,
* monthly rate / estimated OOH budget,
* status aktif,
* kategori paket.

---

## 6.5 Camera / Device Management

Untuk mengelola CCTV atau Computer Vision Camera.

Data utama:

* camera ID,
* nama kamera,
* billboard terkait,
* lokasi kamera,
* stream URL encrypted,
* stream proxy URL,
* RTSP/IP camera internal,
* status online / offline,
* last heartbeat,
* stream quality,
* FPS,
* resolution,
* latency,
* device ID,
* health score,
* catatan maintenance.

Catatan penting:

* raw RTSP tidak boleh ditampilkan ke client.
* live streaming harus lewat secure streaming proxy.
* akses live streaming harus memakai token/session.
* live view harus dicatat di audit log.

---

## 6.6 Campaign Management

Untuk mengelola campaign iklan client.

Data utama:

* campaign ID,
* client ID,
* brand,
* campaign name,
* billboard yang disewa,
* periode mulai,
* periode selesai,
* materi iklan,
* campaign objective,
* estimated OOH budget,
* status campaign,
* dashboard access,
* report access.

---

## 6.7 AI Traffic Analytics

Modul untuk membaca data kendaraan dari CCTV.

Fitur:

* vehicle counting,
* vehicle classification,
* direction detection,
* speed rate,
* traffic density,
* congestion level,
* dwell time,
* peak hour,
* real-time summary,
* hourly summary,
* daily summary,
* monthly summary.

Jenis kendaraan minimum:

* motorcycle,
* car,
* bus,
* truck.

Catatan:

* pickup dan van dapat masuk kategori truck / light truck sesuai aturan final.
* data harus punya confidence score.
* data harus punya data quality score.
* data estimated harus diberi label.
* anomaly harus diberi flag.
* jangan membuat angka tanpa sumber.

---

## 6.8 Proof of Display

Modul bukti billboard tayang.

Karena kamera utama umumnya menghadap jalan, proof of display dapat memakai:

* foto pemasangan billboard,
* timestamp,
* GPS location,
* nama teknisi,
* approval admin,
* foto berkala,
* checklist campaign,
* catatan maintenance.

Untuk paket premium, dapat ditambah kamera khusus proof display.

Akses:

* Teknisi upload proof,
* Admin Tenant approval proof,
* Owner Tenant melihat ringkasan,
* Client melihat proof yang sudah approved.

---

## 6.9 Client Dashboard

Dashboard khusus client penyewa billboard.

Isi utama:

* campaign status,
* periode campaign,
* lokasi billboard,
* total kendaraan hari ini,
* total kendaraan minggu ini,
* total kendaraan bulan ini,
* klasifikasi kendaraan,
* peak hour,
* average speed,
* congestion level,
* estimated exposure,
* proof display,
* download report.

Client dashboard wajib mobile-first.

---

## 6.10 Owner Tenant Dashboard

Dashboard khusus Owner Tenant.

Isi utama:

* total billboard,
* billboard aktif,
* campaign aktif,
* client aktif,
* CCTV online,
* CCTV offline,
* total estimated exposure bulan ini,
* campaign hampir selesai,
* billboard traffic tertinggi,
* report terbaru,
* ringkasan performa tenant,
* grafik trend performa,
* insight bisnis.

Tidak boleh ada tombol:

* create,
* edit,
* delete,
* approve,
* manage user,
* manage role,
* manage camera,
* change subscription.

---

## 6.11 Admin Tenant Dashboard

Dashboard operasional tenant.

Isi utama:

* total vehicle today,
* total vehicle this month,
* estimated exposure,
* active campaign,
* active billboard,
* CCTV online,
* CCTV offline,
* data quality score,
* pending proof approval,
* report waiting review,
* alert maintenance,
* traffic trend,
* vehicle composition,
* hourly heatmap,
* top billboard performance,
* device health,
* latest activity.

Admin Tenant boleh melakukan aksi operasional sesuai permission.

---

## 6.12 Report Center

Modul untuk generate dan mengelola report.

Jenis report:

* daily report,
* weekly report,
* monthly report,
* campaign report,
* billboard report,
* client report.

Format:

* PDF,
* Excel,
* CSV,
* JSON/API jika nanti dibutuhkan.

Status report:

* Draft,
* Generated,
* Waiting Review,
* Approved,
* Sent to Client,
* Failed,
* Archived.

Report harus punya:

* generated_at,
* generated_by,
* report_period,
* report_version,
* formula_version,
* methodology_version,
* data_quality_summary,
* audit trace.

---

## 6.13 Alert & Maintenance

Modul peringatan teknis.

Alert utama:

* CCTV offline,
* stream putus,
* data traffic kosong,
* internet bermasalah,
* storage penuh,
* campaign belum ada proof,
* report gagal dibuat,
* device tidak mengirim heartbeat,
* data quality rendah,
* missing hours tinggi,
* anomaly traffic.

Severity:

* Critical,
* High,
* Medium,
* Low,
* Info.

---

# 7. Struktur Dashboard Berdasarkan Role

## 7.1 Sidebar Admin Tenant

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

---

## 7.2 Sidebar Owner Tenant

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

Owner Tenant hanya read-only.

---

## 7.3 Sidebar Client

```text
My Campaign
Traffic Performance
Proof of Display
Reports
Live View if Allowed
Profile
Logout
```

Client hanya melihat campaign miliknya.

---

## 7.4 Sidebar Teknisi

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

---

## 7.5 Sidebar Owner Platform SaaS

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

---

# 8. Traffic Analytics

Traffic Analytics mengacu pada benchmark manual ADX, tetapi harus lebih lengkap.

Fitur minimum:

* rekap traffic counting,
* filter Day / Week / Month / Custom,
* filter lokasi,
* filter campaign,
* filter client,
* filter vehicle class,
* filter direction,
* date range picker,
* export PDF,
* export CSV,
* export Excel,
* chart traffic,
* table traffic.

Fitur tambahan wajib untuk aplikasi kita:

* daily traffic trend,
* hourly traffic heatmap,
* vehicle composition donut,
* vehicle class trend,
* weekday vs weekend comparison,
* CPV / CPI comparison,
* cumulative exposure,
* data quality score,
* confidence score,
* anomaly flag,
* raw / validated / estimated data mode.

KPI Analytics:

* total vehicle,
* total car,
* total motorcycle,
* total bus,
* total truck,
* estimated exposure,
* average traffic per day,
* peak hour,
* peak day,
* average speed,
* congestion level,
* data quality score.

---

# 9. Live View / CCTV

Live View tidak boleh memakai pola raw RTSP yang langsung dibuka user.

Fitur Live View:

* secure web player,
* stream proxy,
* signed token,
* session expiry,
* one active stream session per user/camera jika diperlukan,
* viewer tracking,
* capture snapshot,
* report issue,
* toggle detection overlay,
* toggle counting line,
* audit log.

Informasi di halaman Live View:

* camera name,
* location,
* billboard,
* status online/offline,
* last heartbeat,
* stream quality,
* resolution,
* FPS,
* latency,
* current viewer,
* session expiry timer,
* vehicle count now,
* detection overlay status.

Larangan:

* jangan expose raw RTSP ke Client,
* jangan simpan RTSP plain text,
* jangan izinkan semua role membuka live stream,
* jangan buka live stream tanpa audit log.

---

# 10. Data Quality & Anti Overclaim

Aplikasi ini harus hati-hati dalam menyebut impression.

Gunakan istilah:

* estimated exposure,
* potential exposure,
* opportunity to see,
* potensi iklan terlihat.

Jangan gunakan klaim:

* pasti dilihat,
* pasti dibaca,
* pasti menghasilkan penjualan,
* pasti meningkatkan sales.

Sistem membaca kendaraan dan peluang exposure, bukan membaca mata manusia.

Setiap data traffic harus punya konteks:

* data source,
* camera ID,
* counting line,
* direction rule,
* exposure zone,
* confidence score,
* data quality score,
* valid hours,
* missing hours,
* uptime,
* anomaly flag,
* formula version,
* methodology version.

Kalau data estimated, tampilkan sebagai estimated.

Kalau data quality rendah, tampilkan warning.

Jangan tutupi data jelek dengan desain cantik. Itu namanya dashboard pakai bedak tebal.

---

# 11. Formula Dasar

## 11.1 Total Vehicle

```text
total_vehicle = car_count + motorcycle_count + bus_count + truck_count
```

---

## 11.2 Average Daily Traffic

```text
avg_daily_traffic = total_vehicle / campaign_days
```

---

## 11.3 CPV

```text
cpv = estimated_ooh_budget / total_vehicle
```

---

## 11.4 Estimated Exposure

Versi sederhana:

```text
estimated_exposure =
(car_count × car_occupancy_multiplier)
+ (motorcycle_count × motorcycle_occupancy_multiplier)
+ (bus_count × bus_occupancy_multiplier)
+ (truck_count × truck_occupancy_multiplier)
```

Versi matang:

```text
estimated_exposure =
vehicle_count
× occupancy_multiplier
× visibility_factor
× exposure_factor
× data_quality_adjustment
```

---

## 11.5 CPI

```text
cpi = estimated_ooh_budget / estimated_exposure
```

---

## 11.6 Data Quality Score

Contoh formula awal:

```text
data_quality_score =
(camera_uptime_score × 0.35)
+ (detection_confidence_score × 0.30)
+ (frame_quality_score × 0.20)
+ (missing_data_score × 0.15)
```

Formula final harus dikunci di dokumen AI_CCTV_ANALYTICS_RULES.md dan DATABASE_DESIGN.md.

---

# 12. Report Output

Report PDF aplikasi harus minimal setara benchmark report ADX, tetapi lebih transparan.

Struktur ideal:

1. Cover
2. Executive Summary
3. Campaign Overview
4. Measurement Methodology
5. Network Map
6. Overall KPI Recap
7. Location Summary
8. Traffic Analysis
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

Report wajib punya:

* client name,
* campaign name,
* brand,
* city / area,
* period,
* report type,
* generated date,
* generated by,
* report version,
* methodology version,
* formula version.

---

# 13. Database High-Level

Tabel utama:

```text
tenants
tenant_profiles
users
roles
permissions
user_roles
clients
billboards
billboard_faces
cameras
edge_devices
campaigns
campaign_billboards
proof_of_display
traffic_events
traffic_hourly_summaries
traffic_daily_summaries
traffic_monthly_summaries
reports
report_exports
report_generation_jobs
notifications
maintenance_logs
maintenance_tickets
subscription_plans
tenant_subscriptions
tier_feature_limits
audit_logs
system_settings
user_sessions
live_stream_sessions
camera_health_logs
camera_snapshots
alert_events
data_quality_snapshots
```

Catatan wajib:

* semua tabel operasional tenant wajib punya tenant_id,
* semua query tenant wajib filter tenant_id,
* Client wajib terkait campaign/client record,
* Owner Tenant wajib tenant_id tetapi read-only,
* Owner Platform SaaS tidak dianggap user operasional tenant,
* semua aksi penting wajib masuk audit_logs.

---

# 14. API High-Level

Endpoint utama yang nanti wajib dirancang detail:

```text
/auth/login
/auth/logout
/auth/me

/platform/tenants
/platform/plans
/platform/subscriptions
/platform/system-health

/analytics/summary
/analytics/timeseries
/analytics/vehicle-composition
/analytics/hourly-heatmap
/analytics/location/:locationId

/billboards
/billboards/:id

/campaigns
/campaigns/:id

/clients
/clients/:id

/devices
/devices/:id
/devices/:id/health

/live-stream/sessions
/live-stream/sessions/:id

/proof-display
/proof-display/:id/approve

/reports
/reports/generate
/reports/:id/download/pdf
/reports/:id/download/excel
/reports/:id/download/csv

/alerts
/maintenance
/audit-logs
/profile
```

Semua endpoint harus:

* authenticated,
* role-checked,
* tenant-filtered,
* audited untuk aksi penting.

---

# 15. MVP Roadmap Ringkas

## Phase 1 — Core SaaS Foundation

Fitur:

* login,
* multi-tenant,
* role user,
* Owner Platform SaaS,
* Admin Tenant,
* Owner Tenant read-only,
* master tenant,
* master billboard,
* master client,
* campaign dasar,
* dashboard dasar.

Target:

Aplikasi sudah punya struktur SaaS yang benar.

---

## Phase 2 — Role & Permission Hardening

Fitur:

* permission matrix,
* tenant isolation,
* audit log,
* role-based sidebar,
* role-based API guard,
* client campaign isolation,
* Owner Tenant read-only hardening.

Target:

Tidak ada role salah kuasa.

---

## Phase 3 — Dashboard & Analytics

Fitur:

* Admin Tenant dashboard,
* Owner Tenant dashboard,
* Client dashboard mobile,
* Traffic Analytics,
* Day / Week / Month / Custom filter,
* location filter,
* vehicle breakdown,
* chart traffic,
* export PDF / CSV / Excel.

Target:

Dashboard dasar siap dipakai client dan tenant.

---

## Phase 4 — Device & Live View

Fitur:

* device list,
* camera status,
* last heartbeat,
* camera health,
* secure live view,
* signed stream token,
* session expiry,
* snapshot,
* audit live access.

Target:

CCTV bisa dimonitor aman tanpa raw RTSP bocor.

---

## Phase 5 — Proof Display & Report Center

Fitur:

* upload proof,
* approval proof,
* report generator,
* report history,
* PDF export,
* Excel export,
* CSV export,
* report status.

Target:

Client bisa menerima bukti campaign berjalan dan report resmi.

---

## Phase 6 — AI Traffic Counting

Fitur:

* vehicle counting,
* car / motorcycle / bus / truck classification,
* hourly summary,
* daily summary,
* traffic chart,
* KPI kendaraan.

Target:

Dashboard mulai memakai data traffic aktual.

---

## Phase 7 — Data Quality & Intelligence

Fitur:

* confidence score,
* data quality score,
* anomaly flag,
* missing hours,
* uptime score,
* estimated data label,
* methodology notes,
* CPV,
* CPI,
* estimated exposure,
* recommendation engine awal.

Target:

Aplikasi naik kelas dari traffic dashboard menjadi intelligence platform.

---

# 16. Dokumentasi Proyek

Dokumen pondasi yang harus disiapkan / diperbarui:

```text
README.md
PROJECT_VISION_BUSINESS_RULES.md
SYSTEM_ARCHITECTURE.md
DATABASE_DESIGN.md
AI_CCTV_ANALYTICS_RULES.md
API_CONTRACTS_AND_EVENT_PIPELINE.md
UI_UX_DASHBOARD_REPORTING_GUIDE.md
SECURITY_PRIVACY_ACCESS_CONTROL.md
ROADMAP.md
```

Enam dokumen yang sedang diprioritaskan update setelah benchmark dashboard ADX:

```text
README.md
UI_UX_DASHBOARD_REPORTING_GUIDE.md
SECURITY_PRIVACY_ACCESS_CONTROL.md
API_CONTRACTS_AND_EVENT_PIPELINE.md
DATABASE_DESIGN.md
ROADMAP.md
```

README ini adalah peta utama. Detail teknis lengkap harus diletakkan di dokumen masing-masing.

---

# 17. Development Rules untuk Agent / Developer

Aturan keras:

* Jangan membuat fitur di luar roadmap tanpa alasan.
* Jangan membuat role baru tanpa update permission matrix.
* Jangan membuat dashboard tanpa role guard.
* Jangan membuat API tanpa tenant guard.
* Jangan membuat export report tanpa audit log.
* Jangan expose raw RTSP ke frontend client.
* Jangan menyimpan password plain text.
* Jangan menampilkan credential di dokumentasi.
* Jangan membuat angka traffic dummy tanpa label dummy/simulation.
* Jangan membuat impression seolah pasti dilihat manusia.
* Jangan menghapus field data quality / confidence dari rancangan.
* Jangan menjadikan tabel hourly besar sebagai UI utama.
* Jangan membuat Owner Tenant punya tombol operasional.
* Jangan membuat Client melihat data client lain.
* Jangan membangun UI cantik tetapi data tidak bisa diaudit.

Aturan positif:

* Mulai dari MVP yang bisa dijual.
* Buat dashboard yang jelas dan ringan.
* Pakai mobile-first untuk Client.
* Pakai web admin untuk data besar.
* Pisahkan raw data dan executive summary.
* Buat export PDF untuk client.
* Buat Excel/CSV untuk data mentah.
* Selalu tampilkan status data.
* Selalu tampilkan sumber data.
* Selalu simpan audit trail.
* Selalu pikirkan tenant isolation.

---

# 18. Non-Goals Tahap Awal

Hal yang tidak dikerjakan di awal:

* facial recognition,
* demographic detection,
* membaca mata manusia,
* klaim orang pasti melihat billboard,
* prediksi sales langsung dari billboard,
* dynamic pricing otomatis yang kompleks,
* marketplace billboard publik,
* mobile app native,
* billing payment gateway kompleks,
* AI recommendation terlalu canggih,
* multi-country support.

Tahap awal fokus Indonesia dan fokus pada aplikasi yang stabil, aman, bisa dipakai, dan bisa menghasilkan report.

---

# 19. Positioning Produk

Platform ini membantu perusahaan billboard memberikan bukti performa iklan kepada client melalui CCTV AI, traffic analytics, proof display, report otomatis, dan dashboard mobile real-time.

Kalimat positioning:

> Billboard Monitoring & Report Platform untuk perusahaan outdoor advertising yang ingin menjual titik iklan dengan data traffic, proof tayang, dan report profesional.

Versi pendek:

> Dari papan iklan menjadi media yang terukur.

Versi galak:

> Jangan jual “katanya ramai”. Jual dengan data.

---

# 20. Kesimpulan

Aplikasi ini adalah SaaS Billboard Monitoring & Report Platform yang menggabungkan:

* tenant management,
* subscription / tier management,
* user role management,
* billboard inventory,
* camera management,
* AI traffic analytics,
* proof of display,
* client dashboard,
* owner tenant dashboard,
* admin tenant dashboard,
* report center,
* alert & maintenance,
* data quality,
* audit trail.

Struktur role utama yang benar:

```text
Owner Platform SaaS
↓
Admin Tenant — Super Power Operasional Tenant
↓
Owner Tenant — Monitoring Only
↓
Sales / Teknisi / Client
```

Tujuan akhirnya adalah membantu perusahaan billboard menjual titik iklan dengan data, bukan sekadar klaim lokasi ramai.

Permission adalah pondasi. Kalau pondasi miring, dashboard secantik apa pun tetap rawan roboh.
