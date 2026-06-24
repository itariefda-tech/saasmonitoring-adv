# SECURITY_PRIVACY_ACCESS_CONTROL.md

# SaaS Billboard Monitoring & Report Platform

## Security, Privacy & Access Control Guide

---

## 1. Tujuan Dokumen

Dokumen ini mengunci aturan keamanan, privasi, session, role permission, tenant isolation, live streaming, RTSP protection, audit log, dan akses report untuk aplikasi **SaaS Billboard Monitoring & Report Platform**.

Dokumen ini wajib dibaca dan dipatuhi oleh agent, developer, UI designer, backend engineer, frontend engineer, DevOps, dan siapa pun yang membangun aplikasi ini.

Aplikasi ini bukan dashboard mainan. Ini platform SaaS multi-tenant yang menyimpan data perusahaan billboard, client, campaign, CCTV, traffic analytics, proof display, dan report bisnis.

Kalau security bolong, aplikasi bukan lagi produk SaaS. Ia berubah menjadi prasmanan data: siapa pun bisa ambil lauk tenant sebelah.

---

## 2. Ruang Lingkup Security

Security yang dikunci di dokumen ini mencakup:

* authentication,
* authorization,
* role-based access control,
* permission guard,
* tenant isolation,
* client campaign isolation,
* session management,
* live streaming security,
* RTSP protection,
* camera credential security,
* report download security,
* proof display file security,
* audit log,
* privacy,
* API security,
* frontend access control,
* backend access control,
* database access control,
* logging,
* monitoring,
* backup,
* incident response,
* development rules untuk agent.

---

## 3. Prinsip Utama Security

Prinsip wajib:

1. Semua user wajib login.
2. Semua request API wajib authenticated kecuali endpoint public yang memang didefinisikan.
3. Semua endpoint wajib dicek permission.
4. Semua data tenant wajib difilter berdasarkan `tenant_id`.
5. Client hanya boleh melihat campaign miliknya.
6. Owner Tenant hanya monitoring/read-only.
7. Admin Tenant hanya berkuasa di tenant sendiri.
8. Owner Platform SaaS mengelola platform, bukan otomatis bebas mengintip data bisnis tenant tanpa alasan audit/support.
9. Raw RTSP tidak boleh tampil di frontend client.
10. Password tidak boleh disimpan plain text.
11. Credential tidak boleh ditulis di dokumentasi.
12. Report download wajib diaudit.
13. Live streaming wajib diaudit.
14. Export data wajib diaudit.
15. Semua aksi penting wajib masuk audit log.
16. Semua data estimated wajib diberi label.
17. Semua data traffic wajib punya sumber, confidence, dan data quality context.
18. Jangan membangun UI cantik tetapi permission bocor.

Prinsip galak:

> Security bukan dekorasi setelah aplikasi jadi. Security adalah pagar sebelum rumah dibangun.

---

## 4. Benchmark Security dari Manual ADX

Manual dashboard traffic counting menjadi benchmark fitur, bukan benchmark security final.

Fitur yang boleh dijadikan acuan:

* login dashboard,
* dashboard setelah login,
* menu analytics,
* filter day/week/month,
* filter lokasi,
* filter periode,
* export PDF/CSV/Excel,
* menu profile,
* logout,
* device list,
* live streaming,
* pembatasan sesi streaming,
* dokumentasi lokasi.

Hal yang tidak boleh ditiru:

* credential ditulis di dokumen,
* password default sama dengan email,
* raw RTSP link ditampilkan ke user,
* user diminta copy link RTSP ke VLC,
* live streaming dibuka tanpa token aman,
* live streaming dibuka tanpa audit log lengkap,
* pembatasan sesi hanya mengandalkan logout manual,
* akses live view tidak dikunci per role/campaign.

Aplikasi kita harus naik kelas:

```text
Manual Dashboard Basic
+
Multi-Tenant SaaS Security
+
RBAC / Permission Matrix
+
Tenant Isolation
+
Secure Stream Proxy
+
Signed Stream Token
+
Audit Log
+
Report Download Control
+
Data Privacy
```

---

## 5. Threat Model

Aplikasi harus dirancang dengan asumsi ada potensi serangan berikut:

### 5.1 Cross-Tenant Data Leak

Risiko:

* Tenant A melihat data Tenant B.
* Admin Tenant A mengakses billboard Tenant B.
* Client Brand A melihat campaign Client Brand B.
* Report tenant lain bisa didownload karena URL mudah ditebak.

Mitigasi:

* semua tabel operasional wajib punya `tenant_id`,
* semua query wajib filter `tenant_id`,
* semua endpoint wajib memakai tenant guard,
* semua report download wajib cek ownership,
* semua object storage path tidak boleh public,
* gunakan signed URL dengan expiry pendek.

---

### 5.2 IDOR — Insecure Direct Object Reference

Risiko:

User mengganti URL seperti:

```text
/reports/100/download
```

menjadi:

```text
/reports/101/download
```

lalu berhasil download report milik client lain.

Mitigasi:

* jangan percaya ID dari URL,
* setiap akses object harus dicek:

  * user id,
  * role,
  * tenant id,
  * client id,
  * campaign id,
  * permission,
  * status object.

Wajib ada helper:

```text
canAccessReport(user, report_id)
canAccessCampaign(user, campaign_id)
canAccessCamera(user, camera_id)
canAccessBillboard(user, billboard_id)
```

---

### 5.3 Credential Leak

Risiko:

* password ditulis di PDF/manual,
* password default sama dengan email,
* RTSP credential bocor,
* token live stream bocor,
* API key masuk GitHub,
* credential muncul di log.

Mitigasi:

* password wajib hash,
* credential tidak boleh ditulis di dokumen,
* raw RTSP terenkripsi di database,
* secret disimpan di environment/secret manager,
* log sanitizer wajib aktif,
* token harus expiry,
* credential rotation wajib tersedia.

---

### 5.4 Raw RTSP Exposure

Risiko:

Client melihat:

```text
rtsp://10.x.x.x:8554/streaming
```

atau URL kamera asli.

Bahaya:

* stream bisa dibuka di luar aplikasi,
* akses tidak tercatat,
* user bisa share link,
* kamera bisa diserang,
* tenant/client lain bisa melihat feed.

Mitigasi:

* raw RTSP hanya disimpan encrypted,
* frontend tidak pernah menerima raw RTSP,
* live view harus lewat secure stream proxy,
* user hanya menerima signed stream session URL,
* session live view punya expiry,
* session live view masuk audit log.

---

### 5.5 Report Data Leak

Risiko:

* PDF report client A dikirim ke client B.
* CSV/Excel mentah berisi data tenant lain.
* Link download report tidak expired.
* File report public di storage.

Mitigasi:

* report file private by default,
* download via authenticated endpoint,
* signed URL expiry pendek,
* report access dicek by role + tenant + campaign,
* download masuk audit log,
* report file diberi watermark jika diperlukan,
* export data raw hanya untuk role tertentu.

---

### 5.6 Session Hijacking

Risiko:

* session token dicuri,
* browser ditinggal login,
* refresh token tidak pernah expired,
* user lama masih aktif setelah password diganti.

Mitigasi:

* access token pendek,
* refresh token rotation,
* HTTP-only secure cookie jika memakai cookie,
* logout invalidates refresh token,
* password change revokes all sessions,
* idle timeout,
* absolute session timeout,
* device/session list untuk admin.

---

### 5.7 Unauthorized Live View

Risiko:

* Client melihat live view padahal tidak diizinkan.
* Owner Tenant melihat semua kamera detail padahal hanya butuh summary.
* Teknisi melihat kamera tenant lain.
* Sales membuka feed yang tidak relevan.

Mitigasi:

* live view punya permission khusus,
* default live view untuk Client adalah disabled,
* Client hanya bisa live view kamera campaign miliknya jika diaktifkan,
* live stream session harus dicatat,
* limit 1 active session per user/camera bisa diterapkan.

---

## 6. Role Security Boundary

Role utama:

```text
Owner Platform SaaS
Admin Tenant
Owner Tenant
Sales Tenant
Teknisi Lapangan
Client / Brand
Viewer / Report Viewer
```

---

## 7. Permission Matrix Security

| Area / Fitur           |  Owner Platform SaaS |     Admin Tenant |       Owner Tenant |         Sales |         Teknisi |                 Client |
| ---------------------- | -------------------: | ---------------: | -----------------: | ------------: | --------------: | ---------------------: |
| Tenant Management      |                 Full |               No |                 No |            No |              No |                     No |
| Subscription/Tier      |                 Full |               No |                 No |            No |              No |                     No |
| User Management Tenant |        Support/Audit |      Full Tenant |                 No |            No |              No |                     No |
| Billboard              |        Support/Audit |      Full Tenant |          Read Only |  Read/Limited |   Assigned Read |          Campaign Only |
| Camera Config          |        Support/Audit |      Full Tenant |       Read Summary |    No/Limited | Update Assigned |                     No |
| Raw RTSP               |  Restricted Internal | Restricted Admin |                 No |            No |              No |                     No |
| Live View              |        Support/Audit |      Full Tenant | Summary / Optional |      Optional |        Assigned | Optional Campaign Only |
| Campaign               |        Support/Audit |      Full Tenant |          Read Only | Draft/Limited |              No |          Campaign Only |
| Proof Upload           |                   No |     Full/Approve |          Read Only |            No | Upload Assigned |                     No |
| Proof Approval         |                   No |              Yes |                 No |            No |              No |                     No |
| Traffic Analytics      | Global Usage / Audit |      Full Tenant |          Read Only |       Limited |  Technical Only |          Campaign Only |
| Report Generate        |           No/Support |              Yes |                 No |       Limited |              No |                     No |
| Report Review/Approve  |           No/Support |              Yes |                 No |            No |              No |                     No |
| Report Download        | Global Usage / Audit |      Full Tenant |         If Allowed |       Limited |      No/Limited |          Campaign Only |
| Audit Log Tenant       |   Platform + Support |           Tenant | No/Limited Summary |            No |     Own Actions |            Own Actions |
| System Settings        |        Full Platform |  Tenant Settings |                 No |            No |              No |                     No |

Catatan galak:

* Owner Tenant bukan Admin Tenant.
* Client bukan staff tenant.
* Teknisi bukan Sales.
* Sales bukan Admin.
* Owner Platform SaaS bukan dalih untuk mengintip data tenant sembarangan.
* Permission harus dicek di backend, bukan hanya sembunyi tombol di frontend.

---

## 8. Authentication Rules

### 8.1 Login

Endpoint:

```text
POST /auth/login
```

Wajib:

* email wajib unik secara sistem atau unik per tenant sesuai keputusan final,
* password wajib diverifikasi dengan hash aman,
* gagal login tidak boleh memberi info apakah email valid,
* login sukses menghasilkan session/token,
* login sukses masuk audit log,
* gagal login berkali-kali kena rate limit.

