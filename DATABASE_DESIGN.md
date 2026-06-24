# DATABASE_DESIGN.md

# SaaS Billboard Monitoring & Report Platform

## 1. Tujuan Dokumen

Dokumen ini menjadi acuan desain database untuk aplikasi **SaaS Billboard Monitoring & Report Platform**.

Database harus mendukung:

* multi-tenant SaaS,
* tenant isolation,
* role dan permission,
* billboard inventory,
* camera / CVC management,
* campaign management,
* AI traffic counting,
* vehicle classification,
* live stream session,
* proof of display,
* traffic event store,
* hourly / daily / monthly summary,
* report generation,
* report snapshot,
* signed export,
* data quality score,
* confidence score,
* anomaly tracking,
* audit log,
* subscription / tier management.

Database tidak boleh hanya dibuat untuk “sekadar tampil dashboard”.

Database harus menjadi tulang punggung agar data traffic, report, live view, dan akses client bisa dipertanggungjawabkan.

Prinsip galak:

> Kalau API gagah tapi database lembek, aplikasi akan terlihat seperti mobil sport dengan rangka gerobak. Bisa jalan, tapi jangan mimpi diajak produksi serius.

---

## 2. Database Engine yang Disarankan

Untuk versi production, gunakan:

```text
PostgreSQL
```

Alasan:

* kuat untuk multi-tenant SaaS,
* mendukung JSONB,
* mendukung indexing yang matang,
* mendukung partitioning untuk event traffic besar,
* mendukung Row Level Security jika nanti dibutuhkan,
* cocok untuk audit log dan time-series summary,
* bisa memakai PostGIS untuk koordinat lokasi billboard jika dibutuhkan.

Untuk MVP simulasi lokal boleh memakai SQLite, tetapi struktur tabel harus tetap mengikuti desain PostgreSQL agar tidak bongkar ulang saat naik production.

---

## 3. Prinsip Desain Utama

## 3.1 Multi-Tenant Sejak Awal

Semua tabel operasional tenant wajib memiliki:

```text
tenant_id
```

Contoh tabel yang wajib punya `tenant_id`:

* users tenant,
* clients,
* billboards,
* billboard_faces,
* cameras,
* edge_devices,
* campaigns,
* campaign_billboards,
* proof_of_display,
* traffic_events,
* traffic_hourly_summaries,
* traffic_daily_summaries,
* traffic_monthly_summaries,
* reports,
* report_exports,
* live_stream_sessions,
* camera_health_logs,
* alert_events,
* maintenance_tickets,
* audit_logs.

Tabel global platform seperti `subscription_plans`, `platform_settings`, dan `global_feature_flags` boleh tidak punya `tenant_id`.

---

## 3.2 Tenant Isolation Tidak Boleh Hanya di UI

Tenant isolation wajib diterapkan di:

```text
UI
API guard
Service layer
Database query
Audit log
Report export
Live stream session
```

Query tenant tidak boleh seperti ini:

```sql
SELECT * FROM campaigns WHERE id = :campaign_id;
```

Query wajib seperti ini:

```sql
SELECT * FROM campaigns
WHERE id = :campaign_id
AND tenant_id = :current_tenant_id;
```

Kecuali untuk Owner Platform SaaS pada mode platform support resmi, dan tetap harus masuk audit log.

---

## 3.3 UUID untuk Primary Key

Gunakan UUID untuk ID utama.

Contoh:

```text
tenant_id UUID
user_id UUID
billboard_id UUID
camera_id UUID
campaign_id UUID
report_id UUID
```

Jangan memakai ID incremental untuk data sensitif multi-tenant, karena mudah ditebak.

---

## 3.4 Timestamp Standar

Setiap tabel utama minimal punya:

```text
created_at TIMESTAMPTZ
updated_at TIMESTAMPTZ
deleted_at TIMESTAMPTZ NULL
```

Untuk data event:

```text
captured_at TIMESTAMPTZ
received_at TIMESTAMPTZ
processed_at TIMESTAMPTZ
```

Semua waktu disimpan dalam UTC. UI boleh menampilkan timezone lokal tenant.

---

## 3.5 Soft Delete untuk Master Data

Master data seperti tenant, user, billboard, camera, campaign, client tidak boleh langsung hard delete.

Gunakan:

```text
deleted_at
deleted_by
delete_reason
```

Hard delete hanya boleh untuk data sementara/cache yang memang aman dihapus.

---

## 3.6 Event Store Harus Append-Only

Tabel seperti `traffic_events` tidak boleh sembarang di-update.

Prinsip:

```text
raw event masuk
validasi dilakukan
summary dibuat
jika koreksi, buat correction event / audit note
jangan diam-diam menimpa sejarah
```

Data traffic itu seperti rekaman saksi. Kalau diedit diam-diam, besok report bisa jadi cerita rakyat.

---

## 3.7 Report Final Harus Snapshot

Report final tidak boleh bergantung sepenuhnya pada query live.

Saat report disetujui, simpan snapshot:

```text
angka final
formula version
methodology version
data quality summary
source data range
generated_by
approved_by
file hash
export URL
```

Alasannya:

* report bulan Maret yang sudah dikirim ke client harus tetap sama meskipun data mentah nanti dikoreksi,
* client butuh konsistensi,
* audit butuh jejak,
* aplikasi butuh bukti.

---

## 4. Konvensi Nama Tabel

Gunakan nama tabel plural snake_case.

Contoh:

```text
tenants
users
billboards
cameras
traffic_events
reports
audit_logs
```

Gunakan nama kolom snake_case.

Contoh:

```text
tenant_id
camera_id
total_vehicle_count
data_quality_score
formula_version_id
```

---

# 5. Struktur Modul Database

Database dibagi menjadi 12 kelompok utama:

```text
1. Platform & Tenant
2. Subscription & Tier
3. User, Role & Permission
4. Client & Brand
5. Billboard Inventory
6. Camera, Device & Live Stream
7. Campaign
8. AI Counting Rule & Formula
9. Traffic Event Store & Summary
10. Proof of Display
11. Report, Snapshot & Export
12. Alert, Maintenance & Audit
```

---

# 6. Platform & Tenant Tables

## 6.1 tenants

Menyimpan perusahaan billboard yang menjadi pelanggan SaaS.

```text
tenants
- id UUID PK
- tenant_code VARCHAR UNIQUE
- company_name VARCHAR
- legal_name VARCHAR NULL
- industry VARCHAR DEFAULT 'outdoor_advertising'
- status VARCHAR
  -- trial, active, suspended, cancelled, archived
- plan_id UUID FK subscription_plans.id NULL
- timezone VARCHAR DEFAULT 'Asia/Jakarta'
- default_locale VARCHAR DEFAULT 'id-ID'
- billing_email VARCHAR NULL
- support_email VARCHAR NULL
- phone VARCHAR NULL
- website_url VARCHAR NULL
- logo_url TEXT NULL
- brand_primary_color VARCHAR NULL
- brand_secondary_color VARCHAR NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
- deleted_at TIMESTAMPTZ NULL
```

