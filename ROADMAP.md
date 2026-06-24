# ROADMAP.md

# SaaS Billboard Monitoring & Report Platform

## 1. Tujuan Roadmap

Dokumen ini adalah peta urutan implementasi aplikasi **SaaS Billboard Monitoring & Report Platform**.

Roadmap ini wajib dipakai oleh agent/developer agar pembangunan aplikasi tidak lompat-lompat, tidak tergoda membuat UI cantik terlalu awal, dan tidak membangun fitur di atas pondasi yang belum kuat.

Urutan utama implementasi:

```text
Schema + tenant guard
→ Auth + role permission
→ Master data
→ Camera health
→ Traffic ingestion
→ Summary aggregation
→ Dashboard analytics
→ Proof display
→ Report job
→ Signed export
→ Audit & production hardening
```

Prinsipnya sederhana:

> Jangan pasang kaca gedung sebelum tiangnya berdiri.
> Jangan bikin dashboard kinclong kalau data, role, tenant, dan audit masih bolong.

---

## 2. Status Dokumen yang Menjadi Acuan

Roadmap ini mengacu pada dokumen pondasi berikut:

1. `README.md`
2. `UI_UX_DASHBOARD_REPORTING_GUIDE.md`
3. `SECURITY_PRIVACY_ACCESS_CONTROL.md`
4. `API_CONTRACTS_AND_EVENT_PIPELINE.md`
5. `DATABASE_DESIGN.md`
6. `ROADMAP.md`

Dokumen pendukung yang tetap harus sinkron:

1. `PROJECT_VISION_BUSINESS_RULES.md`
2. `SYSTEM_ARCHITECTURE.md`
3. `AI_CCTV_ANALYTICS_RULES.md`

Jika ada perubahan besar pada role, database, API, report, live streaming, formula exposure, atau data quality, maka roadmap ini wajib dicek ulang.

---

## 3. Prinsip Utama Implementasi

### 3.1 Tenant First

Aplikasi ini adalah SaaS multi-tenant.

Maka semua fitur operasional wajib memakai `tenant_id`, kecuali fitur global milik Owner Platform SaaS.

Tidak boleh ada query operasional tenant tanpa filter tenant.

Contoh salah:

```sql
SELECT * FROM campaigns;
```

Contoh benar:

```sql
SELECT * FROM campaigns WHERE tenant_id = :current_tenant_id;
```

### 3.2 Role & Permission Before UI

UI harus mengikuti permission, bukan sebaliknya.

Sidebar, tombol, API, query, dan export wajib dicek berdasarkan role dan permission.

Owner Tenant harus read-only.

Client hanya boleh melihat campaign miliknya.

Admin Tenant hanya berkuasa di tenant miliknya.

Owner Platform SaaS hanya mengelola platform, tenant, tier, subscription, dan audit global.

### 3.3 Data Before Decoration

Dashboard boleh modern, tapi data harus benar dulu.

Yang dibangun lebih dulu:

1. schema,
2. tenant guard,
3. auth,
4. permission,
5. master data,
6. ingestion,
7. summary,
8. audit,
9. baru visual dashboard.

Kalau chart muncul dari data dummy, wajib diberi label `SIMULATION`.

### 3.4 Report Must Be Audit-able

Report tidak boleh hanya berupa PDF cantik.

Report wajib punya:

* `report_id`,
* `tenant_id`,
* `campaign_id`,
* `generated_by`,
* `generated_at`,
* `report_period`,
* `report_version`,
* `formula_version`,
* `methodology_version`,
* `data_quality_summary`,
* `status`,
* `export_url`,
* audit trail.

### 3.5 No Raw RTSP Exposure

Raw RTSP tidak boleh tampil di frontend client.

Live stream wajib melalui:

* secure stream proxy,
* signed token,
* session expiry,
* viewer tracking,
* audit log.

---

## 4. Target Rilis Bertahap

Roadmap dibagi menjadi beberapa target rilis:

