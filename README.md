# SaaS Billboard Monitoring & Report Platform

## 1. Gambaran Umum Produk

Aplikasi ini adalah platform **SaaS multi-tenant** untuk perusahaan billboard / outdoor advertising 
yang ingin memberikan dashboard monitoring dan laporan performa iklan kepada client berbasis data.

Sistem menggunakan CCTV / Computer Vision Camera yang dipasang di area billboard untuk membaca kondisi lalu lintas, 
menghitung kendaraan, mengklasifikasi jenis kendaraan, menghitung estimasi potensi exposure, menyediakan bukti billboard tayang, dan menghasilkan report harian, mingguan, serta bulanan.

Produk ini bukan aplikasi CCTV biasa.

Produk ini adalah gabungan dari:

* SaaS platform,
* billboard inventory system,
* campaign management,
* CCTV / CVC monitoring,
* AI traffic analytics,
* proof of display,
* client dashboard,
* report generator,
* data quality control,
* formula transparency,
* mobile-first client experience.

Arah produk wajib tetap:

**Billboard + CCTV / CVC + AI Traffic Analytics + Proof Display + Client Dashboard + Report Automation + SaaS Multi-Tenant Platform.**

---

## 2. Tujuan Utama Aplikasi

Tujuan utama aplikasi ini adalah membantu perusahaan billboard menjual titik iklan dengan data, bukan hanya klaim lokasi ramai.

Billboard tidak lagi hanya dijual dengan kalimat:

> Lokasi strategis, ramai, banyak dilihat orang.

Tetapi harus bisa dibuktikan dengan data seperti:

* jumlah kendaraan yang lewat,
* jenis kendaraan yang lewat,
* jam paling ramai,
* jam paling macet,
* rata-rata kecepatan kendaraan,
* estimasi potensi iklan terlihat,
* bukti billboard sedang tayang,
* kualitas data CCTV,
* performa campaign,
* report yang bisa dibaca client dari HP.

Aplikasi ini harus membantu perusahaan billboard meningkatkan kepercayaan client dan calon client melalui data yang rapi, transparan, dan bisa diaudit.

---

## 3. Prinsip Produk

Produk ini harus dibangun sebagai **SaaS serius**, bukan aplikasi custom satu perusahaan.

Prinsip utama:

* multi-tenant sejak awal,
* tenant isolation wajib keras,
* role dan permission harus jelas,
* report tidak boleh overclaim,
* formula impression harus transparan,
* data traffic wajib punya kualitas dan confidence,
* dashboard client wajib mobile-first,
* report PDF harus professional dan client-friendly,
* hourly/raw data boleh lengkap, tetapi sebaiknya ditempatkan sebagai appendix atau export Excel/CSV,
* AI tidak boleh mengarang angka,
* developer / agent tidak boleh meninggalkan mock permanen.

Billboard yang sebelumnya hanya menjadi media statis harus berubah menjadi aset media yang:

* bisa dimonitor,
* bisa diukur,
* bisa dilaporkan,
* bisa dibandingkan,
* bisa dijual dengan data,
* bisa dipertanggungjawabkan.

---

## 4. Model Bisnis SaaS

Aplikasi ini menggunakan model **multi-tenant SaaS**.

Artinya:

* satu platform pusat,
* banyak perusahaan billboard dapat menggunakan aplikasi,
* setiap perusahaan billboard disebut tenant,
* setiap tenant memiliki data sendiri,
* setiap tenant memiliki user sendiri,
* setiap tenant memiliki client sendiri,
* setiap tenant memiliki billboard sendiri,
* setiap tenant memiliki campaign sendiri,
* data antar tenant tidak boleh tercampur,
* Owner Platform SaaS mengelola tenant dan paket/tier,
* Admin Tenant mengelola operasional tenant,
* Owner Tenant hanya melihat dashboard, KPI, rekap, insight, dan report.

Contoh struktur:

```text
Platform SaaS
├── Owner Platform SaaS
│
├── Tenant A: Perusahaan Billboard A
│   ├── Admin Tenant A
│   ├── Owner Tenant A
│   ├── Sales A
│   ├── Teknisi A
│   ├── Client / Brand A1
│   ├── Client / Brand A2
│   ├── Billboard A1
│   └── Campaign A1
│
├── Tenant B: Perusahaan Billboard B
│   ├── Admin Tenant B
│   ├── Owner Tenant B
│   ├── Sales B
│   ├── Teknisi B
│   ├── Client / Brand B1
│   ├── Billboard B1
│   └── Campaign B1
│
└── Tenant C: Perusahaan Billboard C
    ├── Admin Tenant C
    ├── Owner Tenant C
    ├── Client / Brand C1
    └── Billboard C1
```

Sistem tidak boleh dibangun sebagai aplikasi satu perusahaan saja.

Semua struktur backend, database, permission, dashboard, report, file storage, API, background job, 
dan audit log wajib mempertimbangkan tenant isolation sejak awal.

---

## 5. Role Utama Aplikasi

Bagian ini adalah pondasi. Jangan dibolak-balik.

Role utama yang benar:

```text
Owner Platform SaaS
↓
Admin Tenant — Super Power Operasional Tenant
↓
Owner Tenant — Monitoring Only
↓
Sales / Teknisi / Client
```

---

### 5.1 Owner Platform SaaS

Owner Platform SaaS adalah pemilik aplikasi/platform SaaS secara keseluruhan.

Role ini menggantikan istilah lama:

```text
Super Admin Platform
```

Mulai sekarang, gunakan istilah:

```text
Owner Platform SaaS
```

Owner Platform SaaS bukan bagian dari tenant billboard tertentu. Ia adalah pemilik dan pengendali utama platform.

Tugas utama:

* mengelola seluruh tenant/perusahaan billboard,
* membuat tenant baru,
* mengaktifkan atau menonaktifkan tenant,
* membuat dan mengatur paket subscription/tier,
* mengatur limit paket seperti jumlah billboard, CCTV, user, report, dan fitur analytics,
* melakukan approval tenant baru jika diperlukan,
* melihat status subscription tenant,
* mengatur billing dan subscription,
* mengontrol fitur yang aktif pada masing-masing tier,
* memonitor kesehatan sistem secara global,
* memonitor status semua device secara global,
* melihat audit log platform,
* mengatur konfigurasi global aplikasi,
* memantau usage API, storage, report generation, dan AI processing.

Owner Platform SaaS boleh melihat data platform untuk kebutuhan support, audit, security, 
dan operasional platform. Namun akses detail bisnis tenant harus tetap dibatasi dengan audit log dan alasan akses.

Owner Platform SaaS fokus pada:

* kesehatan platform,
* bisnis SaaS,
* tenant management,
* subscription/tier,
* system health,
* security,
* audit.

Owner Platform SaaS bukan operator harian tenant.

---

### 5.2 Admin Tenant — Super Power Operasional Tenant

Admin Tenant adalah admin utama di masing-masing perusahaan billboard.

Role ini adalah role paling kuat di dalam lingkup tenant/perusahaan billboard.

Tugas utama:

* mengelola data perusahaan billboard miliknya,
* mengelola user internal tenant,
* mengelola billboard,
* mengelola sisi/muka billboard,
* mengelola CCTV,
* mengelola edge device,
* mengelola client/brand penyewa billboard,
* membuat dan mengelola campaign,
* mengatur akses dashboard client,
* upload proof display,
* approval proof display,
* melihat traffic analytics,
* generate dan download report,
* mengelola teknisi,
* melihat alert dan maintenance,
* mengatur konfigurasi operasional tenant.