Catatan:

* Tenant bukan client campaign.
* Tenant adalah perusahaan billboard / advertising owner.
* Client adalah brand penyewa billboard.

---

## 6.2 tenant_profiles

Menyimpan detail tambahan tenant.

```text
tenant_profiles
- id UUID PK
- tenant_id UUID FK tenants.id
- address TEXT NULL
- city VARCHAR NULL
- province VARCHAR NULL
- country VARCHAR DEFAULT 'Indonesia'
- postal_code VARCHAR NULL
- tax_number VARCHAR NULL
- business_license_number VARCHAR NULL
- contact_person_name VARCHAR NULL
- contact_person_phone VARCHAR NULL
- contact_person_email VARCHAR NULL
- company_description TEXT NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

Index:

```sql
CREATE INDEX idx_tenant_profiles_tenant_id ON tenant_profiles(tenant_id);
```

---

# 7. Subscription & Tier Tables

## 7.1 subscription_plans

Paket SaaS yang dikelola oleh Owner Platform SaaS.

```text
subscription_plans
- id UUID PK
- plan_code VARCHAR UNIQUE
- plan_name VARCHAR
- description TEXT NULL
- monthly_price NUMERIC(14,2) DEFAULT 0
- yearly_price NUMERIC(14,2) DEFAULT 0
- currency VARCHAR DEFAULT 'IDR'
- status VARCHAR
  -- active, inactive, archived
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

---

## 7.2 tier_feature_limits

Limit fitur berdasarkan paket.

```text
tier_feature_limits
- id UUID PK
- plan_id UUID FK subscription_plans.id
- max_billboards INT
- max_billboard_faces INT
- max_cameras INT
- max_users INT
- max_clients INT
- max_campaigns_active INT
- allow_pdf_export BOOLEAN DEFAULT true
- allow_excel_export BOOLEAN DEFAULT false
- allow_csv_export BOOLEAN DEFAULT false
- allow_live_view BOOLEAN DEFAULT false
- allow_data_quality_score BOOLEAN DEFAULT false
- allow_anomaly_detection BOOLEAN DEFAULT false
- allow_white_label_report BOOLEAN DEFAULT false
- data_retention_days INT DEFAULT 90
- raw_event_retention_days INT DEFAULT 30
- max_live_stream_minutes INT DEFAULT 10
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

Catatan:

* Data retention harus dikunci dari awal.
* Raw traffic event bisa sangat besar.
* Summary boleh disimpan lebih lama daripada raw event.

---

## 7.3 tenant_subscriptions

Subscription aktif tenant.

```text
tenant_subscriptions
- id UUID PK
- tenant_id UUID FK tenants.id
- plan_id UUID FK subscription_plans.id
- status VARCHAR
  -- trial, active, past_due, suspended, cancelled
- start_date DATE
- end_date DATE NULL
- trial_ends_at TIMESTAMPTZ NULL
- billing_cycle VARCHAR
  -- monthly, yearly, custom
- current_period_start DATE
- current_period_end DATE
- auto_renew BOOLEAN DEFAULT false
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

Index:

```sql
CREATE INDEX idx_tenant_subscriptions_tenant_status
ON tenant_subscriptions(tenant_id, status);
```

---

# 8. User, Role & Permission Tables

## 8.1 users

Menyimpan user semua role.

```text
users
- id UUID PK
- tenant_id UUID NULL
  -- NULL hanya untuk Owner Platform SaaS
- email VARCHAR UNIQUE
- password_hash TEXT
- full_name VARCHAR
- phone VARCHAR NULL
- avatar_url TEXT NULL
- status VARCHAR
  -- active, invited, suspended, disabled
- user_type VARCHAR
  -- platform_owner, tenant_staff, tenant_owner, client_user, technician
- last_login_at TIMESTAMPTZ NULL
- password_changed_at TIMESTAMPTZ NULL
- must_change_password BOOLEAN DEFAULT false
- mfa_enabled BOOLEAN DEFAULT false
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
- deleted_at TIMESTAMPTZ NULL
```

Aturan:

* Owner Platform SaaS boleh `tenant_id = NULL`.
* Admin Tenant, Owner Tenant, Sales, Teknisi wajib punya `tenant_id`.
* Client user wajib punya `tenant_id` dan relasi ke `clients`.

---

## 8.2 roles

```text
roles
- id UUID PK
- role_code VARCHAR UNIQUE
- role_name VARCHAR
- scope VARCHAR
  -- platform, tenant, client
- description TEXT NULL
- is_system_role BOOLEAN DEFAULT true
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

Role awal:

```text
owner_platform_saas
admin_tenant
owner_tenant
sales_tenant
technician
client
viewer
```

---

## 8.3 permissions

```text
permissions
- id UUID PK
- permission_code VARCHAR UNIQUE
- module VARCHAR
- action VARCHAR
  -- create, read, update, delete, approve, export, stream, manage
- description TEXT NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

Contoh permission:

```text
tenant.manage
subscription.manage
billboard.create
billboard.read
billboard.update
billboard.delete
camera.manage
camera.live_view
campaign.manage
proof.approve
report.generate
report.approve
report.download
analytics.read
audit.read
```

---

## 8.4 role_permissions

```text
role_permissions
- id UUID PK
- role_id UUID FK roles.id
- permission_id UUID FK permissions.id
- created_at TIMESTAMPTZ
```

Unique:

```sql
CREATE UNIQUE INDEX uq_role_permissions
ON role_permissions(role_id, permission_id);
```

---

## 8.5 user_roles

```text
user_roles
- id UUID PK
- tenant_id UUID NULL
- user_id UUID FK users.id
- role_id UUID FK roles.id
- assigned_by UUID FK users.id NULL
- assigned_at TIMESTAMPTZ
```

Catatan:

* `tenant_id` harus sama dengan `users.tenant_id` untuk user tenant.
* Owner Platform SaaS boleh tanpa tenant.

---

## 8.6 user_sessions

```text
user_sessions
- id UUID PK
- user_id UUID FK users.id
- tenant_id UUID NULL
- refresh_token_hash TEXT
- ip_address INET NULL
- user_agent TEXT NULL
- device_name VARCHAR NULL
- status VARCHAR
  -- active, revoked, expired
- expires_at TIMESTAMPTZ
- revoked_at TIMESTAMPTZ NULL
- created_at TIMESTAMPTZ
```

---

# 9. Client & Brand Tables

## 9.1 clients

Client adalah brand / perusahaan penyewa billboard.

