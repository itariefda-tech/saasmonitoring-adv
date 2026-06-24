# PROJECT_VISION_BUSINESS_RULES.md

# Project Vision & Business Rules — SaaS Billboard Monitoring & Report Platform

Dokumen ini adalah dokumen visi produk dan aturan bisnis utama untuk aplikasi **SaaS Billboard Monitoring & Report Platform**.

Dokumen ini bukan dokumen dekorasi. Ini adalah pagar beton agar produk tidak berubah arah, tidak salah klaim, tidak salah role, dan tidak melahirkan fitur liar yang terlihat keren tetapi merusak fondasi bisnis.

Jika ada konflik antara ide fitur baru dengan dokumen ini, maka fitur baru harus ditahan sampai business rule diperbarui secara sadar.

---

## 1. Posisi Dokumen

Dokumen ini mengunci:

- visi produk,
- positioning bisnis,
- target pengguna,
- batasan produk,
- model SaaS,
- paket/tier,
- aturan tenant,
- aturan client,
- aturan campaign,
- aturan billboard,
- aturan CCTV,
- aturan proof of display,
- aturan report,
- aturan bahasa klaim,
- aturan anti-overclaim,
- aturan MVP,
- aturan komersialisasi,
- aturan untuk developer/agent.

Dokumen ini harus sinkron dengan:

- `README.md`,
- `SYSTEM_ARCHITECTURE.md`,
- `DATABASE_DESIGN.md`,
- `AI_CCTV_ANALYTICS_RULES.md`,
- `API_CONTRACTS_AND_EVENT_PIPELINE.md`,
- `UI_UX_DASHBOARD_REPORTING_GUIDE.md`,
- `SECURITY_PRIVACY_ACCESS_CONTROL.md`,
- `ROADMAP.md`.

Jika dokumen teknis bicara “bagaimana membangun”, dokumen ini bicara “untuk apa dibangun dan batas bisnisnya apa”.

---

## 2. Visi Produk

Aplikasi ini dibuat untuk membantu perusahaan billboard / outdoor advertising menjual titik iklan dengan bukti data.

Sebelum aplikasi ini, titik billboard sering dijual dengan narasi:

- lokasi ramai,
- strategis,
- banyak kendaraan,
- dekat mall,
- dekat kantor,
- dekat pemukiman,
- cocok untuk awareness.

Masalahnya, narasi seperti itu sering tidak punya data operasional yang kuat.

Visi produk ini adalah mengubah billboard dari:

```text
Papan iklan statis di pinggir jalan
```

menjadi:

```text
Aset media OOH yang dapat dimonitor, diukur, dilaporkan, dan dipertanggungjawabkan dengan data.
```

Aplikasi ini harus membuat perusahaan billboard bisa berkata kepada client:

```text
Ini bukan cuma lokasi ramai.
Ini data traffic-nya.
Ini komposisi kendaraannya.
Ini jam puncaknya.
Ini estimasi exposure-nya.
Ini bukti tayangnya.
Ini report-nya.
Ini kualitas datanya.
```

Itu jualan yang punya tulang punggung. Bukan brosur manis yang kalau ditanya rumusnya langsung batuk-batuk.

---

## 3. Misi Produk

Misi produk:

1. Membantu perusahaan billboard membuktikan nilai titik iklan dengan data traffic.
2. Membantu client memahami performa campaign OOH secara lebih transparan.
3. Membuat report PDF/Excel/CSV yang rapi, client-friendly, dan audit-able.
4. Membuat dashboard monitoring yang bisa dipakai harian oleh tenant dan client.
5. Mengurangi klaim marketing kosong seperti “ramai banget” tanpa data pendukung.
6. Mengunci metodologi traffic counting, estimated exposure, data quality, dan report agar tidak menjadi angka sulap.
7. Membangun SaaS multi-tenant yang bisa dipakai banyak perusahaan billboard tanpa kebocoran data antar tenant.

---

## 4. Positioning Produk

### 4.1 Positioning Utama

```text
SaaS platform untuk perusahaan billboard yang mengubah CCTV traffic counting menjadi dashboard analytics, proof of display, dan report performa campaign OOH yang transparan dan siap dikirim ke client.
```

### 4.2 Kalimat Jual Singkat

```text
Bantu perusahaan billboard menjual lokasi iklan dengan bukti data: traffic kendaraan, estimated exposure, proof display, dan report campaign siap client.
```

### 4.3 Kalimat Jual Lebih Premium

```text
Platform monitoring billboard berbasis CCTV AI yang membantu perusahaan OOH mengukur traffic, memahami potensi exposure, memantau bukti tayang, dan menghasilkan report performa campaign yang lebih transparan, profesional, dan audit-able.
```

### 4.4 Brand Promise

Produk ini menjanjikan:

- data traffic yang lebih jelas,
- report yang lebih profesional,
- campaign yang lebih mudah dipertanggungjawabkan,
- dashboard yang mudah dibaca client,
- proof of display yang lebih rapi,
- tenant isolation yang aman,
- sistem SaaS yang bisa tumbuh.

Produk ini **tidak** menjanjikan:

- orang pasti melihat billboard,
- orang pasti membaca iklan,
- campaign pasti menaikkan sales,
- AI pasti 100% akurat,
- CCTV selalu sempurna dalam segala cuaca,
- data tanpa error,
- impression sebagai fakta biologis mata manusia.

---

## 5. Target Market

### 5.1 Primary Market

Target utama:

- perusahaan billboard,
- perusahaan outdoor advertising,
- media owner OOH,
- operator JPO advertising,
- operator billboard jalan raya,
- operator LED videotron outdoor,
- agency OOH yang mengelola inventory billboard.

### 5.2 Initial Market

Tahap awal fokus Indonesia.

Alasan:

- kebutuhan OOH lokal besar,
- traffic kendaraan padat,
- banyak billboard strategis belum punya data monitoring kuat,
- client brand makin butuh report dan pembuktian,
- regulasi, bahasa, dan format report bisa difokuskan dulu.

### 5.3 Future Market

Setelah produk matang:

- multi-city Indonesia,
- multi-tenant besar,
- enterprise OOH,
- integrasi agency,
- kemungkinan ekspansi regional.

Tetapi MVP jangan sok global dulu. Bayi baru belajar jalan jangan langsung didaftarkan marathon.

---

## 6. Core Value Proposition

Nilai utama produk:

| Nilai | Penjelasan |
|---|---|
| Data Proof | Titik billboard didukung data traffic kendaraan. |
| Client Trust | Client mendapat dashboard dan report yang lebih meyakinkan. |
| Campaign Transparency | Performa campaign dapat dilihat per periode dan per lokasi. |
| Operational Monitoring | Tenant bisa memantau CCTV, campaign, proof, dan report. |
| Report Automation | Report harian/mingguan/bulanan dapat dibuat lebih cepat. |
| Data Integrity | Data diberi confidence score, quality score, status, dan methodology. |
| SaaS Scalability | Satu platform bisa melayani banyak perusahaan billboard. |

---

## 7. Prinsip Bisnis Utama

Prinsip yang wajib dijaga:

1. Produk ini adalah SaaS, bukan aplikasi custom satu tenant.
2. Multi-tenant wajib sejak awal.
3. Tenant isolation adalah hukum besi.
4. Data client hanya boleh dilihat oleh client pemilik campaign.
5. Owner Tenant adalah monitoring-only, bukan admin operasional.
6. Admin Tenant adalah role operasional tertinggi dalam tenant.
7. Owner Platform SaaS mengatur platform, tenant, tier, dan subscription.
8. Dashboard client wajib mobile-first.
9. Report harus client-friendly, bukan dump tabel mentah.
10. Setiap angka harus punya sumber, rumus, status, dan validitas.
11. Estimated exposure harus diberi label sebagai estimasi.
12. Jangan pernah mengklaim sistem membaca mata manusia.
13. Raw RTSP tidak boleh diberikan ke client.
14. Proof of display harus melewati approval sebelum tampil ke client.
15. Report final harus punya snapshot dan versioning.
16. Semua aksi penting harus masuk audit log.

---

## 8. Definisi Produk

### 8.1 Produk Ini Adalah

Produk ini adalah:

- SaaS monitoring billboard,
- dashboard analytics traffic,
- report generator OOH,
- proof of display manager,
- CCTV/device monitoring,
- campaign monitoring,
- data quality monitoring,
- client dashboard,
- tenant dashboard,
- platform owner dashboard.

### 8.2 Produk Ini Bukan

Produk ini bukan:

- sistem facial recognition,
- sistem deteksi identitas pribadi,
- sistem demographic profiling manusia,
- sistem yang membaca mata manusia,
- sistem jaminan sales naik,
- marketplace billboard publik tahap awal,
- payment gateway billing kompleks tahap awal,
- aplikasi CCTV umum untuk semua kebutuhan,
- aplikasi keamanan rumah,
- aplikasi live streaming bebas tanpa audit,
- aplikasi analytics yang boleh membuat angka tanpa metodologi.

---

## 9. Stakeholder Utama

### 9.1 Owner Platform SaaS

Pemilik platform SaaS.

Tanggung jawab bisnis:

- mengelola tenant,
- mengelola plan/tier,
- mengelola subscription,
- mengontrol status tenant,
- melihat health platform,
- melihat usage platform,
- mengelola feature flags,
- melihat audit log platform,
- menentukan strategi harga.

Owner Platform SaaS bukan staf operasional tenant.

### 9.2 Admin Tenant

Admin utama perusahaan billboard.

Tanggung jawab bisnis:

- mengelola billboard,
- mengelola CCTV,
- mengelola client,
- mengelola campaign,
- mengelola proof,
- generate report,
- mengatur user tenant,
- menangani alert maintenance,
- memastikan data campaign siap untuk client.

Admin Tenant adalah “raja operasional” di tenant sendiri, tetapi tidak boleh melihat kerajaan tetangga.

### 9.3 Owner Tenant

Pemilik/pimpinan perusahaan billboard.

Tanggung jawab bisnis:

- melihat performa perusahaan,
- memantau campaign,
- memantau billboard,
- melihat ringkasan traffic,
- melihat status CCTV,
- melihat report,
- membaca insight.

Owner Tenant tidak butuh tombol create/edit/delete. Ia butuh dashboard yang jernih, bukan kokpit pesawat tempur.

### 9.4 Sales Tenant

User penjualan tenant.

Tanggung jawab bisnis:

- melihat performa titik billboard untuk bahan jualan,
- melihat campaign/client yang terkait dengannya,
- membuat draft campaign jika diizinkan,
- membantu menjelaskan report kepada client,
- menggunakan data untuk proposal.

### 9.5 Teknisi Lapangan

User operasional teknis.

Tanggung jawab bisnis:

- memasang kamera,
- memeriksa kamera,
- upload proof lapangan,
- mengisi checklist maintenance,
- melaporkan gangguan,
- memastikan perangkat lapangan sehat.

Teknisi tidak boleh membaca data bisnis tenant secara luas.

### 9.6 Client / Brand

Penyewa billboard.

Kebutuhan bisnis:

- melihat campaign miliknya,
- melihat KPI traffic,
- melihat proof approved,
- melihat report,
- download report jika diizinkan,
- memahami nilai titik billboard.

Client tidak boleh melihat:

- campaign client lain,
- data tenant penuh,
- billing tenant,
- raw RTSP,
- konfigurasi kamera,
- user internal tenant,
- report client lain.

---

## 10. Model SaaS

Produk ini menggunakan model multi-tenant SaaS.

Struktur bisnis:

```text
Owner Platform SaaS
└── Tenant / Perusahaan Billboard
    ├── Admin Tenant
    ├── Owner Tenant
    ├── Sales Tenant
    ├── Teknisi
    ├── Client / Brand
    ├── Billboard
    ├── Camera
    ├── Campaign
    ├── Proof of Display
    └── Report
```

Aturan utama:

- satu tenant = satu perusahaan billboard,
- tenant memiliki billboard sendiri,
- tenant memiliki client sendiri,
- tenant memiliki campaign sendiri,
- tenant memiliki user sendiri,
- data tenant tidak boleh bocor ke tenant lain,
- client hanya melihat campaign miliknya,
- Owner Platform SaaS melihat data global dan usage, bukan operasional tenant harian kecuali support/audit resmi.

---

## 11. Tier / Paket SaaS

Tier harus menjadi pembeda fitur, limit, dan nilai bisnis.

Contoh tier awal:

1. Starter
2. Growth
3. Pro
4. Enterprise
5. Custom Premium

### 11.1 Starter

Untuk tenant kecil atau pilot project.

Batasan contoh:

- billboard terbatas,
- kamera terbatas,
- user terbatas,
- client access terbatas,
- report basic,
- export PDF basic,
- data retention pendek,
- tanpa advanced anomaly,
- tanpa white label.

Cocok untuk:

- trial,
- proof of concept,
- tenant kecil,
- titik billboard awal.

### 11.2 Growth

Untuk tenant yang mulai aktif mengelola banyak campaign.

Fitur contoh:

- lebih banyak billboard,
- lebih banyak kamera,
- client dashboard,
- report PDF/Excel/CSV,
- proof display approval,
- basic data quality,
- basic alert,
- campaign report.

### 11.3 Pro

Untuk tenant serius yang membutuhkan report lebih kuat.

Fitur contoh:

- multi billboard,
- multi camera,
- advanced analytics,
- data quality score,
- confidence score,
- anomaly flag,
- report approval workflow,
- signed report export,
- longer data retention,
- insight/recommendation basic.

### 11.4 Enterprise

Untuk perusahaan billboard besar.

Fitur contoh:

- limit custom,
- multi-city,
- SLA support,
- white label report,
- advanced role policy,
- advanced audit,
- data retention panjang,
- custom branding,
- custom report template,
- API integration jika diperlukan.

### 11.5 Custom Premium

Untuk landmark, paket khusus, atau client besar.

Fitur contoh:

- multi-camera intersection,
- dedicated proof camera,
- advanced exposure analytics,
- custom dashboard,
- custom insight,
- premium report,
- special onboarding,
- dedicated support.

---

## 12. Paket Berdasarkan Tipe Lokasi Billboard

Selain tier SaaS, aplikasi juga mengenal paket lokasi billboard.

### 12.1 Paket Tol / Highway

Konteks:

- jalan cepat,
- umumnya satu arah,
- kendaraan dominan mobil/truk,
- exposure cepat,
- speed rate penting.

Kebutuhan:

- 1 CCTV utama,
- counting satu arah,
- vehicle classification,
- speed rate jika kamera terkalibrasi,
- peak hour,
- CPV/CPI,
- estimated exposure,
- monthly report.

### 12.2 Paket Jalan Biasa / JPO / Satu Muka

Konteks:

- jalan perkotaan,
- JPO,
- flyover,
- jalan padat,
- kendaraan dominan motor/mobil.

Kebutuhan:

- 1 CCTV utama,
- vehicle counting,
- classification,
- congestion level,
- dwell time,
- hourly heatmap,
- proof display,
- report harian/mingguan/bulanan.

### 12.3 Paket Persimpangan / Multi-Arah

Konteks:

- perempatan,
- pertigaan,
- perlimaan,
- billboard terlihat dari beberapa arah.

Kebutuhan:

- 2 sampai 4 CCTV,
- direction mapping,
- report per arah,
- anti double count,
- camera overlap policy,
- exposure zone per arah,
- dashboard premium.

### 12.4 Paket Premium Landmark

Konteks:

- billboard besar,
- titik mahal,
- lokasi ikonik,
- campaign besar.

Kebutuhan:

- multi camera,
- dedicated proof camera jika perlu,
- advanced report,
- data quality section,
- insight otomatis,
- custom layout report,
- client dashboard premium.

---

## 13. Business Rule Tenant

### 13.1 Tenant Creation

Tenant hanya dapat dibuat oleh Owner Platform SaaS.

Data minimal tenant:

- tenant name,
- legal/company name,
- contact person,
- email,
- phone,
- address,
- city,
- status,
- plan/tier,
- subscription start,
- subscription end,
- branding config jika ada.

### 13.2 Tenant Status

Status tenant:

- `trial`,
- `active`,
- `suspended`,
- `inactive`,
- `terminated`.

Aturan:

- tenant `trial` punya limit sesuai trial plan,
- tenant `active` bisa memakai fitur sesuai subscription,
- tenant `suspended` tidak boleh membuat campaign/report baru,
- tenant `inactive` tidak aktif operasional,
- tenant `terminated` hanya boleh dipertahankan untuk audit/arsip sesuai retention policy.

### 13.3 Tenant Suspension

Tenant boleh disuspend karena:

- subscription expired,
- pembayaran bermasalah,
- penyalahgunaan sistem,
- pelanggaran keamanan,
- permintaan bisnis resmi.

Saat suspended:

- user tenant tidak boleh membuat data baru,
- client dashboard bisa dibatasi,
- report download bisa dibatasi sesuai kebijakan,
- data tetap tidak boleh dihapus sembarangan.

### 13.4 Tenant Isolation

Semua data operasional tenant wajib memiliki `tenant_id`.

Tabel yang wajib tenant-scoped:

- users tenant,
- clients,
- billboards,
- billboard_faces,
- cameras,
- campaigns,
- campaign_billboards,
- proof_of_display,
- traffic_events,
- summaries,
- reports,
- exports,
- alerts,
- maintenance,
- audit logs tenant.

Larangan mutlak:

- API mengembalikan data tenant lain,
- report mencampur data tenant lain,
- client melihat campaign client lain,
- object storage path tanpa tenant scope,
- queue worker memproses job tanpa tenant_id.

---

## 14. Business Rule User & Role

### 14.1 Role Resmi

Role resmi:

- Owner Platform SaaS,
- Admin Tenant,
- Owner Tenant,
- Sales Tenant,
- Teknisi,
- Client / Brand,
- Viewer / Report Viewer jika dibutuhkan.