Admin Tenant hanya berkuasa di dalam tenant miliknya sendiri.

Admin Tenant tidak boleh:

* melihat data tenant lain,
* mengubah data tenant lain,
* mengakses subscription global platform,
* membuat paket/tier SaaS,
* mengubah konfigurasi global platform,
* mengambil alih role Owner Platform SaaS.

Admin Tenant adalah raja operasional di tenant sendiri, tapi tetap tidak boleh loncat pagar ke kerajaan tenant lain.

---

### 5.3 Owner Tenant — Monitoring Only

Owner Tenant adalah pemilik atau pimpinan perusahaan billboard yang menggunakan platform.

Role ini **bukan super admin** dan **bukan admin operasional**.

Owner Tenant tidak perlu punya power untuk membuat, mengubah, menghapus, approval, atau mengatur user.

Owner Tenant hanya membutuhkan:

* monitoring,
* KPI card,
* rekap bisnis,
* insight,
* dashboard eksekutif,
* report.

Akses Owner Tenant:

* melihat dashboard ringkasan perusahaan,
* melihat KPI card utama,
* melihat total billboard,
* melihat total billboard aktif,
* melihat total billboard face,
* melihat total campaign aktif,
* melihat total client aktif,
* melihat CCTV online/offline,
* melihat performa traffic secara ringkas,
* melihat report harian, mingguan, dan bulanan,
* melihat campaign hampir selesai,
* melihat billboard dengan traffic tertinggi,
* melihat estimasi potential exposure,
* melihat insight bisnis,
* melihat trend performa,
* download report jika diizinkan.

Larangan Owner Tenant:

* tidak boleh membuat billboard,
* tidak boleh mengubah billboard,
* tidak boleh menghapus billboard,
* tidak boleh membuat campaign,
* tidak boleh mengubah campaign,
* tidak boleh menghapus campaign,
* tidak boleh mengatur user,
* tidak boleh mengatur role,
* tidak boleh mengatur CCTV,
* tidak boleh approval proof display,
* tidak boleh mengubah data client,
* tidak boleh mengubah subscription atau paket,
* tidak boleh mengakses data tenant lain,
* tidak boleh melihat konfigurasi global platform.

Owner Tenant adalah role eksekutif / monitoring.

Bahasa sederhananya:

> Boleh melihat ruang komando, tapi tidak boleh menekan tombol nuklir.

Dashboard Owner Tenant harus bersih, eksekutif, ringkas, dan mudah dipahami.

Tidak ada tombol berbahaya seperti:

* tambah data,
* edit data,
* delete data,
* approval,
* ubah role,
* ubah konfigurasi.

---

### 5.4 Sales Tenant

Sales adalah user internal tenant yang fokus pada aktivitas penjualan billboard dan campaign.

Tugas utama:

* melihat daftar titik billboard,
* melihat billboard yang tersedia,
* melihat performa historis billboard,
* melihat estimasi traffic,
* melihat composition kendaraan sebagai bahan jualan,
* menyiapkan bahan penawaran,
* membandingkan titik billboard,
* melihat client yang menjadi tanggung jawabnya,
* membuat proposal/campaign draft jika diizinkan,
* melihat report campaign miliknya jika diberikan akses.

Sales tidak boleh:

* mengubah data teknis penting tanpa izin,
* menghapus billboard,
* mengatur CCTV,
* approval proof display,
* mengubah role user,
* melihat semua data bisnis tenant tanpa batas,
* mengakses data tenant lain.

Sales adalah alat bantu jualan berbasis data, bukan admin kerajaan.

---

### 5.5 Teknisi Lapangan

Teknisi adalah user untuk pemasangan, pengecekan, dan maintenance perangkat di lapangan.

Tugas utama:

* memasang CCTV,
* mengecek kamera,
* update status kamera,
* upload foto bukti pemasangan iklan,
* upload foto maintenance,
* update status lokasi,
* melaporkan kendala perangkat,
* melakukan checklist lapangan,
* mencatat hasil perbaikan,
* mengirim GPS timestamp bila diperlukan.

Teknisi menggunakan aplikasi mobile atau web mobile.

Teknisi tidak boleh:

* melihat data revenue,
* melihat semua client,
* mengubah campaign,
* menghapus billboard,
* generate report bisnis,
* mengatur role,
* mengakses data tenant lain.

Teknisi fokus pada bukti lapangan dan kesehatan perangkat.

---

### 5.6 Client / Brand Penyewa Billboard

Client adalah user akhir yang menyewa billboard dan ingin melihat performa campaign miliknya.

Client hanya boleh mengakses:

* campaign miliknya,
* billboard yang sedang disewa,
* KPI campaign,
* traffic report,
* proof display yang sudah approved,
* laporan harian,
* laporan mingguan,
* laporan bulanan,
* download PDF/Excel jika diizinkan.

Client tidak boleh melihat:

* campaign client lain,
* data tenant secara penuh,
* revenue perusahaan billboard,
* konfigurasi CCTV,
* data internal tenant,
* data teknisi,
* data subscription tenant,
* dashboard Owner Tenant,
* dashboard Admin Tenant.

Client dashboard wajib mobile-first karena mayoritas client akan membuka dari HP.

---

## 6. Permission Matrix Ringkas

| Role                | Platform/Tier |    Tenant Ops |     Billboard |               CCTV |      Campaign |      Proof Display |        Report |          Monitoring |
| ------------------- | ------------: | ------------: | ------------: | -----------------: | ------------: | -----------------: | ------------: | ------------------: |
| Owner Platform SaaS |          Full | Support/Audit | Support/Audit |      Support/Audit | Support/Audit |      Support/Audit |  Global Usage |         Full Global |
| Admin Tenant        |            No |   Full Tenant |          Full |               Full |          Full |      Full Approval |   Full Tenant |         Full Tenant |
| Owner Tenant        |            No |     Read Only |     Read Only |          Read Only |     Read Only |          Read Only | Read/Download | Full Tenant Summary |
| Sales               |            No |       Limited |          Read |         No/Limited | Draft/Limited |                 No |       Limited |          Sales View |
| Teknisi             |            No |       Limited | Read Assigned | Update Maintenance |            No | Upload Field Proof |    No/Limited |      Technical View |
| Client              |            No |            No | Campaign Only |        Status Only | Campaign Only |      Approved Only | Campaign Only |    Client Dashboard |

Prinsip keras:

* Owner Tenant bukan Admin Tenant.
* Admin Tenant bukan Owner Platform SaaS.
* Client bukan tenant staff.
* Teknisi bukan sales.
* Sales bukan teknisi.
* Semua role harus dibatasi permission, bukan sekadar nama indah di sidebar.

---

## 7. Kategori Paket Billboard Berdasarkan Lokasi

Sistem harus mendukung beberapa kategori paket berdasarkan kondisi lokasi billboard.

---

### 7.1 Paket Jalan Tol

Untuk billboard di jalan tol atau jalan cepat satu arah.

Karakteristik:

* umumnya satu arah kendaraan,
* kecepatan kendaraan tinggi,
* satu CCTV utama biasanya cukup,
* cocok untuk vehicle counting,
* cocok untuk speed rate,
* komposisi kendaraan biasanya lebih banyak mobil dan truk,
* motor bisa sangat rendah atau tidak ada tergantung ruas jalan.

Data utama:

* total kendaraan,
* car count,
* motorcycle count,
* bus count,
* truck count,
* average speed,
* peak hour,
* traffic per jam,
* estimated exposure,
* CPV,
* CPI.

---

### 7.2 Paket Jalan Biasa / JPO / Satu Muka

Untuk billboard jalan biasa, flyover, JPO, atau jalan utama satu arah.

Karakteristik:

* kendaraan lebih variatif,
* banyak motor,
* potensi macet tinggi,
* billboard sering terlihat lebih lama saat padat,
* cocok untuk mass awareness campaign.

Data utama:

* total kendaraan,
* motor/mobil/bus/truk,
* jam ramai,
* congestion level,
* dwell time,
* proof display,
* daily/weekly/monthly report.

---

### 7.3 Paket Perempatan / Pertigaan / Perlimaan

Untuk billboard yang terlihat dari banyak arah.

Karakteristik:

* kendaraan datang dari beberapa arah,
* bisa membutuhkan lebih dari satu CCTV,
* perlu mapping arah kendaraan,
* potensi double count harus dikendalikan,
* perlu exposure zone per arah.

Data utama:

* traffic per arah,
* vehicle count per arah,
* vehicle class per arah,
* congestion per arah,
* exposure zone per arah,
* report per sisi/muka billboard,
* anti double count rule.

---

### 7.4 Paket Premium Landmark

Untuk billboard besar, strategis, dan bernilai tinggi.

Karakteristik:

* titik premium,
* volume traffic tinggi,
* client membutuhkan report lebih mewah,
* cocok untuk dashboard dan insight premium.

Data utama:

* multi-camera analytics,
* advanced report,
* proof display lengkap,
* historical comparison,
* estimated exposure score,
* executive recommendation,
* cross-location comparison,
* data integrity page.

---

## 8. Modul Utama Aplikasi

---

### Modul 1 — Tenant Management

Modul untuk mengelola perusahaan billboard yang menjadi pelanggan SaaS.

Modul ini hanya boleh diakses oleh Owner Platform SaaS.

Fitur:

* tambah tenant/perusahaan billboard,
* edit data tenant,
* aktif/nonaktif tenant,
* approval tenant baru,
* tenant branding,
* tenant status,
* audit tenant,
* package assignment,
* subscription status,
* billing status,
* suspend tenant jika bermasalah.

---

### Modul 2 — Subscription Plan / Tier Management

Modul untuk mengatur paket SaaS.

Modul ini hanya milik Owner Platform SaaS.

Fitur:

* membuat paket/tier,
* mengatur harga paket,
* mengatur limit billboard,
* mengatur limit CCTV,
* mengatur limit user,
* mengatur fitur yang aktif,
* mengatur jenis report,
* mengatur fitur analytics,
* mengatur masa aktif subscription,
* upgrade/downgrade tenant,
* suspend tenant jika pembayaran bermasalah.

Contoh tier:

* Starter
* Growth
* Pro
* Enterprise
* Custom Premium

Contoh limit tier:

* jumlah billboard,
* jumlah billboard face,
* jumlah CCTV,
* jumlah user internal,
* jumlah client dashboard,
* report PDF,
* report Excel,
* data retention,
* advanced analytics,
* white label,
* custom branding.

---

### Modul 3 — User & Role Management

Modul untuk mengatur user dan hak akses.

Role utama:

* Owner Platform SaaS,
* Admin Tenant,
* Owner Tenant,
* Sales,
* Teknisi,
* Client / Brand,
* Viewer / Report Viewer jika nanti dibutuhkan.

Prinsip role:

* Owner Platform SaaS mengatur tenant dan tier.
* Admin Tenant mengatur operasional tenant.
* Owner Tenant hanya monitoring.
* Sales fokus data penjualan dan campaign.
* Teknisi fokus maintenance.
* Client hanya melihat campaign miliknya.
* Semua role wajib mengikuti tenant isolation.
* Semua aksi penting wajib masuk audit log.

---

### Modul 4 — Billboard Management

Modul untuk mengelola data billboard.

Modul ini dikelola oleh Admin Tenant.

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
* sisi/muka billboard,
* arah hadap,
* visibility angle,
* kategori lokasi,
* foto lokasi,
* status aktif,
* monthly rate / estimated OOH rate jika digunakan untuk report,
* kategori paket.

Data billboard wajib mendukung lebih dari satu sisi/muka.

Contoh:

```text
Billboard A
├── Face 1 arah Jakarta
└── Face 2 arah Tangerang

Billboard B
├── Face Utara
└── Face Selatan
```

Owner Tenant boleh melihat data billboard, tetapi tidak boleh mengubah.

---

### Modul 5 — Camera & Device Management

Modul untuk mengelola CCTV, CVC, dan perangkat edge.

Data utama:

* camera ID,
* device ID,
* nama kamera,
* lokasi kamera,
* billboard terkait,
* billboard face terkait,
* RTSP/IP camera,
* stream URL,
* camera position,
* direction captured,
* FPS,
* resolution,
* status online/offline,
* last heartbeat,
* last data received,
* stream health,
* kualitas stream,
* catatan maintenance.

Admin Tenant boleh mengelola kamera.

Teknisi boleh update status lapangan dan maintenance.

Owner Tenant hanya boleh melihat status ringkas.

Client hanya boleh melihat status CCTV yang terkait campaign miliknya jika diizinkan.

CCTV merupakan bagian dari produk, sehingga sistem wajib memantau kesehatan kamera secara serius.

---

### Modul 6 — Campaign Management

Modul untuk mengelola campaign iklan client.

Data utama:

* campaign ID,
* client ID,
* nama campaign,
* brand,
* objective,
* billboard yang disewa,
* billboard face yang digunakan,
* periode mulai,
* periode selesai,
* materi iklan,
* estimated OOH budget,
* status campaign,
* proof display requirement,
* akses dashboard client,
* report schedule.

Status campaign minimal:

* draft,
* scheduled,
* active,
* paused,
* completed,
* cancelled.

Admin Tenant boleh membuat dan mengelola campaign.

Sales boleh membuat draft campaign jika diberikan izin.

Owner Tenant hanya melihat ringkasan campaign.

Client hanya melihat campaign miliknya.

---

### Modul 7 — AI Traffic Analytics

Modul untuk membaca data kendaraan dari CCTV / CVC.

Fitur:

* vehicle counting,
* vehicle classification,
* direction detection,
* speed estimation,
* congestion detection,
* dwell time estimation,
* peak hour calculation,
* hourly summary,
* daily summary,
* monthly summary,
* real-time KPI,
* anomaly detection,
* confidence score,
* data quality score.

Jenis kendaraan minimal:

* motorcycle / motor,
* car / mobil,
* bus,
* truck / truk,
* other vehicle jika diperlukan.

Catatan penting:

Sistem tidak boleh overclaim.

AI menghitung kendaraan dan peluang exposure, bukan membaca mata manusia.

---

### Modul 8 — AI Counting Rule

Modul untuk mengatur aturan hitung AI.

Data utama:

* camera ID,
* counting line coordinates,
* direction filter,
* exposure zone coordinates,
* vehicle classes,
* minimum confidence,
* duplicate tracking window,
* active status,
* rule version,
* validation status.

Aturan penting:

* kendaraan hanya dihitung jika melewati counting line yang valid,
* arah kendaraan harus sesuai direction rule,
* kendaraan di luar exposure direction tidak boleh dihitung untuk campaign exposure,
* potensi double count harus dicegah,
* perubahan rule harus masuk audit log,
* setiap report harus menyimpan methodology version dan formula version.