```text
clients
- id UUID PK
- tenant_id UUID FK tenants.id
- company_name VARCHAR
- brand_name VARCHAR NULL
- industry VARCHAR NULL
- contact_person_name VARCHAR NULL
- contact_person_email VARCHAR NULL
- contact_person_phone VARCHAR NULL
- logo_url TEXT NULL
- status VARCHAR
  -- active, inactive, archived
- created_by UUID FK users.id NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
- deleted_at TIMESTAMPTZ NULL
```

Index:

```sql
CREATE INDEX idx_clients_tenant_id ON clients(tenant_id);
```

---

## 9.2 client_users

Menghubungkan user login dengan client.

```text
client_users
- id UUID PK
- tenant_id UUID FK tenants.id
- client_id UUID FK clients.id
- user_id UUID FK users.id
- status VARCHAR
  -- active, suspended
- can_view_live BOOLEAN DEFAULT false
- can_download_report BOOLEAN DEFAULT true
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

Unique:

```sql
CREATE UNIQUE INDEX uq_client_users
ON client_users(tenant_id, client_id, user_id);
```

---

# 10. Billboard Inventory Tables

## 10.1 billboards

Master titik billboard.

```text
billboards
- id UUID PK
- tenant_id UUID FK tenants.id
- billboard_code VARCHAR
- name VARCHAR
- description TEXT NULL
- address TEXT
- city VARCHAR
- province VARCHAR
- country VARCHAR DEFAULT 'Indonesia'
- latitude NUMERIC(10,7) NULL
- longitude NUMERIC(10,7) NULL
- road_type VARCHAR
  -- toll, arterial, jpo, intersection, flyover, urban, custom
- package_type VARCHAR
  -- toll, single_face, intersection, premium_landmark
- size_width_meter NUMERIC(8,2) NULL
- size_height_meter NUMERIC(8,2) NULL
- monthly_rate NUMERIC(14,2) NULL
- estimated_ooh_budget NUMERIC(14,2) NULL
- status VARCHAR
  -- active, inactive, maintenance, archived
- location_photo_url TEXT NULL
- map_snapshot_url TEXT NULL
- created_by UUID FK users.id NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
- deleted_at TIMESTAMPTZ NULL
```

Unique:

```sql
CREATE UNIQUE INDEX uq_billboards_tenant_code
ON billboards(tenant_id, billboard_code)
WHERE deleted_at IS NULL;
```

---

## 10.2 billboard_faces

Sisi / muka billboard.

```text
billboard_faces
- id UUID PK
- tenant_id UUID FK tenants.id
- billboard_id UUID FK billboards.id
- face_code VARCHAR
- face_name VARCHAR
- orientation VARCHAR
  -- north, east, south, west, custom
- facing_direction_degrees NUMERIC(6,2) NULL
- visibility_angle_degrees NUMERIC(6,2) NULL
- view_distance_meter NUMERIC(10,2) NULL
- is_primary BOOLEAN DEFAULT true
- status VARCHAR
  -- active, inactive, maintenance
- creative_photo_url TEXT NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
- deleted_at TIMESTAMPTZ NULL
```

Catatan:

* Billboard satu muka tetap punya minimal 1 record di `billboard_faces`.
* Billboard perempatan bisa punya beberapa face / exposure direction.

---

# 11. Camera, Device & Live Stream Tables

## 11.1 edge_devices

Device fisik di lapangan.

```text
edge_devices
- id UUID PK
- tenant_id UUID FK tenants.id
- device_code VARCHAR
- device_name VARCHAR
- device_type VARCHAR
  -- cvc_box, nvr, ip_camera_gateway, edge_ai
- serial_number VARCHAR NULL
- hardware_id VARCHAR NULL
- firmware_version VARCHAR NULL
- installed_at TIMESTAMPTZ NULL
- last_heartbeat_at TIMESTAMPTZ NULL
- status VARCHAR
  -- online, offline, degraded, maintenance, retired
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
- deleted_at TIMESTAMPTZ NULL
```

---

## 11.2 cameras

Master kamera / CVC.

```text
cameras
- id UUID PK
- tenant_id UUID FK tenants.id
- edge_device_id UUID FK edge_devices.id NULL
- billboard_id UUID FK billboards.id
- billboard_face_id UUID FK billboard_faces.id NULL
- camera_code VARCHAR
- camera_name VARCHAR
- camera_position VARCHAR
  -- front, left, right, rear, proof_display, custom
- captured_direction VARCHAR
  -- forward, opposite, left_to_right, right_to_left, multi_direction, custom
- stream_url_encrypted TEXT NULL
- stream_proxy_path TEXT NULL
- stream_provider VARCHAR NULL
- fps INT NULL
- resolution_width INT NULL
- resolution_height INT NULL
- latency_ms INT NULL
- status VARCHAR
  -- online, offline, degraded, maintenance, disabled
- last_heartbeat_at TIMESTAMPTZ NULL
- last_frame_at TIMESTAMPTZ NULL
- health_score NUMERIC(5,2) NULL
- notes TEXT NULL
- created_by UUID FK users.id NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
- deleted_at TIMESTAMPTZ NULL
```

Larangan:

```text
Jangan tampilkan stream_url_encrypted ke client.
Jangan simpan raw RTSP plain text.
Jangan izinkan frontend menerima RTSP asli.
Live stream wajib lewat stream proxy dan signed session.
```

---

## 11.3 camera_health_logs

Log kesehatan kamera.

```text
camera_health_logs
- id UUID PK
- tenant_id UUID FK tenants.id
- camera_id UUID FK cameras.id
- checked_at TIMESTAMPTZ
- status VARCHAR
  -- online, offline, degraded
- fps NUMERIC(8,2) NULL
- resolution_width INT NULL
- resolution_height INT NULL
- latency_ms INT NULL
- packet_loss_percent NUMERIC(5,2) NULL
- frame_quality_score NUMERIC(5,2) NULL
- blur_score NUMERIC(5,2) NULL
- brightness_score NUMERIC(5,2) NULL
- occlusion_score NUMERIC(5,2) NULL
- uptime_minutes INT DEFAULT 0
- error_message TEXT NULL
- created_at TIMESTAMPTZ
```

Index:

```sql
CREATE INDEX idx_camera_health_logs_camera_time
ON camera_health_logs(tenant_id, camera_id, checked_at DESC);
```

---

## 11.4 camera_snapshots

Snapshot kamera untuk bukti monitoring dan report.

```text
camera_snapshots
- id UUID PK
- tenant_id UUID FK tenants.id
- camera_id UUID FK cameras.id
- billboard_id UUID FK billboards.id
- campaign_id UUID FK campaigns.id NULL
- snapshot_type VARCHAR
  -- live_view, report, proof, anomaly, scheduled
