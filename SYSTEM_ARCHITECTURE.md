# SYSTEM_ARCHITECTURE.md

# System Architecture — SaaS Billboard Monitoring & Report Platform

Dokumen ini adalah arsitektur sistem resmi untuk aplikasi **SaaS Billboard Monitoring & Report Platform**.

Dokumen ini mengunci bagaimana aplikasi dibangun dari sisi frontend, backend, database, stream CCTV, AI analytics, ingestion, aggregation, report generation, signed export, audit log, tenant guard, role guard, dan observability.

Tujuannya sederhana: sistem boleh terlihat elegan di dashboard, tetapi pondasinya harus seperti cor beton production. Kalau arsitektur rapuh, angka traffic bisa tampil gagah di PDF, tetapi di belakangnya kabelnya kusut seperti headset lama di laci.

---

## 1. Posisi Dokumen

Dokumen ini wajib sinkron dengan:

1. `README.md`
2. `PROJECT_VISION_BUSINESS_RULES.md`
3. `DATABASE_DESIGN.md`
4. `AI_CCTV_ANALYTICS_RULES.md`
5. `API_CONTRACTS_AND_EVENT_PIPELINE.md`
6. `UI_UX_DASHBOARD_REPORTING_GUIDE.md`
7. `SECURITY_PRIVACY_ACCESS_CONTROL.md`
8. `ROADMAP.md`

Dokumen ini menjadi sumber kebenaran untuk:

- batas komponen sistem,
- alur data dari CCTV sampai report,
- arsitektur multi-tenant,
- tenant guard middleware,
- role/permission enforcement,
- secure live streaming,
- ingestion pipeline,
- AI processing pipeline,
- aggregation job,
- data quality engine,
- anomaly engine,
- report worker,
- signed export storage,
- audit logger,
- deployment dan observability.

Jika ada perbedaan antara dokumen ini dan implementasi kode, maka kode harus dikoreksi atau dokumen ini harus direvisi secara eksplisit.

---

## 2. Prinsip Arsitektur Utama

### 2.1 Multi-Tenant Sejak Awal

Aplikasi ini adalah SaaS multi-tenant.

Artinya:

- satu platform pusat,
- banyak perusahaan billboard dapat memakai platform,
- setiap tenant memiliki data sendiri,
- setiap tenant memiliki user sendiri,
- setiap tenant memiliki client sendiri,
- setiap tenant memiliki billboard, camera, campaign, report, dan proof sendiri,
- data antar-tenant wajib terisolasi.

Tidak boleh membuat MVP single-tenant dengan janji “nanti gampang dimulti-tenant-kan”. Biasanya “nanti gampang” itu saudara kandung dari “besok diet”.

### 2.2 Tenant Guard Wajib di Semua Layer

Tenant isolation tidak boleh hanya ada di frontend.

Tenant guard wajib ada di:

- database schema,
- query repository/service,
- API middleware,
- background worker,
- report worker,
- export/download endpoint,
- live stream session,
- audit log,
- object storage path,
- queue payload.

Semua resource operasional tenant wajib memiliki `tenant_id`, kecuali resource global milik platform seperti `subscription_plans`, `platform_settings`, dan global audit tertentu.

### 2.3 Role Guard dan Permission Guard Wajib Terpisah

Role adalah label. Permission adalah izin nyata.

Sistem tidak boleh hanya mengecek nama role seperti:

```text
if role == "admin"
```

Sistem wajib mengecek permission eksplisit seperti:

```text
campaign.create
report.generate
camera.live_view
proof.approve
platform.tenant.suspend
```

Role utama:

- Owner Platform SaaS,
- Admin Tenant,
- Owner Tenant,
- Sales Tenant,
- Teknisi Lapangan,
- Client / Brand,
- Viewer / Report Viewer jika dibutuhkan.

Aturan galak:

- Owner Platform SaaS bukan Admin Tenant.
- Admin Tenant hanya berkuasa di tenant sendiri.
- Owner Tenant hanya monitoring/read-only.
- Client hanya melihat campaign miliknya.
- Teknisi tidak boleh melihat data bisnis luas.
- Sales tidak boleh mengelola device atau approval proof.

### 2.4 Raw RTSP Tidak Boleh Bocor ke Frontend

Raw RTSP adalah rahasia infrastruktur.

Frontend tidak boleh menerima:

- raw RTSP URL,
- username/password camera,
- IP camera internal,
- link stream internal,
- token kamera permanen.

Live view harus melalui:

- secure stream proxy,
- signed live stream session,
- session expiry,
- role guard,
- tenant guard,
- audit log.

Manual dashboard benchmark yang meminta user copy RTSP ke VLC boleh dipakai sebagai acuan fitur live view, tetapi bukan acuan keamanan production.

### 2.5 Angka Traffic Harus Audit-able

Setiap angka di dashboard dan report harus bisa ditelusuri ke:

- tenant,
- campaign,
- billboard,
- billboard face,
- camera,
- AI rule version,
- counting line,
- direction rule,
- exposure zone,
- raw event,
- summary source,
- confidence score,
- data quality score,
- formula version,
- methodology version,
- generated job,
- generated user/system.

Kalau angka tidak bisa ditelusuri, angka itu tidak boleh masuk report final.

### 2.6 Frontend Boleh Cantik, Backend Harus Galak

Frontend boleh modern, mobile-first, dan enak dilihat.

Namun aturan keamanan, validasi, permission, tenant isolation, dan report integrity harus ditegakkan di backend.

Frontend bukan satpam. Frontend itu etalase. Satpamnya backend.

---

## 3. Arsitektur Level Tinggi