---

### Modul 9 — Proof of Display

Modul bukti billboard tayang.

Karena kamera utama biasanya menghadap jalan, bukti tayang untuk billboard statis dapat menggunakan:

* foto pemasangan billboard,
* timestamp,
* GPS location,
* nama teknisi,
* approval admin,
* foto berkala,
* checklist campaign,
* catatan maintenance.

Untuk paket premium dapat ditambah kamera khusus proof display.

Akses:

* Admin Tenant dapat approval.
* Teknisi dapat upload bukti lapangan.
* Owner Tenant dapat melihat ringkasan.
* Client dapat melihat proof yang sudah disetujui.

Status proof minimal:

* pending,
* approved,
* rejected,
* need revision.

---

### Modul 10 — Client Dashboard

Dashboard khusus client penyewa billboard.

Dashboard ini wajib mobile-first.

Isi dashboard:

* status campaign,
* periode campaign,
* lokasi billboard,
* total kendaraan hari ini,
* total kendaraan minggu ini,
* total kendaraan bulan ini,
* klasifikasi kendaraan,
* vehicle composition,
* peak hour,
* speed rate,
* congestion level,
* estimated potential exposure,
* proof display,
* report harian,
* report mingguan,
* report bulanan,
* download PDF,
* download Excel jika diizinkan.

Dashboard client harus sederhana, cepat, dan mudah dibaca dari HP.

Client tidak boleh dipaksa membaca tabel panjang seperti sedang menghitung warisan Excel jam 2 malam.

---

### Modul 11 — Owner Tenant Dashboard

Dashboard khusus Owner Tenant.

Dashboard ini hanya untuk monitoring bisnis dan report.

Isi dashboard:

* total billboard,
* total billboard aktif,
* total billboard face,
* total campaign aktif,
* total client aktif,
* CCTV online,
* CCTV offline,
* total estimated exposure bulan ini,
* campaign hampir selesai,
* billboard traffic tertinggi,
* report terbaru,
* ringkasan performa tenant,
* grafik trend performa,
* data quality summary,
* alert ringkasan.

Owner Tenant tidak perlu tombol create/edit/delete untuk data operasional.

Dashboard Owner Tenant harus bersih, eksekutif, ringkas, dan mudah dipahami.

---

### Modul 12 — Admin Tenant Dashboard

Dashboard khusus Admin Tenant.

Isi dashboard:

* total billboard,
* total billboard aktif,
* total campaign aktif,
* total client aktif,
* CCTV online,
* CCTV offline,
* report belum dikirim,
* campaign hampir berakhir,
* billboard traffic tertinggi,
* total estimated exposure bulan ini,
* proof pending approval,
* alert maintenance aktif,
* data quality issue,
* missing data hour,
* anomaly count.

Admin Tenant boleh melihat KPI sekaligus melakukan aksi operasional.

---

### Modul 13 — Owner Platform SaaS Dashboard

Dashboard khusus Owner Platform SaaS.

Isi dashboard:

* total tenant,
* tenant aktif,
* tenant trial,
* tenant suspended,
* total billboard seluruh platform,
* total CCTV aktif,
* total CCTV offline,
* total campaign berjalan,
* subscription aktif,
* subscription bermasalah,
* MRR/ARR jika billing sudah dibuat,
* plan/tier terpopuler,
* system health,
* error rate,
* storage usage,
* API usage,
* AI processing usage,
* report generation usage.

Owner Platform SaaS fokus pada kesehatan platform dan bisnis SaaS, bukan operasional harian tenant.

---

### Modul 14 — Report Generator

Modul untuk membuat laporan.

Jenis laporan:

* daily report,
* weekly report,
* monthly report,
* campaign summary report,
* billboard performance report,
* client report,
* cross-location comparison report.

Format output:

* dashboard web/mobile,
* PDF,
* Excel,
* CSV raw data jika dibutuhkan.

Akses report:

* Admin Tenant dapat generate dan download.
* Owner Tenant dapat melihat/download jika diizinkan.
* Client dapat melihat/download report campaign miliknya.
* Owner Platform SaaS dapat melihat report usage secara global, bukan detail bisnis tenant kecuali untuk kebutuhan support/audit resmi.

Report harus berisi:

* angka,
* grafik,
* proof,
* insight,
* metodologi,
* formula,
* data quality,
* disclaimer.

---

### Modul 15 — Alert & Maintenance

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
* missing hours terlalu banyak,
* confidence score rendah,
* data quality score rendah,
* anomaly traffic terdeteksi.

Admin Tenant dan Teknisi mendapat alert operasional.

Owner Tenant hanya melihat ringkasan status.

Owner Platform SaaS melihat alert global platform.

---

## 9. KPI Card Utama

---

### 9.1 KPI Untuk Client

KPI minimal:

* Campaign Aktif
* Sisa Hari Campaign
* Billboard Location
* Total Kendaraan Hari Ini
* Total Kendaraan Minggu Ini
* Total Kendaraan Bulan Ini
* Estimated Potential Exposure
* Motor Lewat
* Mobil Lewat
* Bus Lewat
* Truk Lewat
* Peak Hour
* Average Speed
* Congestion Level
* Proof Display Status
* CCTV Status
* Report Terbaru

---

### 9.2 KPI Untuk Admin Tenant

KPI minimal:

* Total Billboard
* Total Billboard Aktif
* Total Billboard Face
* Total Campaign Aktif
* Total Client Aktif
* CCTV Online
* CCTV Offline
* Report Belum Dikirim
* Campaign Hampir Berakhir
* Billboard Traffic Tertinggi
* Total Estimated Exposure Bulan Ini
* Proof Pending Approval
* Alert Maintenance Aktif
* Data Quality Issue
* Missing Data Hours
* Anomaly Count

Admin Tenant boleh melihat KPI sekaligus melakukan aksi operasional.

---

### 9.3 KPI Untuk Owner Tenant

KPI minimal:

* Total Billboard
* Total Billboard Aktif
* Total Campaign Aktif
* Total Client Aktif
* CCTV Online
* CCTV Offline
* Total Estimated Exposure Bulan Ini
* Billboard Traffic Tertinggi
* Campaign Hampir Berakhir
* Report Terbaru
* Performance Trend
* Data Quality Summary

Owner Tenant hanya monitoring.

Tidak ada tombol berbahaya seperti:

* tambah data,
* edit data,
* delete data,
* approval,
* ubah role,
* ubah konfigurasi.

---

### 9.4 KPI Untuk Owner Platform SaaS

KPI minimal:

* Total Tenant
* Tenant Aktif
* Tenant Trial
* Tenant Suspended
* Total Billboard Seluruh Platform
* Total CCTV Aktif
* Total CCTV Offline
* Total Campaign Berjalan
* Subscription Aktif
* Subscription Bermasalah
* MRR/ARR jika billing sudah dibuat
* Plan/Tier Terpopuler
* System Health
* Error Rate
* Storage Usage
* API Usage
* AI Processing Usage
* Report Generation Usage

---

## 10. Data Flow Sistem

Alur data utama:

```text
CCTV / CVC menangkap kondisi jalan
↓
Edge device / AI service membaca video stream
↓
AI mendeteksi kendaraan
↓
AI melakukan tracking object
↓
AI menghitung kendaraan berdasarkan counting line
↓
AI mengklasifikasi kendaraan: motor, mobil, bus, truk
↓
AI menghitung arah, speed, density, congestion jika tersedia
↓
Data dikirim ke cloud backend sebagai metadata
↓
Backend menyimpan event dan summary
↓
Aggregator membuat hourly/daily/monthly summary
↓
Formula engine menghitung exposure, CPV, CPI
↓
Quality engine menghitung confidence dan data quality score
↓
Dashboard menampilkan KPI real-time/ringkas
↓
Report harian/mingguan/bulanan dibuat otomatis
↓
Admin Tenant mengelola operasional
↓
Owner Tenant memonitor KPI dan report
↓
Client membuka dashboard mobile
↓
Owner Platform SaaS memonitor tenant, tier, subscription, dan system health
```

Sistem harus mengutamakan metadata, bukan menyimpan video mentah terus-menerus.

Video/snapshot hanya disimpan bila dibutuhkan untuk sample, proof, audit, atau debugging.

---

## 11. Output Report

Benchmark report yang harus dikejar aplikasi adalah report traffic bulanan yang menampilkan:

* cover,
* measurement service,
* penjelasan CVC,
* network map,
* location summary,
* KPI lokasi,
* traffic chart analysis,
* hourly data report,
* recap total,
* closing/branding.

Namun aplikasi kita harus naik kelas dengan tambahan:

* executive summary,
* methodology,
* data quality score,
* confidence score,
* formula transparency,
* anomaly explanation,
* recommendation,
* raw data appendix,
* report version,
* formula version,
* methodology version,
* audit trail.

---

### 11.1 Daily Report

Isi minimal:

* tanggal report,
* nama client,
* nama campaign,
* billboard location,
* total kendaraan,
* kendaraan per jam,
* klasifikasi kendaraan,
* peak hour,
* average speed jika tersedia,
* congestion level jika tersedia,
* estimated potential exposure,
* proof display status,
* CCTV uptime,
* data quality score,
* insight harian.

---

### 11.2 Weekly Report

Isi minimal:

* periode minggu,
* total kendaraan 7 hari,
* grafik traffic harian,
* peak day,
* peak hour mingguan,
* klasifikasi kendaraan,
* trend speed jika tersedia,
* trend congestion jika tersedia,
* proof display summary,
* data quality summary,
* insight mingguan.

---

### 11.3 Monthly Report

Isi minimal:

* periode bulan,
* nama client,
* nama campaign,
* brand,
* city/area,
* total lokasi dipantau,
* total vehicle traffic,
* total estimated impression,
* average daily traffic,
* CPV,
* CPI,
* vehicle composition,
* grafik mingguan,
* weekday vs weekend,
* top peak hour,
* campaign summary,
* proof display summary,
* data integrity,
* executive insight,
* recommendation,
* appendix raw data.

---

## 12. Struktur Report PDF Final

Struktur ideal report PDF:

1. Cover
2. Executive Summary
3. Campaign Overview
4. Measurement Methodology
5. Network Map
6. Overall KPI Recap
7. Location 1 Summary
8. Location 1 Traffic Analysis
9. Location 1 Hourly Heatmap
10. Location 1 Data Quality
11. Location 2 Summary
12. Location 2 Traffic Analysis
13. Location 2 Hourly Heatmap
14. Location 2 Data Quality
15. Location 3 Summary
16. Location 3 Traffic Analysis
17. Location 3 Hourly Heatmap
18. Location 3 Data Quality
19. Cross-location Comparison
20. Cost Efficiency Analysis
21. Audience & Vehicle Composition Insight
22. Recommendation
23. Raw Data Appendix
24. Disclaimer & Formula Notes
25. Closing Page

Report utama harus client-friendly.

Tabel hourly besar tetap boleh ada, tetapi sebaiknya berada di appendix atau file Excel, 
bukan memenuhi halaman utama seperti tembok angka yang membuat client ingin resign dari membaca.

---

## 13. Formula Resmi yang Harus Dikunci

Aplikasi wajib punya formula resmi.

---

### 13.1 Total Vehicle Traffic

```text
total_vehicle = car_count + motorcycle_count + bus_count + truck_count
```

---

### 13.2 Average Daily Traffic

```text
avg_daily_traffic = total_vehicle / campaign_days
```

---

### 13.3 CPV

```text
cpv = estimated_ooh_budget / total_vehicle
```

CPV = Cost per Vehicle.

---

### 13.4 Estimated Impression / Estimated Potential Exposure

Versi sederhana:

```text
estimated_impression =
(car_count × car_occupancy_multiplier)
+ (motorcycle_count × motorcycle_occupancy_multiplier)
+ (bus_count × bus_occupancy_multiplier)
+ (truck_count × truck_occupancy_multiplier)
```

Versi lebih matang:

```text
estimated_impression =
vehicle_count
× occupancy_multiplier
× visibility_factor
× exposure_factor
× data_quality_adjustment
```

Aplikasi wajib menjelaskan formula impression multiplier.

Jangan biarkan angka impression muncul seperti wahyu dari langit marketing.

---

### 13.5 CPI

```text
cpi = estimated_ooh_budget / estimated_impression
```

CPI = Cost per Impression.

---

### 13.6 Data Quality Score

Contoh formula awal:

```text
data_quality_score =
(camera_uptime_score × 0.35)
+ (detection_confidence_score × 0.30)
+ (frame_quality_score × 0.20)
+ (missing_data_score × 0.15)
```

Data quality score harus membantu client memahami seberapa layak data dijadikan dasar laporan.

---

## 14. Data Integrity

Aplikasi wajib memiliki halaman atau bagian **Data Integrity** pada report.

Isi minimal:

* expected hours,
* valid captured hours,
* missing hours,
* camera uptime,
* average AI confidence,
* data quality score,
* anomaly count,
* downtime note,
* estimated data note jika ada data yang diestimasi.

Contoh:

| Metric             |  Value |
| ------------------ | -----: |
| Expected Hours     |    744 |
| Valid Hours        |    736 |
| Missing Hours      |      8 |
| Camera Uptime      |  98.9% |
| Avg AI Confidence  |  91.4% |
| Data Quality Score | 94/100 |

Traffic count tanpa quality score itu seperti laporan keuangan tanpa bukti transaksi. 
Bisa dipercaya? Bisa-bisa cuma dipercaya oleh printer yang mencetaknya.

---

## 15. Istilah Penting

### Vehicle Count

Jumlah kendaraan yang terdeteksi melewati counting line.

### Vehicle Classification

Pengelompokan kendaraan berdasarkan jenis:

* motor,
* mobil,
* bus,
* truk,
* kendaraan lain jika dibutuhkan.

### Peak Hour

Jam dengan jumlah kendaraan tertinggi.

### Average Speed

Estimasi rata-rata kecepatan kendaraan yang melewati area pantau.

### Congestion Level

Level kepadatan lalu lintas.

Contoh:

* lancar,
* ramai lancar,
* padat,
* macet,
* macet berat.

### Dwell Time

Estimasi durasi kendaraan berada dalam area potensi melihat billboard.

### Exposure Zone

Area jalan di mana kendaraan memiliki peluang melihat billboard.

### Estimated Exposure / Estimated Impression

Estimasi potensi billboard terlihat oleh audiens.

Catatan penting:

Estimated exposure bukan klaim bahwa semua orang pasti melihat iklan.

Gunakan istilah:

* estimated exposure,
* estimated impression,
* potential exposure,
* opportunity to see,
* potensi iklan terlihat.

Jangan menggunakan klaim absolut seperti:

* pasti dilihat,
* pasti dibaca,
* pasti menghasilkan penjualan,
* pasti meningkatkan omzet,
* semua orang melihat iklan.

Sistem membaca kendaraan dan peluang exposure, bukan membaca mata manusia.

---

## 16. Database Concept Minimal

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
ai_counting_rules
traffic_events
traffic_hourly_summaries
traffic_daily_summaries
traffic_monthly_summaries
reports
report_snapshots
notifications
maintenance_logs
subscription_plans
tenant_subscriptions
tier_feature_limits
audit_logs
system_settings
```

Catatan role database:

* Owner Platform SaaS tidak terikat sebagai user operasional tenant.
* Admin Tenant wajib punya tenant_id.
* Owner Tenant wajib punya tenant_id, tetapi permission-nya read-only.
* Client wajib terkait dengan tenant dan client/campaign record.
* Teknisi wajib terkait tenant.
* Semua tabel operasional tenant wajib punya tenant_id.
* Semua query tenant wajib difilter berdasarkan tenant_id.
* Semua aksi penting wajib masuk audit_logs.
* Report wajib menyimpan formula_version dan methodology_version.

---

## 17. Data Traffic Minimal

### Hourly Traffic Fact

Field minimal:

```text
id
tenant_id
campaign_id
billboard_id
billboard_face_id
camera_id
date
hour
car_count
motorcycle_count
bus_count
truck_count
total_vehicle_count
estimated_impression
average_confidence
data_quality_score
camera_uptime_minutes
is_estimated
anomaly_flag
created_at
```

### Daily Traffic Summary

Field minimal:

```text
id
tenant_id
campaign_id
billboard_id
billboard_face_id
date
total_car
total_motorcycle
total_bus
total_truck
total_vehicle
total_impression
peak_hour
quality_score
anomaly_count
created_at
```

### Report Snapshot

Field minimal:

```text
report_id
tenant_id
campaign_id
generated_at
generated_by
report_period
report_status
pdf_url
excel_url
methodology_version
formula_version
approval_status
approved_by
approved_at
```

---

## 18. Prinsip UI/UX

Aplikasi harus mengikuti prinsip berikut:

* mobile-first untuk client,
* web admin untuk pengelolaan data besar,
* dashboard ringkas dan cepat dibaca,
* KPI card harus jelas,
* grafik tidak boleh membingungkan,
* report harus profesional,
* warna dan dekor tidak boleh mengalahkan data,
* loading harus ringan,
* data penting harus muncul lebih dulu,
* client tidak boleh dipaksa membaca tabel panjang,
* Owner Tenant dashboard harus eksekutif dan read-only,
* Admin Tenant dashboard boleh operasional,
* Owner Platform dashboard fokus SaaS/platform.

Prioritas tampilan:

1. Client mobile dashboard.
2. Owner Tenant monitoring dashboard.
3. Admin Tenant web dashboard.
4. Technician mobile workflow.
5. Report PDF/Excel.
6. Dekor visual.

Dekor dilakukan setelah fungsi utama stabil.

---

## 19. Prinsip Backend

Backend harus mengikuti prinsip berikut:

* multi-tenant sejak awal,
* semua data operasional wajib memiliki tenant_id,
* permission tidak boleh longgar,
* API harus jelas,
* validasi input wajib,
* audit log wajib untuk aksi penting,
* report harus bisa diregenerate,
* summary data harus disimpan agar dashboard tidak berat,
* raw traffic event jangan selalu dipakai langsung untuk dashboard,
* gunakan aggregation untuk hourly/daily/monthly report,
* role guard wajib di API dan UI,
* semua query tenant wajib difilter tenant_id,
* semua report wajib punya formula version,
* semua report final wajib punya snapshot.

Data antar tenant tidak boleh bocor.

Ini aturan keras.

---

## 20. Prinsip CCTV & AI Analytics

CCTV / CVC digunakan untuk:

* monitoring jalan,
* vehicle counting,
* vehicle classification,
* speed estimation,
* congestion detection,
* traffic summary,
* campaign measurement.

CCTV utama menghadap jalan.

Untuk proof billboard tayang, sistem menggunakan:

* foto pemasangan,
* GPS timestamp,
* approval admin,
* checklist teknisi,
* foto berkala,
* optional proof camera untuk paket premium.

AI tidak boleh mengklaim hal yang tidak dihitung.

Sistem membaca kendaraan dan peluang exposure, bukan membaca mata manusia.

---

## 21. Privacy & Data Safety

Sistem harus menghindari fitur sensitif yang belum diperlukan.

Pada tahap awal, jangan membangun:

* face recognition,
* license plate recognition,
* tracking individu,
* demographic detection,
* emotion detection.

Fokus pada data agregat:

* jumlah kendaraan,
* jenis kendaraan,
* speed,
* congestion,
* exposure estimation.

Data yang disimpan harus secukupnya.

Snapshot dan video sample harus memiliki retention policy.

---

## 22. Tech Direction

Tech stack dapat disesuaikan, tetapi prinsip arsitektur tidak boleh berubah.

Rekomendasi arah:

### Backend

* REST API atau hybrid REST + realtime channel.
* Multi-tenant architecture.
* Relational database.
* Background job untuk report dan aggregation.
* Authentication dan role permission kuat.
* Formula engine.
* Report snapshot engine.
* Audit log.

### Frontend Web Admin

* Dashboard Admin Tenant.
* Master data.
* Campaign management.
* Camera management.
* Report management.
* Proof approval.
* Alert monitoring.

### Owner Tenant Dashboard

* KPI summary.
* Business insight.
* Read-only report.
* Trend performance.
* CCTV status summary.
* No dangerous action button.

### Owner Platform Dashboard

* Tenant management.
* Subscription/tier management.
* System health.
* Usage monitoring.
* Billing status.
* Global alert.

### Mobile Client

* KPI campaign.
* Report ringkas.
* Proof display.
* Notification.
* Download report.

### Mobile Technician

* Upload proof.
* Camera check.
* Maintenance checklist.
* GPS timestamp.
* Issue report.

### AI/Edge Layer

* RTSP input.
* Vehicle detection.
* Vehicle classification.
* Metadata sender.
* Heartbeat sender.
* Offline buffer.
* Counting rule.
* Confidence scoring.

---

## 23. Urutan Pengembangan yang Disarankan

Pengembangan harus dilakukan bertahap.

Jangan langsung membangun semua fitur.

---

### Phase 1 — Project Foundation

* setup project,
* struktur folder,
* authentication,
* tenant model,
* role model,
* permission seed,
* basic dashboard.

Target:

Sistem punya pondasi SaaS yang benar.

---

### Phase 2 — SaaS Tenant & Role Hardening

* Owner Platform SaaS,
* Admin Tenant,
* Owner Tenant read-only,
* Sales,
* Teknisi,
* Client,
* tenant isolation,
* role-based sidebar,
* role-based API guard,
* audit log.

Target:

Tidak ada role yang salah kuasa.

Kalau role kacau, aplikasi SaaS bisa jadi pasar malam: semua orang pegang kunci loket.

---

### Phase 3 — Billboard, Client, Campaign Core

* billboard management,
* billboard face,
* client management,
* campaign management,
* assign campaign to billboard face,
* campaign period,
* campaign status.

Target:

Sistem sudah bisa mengelola inventory dan campaign.

---

### Phase 4 — Proof Display & Basic Report

* upload proof display,
* approval proof,
* report sederhana,
* export PDF,
* export Excel dasar,
* report snapshot.

Target:

Client sudah bisa mendapat bukti campaign berjalan.

---

### Phase 5 — CCTV & Device Monitoring

* register camera,
* register edge device,
* heartbeat,
* camera online/offline,
* stream health,
* alert kamera mati.

Target:

Perusahaan billboard bisa memonitor kesehatan CCTV.

---

### Phase 6 — AI Traffic Count & Summary

* vehicle count,
* hourly summary,
* daily summary,
* chart traffic,
* KPI kendaraan,
* basic confidence score.

Target:

Client mulai mendapatkan data traffic aktual.

---

### Phase 7 — Vehicle Classification, Speed, Congestion

* motor,
* mobil,
* bus,
* truk,
* speed rate,
* congestion level,
* dwell time,
* peak hour,
* exposure zone.

Target:

Report lebih bernilai bisnis.

---

### Phase 8 — Formula, CPV, CPI, Data Quality

* formula engine,
* estimated impression,
* CPV,
* CPI,
* data quality score,
* anomaly flag,
* methodology version,
* formula version.

Target:

Report tidak menjadi angka sulap.

---

### Phase 9 — Client Dashboard Mobile-First

* login client,
* lihat campaign,
* KPI card,
* report harian/mingguan/bulanan,
* proof display,
* mobile chart,
* download report.

Target:

Produk punya pembeda kuat: client bisa melihat performa billboard dari HP.

---

### Phase 10 — Report PDF/Excel Premium

* monthly report,
* executive summary,
* methodology,
* network map,
* location KPI,
* traffic chart,
* heatmap,
* data integrity,
* cross-location comparison,
* recommendation,
* appendix raw data.

Target:

Output report minimal setara benchmark, idealnya lebih transparan dan lebih profesional.

---

### Phase 11 — Technician Workflow

* mobile technician dashboard,
* upload proof,
* camera check,
* maintenance log,
* issue report,
* GPS timestamp.

Target:

Operasional lapangan terkoneksi ke sistem.

---

### Phase 12 — UI Polish & Decoration

* UI polish,
* responsive tuning,
* loading states,
* empty states,
* microcopy,
* visual improvement,
* PDF design polishing.

Target:

Aplikasi enak dipakai dan report enak dilihat.

---

### Phase 13 — Audit, Testing, Production Readiness

* security audit,
* tenant isolation audit,
* permission audit,
* performance audit,
* report accuracy audit,
* camera data audit,
* data quality audit,
* mobile UI audit,
* production checklist.

Target:

Siap production, bukan sekadar “jalan di laptop saya.”

---

## 24. Aturan Keras untuk Agent / Developer

---

### 24.1 Wajib Membaca Dokumen

Sebelum coding, agent wajib membaca:

* README.md,
* PROJECT_VISION_BUSINESS_RULES.md,
* SYSTEM_ARCHITECTURE.md,
* DATABASE_DESIGN.md,
* AI_CCTV_ANALYTICS_RULES.md,
* API_CONTRACTS_AND_EVENT_PIPELINE.md,
* UI_UX_DASHBOARD_REPORTING_GUIDE.md,
* SECURITY_PRIVACY_ACCESS_CONTROL.md,
* ROADMAP.md,
* TODO.md jika sudah ada.

Jika dokumen belum tersedia, agent harus mengikuti README.md ini sebagai sumber utama sementara.

---

### 24.2 Tidak Boleh Mengubah Arah Produk

Agent tidak boleh mengubah produk menjadi:

* aplikasi CCTV biasa,
* aplikasi single-company,
* aplikasi report manual saja,
* aplikasi website admin saja,
* aplikasi tanpa multi-tenant,
* aplikasi tanpa client dashboard,
* aplikasi tanpa proof display,
* aplikasi tanpa arah SaaS,
* aplikasi tanpa formula transparency,
* aplikasi tanpa role permission yang jelas.

Arah produk wajib tetap:

**SaaS Billboard Monitoring & Report Platform.**

---

### 24.3 Tidak Boleh Mencampur Data Tenant

Semua fitur yang berhubungan dengan tenant wajib aman.

Agent wajib memastikan:

* tenant A tidak melihat data tenant B,
* client A tidak melihat campaign client B,
* technician hanya melihat lokasi yang ditugaskan,
* report hanya mengambil data sesuai tenant dan client,
* API tidak mengembalikan data lintas tenant,
* file storage dipisahkan berdasarkan tenant context,
* export report tidak bocor antar tenant.

Tenant isolation adalah aturan inti.

---

### 24.4 Tidak Boleh Membuat Role Salah Kuasa

Aturan role wajib:

* Owner Platform SaaS mengatur platform, tenant, subscription, dan tier.
* Admin Tenant mengatur operasional tenant.
* Owner Tenant hanya monitoring/read-only.
* Client hanya melihat campaign miliknya.
* Teknisi hanya untuk maintenance/proof lapangan.
* Sales hanya untuk jualan dan draft terbatas.

Owner Tenant tidak boleh diam-diam diberi tombol admin.

Admin Tenant tidak boleh diam-diam jadi Owner Platform.

Client tidak boleh melihat data client lain.

---

### 24.5 Tidak Boleh Membuat Fitur Asal Jadi

Setiap fitur wajib memperhatikan:

* data model,
* permission,
* UI,
* validation,
* error handling,
* empty state,
* loading state,
* audit log bila perlu,
* report impact,
* mobile impact.

Jangan membuat fitur hanya tampil di UI tetapi tidak terhubung dengan data asli.

Jangan membuat mock permanen.

Jangan meninggalkan dummy data tanpa label jelas.

---

### 24.6 Tidak Boleh Overclaim Data

Report tidak boleh menyatakan bahwa iklan pasti dilihat oleh semua orang.

Gunakan istilah:

* estimated exposure,
* estimated impression,
* potential exposure,
* opportunity to see,
* potensi iklan terlihat.

Hindari:

* pasti dilihat,
* pasti dibaca,
* pasti menghasilkan penjualan,
* pasti meningkatkan omzet,
* semua orang melihat iklan.

---

### 24.7 Wajib Menjaga Mobile-First untuk Client

Client dashboard wajib nyaman di HP.

Agent tidak boleh hanya membuat tampilan desktop lalu membiarkan mobile berantakan.

Prioritas client:

* KPI cepat dibaca,
* grafik ringkas,
* report mudah diakses,
* proof display mudah dilihat,
* tombol download jelas,
* tidak terlalu banyak tabel panjang.

---

### 24.8 Wajib Update TODO dan Roadmap

Saat implementasi sudah dimulai, agent wajib menggunakan:

* TODO.md,
* ROADMAP.md.

Setiap selesai mengerjakan phase atau task penting:

* checklist task yang selesai,
* tulis output yang dihasilkan,
* tulis file yang berubah,
* tulis catatan risiko,
* tulis rekomendasi langkah berikutnya.

Ini penting karena proses development bisa terputus akibat listrik, internet, atau session agent berhenti.

---

### 24.9 Wajib Audit Berkala

Agent wajib melakukan audit berkala terhadap:

* orphan code,
* mock data,
* unused component,
* unused API,
* permission leak,
* tenant isolation issue,
* slow query,
* oversized payload,
* broken mobile UI,
* broken report,
* inconsistent naming,
* formula mismatch,
* report overclaim,
* missing audit log,
* missing data quality.

Audit bukan dilakukan hanya di akhir.

Audit harus dilakukan setiap selesai phase besar.

---

## 25. Batasan MVP

MVP tidak perlu langsung membangun semua fitur enterprise.

Fitur yang tidak wajib di MVP awal:

* face recognition,
* license plate recognition,
* demographic detection,
* emotion detection,
* full video storage 24/7,
* programmatic ads,
* public billboard marketplace,
* AI pricing otomatis,
* payment gateway kompleks,
* white label penuh.

MVP harus fokus pada:

* SaaS tenant,
* role permission yang benar,
* Owner Platform SaaS,
* Admin Tenant,
* Owner Tenant read-only,
* billboard management,
* client management,
* campaign management,
* proof display,
* CCTV monitoring,
* traffic count,
* client dashboard,
* basic report,
* formula CPV/CPI,
* estimated exposure,
* data quality basic.

---

## 26. Struktur Folder yang Disarankan

Struktur final dapat menyesuaikan tech stack.

Contoh konsep umum:

```text
project-root/
├── docs/
│   ├── README.md
│   ├── PROJECT_VISION_BUSINESS_RULES.md
│   ├── SYSTEM_ARCHITECTURE.md
│   ├── DATABASE_DESIGN.md
│   ├── AI_CCTV_ANALYTICS_RULES.md
│   ├── API_CONTRACTS_AND_EVENT_PIPELINE.md
│   ├── UI_UX_DASHBOARD_REPORTING_GUIDE.md
│   ├── SECURITY_PRIVACY_ACCESS_CONTROL.md
│   └── ROADMAP.md
│
├── backend/
│   ├── app/
│   ├── api/
│   ├── models/
│   ├── services/
│   ├── jobs/
│   ├── reports/
│   └── tests/
│
├── frontend-admin/
│   ├── src/
│   ├── components/
│   ├── pages/
│   └── services/
│
├── mobile-client/
│   ├── src/
│   ├── screens/
│   ├── components/
│   └── services/
│
├── mobile-technician/
│   ├── src/
│   ├── screens/
│   ├── components/
│   └── services/
│
├── edge-ai/
│   ├── camera/
│   ├── detection/
│   ├── tracking/
│   ├── metadata/
│   └── heartbeat/
│
├── reports/
├── scripts/
├── tests/
├── TODO.md
└── CHANGELOG.md
```

Jika project dimulai lebih sederhana, struktur boleh disederhanakan.

Namun pemisahan konsep backend, frontend, mobile, edge AI, reports, dan docs tetap harus jelas.

---

## 27. Definition of Done

Sebuah fitur dianggap selesai jika memenuhi:

* data model jelas,
* API berjalan,
* UI terhubung ke data asli,
* permission benar,
* tenant isolation aman,
* validasi input ada,
* error handling ada,
* empty state ada,
* loading state ada,
* mobile view aman,
* tidak ada mock permanen,
* tidak bocor antar tenant,
* perubahan dicatat di TODO/roadmap,
* fitur diuji minimal secara manual,
* tidak merusak fitur lain.

Untuk fitur penting seperti report, campaign, tenant, role, dan permission, pengujian harus lebih ketat.

---

## 28. Risiko Utama Project

Risiko yang harus diperhatikan:

* tenant data bocor,
* role salah kuasa,
* Owner Tenant berubah jadi admin diam-diam,
* CCTV offline terlalu sering,
* AI counting tidak akurat,
* report overclaim,
* formula impression tidak transparan,
* data quality tidak dihitung,
* dashboard terlalu berat,
* mobile UI tidak nyaman,
* database summary tidak dirancang dari awal,
* video storage membengkak,
* permission terlalu longgar,
* agent meninggalkan mock/dummy/orphan code,
* report PDF terlalu padat dan tidak client-friendly.

Setiap risiko harus dicegah sejak desain, bukan ditambal setelah rusak.

---

## 29. Roadmap Ringkas

Roadmap detail akan ditulis di dokumen:

```text
ROADMAP.md
```

Ringkasan roadmap:

```text
Phase 1  - Project Foundation
Phase 2  - SaaS Tenant & Role Hardening
Phase 3  - Billboard, Client, Campaign Core
Phase 4  - Proof Display & Basic Report
Phase 5  - CCTV & Device Monitoring
Phase 6  - AI Traffic Count & Summary
Phase 7  - Vehicle Classification, Speed, Congestion
Phase 8  - Formula, CPV, CPI, Data Quality
Phase 9  - Client Dashboard Mobile-First
Phase 10 - Report PDF/Excel Premium
Phase 11 - Technician Workflow
Phase 12 - UI Polish & Decoration
Phase 13 - Audit, Testing, Production Readiness
```

---

## 30. Dokumen Lanjutan

Setelah README.md, dokumen berikutnya yang harus dibuat atau diperkuat:

### 30.1 PROJECT_VISION_BUSINESS_RULES.md

Berisi:

* visi produk,
* target market,
* model SaaS,
* paket subscription,
* paket billboard,
* aturan campaign,
* aturan proof display,
* aturan report,
* batasan klaim data,
* formula CPV/CPI/exposure,
* role utama.

### 30.2 SYSTEM_ARCHITECTURE.md

Berisi:

* arsitektur backend,
* frontend,
* mobile app,
* edge AI,
* CCTV data flow,
* report job,
* realtime data,
* storage,
* deployment.

### 30.3 DATABASE_DESIGN.md

Berisi:

* tabel inti,
* relasi antar tabel,
* tenant isolation,
* role permission,
* campaign structure,
* traffic summary,
* proof display,
* report files,
* audit logs,
* formula version,
* methodology version.

### 30.4 AI_CCTV_ANALYTICS_RULES.md

Berisi:

* vehicle counting rule,
* classification rule,
* speed estimation,
* congestion rule,
* dwell time,
* exposure zone,
* confidence score,
* data quality score,
* anomaly detection,
* larangan overclaim.

### 30.5 API_CONTRACTS_AND_EVENT_PIPELINE.md

Berisi:

* API contract,
* event payload,
* traffic event,
* hourly summary,
* report generation pipeline,
* device heartbeat,
* alert pipeline,
* webhook jika diperlukan.

### 30.6 UI_UX_DASHBOARD_REPORTING_GUIDE.md

Berisi:

* mobile-first dashboard,
* KPI card,
* chart,
* report layout,
* PDF format,
* Excel format,
* empty state,
* loading state,
* visual style,
* role-based dashboard.

### 30.7 SECURITY_PRIVACY_ACCESS_CONTROL.md

Berisi:

* tenant isolation,
* permission matrix,
* API guard,
* file access,
* audit log,
* privacy,
* data retention,
* larangan face/license plate recognition di MVP.

### 30.8 ROADMAP.md

Berisi:

* phase development,
* checklist,
* aturan TODO.md,
* aturan update progress,
* audit per phase,
* production checklist.

---

## 31. Kesimpulan

Aplikasi ini adalah **SaaS Billboard Monitoring & Report Platform** yang menggabungkan billboard inventory, 
CCTV/CVC monitoring, AI traffic analytics, proof display, client dashboard, Owner Tenant dashboard, 
Admin Tenant dashboard, subscription/tier management, report otomatis, formula transparency, dan data quality control.

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

Role harus benar sejak awal.

Formula harus jelas.

Report harus bisa diaudit.

Data harus punya kualitas.

Karena dalam SaaS, permission itu pondasi. Kalau pondasi miring, dashboard secantik apa pun tetap rawan roboh.