```text
MVP-0  : Pondasi teknis internal
MVP-1  : SaaS admin + master data
MVP-2  : Camera health + secure live view
MVP-3  : Traffic ingestion + summary
MVP-4  : Dashboard analytics + role dashboard
MVP-5  : Proof display + report generator
MVP-6  : Signed export + report approval
MVP-7  : Data quality + insight awal
V1     : Pilot production untuk 1–3 tenant
V1.5   : Hardening SaaS multi-tenant
V2     : Advanced analytics dan automation
```

---

# PHASE 0 — Documentation Freeze & Agent Guard

## Tujuan

Mengunci semua dokumen pondasi agar agent tidak membangun aplikasi berdasarkan asumsi liar.

## Pekerjaan

* Review `README.md`.
* Review `DATABASE_DESIGN.md`.
* Review `API_CONTRACTS_AND_EVENT_PIPELINE.md`.
* Review `SECURITY_PRIVACY_ACCESS_CONTROL.md`.
* Review `UI_UX_DASHBOARD_REPORTING_GUIDE.md`.
* Review roadmap ini.
* Tandai dokumen yang masih pending sinkronisasi.
* Buat daftar istilah resmi:

  * Owner Platform SaaS,
  * Admin Tenant,
  * Owner Tenant,
  * Client,
  * Teknisi,
  * Sales,
  * estimated exposure,
  * data quality score,
  * confidence score,
  * signed export,
  * live stream session.

## Output

* Semua dokumen core punya struktur yang saling nyambung.
* Tidak ada istilah role yang tabrakan.
* Tidak ada “Superadmin” lama yang tertinggal.
* Agent punya aturan kerja yang jelas.

## Definition of Done

* Dokumen prioritas sudah sinkron.
* Role utama sudah konsisten.
* Tenant isolation disebut di README, API, database, security, dan roadmap.
* Report output sudah punya arah yang sama dengan benchmark ADX tetapi lebih aman dan audit-able.

---

# PHASE 1 — Project Bootstrap & App Skeleton

## Tujuan

Membuat kerangka aplikasi yang rapi sebelum fitur bisnis dimasukkan.

## Pekerjaan Backend

* Setup repo.
* Setup environment.
* Setup struktur folder.
* Setup config environment:

  * development,
  * staging,
  * production.
* Setup database connection.
* Setup migration tool.
* Setup logging.
* Setup error handler.
* Setup response format standar API.
* Setup seed awal untuk role dan permission.

## Pekerjaan Frontend

* Setup layout dasar.
* Setup routing.
* Setup auth layout.
* Setup dashboard shell.
* Setup role-based sidebar placeholder.
* Setup component dasar:

  * card,
  * table,
  * form,
  * modal,
  * badge,
  * alert,
  * chart wrapper.

## Output

* Aplikasi bisa run lokal.
* API health check aktif.
* Frontend dashboard shell tampil.
* Belum perlu fitur bisnis penuh.

## Definition of Done

* Tidak ada fitur dummy tanpa label.
* Struktur folder rapi.
* Environment variable tidak hardcoded.
* Secret tidak masuk repo.
* Basic CI/lint/test minimal tersedia.

---

# PHASE 2 — Database Schema & Migration Foundation

## Tujuan

Membangun database sebagai pondasi utama SaaS.

## Pekerjaan

Buat migration untuk tabel inti:

* `tenants`
* `tenant_profiles`
* `subscription_plans`
* `tenant_subscriptions`
* `tier_feature_limits`
* `users`
* `roles`
* `permissions`
* `user_roles`
* `user_tenants`
* `clients`
* `billboards`
* `billboard_faces`
* `cameras`
* `edge_devices`
* `campaigns`
* `campaign_billboards`
* `proof_of_display`
* `traffic_events`
* `traffic_hourly_summaries`
* `traffic_daily_summaries`
* `traffic_monthly_summaries`
* `report_generation_jobs`
* `reports`
* `report_exports`
* `live_stream_sessions`
* `camera_health_logs`
* `camera_snapshots`
* `alert_events`
* `data_quality_snapshots`
* `audit_logs`
* `system_settings`