```text
+---------------------------------------------------------------+
|                         Users / Roles                         |
| Owner Platform | Admin Tenant | Owner Tenant | Sales | Client |
| Teknisi                                                      |
+-------------------------------+-------------------------------+
                                |
                                v
+---------------------------------------------------------------+
|                      Web App / Client App                     |
| Admin Web Dashboard | Owner Dashboard | Client Mobile Dashboard |
| Live View UI | Report Center | Traffic Analytics UI            |
+-------------------------------+-------------------------------+
                                |
                                v
+---------------------------------------------------------------+
|                         API Gateway / BFF                     |
| Auth Middleware | Tenant Guard | Role Guard | Rate Limit        |
| Request Validation | Audit Context | Response Filtering           |
+-------------------------------+-------------------------------+
                                |
                                v
+---------------------------------------------------------------+
|                       Core Backend Services                   |
| Tenant Service | User & RBAC Service | Billboard Service          |
| Camera Service | Campaign Service | Proof Service | Report Service |
| Analytics Query Service | Notification Service                |
+-------------------------------+-------------------------------+
                                |
        +-----------------------+------------------------+
        |                                                |
        v                                                v
+--------------------------+                +--------------------------+
| Operational Database     |                | Object Storage           |
| PostgreSQL recommended   |                | PDF / Excel / CSV        |
| tenant data, events,     |                | snapshots, proof photos, |
| summaries, reports,      |                | report exports           |
| audit logs               |                | signed URLs              |
+--------------------------+                +--------------------------+
        |                                                ^
        v                                                |
+---------------------------------------------------------------+
|                     Queue / Background Jobs                   |
| ingestion jobs | aggregation jobs | report jobs | alert jobs      |
| data quality jobs | anomaly jobs | cleanup jobs                |
+-------------------------------+-------------------------------+
                                |
                                v
+---------------------------------------------------------------+
|                    AI / CCTV Processing Layer                 |
| Stream Ingestor | Frame Sampler | Detection Worker             |
| Tracking Worker | Counting Engine | Classification Engine        |
| Data Quality Engine | Anomaly Engine                         |
+-------------------------------+-------------------------------+
                                ^
                                |
+---------------------------------------------------------------+
|                         Camera / Edge Layer                   |
| CCTV / CVC | Edge Device | Heartbeat Agent | Stream Source     |
+---------------------------------------------------------------+
```

---

## 4. Komponen Utama Sistem

## 4.1 Web Application

Web application adalah antarmuka utama semua role.

Terdiri dari:

- Admin Tenant Dashboard,
- Owner Tenant Executive Dashboard,
- Client Mobile Dashboard,
- Owner Platform SaaS Dashboard,
- Teknisi Dashboard,
- Traffic Analytics,
- Live View,
- Report Center,
- Proof of Display,
- Settings dan Profile.

Aturan frontend:

- menu harus role-based,
- tombol harus permission-based,
- Client dashboard wajib mobile-first,
- Owner Tenant tidak boleh melihat tombol create/edit/delete/approve,
- live view tidak boleh menerima raw RTSP,
- export/download harus melalui signed backend endpoint,
- data estimated/low quality/anomaly wajib tampil sebagai badge/warning.

Frontend tidak boleh:

- menyimpan secret,
- menyimpan raw RTSP,
- membuat formula final sendiri,
- menghitung report final sendiri,
- memutuskan tenant access sendiri,
- memalsukan status data.

---

## 4.2 API Gateway / Backend-for-Frontend

API Gateway atau BFF adalah pintu masuk semua request dari frontend.

Tanggung jawab:

- autentikasi,
- session validation,
- tenant guard,
- role guard,
- permission guard,
- request validation,
- response shaping,
- rate limiting,
- audit context injection,
- error standardization.

Setiap request wajib membawa konteks:

```text
request_id
user_id
role_ids
permission_codes
tenant_id, jika user tenant/client
client_id, jika user client
session_id
ip_address
user_agent
```

Untuk Owner Platform SaaS, `tenant_id` boleh kosong untuk endpoint platform global. Tetapi untuk endpoint support/audit tenant, tenant scope harus eksplisit.

---

## 4.3 Auth Service

Auth Service menangani login, logout, session, token, password, dan identitas user.

Fitur wajib:

- login menggunakan email dan password,
- password hash kuat,
- session expiry,
- refresh session/token jika arsitektur memakai token,
- logout revoke session,
- lock account setelah percobaan gagal berulang,
- optional 2FA untuk Owner Platform dan Admin Tenant,
- audit login/logout/failed login,
- user status active/inactive/suspended.

Larangan:

- password plain text,
- password default sama dengan email,
- credential ditulis di dokumen,
- token permanen tanpa expiry,
- membiarkan user suspended tetap login.

---

## 4.4 RBAC / Permission Service

RBAC Service mengatur role dan permission.

Sistem wajib mendukung:

- role bawaan platform,
- permission granular,
- role assignment per user,
- tenant-scoped role,
- client-scoped access,
- read-only Owner Tenant,
- audit perubahan role/permission.

Permission minimum:

```text
platform.tenant.read
platform.tenant.create
platform.tenant.update
platform.tenant.suspend
platform.plan.manage
platform.subscription.manage
platform.audit.read

tenant.dashboard.read
tenant.settings.update
user.read
user.create
user.update
user.disable
role.assign

billboard.read
billboard.create
billboard.update
billboard.delete

camera.read
camera.create
camera.update
camera.delete
camera.live_view
camera.health.read
camera.maintenance.update

campaign.read
campaign.create
campaign.update
campaign.delete
campaign.assign_client

client.read
client.create
client.update
client.disable

proof.read
proof.upload
proof.approve
proof.reject

analytics.read
analytics.export
analytics.raw.read

report.read
report.generate
report.review
report.approve
report.download
report.send_to_client

audit.read
```

Permission tidak boleh hard-coded di banyak tempat. Permission harus dikontrol dari satu lapisan guard.

---

## 4.5 Tenant Service

Tenant Service mengatur perusahaan billboard sebagai pelanggan SaaS.

Tanggung jawab:

- tenant profile,
- tenant status,
- subscription plan,
- tier limit,
- branding,
- feature flags,
- usage limits,
- suspend/activate tenant,
- tenant audit.

Status tenant:

```text
trial
active
past_due
suspended
cancelled
archived
```

Jika tenant `suspended`, maka:

- user tenant tidak boleh mengakses dashboard operasional,
- client tenant tidak boleh melihat report baru kecuali aturan bisnis mengizinkan read-only grace period,
- ingestion bisa tetap berjalan atau dihentikan sesuai plan/policy,
- report generation baru harus diblokir,
- export baru harus diblokir,
- semua attempt masuk audit log.

---

## 4.6 Billboard Service

Billboard Service mengelola inventory billboard.

Entitas utama:

- billboard,
- billboard face/sisi/muka,
- location,
- road type,
- visibility angle,
- package category,
- monthly rate / estimated OOH budget,
- photo location,
- map coordinate.

Aturan:

- billboard wajib milik satu tenant,
- billboard boleh punya banyak face,
- face boleh punya satu atau lebih camera tergantung paket/lokasi,
- tiap camera wajib terhubung ke billboard/face,
- deletion harus soft delete jika sudah punya campaign/report/event.

---

## 4.7 Camera / Device Service

Camera Service mengelola CCTV atau Computer Vision Camera.

Tanggung jawab:

- camera registry,
- device registry,
- encrypted stream URL,
- stream proxy URL,
- camera health,
- heartbeat,
- live stream session,
- snapshot,
- technical maintenance,
- calibration status.

Data camera minimum:

```text
camera_id
tenant_id
billboard_id
billboard_face_id
edge_device_id
camera_name
camera_position
road_direction
stream_url_encrypted
stream_proxy_key
status
last_heartbeat_at
fps
resolution
latency_ms
stream_quality_score
health_score
calibration_status
created_at
updated_at
```

Aturan:

- raw stream URL harus encrypted at rest,
- raw stream URL hanya boleh dibaca oleh backend service tertentu,
- frontend hanya boleh menerima signed playback URL,
- setiap live view wajib membuat `live_stream_session`,
- setiap snapshot wajib tercatat di audit/event log,
- camera offline harus membuat alert.

---

## 4.8 Secure Stream Proxy

Secure Stream Proxy adalah komponen yang mengubah stream internal menjadi live playback yang aman untuk web.

Fungsi:

- menerima request live view dari backend,
- mengambil stream internal dari camera/edge,
- mengubah stream ke format web-friendly jika diperlukan,
- membuat signed playback session,
- membatasi durasi session,
- membatasi role/user/camera,
- mencatat viewer,
- memutus session saat expired.

Aturan session:

```text
session_id
user_id
tenant_id
camera_id
started_at
expires_at
status
viewer_ip
viewer_user_agent
permission_snapshot
```

Kebijakan awal:

- session live view default 10 menit atau sesuai konfigurasi tenant/tier,
- dapat diperpanjang jika user masih aktif dan permission valid,
- optional one active stream session per user/camera,
- Admin Tenant dan Teknisi boleh melihat camera sesuai permission,
- Client hanya boleh live view jika campaign terkait dan permission live view aktif,
- Owner Tenant boleh melihat status/live view hanya jika permission diberikan.

Larangan:

- menampilkan raw RTSP,
- copy-paste RTSP ke user,
- stream tanpa session,
- stream tanpa audit,
- stream untuk user yang tidak terkait campaign/tenant.

---

## 4.9 Edge Device / Field Agent

Edge Device adalah perangkat lapangan yang menghubungkan CCTV dengan sistem pusat.

Bisa berupa:

- mini PC,
- NVR gateway,
- edge AI device,
- server lokal kecil,
- cloud-connected camera agent.

Tanggung jawab edge:

- mengirim heartbeat,
- menjaga stream connectivity,
- mengirim metadata health,
- optional melakukan frame sampling,
- optional menjalankan AI inference lokal,
- fallback buffer jika internet putus,
- mengirim event batch saat koneksi kembali.

Mode processing:

### A. Cloud AI Processing

```text
Camera/Edge -> Stream/Frame -> Cloud AI Worker -> Event Store
```

Kelebihan:

- kontrol model lebih mudah,
- update model terpusat,
- lebih mudah audit.

Kekurangan:

- butuh bandwidth lebih besar,
- latency tergantung jaringan.

### B. Edge AI Processing

```text
Camera -> Edge AI -> Event Batch -> Cloud Event Store
```

Kelebihan:

- hemat bandwidth,
- bisa tetap bekerja saat koneksi terbatas,
- latency lebih rendah.

Kekurangan:

- deployment model lebih rumit,
- perlu device management kuat,
- audit model/version harus disiplin.

### C. Hybrid Processing

```text
Edge melakukan pre-processing, cloud melakukan validation/aggregation/report.
```

Rekomendasi MVP:

- mulai dengan cloud-friendly atau hybrid ringan,
- event yang dikirim harus punya schema baku,
- jangan mengunci arsitektur ke satu vendor kamera.

---

## 4.10 AI Processing Layer

AI Processing Layer menjalankan aturan dari `AI_CCTV_ANALYTICS_RULES.md`.

Komponen:

1. Stream Ingestor
2. Frame Sampler
3. Detection Worker
4. Classification Worker
5. Tracking Worker
6. Counting Engine
7. Direction Validator
8. Exposure Zone Evaluator
9. Confidence Calculator
10. Data Quality Engine
11. Anomaly Engine
12. Event Publisher

Pipeline:

```text
stream/frame
-> frame sampling
-> vehicle detection
-> vehicle classification
-> object tracking
-> counting line crossing
-> direction validation
-> duplicate filtering
-> exposure zone evaluation
-> confidence calculation
-> event creation
-> event store
-> summary aggregation
```

Aturan:

- setiap event wajib punya `camera_id`, `tenant_id`, `timestamp`, `vehicle_class`, `confidence`, `rule_version`, dan `data_status`,
- event raw tidak boleh langsung dianggap final,
- event final harus melewati validation/quality checks,
- low confidence event bisa masuk raw event tetapi tidak selalu masuk summary final,
- estimated event harus diberi label.

---

## 4.11 Ingestion Service

Ingestion Service menerima traffic event dari AI worker atau edge device.

Tanggung jawab:

- validasi payload,
- idempotency check,
- tenant/camera mapping,
- schema version validation,
- timestamp normalization,
- duplicate prevention,
- raw event storage,
- publish event ke queue aggregation,
- ingestion audit.

Payload event harus mengandung:

```text
event_id
schema_version
tenant_id
camera_id
billboard_id
billboard_face_id
campaign_id, optional jika mapping aktif
timestamp
vehicle_class
direction
track_id
confidence_score
classification_confidence
tracking_confidence
counting_rule_id
counting_rule_version
exposure_zone_id
source_type
processing_mode
is_estimated
quality_flags
```

Aturan idempotency:

- `event_id` harus unique,
- jika edge tidak bisa membuat event_id global, backend membuat deterministic hash dari camera_id + timestamp + track_id + rule_version,
- duplicate event tidak boleh menggandakan count,
- duplicate attempt dicatat sebagai ingestion warning.

---

## 4.12 Event Store

Traffic Event Store menyimpan data event granular.

Fungsi:

- sumber raw/validated data,
- dasar audit angka,
- dasar re-aggregation jika formula/rule berubah,
- sumber investigasi anomaly,
- sumber appendix data jika dibutuhkan.

Event store dapat menggunakan PostgreSQL partitioning untuk MVP awal. Jika volume sangat tinggi, bisa dipindah ke time-series database atau data warehouse.

Aturan:

- event tidak boleh dihapus fisik sembarangan,
- koreksi data harus memakai adjustment/revision record,
- summary harus bisa ditelusuri ke range event,
- retention harus mengikuti tier/subscription.

---

## 4.13 Aggregator Service

Aggregator Service mengubah event menjadi summary.

Jenis summary:

- realtime/minutely summary jika dibutuhkan,
- hourly summary,
- daily summary,
- monthly summary,
- campaign period summary,
- report snapshot summary.

Pipeline:

```text
traffic_events
-> aggregation job
-> hourly summary
-> daily summary
-> monthly summary
-> campaign summary
-> report snapshot
```

Aturan:

- aggregation wajib tenant-scoped,
- aggregation wajib idempotent,
- aggregation wajib menyimpan `source_event_count`,
- aggregation wajib menyimpan `formula_version`,
- aggregation wajib menyimpan `methodology_version`,
- aggregation wajib menyimpan `data_quality_score`,
- aggregation wajib menyimpan `is_estimated` jika ada estimated data,
- perubahan rule/formula harus membuat re-aggregation job, bukan mengubah angka diam-diam.

---

## 4.14 Data Quality Engine

Data Quality Engine menghitung kualitas data.

Input:

- camera uptime,
- detection confidence,
- frame quality,
- missing data,
- stream stability,
- calibration status,
- anomaly flags.

Output:

```text
data_quality_score
data_quality_level
quality_flags
missing_hours
valid_hours
camera_uptime_minutes
avg_confidence
frame_quality_score
```

Level:

```text
Excellent: 90-100
Good: 80-89
Fair: 70-79
Poor: 50-69
Critical: 0-49
```

Aturan:

- data quality harus tampil di dashboard analytics,
- data quality harus masuk report,
- report final harus memberi warning jika quality rendah,
- data quality tidak boleh disembunyikan karena “takut client ilfeel”. Justru client percaya karena kita jujur.

---

## 4.15 Anomaly Engine

Anomaly Engine mendeteksi pola mencurigakan.

Jenis anomaly:

- sudden traffic drop,
- sudden traffic spike,
- zero traffic abnormal,
- vehicle class distribution shift,
- camera offline,
- stream unstable,
- low confidence,
- low frame quality,
- duplicate suspected,
- counting rule changed,
- calibration missing,
- missing hours tinggi.

Output:

```text
anomaly_id
tenant_id
camera_id
billboard_id
campaign_id
anomaly_type
severity
start_time
end_time
affected_metric
description
possible_reason
status
```

Status:

```text
open
acknowledged
resolved
ignored_with_reason
```

Aturan:

- anomaly high/critical harus membuat alert,
- anomaly harus masuk data integrity report,
- anomaly yang mempengaruhi report final harus diberi catatan,
- anomaly tidak boleh otomatis dihapus.

---

## 4.16 Analytics Query Service

Analytics Query Service membaca summary untuk dashboard.

Endpoint analytics tidak boleh menghitung berat langsung dari raw event untuk setiap page load.

Sumber utama dashboard:

- hourly summaries,
- daily summaries,
- monthly summaries,
- campaign summaries,
- data quality snapshots,
- anomaly events.

Fitur query:

- filter day/week/month/custom,
- filter location,
- filter billboard,
- filter campaign,
- filter client,
- filter vehicle class,
- filter direction,
- compare location,
- hourly heatmap,
- vehicle composition,
- CPV/CPI,
- estimated exposure.

Aturan:

- Client hanya boleh query campaign miliknya,
- Owner Tenant hanya read-only tenant summary,
- Admin Tenant full tenant analytics,
- Owner Platform hanya global usage/platform health, bukan detail bisnis tenant kecuali mode support/audit resmi.

---

## 4.17 Report Service

Report Service mengelola lifecycle report.

Status report:

```text
draft
generating
generated
waiting_review
approved
sent_to_client
failed
archived
```

Report harus punya:

```text
report_id
tenant_id
campaign_id
client_id
report_type
period_start
period_end
status
generated_by
generated_at
approved_by
approved_at
formula_version
methodology_version
data_snapshot_id
pdf_export_id
excel_export_id
csv_export_id
audit_trace_id
```

Report final tidak boleh membaca data live langsung setiap kali dibuka. Report final harus berbasis snapshot.

Alasannya: report yang sudah dikirim ke client harus stabil. Kalau hari ini dibuka angka berubah, besok client tanya, lusa kita garuk kepala. Jangan bikin PDF jadi makhluk gaib.

---

## 4.18 Report Worker

Report Worker adalah background job untuk membuat PDF/Excel/CSV.

Pipeline:

```text
report generation request
-> permission check
-> tenant guard
-> create report job
-> build data snapshot
-> validate data quality
-> render charts
-> render PDF
-> render Excel/CSV
-> upload to object storage
-> create signed export metadata
-> update report status
-> write audit log
-> notify requester
```

Aturan:

- report generation harus asynchronous,
- report job harus retryable,
- report job harus idempotent,
- report output harus versioned,
- report final harus punya checksum/hash,
- report final harus punya formula_version dan methodology_version,
- report failed harus menyimpan error reason internal,
- error detail sensitif tidak boleh tampil ke client.

---

## 4.19 Object Storage & Signed Export

Object Storage menyimpan file:

- PDF report,
- Excel export,
- CSV export,
- proof photo,
- camera snapshot,
- chart image,
- campaign creative.

Struktur path wajib tenant-scoped:

```text
/tenants/{tenant_id}/campaigns/{campaign_id}/reports/{report_id}/report.pdf
/tenants/{tenant_id}/campaigns/{campaign_id}/reports/{report_id}/raw-data.xlsx
/tenants/{tenant_id}/proofs/{proof_id}/photo.jpg
/tenants/{tenant_id}/cameras/{camera_id}/snapshots/{snapshot_id}.jpg
```

Download file harus melalui signed URL atau signed backend endpoint.

Aturan:

- file tenant A tidak boleh bisa ditebak oleh tenant B,
- signed URL harus punya expiry,
- download harus dicatat di audit log,
- report final harus immutable kecuali dibuat versi baru,
- object storage tidak boleh public bucket sembarangan.

---

## 4.20 Audit Logger

Audit Logger mencatat aksi penting.

Aksi yang wajib diaudit:

- login/logout/failed login,
- create/update/delete/suspend tenant,
- create/update/delete user,
- assign role,
- create/update/delete billboard,
- create/update/delete camera,
- view live stream,
- create/update/delete campaign,
- upload/approve/reject proof,
- generate/review/approve/download/send report,
- export analytics,
- change formula/methodology,
- change AI rule/counting line,
- access support/audit mode,
- failed permission attempt,
- failed tenant access attempt.

Audit log minimum:

```text
audit_id
request_id
tenant_id, nullable untuk platform global
actor_user_id
actor_role
actor_type
action
resource_type
resource_id
before_json
after_json
ip_address
user_agent
result
created_at
```

Aturan:

- audit log tidak boleh diedit user biasa,
- audit log tidak boleh dihapus tanpa retention policy,
- audit log harus searchable oleh role yang berhak,
- failed access attempt tetap dicatat.

---

## 4.21 Notification & Alert Service

Notification Service mengirim pemberitahuan untuk:

- camera offline,
- camera online kembali,
- stream putus,
- missing data tinggi,
- data quality rendah,
- anomaly high/critical,
- proof pending approval,
- report generated,
- report failed,
- campaign hampir selesai,
- subscription bermasalah.

Channel awal:

- in-app notification,
- email optional,
- WhatsApp/Telegram optional untuk future.

Severity:

```text
critical
high
medium
low
info
```

Aturan:

- alert teknis dikirim ke Admin Tenant dan Teknisi,
- Owner Tenant hanya melihat ringkasan, bukan dibombardir notifikasi teknis kecil,
- Owner Platform melihat global platform alert,
- Client hanya menerima notifikasi yang terkait campaign/report miliknya.

---

## 5. Alur Data Utama

## 5.1 Alur Login dan Akses Dashboard

```text
User login
-> Auth Service validasi credential
-> session/token dibuat
-> user roles & permissions dimuat
-> tenant context dimuat
-> frontend menampilkan dashboard sesuai role
-> setiap API request melewati Auth + Tenant Guard + Permission Guard
```

Aturan:

- menu yang tidak punya permission jangan ditampilkan,
- endpoint tetap harus menolak request tanpa permission meskipun user mencoba akses manual,
- Owner Tenant read-only enforced di API, bukan hanya sembunyi tombol.

---

## 5.2 Alur Live View CCTV

```text
User klik Live View
-> frontend request create live stream session
-> backend cek auth, tenant, permission, camera access
-> backend membuat live_stream_session
-> stream proxy membuat signed playback URL
-> frontend memutar stream via web player
-> session expiry timer berjalan
-> audit log mencatat view
-> session expired/closed
```

Aturan:

- raw RTSP tidak pernah dikirim ke frontend,
- session harus punya expiry,
- permission dicek sebelum session dibuat,
- Client hanya boleh melihat camera campaign miliknya jika live view diaktifkan,
- setiap start/stop/expired masuk log.

---

## 5.3 Alur Traffic Counting

```text
Camera/Edge stream
-> Stream Ingestor / Frame Sampler
-> AI Detection
-> Classification
-> Tracking
-> Counting Line Crossing
-> Direction Validation
-> Duplicate Filter
-> Confidence Calculation
-> Traffic Event Created
-> Ingestion Service
-> Event Store
-> Aggregator Queue
-> Hourly/Daily Summary
-> Dashboard Analytics
```

Aturan:

- event harus punya schema version,
- event harus punya AI rule version,
- event harus punya confidence,
- event harus punya data status,
- aggregation tidak boleh menggandakan duplicate event.

---

## 5.4 Alur Data Quality dan Anomaly

```text
Camera health + traffic event + frame quality
-> Data Quality Engine
-> Data Quality Snapshot
-> Anomaly Engine
-> Alert Event jika perlu
-> Dashboard badge/warning
-> Report data integrity section
```

Aturan:

- low quality data harus terlihat,
- anomaly harus tercatat,
- report final harus menyebut missing hours/anomaly yang mempengaruhi angka,
- estimated data harus diberi label.

---

## 5.5 Alur Report Generation

```text
Admin Tenant klik Generate Report
-> backend cek permission report.generate
-> backend cek tenant/campaign access
-> create report_generation_job
-> worker membuat data snapshot
-> worker menghitung KPI final
-> worker render chart
-> worker render PDF/Excel/CSV
-> worker upload ke object storage
-> worker membuat signed export metadata
-> status report menjadi Generated / Waiting Review
-> Admin review
-> Admin approve
-> Client dapat melihat/download report approved
```

Aturan:

- Client tidak boleh melihat draft report,
- report final harus approved sebelum dikirim,
- report download harus signed dan audited,
- data report harus snapshot, bukan query live berubah-ubah.

---

## 5.6 Alur Proof of Display

```text
Teknisi upload proof
-> backend cek permission proof.upload
-> file disimpan ke object storage
-> proof status pending_review
-> Admin Tenant review
-> approve/reject dengan catatan
-> approved proof tampil di Client Dashboard dan Report
-> audit log mencatat semua aksi
```

Aturan:

- Client hanya melihat approved proof,
- Owner Tenant hanya melihat ringkasan proof,
- proof harus punya timestamp dan GPS jika tersedia,
- proof campaign final harus tercantum di report.

---

## 6. Data Layer Architecture

## 6.1 Database Utama

Rekomendasi production: PostgreSQL.

Alasan:

- kuat untuk relational SaaS,
- mendukung constraint dan transaction,
- mendukung indexing kuat,
- mendukung partitioning untuk event besar,
- ekosistem matang.

Tabel inti:

```text
tenants
tenant_profiles
subscription_plans
tenant_subscriptions
tier_feature_limits
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
live_stream_sessions
camera_health_logs
camera_snapshots
alert_events
data_quality_snapshots
anomaly_events
maintenance_logs
maintenance_tickets
audit_logs
system_settings
```

Aturan database:

- semua tabel operasional tenant wajib punya `tenant_id`,
- semua foreign key wajib jelas,
- semua tabel besar wajib punya index sesuai query,
- event table wajib dipikirkan partitioning,
- delete data penting harus soft delete,
- report final harus immutable/versioned,
- semua formula/methodology harus versioned.

---

## 6.2 Queue / Job Broker

Queue digunakan untuk pekerjaan berat dan asynchronous.

Jenis job:

```text
ingestion.validate_event
aggregation.hourly
aggregation.daily
aggregation.monthly
data_quality.calculate
anomaly.detect
report.generate_pdf
report.generate_excel
report.generate_csv
notification.send
camera.health_check
export.cleanup_expired
```

Aturan queue:

- job harus idempotent,
- job harus punya retry policy,
- job gagal harus tercatat,
- job payload harus membawa tenant_id,
- worker harus menerapkan tenant guard logic,
- jangan memasukkan secret/raw RTSP ke job payload kecuali encrypted reference.

---

## 6.3 Cache Layer

Cache dapat digunakan untuk:

- session,
- permission snapshot,
- dashboard KPI singkat,
- live stream session state,
- rate limiting,
- feature flags.

Rekomendasi: Redis atau compatible cache.

Aturan:

- cache bukan sumber kebenaran utama,
- cache harus tenant-scoped,
- jangan menyimpan secret plain text,
- invalidasi cache saat permission/role berubah,
- critical permission tetap harus bisa divalidasi aman.

---

## 6.4 Object Storage

Object storage digunakan untuk file besar.

File tidak boleh disimpan di database sebagai blob besar kecuali metadata.

Metadata file disimpan di database:

```text
file_id
tenant_id
resource_type
resource_id
storage_path
mime_type
size_bytes
checksum
created_by
created_at
visibility
```

Download harus signed.

---

## 7. API Architecture

## 7.1 API Families

Endpoint utama:

```text
/auth/*
/platform/*
/tenants/*
/users/*
/roles/*
/billboards/*
/cameras/*
/live-stream/*
/campaigns/*
/clients/*
/proof-display/*
/analytics/*
/reports/*
/alerts/*
/maintenance/*
/audit-logs/*
/profile/*
```

Setiap endpoint wajib:

- authenticated kecuali endpoint publik tertentu,
- tenant-filtered jika resource tenant,
- permission-checked,
- request-validated,
- response-filtered,
- audited untuk aksi penting.

---

## 7.2 API Response Standard

Response sukses:

```json
{
  "success": true,
  "data": {},
  "meta": {
    "request_id": "req_xxx"
  }
}
```

Response error:

```json
{
  "success": false,
  "error": {
    "code": "FORBIDDEN",
    "message": "You do not have permission to access this resource.",
    "request_id": "req_xxx"
  }
}
```

Larangan:

- jangan bocorkan stack trace ke client,
- jangan bocorkan raw SQL error,
- jangan bocorkan path internal storage,
- jangan bocorkan raw RTSP.

---

## 7.3 Tenant Guard Pattern

Setiap query resource tenant wajib seperti ini secara konsep:

```text
WHERE tenant_id = current_context.tenant_id
```

Untuk Client:

```text
WHERE tenant_id = current_context.tenant_id
AND client_id = current_context.client_id
AND campaign_id IN allowed_campaign_ids
```

Untuk Owner Platform support mode:

```text
require explicit support_scope_tenant_id
require support/audit permission
write audit log
limit action to read/support unless elevated permission exists
```

Tidak boleh ada endpoint tenant resource tanpa tenant filter.

---

## 8. Security Architecture

## 8.1 Security Boundary

Boundary utama:

- public internet,
- web app,
- API gateway,
- backend services,
- workers,
- database,
- object storage,
- stream proxy,
- edge devices.

Setiap boundary harus punya kontrol:

- authentication,
- authorization,
- encryption,
- logging,
- rate limiting jika relevan.

---

## 8.2 Secret Management

Secret yang wajib dilindungi:

- database credentials,
- object storage keys,
- stream credentials,
- RTSP credentials,
- JWT/session secret,
- API keys,
- SMTP/notification credentials.

Aturan:

- secret tidak boleh masuk Git,
- secret tidak boleh tampil di frontend,
- secret tidak boleh ditulis di README publik,
- stream URL harus encrypted at rest,
- log tidak boleh mencetak secret.

---

## 8.3 Rate Limiting

Endpoint yang harus rate-limited:

- login,
- forgot password,
- live stream session create,
- export/download,
- report generate,
- analytics heavy query,
- ingestion endpoint.

Tujuan:

- mencegah brute force,
- mencegah abuse report generation,
- mencegah live stream dibuka liar,
- menjaga biaya compute.

---

## 9. Observability Architecture

## 9.1 Logging

Setiap service wajib menghasilkan structured log.

Field minimum:

```text
timestamp
level
service
request_id
tenant_id
user_id
job_id
resource_type
resource_id
action
message
```

Log tidak boleh berisi:

- password,
- token penuh,
- raw RTSP,
- secret,
- data sensitif yang tidak perlu.

---

## 9.2 Metrics

Metrics yang wajib dipantau:

- API request rate,
- API error rate,
- response latency,
- DB query latency,
- queue depth,
- job failure rate,
- report generation time,
- stream session count,
- camera online/offline count,
- ingestion event rate,
- aggregation delay,
- AI processing latency,
- storage usage,
- tenant usage per tier.

---

## 9.3 Tracing

Gunakan `request_id` dan `job_id` untuk melacak alur:

```text
frontend request -> API -> service -> DB/job -> worker -> storage -> notification
```

Report generation harus punya trace utuh dari request sampai file final.

---

## 10. Deployment Architecture

## 10.1 Environment

Minimal environment:

```text
local
development
staging
production
```