Tidak boleh membuat role baru tanpa update:

- permission matrix,
- API guard,
- UI sidebar,
- database seed,
- audit event,
- documentation.

### 14.2 Owner Platform SaaS

Boleh:

- kelola tenant,
- kelola subscription,
- kelola plan/tier,
- lihat usage platform,
- lihat system health,
- lihat audit platform,
- suspend tenant,
- manage feature flags.

Tidak boleh diperlakukan sebagai:

- Admin Tenant biasa,
- user client,
- user sales,
- teknisi tenant.

### 14.3 Admin Tenant

Boleh dalam tenant sendiri:

- CRUD billboard,
- CRUD camera,
- CRUD client,
- CRUD campaign,
- manage users tenant,
- approve proof,
- generate report,
- download report,
- view analytics,
- view alerts,
- manage maintenance.

Tidak boleh:

- melihat tenant lain,
- mengubah tenant lain,
- mengatur tier SaaS global,
- mengakses billing platform global,
- bypass report approval jika policy mewajibkan approval.

### 14.4 Owner Tenant

Boleh:

- melihat executive dashboard,
- melihat KPI,
- melihat report,
- melihat summary traffic,
- melihat status campaign,
- melihat status CCTV,
- download report jika permission diberikan.

Tidak boleh:

- create,
- edit,
- delete,
- approve,
- manage user,
- manage role,
- manage camera,
- change subscription,
- upload proof,
- ubah campaign.

Owner Tenant bukan Admin Tenant versi jas rapi. Ia hanya monitoring.

### 14.5 Sales Tenant

Boleh:

- melihat inventory billboard yang diizinkan,
- melihat ringkasan traffic untuk proposal,
- melihat client/campaign yang terkait,
- membuat draft campaign jika permission ada,
- melihat report campaign terkait.

Tidak boleh:

- approve proof,
- mengatur camera,
- mengatur tenant,
- menghapus campaign final,
- mengakses semua client tanpa izin.

### 14.6 Teknisi

Boleh:

- melihat assigned camera,
- melihat device health yang ditugaskan,
- upload foto lapangan,
- upload proof lapangan,
- isi checklist,
- buat incident report,
- update maintenance status.

Tidak boleh:

- melihat billing,
- melihat semua client,
- melihat semua report bisnis,
- approve proof final,
- generate report client,
- mengubah campaign.

### 14.7 Client / Brand

Boleh:

- melihat campaign miliknya,
- melihat KPI campaign,
- melihat proof approved,
- melihat report campaign miliknya,
- download report jika diizinkan,
- melihat live view jika fitur dan permission aktif.

Tidak boleh:

- melihat campaign lain,
- melihat client lain,
- melihat raw RTSP,
- melihat konfigurasi kamera,
- melihat user tenant,
- mengakses report tenant penuh,
- melihat audit internal tenant.

---

## 15. Business Rule Client

### 15.1 Client Ownership

Client selalu dimiliki oleh tenant.

```text
client.tenant_id wajib ada
```

Client tidak bisa menjadi milik dua tenant sekaligus kecuali nanti ada desain enterprise khusus. Untuk MVP, satu client record = satu tenant.

### 15.2 Client User Access

Client user harus terikat ke:

- tenant_id,
- client_id,
- campaign_id atau daftar campaign yang diizinkan.

Client user tidak boleh hanya diberi tenant_id tanpa campaign scope.

### 15.3 Client Dashboard

Client dashboard harus menampilkan:

- campaign status,
- campaign period,
- location/billboard summary,
- total vehicle,
- vehicle composition,
- estimated exposure,
- peak hour,
- proof display approved,
- report download,
- data quality warning jika ada.

Client dashboard harus mobile-first.

### 15.4 Client Report Access

Client hanya boleh mengakses report:

- milik campaign-nya,
- status approved/final/sent,
- melalui signed URL,
- dengan audit log download.

Draft report tidak boleh tampil ke client kecuali role internal mengizinkan.

---

## 16. Business Rule Billboard

### 16.1 Billboard Master

Data minimum billboard:

- tenant_id,
- billboard_id,
- code,
- name,
- address,
- city,
- province,
- latitude,
- longitude,
- road_type,
- size,
- orientation,
- visibility_angle,
- monthly_rate / estimated OOH budget,
- status,
- photo location.

### 16.2 Billboard Face

Satu billboard bisa punya beberapa face/sisi.

Data minimum face:

- face_id,
- billboard_id,
- face_name,
- direction,
- orientation,
- size,
- visibility rule,
- status.

### 16.3 Billboard Status

Status billboard:

- `available`,
- `booked`,
- `active_campaign`,
- `maintenance`,
- `inactive`.

### 16.4 Billboard Visibility Rule

Setiap billboard/face harus punya aturan visibility:

- arah hadap,
- arah traffic yang relevan,
- exposure zone,
- road segment,
- catatan blind spot jika ada.

Tanpa aturan visibility, estimated exposure tidak boleh tampil sebagai final. Maksimal tampil sebagai simulation/draft.

---

## 17. Business Rule Camera / CCTV

### 17.1 Camera Ownership

Satu kamera wajib terkait ke:

- tenant_id,
- billboard_id,
- billboard_face_id jika relevan,
- device_id,
- camera rule.

Kamera tidak boleh dipakai untuk billboard milik tenant lain.

### 17.2 Camera Purpose

Tujuan kamera:

- traffic counting,
- vehicle classification,
- direction detection,
- traffic condition monitoring,
- proof snapshot jika memungkinkan,
- dedicated proof display jika paket premium.

### 17.3 Camera Main Direction

Pada mayoritas kasus, kamera utama menghadap jalan, bukan menghadap billboard.

Konsekuensi:

- traffic data boleh dihitung dari kamera utama,
- proof display tidak boleh mengandalkan kamera jalan jika billboard tidak terlihat jelas,
- proof display bisa memakai foto lapangan teknisi,
- paket premium bisa menambah kamera khusus proof.

### 17.4 Live View

Live view adalah fitur terbatas.