- image_url TEXT
- captured_at TIMESTAMPTZ
- captured_by UUID FK users.id NULL
- metadata_json JSONB NULL
- created_at TIMESTAMPTZ
```

---

## 11.5 live_stream_sessions

Session akses live stream.

```text
live_stream_sessions
- id UUID PK
- tenant_id UUID FK tenants.id
- camera_id UUID FK cameras.id
- user_id UUID FK users.id
- client_id UUID FK clients.id NULL
- campaign_id UUID FK campaigns.id NULL
- signed_token_hash TEXT
- stream_proxy_url TEXT
- status VARCHAR
  -- active, expired, revoked, ended, failed
- started_at TIMESTAMPTZ
- expires_at TIMESTAMPTZ
- ended_at TIMESTAMPTZ NULL
- ip_address INET NULL
- user_agent TEXT NULL
- viewer_role VARCHAR
- reason TEXT NULL
- created_at TIMESTAMPTZ
```

Constraint penting:

```sql
CREATE INDEX idx_live_stream_sessions_active
ON live_stream_sessions(tenant_id, camera_id, user_id, status);
```

Jika memakai aturan satu user hanya boleh satu live stream aktif per kamera:

```sql
CREATE UNIQUE INDEX uq_active_live_stream_per_user_camera
ON live_stream_sessions(tenant_id, camera_id, user_id)
WHERE status = 'active';
```

Aturan:

* live stream harus punya expiry,
* semua akses live stream masuk audit log,
* client hanya boleh live view jika campaign terkait dan permission aktif,
* Owner Tenant read-only boleh melihat jika tenant mengizinkan,
* raw RTSP tidak pernah keluar ke browser.

---

# 12. Campaign Tables

## 12.1 campaigns

Master campaign iklan.

```text
campaigns
- id UUID PK
- tenant_id UUID FK tenants.id
- client_id UUID FK clients.id
- campaign_code VARCHAR
- campaign_name VARCHAR
- brand_name VARCHAR NULL
- objective VARCHAR NULL
  -- awareness, product_launch, tactical, automotive, property, custom
- start_date DATE
- end_date DATE
- status VARCHAR
  -- draft, active, paused, completed, cancelled, archived
- estimated_ooh_budget NUMERIC(14,2) NULL
- report_access_enabled BOOLEAN DEFAULT true
- dashboard_access_enabled BOOLEAN DEFAULT true
- live_view_enabled BOOLEAN DEFAULT false
- created_by UUID FK users.id NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
- deleted_at TIMESTAMPTZ NULL
```

Index:

```sql
CREATE INDEX idx_campaigns_tenant_client_status
ON campaigns(tenant_id, client_id, status);
```

---

## 12.2 campaign_billboards

Relasi campaign dengan billboard / face / camera.

```text
campaign_billboards
- id UUID PK
- tenant_id UUID FK tenants.id
- campaign_id UUID FK campaigns.id
- billboard_id UUID FK billboards.id
- billboard_face_id UUID FK billboard_faces.id NULL
- primary_camera_id UUID FK cameras.id NULL
- start_date DATE
- end_date DATE
- booked_rate NUMERIC(14,2) NULL
- status VARCHAR
  -- active, paused, completed, removed
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

Catatan:

* Satu campaign bisa memakai banyak billboard.
* Satu billboard bisa punya beberapa kamera untuk persimpangan.
* Untuk report per lokasi, data harus bisa ditarik dari `campaign_billboards`.

---

## 12.3 campaign_creatives

Materi iklan campaign.

```text
campaign_creatives
- id UUID PK
- tenant_id UUID FK tenants.id
- campaign_id UUID FK campaigns.id
- billboard_id UUID FK billboards.id NULL
- creative_name VARCHAR
- creative_type VARCHAR
  -- image, video, print_design, other
- file_url TEXT
- installed_at TIMESTAMPTZ NULL
- status VARCHAR
  -- draft, installed, approved, replaced, archived
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

---

# 13. AI Counting Rule & Formula Tables

## 13.1 ai_counting_rules

Aturan counting per kamera.

```text
ai_counting_rules
- id UUID PK
- tenant_id UUID FK tenants.id
- camera_id UUID FK cameras.id
- rule_name VARCHAR
- counting_line_coordinates JSONB
- direction_filter VARCHAR
  -- inbound, outbound, left_to_right, right_to_left, both, custom
- exposure_zone_coordinates JSONB NULL
- roi_coordinates JSONB NULL
- vehicle_classes JSONB
  -- motorcycle, car, bus, truck
- minimum_confidence NUMERIC(5,2) DEFAULT 0.60
- duplicate_tracking_window_seconds INT DEFAULT 3
- speed_calibration_json JSONB NULL
- active_status BOOLEAN DEFAULT true
- version INT DEFAULT 1
- created_by UUID FK users.id NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

Catatan:

* Counting line harus disimpan.
* Exposure zone harus disimpan.
* Direction rule harus disimpan.
* Kalau rule berubah, jangan overwrite tanpa versioning.

---

## 13.2 methodology_versions

Versi metodologi report.

```text
methodology_versions
- id UUID PK
- version_code VARCHAR UNIQUE
- title VARCHAR
- description TEXT
- vehicle_counting_method TEXT
- confidence_policy TEXT
- data_quality_policy TEXT
- downtime_handling_policy TEXT
- impression_policy TEXT
- overclaim_disclaimer TEXT
- status VARCHAR
  -- draft, active, deprecated
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

---

## 13.3 formula_versions

Versi formula perhitungan.

```text
formula_versions
- id UUID PK
- version_code VARCHAR UNIQUE
- title VARCHAR
- total_vehicle_formula TEXT
- estimated_exposure_formula TEXT
- cpv_formula TEXT
- cpi_formula TEXT
- data_quality_formula TEXT
- status VARCHAR
  -- draft, active, deprecated
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

---

## 13.4 occupancy_multipliers

Multiplier estimasi exposure per jenis kendaraan.

```text
occupancy_multipliers
- id UUID PK
- formula_version_id UUID FK formula_versions.id
- vehicle_class VARCHAR
  -- motorcycle, car, bus, truck
- multiplier NUMERIC(8,3)
- description TEXT NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

Catatan:

* Jangan hardcode multiplier di frontend.
* Formula harus bisa diaudit.
* Jika multiplier berubah, report lama tetap memakai formula_version lama.

---

# 14. Traffic Event Store & Summary Tables

## 14.1 traffic_events

Event mentah hasil AI counting.

```text
traffic_events
- id UUID PK
- tenant_id UUID FK tenants.id
- camera_id UUID FK cameras.id
- billboard_id UUID FK billboards.id
- billboard_face_id UUID FK billboard_faces.id NULL
- campaign_id UUID FK campaigns.id NULL
- rule_id UUID FK ai_counting_rules.id NULL
- track_id VARCHAR NULL
- vehicle_class VARCHAR
  -- motorcycle, car, bus, truck, unknown