Aturan:

- production data tidak boleh dipakai sembarangan di local,
- staging harus mirip production,
- local boleh memakai dummy/simulation data,
- dummy data wajib diberi label simulation.

---

## 10.2 Recommended Production Components

Rekomendasi komponen production:

```text
Web App
API Server
Worker Server
PostgreSQL Database
Redis / Queue Broker
Object Storage
Stream Proxy Server
AI Processing Worker
Monitoring Stack
Backup System
```

Untuk MVP kecil, beberapa service boleh digabung secara deployment, tetapi boundary logis tetap harus dipertahankan.

Contoh MVP deployment:

```text
1 server app: web + API
1 worker process: report + aggregation + data quality
1 PostgreSQL
1 Redis
1 object storage compatible service
1 stream proxy
```

Jangan terlalu cepat membuat microservices jika tim belum siap. Microservices tanpa disiplin itu bukan arsitektur modern, itu arisan error antar-container.

---

## 10.3 Backup & Recovery

Wajib backup:

- database,
- object storage metadata,
- report exports,
- proof files,
- configuration,
- audit logs.

Aturan:

- backup database harian minimal,
- retention backup jelas,
- restore test berkala,
- object storage lifecycle policy,
- audit log retention mengikuti kebijakan bisnis/legal.

---

## 11. Scalability Strategy

## 11.1 Tahap MVP

Fokus:

- multi-tenant benar,
- role/permission benar,
- dashboard dasar,
- camera health,
- traffic summary,
- report generation,
- audit log.

Event volume masih bisa dikelola dengan PostgreSQL + partitioning ringan.

## 11.2 Tahap Growth

Tambahan:

- worker dipisah,
- queue lebih serius,
- cache dashboard,
- object storage production,
- stream proxy dedicated,
- AI worker scale-out,
- data partitioning lebih matang.

## 11.3 Tahap Enterprise

Tambahan:

- time-series/event store khusus,
- data warehouse,
- multi-region storage,
- advanced observability,
- tenant-level resource quota,
- advanced anomaly model,
- SLA monitoring.

---

## 12. Data Retention Policy

Retention awal dapat disesuaikan tier.

Contoh:

| Data | Starter | Growth | Pro | Enterprise |
|---|---:|---:|---:|---:|
| Raw traffic events | 30 hari | 90 hari | 180 hari | Custom |
| Hourly summary | 12 bulan | 24 bulan | 36 bulan | Custom |
| Daily/monthly summary | 24 bulan | 36 bulan | 60 bulan | Custom |
| Report PDF | 12 bulan | 24 bulan | 36 bulan | Custom |
| Audit log | 12 bulan | 24 bulan | 36 bulan | Custom |
| Proof photos | 12 bulan | 24 bulan | 36 bulan | Custom |

Aturan:

- summary boleh disimpan lebih lama dari raw event,
- report final tidak boleh hilang selama masih dalam masa retention,
- data deletion harus tenant-scoped,
- delete policy harus masuk audit.

---

## 13. Report Architecture Detail

Report bukan sekadar export tampilan dashboard.

Report adalah dokumen final dengan:

- data snapshot,
- formula version,
- methodology version,
- approval status,
- checksum,
- file export,
- audit trail.

Struktur report ideal:

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

Arsitektur report wajib mendukung:

- PDF executive report,
- Excel raw data,
- CSV hourly traffic,
- report history,
- approval workflow,
- signed download,
- client access control.

---

## 14. Formula & Methodology Versioning

Formula dan methodology tidak boleh hard-coded sembarangan.

Wajib ada:

```text
formula_version
methodology_version
ai_model_version
counting_rule_version
exposure_rule_version
```

Jika formula berubah:

- jangan overwrite report lama,
- buat versi baru,
- re-aggregation harus jelas,
- report lama tetap bisa dibuka dengan formula lamanya,
- audit log mencatat perubahan.

---

## 15. Simulation Mode Architecture

Untuk MVP awal, sistem boleh memakai simulation data.

Namun wajib diberi label:

```text
SIMULATION
DUMMY
NOT REAL CAMERA DATA
```

Aturan simulation:

- simulation data tidak boleh masuk report final client tanpa label,
- simulation data tidak boleh dicampur dengan real data tanpa status,
- dashboard harus menampilkan badge simulation,
- API harus mengembalikan `data_status = simulation`,
- export harus mencantumkan catatan simulation.

---

## 16. Failure Handling

## 16.1 Camera Offline

Jika camera offline:

```text
heartbeat missing
-> camera status offline
-> alert created
-> data quality affected
-> missing hours recorded
-> dashboard warning
-> report data integrity note
```

## 16.2 Ingestion Failure

Jika ingestion gagal:

- retry jika error sementara,
- reject jika schema invalid,
- quarantine jika payload mencurigakan,
- alert jika event hilang terus-menerus.

## 16.3 Aggregation Failure

Jika aggregation gagal:

- job retry,
- summary status failed/stale,
- dashboard menampilkan stale warning,
- report generation diblokir jika summary tidak valid.

## 16.4 Report Failure

Jika report gagal:

- status `failed`,
- error internal dicatat,
- user mendapat pesan aman,
- retry manual/automatic jika memungkinkan,
- audit log tetap dibuat.

---

## 17. Testing Architecture

Testing wajib mencakup:

### 17.1 Unit Test

- formula vehicle count,
- CPV/CPI,
- estimated exposure,
- data quality score,
- permission guard,
- tenant guard,
- report status transition.

### 17.2 Integration Test

- login -> dashboard,
- tenant isolation,
- client campaign isolation,
- report generation,
- live stream session creation,
- proof approval,
- aggregation job.

### 17.3 Security Test

- user tenant A tidak bisa akses tenant B,
- client A tidak bisa akses campaign client B,
- Owner Tenant tidak bisa create/update/delete,
- raw RTSP tidak muncul di response,
- suspended tenant diblokir,
- signed URL expired tidak bisa dipakai.

### 17.4 Data Pipeline Test

- duplicate event tidak menggandakan count,
- low confidence event diberi status sesuai rule,
- missing hours menurunkan quality score,
- anomaly flag muncul untuk sudden drop/spike,
- re-aggregation menghasilkan summary sesuai formula version.

