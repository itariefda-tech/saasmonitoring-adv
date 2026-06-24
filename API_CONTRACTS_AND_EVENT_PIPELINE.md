# API_CONTRACTS_AND_EVENT_PIPELINE.md

# API Contracts & Event Pipeline

## SaaS Billboard Monitoring & Report Platform

Dokumen ini adalah kontrak utama API dan alur event untuk aplikasi SaaS Billboard Monitoring & Report.

Dokumen ini wajib menjadi acuan untuk backend, frontend, worker, AI pipeline, report generator, streaming proxy, dan agent coding.

API tidak boleh dibuat liar hanya karena UI butuh cepat tampil.
Setiap endpoint harus tunduk pada:

* authentication,
* authorization,
* tenant isolation,
* role permission,
* audit log,
* data quality policy,
* report integrity,
* live stream security,
* anti-overclaim policy.

Kalau API adalah jalan raya aplikasi, maka dokumen ini adalah rambu, marka, portal, CCTV, dan tilang elektroniknya. Tanpa ini, backend bisa ramai, tapi kacau seperti kabel jaringan tanpa ducting.

---

# 1. Tujuan Dokumen

Dokumen ini bertujuan untuk:

1. Mengunci struktur API utama.
2. Mengunci aturan response dan error.
3. Mengunci standar tenant guard.
4. Mengunci role guard.
5. Mengunci kontrak endpoint per modul.
6. Mengunci event pipeline dari CCTV/AI sampai dashboard/report.
7. Mengunci lifecycle report generation.
8. Mengunci lifecycle live streaming.
9. Mengunci audit log untuk semua aksi penting.
10. Menghindari endpoint liar yang tidak sesuai security dan database.

Dokumen ini bukan dokumentasi final OpenAPI lengkap, tetapi pondasi kontrak yang wajib diterjemahkan menjadi:

* REST API spec,
* OpenAPI/Swagger,
* backend route map,
* service interface,
* event schema,
* worker job schema,
* integration test,
* permission test.

---

# 2. Hubungan dengan Dokumen Lain

Dokumen ini harus sinkron dengan:

1. `README.md`
   Sebagai peta besar produk.

2. `PROJECT_VISION_BUSINESS_RULES.md`
   Sebagai aturan bisnis, target produk, batasan klaim, dan positioning.

3. `SYSTEM_ARCHITECTURE.md`
   Sebagai arsitektur service, worker, queue, storage, streaming proxy, dan deployment.

4. `DATABASE_DESIGN.md`
   Sebagai struktur tabel, relasi, index, tenant isolation, dan data retention.

5. `AI_CCTV_ANALYTICS_RULES.md`
   Sebagai aturan vehicle counting, confidence, data quality, anomaly, dan formula exposure.

6. `UI_UX_DASHBOARD_REPORTING_GUIDE.md`
   Sebagai acuan kebutuhan data dashboard, analytics, report, live view, dan mobile client.

7. `SECURITY_PRIVACY_ACCESS_CONTROL.md`
   Sebagai aturan keamanan, role permission, session, RTSP, audit, dan akses report.

8. `ROADMAP.md`
   Sebagai urutan implementasi bertahap.

Aturan penting:

API tidak boleh membuat field, role, atau event baru tanpa menyesuaikan dokumen terkait.

---

# 3. Prinsip Utama API

## 3.1 API Harus Multi-Tenant dari Awal

Semua resource operasional tenant wajib terikat pada `tenant_id`.

Contoh resource tenant:

* users tenant,
* clients,
* billboards,
* billboard_faces,
* cameras,
* edge_devices,
* campaigns,
* proof_of_display,
* traffic_events,
* traffic_summaries,
* reports,
* maintenance,
* alerts,
* live_stream_sessions.

Owner Platform SaaS boleh mengakses level platform sesuai permission.

Admin Tenant hanya boleh mengakses data tenant sendiri.

Owner Tenant hanya read-only pada tenant sendiri.

Client hanya boleh mengakses campaign miliknya.

Teknisi hanya boleh mengakses device/proof/maintenance yang ditugaskan.

---

## 3.2 Tenant ID Tidak Boleh Dipercaya dari Frontend

Frontend tidak boleh menjadi sumber kebenaran tenant.

Sumber tenant harus berasal dari:

* access token,
* session context,
* server-side user profile,
* platform-level explicit support mode yang diaudit.

Request seperti ini berbahaya:

```http
GET /api/v1/campaigns?tenant_id=tenant_b
```

Backend tidak boleh langsung percaya parameter tersebut.

Aturan:

* untuk user tenant biasa, `tenant_id` harus diambil dari token/session,
* parameter `tenant_id` dari query/body harus diabaikan atau ditolak,
* hanya Owner Platform SaaS dengan permission khusus boleh memilih tenant,
* semua akses lintas tenant harus masuk audit log.

---

## 3.3 Semua Endpoint Harus Punya Guard

Setiap endpoint wajib melewati minimal 4 guard:

```text
Auth Guard
↓
Tenant Guard
↓
Role/Permission Guard
↓
Resource Ownership Guard
```

Untuk aksi penting tambahkan:

```text
Audit Guard
↓
Rate Limit Guard
↓
Idempotency Guard jika write operation
```

Tidak boleh ada endpoint “sementara” tanpa guard.

Endpoint sementara sering menjadi pintu belakang permanen. Itu penyakit klasik aplikasi yang dibangun buru-buru.

---

## 3.4 Backend adalah Source of Truth

Frontend tidak boleh menghitung KPI kritis sebagai sumber utama.

Frontend boleh:

* render card,
* render chart,
* render table,
* melakukan format angka,
* melakukan filter tampilan ringan.

Backend wajib menghitung:

* total vehicle,
* estimated exposure,
* CPV,
* CPI,
* peak hour,
* data quality score,
* confidence score,
* anomaly flag,
* report status,
* permission result.

Kalau frontend menghitung sendiri, angka bisa beda antara dashboard, PDF, dan Excel. Itu awal bencana kecil yang nanti tumbuh jadi rapat panjang.

---

# 4. API Base Standard

## 4.1 Base URL

```text
/api/v1
```

Semua endpoint versi awal memakai prefix:

```text
/api/v1/...
```

Jika ada breaking change besar di masa depan, gunakan:

```text
/api/v2/...
```

Jangan mengubah behavior endpoint v1 secara diam-diam.

---

## 4.2 Format Data

Semua request dan response default menggunakan JSON.

```http
Content-Type: application/json
Accept: application/json
```

Upload file menggunakan:

```http
multipart/form-data
```

Download file menggunakan signed URL atau stream response yang sudah diaudit.

---

## 4.3 Naming Convention

Gunakan format endpoint plural:

```text
/users
/clients
/billboards
/campaigns
/devices
/reports
```

Gunakan kebab-case untuk action:

```text
/reports/{id}/download-pdf
/proof-display/{id}/approve
/live-stream/sessions/{id}/terminate
```

Gunakan snake_case untuk JSON field:

```json
{
  "campaign_id": "cmp_123",
  "data_quality_score": 94.5
}
```

---

## 4.4 ID Format

Semua ID publik API sebaiknya memakai prefixed UUID/ULID, bukan auto increment numeric.

Contoh:

```text
ten_01H...
usr_01H...
cli_01H...
bbd_01H...
cam_01H...
cmp_01H...
rpt_01H...
evt_01H...
```

Tujuannya:

* lebih aman dari enumeration,
* lebih mudah dibaca di log,
* lebih jelas jenis entity,
* lebih cocok untuk distributed system.

---

# 5. Standard Response Contract

## 5.1 Success Response

Semua response sukses menggunakan format:

```json
{
  "success": true,
  "data": {},
  "meta": {
    "request_id": "req_01HXYZ",
    "timestamp": "2026-03-31T10:15:00+07:00"
  }
}
```

Untuk list:

```json
{
  "success": true,
  "data": [],
  "meta": {
    "request_id": "req_01HXYZ",
    "timestamp": "2026-03-31T10:15:00+07:00",
    "pagination": {
      "page": 1,
      "per_page": 20,
      "total": 235,
      "total_pages": 12
    }
  }
}
```

---

## 5.2 Error Response

Semua error wajib konsisten.

```json
{
  "success": false,
  "error": {
    "code": "FORBIDDEN",
    "message": "You do not have permission to access this resource.",
    "details": null
  },
  "meta": {
    "request_id": "req_01HXYZ",
    "timestamp": "2026-03-31T10:15:00+07:00"
  }
}
```

---

## 5.3 Common Error Code

| HTTP | Code                | Makna                                         |
| ---: | ------------------- | --------------------------------------------- |
|  400 | BAD_REQUEST         | Request tidak valid                           |
|  401 | UNAUTHENTICATED     | Belum login / token invalid                   |
|  403 | FORBIDDEN           | Role/permission tidak cukup                   |
|  404 | NOT_FOUND           | Data tidak ditemukan atau tidak boleh dilihat |
|  409 | CONFLICT            | Konflik status/data                           |
|  422 | VALIDATION_ERROR    | Validasi field gagal                          |
|  423 | LOCKED              | Resource terkunci / sedang diproses           |
|  429 | RATE_LIMITED        | Terlalu banyak request                        |
|  500 | INTERNAL_ERROR      | Error server                                  |
|  503 | SERVICE_UNAVAILABLE | Service/worker/stream tidak tersedia          |

Catatan penting:

Untuk mencegah data leakage, resource tenant lain boleh dikembalikan sebagai `404 NOT_FOUND`, bukan `403`, agar user tidak tahu bahwa resource tersebut ada.

---

# 6. Authentication Contract

## 6.1 Login

```http
POST /api/v1/auth/login
```

Request:

```json
{
  "email": "admin@tenant.com",
  "password": "strong_password"
}
```

Response:

```json
{
  "success": true,
  "data": {
    "access_token": "jwt_access_token",
    "refresh_token": "jwt_refresh_token",
    "token_type": "Bearer",
    "expires_in": 900,
    "user": {
      "id": "usr_01H",
      "name": "Admin Tenant",
      "email": "admin@tenant.com",
      "role": "ADMIN_TENANT",
      "tenant_id": "ten_01H",
      "permissions": [
        "campaign.read",
        "campaign.create",
        "report.generate"
      ]
    }
  }
}
```

Aturan:

* password wajib di-hash,
* tidak boleh menyimpan password plain text,
* gagal login berulang harus rate-limited,
* login sukses/gagal masuk audit/security log,
* token harus short-lived,
* refresh token harus bisa dicabut.

---

## 6.2 Logout

```http
POST /api/v1/auth/logout
```

Aturan:

* revoke refresh token,
* terminate active live stream session user jika perlu,
* audit log `auth.logout`.

---

## 6.3 Me

```http
GET /api/v1/auth/me
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "usr_01H",
    "name": "Owner Tenant",
    "email": "owner@tenant.com",
    "role": "OWNER_TENANT",
    "tenant_id": "ten_01H",
    "tenant_name": "Tenant Billboard A",
    "permissions": [
      "dashboard.read",
      "report.read",
      "report.download"
    ],
    "features": {
      "live_view": true,
      "export_pdf": true,
      "export_excel": false,
      "data_quality_score": true
    }
  }
}
```

Endpoint ini dipakai frontend untuk:

* render sidebar,
* render role-based menu,
* menampilkan feature sesuai tier,
* menyembunyikan tombol berbahaya.

Tetapi menyembunyikan tombol di frontend bukan pengganti backend permission.

---

# 7. Role & Permission Contract

## 7.1 Role Utama

Role resmi:

```text
OWNER_PLATFORM
ADMIN_TENANT
OWNER_TENANT
SALES_TENANT
TECHNICIAN
CLIENT
VIEWER
```

Tidak boleh membuat role baru tanpa update:

* `SECURITY_PRIVACY_ACCESS_CONTROL.md`,
* `DATABASE_DESIGN.md`,
* dokumen ini,
* permission seed,
* test permission.

---

## 7.2 Permission Format

Gunakan format:

```text
resource.action
```

Contoh:

```text
tenant.create
tenant.update
billboard.read
billboard.create
campaign.update
report.generate
report.download
live_stream.view
proof.approve
audit_log.read
```

---

## 7.3 Permission Minimum per Role

| Permission Area   | Owner Platform |      Admin Tenant |  Owner Tenant |            Sales |         Teknisi |            Client |
| ----------------- | -------------: | ----------------: | ------------: | ---------------: | --------------: | ----------------: |
| Platform Tenant   |           Full |                No |            No |               No |              No |                No |
| Subscription/Tier |           Full |                No |            No |               No |              No |                No |
| Tenant Users      |  Support/Audit |       Full Tenant |            No |               No |              No |                No |
| Billboard         |  Support/Audit |       Full Tenant |          Read |     Read/Limited |   Assigned Read |     Campaign Only |
| Camera/Device     |  Support/Audit |       Full Tenant |  Summary Read |       No/Limited | Assigned Update |       Status Only |
| Live Stream       |  Support/Audit |       Full Tenant | Optional Read | Optional/Limited |   Assigned View | Optional Campaign |
| Campaign          |  Support/Audit |       Full Tenant |          Read |    Draft/Limited |              No |      Campaign Own |
| Proof Display     |  Support/Audit |           Approve |          Read |               No |          Upload |      Approved Own |
| Analytics         |   Global Usage |       Tenant Full |   Tenant Read |          Limited |       Technical |      Campaign Own |
| Report            |   Global Usage | Generate/Download | Read/Download |          Limited |      No/Limited |      Campaign Own |
| Audit Log         |       Platform |            Tenant |    No/Limited |               No |              No |                No |

---

# 8. Tenant Guard Rules

## 8.1 Tenant Context

Backend harus membangun `request_context` setelah auth:

```json
{
  "request_id": "req_01H",
  "user_id": "usr_01H",
  "role": "ADMIN_TENANT",
  "tenant_id": "ten_01H",
  "permissions": [],
  "ip_address": "103.x.x.x",
  "user_agent": "Mozilla/5.0"
}
```

---

## 8.2 Query Tenant Rule

Untuk user tenant biasa:

```sql
WHERE tenant_id = current_user.tenant_id
```

Untuk client:

```sql
WHERE tenant_id = current_user.tenant_id
AND client_id = current_user.client_id
```

Untuk campaign client:

```sql
WHERE campaign_id IN (
  SELECT campaign_id
  FROM campaign_client_access
  WHERE user_id = current_user.id
)
```