- confidence_score NUMERIC(5,2)
- speed_kmh NUMERIC(8,2) NULL
- direction VARCHAR NULL
- lane_id VARCHAR NULL
- bbox_json JSONB NULL
- crossing_line_at TIMESTAMPTZ NULL
- captured_at TIMESTAMPTZ
- received_at TIMESTAMPTZ
- processed_at TIMESTAMPTZ NULL
- data_status VARCHAR
  -- raw, validated, estimated, excluded
- exclusion_reason TEXT NULL
- metadata_json JSONB NULL
- created_at TIMESTAMPTZ
```

Index wajib:

```sql
CREATE INDEX idx_traffic_events_tenant_camera_time
ON traffic_events(tenant_id, camera_id, captured_at DESC);

CREATE INDEX idx_traffic_events_tenant_campaign_time
ON traffic_events(tenant_id, campaign_id, captured_at DESC);

CREATE INDEX idx_traffic_events_vehicle_class
ON traffic_events(tenant_id, vehicle_class, captured_at DESC);
```

Partitioning:

```text
traffic_events sebaiknya dipartisi per bulan berdasarkan captured_at.
```

---

## 14.2 traffic_hourly_summaries

Summary per jam.

```text
traffic_hourly_summaries
- id UUID PK
- tenant_id UUID FK tenants.id
- campaign_id UUID FK campaigns.id NULL
- billboard_id UUID FK billboards.id
- billboard_face_id UUID FK billboard_faces.id NULL
- camera_id UUID FK cameras.id
- summary_date DATE
- summary_hour INT
- bucket_start TIMESTAMPTZ
- bucket_end TIMESTAMPTZ
- motorcycle_count INT DEFAULT 0
- car_count INT DEFAULT 0
- bus_count INT DEFAULT 0
- truck_count INT DEFAULT 0
- unknown_count INT DEFAULT 0
- total_vehicle_count INT DEFAULT 0
- estimated_exposure NUMERIC(18,2) DEFAULT 0
- avg_speed_kmh NUMERIC(8,2) NULL
- congestion_level VARCHAR NULL
  -- low, medium, high, severe, unknown
- avg_confidence_score NUMERIC(5,2) NULL
- data_quality_score NUMERIC(5,2) NULL
- camera_uptime_minutes INT DEFAULT 0
- expected_minutes INT DEFAULT 60
- valid_minutes INT DEFAULT 0
- missing_minutes INT DEFAULT 0
- is_estimated BOOLEAN DEFAULT false
- anomaly_flag BOOLEAN DEFAULT false
- formula_version_id UUID FK formula_versions.id NULL
- methodology_version_id UUID FK methodology_versions.id NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

Unique:

```sql
CREATE UNIQUE INDEX uq_traffic_hourly_summary
ON traffic_hourly_summaries(tenant_id, camera_id, bucket_start);
```

---

## 14.3 traffic_daily_summaries

Summary harian.

```text
traffic_daily_summaries
- id UUID PK
- tenant_id UUID FK tenants.id
- campaign_id UUID FK campaigns.id NULL
- billboard_id UUID FK billboards.id
- billboard_face_id UUID FK billboard_faces.id NULL
- camera_id UUID FK cameras.id NULL
- summary_date DATE
- motorcycle_count INT DEFAULT 0
- car_count INT DEFAULT 0
- bus_count INT DEFAULT 0
- truck_count INT DEFAULT 0
- unknown_count INT DEFAULT 0
- total_vehicle_count INT DEFAULT 0
- estimated_exposure NUMERIC(18,2) DEFAULT 0
- avg_speed_kmh NUMERIC(8,2) NULL
- peak_hour INT NULL
- peak_hour_vehicle_count INT NULL
- dominant_vehicle_class VARCHAR NULL
- avg_confidence_score NUMERIC(5,2) NULL
- data_quality_score NUMERIC(5,2) NULL
- expected_hours INT DEFAULT 24
- valid_hours INT DEFAULT 0
- missing_hours INT DEFAULT 0
- anomaly_count INT DEFAULT 0
- estimated_hours INT DEFAULT 0
- formula_version_id UUID FK formula_versions.id NULL
- methodology_version_id UUID FK methodology_versions.id NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

Unique:

```sql
CREATE UNIQUE INDEX uq_traffic_daily_summary
ON traffic_daily_summaries(tenant_id, billboard_id, camera_id, summary_date);
```

---

## 14.4 traffic_monthly_summaries

Summary bulanan.

```text
traffic_monthly_summaries
- id UUID PK
- tenant_id UUID FK tenants.id
- campaign_id UUID FK campaigns.id NULL
- billboard_id UUID FK billboards.id
- billboard_face_id UUID FK billboard_faces.id NULL
- camera_id UUID FK cameras.id NULL
- summary_month DATE
  -- gunakan tanggal pertama bulan tersebut
- motorcycle_count INT DEFAULT 0
- car_count INT DEFAULT 0
- bus_count INT DEFAULT 0
- truck_count INT DEFAULT 0
- unknown_count INT DEFAULT 0
- total_vehicle_count INT DEFAULT 0
- estimated_exposure NUMERIC(18,2) DEFAULT 0
- average_daily_traffic NUMERIC(18,2) DEFAULT 0
- avg_speed_kmh NUMERIC(8,2) NULL
- peak_day DATE NULL
- peak_hour INT NULL
- dominant_vehicle_class VARCHAR NULL
- avg_confidence_score NUMERIC(5,2) NULL
- data_quality_score NUMERIC(5,2) NULL
- expected_hours INT DEFAULT 0
- valid_hours INT DEFAULT 0
- missing_hours INT DEFAULT 0
- anomaly_count INT DEFAULT 0
- estimated_hours INT DEFAULT 0
- formula_version_id UUID FK formula_versions.id NULL
- methodology_version_id UUID FK methodology_versions.id NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

---

## 14.5 data_quality_snapshots

Snapshot kualitas data.

```text
data_quality_snapshots
- id UUID PK
- tenant_id UUID FK tenants.id
- camera_id UUID FK cameras.id
- billboard_id UUID FK billboards.id
- campaign_id UUID FK campaigns.id NULL
- period_type VARCHAR
  -- hourly, daily, weekly, monthly, custom
- period_start TIMESTAMPTZ
- period_end TIMESTAMPTZ
- camera_uptime_score NUMERIC(5,2)
- detection_confidence_score NUMERIC(5,2)
- frame_quality_score NUMERIC(5,2)
- missing_data_score NUMERIC(5,2)
- final_data_quality_score NUMERIC(5,2)
- expected_minutes INT
- valid_minutes INT
- missing_minutes INT
- avg_confidence_score NUMERIC(5,2)
- anomaly_count INT DEFAULT 0
- notes TEXT NULL
- created_at TIMESTAMPTZ
```