Aturan:

- tidak boleh expose raw RTSP,
- harus memakai secure stream proxy,
- harus memakai signed session/token,
- harus punya expiry,
- harus masuk audit log,
- boleh dibatasi satu sesi aktif per user/camera,
- client hanya boleh live view jika tenant mengizinkan dan campaign terkait.

### 17.5 Camera Health

Camera health harus memantau:

- online/offline,
- last heartbeat,
- stream quality,
- latency,
- FPS,
- resolution,
- uptime,
- missing data,
- error count,
- maintenance status.

---

## 18. Business Rule Campaign

### 18.1 Campaign Definition

Campaign adalah kontrak/aktivitas iklan client pada satu atau beberapa billboard dalam periode tertentu.

Data minimum:

- tenant_id,
- client_id,
- campaign_name,
- brand,
- objective,
- start_date,
- end_date,
- billboard/face list,
- creative/material,
- estimated OOH budget,
- status,
- dashboard access policy,
- report access policy.

### 18.2 Campaign Status

Status campaign:

- `draft`,
- `scheduled`,
- `active`,
- `paused`,
- `completed`,
- `cancelled`,
- `archived`.

### 18.3 Draft Campaign

Draft campaign:

- boleh dibuat Admin Tenant,
- boleh dibuat Sales jika permission ada,
- tidak tampil ke client,
- tidak boleh generate final report,
- boleh memakai simulation data.

### 18.4 Scheduled Campaign

Scheduled campaign:

- sudah punya periode,
- sudah punya billboard,
- client bisa diberi akses terbatas,
- proof belum wajib tersedia,
- traffic belum dihitung sebagai campaign aktif sebelum start date.

### 18.5 Active Campaign

Active campaign:

- periode sudah berjalan,
- traffic data mulai dihitung,
- proof display harus dipantau,
- report harian bisa digenerate,
- client dashboard aktif sesuai permission.

### 18.6 Completed Campaign

Completed campaign:

- periode sudah selesai,
- report final bisa dibuat,
- data harus disnapshot,
- proof final harus dilengkapi,
- client dapat mengakses final report.

### 18.7 Campaign Pause

Jika campaign pause:

- traffic tetap bisa tercatat sebagai lokasi,
- campaign KPI harus menandai pause period,
- report harus menjelaskan periode pause,
- exposure campaign tidak boleh dihitung untuk periode yang tidak valid jika policy menyatakan pause tidak dihitung.

### 18.8 Campaign Multi-Location

Campaign bisa berjalan di banyak billboard.

Aturan:

- report harus bisa per lokasi,
- report harus bisa total campaign,
- cross-location comparison harus jelas,
- data quality per lokasi tidak boleh disamaratakan,
- lokasi low quality harus diberi warning.

---

## 19. Business Rule Traffic Analytics

### 19.1 Traffic Data adalah Evidence Pendukung

Traffic data membantu menjelaskan potensi performa lokasi.

Traffic data bukan bukti bahwa manusia pasti melihat atau membaca iklan.

### 19.2 Vehicle Classes Minimum

Kelas kendaraan minimum:

- motorcycle,
- car,
- bus,
- truck.

Aturan pickup/van mengikuti `AI_CCTV_ANALYTICS_RULES.md`.

### 19.3 Data Status

Setiap data traffic harus punya status:

- `raw`,
- `validated`,
- `estimated`,
- `adjusted`,
- `rejected`,
- `simulation`.

Dashboard dan report harus menampilkan status data saat relevan.

### 19.4 Traffic Metrics

Metric minimum:

- total vehicle,
- vehicle composition,
- average daily traffic,
- peak hour,
- peak day,
- estimated exposure,
- CPV,
- CPI,
- data quality score,
- confidence score,
- anomaly count,
- valid hours,
- missing hours.

### 19.5 Raw vs Validated vs Estimated

Aturan tampilan:

- raw data untuk internal/debug,
- validated data untuk dashboard/report utama,
- estimated data boleh tampil dengan label,
- rejected data tidak boleh masuk report final,
- simulation data tidak boleh menyamar sebagai data real.

---

## 20. Business Rule Estimated Exposure

### 20.1 Definisi

Estimated exposure adalah estimasi potensi iklan terlihat berdasarkan traffic kendaraan, occupancy multiplier, visibility factor, exposure zone, dwell factor, dan data quality adjustment.

Estimated exposure bukan jumlah manusia yang pasti melihat iklan.

### 20.2 Istilah yang Diizinkan

Gunakan:

- estimated exposure,
- potential exposure,
- opportunity to see,
- potensi iklan terlihat,
- estimasi peluang paparan.

### 20.3 Istilah yang Dilarang

Jangan gunakan:

- pasti dilihat,
- pasti dibaca,
- pasti menjual,
- confirmed viewers,
- actual eyes,
- guaranteed audience,
- human attention detected,
- 100% impression valid.

### 20.4 Formula Transparency

Setiap report yang menampilkan estimated exposure wajib punya:

- formula version,
- methodology version,
- multiplier source,
- visibility factor,
- data quality policy,
- estimated data policy,
- disclaimer.

Jika formula belum matang, tampilkan sebagai beta/simulation/internal. Jangan dipoles jadi angka sakti.

---

## 21. Business Rule CPV dan CPI

### 21.1 CPV

```text
CPV = estimated_ooh_budget / total_vehicle
```

CPV berarti cost per vehicle detected/counted.

CPV bukan biaya per manusia yang melihat iklan.

### 21.2 CPI

```text
CPI = estimated_ooh_budget / estimated_exposure
```

CPI berarti cost per estimated exposure.

CPI wajib diberi konteks formula exposure.

### 21.3 Budget Source

`estimated_ooh_budget` harus berasal dari:

- monthly rate billboard,
- campaign budget,
- input manual Admin Tenant,
- imported contract value,
- atau konfigurasi pricing tenant.

Budget source harus jelas.

### 21.4 Cost Efficiency Warning

Jangan menampilkan CPI/CPV jika:

- budget kosong,
- total vehicle nol,
- estimated exposure nol,
- data quality critical,
- report masih simulation tanpa label.

---

## 22. Business Rule Data Quality

### 22.1 Data Quality Wajib Ada

Setiap traffic summary minimal harus punya:

- data_quality_score,
- average_confidence,
- valid_hours,
- missing_hours,
- uptime,
- anomaly_count.

### 22.2 Low Quality Warning

Jika data quality rendah:

- dashboard harus menampilkan warning,
- report harus punya catatan data integrity,
- estimated exposure harus ditandai hati-hati,
- insight otomatis harus mengurangi tingkat kepastian.

### 22.3 Critical Data

Jika data quality critical:

- jangan tampilkan sebagai final tanpa review,
- jangan gunakan untuk klaim utama,
- report perlu approval manual,
- periode terdampak harus dijelaskan.

---

## 23. Business Rule Proof of Display

### 23.1 Definisi

Proof of Display adalah bukti bahwa materi billboard/campaign sedang atau sudah tayang di lokasi.

Proof bisa berupa:

- foto pemasangan,
- foto berkala,
- timestamp,
- GPS location,
- nama teknisi,
- checklist campaign,
- catatan maintenance,
- approval admin,
- dedicated proof camera snapshot jika tersedia.

### 23.2 Proof Status

Status proof:

- `uploaded`,
- `waiting_review`,
- `approved`,
- `rejected`,
- `archived`.

### 23.3 Proof Approval

Aturan:

- Teknisi boleh upload proof,
- Admin Tenant approve/reject,
- Owner Tenant melihat ringkasan,
- Client hanya melihat proof approved,
- proof rejected tidak tampil ke client.

### 23.4 Proof Requirement

Campaign aktif minimal harus punya:

- proof awal pemasangan,
- proof selama campaign jika policy tenant mewajibkan,
- proof akhir jika dibutuhkan report final.

Jika campaign belum punya proof, dashboard Admin Tenant harus menampilkan warning.

### 23.5 Proof Anti-Manipulation

Proof harus menyimpan:

- uploaded_by,
- uploaded_at,
- GPS metadata jika ada,
- timestamp,
- file hash jika memungkinkan,
- approval_by,
- approval_at,
- rejection reason jika ditolak.

---

## 24. Business Rule Report

### 24.1 Jenis Report

Report utama:

- daily report,
- weekly report,
- monthly report,
- campaign report,
- billboard report,
- client report.

### 24.2 Format Report

Format:

- PDF untuk executive/client report,
- Excel untuk data detail,
- CSV untuk raw/hourly export,
- JSON/API untuk integrasi tahap lanjut.

### 24.3 Report Status

Status report:

- `draft`,
- `generated`,
- `waiting_review`,
- `approved`,
- `sent_to_client`,
- `failed`,
- `archived`.

### 24.4 Report Final Harus Snapshot

Report final tidak boleh bergantung pada query live yang berubah-ubah.

Report final wajib menyimpan:

- snapshot data,
- report version,
- generated_at,
- generated_by,
- approved_at,
- approved_by,
- formula_version,
- methodology_version,
- data_quality_summary,
- export file reference.

### 24.5 Report Content Minimum

Report PDF minimum:

1. Cover,
2. Executive Summary,
3. Campaign Overview,
4. Measurement Methodology,
5. Network Map,
6. Overall KPI Recap,
7. Location Summary,
8. Traffic Analysis,
9. Vehicle Composition,
10. Cost Efficiency,
11. Data Quality & Integrity,
12. Proof of Display,
13. Recommendation,
14. Raw Data Appendix,
15. Disclaimer & Formula Notes,
16. Closing Page.

### 24.6 Report Approval

Report client-facing harus melalui approval jika:

- data quality rendah,
- ada estimated data,
- ada anomaly besar,
- ada proof pending,
- report final bulanan,
- tenant policy mewajibkan approval.

### 24.7 Report Download

Download report harus:

- authenticated,
- authorized,
- tenant-scoped,
- campaign-scoped untuk client,
- memakai signed URL,
- memiliki expiry,
- dicatat audit log.

---

## 25. Business Rule Auto Insight & Recommendation

### 25.1 Auto Insight

Auto insight boleh menjelaskan:

- traffic naik/turun,
- peak hour,
- peak day,
- komposisi kendaraan dominan,
- lokasi traffic tertinggi,
- CPI/CPV terbaik,
- anomaly yang terdeteksi,
- data quality issue,
- weekday vs weekend pattern.

### 25.2 Auto Insight Tidak Boleh Overclaim

Auto insight tidak boleh berkata:

- traffic naik karena iklan berhasil,
- sales naik karena billboard,
- orang pasti melihat,
- campaign pasti efektif,
- penyebab anomaly pasti X tanpa data pendukung.

Gunakan bahasa:

- “terindikasi”,
- “kemungkinan”,
- “berdasarkan pola data”,
- “perlu validasi tambahan”,
- “dapat dipertimbangkan”.

### 25.3 Recommendation

Recommendation harus membantu keputusan bisnis, misalnya:

- lokasi dengan traffic tertinggi cocok untuk reach,
- lokasi dengan car composition tinggi cocok untuk otomotif/premium mobility,
- lokasi dengan motorcycle dominance tinggi cocok untuk mass-market awareness,
- lokasi dengan CPI lebih rendah lebih efisien untuk exposure estimation,
- lokasi low quality perlu maintenance sebelum dipakai report final.

Recommendation bukan ramalan dukun dashboard. Harus ada data pendukung.

---

## 26. Business Rule Dashboard

### 26.1 Dashboard Admin Tenant

Fokus:

- operasional,
- monitoring campaign,
- alert,
- proof approval,
- report management,
- device health.

Admin Tenant boleh punya tombol aksi sesuai permission.

### 26.2 Dashboard Owner Tenant

Fokus:

- executive summary,
- KPI bisnis,
- report,
- insight,
- trend,
- status CCTV ringkas.

Owner Tenant tidak boleh punya tombol operasional berbahaya.

### 26.3 Dashboard Client

Fokus:

- campaign miliknya,
- KPI mudah dibaca,
- proof approved,
- report download,
- mobile-first.

Client dashboard harus sederhana. Jangan lempar client ke lab teknisi penuh grafik silang-sengkarut.

### 26.4 Dashboard Owner Platform SaaS

Fokus:

- tenant count,
- tenant status,
- subscription,
- usage,
- system health,
- global device status,
- audit,
- feature flags.

Owner Platform SaaS tidak fokus membaca detail campaign client tenant kecuali support/audit resmi.

---

## 27. Business Rule Subscription & Billing

### 27.1 Subscription Unit

Subscription dihitung per tenant.

Limit bisa berdasarkan:

- jumlah billboard,
- jumlah camera,
- jumlah user,
- jumlah client access,
- data retention,
- report exports,
- advanced analytics,
- white label,
- support level.

### 27.2 Feature Flag per Tier

Fitur harus bisa dikontrol per tier:

- live view,
- PDF report,
- Excel export,
- CSV export,
- data quality score,
- anomaly detection,
- advanced insight,
- white label report,
- API integration,
- data retention.

### 27.3 Over Limit Policy

Jika tenant melewati limit:

- sistem harus mencegah create baru,
- atau meminta upgrade,
- atau menandai over-limit,
- tidak boleh diam-diam membiarkan sampai sistem kacau.

### 27.4 Billing MVP

Untuk MVP, billing payment gateway kompleks boleh ditunda.

Yang wajib ada:

- plan/tier,
- tenant subscription status,
- subscription start/end,
- manual payment status,
- suspend/activate tenant,
- feature limit enforcement.

---

## 28. Business Rule Notification & Alert

Alert utama:

- CCTV offline,
- stream putus,
- data traffic kosong,
- missing hours tinggi,
- data quality rendah,
- anomaly traffic,
- proof belum ada,
- proof pending approval,
- report gagal dibuat,
- storage penuh,
- subscription hampir habis,
- tenant suspended.

Severity:

- Critical,
- High,
- Medium,
- Low,
- Info.

Aturan:

- Admin Tenant menerima alert operasional,
- Teknisi menerima alert device/maintenance,
- Owner Tenant melihat ringkasan,
- Owner Platform SaaS melihat alert global,
- Client hanya melihat alert yang relevan untuk campaign jika perlu.

---

## 29. Business Rule Security & Privacy

Aturan utama:

- jangan simpan password plain text,
- jangan expose raw RTSP,
- jangan tampilkan credential di dokumentasi,
- jangan simpan secret di frontend,
- jangan izinkan live view tanpa audit,
- jangan izinkan export tanpa signed URL,
- jangan izinkan Client melihat data client lain,
- jangan izinkan Owner Tenant melakukan operasi admin,
- jangan izinkan Admin Tenant melihat tenant lain.

Privacy:

- tidak ada facial recognition,
- tidak ada identifikasi orang,
- tidak ada profiling atribut sensitif,
- fokus hanya traffic kendaraan dan bukti tayang billboard.

---

## 30. Business Rule Legal & Disclaimer

Setiap report dan dashboard yang menampilkan exposure harus punya disclaimer.

Contoh disclaimer:

```text
Traffic and estimated exposure data are generated from CCTV/Computer Vision analytics based on detected vehicle movement, configured counting lines, vehicle classification, and methodology settings. Estimated exposure represents potential opportunity to see, not guaranteed human attention, confirmed viewership, reading behavior, or sales impact.
```

Versi Indonesia:

```text
Data traffic dan estimated exposure dihasilkan dari analisis CCTV/Computer Vision berdasarkan kendaraan yang terdeteksi, counting line, klasifikasi kendaraan, dan metodologi yang dikonfigurasi. Estimated exposure adalah estimasi potensi paparan iklan, bukan jaminan bahwa manusia pasti melihat, membaca, atau membeli produk dari iklan tersebut.
```

Disclaimer wajib muncul di:

- methodology page report,
- formula notes,
- report footer jika diperlukan,
- dashboard tooltip untuk estimated exposure.

---

## 31. MVP Business Scope

### 31.1 MVP Wajib

MVP wajib mencakup:

- login,
- tenant management,
- role dasar,
- tenant isolation,
- Admin Tenant,
- Owner Tenant read-only,
- Client campaign isolation,
- billboard management,
- camera management,
- campaign management,
- proof upload & approval,
- traffic analytics basic,
- hourly/daily summary,
- PDF report basic,
- Excel/CSV export,
- report history,
- audit log aksi penting,
- secure live view basic,
- data quality basic,
- estimated exposure dengan label.

### 31.2 MVP Tidak Wajib

Boleh ditunda:

- payment gateway otomatis,
- mobile app native,
- marketplace billboard publik,
- AI recommendation canggih,
- dynamic pricing otomatis,
- multi-country,
- facial/demographic analytics,
- enterprise API kompleks,
- custom white label penuh.

### 31.3 MVP Success Criteria

MVP dianggap berhasil jika:

- tenant bisa dibuat,
- role tidak bocor,
- billboard dan campaign bisa dikelola,
- CCTV/device bisa dicatat dan dipantau,
- traffic summary bisa tampil,
- client hanya melihat campaign miliknya,
- Owner Tenant hanya read-only,
- report PDF bisa digenerate,
- export bisa diaudit,
- estimated exposure punya disclaimer,
- raw RTSP tidak bocor,
- angka utama punya formula/status.

---

## 32. Business Metrics untuk Produk SaaS

Metric untuk Owner Platform SaaS:

- total tenant,
- tenant active,
- tenant trial,
- tenant suspended,
- total billboard,
- total camera,
- active camera,
- offline camera,
- total campaign,
- active campaign,
- report generated,
- report downloaded,
- storage usage,
- AI processing usage,
- plan/tier distribution,
- subscription status,
- MRR/ARR jika billing sudah matang.

Metric untuk Tenant:

- total billboard,
- billboard active,
- active campaign,
- client active,
- camera online/offline,
- proof pending,
- report pending review,
- total vehicle,
- estimated exposure,
- top billboard,
- campaign ending soon.