## Aturan Database

Semua tabel operasional tenant wajib punya:

```text
tenant_id
created_at
updated_at
created_by
updated_by
status
```

Tabel event dan log wajib punya metadata yang cukup untuk audit.

Traffic data wajib punya:

```text
camera_id
billboard_id
campaign_id
data_source
confidence_score
data_quality_score
is_estimated
anomaly_flag
formula_version
methodology_version
```

## Index Wajib

Minimal index:

* `tenant_id`
* `tenant_id + status`
* `tenant_id + created_at`
* `tenant_id + campaign_id`
* `tenant_id + billboard_id`
* `tenant_id + camera_id`
* `camera_id + captured_at`
* `campaign_id + date`
* `report_id + export_type`
* `audit_logs.tenant_id + created_at`

## Output

* Migration database selesai.
* Seeder role/permission awal tersedia.
* Seeder subscription plan awal tersedia.
* ERD high-level bisa dibuat dari schema.

## Definition of Done

* Semua tabel operasional tenant punya `tenant_id`.
* Tidak ada field RTSP plain text tanpa encryption strategy.
* Report dan export punya tabel sendiri.
* Traffic raw event dan summary dipisah.
* Audit log siap dipakai dari awal.

---

# PHASE 3 — Auth, Session, Tenant Guard & RBAC

## Tujuan

Mengunci akses sebelum fitur bisnis berjalan.

## Pekerjaan Auth

* Login.
* Logout.
* Current user `/auth/me`.
* Password hashing.
* Session/JWT strategy.
* Refresh token jika dibutuhkan.
* Session expiry.
* Password reset flow.
* Profile update.
* Change password.

## Pekerjaan Tenant Guard

* Resolve tenant dari user.
* Resolve active tenant.
* Middleware tenant guard.
* Prevent cross-tenant access.
* Owner Platform SaaS global mode.
* Tenant suspended guard.
* Tenant subscription active guard.

## Pekerjaan RBAC

Role utama:

* Owner Platform SaaS
* Admin Tenant
* Owner Tenant
* Sales
* Teknisi
* Client
* Viewer / Report Viewer jika dibutuhkan

Permission group:

* `tenant.manage`
* `subscription.manage`
* `user.manage`
* `role.manage`
* `billboard.manage`
* `camera.manage`
* `campaign.manage`
* `proof.upload`
* `proof.approve`
* `analytics.view`
* `report.generate`
* `report.review`
* `report.download`
* `live.view`
* `maintenance.manage`
* `audit.view`

## Output

* Login berfungsi.
* Role-based sidebar mulai aktif.
* Endpoint terlindungi auth.
* Endpoint operasional terlindungi tenant guard.
* Permission guard aktif.

## Definition of Done

* Owner Tenant tidak bisa create/edit/delete data operasional.
* Client tidak bisa melihat campaign client lain.
* Admin Tenant tidak bisa melihat tenant lain.
* Owner Platform SaaS tidak dianggap user operasional tenant.
* Semua aksi penting masuk audit log.

---

# PHASE 4 — Master Data SaaS & Tenant Operation

## Tujuan

Membuat data dasar aplikasi berjalan: tenant, billboard, camera, client, campaign.

## Pekerjaan Owner Platform SaaS

* CRUD tenant.
* Activate/suspend tenant.
* CRUD subscription plan.
* Assign plan ke tenant.
* Set limit:

  * billboard limit,
  * camera limit,
  * user limit,
  * report limit,
  * retention limit,
  * live view access,
  * data quality feature,
  * export feature.
* Platform dashboard awal.

## Pekerjaan Admin Tenant

* CRUD billboard.
* CRUD billboard face.
* CRUD client/brand.
* CRUD campaign.
* Assign billboard ke campaign.
* Assign client access.
* Manage internal user.
* Manage teknisi.
* Manage sales.

## Pekerjaan Owner Tenant

* Read-only dashboard.
* Read-only billboard list.
* Read-only campaign list.
* Read-only report list.