---

## 14.6 anomaly_events

Menyimpan anomali traffic.

```text
anomaly_events
- id UUID PK
- tenant_id UUID FK tenants.id
- camera_id UUID FK cameras.id
- billboard_id UUID FK billboards.id
- campaign_id UUID FK campaigns.id NULL
- anomaly_type VARCHAR
  -- traffic_drop, traffic_spike, camera_offline, low_confidence, missing_data, duplicate_suspected
- severity VARCHAR
  -- critical, high, medium, low, info
- detected_at TIMESTAMPTZ
- period_start TIMESTAMPTZ
- period_end TIMESTAMPTZ
- affected_vehicle_class VARCHAR NULL
- description TEXT
- possible_reason TEXT NULL
- is_included_in_report BOOLEAN DEFAULT true
- reviewed_by UUID FK users.id NULL
- reviewed_at TIMESTAMPTZ NULL
- status VARCHAR
  -- open, reviewed, ignored, resolved
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

---

# 15. Proof of Display Tables

## 15.1 proof_of_display

Bukti billboard tayang.

```text
proof_of_display
- id UUID PK
- tenant_id UUID FK tenants.id
- campaign_id UUID FK campaigns.id
- billboard_id UUID FK billboards.id
- billboard_face_id UUID FK billboard_faces.id NULL
- uploaded_by UUID FK users.id
- proof_type VARCHAR
  -- installation_photo, periodic_photo, maintenance_photo, creative_photo, proof_camera_snapshot
- file_url TEXT
- thumbnail_url TEXT NULL
- captured_at TIMESTAMPTZ NULL
- uploaded_at TIMESTAMPTZ
- gps_latitude NUMERIC(10,7) NULL
- gps_longitude NUMERIC(10,7) NULL
- notes TEXT NULL
- approval_status VARCHAR
  -- pending, approved, rejected
- approved_by UUID FK users.id NULL
- approved_at TIMESTAMPTZ NULL
- rejection_reason TEXT NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

Aturan:

* Client hanya melihat proof yang approved.
* Teknisi boleh upload sesuai assignment.
* Admin Tenant approval.
* Owner Tenant read-only.

---

# 16. Report, Snapshot & Export Tables

## 16.1 report_generation_jobs

Job generate report.

```text
report_generation_jobs
- id UUID PK
- tenant_id UUID FK tenants.id
- campaign_id UUID FK campaigns.id NULL
- requested_by UUID FK users.id
- report_type VARCHAR
  -- daily, weekly, monthly, campaign, billboard, client
- period_start DATE
- period_end DATE
- status VARCHAR
  -- queued, processing, generated, failed, cancelled
- progress_percent INT DEFAULT 0
- error_message TEXT NULL
- started_at TIMESTAMPTZ NULL
- finished_at TIMESTAMPTZ NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

---

## 16.2 reports

Master report.

```text
reports
- id UUID PK
- tenant_id UUID FK tenants.id
- campaign_id UUID FK campaigns.id NULL
- client_id UUID FK clients.id NULL
- report_job_id UUID FK report_generation_jobs.id NULL
- report_code VARCHAR
- report_title VARCHAR
- report_type VARCHAR
  -- daily, weekly, monthly, campaign, billboard, client
- period_start DATE
- period_end DATE
- report_status VARCHAR
  -- draft, generated, waiting_review, approved, sent_to_client, failed, archived
- report_version INT DEFAULT 1
- formula_version_id UUID FK formula_versions.id
- methodology_version_id UUID FK methodology_versions.id
- generated_by UUID FK users.id
- generated_at TIMESTAMPTZ
- approved_by UUID FK users.id NULL
- approved_at TIMESTAMPTZ NULL
- sent_to_client_at TIMESTAMPTZ NULL
- data_quality_summary_json JSONB NULL
- executive_summary_json JSONB NULL
- recommendation_json JSONB NULL
- snapshot_json JSONB NULL
- source_range_json JSONB NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

Catatan:

* `snapshot_json` menyimpan angka final report.
* Report yang sudah approved tidak boleh berubah diam-diam.
* Jika perlu revisi, buat report version baru.

---

## 16.3 report_location_snapshots

Snapshot KPI per lokasi di report.

```text
report_location_snapshots
- id UUID PK
- tenant_id UUID FK tenants.id
- report_id UUID FK reports.id
- campaign_id UUID FK campaigns.id NULL
- billboard_id UUID FK billboards.id
- billboard_face_id UUID FK billboard_faces.id NULL
- camera_id UUID FK cameras.id NULL
- location_name VARCHAR
- address TEXT NULL
- total_vehicle_count INT DEFAULT 0
- motorcycle_count INT DEFAULT 0
- car_count INT DEFAULT 0
- bus_count INT DEFAULT 0
- truck_count INT DEFAULT 0
- estimated_exposure NUMERIC(18,2) DEFAULT 0
- average_daily_traffic NUMERIC(18,2) DEFAULT 0
- cpv NUMERIC(14,2) NULL
- cpi NUMERIC(14,2) NULL
- estimated_ooh_budget NUMERIC(14,2) NULL
- avg_confidence_score NUMERIC(5,2) NULL
- data_quality_score NUMERIC(5,2) NULL
- peak_day DATE NULL
- peak_hour INT NULL
- dominant_vehicle_class VARCHAR NULL
- anomaly_count INT DEFAULT 0
- valid_hours INT DEFAULT 0
- missing_hours INT DEFAULT 0
- created_at TIMESTAMPTZ
```

---

## 16.4 report_exports

File export report.

```text
report_exports
- id UUID PK
- tenant_id UUID FK tenants.id
- report_id UUID FK reports.id
- export_type VARCHAR
  -- pdf, excel, csv, json
- file_url TEXT
- file_hash TEXT NULL
- file_size_bytes BIGINT NULL
- status VARCHAR
  -- generated, expired, revoked, failed
- generated_by UUID FK users.id
- generated_at TIMESTAMPTZ
- expires_at TIMESTAMPTZ NULL
- created_at TIMESTAMPTZ
```

---

## 16.5 signed_report_downloads

Token download sementara.

```text
signed_report_downloads
- id UUID PK
- tenant_id UUID FK tenants.id
- report_id UUID FK reports.id
- report_export_id UUID FK report_exports.id
- user_id UUID FK users.id
- token_hash TEXT
- ip_address INET NULL
- user_agent TEXT NULL
- status VARCHAR
  -- active, used, expired, revoked
- expires_at TIMESTAMPTZ
- used_at TIMESTAMPTZ NULL
- created_at TIMESTAMPTZ
```

Aturan:

* Jangan expose file storage publik tanpa signed token.
* Setiap download report harus tercatat.
* Client tidak boleh download report campaign lain.

---

# 17. Alert & Maintenance Tables

## 17.1 alert_events

```text
alert_events
- id UUID PK
- tenant_id UUID FK tenants.id
- camera_id UUID FK cameras.id NULL
- billboard_id UUID FK billboards.id NULL
- campaign_id UUID FK campaigns.id NULL
- alert_type VARCHAR
  -- camera_offline, stream_down, missing_data, low_quality, report_failed, proof_missing, storage_warning
- severity VARCHAR
  -- critical, high, medium, low, info
- title VARCHAR
- message TEXT
- status VARCHAR
  -- open, acknowledged, resolved, ignored
- triggered_at TIMESTAMPTZ
- acknowledged_by UUID FK users.id NULL
- acknowledged_at TIMESTAMPTZ NULL
- resolved_by UUID FK users.id NULL
- resolved_at TIMESTAMPTZ NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

---

## 17.2 maintenance_tickets

```text
maintenance_tickets
- id UUID PK
- tenant_id UUID FK tenants.id
- camera_id UUID FK cameras.id NULL
- billboard_id UUID FK billboards.id NULL
- assigned_to UUID FK users.id NULL
- created_by UUID FK users.id
- title VARCHAR
- description TEXT
- priority VARCHAR
  -- critical, high, medium, low
- status VARCHAR
  -- open, assigned, in_progress, resolved, closed, cancelled
- due_at TIMESTAMPTZ NULL
- resolved_at TIMESTAMPTZ NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

---

## 17.3 maintenance_logs

```text
maintenance_logs
- id UUID PK
- tenant_id UUID FK tenants.id
- maintenance_ticket_id UUID FK maintenance_tickets.id NULL
- camera_id UUID FK cameras.id NULL
- billboard_id UUID FK billboards.id NULL
- technician_id UUID FK users.id NULL
- log_type VARCHAR
  -- inspection, repair, installation, replacement, note
- notes TEXT
- photo_url TEXT NULL
- gps_latitude NUMERIC(10,7) NULL
- gps_longitude NUMERIC(10,7) NULL
- logged_at TIMESTAMPTZ
- created_at TIMESTAMPTZ
```

---

# 18. Audit & System Tables

## 18.1 audit_logs

Semua aksi penting wajib masuk sini.

```text
audit_logs
- id UUID PK
- tenant_id UUID NULL
- user_id UUID FK users.id NULL
- actor_role VARCHAR NULL
- action VARCHAR
  -- create, update, delete, approve, reject, login, logout, export, stream_start, stream_end
- module VARCHAR
  -- tenant, user, billboard, camera, campaign, proof, report, live_stream, subscription, system
- entity_type VARCHAR
- entity_id UUID NULL
- before_json JSONB NULL
- after_json JSONB NULL
- ip_address INET NULL
- user_agent TEXT NULL
- request_id VARCHAR NULL
- trace_id VARCHAR NULL
- severity VARCHAR
  -- info, warning, critical
- created_at TIMESTAMPTZ
```

Index:

```sql
CREATE INDEX idx_audit_logs_tenant_time
ON audit_logs(tenant_id, created_at DESC);

CREATE INDEX idx_audit_logs_entity
ON audit_logs(entity_type, entity_id);
```

Catatan:

* Report download masuk audit.
* Live stream start/end masuk audit.
* Approval proof masuk audit.
* Role change masuk audit.
* Tenant suspend masuk audit.
* Owner Platform support access masuk audit.

---

## 18.2 system_settings

```text
system_settings
- id UUID PK
- setting_key VARCHAR UNIQUE
- setting_value JSONB
- description TEXT NULL
- updated_by UUID FK users.id NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

---

## 18.3 tenant_settings

```text
tenant_settings
- id UUID PK
- tenant_id UUID FK tenants.id
- setting_key VARCHAR
- setting_value JSONB
- updated_by UUID FK users.id NULL
- created_at TIMESTAMPTZ
- updated_at TIMESTAMPTZ
```

Unique:

```sql
CREATE UNIQUE INDEX uq_tenant_settings
ON tenant_settings(tenant_id, setting_key);
```

---

# 19. Formula Dasar yang Harus Didukung Database

## 19.1 Total Vehicle

```text
total_vehicle_count =
motorcycle_count + car_count + bus_count + truck_count
```

---

## 19.2 Average Daily Traffic

```text
average_daily_traffic =
total_vehicle_count / campaign_days
```

---

## 19.3 Estimated Exposure

Versi awal:

```text
estimated_exposure =
(motorcycle_count × motorcycle_multiplier)
+ (car_count × car_multiplier)
+ (bus_count × bus_multiplier)
+ (truck_count × truck_multiplier)
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

## 19.4 CPV

```text
cpv =
estimated_ooh_budget / total_vehicle_count
```

---

## 19.5 CPI

```text
cpi =
estimated_ooh_budget / estimated_exposure
```

---

## 19.6 Data Quality Score

Formula awal:

```text
data_quality_score =
(camera_uptime_score × 0.35)
+ (detection_confidence_score × 0.30)
+ (frame_quality_score × 0.20)
+ (missing_data_score × 0.15)
```

Catatan:

* Formula final harus disimpan di `formula_versions`.
* Report final harus menyimpan `formula_version_id`.
* Jangan hardcode formula hanya di frontend.

---

# 20. Index Strategy

## 20.1 Index Wajib Multi-Tenant

Tabel tenant besar wajib punya index pola:

```sql
CREATE INDEX idx_table_tenant_id ON table_name(tenant_id);
```

Untuk tabel time-series:

```sql
CREATE INDEX idx_table_tenant_time
ON table_name(tenant_id, created_at DESC);
```

---

## 20.2 Index Traffic Event

```sql
CREATE INDEX idx_traffic_events_camera_time
ON traffic_events(tenant_id, camera_id, captured_at DESC);

CREATE INDEX idx_traffic_events_campaign_time
ON traffic_events(tenant_id, campaign_id, captured_at DESC);

CREATE INDEX idx_traffic_events_billboard_time
ON traffic_events(tenant_id, billboard_id, captured_at DESC);
```

---

## 20.3 Index Summary

```sql
CREATE INDEX idx_hourly_summary_campaign_date
ON traffic_hourly_summaries(tenant_id, campaign_id, summary_date, summary_hour);

CREATE INDEX idx_daily_summary_campaign_date
ON traffic_daily_summaries(tenant_id, campaign_id, summary_date);

CREATE INDEX idx_monthly_summary_campaign_month
ON traffic_monthly_summaries(tenant_id, campaign_id, summary_month);
```

---

## 20.4 Index Report