Response login tidak boleh mengandung:

* password,
* password hash,
* raw role permission internal yang berlebihan,
* raw RTSP,
* API key,
* secret.

---

### 8.2 Password Policy

Password minimal:

* minimal 10 karakter,
* kombinasi huruf besar, huruf kecil, angka, simbol disarankan,
* tidak boleh sama dengan email,
* tidak boleh sama dengan nama user,
* tidak boleh memakai password default permanen,
* password sementara wajib diganti saat first login.

Password storage:

* gunakan hash seperti Argon2id atau bcrypt,
* jangan simpan plain text,
* jangan log password,
* jangan kirim password via email setelah user dibuat.

---

### 8.3 First Login

Saat user dibuat oleh Admin Tenant:

* sistem mengirim invitation link,
* link punya expiry,
* user membuat password sendiri,
* password tidak ditentukan di dokumen/manual,
* user wajib accept terms jika diperlukan.

Flow:

```text
Admin create user
↓
System sends invite link
↓
User sets password
↓
User login
↓
Session created
↓
Audit log recorded
```

---

### 8.4 Forgot Password

Endpoint:

```text
POST /auth/forgot-password
POST /auth/reset-password
```

Aturan:

* response selalu generik,
* token reset password punya expiry pendek,
* token hanya sekali pakai,
* setelah reset password, semua session lama dicabut,
* aktivitas masuk audit log.

---

### 8.5 MFA / Two-Factor Authentication

MVP boleh optional, tetapi rancangan harus siap.

MFA wajib tersedia untuk:

* Owner Platform SaaS,
* Admin Tenant,
* user dengan akses report export,
* user dengan akses camera/live view,
* user dengan akses tenant settings.

MFA bisa menggunakan:

* authenticator app,
* email OTP sebagai fallback awal,
* passkey jika nanti sudah matang.

---

## 9. Session Management

### 9.1 Tipe Token

Rekomendasi:

* access token pendek,
* refresh token lebih panjang,
* refresh token rotation,
* revoke token saat logout,
* revoke token saat password diganti.

Jika memakai cookie:

* `HttpOnly`,
* `Secure`,
* `SameSite=Lax` atau `Strict`,
* CSRF protection untuk mutation request.

Jika memakai bearer token:

* simpan dengan hati-hati,
* hindari localStorage untuk token sensitif jika memungkinkan,
* refresh token jangan mudah diakses script frontend.

---

### 9.2 Session Timeout

Default:

```text
Access token: 15 menit
Refresh token: 7 sampai 30 hari sesuai policy
Idle timeout dashboard: 30 menit
Absolute session timeout: 12 jam
Live stream session: 10 menit default
```

Live stream session boleh diperpanjang dengan explicit user action, bukan diam-diam selamanya.

---

### 9.3 Logout

Endpoint:

```text
POST /auth/logout
```

Wajib:

* revoke current refresh token,
* hapus cookie/token client,
* catat audit log,
* live stream session aktif milik user ditutup,
* redirect ke login.

---

### 9.4 Concurrent Session Policy

Perlu dibedakan:

* session login dashboard,
* session live stream.

Dashboard login:

* boleh multi device jika policy mengizinkan,
* Admin Tenant bisa melihat session aktif user tenant.

Live stream:

* default maksimal 1 active live stream per user per camera,
* paket/tier bisa menentukan limit viewer,
* concurrent live view wajib dicatat.

---

## 10. Tenant Isolation

### 10.1 Prinsip Tenant Isolation

Semua data tenant wajib terpisah secara logis.

Tabel operasional wajib punya:

```text
tenant_id
```

Minimal tabel yang wajib punya `tenant_id`:

```text
users
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
maintenance_logs
maintenance_tickets
notifications
alert_events
camera_health_logs
camera_snapshots
data_quality_snapshots
audit_logs
live_stream_sessions
```

---

### 10.2 Query Guard

Semua query tenant wajib seperti:

```sql
WHERE tenant_id = current_user.tenant_id
```

Dilarang:

```sql
SELECT * FROM reports WHERE id = :id;
```

Wajib:

```sql
SELECT * FROM reports
WHERE id = :id
AND tenant_id = :current_tenant_id;
```

Untuk Client wajib lebih ketat:

```sql
SELECT * FROM reports
WHERE id = :id
AND tenant_id = :current_tenant_id
AND client_id = :current_client_id;
```

Untuk campaign:

```sql
SELECT * FROM campaigns
WHERE id = :id
AND tenant_id = :current_tenant_id
AND client_id = :current_client_id;
```

---

### 10.3 Owner Platform SaaS Access

Owner Platform SaaS boleh:

* mengelola tenant,
* mengelola paket/tier,
* melihat status subscription,
* melihat system health,
* melihat usage aggregate,
* melihat audit log platform,
* melakukan support/audit resmi.

Owner Platform SaaS tidak otomatis boleh:

* melihat detail campaign client tanpa alasan,
* download report tenant tanpa audit reason,
* membuka live stream tenant tanpa support ticket/reason,
* mengubah data operasional tenant sembarangan.

Support access harus memakai:

```text
support_mode = true
reason_required = true
audit_log_required = true
time_limited = true
read_only_default = true
```

---

## 11. Role-Based Access Control

### 11.1 Permission Tidak Boleh Hanya Sidebar

Frontend boleh menyembunyikan menu, tetapi backend tetap wajib menjadi benteng utama.

Salah:

```text
Client tidak melihat tombol download report, berarti aman.
```

Benar:

```text
Endpoint download report tetap cek permission, tenant, client, campaign, status report, dan audit log.
```

---

### 11.2 Permission Naming

Gunakan permission eksplisit.

Contoh:

```text
tenant.manage
tenant.view
users.manage
users.view
billboards.create
billboards.update
billboards.delete
billboards.view
cameras.manage
cameras.view
cameras.live_view
cameras.live_view_client
campaigns.manage
campaigns.view
proof.upload
proof.approve
proof.view
analytics.view
analytics.export
reports.generate
reports.review
reports.approve
reports.download_pdf
reports.download_excel
reports.download_csv
reports.view_history
audit.view
settings.manage
```

---

### 11.3 Backend Guard Pattern

Setiap endpoint wajib punya guard:

```text
requireAuth()
requireTenant()
requirePermission(permission_name)
requireResourceAccess(resource_type, resource_id)
auditIfSensitive(action)
```

Contoh flow:

```text
Request masuk
↓
Validate token
↓
Load user
↓
Check active tenant/subscription
↓
Check role
↓
Check permission
↓
Check resource ownership
↓
Execute action
↓
Write audit log
↓
Return response
```

---

## 12. Client Campaign Isolation

Client adalah pihak eksternal penyewa billboard.

Client hanya boleh melihat:

* campaign miliknya,
* billboard yang terkait campaign miliknya,
* traffic analytics campaign miliknya,
* proof display yang sudah approved,
* report campaign miliknya,
* live view jika permission campaign mengizinkan.

Client tidak boleh melihat:

* data client lain,
* report client lain,
* raw traffic semua tenant,
* internal dashboard tenant,
* billing tenant,
* user tenant,
* konfigurasi kamera,
* raw RTSP,
* proof yang belum approved,
* audit log internal.

Query client wajib filter:

```text
tenant_id
client_id
campaign_id
permission
status
```

---

## 13. Owner Tenant Read-Only Enforcement

Owner Tenant adalah role monitoring-only.

Owner Tenant boleh:

* melihat executive dashboard,
* melihat KPI tenant,
* melihat campaign summary,
* melihat billboard performance,
* melihat CCTV health summary,
* melihat report jika diizinkan,
* download report jika diizinkan,
* melihat insight bisnis.

Owner Tenant tidak boleh:

* create,
* update,
* delete,
* approve,
* manage user,
* manage role,
* manage camera,
* manage subscription,
* manage tenant setting,
* upload proof,
* edit report,
* generate report jika policy memutuskan hanya Admin Tenant.

Backend wajib menolak mutation request dari Owner Tenant.

HTTP response:

```text
403 Forbidden
```

Pesan:

```text
You do not have permission to perform this action.
```

Jangan beri pesan yang membuka struktur internal.

---

## 14. Live Streaming Security

### 14.1 Prinsip Live View

Live View adalah fitur sensitif.

Alasan:

* menampilkan kondisi real-time jalan/lokasi,
* bisa mengandung kendaraan, orang, area sekitar,
* terkait perangkat CCTV,
* raw stream bisa disalahgunakan jika bocor.

Live View tidak boleh diperlakukan seperti embed video biasa.

---

### 14.2 Raw RTSP Policy

Dilarang keras:

* menampilkan raw RTSP di frontend,
* mengirim raw RTSP ke browser,
* menyalin RTSP ke clipboard user,
* meminta user membuka RTSP via VLC,
* menyimpan RTSP plain text,
* menaruh RTSP di log,
* menaruh RTSP di screenshot dokumentasi,
* menaruh RTSP di PDF manual.

Raw RTSP hanya boleh digunakan oleh:

* backend stream service,
* edge processing service,
* secure streaming proxy,
* internal worker.

---

### 14.3 Secure Stream Proxy

Arsitektur live stream:

```text
Camera / Edge Device
↓
Internal RTSP / Device Tunnel
↓
Streaming Proxy
↓
Tokenized Web Stream
↓
Secure Web Player
↓
Authorized User
```

Frontend menerima:

```text
/live-stream/play/:session_id?token=SIGNED_TOKEN
```

Frontend tidak menerima:

```text
rtsp://camera-ip:8554/streaming
```

---

### 14.4 Live Stream Session

Tabel minimal:

```text
live_stream_sessions
- id
- tenant_id
- user_id
- camera_id
- campaign_id nullable
- role_at_access
- session_token_hash
- started_at
- expires_at
- ended_at
- ip_address
- user_agent
- status
- end_reason
```

Status:

```text
active
expired
ended_by_user
revoked
replaced_by_new_session
blocked
```

---

### 14.5 Live Stream Token

Token live stream wajib:

* signed,
* time-limited,
* scoped to camera/session,
* tidak reusable setelah expiry,
* tidak berisi raw RTSP,
* tidak bisa dipakai untuk kamera lain,
* dicabut saat logout,
* dicabut saat permission berubah.

Isi token minimal:

```text
session_id
user_id
tenant_id
camera_id
expires_at
scope = live_stream.view
```

---

### 14.6 Live Stream Timeout

Default:

```text
Live stream session duration: 10 menit
```

Sebelum expired, tampilkan:

```text
Sesi live view akan berakhir dalam 60 detik.
[Perpanjang Sesi]
```

Perpanjangan wajib:

* cek token masih valid,
* cek permission masih valid,
* update audit log,
* tidak otomatis infinite.

---

### 14.7 Live View Permission by Role

Admin Tenant:

* boleh live view semua kamera tenant.

Owner Tenant:

* default melihat status summary,
* live view optional jika diizinkan tenant policy.

Sales:

* default tidak boleh,
* boleh jika campaign/sales assignment mengizinkan.

Teknisi:

* boleh live view kamera assigned untuk maintenance.

Client:

* default tidak boleh,
* boleh hanya kamera yang terkait campaign miliknya dan hanya jika permission aktif.

Owner Platform SaaS:

* boleh support view hanya dengan reason/support mode.

---

### 14.8 Live View Audit Log

Setiap akses live view wajib audit.

Data audit:

```text
action = live_stream.open
tenant_id
user_id
role
camera_id
billboard_id
campaign_id nullable
ip_address
user_agent
started_at
expires_at
status
reason nullable
```

Saat ditutup:

```text
action = live_stream.close
session_id
ended_at
duration_seconds
end_reason
```

---

## 15. Camera & Device Credential Security

### 15.1 Camera Credential Storage

Data sensitif:

* RTSP URL,
* camera username,
* camera password,
* device token,
* API key edge device,
* VPN/tunnel credential.

Aturan:

* simpan encrypted at rest,
* jangan tampilkan di frontend,
* jangan tampilkan full value di admin UI,
* hanya tampilkan masked value jika perlu.

Contoh masked:

```text
rtsp://********:****@10.200.*.*:8554/streaming
```

---

### 15.2 Camera Registration

Setiap camera/device wajib punya:

```text
camera_id
tenant_id
billboard_id
device_id
stream_url_encrypted
stream_proxy_url
status
last_heartbeat
health_score
created_by
created_at
updated_at
```

Camera tidak boleh aktif tanpa:

* tenant valid,
* billboard valid,
* permission setup,
* stream health check,
* audit log creation.

---

### 15.3 Device Token

Edge device wajib menggunakan device token.

Aturan:

* device token berbeda dari user token,
* token scoped per device,
* token bisa di-revoke,
* token bisa di-rotate,
* token tidak boleh dipakai akses dashboard,
* device hanya boleh kirim data untuk tenant/camera miliknya.

---

## 16. Report Download Security

### 16.1 Report Access Rule

Report adalah data bisnis sensitif.

Report hanya bisa diakses oleh:

* Admin Tenant untuk report tenant,
* Owner Tenant jika diizinkan,
* Sales jika report terkait campaign/client assigned,
* Client untuk campaign miliknya,
* Owner Platform SaaS hanya usage/support/audit dengan reason.

---

### 16.2 Report Download Flow

Flow aman:

```text
User klik download
↓
Backend cek authentication
↓
Backend cek permission
↓
Backend cek tenant_id
↓
Backend cek client_id/campaign_id jika user Client
↓
Backend cek status report
↓
Backend catat audit log
↓
Backend generate signed download URL / stream file
↓
File didownload
```

---

### 16.3 Report File Storage

Report file wajib private.

Dilarang:

```text
/public/reports/client-a-march.pdf
```

Wajib:

```text
private bucket/object storage
signed URL expiry 1-10 menit
download via backend guard
```

---

### 16.4 Report Status Control

Status report:

```text
draft
generated
waiting_review
approved
sent_to_client
archived
revoked
failed
```

Client hanya boleh download report dengan status:

```text
approved
sent_to_client
```

Client tidak boleh melihat:

```text
draft
waiting_review
failed
revoked
```

---

### 16.5 Report Audit Log

Setiap download report wajib log:

```text
action = report.download
report_id
report_type
file_type
tenant_id
client_id
campaign_id
user_id
role
ip_address
user_agent
downloaded_at
```

File type:

```text
pdf
excel
csv
json
```

---

### 16.6 Export CSV/Excel Security

CSV/Excel wajib diamankan dari formula injection.

Jika value dimulai dengan:

```text
=
+
-
@
```

maka escape value.

Contoh:

```text
'=SUM(...)
```

Export raw data hanya untuk role yang punya permission:

```text
analytics.export
reports.download_excel
reports.download_csv
```

Client default hanya PDF executive report, kecuali tenant mengizinkan CSV/Excel.

---

## 17. Proof of Display Security

Proof of Display bisa berupa:

* foto pemasangan,
* foto billboard,
* timestamp,
* GPS location,
* nama teknisi,
* checklist,
* approval admin,
* catatan maintenance.

Aturan:

* file proof private,
* client hanya melihat proof yang approved,
* proof pending hanya untuk Admin Tenant/Teknisi terkait,
* proof delete harus restricted,
* perubahan approval masuk audit log,
* metadata harus tersimpan.

Tabel minimal:

```text
proof_of_display
- id
- tenant_id
- campaign_id
- billboard_id
- uploaded_by
- approved_by nullable
- status
- file_url_private
- file_hash
- captured_at
- uploaded_at
- gps_latitude nullable
- gps_longitude nullable
- notes
```

Status:

```text
pending
approved
rejected
archived
```

---

## 18. Privacy Rules

### 18.1 Data yang Dikumpulkan

Aplikasi fokus pada:

* jumlah kendaraan,
* klasifikasi kendaraan,
* arah kendaraan,
* speed rate,
* congestion level,
* exposure zone,
* camera health,
* proof display,
* report campaign.

Aplikasi tidak dirancang untuk:

* facial recognition,
* identifikasi orang,
* identifikasi plat nomor untuk tracking individu,
* demographic detection,
* membaca mata manusia,
* mengklaim orang pasti melihat billboard.

---

### 18.2 Privacy by Design

Prinsip:

* kumpulkan data seperlunya,
* jangan simpan video mentah jika tidak diperlukan,
* jika menyimpan snapshot, beri retention policy,
* batasi akses live stream,
* batasi akses proof,
* jangan tampilkan raw stream ke publik,
* hindari fitur yang mengarah ke surveillance individu.

---

### 18.3 Data Retention

Rekomendasi awal:

```text
traffic_events raw: 90-180 hari
hourly summaries: 2-5 tahun
daily summaries: 2-5 tahun
monthly summaries: 5 tahun
reports: sesuai kontrak tenant
audit logs: minimal 1-3 tahun
live stream session logs: minimal 1 tahun
camera health logs: 1-2 tahun
snapshots/proof: sesuai durasi campaign + masa arsip
```

Retention final bisa diatur per tier dan kebutuhan kontrak.

---

### 18.4 Public Area Sensitivity

Karena CCTV menghadap area publik, sistem wajib hati-hati.

Aturan:

* dashboard tidak boleh menonjolkan individu,
* snapshot untuk proof/display harus relevan,
* jangan membuat fitur zoom identitas orang,
* jangan membuat klaim personal,
* data utama adalah traffic aggregate, bukan identitas manusia.

---

## 19. Audit Log

### 19.1 Aksi yang Wajib Diaudit

Wajib audit:

* login success,
* login failed threshold,
* logout,
* password changed,
* user created,
* user updated,
* user disabled,
* role changed,
* permission changed,
* tenant created,
* tenant suspended,
* subscription changed,
* billboard created/updated/deleted,
* camera created/updated/deleted,
* camera credential updated,
* live stream opened,
* live stream closed,
* report generated,
* report approved,
* report downloaded,
* report revoked,
* proof uploaded,
* proof approved/rejected,
* export CSV/Excel,
* support mode access,
* settings changed.

---

### 19.2 Audit Log Fields

Minimal:

```text
audit_logs
- id
- tenant_id nullable
- actor_user_id
- actor_role
- action
- resource_type
- resource_id
- target_user_id nullable
- before_snapshot nullable
- after_snapshot nullable
- ip_address
- user_agent
- request_id
- reason nullable
- created_at
```

Untuk data sensitif, `before_snapshot` dan `after_snapshot` harus disanitasi.

Jangan simpan:

* password,
* token,
* full RTSP URL,
* API key,
* secret.

---

### 19.3 Audit Log Integrity

Audit log tidak boleh mudah dihapus.

Aturan:

* hanya Owner Platform SaaS tertentu yang bisa melihat platform audit,
* Admin Tenant hanya bisa melihat audit tenant sendiri,
* audit log delete hard tidak tersedia di UI,
* archive boleh ada dengan policy khusus,
* perubahan audit log harus dicegah.

---

## 20. API Security

### 20.1 Endpoint Guard

Semua endpoint wajib:

```text
authenticated
role-checked
permission-checked
tenant-filtered
resource-access-checked
validated
rate-limited for sensitive endpoint
audited if sensitive
```

Endpoint sensitif:

```text
/auth/login
/auth/forgot-password
/auth/reset-password
/users
/roles
/permissions
/cameras
/live-stream/sessions
/reports/generate
/reports/:id/download
/proof-display/:id/approve
/platform/tenants
/platform/subscriptions
/platform/settings
```

---

### 20.2 Input Validation

Semua input wajib divalidasi:

* type,
* length,
* format,
* enum,
* date range,
* file type,
* file size,
* permission scope.

Jangan percaya input frontend.

---

### 20.3 Rate Limit

Rate limit wajib untuk:

* login,
* forgot password,
* reset password,
* live stream session create,
* report generate,
* report download,
* export CSV/Excel,
* upload proof.

Contoh awal:

```text
login: 5 attempts / 10 minutes / IP + email
forgot password: 3 attempts / hour / email
report generate: 10 requests / hour / user
live stream create: 20 requests / hour / user
```

---

### 20.4 CORS

CORS wajib restrict ke domain resmi.

Dilarang:

```text
Access-Control-Allow-Origin: *
```

Kecuali endpoint public statis yang benar-benar aman.

---

### 20.5 CSRF

Jika memakai cookie-based auth, mutation request wajib CSRF protection.

Endpoint mutation:

```text
POST
PUT
PATCH
DELETE
```

---

## 21. Frontend Security

Frontend wajib:

* tidak menyimpan raw RTSP,
* tidak menyimpan secret,
* tidak mengandalkan hidden button sebagai security,
* tidak menampilkan menu berdasarkan role saja tanpa backend guard,
* tidak menampilkan data tenant sebelum API authorized,
* tidak menyimpan token sensitif sembarangan,
* sanitize output user generated content,
* handle 401/403 dengan benar.

Jika API mengembalikan 403:

* tampilkan pesan singkat,
* jangan bocorkan resource exists/not exists.

Contoh:

```text
Anda tidak memiliki akses ke data ini.
```

---

## 22. Backend Security

Backend wajib:

* menjadi sumber kebenaran permission,
* enforce tenant isolation,
* enforce client campaign isolation,
* encrypt credential kamera,
* sign stream token,
* audit sensitive actions,
* sanitize logs,
* validate input,
* protect file download,
* protect report generation.

Backend tidak boleh:

* mengirim raw RTSP ke frontend,
* mengirim password hash,
* mengirim full credential,
* query tanpa tenant guard,
* export tanpa permission,
* download file tanpa audit.

---

## 23. Database Security

### 23.1 Required Security Tables

Tabel security minimal:

```text
users
roles
permissions
role_permissions
user_roles
user_sessions
password_reset_tokens
mfa_settings
tenant_access_policies
live_stream_sessions
report_download_logs
audit_logs
api_keys
device_tokens
camera_credentials
support_access_logs
```

---

### 23.2 Sensitive Field Encryption

Field yang wajib encrypted:

```text
cameras.stream_url
camera_credentials.username
camera_credentials.password
device_tokens.token_hash
api_keys.key_hash
refresh_tokens.token_hash
password_reset_tokens.token_hash
```

Token sebaiknya disimpan dalam bentuk hash, bukan token asli.

---

### 23.3 Soft Delete

Untuk data penting gunakan soft delete:

```text
deleted_at
deleted_by
delete_reason
```

Hard delete hanya untuk:

* data sementara,
* data sesuai retention policy,
* data yang memang wajib dihapus.

---

## 24. File Upload Security

Upload berlaku untuk:

* proof display,
* billboard photo,
* campaign creative,
* profile photo,
* report asset.

Aturan:

* file type whitelist,
* ukuran maksimal,
* scan file jika memungkinkan,
* rename file server-side,
* jangan gunakan nama file user mentah,
* simpan di private storage untuk file sensitif,
* generate thumbnail aman,
* cek permission sebelum akses file.

Allowed type awal:

```text
image/jpeg
image/png
image/webp
application/pdf
text/csv
application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
```

---

## 25. Logging & Monitoring

Log wajib:

* error API,
* auth event,
* permission denied,
* report generation failed,
* stream failure,
* camera offline,
* suspicious access,
* repeated failed login,
* cross-tenant access attempt.

Log tidak boleh berisi:

* password,
* token,
* raw RTSP,
* API key,
* full camera credential,
* sensitive signed URL.

Alert wajib:

* repeated failed login,
* tenant isolation violation attempt,
* report download spike,
* live stream abuse,
* camera credential update,
* admin role change,
* Owner Platform SaaS support mode access,
* storage private bucket accidentally public.

---

## 26. Backup & Disaster Recovery

Backup wajib mencakup:

* database,
* report metadata,
* report files,
* proof files,
* configuration,
* audit logs,
* camera/device config encrypted.

Backup harus:

* terenkripsi,
* punya retention,
* diuji restore,
* tidak public,
* akses dibatasi.

RPO/RTO awal:

```text
RPO: maksimal kehilangan data 24 jam untuk MVP
RTO: maksimal pemulihan 24-48 jam untuk MVP
```

Untuk production enterprise, target harus lebih ketat.

---

## 27. Incident Response

Jika terjadi insiden:

1. Deteksi.
2. Isolasi.
3. Cabut token/session terdampak.
4. Disable akses mencurigakan.
5. Rotasi credential jika perlu.
6. Review audit log.
7. Identifikasi tenant/client terdampak.
8. Restore layanan.
9. Buat postmortem.
10. Tambal sistem.

Insiden yang wajib dianggap serius:

* raw RTSP bocor,
* report client salah akses,
* tenant data leak,
* credential leak,
* admin account takeover,
* audit log hilang,
* public bucket exposure.

---

## 28. Security Rules untuk Report Builder

Report Builder wajib:

* mengambil data sesuai tenant,
* mengambil data sesuai campaign/client,
* menyimpan formula version,
* menyimpan methodology version,
* menyimpan generated_by,
* menyimpan generated_at,
* menyimpan data quality summary,
* menyimpan report status,
* audit generate/download/approve,
* memastikan report client hanya berisi data client tersebut.

Report tidak boleh:

* mencampur data tenant lain,
* mencampur campaign client lain,
* menghilangkan label estimated,
* menghilangkan warning data quality rendah,
* membuat impression seolah pasti dilihat manusia.

---

## 29. Security Rules untuk Analytics

Analytics wajib:

* tenant-filtered,
* campaign-filtered jika user Client,
* role-filtered,
* punya data source,
* punya quality context,
* punya confidence context.

Client hanya melihat:

```text
analytics for assigned campaign
```

Admin Tenant melihat:

```text
analytics for tenant
```

Owner Platform SaaS melihat:

```text
global usage aggregate
```

bukan default detail bisnis tenant.

---

## 30. Security Rules untuk AI CCTV Data

AI event wajib menyimpan:

```text
tenant_id
camera_id
billboard_id
campaign_id nullable
timestamp
vehicle_class
direction
confidence_score
data_quality_score
source
processing_version
```

Data traffic tidak boleh dibuat tanpa:

* camera source,
* timestamp,
* confidence,
* status raw/validated/estimated,
* tenant context.

Jika simulasi/dummy:

```text
is_simulation = true
```

Tampilan UI wajib memberi label:

```text
Simulation Data
Estimated Data
Validated Data
Low Quality Data
```

---

## 31. Environment Security

Pisahkan:

```text
development
staging
production
```

Aturan:

* production data tidak boleh dipakai sembarangan di development,
* secret per environment berbeda,
* debug mode mati di production,
* error stack trace tidak tampil ke user,
* seed data dummy diberi label,
* staging boleh pakai data dummy/sanitized.

---

## 32. Secret Management

Secret tidak boleh masuk:

* GitHub,
* README,
* screenshot,
* PDF manual,
* console log,
* frontend bundle.

Secret harus disimpan di:

* environment variable,
* secret manager,
* encrypted config.

Secret yang wajib dilindungi:

```text
DATABASE_URL
JWT_SECRET
REFRESH_TOKEN_SECRET
STREAM_SIGNING_SECRET
CAMERA_ENCRYPTION_KEY
OBJECT_STORAGE_KEY
SMTP_PASSWORD
API_KEYS
```

---

## 33. Minimal Security Acceptance Criteria

Sebelum fitur dianggap selesai, wajib lolos checklist:

### Auth

* user bisa login,
* password hash,
* password tidak muncul di response,
* logout revoke session,
* forgot password aman,
* rate limit login aktif.

### RBAC

* Client tidak bisa akses admin endpoint,
* Owner Tenant tidak bisa create/edit/delete,
* Teknisi tidak bisa akses data bisnis luas,
* Sales tidak bisa approve proof,
* Admin Tenant tidak bisa akses tenant lain.

### Tenant Isolation

* semua query punya tenant guard,
* IDOR test gagal,
* report tenant lain tidak bisa dibuka,
* campaign client lain tidak bisa dibuka.

### Live View

* raw RTSP tidak muncul di frontend,
* live stream via signed token,
* session expiry aktif,
* live view audit log tercatat,
* Client hanya bisa live view campaign miliknya jika diizinkan.

### Report

* report draft tidak bisa diakses client,
* report approved bisa didownload sesuai permission,
* download report masuk audit log,
* signed URL expired,
* CSV/Excel export permission checked.

### File Upload

* file type divalidasi,
* file private,
* proof pending tidak terlihat client,
* proof approved terlihat client sesuai campaign.

---

## 34. Forbidden Patterns untuk Agent

Agent dilarang membuat:

```text
if role == 'admin' tanpa tenant guard
```

Dilarang:

```text
SELECT * FROM reports WHERE id = :id
```

Dilarang:

```text
return camera.rtsp_url
```

Dilarang:

```text
password = email
```

Dilarang:

```text
localStorage.setItem('refresh_token', token)
```

Dilarang:

```text
/public/reports/report.pdf
```

Dilarang:

```text
download report tanpa audit log
```

Dilarang:

```text
Owner Tenant punya tombol edit/delete/approve
```

Dilarang:

```text
Client bisa melihat all campaigns
```

Dilarang:

```text
dummy traffic tanpa label simulation
```

Dilarang:

```text
impression = pasti dilihat orang
```

---

## 35. Recommended Middleware

Backend minimal punya middleware:

```text
authMiddleware
tenantMiddleware
permissionMiddleware
resourceAccessMiddleware
auditMiddleware
rateLimitMiddleware
validationMiddleware
errorHandlerMiddleware
```

Contoh urutan:

```text
authMiddleware
↓
tenantMiddleware
↓
permissionMiddleware
↓
resourceAccessMiddleware
↓
controller
↓
auditMiddleware
```

---

## 36. API Response Security

Untuk unauthorized:

```text
401 Unauthorized
```

Untuk forbidden:

```text
403 Forbidden
```

Untuk resource tenant lain:

```text
404 Not Found
```

Gunakan 404 untuk mencegah user tahu bahwa resource tenant lain memang ada.

Contoh:

```json
{
  "error": "Resource not found"
}
```

Jangan:

```json
{
  "error": "Report exists but belongs to another tenant"
}
```

Itu namanya membocorkan peta rumah tetangga.

---

## 37. MVP Security Phase

### Phase 1 — Core Security

* login,
* logout,
* password hash,
* role basic,
* tenant_id on tables,
* tenant guard,
* basic audit log.

### Phase 2 — Permission Hardening

* permission matrix,
* backend permission guard,
* Owner Tenant read-only enforcement,
* Client campaign isolation,
* IDOR tests.

### Phase 3 — Report Security

* private report storage,
* signed download URL,
* report status,
* download audit log,
* export permission.

### Phase 4 — Live View Security

* secure stream proxy,
* signed stream token,
* session expiry,
* viewer limit,
* live stream audit log,
* no raw RTSP frontend.

### Phase 5 — Privacy & Compliance Hardening

* retention policy,
* data minimization,
* sensitive log sanitizer,
* support mode audit,
* incident response checklist.

---

## 38. Definition of Done

Sebuah fitur dianggap selesai hanya jika:

* backend guard sudah ada,
* frontend guard sudah ada,
* permission matrix sesuai,
* tenant isolation aman,
* audit log tersedia untuk aksi penting,
* test akses role sudah dilakukan,
* test cross-tenant sudah dilakukan,
* tidak ada credential bocor,
* tidak ada raw RTSP bocor,
* report/download sudah protected,
* error handling tidak bocorkan data,
* dokumentasi endpoint diperbarui.

Kalau fitur hanya cantik di UI tapi belum aman di backend, statusnya bukan done.

Statusnya:

```text
BELUM LAYAK MASUK PRODUCTION
```

---

## 39. Kesimpulan

Security aplikasi ini harus dibangun sejak pondasi.

Yang paling wajib dikunci:

```text
Authentication
Session
RBAC
Tenant Isolation
Client Campaign Isolation
Secure Live Streaming
RTSP Protection
Report Download Control
Audit Log
Privacy
```

Manual ADX memberi pelajaran fitur. Tapi aplikasi kita harus lebih aman, lebih rapi, lebih audit-able, dan lebih siap menjadi SaaS.

Jangan sampai aplikasi terlihat seperti dashboard premium, tapi pintu belakangnya terbuka seperti warung kopi tengah malam.

Security adalah pagar. Audit log adalah CCTV sistem. Permission adalah kunci. Tenant isolation adalah tembok. Kalau empat ini kuat, SaaS punya martabat.