## Pekerjaan Client

* Melihat campaign miliknya.
* Melihat billboard campaign miliknya.
* Belum perlu analytics penuh.

## Output

* Tenant bisa dibuat.
* Billboard bisa dibuat.
* Client bisa dibuat.
* Campaign bisa dibuat.
* Campaign bisa dihubungkan ke billboard.
* Role read-only mulai terbukti.

## Definition of Done

* Tidak ada create/edit/delete di Owner Tenant dashboard.
* Client hanya melihat data miliknya.
* Semua master data tenant memakai tenant guard.
* Limit paket mulai dicek.

---

# PHASE 5 — Camera / Device Management & Camera Health

## Tujuan

Membuat CCTV sebagai perangkat terdaftar, terukur, dan bisa dimonitor.

## Pekerjaan

* Register camera.
* Register edge device.
* Assign camera ke billboard.
* Assign camera ke billboard face.
* Store stream URL secara aman.
* Store stream proxy URL.
* Heartbeat endpoint.
* Update online/offline status.
* Camera health logs.
* Last heartbeat.
* FPS.
* Resolution.
* Latency.
* Stream quality.
* Device alias.
* Maintenance note.
* Camera snapshot.
* Camera status dashboard.

## Alert Awal

* Camera offline.
* No heartbeat.
* Stream unstable.
* Low FPS.
* High latency.
* No data received.
* Storage issue.

## Output

* Admin Tenant bisa melihat daftar camera.
* Teknisi bisa update maintenance.
* Owner Tenant bisa melihat ringkasan CCTV online/offline.
* Client hanya bisa melihat status camera campaign jika diizinkan.

## Definition of Done

* Raw RTSP tidak tampil di client.
* Stream URL tidak disimpan plain text.
* Camera health tersimpan sebagai log.
* Status online/offline tidak hanya dummy.
* Alert camera mati bisa dibuat.

---

# PHASE 6 — Secure Live View / CCTV Streaming

## Tujuan

Memberikan live view aman tanpa membocorkan raw RTSP.

## Pekerjaan

* Stream proxy service.
* Create live stream session.
* Signed stream token.
* Session expiry.
* One active session per user/camera jika dibutuhkan.
* Viewer tracking.
* Audit live access.
* Snapshot capture.
* Detection overlay toggle.
* Counting line overlay toggle.
* Report issue from live view.

## UI Live View

Informasi wajib:

* camera name,
* location,
* billboard,
* status,
* last heartbeat,
* stream quality,
* FPS,
* resolution,
* latency,
* current viewer,
* session expiry timer,
* detection overlay status,
* counting line status.

## Output

* Live view bisa dibuka dari web player.
* Tidak perlu copy RTSP ke VLC.
* Session punya waktu aktif.
* Semua akses live tercatat.

## Definition of Done

* Tidak ada raw RTSP di frontend.
* Tidak ada live stream tanpa audit.
* Token expired benar-benar tidak bisa dipakai.
* Client hanya bisa live view jika permission aktif.
* Session limit berjalan jika dikonfigurasi.

---

# PHASE 7 — Traffic Ingestion & Event Pipeline

## Tujuan

Membuat pipeline masuknya data traffic dari AI/CCTV ke sistem.

## Pekerjaan

* Endpoint ingestion traffic event.
* Event validation.
* Vehicle class validation.
* Direction validation.
* Camera validation.
* Campaign active matching.
* Billboard matching.
* Store raw traffic events.
* Idempotency key.
* Duplicate prevention.
* Batch ingestion.
* Retry handling.
* Dead letter queue / failed ingestion log.
* Audit technical ingestion issue.

## Data Traffic Minimum

Vehicle class:

* motorcycle,
* car,
* bus,
* truck.

Metadata wajib:

* `tenant_id`
* `camera_id`
* `billboard_id`
* `campaign_id`
* `captured_at`
* `vehicle_class`
* `direction`
* `confidence_score`
* `track_id`
* `counting_line_id`
* `source`
* `is_estimated`
* `anomaly_flag`