```sql
CREATE INDEX idx_reports_tenant_campaign_period
ON reports(tenant_id, campaign_id, period_start, period_end);

CREATE INDEX idx_report_exports_report
ON report_exports(tenant_id, report_id, export_type);
```

---

## 20.5 Index Live Stream

```sql
CREATE INDEX idx_live_stream_sessions_user_status
ON live_stream_sessions(tenant_id, user_id, status);

CREATE INDEX idx_live_stream_sessions_camera_status
ON live_stream_sessions(tenant_id, camera_id, status);
```

---

# 21. Row Level Security Optional untuk Production

Jika memakai PostgreSQL RLS, contoh konsep:

```sql
ALTER TABLE billboards ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation_billboards
ON billboards
USING (tenant_id = current_setting('app.tenant_id')::uuid);
```

Catatan:

* RLS bagus, tetapi tidak menggantikan API guard.
* API tetap wajib memfilter `tenant_id`.
* Owner Platform SaaS support access harus punya mekanisme khusus dan audit log.

---

# 22. Data Retention Policy

## 22.1 Raw Traffic Events

Default:

```text
Starter: 30 hari
Growth: 60 hari
Pro: 180 hari
Enterprise: custom
```

## 22.2 Summary Data

Default:

```text
Hourly summary: 1 tahun
Daily summary: 3 tahun
Monthly summary: 5 tahun atau custom
```

## 22.3 Report Snapshot

Report final disimpan sesuai kontrak tenant.

Default:

```text
minimal 3 tahun
```

## 22.4 Audit Logs

Audit log penting minimal disimpan:

```text
2 tahun
```

Untuk enterprise bisa lebih panjang.

---

# 23. Data Status Standard

Gunakan status berikut untuk data traffic:

```text
raw
validated
estimated
excluded
```

Arti:

* `raw`: data baru masuk dari AI.
* `validated`: data lolos rule kualitas.
* `estimated`: data hasil estimasi karena missing/downtime.
* `excluded`: data dikeluarkan dari summary/report.

Report wajib menampilkan label jika ada data estimated.

---

# 24. Anti-Overclaim Database Rules

Database harus menyimpan konteks agar aplikasi tidak overclaim.

Setiap summary dan report wajib punya:

```text
formula_version_id
methodology_version_id
data_quality_score
avg_confidence_score
valid_hours
missing_hours
estimated_hours
anomaly_count
```

Gunakan istilah:

```text
estimated_exposure
potential_exposure
opportunity_to_see
```

Jangan gunakan istilah final di schema seperti:

```text
actual_impression
real_viewers
people_who_saw
guaranteed_views
```

Karena sistem membaca kendaraan dan peluang exposure, bukan membaca mata manusia.

---

# 25. Migration Order yang Disarankan

Urutan migration awal:

```text
001_create_platform_and_tenants
002_create_subscription_tables
003_create_users_roles_permissions
004_create_clients
005_create_billboards_and_faces
006_create_devices_and_cameras
007_create_campaigns
008_create_ai_rules_formula_methodology
009_create_traffic_events
010_create_traffic_summaries
011_create_data_quality_and_anomalies
012_create_proof_of_display
013_create_reports_jobs_exports_snapshots
014_create_live_stream_sessions
015_create_alerts_maintenance
016_create_audit_logs
017_create_settings
018_create_indexes
019_create_rls_optional
020_seed_default_roles_permissions
```

---

# 26. Seed Data Awal

Seed role:

```text
Owner Platform SaaS
Admin Tenant
Owner Tenant
Sales Tenant
Teknisi
Client
Viewer
```

Seed vehicle class:

```text
motorcycle
car
bus
truck
unknown
```

Seed report status:

```text
draft
generated
waiting_review
approved
sent_to_client
failed
archived
```

Seed alert severity:

```text
critical
high
medium
low
info
```

Seed methodology version:

```text
METHODOLOGY_V1
```

Seed formula version:

```text
FORMULA_V1
```

---

# 27. Agent / Developer Rules

Aturan keras untuk agent coding:

1. Jangan membuat tabel operasional tanpa `tenant_id`.
2. Jangan membuat query tenant tanpa filter `tenant_id`.
3. Jangan menyimpan raw RTSP plain text.
4. Jangan expose stream URL asli ke frontend.
5. Jangan membuat report final tanpa snapshot.
6. Jangan membuat report tanpa formula version.
7. Jangan membuat report tanpa methodology version.
8. Jangan membuat traffic summary tanpa data quality score.
9. Jangan membuat estimated exposure tanpa formula yang jelas.
10. Jangan membuat Owner Tenant punya permission create/edit/delete.
11. Jangan membuat Client bisa melihat campaign client lain.
12. Jangan menghapus audit log.
13. Jangan update traffic event raw tanpa audit/correction mechanism.
14. Jangan menaruh business logic rumus hanya di frontend.
15. Jangan menjadikan tabel hourly report sebagai satu-satunya sumber data mentah.
16. Jangan mengabaikan index untuk tabel traffic.
17. Jangan membuat export file publik tanpa signed download.
18. Jangan membuat live stream tanpa session expiry.
19. Jangan membuat permission hanya berdasarkan nama menu sidebar.
20. Jangan membuat dashboard cantik tetapi database-nya bocor tenant.

---

# 28. MVP Database Scope

Untuk MVP awal, tabel minimum yang wajib dibuat:

```text
tenants
tenant_profiles
subscription_plans
tier_feature_limits
tenant_subscriptions
users
roles
permissions
role_permissions
user_roles
user_sessions
clients
client_users
billboards
billboard_faces
edge_devices
cameras
camera_health_logs
live_stream_sessions
campaigns
campaign_billboards
ai_counting_rules
methodology_versions
formula_versions
occupancy_multipliers
traffic_events
traffic_hourly_summaries
traffic_daily_summaries
data_quality_snapshots
proof_of_display
report_generation_jobs
reports
report_location_snapshots
report_exports
signed_report_downloads
alert_events
maintenance_tickets
maintenance_logs
audit_logs
system_settings
tenant_settings
```

Tabel `traffic_monthly_summaries` boleh masuk MVP jika report bulanan langsung menjadi target utama.

---

# 29. Kesimpulan

Database platform ini harus dirancang sebagai sistem pembuktian data, bukan sekadar penyimpan angka.

Core yang wajib dikunci:

```text
tenant_id
role/permission
camera health
live stream session
traffic event store
hourly/daily/monthly summary
data quality score
confidence score
formula version
methodology version
report snapshot
signed export
audit log
```

Kalau database ini rapi, API akan kuat, dashboard akan aman, report akan bisa dipercaya, dan client akan melihat produk ini sebagai platform profesional.

Kalau database kacau, report akan terlihat cantik di PDF, tapi fondasinya seperti panggung dangdut di atas rawa: meriah, tapi ngeri kalau diinjak ramai-ramai.