Untuk teknisi:

```sql
WHERE assigned_technician_id = current_user.id
OR camera_id IN assigned_camera_ids
```

Untuk Owner Platform SaaS:

* boleh akses global sesuai permission,
* support access ke tenant harus memiliki reason,
* semua support access wajib audit.

---

## 8.3 Cross-Tenant Access

Cross-tenant access hanya boleh untuk:

* Owner Platform SaaS,
* system worker,
* support mode resmi,
* billing/subscription aggregation,
* system health aggregation.

Wajib audit:

```json
{
  "event": "platform.cross_tenant_access",
  "actor_user_id": "usr_platform",
  "target_tenant_id": "ten_01H",
  "reason": "support_ticket_123",
  "resource_type": "campaign",
  "resource_id": "cmp_01H"
}
```

Tanpa reason, cross-tenant access ditolak.

---

# 9. API Endpoint Overview

Endpoint utama:

```text
/auth/login
/auth/logout
/auth/me

/platform/tenants
/platform/plans
/platform/subscriptions
/platform/system-health
/platform/feature-flags

/profile

/users
/users/{id}
/users/{id}/roles
/users/{id}/status

/clients
/clients/{id}

/billboards
/billboards/{id}
/billboards/{id}/faces

/campaigns
/campaigns/{id}
/campaigns/{id}/billboards
/campaigns/{id}/access

/devices
/devices/{id}
/devices/{id}/health
/devices/{id}/heartbeat
/devices/{id}/snapshots

/live-stream/sessions
/live-stream/sessions/{id}
/live-stream/sessions/{id}/terminate

/analytics/summary
/analytics/timeseries
/analytics/vehicle-composition
/analytics/hourly-heatmap
/analytics/location/{location_id}

/proof-display
/proof-display/{id}
/proof-display/{id}/approve
/proof-display/{id}/reject

/reports
/reports/generate
/reports/{id}
/reports/{id}/approve
/reports/{id}/send
/reports/{id}/download/pdf
/reports/{id}/download/excel
/reports/{id}/download/csv

/alerts
/alerts/{id}
/alerts/{id}/acknowledge

/maintenance
/maintenance/{id}

/audit-logs

/ingest/traffic-events
/ingest/heartbeat
/ingest/quality-snapshots
```

Semua endpoint harus:

* authenticated,
* role-checked,
* tenant-filtered,
* audited untuk aksi penting.

---

# 10. Platform API

Endpoint platform hanya untuk Owner Platform SaaS.

## 10.1 List Tenants

```http
GET /api/v1/platform/tenants
```

Query:

```text
status=active|trial|suspended
plan_id=plan_01H
search=keyword
page=1
per_page=20
```

Access:

```text
OWNER_PLATFORM
```

Audit:

* tidak wajib untuk read biasa,
* wajib jika menggunakan support/cross-tenant view.

---

## 10.2 Create Tenant

```http
POST /api/v1/platform/tenants
```

Request:

```json
{
  "company_name": "Billboard Company A",
  "legal_name": "PT Billboard A",
  "email": "admin@billboard-a.com",
  "phone": "+628xxx",
  "plan_id": "plan_01H",
  "status": "active",
  "branding": {
    "logo_url": null,
    "primary_color": "#D71920"
  }
}
```

Access:

```text
OWNER_PLATFORM
```

Audit event:

```text
tenant.created
```

---

## 10.3 Update Tenant Status

```http
PATCH /api/v1/platform/tenants/{tenant_id}/status
```

Request:

```json
{
  "status": "suspended",
  "reason": "subscription_overdue"
}
```

Aturan:

* tenant suspended tidak boleh login kecuali role tertentu,
* client tenant suspended tidak boleh download report,
* worker analytics dapat tetap menyimpan data sesuai policy,
* status change wajib audit.

Audit event:

```text
tenant.status_changed
```

---

## 10.4 Platform System Health

```http
GET /api/v1/platform/system-health
```

Response data:

```json
{
  "api_status": "healthy",
  "database_status": "healthy",
  "queue_status": "healthy",
  "worker_status": "degraded",
  "stream_proxy_status": "healthy",
  "storage_status": "healthy",
  "active_tenants": 15,
  "active_cameras": 120,
  "offline_cameras": 8,
  "failed_report_jobs": 2,
  "error_rate": 0.04
}
```

Access:

```text
OWNER_PLATFORM
```

---

# 11. Profile API

## 11.1 Get Profile

```http
GET /api/v1/profile
```

Access:

```text
ALL_AUTHENTICATED_USERS
```

---

## 11.2 Update Profile

```http
PATCH /api/v1/profile
```

Request:

```json
{
  "name": "User Name",
  "photo_url": "https://storage/app/profile/usr_01H.jpg"
}
```

Audit:

```text
profile.updated
```

---

## 11.3 Change Password

```http
PATCH /api/v1/profile/password
```

Request:

```json
{
  "current_password": "old_password",
  "new_password": "new_strong_password",
  "confirm_password": "new_strong_password"
}
```

Aturan:

* wajib validasi password strength,
* revoke session lain jika diperlukan,
* audit security event.

Audit:

```text
profile.password_changed
```

---

# 12. User & Role API

## 12.1 List Tenant Users

```http
GET /api/v1/users
```

Access:

```text
ADMIN_TENANT
OWNER_PLATFORM in support mode
```

Owner Tenant tidak boleh mengelola user.

---

## 12.2 Create User

```http
POST /api/v1/users
```

Request:

```json
{
  "name": "Sales Tenant",
  "email": "sales@tenant.com",
  "role": "SALES_TENANT",
  "status": "active"
}
```

Aturan:

* role yang bisa dibuat Admin Tenant hanya role dalam tenant,
* Admin Tenant tidak boleh membuat Owner Platform,
* Client user harus terkait client record,
* Teknisi user bisa terkait assignment device.

Audit:

```text
user.created
```

---

## 12.3 Update User Role

```http
PATCH /api/v1/users/{id}/roles
```

Request:

```json
{
  "role": "OWNER_TENANT"
}
```

Aturan:

* hanya Admin Tenant untuk tenant sendiri,
* tidak boleh menaikkan user ke role platform,
* perubahan role wajib audit,
* perubahan role harus terminate session lama user tersebut.

Audit:

```text
user.role_changed
```

---

# 13. Client API

## 13.1 List Clients

```http
GET /api/v1/clients
```

Access:

```text
ADMIN_TENANT
OWNER_TENANT read-only
SALES_TENANT limited
OWNER_PLATFORM support/audit
```

Client role tidak boleh melihat daftar client.

---

## 13.2 Create Client

```http
POST /api/v1/clients
```

Access:

```text
ADMIN_TENANT
```

Request:

```json
{
  "company_name": "BYD",
  "brand_name": "BYD",
  "industry": "Automotive",
  "contact_person": "Client PIC",
  "email": "client@brand.com",
  "phone": "+628xxx",
  "status": "active"
}
```

Audit:

```text
client.created
```

---

# 14. Billboard API

## 14.1 List Billboards

```http
GET /api/v1/billboards
```

Query:

```text
city=Tangerang
status=active
road_type=toll|urban|intersection|landmark
search=keyword
page=1
per_page=20
```

Access:

```text
ADMIN_TENANT full
OWNER_TENANT read-only
SALES_TENANT read/limited
CLIENT campaign-only
TECHNICIAN assigned-only
```

Aturan:

* Client hanya melihat billboard yang terkait campaign miliknya.
* Owner Tenant tidak boleh create/update/delete.
* Teknisi hanya melihat billboard terkait assignment.

---

## 14.2 Create Billboard

```http
POST /api/v1/billboards
```

Access:

```text
ADMIN_TENANT
```

Request:

```json
{
  "name": "JPO TangCity Mall",
  "code": "BBD-TNG-001",
  "address": "Jl. Jenderal Sudirman, Tangerang",
  "city": "Tangerang",
  "province": "Banten",
  "latitude": -6.1789,
  "longitude": 106.6319,
  "road_type": "urban",
  "size": "10x5m",
  "orientation": "front",
  "visibility_angle": 75,
  "monthly_rate": 286000000,
  "status": "active"
}
```

Audit:

```text
billboard.created
```

---

## 14.3 Billboard Faces

```http
GET /api/v1/billboards/{id}/faces
POST /api/v1/billboards/{id}/faces
```

Face data:

```json
{
  "face_name": "Front Face",
  "direction": "northbound",
  "visibility_angle": 75,
  "status": "active"
}
```

Wajib mendukung multi-face karena billboard bisa punya lebih dari satu sisi/muka.

---

# 15. Campaign API

## 15.1 List Campaigns

```http
GET /api/v1/campaigns
```

Query:

```text
status=active|draft|completed
client_id=cli_01H
billboard_id=bbd_01H
date_from=2026-03-01
date_to=2026-03-31
```

Access:

```text
ADMIN_TENANT full
OWNER_TENANT read-only
SALES_TENANT own/limited
CLIENT own campaign only
OWNER_PLATFORM support/audit
```

---

## 15.2 Create Campaign

```http
POST /api/v1/campaigns
```

Access:

```text
ADMIN_TENANT
SALES_TENANT draft only if permission enabled
```

Request:

```json
{
  "client_id": "cli_01H",
  "brand": "BYD",
  "campaign_name": "BYD March 2026 Tangerang",
  "objective": "awareness",
  "start_date": "2026-03-01",
  "end_date": "2026-03-31",
  "estimated_ooh_budget": 286000000,
  "status": "draft",
  "billboard_ids": [
    "bbd_01H",
    "bbd_02H"
  ]
}
```

Audit:

```text
campaign.created
```

---

## 15.3 Update Campaign Status

```http
PATCH /api/v1/campaigns/{id}/status
```

Request:

```json
{
  "status": "active"
}
```

Status:

```text
draft
active
paused
completed
archived
cancelled
```

Aturan:

* campaign active harus punya client,
* campaign active harus punya minimal satu billboard,
* campaign active harus punya periode valid,
* perubahan status wajib audit.

---

## 15.4 Campaign Access for Client

```http
POST /api/v1/campaigns/{id}/access
```

Request:

```json
{
  "user_id": "usr_client",
  "permissions": [
    "campaign.read",
    "analytics.read",
    "proof.read",
    "report.read",
    "report.download"
  ],
  "live_view_enabled": false
}
```

Aturan:

* client access harus eksplisit,
* live view untuk client default off,
* semua akses client harus campaign-scoped.

Audit:

```text
campaign.client_access_granted
```

---

# 16. Device & Camera API

## 16.1 List Devices

```http
GET /api/v1/devices
```

Access:

```text
ADMIN_TENANT full
OWNER_TENANT summary read
TECHNICIAN assigned
CLIENT status only if campaign-related
OWNER_PLATFORM global/support
```

Response item:

```json
{
  "id": "cam_01H",
  "name": "CVC TangCity 01",
  "billboard_id": "bbd_01H",
  "location_name": "JPO TangCity Mall",
  "status": "online",
  "last_heartbeat_at": "2026-03-31T10:10:00+07:00",
  "health_score": 96,
  "stream_quality": "good",
  "fps": 25,
  "resolution": "1920x1080",
  "latency_ms": 450
}
```

Catatan:

* raw RTSP tidak pernah dikirim ke frontend biasa,
* stream URL internal harus encrypted,
* frontend hanya menerima proxy playback URL melalui live stream session.

---

## 16.2 Create Device

```http
POST /api/v1/devices
```

Access:

```text
ADMIN_TENANT
```

Request:

```json
{
  "name": "CVC TangCity 01",
  "billboard_id": "bbd_01H",
  "camera_position": "front-road",
  "direction": "northbound",
  "stream_url": "rtsp://internal-camera/streaming",
  "fps": 25,
  "resolution": "1920x1080",
  "status": "active"
}
```

Backend wajib:

* encrypt `stream_url`,
* tidak pernah expose plain stream URL,
* simpan audit,
* membuat initial camera health record.

Audit:

```text
device.created
```

---

## 16.3 Device Health

```http
GET /api/v1/devices/{id}/health
```

Response:

```json
{
  "device_id": "cam_01H",
  "status": "online",
  "last_heartbeat_at": "2026-03-31T10:10:00+07:00",
  "uptime_percentage_24h": 99.2,
  "fps": 25,
  "resolution": "1920x1080",
  "latency_ms": 450,
  "stream_quality": "good",
  "health_score": 96,
  "issues": []
}
```

---

## 16.4 Device Snapshot

```http
POST /api/v1/devices/{id}/snapshots
```

Access:

```text
ADMIN_TENANT
TECHNICIAN assigned
```

Use case:

* capture snapshot dari live view,
* bukti kondisi kamera,
* dokumentasi lokasi,
* troubleshooting.

Audit:

```text
device.snapshot_created
```

---

# 17. Live Stream API

Live stream adalah area sensitif. Tidak boleh expose raw RTSP ke client.

## 17.1 Create Live Stream Session

```http
POST /api/v1/live-stream/sessions
```

Request:

```json
{
  "camera_id": "cam_01H",
  "purpose": "monitoring",
  "overlay": {
    "show_detection_box": true,
    "show_counting_line": true
  }
}
```

Access:

```text
ADMIN_TENANT
OWNER_TENANT if enabled
TECHNICIAN assigned
CLIENT if campaign-related and enabled
OWNER_PLATFORM support/audit
```

Response:

```json
{
  "success": true,
  "data": {
    "session_id": "lss_01H",
    "camera_id": "cam_01H",
    "playback_url": "https://stream-proxy.app/live/lss_01H/index.m3u8",
    "expires_at": "2026-03-31T10:25:00+07:00",
    "max_duration_seconds": 600,
    "viewer_limit_policy": "one_active_session_per_user_camera"
  }
}
```

Aturan:

* `playback_url` harus signed dan short-lived,
* default durasi bisa 10 menit untuk benchmark awal,
* session expiry wajib jelas,
* satu user satu camera bisa dibatasi satu session aktif,
* raw RTSP tidak dikirim,
* session start wajib audit,
* session expired wajib event.

Audit:

```text
live_stream.session_started
```

---

## 17.2 Get Live Stream Session

```http
GET /api/v1/live-stream/sessions/{id}
```

Response:

```json
{
  "session_id": "lss_01H",
  "status": "active",
  "camera_id": "cam_01H",
  "started_at": "2026-03-31T10:15:00+07:00",
  "expires_at": "2026-03-31T10:25:00+07:00",
  "remaining_seconds": 480
}
```

---

## 17.3 Terminate Live Stream Session

```http
POST /api/v1/live-stream/sessions/{id}/terminate
```

Request:

```json
{
  "reason": "user_logout"
}
```

Audit:

```text
live_stream.session_terminated
```

---

## 17.4 Live Stream Forbidden Rules

Backend wajib menolak live stream jika:

* camera bukan milik tenant user,
* camera tidak terkait campaign client,
* role tidak punya permission,
* feature live view tidak aktif di tier tenant,
* tenant suspended,
* session limit tercapai,
* camera offline,
* stream proxy error,
* user sedang dalam status disabled.

---

# 18. Traffic Analytics API

Analytics harus mengambil data dari summary tervalidasi, bukan langsung dari raw detection event.

## 18.1 Analytics Summary

```http
GET /api/v1/analytics/summary
```

Query:

```text
period=daily|weekly|monthly|custom
date_from=2026-03-01
date_to=2026-03-31
location_id=bbd_01H
campaign_id=cmp_01H
client_id=cli_01H
vehicle_class=all|car|motorcycle|bus|truck
direction=northbound
data_mode=validated|estimated|raw
```

Access:

```text
ADMIN_TENANT tenant
OWNER_TENANT read-only
SALES_TENANT limited
CLIENT campaign-own
OWNER_PLATFORM support/audit
```

Response:

```json
{
  "total_vehicle": 3636404,
  "total_car": 724000,
  "total_motorcycle": 2562000,
  "total_bus": 27600,
  "total_truck": 322804,
  "estimated_exposure": 4875559,
  "average_daily_traffic": 117303,
  "peak_day": "2026-03-26",
  "peak_hour": "17:00",
  "average_speed_kmh": 28.5,
  "congestion_level": "medium",
  "data_quality_score": 94.2,
  "average_confidence_score": 91.8,
  "valid_hours": 736,
  "expected_hours": 744,
  "missing_hours": 8,
  "anomaly_count": 2,
  "formula_version": "exposure_v1.0",
  "methodology_version": "cvc_method_v1.0"
}
```

Aturan:

* data low quality harus diberi warning,
* estimated data harus jelas labelnya,
* jangan tampilkan impression sebagai “pasti dilihat”,
* gunakan istilah `estimated_exposure`.

---

## 18.2 Timeseries

```http
GET /api/v1/analytics/timeseries
```

Query:

```text
granularity=hourly|daily|weekly|monthly
date_from=2026-03-01
date_to=2026-03-31
campaign_id=cmp_01H
location_id=bbd_01H
```

Response item:

```json
{
  "timestamp": "2026-03-01T17:00:00+07:00",
  "car": 1427,
  "motorcycle": 6289,
  "bus": 13,
  "truck": 643,
  "total_vehicle": 8372,
  "estimated_exposure": 10320,
  "data_quality_score": 95.1,
  "is_estimated": false,
  "anomaly_flag": false
}
```

---

## 18.3 Vehicle Composition

```http
GET /api/v1/analytics/vehicle-composition
```

Response:

```json
{
  "total_vehicle": 3636404,
  "composition": [
    {
      "class": "motorcycle",
      "count": 2562000,
      "percentage": 70.47
    },
    {
      "class": "car",
      "count": 724000,
      "percentage": 19.91
    },
    {
      "class": "truck",
      "count": 322804,
      "percentage": 8.86
    },
    {
      "class": "bus",
      "count": 27600,
      "percentage": 0.76
    }
  ]
}
```

---

## 18.4 Hourly Heatmap

```http
GET /api/v1/analytics/hourly-heatmap
```

Response item:

```json
{
  "date": "2026-03-01",
  "hour": 17,
  "total_vehicle": 8372,
  "intensity": "high",
  "data_quality_score": 95.1
}
```

---

## 18.5 Location Analytics

```http
GET /api/v1/analytics/location/{location_id}
```

Response:

```json
{
  "location_id": "bbd_01H",
  "location_name": "JPO TangCity Mall",
  "road_type": "urban",
  "summary": {},
  "vehicle_composition": [],
  "timeseries": [],
  "quality": {
    "data_quality_score": 94.2,
    "valid_hours": 736,
    "missing_hours": 8
  },
  "insights": [
    {
      "type": "peak_pattern",
      "message": "Traffic typically peaks around afternoon commute hours."
    }
  ]
}
```

---

# 19. Proof of Display API

## 19.1 Upload Proof

```http
POST /api/v1/proof-display
```

Content-Type:

```text
multipart/form-data
```

Fields:

```text
campaign_id
billboard_id
proof_type=installation|periodic|maintenance|creative_check
photo_file
captured_at
latitude
longitude
notes
```

Access:

```text
ADMIN_TENANT
TECHNICIAN assigned
```

Response:

```json
{
  "id": "pod_01H",
  "status": "pending_review",
  "photo_url": "https://storage/proof/pod_01H.jpg"
}
```

Audit:

```text
proof.uploaded
```

---

## 19.2 Approve Proof

```http
POST /api/v1/proof-display/{id}/approve
```

Access:

```text
ADMIN_TENANT
```

Request:

```json
{
  "notes": "Billboard creative is visible and approved."
}
```

Audit:

```text
proof.approved
```

---

## 19.3 Reject Proof

```http
POST /api/v1/proof-display/{id}/reject
```

Access:

```text
ADMIN_TENANT
```

Request:

```json
{
  "reason": "Photo is blurry. Please retake."
}
```

Audit:

```text
proof.rejected
```

Aturan:

* Client hanya melihat proof yang approved.
* Owner Tenant boleh melihat summary/read-only.
* Proof rejected tidak tampil di client dashboard.
* Semua approval/rejection wajib audit.

---

# 20. Report API

Report adalah output resmi ke client. Endpoint report harus sangat rapi.

## 20.1 List Reports

```http
GET /api/v1/reports
```

Query:

```text
campaign_id=cmp_01H
client_id=cli_01H
status=draft|generated|waiting_review|approved|sent|failed|archived
period_type=daily|weekly|monthly
date_from=2026-03-01
date_to=2026-03-31
```

Access:

```text
ADMIN_TENANT full
OWNER_TENANT read/download if allowed
SALES_TENANT limited
CLIENT own campaign only
OWNER_PLATFORM usage/support
```

---

## 20.2 Generate Report

```http
POST /api/v1/reports/generate
```

Access:

```text
ADMIN_TENANT
```

Request:

```json
{
  "campaign_id": "cmp_01H",
  "period_type": "monthly",
  "date_from": "2026-03-01",
  "date_to": "2026-03-31",
  "format": [
    "pdf",
    "excel",
    "csv"
  ],
  "include_sections": [
    "cover",
    "executive_summary",
    "campaign_overview",
    "measurement_methodology",
    "network_map",
    "overall_kpi",
    "location_summary",
    "traffic_analysis",
    "hourly_heatmap",
    "vehicle_composition",
    "cost_efficiency",
    "data_quality",
    "proof_display",
    "recommendation",
    "raw_data_appendix",
    "disclaimer"
  ]
}
```

Response:

```json
{
  "report_id": "rpt_01H",
  "job_id": "job_01H",
  "status": "queued",
  "estimated_ready_at": "2026-03-31T10:20:00+07:00"
}
```

Audit:

```text
report.generation_requested
```

Aturan:

* generate report harus async,
* report harus menyimpan snapshot data,
* report harus menyimpan formula version,
* report harus menyimpan methodology version,
* report harus menyimpan generated_by,
* report tidak boleh berubah setelah approved kecuali generate versi baru.

---

## 20.3 Get Report Detail

```http
GET /api/v1/reports/{id}
```

Response:

```json
{
  "id": "rpt_01H",
  "campaign_id": "cmp_01H",
  "client_id": "cli_01H",
  "period_type": "monthly",
  "date_from": "2026-03-01",
  "date_to": "2026-03-31",
  "status": "approved",
  "report_version": "1.0",
  "formula_version": "exposure_v1.0",
  "methodology_version": "cvc_method_v1.0",
  "generated_at": "2026-03-31T10:20:00+07:00",
  "generated_by": "usr_01H",
  "approved_at": "2026-03-31T11:00:00+07:00",
  "approved_by": "usr_admin",
  "data_quality_summary": {
    "average_score": 94.2,
    "valid_hours": 736,
    "missing_hours": 8,
    "anomaly_count": 2
  },
  "exports": {
    "pdf": "available",
    "excel": "available",
    "csv": "available"
  }
}
```

---

## 20.4 Approve Report

```http
POST /api/v1/reports/{id}/approve
```

Access:

```text
ADMIN_TENANT
```

Request:

```json
{
  "notes": "Report reviewed and approved for client."
}
```

Audit:

```text
report.approved
```

Aturan:

* report failed tidak bisa approved,
* report harus punya file output,
* report harus punya data quality summary,
* report harus punya formula/methodology version,
* approval wajib audit.

---

## 20.5 Download Report PDF

```http
GET /api/v1/reports/{id}/download/pdf
```

Access:

```text
ADMIN_TENANT
OWNER_TENANT if allowed
CLIENT own campaign if approved
SALES_TENANT limited if allowed
```

Response:

```json
{
  "download_url": "https://storage/signed/report/rpt_01H.pdf?token=...",
  "expires_at": "2026-03-31T10:30:00+07:00"
}
```

Aturan:

* download URL harus signed,
* expiry pendek,
* client hanya boleh download approved report,
* report download wajib audit.

Audit:

```text
report.downloaded
```

---

## 20.6 Download Excel/CSV

```http
GET /api/v1/reports/{id}/download/excel
GET /api/v1/reports/{id}/download/csv
```

Aturan:

* Excel/CSV berisi raw/appendix data,
* akses bisa lebih ketat daripada PDF,
* semua download wajib audit,
* jangan tampilkan raw data campaign lain.

---

# 21. Alert API

## 21.1 List Alerts

```http
GET /api/v1/alerts
```

Query:

```text
severity=critical|high|medium|low|info
status=open|acknowledged|resolved
type=camera_offline|missing_data|report_failed|low_quality
```

Access:

```text
ADMIN_TENANT
TECHNICIAN assigned technical alerts
OWNER_TENANT summary read
OWNER_PLATFORM global
```

---

## 21.2 Acknowledge Alert

```http
POST /api/v1/alerts/{id}/acknowledge
```

Request:

```json
{
  "notes": "Technician has been assigned."
}
```

Audit:

```text
alert.acknowledged
```

---

# 22. Maintenance API

## 22.1 Create Maintenance Ticket

```http
POST /api/v1/maintenance
```

Request:

```json
{
  "camera_id": "cam_01H",
  "billboard_id": "bbd_01H",
  "issue_type": "camera_offline",
  "severity": "high",
  "assigned_to": "usr_technician",
  "description": "Camera offline since 09:15."
}
```

Access:

```text
ADMIN_TENANT
TECHNICIAN limited if assigned/report issue
```

Audit:

```text
maintenance.ticket_created
```

---

## 22.2 Update Maintenance Ticket

```http
PATCH /api/v1/maintenance/{id}
```

Request:

```json
{
  "status": "resolved",
  "resolution_notes": "Power adapter replaced.",
  "resolved_at": "2026-03-31T15:00:00+07:00"
}
```

Audit:

```text
maintenance.ticket_updated
```

---

# 23. Audit Log API

## 23.1 List Audit Logs

```http
GET /api/v1/audit-logs
```

Query:

```text
actor_user_id=usr_01H
event=report.downloaded
resource_type=report
resource_id=rpt_01H
date_from=2026-03-01
date_to=2026-03-31
```

Access:

```text
OWNER_PLATFORM global
ADMIN_TENANT tenant audit
```

Owner Tenant dan Client tidak boleh melihat audit log detail kecuali dibuatkan summary khusus.

Audit log sendiri tidak perlu diaudit untuk read biasa, kecuali cross-tenant/platform support mode.

---

# 24. Ingestion API

Ingestion API dipakai oleh edge device, AI worker, atau trusted internal service.

Tidak boleh dipakai langsung oleh frontend.

## 24.1 Device Heartbeat

```http
POST /api/v1/ingest/heartbeat
```

Auth:

```text
Device token / signed service token
```

Request:

```json
{
  "device_id": "cam_01H",
  "edge_device_id": "edg_01H",
  "timestamp": "2026-03-31T10:15:00+07:00",
  "status": "online",
  "fps": 25,
  "resolution": "1920x1080",
  "latency_ms": 420,
  "stream_quality": "good",
  "cpu_usage": 41.5,
  "memory_usage": 62.2,
  "storage_usage": 55.1
}
```

Pipeline result:

* update camera health,
* update last heartbeat,
* trigger alert if recovered/offline,
* store camera health log.

Event:

```text
camera.heartbeat.received
```

---

## 24.2 Traffic Detection Batch

```http
POST /api/v1/ingest/traffic-events
```

Request:

```json
{
  "device_id": "cam_01H",
  "camera_id": "cam_01H",
  "billboard_id": "bbd_01H",
  "timestamp_start": "2026-03-31T17:00:00+07:00",
  "timestamp_end": "2026-03-31T17:01:00+07:00",
  "model_version": "vehicle_detector_v1.2",
  "rule_version": "counting_rule_v1.0",
  "events": [
    {
      "track_id": "trk_abc123",
      "vehicle_class": "motorcycle",
      "confidence": 0.93,
      "direction": "northbound",
      "counting_line_id": "line_01H",
      "crossed_at": "2026-03-31T17:00:12+07:00",
      "speed_kmh": 27.4,
      "bbox": [120, 240, 180, 320]
    }
  ],
  "quality": {
    "frame_quality_score": 91.5,
    "occlusion_level": "low",
    "weather_condition": "unknown",
    "night_mode": false
  }
}
```

Aturan:

* raw event disimpan sebagai raw/ingested,
* jangan langsung menjadi KPI final,
* wajib validation,
* wajib deduplication,
* wajib confidence filtering,
* wajib aggregation.

Event:

```text
traffic.batch.received
```

---

## 24.3 Quality Snapshot

```http
POST /api/v1/ingest/quality-snapshots
```

Request:

```json
{
  "camera_id": "cam_01H",
  "timestamp": "2026-03-31T17:00:00+07:00",
  "camera_uptime_score": 99.2,
  "detection_confidence_score": 91.8,
  "frame_quality_score": 92.5,
  "missing_data_score": 98.0,
  "data_quality_score": 94.2,
  "notes": "Normal condition"
}
```

Event:

```text
data_quality.snapshot_received
```

---

# 25. Event Pipeline

## 25.1 Main Data Flow

```text
CCTV / CVC
↓
Edge Device / AI Processor
↓
Detection Batch Ingestion API
↓
Message Queue
↓
Raw Traffic Event Store
↓
Validation Worker
↓
Deduplication Worker
↓
Quality Scoring Worker
↓
Hourly Aggregation Worker
↓
Daily / Monthly Summary Worker
↓
Analytics API
↓
Dashboard / Report Center
```

---

## 25.2 Report Flow

```text
Admin Tenant requests report generation
↓
API validates permission + tenant + campaign
↓
Report job created
↓
Report worker pulls summary snapshot
↓
Formula engine calculates KPI
↓
Chart engine generates visuals
↓
PDF/Excel/CSV generated
↓
Files stored in secure storage
↓
Report status becomes Generated / Waiting Review
↓
Admin reviews and approves
↓
Client can download approved report
↓
Every generation/approval/download enters audit log
```

---

## 25.3 Live Stream Flow

```text
User requests live stream session
↓
API validates role + tenant + camera + feature tier
↓
API creates short-lived live stream session
↓
Stream proxy maps encrypted internal RTSP to secure playback URL
↓
Frontend plays secure playback URL
↓
Session expires automatically
↓
Audit logs session start/end/download/snapshot if any
```

No raw RTSP goes to frontend.

---

## 25.4 Alert Flow

```text
Heartbeat missing / low quality / report failed / data missing
↓
Worker creates alert event
↓
Alert appears in dashboard
↓
Admin/Teknisi acknowledges alert
↓
Maintenance ticket may be created
↓
Resolution updates alert status
↓
Audit log records critical actions
```

---

# 26. Event Naming Standard

Gunakan format:

```text
domain.entity_action
```

Contoh:

```text
auth.login_succeeded
auth.login_failed
auth.logout

tenant.created
tenant.updated
tenant.status_changed

user.created
user.updated
user.role_changed
user.disabled

billboard.created
billboard.updated
billboard.deleted

camera.created
camera.updated
camera.heartbeat_received
camera.offline_detected
camera.recovered
camera.snapshot_created

live_stream.session_started
live_stream.session_expired
live_stream.session_terminated

traffic.batch_received
traffic.event_validated
traffic.event_rejected
traffic.hourly_summary_updated
traffic.daily_summary_updated
traffic.anomaly_detected

data_quality.snapshot_received
data_quality.low_score_detected

proof.uploaded
proof.approved
proof.rejected

report.generation_requested
report.generation_started
report.generation_completed
report.generation_failed
report.approved
report.sent_to_client
report.downloaded

alert.created
alert.acknowledged
alert.resolved

maintenance.ticket_created
maintenance.ticket_updated
maintenance.ticket_resolved
```

---

# 27. Event Envelope Contract

Semua event internal harus memakai envelope:

```json
{
  "event_id": "evt_01H",
  "event_name": "report.generation_requested",
  "occurred_at": "2026-03-31T10:15:00+07:00",
  "tenant_id": "ten_01H",
  "actor": {
    "type": "user",
    "id": "usr_01H",
    "role": "ADMIN_TENANT"
  },
  "resource": {
    "type": "report",
    "id": "rpt_01H"
  },
  "metadata": {
    "request_id": "req_01H",
    "ip_address": "103.x.x.x",
    "user_agent": "Mozilla/5.0"
  },
  "payload": {}
}
```

Untuk system worker:

```json
{
  "actor": {
    "type": "system",
    "id": "worker_report_generator"
  }
}
```

Untuk device:

```json
{
  "actor": {
    "type": "device",
    "id": "edg_01H"
  }
}
```

---

# 28. Report Status Lifecycle

```text
draft
↓
queued
↓
generating
↓
generated
↓
waiting_review
↓
approved
↓
sent_to_client
↓
archived
```

Failure path:

```text
queued/generating
↓
failed
↓
retrying
↓
generated
```

Rules:

* Client hanya melihat `approved` atau `sent_to_client`.
* Draft/generated internal tidak tampil ke client.
* Report approved bersifat immutable.
* Revisi report harus membuat versi baru.
* Download report wajib audit.

---

# 29. Proof Status Lifecycle

```text
uploaded
↓
pending_review
↓
approved
```

Reject path:

```text
pending_review
↓
rejected
↓
reuploaded
↓
pending_review
```

Rules:

* Client hanya melihat approved proof.
* Teknisi bisa upload/reupload.
* Admin Tenant approve/reject.
* Owner Tenant read-only.
* Semua approval/rejection wajib audit.

---

# 30. Camera Status Lifecycle

```text
registered
↓
active
↓
online
↓
offline
↓
maintenance
↓
retired
```

Rules:

* offline otomatis jika heartbeat hilang melewati threshold,
* low data quality tidak selalu berarti offline,
* camera maintenance tidak boleh dianggap data valid normal,
* report harus mencatat missing hours dan downtime.

---

# 31. Rate Limit Rules

Rate limit minimum:

| Endpoint Area         |                 Limit Awal |
| --------------------- | -------------------------: |
| auth/login            |     5 request / menit / IP |
| report generate       |  10 request / jam / tenant |
| report download       |    60 request / jam / user |
| live stream session   |     10 create / jam / user |
| analytics             | 120 request / menit / user |
| ingest heartbeat      |     sesuai device interval |
| ingest traffic-events |   sesuai edge batch policy |

Rate limit harus dapat dikonfigurasi per tier.

---

# 32. Idempotency Rules

Write endpoint penting harus mendukung idempotency:

```http
Idempotency-Key: 8b0f1a...
```

Wajib untuk:

* report generation,
* proof upload,
* payment/subscription update jika nanti ada,
* ingestion batch,
* event processing.

Tujuannya mencegah data dobel saat retry.

---

# 33. Data Quality Rules in API

Setiap response analytics/report wajib membawa konteks kualitas data:

```json
{
  "data_quality_score": 94.2,
  "average_confidence_score": 91.8,
  "valid_hours": 736,
  "expected_hours": 744,
  "missing_hours": 8,
  "is_estimated": false,
  "anomaly_count": 2,
  "data_warning": null
}
```

Jika quality rendah:

```json
{
  "data_warning": {
    "level": "medium",
    "message": "Data quality is below recommended threshold due to missing hours."
  }
}
```

API tidak boleh menyembunyikan data quality rendah.

Dashboard cantik tidak boleh menjadi bedak untuk data yang sakit.

---

# 34. Formula Metadata Contract

Semua analytics dan report wajib menyertakan:

```json
{
  "formula_version": "exposure_v1.0",
  "methodology_version": "cvc_method_v1.0"
}
```

Untuk estimated exposure:

```json
{
  "estimated_exposure": 4875559,
  "exposure_label": "estimated_exposure",
  "formula_notes": "Estimated exposure is calculated using vehicle-class occupancy multipliers and visibility adjustment. It does not represent guaranteed human views."
}
```

Larangan:

* jangan pakai label `guaranteed_impression`,
* jangan pakai `people_seen`,
* jangan klaim pasti dilihat,
* jangan klaim pasti meningkatkan sales.

---

# 35. Frontend Integration Rules

Frontend wajib:

* mengambil menu dari `/auth/me`,
* menyembunyikan menu sesuai permission,
* tetap mengandalkan backend guard,
* tidak menyimpan token di tempat rawan,
* tidak menyimpan raw RTSP,
* tidak menghitung KPI final sendiri,
* menampilkan warning data quality,
* menampilkan status report,
* menampilkan session expiry live stream,
* menampilkan role read-only dengan jelas.

Frontend tidak boleh:

* menampilkan tombol create/edit/delete untuk Owner Tenant,
* menampilkan campaign client lain ke Client,
* menampilkan raw RTSP,
* membuat export langsung dari data mentah tanpa audit,
* memanggil endpoint tenant lain dengan manipulasi ID.

---

# 36. Backend Agent Development Rules

Agent/developer backend wajib mengikuti aturan ini:

1. Jangan membuat endpoint tanpa permission.
2. Jangan membuat endpoint tanpa tenant guard.
3. Jangan membuat report download tanpa audit.
4. Jangan membuat live stream tanpa signed session.
5. Jangan expose raw RTSP.
6. Jangan menyimpan password plain text.
7. Jangan percaya `tenant_id` dari frontend.
8. Jangan membuat angka traffic tanpa source.
9. Jangan membuat estimated exposure tanpa formula version.
10. Jangan membuat report tanpa methodology version.
11. Jangan membuat Client bisa melihat data client lain.
12. Jangan membuat Owner Tenant bisa mutate data.
13. Jangan membuat Admin Tenant bisa akses tenant lain.
14. Jangan membuat support mode tanpa reason.
15. Jangan membuat worker lintas tenant tanpa audit/system context.
16. Jangan membuat data summary dari raw event yang belum tervalidasi.
17. Jangan membuat PDF berbeda angka dengan dashboard.
18. Jangan membuat Excel/CSV tanpa permission.
19. Jangan membuat role baru tanpa update dokumen.
20. Jangan bypass audit demi “sementara”.

Aturan keras:

Fitur sementara yang menyentuh security biasanya berubah menjadi lubang permanen. Jadi jangan.

---

# 37. Minimum Testing Requirement

Setiap endpoint wajib punya test minimal:

## 37.1 Auth Test

* unauthenticated request ditolak,
* invalid token ditolak,
* expired token ditolak.

## 37.2 Permission Test

* role yang boleh berhasil,
* role yang tidak boleh gagal,
* Owner Tenant tidak bisa write,
* Client tidak bisa melihat client lain,
* Teknisi tidak bisa melihat data bisnis luas.

## 37.3 Tenant Isolation Test

* user tenant A tidak bisa akses tenant B,
* manipulasi ID resource tenant lain gagal,
* query `tenant_id` palsu diabaikan/ditolak.

## 37.4 Audit Test

Aksi berikut wajib menghasilkan audit log:

* login,
* logout,
* user role change,
* tenant status change,
* device create/update,
* live stream start/terminate,
* proof approve/reject,
* report generate,
* report approve,
* report download,
* support cross-tenant access.

## 37.5 Report Consistency Test

* angka dashboard sama dengan angka report,
* Excel/CSV sesuai report snapshot,
* formula version muncul,
* methodology version muncul,
* data quality summary muncul.

## 37.6 Live Stream Security Test

* raw RTSP tidak muncul di response,
* signed URL expired,
* user tanpa permission ditolak,
* client hanya bisa live view campaign terkait jika enabled,
* session termination berjalan.

---

# 38. MVP API Priority

## Phase API 1 — Auth, Tenant, Role

Endpoint:

```text
/auth/login
/auth/logout
/auth/me
/profile
/platform/tenants
/users
```

Target:

* login berjalan,
* tenant isolation aktif,
* role-based menu siap,
* audit log dasar aktif.

---

## Phase API 2 — Master Data

Endpoint:

```text
/clients
/billboards
/billboards/{id}/faces
/campaigns
/campaigns/{id}/access
```

Target:

* tenant bisa input client,
* tenant bisa input billboard,
* tenant bisa buat campaign,
* client campaign access mulai terkunci.

---

## Phase API 3 — Device & Live View

Endpoint:

```text
/devices
/devices/{id}/health
/live-stream/sessions
/live-stream/sessions/{id}/terminate
/ingest/heartbeat
```

Target:

* device bisa didaftarkan,
* heartbeat masuk,
* camera online/offline terbaca,
* live stream aman tanpa raw RTSP.

---

## Phase API 4 — Analytics

Endpoint:

```text
/ingest/traffic-events
/ingest/quality-snapshots
/analytics/summary
/analytics/timeseries
/analytics/vehicle-composition
/analytics/hourly-heatmap
```

Target:

* traffic data bisa masuk,
* summary bisa tampil,
* dashboard punya KPI utama,
* data quality mulai tampil.

---

## Phase API 5 — Proof & Report

Endpoint:

```text
/proof-display
/proof-display/{id}/approve
/reports
/reports/generate
/reports/{id}/approve
/reports/{id}/download/pdf
/reports/{id}/download/excel
/reports/{id}/download/csv
```

Target:

* proof display bisa upload/approve,
* report bisa generate,
* client hanya melihat approved report,
* download diaudit.

---

## Phase API 6 — Alert & Maintenance

Endpoint:

```text
/alerts
/alerts/{id}/acknowledge
/maintenance
/maintenance/{id}
```

Target:

* camera offline terdeteksi,
* missing data alert muncul,
* maintenance ticket bisa dikelola.

---

# 39. Definition of Done API

Satu endpoint dianggap selesai hanya jika:

* route dibuat,
* request validation dibuat,
* response contract konsisten,
* auth guard aktif,
* tenant guard aktif,
* permission guard aktif,
* resource ownership guard aktif,
* audit log untuk aksi penting aktif,
* error response konsisten,
* test permission ada,
* test tenant isolation ada,
* dokumentasi OpenAPI diperbarui,
* tidak expose data sensitif,
* tidak membuat angka tanpa sumber,
* tidak melanggar dokumen security.

Kalau endpoint hanya “jalan” tapi belum aman, statusnya belum done. Itu baru nyala, belum layak dipasang di jalan raya.

---

# 40. Kesimpulan

API platform ini harus menjadi tulang punggung SaaS yang aman, rapi, dan bisa diaudit.

Target API bukan sekadar membuat dashboard tampil.

Target API adalah memastikan:

* data tenant tidak bocor,
* role tidak salah kuasa,
* client hanya melihat campaign miliknya,
* live stream aman,
* report resmi bisa dipertanggungjawabkan,
* angka analytics punya sumber dan formula,
* semua aksi penting punya jejak audit,
* event pipeline dari CCTV sampai report berjalan konsisten.

Sistem ini harus mampu menjawab pertanyaan paling penting dari client:

“Angka ini dari mana, dihitung bagaimana, valid atau tidak, dan siapa yang membuat report-nya?”

Kalau API bisa menjawab itu, platform ini bukan cuma aplikasi monitoring.
Ia menjadi mesin kepercayaan untuk bisnis billboard berbasis data.