## Output

* Sistem menerima event traffic.
* Event tersimpan.
* Event tidak langsung ditampilkan sebagai truth tanpa validasi.
* Data raw dan summary dipisah.

## Definition of Done

* Event dari camera non-tenant tidak diterima.
* Event dari camera tidak aktif ditolak atau diberi status invalid.
* Event duplicate tidak menggandakan count.
* Semua event punya source dan confidence.
* Data simulasi wajib diberi label simulation.

---

# PHASE 8 — Summary Aggregation & Data Quality

## Tujuan

Mengolah traffic raw menjadi summary yang siap dipakai dashboard dan report.

## Pekerjaan Summary

* Hourly summary job.
* Daily summary job.
* Monthly summary job.
* Vehicle class aggregation.
* Direction aggregation.
* Campaign aggregation.
* Billboard aggregation.
* Camera aggregation.
* Peak hour calculation.
* Peak day calculation.
* Average daily traffic.
* CPV.
* CPI.
* Estimated exposure.

## Pekerjaan Data Quality

Hitung:

* camera uptime,
* expected hours,
* valid captured hours,
* missing hours,
* average confidence,
* frame quality jika tersedia,
* detection quality,
* anomaly count,
* data quality score.

## Formula Awal

```text
total_vehicle = car_count + motorcycle_count + bus_count + truck_count
```

```text
avg_daily_traffic = total_vehicle / campaign_days
```

```text
cpv = estimated_ooh_budget / total_vehicle
```

```text
estimated_exposure =
(car_count × car_occupancy_multiplier)
+ (motorcycle_count × motorcycle_occupancy_multiplier)
+ (bus_count × bus_occupancy_multiplier)
+ (truck_count × truck_occupancy_multiplier)
```

```text
cpi = estimated_ooh_budget / estimated_exposure
```

```text
data_quality_score =
(camera_uptime_score × 0.35)
+ (detection_confidence_score × 0.30)
+ (frame_quality_score × 0.20)
+ (missing_data_score × 0.15)
```

## Output

* Summary per jam tersedia.
* Summary harian tersedia.
* Summary bulanan tersedia.
* Data quality snapshot tersedia.
* Dashboard tidak membaca raw event langsung untuk KPI besar.

## Definition of Done

* Summary bisa dihitung ulang.
* Formula version tersimpan.
* Methodology version tersimpan.
* Estimated data diberi label.
* Data quality rendah memunculkan warning.

---

# PHASE 9 — Role-Based Dashboard & Traffic Analytics

## Tujuan

Membangun dashboard sesuai role dan data yang sudah valid.

## Dashboard Admin Tenant

Isi:

* total vehicle today,
* total vehicle this month,
* active campaign,
* active billboard,
* active client,
* CCTV online/offline,
* pending proof approval,
* report waiting review,
* data quality score,
* alert maintenance,
* traffic trend,
* vehicle composition,
* hourly heatmap,
* top billboard.

## Dashboard Owner Tenant

Isi:

* executive KPI,
* total billboard,
* billboard aktif,
* campaign aktif,
* client aktif,
* CCTV health summary,
* estimated exposure,
* report terbaru,
* performance trend,
* insight bisnis.

Tidak boleh ada tombol:

* create,
* edit,
* delete,
* approve,
* manage user,
* manage role,
* manage camera.

## Dashboard Client

Mobile-first.

Isi:

* campaign status,
* campaign period,
* campaign location,
* total vehicle today,
* total vehicle this week,
* total vehicle this month,
* estimated exposure,
* vehicle composition,
* peak hour,
* proof display approved,
* report download.

## Traffic Analytics

Filter wajib:

* day,
* week,
* month,
* custom range,
* location,
* campaign,
* client,
* vehicle class,
* direction.

Chart wajib:

* daily traffic trend,
* hourly heatmap,
* vehicle composition,
* vehicle class trend,
* weekday vs weekend,
* CPV/CPI comparison,
* cumulative exposure.

## Output