---

## 33. Commercial Packaging Notes

Agar produk bisa dijual, paket harus mudah dipahami.

Contoh kalimat paket:

### Starter

```text
Untuk perusahaan billboard yang ingin mulai memberikan dashboard traffic dan report basic kepada client.
```

### Growth

```text
Untuk tenant yang mulai mengelola banyak campaign dan membutuhkan export report serta dashboard client.
```

### Pro

```text
Untuk perusahaan OOH yang ingin report lebih transparan dengan data quality, confidence score, anomaly flag, dan approval workflow.
```

### Enterprise

```text
Untuk perusahaan billboard besar yang membutuhkan limit custom, white label, advanced audit, dan support khusus.
```

---

## 34. Rules untuk Agent / Developer

Aturan keras:

1. Jangan membuat fitur tanpa mengerti role bisnisnya.
2. Jangan membuat role baru tanpa update permission matrix.
3. Jangan membuat dashboard Owner Tenant dengan tombol create/edit/delete.
4. Jangan membuat Client bisa melihat campaign client lain.
5. Jangan membuat Admin Tenant bisa melihat tenant lain.
6. Jangan membuat report tanpa methodology dan formula version.
7. Jangan membuat estimated exposure tanpa disclaimer.
8. Jangan membuat CPI tanpa formula exposure yang jelas.
9. Jangan menampilkan raw RTSP ke frontend client.
10. Jangan membuat proof tampil ke client sebelum approved.
11. Jangan membuat data estimated tampil seperti validated.
12. Jangan membuat simulation data tampil sebagai real data.
13. Jangan membuat dashboard cantik tetapi tenant guard bolong.
14. Jangan membuat tabel hourly besar sebagai halaman utama client.
15. Jangan membuat angka traffic dummy tanpa label dummy/simulation.
16. Jangan menghapus audit log requirement.
17. Jangan mengabaikan data quality score.
18. Jangan mengabaikan confidence score.
19. Jangan membuat claim sales impact otomatis.
20. Jangan membangun fitur mahal sebelum MVP stabil.

Aturan positif:

1. Mulai dari workflow tenant → billboard → camera → campaign → proof → analytics → report.
2. Buat dashboard client mobile-first.
3. Buat admin dashboard operasional, bukan hiasan.
4. Buat report PDF yang client-friendly.
5. Simpan raw data dan summary secara terstruktur.
6. Pisahkan raw, validated, estimated, rejected, dan simulation data.
7. Gunakan signed URL untuk export.
8. Gunakan audit log untuk aksi penting.
9. Tampilkan data quality warning secara jujur.
10. Pastikan tenant isolation di API, DB, worker, report, dan storage.

---

## 35. Acceptance Criteria Business Rules

Dokumen bisnis dianggap dipatuhi jika:

- produk tetap fokus pada SaaS billboard monitoring & report,
- semua role utama sesuai definisi,
- Owner Platform SaaS tidak dicampur dengan Admin Tenant,
- Owner Tenant tetap monitoring-only,
- Client hanya melihat campaign miliknya,
- tenant isolation ditegakkan,
- tier/plan bisa membatasi fitur dan limit,
- campaign punya status lifecycle jelas,
- proof punya approval workflow,
- report punya status lifecycle jelas,
- report final memakai snapshot,
- estimated exposure tidak overclaim,
- CPV/CPI punya formula jelas,
- data quality dan confidence tidak dihapus,
- raw RTSP tidak bocor,
- audit log tersedia untuk aksi penting,
- MVP tidak melebar ke fitur non-core.

---

## 36. Final Product Direction

Arah final produk:

```text
Bukan sekadar dashboard CCTV.
Bukan sekadar tabel kendaraan.
Bukan sekadar report PDF.

Produk ini adalah mesin pembuktian nilai media OOH:
traffic analytics + proof display + data integrity + client report + SaaS tenant management.
```

Produk harus terasa:

- profesional,
- transparan,
- aman,
- mudah dijual,
- mudah dipahami client,
- kuat secara teknis,
- tidak lebay dalam klaim.

Kalau dashboard adalah wajah, maka business rules adalah tulang belakang. Wajah boleh glow up, tapi tulang belakang jangan bengkok.

---

## 37. What This Document Enables

Dengan dokumen ini, Codex/developer harus bisa memahami bahwa implementasi MVP harus mengikuti urutan bisnis:

```text
Platform Owner creates tenant
↓
Tenant Admin manages billboard inventory
↓
Tenant Admin registers camera/device
↓
Tenant Admin creates client
↓
Tenant Admin creates campaign
↓
Technician uploads proof
↓
Admin approves proof
↓
AI/ingestion produces traffic data
↓
Aggregator creates summaries
↓
Dashboard displays validated/estimated metrics
↓
Report worker creates report snapshot
↓
Admin reviews/approves report
↓
Client accesses campaign dashboard/report only
```

Jika implementasi keluar dari alur ini tanpa alasan kuat, berarti agent sedang jalan-jalan ke hutan fitur. Tarik balik ke jalan raya.

---

## 38. Kesimpulan

`PROJECT_VISION_BUSINESS_RULES.md` adalah dokumen pengunci arah produk.

Intinya:

- produk ini untuk perusahaan billboard,
- bentuknya SaaS multi-tenant,
- output utamanya dashboard, proof, analytics, dan report,
- datanya harus transparan,
- role-nya harus tegas,
- report-nya harus bisa dipertanggungjawabkan,
- estimated exposure tidak boleh overclaim,
- Client hanya melihat campaign miliknya,
- Owner Tenant hanya monitoring,
- Admin Tenant mengelola operasional tenant,
- Owner Platform SaaS mengelola platform.

Produk yang bagus bukan yang paling banyak fiturnya. Produk yang bagus adalah yang tahu siapa pengguna utamanya, apa nilai bisnisnya, apa batas klaimnya, dan bagaimana menjaga kepercayaan client.

SaaS ini harus dibangun seperti jembatan: indah boleh, tapi pertama-tama harus kuat menahan beban.