### 17.5 Report Test

- report memakai snapshot,
- PDF/Excel/CSV berhasil dibuat,
- report approved baru terlihat client,
- report download masuk audit log,
- report lama tidak berubah setelah formula baru.

---

## 18. Developer / Agent Rules

Aturan keras untuk developer/agent:

1. Jangan membuat API tenant tanpa tenant guard.
2. Jangan membuat query tenant tanpa `tenant_id` filter.
3. Jangan membuat UI role tanpa backend permission guard.
4. Jangan expose raw RTSP ke frontend.
5. Jangan menyimpan stream credential plain text.
6. Jangan membuat report tanpa snapshot.
7. Jangan membuat export tanpa signed URL dan audit log.
8. Jangan membuat angka analytics tanpa source dan status.
9. Jangan membuat estimated exposure tanpa formula version.
10. Jangan menyembunyikan low data quality.
11. Jangan membuat Owner Tenant bisa create/edit/delete.
12. Jangan membuat Client bisa melihat campaign client lain.
13. Jangan membuat background worker yang melewati tenant isolation.
14. Jangan membuat dummy data tanpa label simulation.
15. Jangan membuat dashboard cantik tapi permission bocor.

Aturan positif:

1. Mulai dari flow yang bisa dijual dan diaudit.
2. Gunakan summary table untuk dashboard cepat.
3. Gunakan queue untuk pekerjaan berat.
4. Gunakan snapshot untuk report final.
5. Gunakan audit log untuk aksi penting.
6. Gunakan signed export untuk download.
7. Gunakan data quality badge di dashboard dan report.
8. Gunakan explicit permission untuk semua aksi.
9. Gunakan structured logging dan request_id.
10. Pisahkan raw data, validated data, estimated data, dan final report.

---

## 19. MVP Architecture Scope

MVP yang sehat harus mencakup:

### Core SaaS

- auth,
- tenant,
- role/permission,
- tenant guard,
- user management,
- audit log dasar.

### Core Billboard Business

- billboard,
- billboard face,
- camera,
- client,
- campaign,
- proof display.

### Dashboard & Analytics

- Admin Tenant dashboard,
- Owner Tenant dashboard read-only,
- Client dashboard mobile-first,
- analytics summary,
- vehicle composition,
- day/week/month/custom filter.

### Device & Live View

- camera registry,
- heartbeat,
- camera health,
- secure live stream session,
- no raw RTSP leak.

### Report

- report generation job,
- PDF export,
- Excel/CSV export,
- report status,
- approval,
- signed download.

### AI Data Minimum

- event ingestion,
- hourly summary,
- daily summary,
- vehicle classes,
- confidence score,
- data quality score,
- estimated exposure,
- anomaly flag basic.

---

## 20. Non-Goals Architecture Tahap Awal

Tidak dikerjakan di awal:

- facial recognition,
- demographic detection,
- membaca mata manusia,
- prediksi sales langsung,
- dynamic pricing kompleks,
- marketplace billboard publik,
- mobile app native,
- multi-country support,
- real-time bidding,
- fully distributed microservices,
- AI model training platform sendiri,
- BI warehouse kompleks.

Fokus awal: sistem SaaS billboard yang benar, aman, bisa menghitung traffic, bisa menghasilkan report, dan bisa dipertanggungjawabkan.

---

## 21. Acceptance Criteria Architecture

Arsitektur dianggap layak MVP production jika:

1. Tenant A tidak bisa mengakses data Tenant B.
2. Client hanya bisa melihat campaign miliknya.
3. Owner Tenant benar-benar read-only.
4. Admin Tenant hanya berkuasa di tenant sendiri.
5. Owner Platform mengelola tenant/tier tanpa menjadi admin liar semua tenant.
6. Semua API penting punya auth, tenant guard, permission guard.
7. Raw RTSP tidak pernah muncul di frontend.
8. Live view memakai signed session dan audit log.
9. Traffic event punya source, status, confidence, dan rule version.
10. Summary punya formula version dan data quality score.
11. Report final memakai snapshot, bukan live mutable query.
12. Report download memakai signed URL dan audit log.
13. Proof display punya approval workflow.
14. Low quality/anomaly/estimated data tampil sebagai warning.
15. Background worker tetap tenant-scoped.
16. Audit log mencatat aksi penting dan failed access attempt.
17. Dashboard tetap cepat karena membaca summary, bukan raw event berat.
18. Formula/methodology bisa dilacak di report.
19. Simulation data tidak pernah menyamar sebagai real data.
20. Sistem bisa dikembangkan ke scale lebih besar tanpa bongkar total pondasi.

---

## 22. Ringkasan Praktis untuk Codex / Developer

Bangun sistem dengan urutan arsitektur berikut:

```text
1. Auth + session
2. Tenant schema + tenant guard
3. RBAC + permission guard
4. Audit logger
5. Master tenant/client/billboard/camera/campaign
6. Camera health + heartbeat
7. Secure live stream session
8. Traffic event ingestion
9. Hourly/daily aggregation
10. Data quality + anomaly basic
11. Analytics query service
12. Proof display workflow
13. Report generation job
14. Signed export download
15. Final dashboard polish
```

Jangan lompat ke desain dashboard premium sebelum tenant guard, permission guard, dan audit log berdiri.

Aplikasi boleh bertumbuh pelan, tapi jangan lahir pincang. MVP yang kecil tapi benar jauh lebih berharga daripada dashboard megah yang role-nya bocor seperti ember tua.

---

## 23. Kesimpulan

Arsitektur platform ini harus memegang tiga kunci:

1. **SaaS isolation** — data antar-tenant tidak boleh bocor.
2. **Analytics integrity** — angka traffic harus punya sumber, formula, confidence, quality, dan audit trail.
3. **Secure media access** — live stream dan report export harus aman, signed, dan tercatat.

Jika tiga hal ini kuat, platform bisa tumbuh menjadi mesin laporan billboard yang meyakinkan client dan aman untuk perusahaan advertising.

Jika tiga hal ini lemah, aplikasi hanya akan menjadi dashboard cantik yang membawa angka tanpa tulang. Dan angka tanpa tulang itu mudah roboh ketika client mulai bertanya: “Ini hitungannya dari mana?”