* Dashboard utama siap dipakai.
* Analytics tidak cuma angka, tapi punya konteks.
* Client dapat melihat performa campaign dari HP.

## Definition of Done

* Semua dashboard role-based.
* Semua API dashboard tenant-filtered.
* Owner Tenant read-only.
* Client campaign isolation terbukti.
* Data quality warning tampil jika kualitas rendah.

---

# PHASE 10 — Proof of Display

## Tujuan

Memberikan bukti billboard tayang yang bisa dilihat client.

## Pekerjaan

* Upload proof oleh Teknisi.
* Upload proof oleh Admin Tenant jika diperlukan.
* Timestamp otomatis.
* GPS metadata jika tersedia.
* Campaign link.
* Billboard link.
* Camera/location link.
* Approval Admin Tenant.
* Reject proof dengan alasan.
* Proof history.
* Client hanya melihat proof approved.

## Proof Data

Minimal:

* proof photo,
* campaign,
* billboard,
* uploaded_by,
* uploaded_at,
* location,
* note,
* status,
* approved_by,
* approved_at.

## Output

* Proof bisa diupload.
* Proof bisa diapprove.
* Proof muncul di dashboard client.
* Proof bisa masuk report.

## Definition of Done

* Client tidak melihat proof pending/rejected.
* Semua approval masuk audit log.
* Proof terkait campaign aktif.
* Proof bisa difilter berdasarkan campaign dan tanggal.

---

# PHASE 11 — Report Center & Report Job

## Tujuan

Membuat sistem report yang bisa menghasilkan laporan harian, mingguan, bulanan, campaign, billboard, dan client.

## Pekerjaan

* Report generator job.
* Report queue.
* Report status.
* Draft report.
* Waiting review.
* Approved.
* Sent to client.
* Failed.
* Archived.
* Retry failed report.
* Report preview.
* Report history.
* Report snapshot.

## Struktur Report PDF

Minimal:

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

## Report Metadata

Report wajib punya:

* client name,
* campaign name,
* brand,
* city/area,
* period,
* report type,
* generated date,
* generated by,
* report version,
* formula version,
* methodology version,
* data quality summary.

## Output

* Report job berjalan.
* Report punya status.
* Report bisa direview sebelum dikirim.
* Report snapshot tersimpan.

## Definition of Done

* Report tidak membaca data live langsung tanpa snapshot.
* Report bisa diregenerate dengan versioning.
* Failed report tercatat.
* Semua generate dan approval masuk audit log.
* Report menyebut estimated exposure, bukan klaim “pasti dilihat”.

---

# PHASE 12 — Signed Export & Download Security

## Tujuan

Mengamankan file export report.

## Pekerjaan

* Export PDF.
* Export Excel.
* Export CSV.
* Generate signed URL.
* Expiry signed URL.
* Download audit log.
* Permission check sebelum download.
* Client hanya download report campaign miliknya.
* Owner Tenant download jika permission aktif.
* Admin Tenant download report tenant miliknya.
* Owner Platform SaaS melihat usage global, bukan asal membuka isi tenant.

## Output

* PDF bisa diunduh.
* Excel bisa diunduh.
* CSV bisa diunduh.
* Signed export aman.
* Download tercatat.

## Definition of Done

* File export tidak public permanen.
* Signed URL expired.
* Report client lain tidak bisa diakses.
* Semua download penting masuk audit.
* Export punya checksum/hash jika diperlukan.

---

# PHASE 13 — Alert, Maintenance & Incident Workflow

## Tujuan

Membuat sistem peringatan teknis agar operasional CCTV dan data tidak diam-diam rusak.

## Alert

Jenis alert:

* CCTV offline,
* no heartbeat,
* stream putus,
* data traffic kosong,
* missing hours tinggi,
* data quality rendah,
* report gagal dibuat,
* proof belum diupload,
* proof belum diapprove,
* storage penuh,
* anomaly traffic,
* device latency tinggi.

Severity:

* Critical,
* High,
* Medium,
* Low,
* Info.

## Maintenance

Fitur:

* maintenance ticket,
* assign teknisi,
* status ticket,
* field checklist,
* upload foto,
* notes,
* resolved timestamp.

## Output

* Alert muncul di dashboard Admin Tenant.
* Teknisi melihat tugas teknis.
* Owner Tenant melihat ringkasan health.
* Client hanya melihat status terbatas jika relevan.

## Definition of Done

* Critical alert tidak tenggelam.
* Alert punya status open/resolved.
* Maintenance punya riwayat.
* Semua tindakan maintenance masuk audit/log.

---

# PHASE 14 — Insight & Recommendation Engine Awal

## Tujuan

Membuat aplikasi tidak hanya menampilkan angka, tetapi memberi makna bisnis.

## Insight Awal

* Highest traffic location.
* Best CPI location.
* Best CPV location.
* Highest motorcycle composition.
* Highest car composition.
* Best mass awareness location.
* Best premium mobility location.
* Peak hour recommendation.
* Anomaly explanation.
* Weekday vs weekend behavior.
* Traffic drop/rise explanation.

## Aturan Insight

Insight tidak boleh overclaim.

Boleh mengatakan:

* “berpotensi lebih cocok”
* “terindikasi”
* “berdasarkan data traffic”
* “estimated exposure”
* “opportunity to see”

Tidak boleh mengatakan:

* “pasti dilihat”
* “pasti efektif”
* “pasti menaikkan penjualan”
* “pasti dibaca”

## Output

* Insight muncul di dashboard.
* Insight masuk report.
* Insight bisa direview Admin Tenant.

## Definition of Done

* Semua insight punya basis data.
* Insight menyebut periode data.
* Insight tidak berlebihan.
* Insight tidak mengganti keputusan manusia, hanya membantu analisa.

---

# PHASE 15 — Simulation Mode & Pilot Tenant

## Tujuan

Menjalankan aplikasi dalam mode simulasi sebelum production.

## Pekerjaan

* Seed tenant demo.
* Seed billboard demo.
* Seed campaign demo.
* Seed camera demo.
* Seed traffic simulation.
* Label semua data simulasi.
* Generate report simulasi.
* Test role semua user.
* Test export.
* Test live view mock/proxy.
* Test data quality low/high.

## Pilot Tenant

Pilot minimal:

* 1 tenant,
* 1 Admin Tenant,
* 1 Owner Tenant,
* 1 Client,
* 1 Teknisi,
* 1–3 billboard,
* 1–3 camera,
* 1 campaign aktif,
* report harian/mingguan/bulanan.

## Output

* Demo end-to-end berjalan.
* Report final bisa dibuat.
* Dashboard client bisa dipresentasikan.

## Definition of Done

* Tidak ada data simulasi tanpa label.
* Tenant isolation lulus uji.
* Role isolation lulus uji.
* Report bisa digenerate.
* Export bisa diunduh aman.
* Audit log terbentuk.

---

# PHASE 16 — Security, Privacy & Final Audit

## Tujuan

Mengunci aplikasi sebelum produksi.

## Audit Area

* Authentication.
* Authorization.
* Tenant isolation.
* Role permission.
* Client campaign isolation.
* Owner Tenant read-only.
* Admin Tenant tenant-only.
* Owner Platform global-only.
* RTSP security.
* Stream token expiry.
* Signed export expiry.
* Audit log completeness.
* Password policy.
* Secret management.
* Rate limiting.
* Input validation.
* File upload validation.
* Report download security.
* Backup strategy.
* Error logging.
* Data retention.

## Testing Wajib

* Cross-tenant access test.
* Client access test.
* Owner Tenant mutation test.
* Expired token test.
* Expired signed URL test.
* Live session hijack test.
* Report download permission test.
* SQL injection basic test.
* Upload file validation test.
* Audit log verification.

## Output

* Security checklist lulus.
* Role checklist lulus.
* Report checklist lulus.
* Data quality checklist lulus.
* Production readiness checklist lulus.

## Definition of Done

* Tidak ada role salah kuasa.
* Tidak ada raw RTSP bocor.
* Tidak ada report public tanpa signed URL.
* Tidak ada dashboard yang membuka data tenant lain.
* Tidak ada angka exposure tanpa formula dan methodology version.
* Tidak ada fitur production yang hanya berbasis dummy.

---

# PHASE 17 — Production Rollout V1

## Tujuan

Meluncurkan aplikasi untuk penggunaan nyata terbatas.

## Pekerjaan

* Setup production environment.
* Setup domain.
* Setup SSL.
* Setup database production.
* Setup storage.
* Setup backup.
* Setup monitoring.
* Setup logging.
* Setup error alert.
* Setup admin owner account.
* Setup tenant pertama.
* Setup camera production.
* Setup report template final.
* Setup onboarding guide.

## Output

* Aplikasi production aktif.
* Tenant pertama bisa onboarding.
* Dashboard bisa digunakan.
* Report bisa digenerate.
* Monitoring system berjalan.

## Definition of Done

* Backup berjalan.
* Error log termonitor.
* Owner Platform bisa memonitor tenant.
* Admin Tenant bisa menjalankan operasional.
* Client bisa melihat campaign.
* Report pertama bisa dikirim.

---

# PHASE 18 — Post-Launch Improvement

## Tujuan

Meningkatkan produk setelah pilot berjalan.

## Improvement

* Better dashboard UX.
* Better report design.
* White label report.
* More advanced heatmap.
* Advanced anomaly detection.
* More flexible formula engine.
* More camera vendor support.
* Better live stream performance.
* Notification system.
* Email report delivery.
* WhatsApp notification jika dibutuhkan.
* Billing automation.
* Subscription invoice.
* Plan upgrade/downgrade automation.
* Data retention policy per tier.
* API integration untuk client enterprise.

## Output

* Produk naik kelas dari MVP ke SaaS matang.

---

## 5. Urutan Implementasi Teknis yang Tidak Boleh Dilanggar

Agent/developer wajib mengikuti urutan ini:

```text
1. Database migration
2. Tenant guard
3. Auth
4. RBAC
5. Audit log
6. Master tenant
7. Master billboard
8. Master client
9. Campaign
10. Camera/device
11. Camera health
12. Secure live stream
13. Traffic ingestion
14. Traffic summary
15. Data quality
16. Dashboard analytics
17. Proof display
18. Report job
19. Report approval
20. Signed export
21. Final security audit
```

Jika ingin membuat fitur di luar urutan ini, wajib ada alasan teknis yang jelas.

---

## 6. Non-Goals Tahap Awal

Tidak dikerjakan di awal:

* facial recognition,
* demographic detection,
* membaca mata manusia,
* klaim orang pasti melihat billboard,
* prediksi sales langsung dari billboard,
* dynamic pricing otomatis kompleks,
* marketplace billboard publik,
* mobile app native,
* billing payment gateway kompleks,
* AI recommendation terlalu canggih,
* multi-country support,
* public API untuk semua client,
* advanced computer vision training dashboard.

---

## 7. Agent Development Rules

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
* Jangan menghapus field data quality dan confidence.
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
* Selalu bedakan raw, validated, estimated, dan simulation data.

---

## 8. Kesimpulan Roadmap

Roadmap ini mengunci arah pembangunan aplikasi:

```text
SaaS foundation dulu,
security dulu,
tenant isolation dulu,
data pipeline dulu,
baru dashboard dan report dibuat indah.
```

Aplikasi ini tidak boleh menjadi sekadar dashboard angka.

Aplikasi ini harus menjadi mesin pembuktian performa billboard yang:

* multi-tenant,
* aman,
* role-based,
* audit-able,
* transparan rumusnya,
* jelas kualitas datanya,
* kuat untuk report client,
* siap naik kelas menjadi SaaS OOH intelligence platform.

Kalau fondasinya benar, dashboard akan berdiri seperti gedung.
Kalau fondasinya salah, dashboard cuma panggung hajatan: ramai, terang, tapi habis acara langsung dibongkar.
