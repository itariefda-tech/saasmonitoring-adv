# AI_CCTV_ANALYTICS_RULES.md

# AI CCTV Analytics Rules — SaaS Billboard Monitoring & Report Platform

Dokumen ini adalah aturan resmi untuk semua fitur AI CCTV analytics pada aplikasi billboard monitoring & reporting.

Dokumen ini mengunci cara sistem menghitung kendaraan, mengklasifikasi kendaraan, membaca arah, menghitung speed rate, congestion, dwell time, estimated exposure, confidence score, data quality score, anomaly, dan aturan anti-overclaim.

Tujuan utamanya sederhana: angka yang tampil di dashboard dan report harus bisa dijelaskan, diaudit, dan dipertanggungjawabkan.

Kalau angka tidak punya sumber, rumus, confidence, quality, dan jejak audit, maka angka itu tidak boleh diperlakukan sebagai data final. Itu baru bisikan mesin, belum keputusan bisnis.

---

## 1. Posisi Dokumen

Dokumen ini wajib sinkron dengan:

1. `README.md`
2. `PROJECT_VISION_BUSINESS_RULES.md`
3. `SYSTEM_ARCHITECTURE.md`
4. `DATABASE_DESIGN.md`
5. `API_CONTRACTS_AND_EVENT_PIPELINE.md`
6. `UI_UX_DASHBOARD_REPORTING_GUIDE.md`
7. `SECURITY_PRIVACY_ACCESS_CONTROL.md`
8. `ROADMAP.md`

Dokumen ini adalah sumber kebenaran untuk:

- aturan vehicle counting,
- aturan vehicle classification,
- aturan counting line,
- aturan direction,
- aturan anti double count,
- aturan exposure zone,
- formula estimated exposure,
- formula CPV/CPI yang terkait analytics,
- confidence score,
- data quality score,
- anomaly detection,
- downtime handling,
- estimated data handling,
- larangan overclaim.

Jika ada perbedaan antara dokumen ini dan implementasi kode, maka kode harus dikoreksi atau dokumen ini harus direvisi secara eksplisit dengan versioning. Jangan diam-diam mengubah formula seperti mengganti bumbu rendang dengan gula aren lalu berharap client tidak sadar.

---

## 2. Prinsip Utama AI Analytics

### 2.1 Sistem Membaca Kendaraan, Bukan Mata Manusia

Sistem hanya boleh menyimpulkan peluang exposure berdasarkan kendaraan, arah, area pandang, durasi, dan kualitas data.

Gunakan istilah:

- `estimated exposure`,
- `potential exposure`,
- `opportunity to see`,
- `potensi iklan terlihat`.

Jangan gunakan klaim:

- pasti dilihat,
- pasti dibaca,
- pasti menghasilkan penjualan,
- pasti meningkatkan sales,
- guaranteed impression,
- guaranteed human views.

Alasan: kamera tidak membaca bola mata manusia. Kamera membaca kendaraan dan konteks jalan. Jadi jangan sok jadi paranormal retina.

---

### 2.2 Semua Angka Harus Punya Status

Setiap angka traffic harus punya status:

| Status | Makna |
|---|---|
| `raw` | Data mentah dari AI engine/event stream, belum divalidasi penuh. |
| `validated` | Data sudah lolos validasi rule minimum. |
| `estimated` | Data hasil estimasi karena ada downtime/missing/low quality. |
| `adjusted` | Data dikoreksi manual/otomatis dengan alasan yang tercatat. |
| `rejected` | Data dibuang dari summary karena invalid. |
| `simulation` | Data dummy/simulasi, tidak boleh muncul sebagai report final client. |

Tidak boleh ada angka tanpa status.

---

### 2.3 Semua Data Harus Punya Jejak

Setiap event dan summary wajib menyimpan minimal:

- `tenant_id`,
- `campaign_id` jika terkait campaign,
- `billboard_id`,
- `billboard_face_id` jika ada,
- `camera_id`,
- `edge_device_id` jika ada,
- `counting_rule_id`,
- `formula_version`,
- `methodology_version`,
- `data_quality_score`,
- `average_confidence_score`,
- `data_status`,
- `is_estimated`,
- `anomaly_flag`,
- `generated_at` / `captured_at`,
- `audit_trace_id` jika dipakai di report.

Prinsipnya: dari angka di PDF, developer harus bisa mundur sampai sumber kamera dan rule yang melahirkannya.

---

## 3. Konteks Kamera dan Billboard

### 3.1 Kamera Utama Menghadap Jalan

Kamera utama dipakai untuk membaca lalu lintas. Kamera ini tidak selalu menghadap langsung ke materi billboard.

Karena itu, proof of display tidak boleh diasumsikan dari kamera traffic kecuali memang camera view memperlihatkan billboard atau ada kamera proof khusus.

Proof of display tetap memakai:

- foto pemasangan,
- timestamp,
- GPS,
- upload teknisi,
- approval admin,
- foto berkala,
- kamera proof khusus untuk paket premium.

---

### 3.2 Satu Kamera Wajib Terikat ke Pemilik Billboard yang Sah

Satu kamera tidak boleh dipakai sembarangan untuk mengklaim traffic banyak billboard milik tenant berbeda.

Aturan:

- kamera wajib milik/diotorisasi oleh tenant,
- kamera wajib terkait ke billboard atau billboard face,
- kamera wajib punya arah pandang yang jelas,
- kamera wajib punya counting rule aktif,
- kamera wajib punya exposure zone jika dipakai untuk estimated exposure,
- multi-tenant access ke kamera yang sama dilarang kecuali ada skema sharing resmi dan audit legal.

---

### 3.3 Tipe Lokasi

Sistem wajib mendukung tipe lokasi berikut:

| Tipe Lokasi | Aturan Dasar |
|---|---|
| `toll_one_direction` | Umumnya satu arah; fokus speed, vehicle class, peak hour, estimated exposure. |
| `urban_arterial` | Jalan kota; bisa satu/dua arah; fokus volume, congestion, dwell time, peak hour. |
| `jpo_one_face` | Umumnya satu muka; traffic dari arah tertentu paling relevan. |
| `intersection_multi_direction` | Perempatan/pertigaan/perlimaan; perlu multi-camera/multi-direction dan anti double count. |
| `premium_landmark` | Bisa multi-camera dan proof lebih kuat. |

Tipe lokasi menentukan rule direction, exposure zone, dan cara summary digabung.

---

## 4. Vehicle Classes

### 4.1 Kelas Minimum

Kelas kendaraan minimum:

| Class Code | Label Indonesia | Label English |
|---|---|---|
| `motorcycle` | Motor | Motorcycle |
| `car` | Mobil | Car |
| `bus` | Bus | Bus |
| `truck` | Truk | Truck |

---

### 4.2 Aturan Pickup dan Van

Untuk MVP Indonesia:

- pickup boleh masuk `truck` atau `light_truck`,
- van boleh masuk `car` atau `light_truck`,
- keputusan final wajib disimpan di `vehicle_class_mapping`,
- report client harus memakai 4 kelas utama: motorcycle, car, bus, truck,
- jika sistem internal memakai sub-class, summary report tetap harus bisa digabung ke 4 kelas utama.

Rekomendasi awal:

| Deteksi Internal | Mapping Report |
|---|---|
| sedan/hatchback/SUV/MPV | `car` |
| pickup kecil | `truck` / `light_truck` lalu rollup ke `truck` |
| van penumpang | `car` jika mirip MPV besar; `truck` jika van logistik |
| bus kecil/besar | `bus` |
| truk ringan/berat | `truck` |

Jangan membuat kelas kendaraan baru di UI/report tanpa update dokumen ini, database, API, dashboard, dan report template.

---

## 5. AI Detection Pipeline

Pipeline minimum:

```text
CCTV / CVC Stream
↓
Frame Capture / Sampling
↓
Object Detection
↓
Vehicle Classification
↓
Object Tracking
↓
Counting Line Crossing
↓
Direction Validation
↓
Duplicate Filtering
↓
Confidence & Quality Calculation
↓
Traffic Event Store
↓
Hourly Aggregator
↓
Daily Aggregator
↓
Dashboard / Report Snapshot
```

Setiap tahap harus bisa menghasilkan log teknis minimum untuk debugging.

---

## 6. Counting Rule

### 6.1 Counting Line

Counting line adalah garis virtual yang menentukan kapan kendaraan dihitung.

Syarat counting line:

- disimpan sebagai koordinat polygon/line pada frame kamera,
- terkait ke `camera_id`,
- punya `direction_rule`,
- punya `minimum_confidence`,
- punya `duplicate_tracking_window_seconds`,
- punya `active_from` dan `active_until`,
- punya `calibration_status`,
- versioned.

Data kendaraan hanya dihitung jika kendaraan melewati counting line sesuai arah yang valid.

---

### 6.2 Direction Rule

Direction rule menentukan arah kendaraan yang dihitung.

Contoh:

| Direction Code | Makna |
|---|---|
| `left_to_right` | Kendaraan bergerak kiri ke kanan frame. |
| `right_to_left` | Kendaraan bergerak kanan ke kiri frame. |
| `top_to_bottom` | Kendaraan bergerak atas ke bawah frame. |
| `bottom_to_top` | Kendaraan bergerak bawah ke atas frame. |
| `inbound` | Menuju area exposure/billboard. |
| `outbound` | Menjauh dari area exposure/billboard. |
| `both_validated` | Dua arah dihitung, tetapi masing-masing diberi label arah. |

Tidak boleh menghitung dua arah sebagai satu angka tanpa label direction jika lokasi memiliki dua arah relevan.

---

### 6.3 Anti Double Count

Sistem wajib mencegah kendaraan yang sama dihitung berkali-kali.

Aturan minimum:

- gunakan tracking ID per objek,
- satu tracking ID hanya boleh dihitung satu kali per counting line dalam window tertentu,
- jika tracking ID hilang lalu muncul lagi dalam area yang sama, gunakan duplicate window,
- default duplicate window awal: 5–15 detik, configurable per kamera,
- multi-camera intersection wajib punya deduplication strategy jika overlap view terjadi.

Field yang dibutuhkan:

- `tracking_id`,
- `object_id`,
- `counting_line_id`,
- `first_seen_at`,
- `crossed_at`,
- `last_seen_at`,
- `duplicate_check_key`,
- `is_duplicate`.

---

## 7. Confidence Score

### 7.1 Definisi

Confidence score adalah tingkat keyakinan AI terhadap deteksi dan klasifikasi kendaraan.

Confidence bukan kebenaran absolut. Confidence adalah sinyal probabilistik.

---

### 7.2 Level Confidence

| Score | Status | Perlakuan |
|---:|---|---|
| `>= 0.85` | High | Boleh masuk validated jika quality lain bagus. |
| `0.70 - 0.84` | Medium | Boleh dihitung, tetapi ditandai normal/acceptable. |
| `0.50 - 0.69` | Low | Masuk raw/review; dapat dihitung jika rule tenant mengizinkan. |
| `< 0.50` | Very Low | Jangan masuk validated count; masuk rejected atau review. |

Threshold harus configurable per tenant/camera/rule, tetapi default platform harus konservatif.

---

### 7.3 Confidence per Event

Setiap traffic event wajib menyimpan:

- `detection_confidence`,
- `classification_confidence`,
- `tracking_confidence`,
- `direction_confidence`,
- `final_confidence_score`.

Contoh formula awal:

```text
final_confidence_score =
(detection_confidence × 0.40)
+ (classification_confidence × 0.30)
+ (tracking_confidence × 0.20)
+ (direction_confidence × 0.10)
```

Formula ini harus versioned melalui `formula_version`.

---

## 8. Data Quality Score

### 8.1 Definisi

Data Quality Score adalah nilai kualitas data pada periode tertentu, biasanya per jam, per hari, dan per report.

Data Quality Score bukan hanya confidence AI. Data quality juga membaca uptime kamera, missing data, frame quality, blur, occlusion, pencahayaan, dan anomaly.

---

### 8.2 Formula Awal

Formula awal platform:

```text
data_quality_score =
(camera_uptime_score × 0.35)
+ (detection_confidence_score × 0.30)
+ (frame_quality_score × 0.20)
+ (missing_data_score × 0.15)
```

Skala: `0 - 100`.

---

### 8.3 Komponen Score

#### A. Camera Uptime Score

```text
camera_uptime_score = valid_online_minutes / expected_minutes × 100
```

Contoh per jam:

```text
expected_minutes = 60
valid_online_minutes = 57
camera_uptime_score = 95
```

#### B. Detection Confidence Score

```text
detection_confidence_score = average_final_confidence × 100
```

#### C. Frame Quality Score

Frame quality dihitung dari beberapa sinyal:

- brightness,
- blur,
- rain/noise,
- occlusion,
- resolution,
- frame drop,
- night visibility.

Formula awal:

```text
frame_quality_score =
(brightness_score × 0.20)
+ (blur_score × 0.25)
+ (occlusion_score × 0.25)
+ (resolution_score × 0.15)
+ (frame_stability_score × 0.15)
```

#### D. Missing Data Score

```text
missing_data_score = 100 - (missing_minutes / expected_minutes × 100)
```

Jika missing data terlalu tinggi, periode tersebut tidak boleh tampil sebagai data normal.

---

### 8.4 Data Quality Level

| Score | Level | Perlakuan UI/Report |
|---:|---|---|
| `90 - 100` | Excellent | Tampil normal. |
| `80 - 89` | Good | Tampil normal dengan metadata. |
| `70 - 79` | Fair | Tampilkan warning ringan. |
| `60 - 69` | Poor | Tampilkan warning jelas; jangan jadi klaim utama. |
| `< 60` | Critical | Jangan jadi angka final tanpa review/estimasi/approval. |

---

## 9. Data Status dan Validasi

### 9.1 Validated Data

Data boleh menjadi `validated` jika:

- camera uptime periode memenuhi threshold,
- confidence rata-rata memenuhi threshold,
- frame quality memenuhi threshold,
- counting line aktif dan valid,
- direction rule valid,
- tidak ada anomaly critical,
- tidak terdeteksi duplicate berlebihan.

---

### 9.2 Estimated Data

Data menjadi `estimated` jika:

- kamera offline pada sebagian periode,
- missing data di bawah batas yang masih bisa diestimasi,
- frame quality rendah tetapi pola historis cukup kuat,
- sistem melakukan interpolation/backfill.

Estimated data wajib:

- diberi label `estimated`,
- menyimpan alasan estimasi,
- menyimpan metode estimasi,
- menyimpan periode terdampak,
- tidak boleh disamarkan sebagai validated data.

Field:

- `is_estimated = true`,
- `estimation_method`,
- `estimation_reason`,
- `source_period_reference`,
- `approved_by` jika perlu approval manual.

---

### 9.3 Rejected Data

Data menjadi `rejected` jika:

- confidence terlalu rendah,
- frame rusak parah,
- kamera salah arah,
- counting line belum dikalibrasi,
- duplicate count ekstrem,
- timestamp rusak,
- tenant/camera/campaign tidak valid,
- data simulation masuk ke environment production.

Rejected data tidak boleh masuk dashboard client final atau report final.

---

## 10. Hourly, Daily, Monthly Aggregation

### 10.1 Traffic Event

Traffic event adalah data granular setiap kendaraan yang dihitung.

Minimal field:

- `traffic_event_id`,
- `tenant_id`,
- `camera_id`,
- `billboard_id`,
- `campaign_id`,
- `vehicle_class`,
- `direction`,
- `counting_line_id`,
- `crossed_at`,
- `final_confidence_score`,
- `data_quality_snapshot_id`,
- `tracking_id`,
- `is_duplicate`,
- `data_status`.

---

### 10.2 Hourly Summary

Hourly summary adalah sumber utama dashboard dan report.

Minimal field:

- `date`,
- `hour`,
- `camera_id`,
- `billboard_id`,
- `campaign_id`,
- `car_count`,
- `motorcycle_count`,
- `bus_count`,
- `truck_count`,
- `total_vehicle_count`,
- `estimated_exposure`,
- `average_speed`,
- `congestion_level`,
- `average_confidence_score`,
- `data_quality_score`,
- `valid_minutes`,
- `missing_minutes`,
- `is_estimated`,
- `anomaly_flag`.

---

### 10.3 Daily Summary

Daily summary dihitung dari hourly summary.

```text
daily_total_vehicle = sum(hourly_total_vehicle)
daily_car = sum(hourly_car_count)
daily_motorcycle = sum(hourly_motorcycle_count)
daily_bus = sum(hourly_bus_count)
daily_truck = sum(hourly_truck_count)
daily_estimated_exposure = sum(hourly_estimated_exposure)
daily_quality_score = weighted_average(hourly_quality_score, valid_minutes)
```

---

### 10.4 Monthly Summary

Monthly summary dihitung dari daily summary.

```text
monthly_total_vehicle = sum(daily_total_vehicle)
monthly_estimated_exposure = sum(daily_estimated_exposure)
avg_daily_traffic = monthly_total_vehicle / valid_campaign_days
monthly_quality_score = weighted_average(daily_quality_score, valid_hours)
```

Report bulanan tidak boleh menghitung hari missing sebagai hari valid tanpa label.

---

## 11. Formula Traffic dan Report

### 11.1 Total Vehicle

```text
total_vehicle = car_count + motorcycle_count + bus_count + truck_count
```

---

### 11.2 Vehicle Composition

```text
vehicle_class_percentage = vehicle_class_count / total_vehicle × 100
```

Jika `total_vehicle = 0`, persentase harus `0` atau `N/A`, bukan error.

---

### 11.3 Average Daily Traffic

```text
avg_daily_traffic = total_vehicle / valid_campaign_days
```

`valid_campaign_days` harus memperhitungkan periode campaign aktif dan data availability.

---

### 11.4 CPV

```text
cpv = estimated_ooh_budget / total_vehicle
```

Jika `total_vehicle = 0`, CPV harus `N/A`, bukan infinity.

---

### 11.5 CPI

```text
cpi = estimated_ooh_budget / estimated_exposure
```

Jika `estimated_exposure = 0`, CPI harus `N/A`.

---

## 12. Estimated Exposure

### 12.1 Definisi

Estimated exposure adalah estimasi peluang iklan terlihat berdasarkan kendaraan yang melewati exposure zone, vehicle class, occupancy multiplier, visibility factor, exposure factor, dwell factor, dan data quality adjustment.

Estimated exposure bukan jumlah orang yang pasti melihat.

---

### 12.2 Formula Sederhana MVP

```text
estimated_exposure =
(car_count × car_occupancy_multiplier)
+ (motorcycle_count × motorcycle_occupancy_multiplier)
+ (bus_count × bus_occupancy_multiplier)
+ (truck_count × truck_occupancy_multiplier)
```

Occupancy multiplier wajib tersimpan di database/config, bukan hard-coded di frontend.

---

### 12.3 Formula Production

```text
estimated_exposure =
base_vehicle_exposure
× visibility_factor
× exposure_zone_factor
× dwell_factor
× direction_factor
× data_quality_adjustment
```

Dengan:

```text
base_vehicle_exposure =
(car_count × car_occupancy_multiplier)
+ (motorcycle_count × motorcycle_occupancy_multiplier)
+ (bus_count × bus_occupancy_multiplier)
+ (truck_count × truck_occupancy_multiplier)
```

---

### 12.4 Occupancy Multiplier

Multiplier awal boleh dipakai sebagai seed simulasi, tetapi final harus bisa dikonfigurasi per negara, kota, road type, tenant, atau campaign.

Contoh struktur config:

| Vehicle Class | Field Config |
|---|---|
| motorcycle | `motorcycle_occupancy_multiplier` |
| car | `car_occupancy_multiplier` |
| bus | `bus_occupancy_multiplier` |
| truck | `truck_occupancy_multiplier` |

Aturan keras:

- multiplier tidak boleh disembunyikan,
- report harus menyimpan `formula_version`,
- methodology page harus menjelaskan multiplier yang dipakai,
- perubahan multiplier tidak boleh mengubah report lama tanpa snapshot/versioning.

---

### 12.5 Visibility Factor

Visibility factor menghitung seberapa relevan kendaraan terhadap billboard.

Parameter:

- billboard facing direction,
- camera direction,
- road direction,
- visibility angle,
- distance estimate,
- obstruction,
- side/front visibility.

Skala awal:

| Value | Makna |
|---:|---|
| `1.00` | Sangat relevan; kendaraan berada dalam arah exposure utama. |
| `0.75` | Relevan, tetapi angle tidak ideal. |
| `0.50` | Mungkin terlihat, tetapi exposure lemah. |
| `0.25` | Sangat lemah; hanya dipakai jika disetujui. |
| `0.00` | Tidak dihitung exposure. |

---

### 12.6 Exposure Zone Factor

Exposure zone adalah area dalam frame/jalan yang dianggap punya peluang melihat billboard.

Aturan:

- exposure zone wajib berupa polygon/area,
- tidak semua kendaraan yang lewat otomatis masuk exposure,
- kendaraan harus melewati area relevan,
- exposure zone harus versioned,
- perubahan exposure zone harus masuk audit log.

---

### 12.7 Dwell Factor

Dwell factor memperhitungkan durasi kendaraan berada di exposure zone.

Contoh awal:

| Dwell Time | Dwell Factor |
|---:|---:|
| `< 1 detik` | 0.25 |
| `1 - 3 detik` | 0.50 |
| `3 - 7 detik` | 0.75 |
| `> 7 detik` | 1.00 |

Dwell factor tidak boleh dipakai jika kamera tidak bisa menghitung durasi object tracking dengan cukup stabil.

---

### 12.8 Data Quality Adjustment

```text
data_quality_adjustment = data_quality_score / 100
```

Contoh:

```text
estimated_exposure_before_quality = 100,000
data_quality_score = 80
estimated_exposure_after_quality = 80,000
```

Catatan: keputusan apakah report menampilkan exposure sebelum atau sesudah quality adjustment harus dikunci per formula version.

Rekomendasi production: tampilkan angka final setelah adjustment dan tampilkan catatan metodologi.

---

## 13. Speed Rate

### 13.1 Syarat Menghitung Speed

Speed rate hanya boleh dihitung jika kamera/rule sudah dikalibrasi.

Syarat:

- ada skala jarak nyata,
- ada calibration reference,
- FPS stabil,
- object tracking stabil,
- perspective correction jika diperlukan,
- road segment yang diukur jelas.

Jika tidak memenuhi syarat, UI harus menampilkan:

```text
Speed data not available / belum dikalibrasi
```

Jangan tampilkan angka speed palsu. Lebih baik jujur pahit daripada dashboard wangi tapi bohong.

---

### 13.2 Formula Speed

```text
speed_kmh = distance_meter / time_second × 3.6
```

Speed di dashboard adalah average speed, bukan speed enforcement.

Larangan:

- jangan gunakan untuk tilang,
- jangan klaim presisi seperti radar resmi,
- jangan tampilkan speed individual ke client jika tidak perlu,
- tampilkan average speed per periode/lokasi.

---

## 14. Congestion Level

### 14.1 Definisi

Congestion level adalah indikator kemacetan berdasarkan gabungan volume kendaraan, speed, density, queue, dan dwell time.

---

### 14.2 Level Congestion

| Level | Label | Makna |
|---:|---|---|
| 0 | Free Flow | Lalu lintas lancar. |
| 1 | Light | Ramai ringan. |
| 2 | Moderate | Padat sedang. |
| 3 | Heavy | Padat/macet. |
| 4 | Severe | Macet berat/tersendat panjang. |

---

### 14.3 Formula Awal

Jika speed tersedia:

```text
congestion_score =
(speed_score × 0.45)
+ (density_score × 0.30)
+ (dwell_score × 0.15)
+ (queue_score × 0.10)
```

Jika speed belum tersedia:

```text
congestion_score =
(density_score × 0.50)
+ (dwell_score × 0.30)
+ (queue_score × 0.20)
```

Congestion formula harus disimpan sebagai `formula_version`.

---

## 15. Dwell Time

### 15.1 Definisi

Dwell time adalah durasi kendaraan berada di exposure zone atau area observasi.

```text
dwell_time_seconds = exit_time_from_zone - enter_time_to_zone
```

---

### 15.2 Kegunaan

Dwell time dipakai untuk:

- congestion signal,
- exposure factor,
- peak congestion analysis,
- report insight.

Dwell time tidak boleh diklaim sebagai durasi orang melihat billboard.

---

## 16. Peak Hour dan Peak Day

### 16.1 Peak Hour

```text
peak_hour = hour with max(total_vehicle_count)
```

Jika ada beberapa jam dengan nilai sama, gunakan:

1. jam dengan data quality lebih tinggi,
2. jam dengan confidence lebih tinggi,
3. jam pertama secara kronologis.

---

### 16.2 Peak Day

```text
peak_day = date with max(total_vehicle_count)
```

Peak day harus menyimpan total traffic dan quality score.

---

## 17. Anomaly Detection

### 17.1 Jenis Anomaly

| Code | Makna |
|---|---|
| `sudden_drop` | Traffic turun tajam dibanding baseline. |
| `sudden_spike` | Traffic naik tidak wajar dibanding baseline. |
| `zero_traffic_when_expected` | Traffic 0 pada jam yang biasanya ramai. |
| `class_distribution_shift` | Komposisi kendaraan berubah ekstrem. |
| `camera_offline` | Kamera mati. |
| `low_confidence` | Confidence turun di bawah threshold. |
| `low_frame_quality` | Frame blur/gelap/occluded. |
| `duplicate_suspected` | Duplicate count dicurigai tinggi. |
| `rule_changed` | Rule berubah dalam periode report. |
| `calibration_missing` | Rule/kamera belum dikalibrasi. |

---

### 17.2 Severity

| Severity | Perlakuan |
|---|---|
| `info` | Dicatat, tidak mengganggu report. |
| `low` | Tampil sebagai catatan kecil. |
| `medium` | Tampil di data quality section. |
| `high` | Butuh review sebelum report final. |
| `critical` | Data tidak boleh dipakai sebagai final tanpa koreksi/approval. |

---

### 17.3 Baseline Anomaly

Baseline bisa berasal dari:

- rata-rata 7 hari terakhir,
- rata-rata weekday/weekend,
- periode campaign sebelumnya,
- pola jam yang sama,
- manual baseline per lokasi.

Contoh rule awal:

```text
if current_hour_total < baseline_hour_total × 0.35:
    anomaly = sudden_drop

if current_hour_total > baseline_hour_total × 2.50:
    anomaly = sudden_spike
```

Threshold harus configurable.

---

## 18. Downtime dan Missing Data

### 18.1 Expected Hours

Untuk campaign 24 jam:

```text
expected_hours = campaign_days × 24
```

Untuk campaign dengan jadwal khusus:

```text
expected_hours = sum(active_schedule_hours)
```

---

### 18.2 Valid Hours

Valid hours adalah jam yang memenuhi threshold minimum:

- camera uptime cukup,
- frame quality cukup,
- confidence cukup,
- rule aktif.

---

### 18.3 Missing Hours

```text
missing_hours = expected_hours - valid_hours
```

Missing hours wajib ditampilkan di data integrity report.

---

### 18.4 Estimation Policy

Estimasi missing data hanya boleh dilakukan jika:

- missing tidak terlalu besar,
- ada baseline historis cukup,
- tidak ada anomaly critical,
- estimation method jelas,
- report menampilkan label estimated.

Rekomendasi batas awal:

| Missing Ratio | Perlakuan |
|---:|---|
| `0 - 5%` | Boleh auto-estimate dengan label. |
| `>5% - 15%` | Butuh review admin. |
| `>15% - 30%` | Butuh approval khusus dan warning report. |
| `>30%` | Jangan generate report final otomatis. |

---

## 19. Multi-Camera dan Intersection Rules

### 19.1 Multi-Direction

Untuk perempatan/pertigaan/perlimaan:

- setiap arah harus punya label,
- setiap kamera harus punya counting line sendiri,
- summary harus bisa per arah dan total gabungan,
- exposure zone harus jelas per billboard face,
- anti double count wajib jika view overlap.

---

### 19.2 Aggregation Rule

```text
intersection_total_vehicle = sum(valid_direction_vehicle_count) - duplicate_estimate
```

Jika duplicate tidak bisa dihitung akurat, jangan kurangi diam-diam. Tampilkan catatan:

```text
Multi-direction total may include overlapping traffic if camera views overlap and deduplication is not enabled.
```

Versi Indonesia:

```text
Total multi-arah dapat mengandung overlap jika area kamera saling bertumpuk dan deduplication belum aktif.
```

---

## 20. Dashboard Display Rules

### 20.1 KPI Wajib

Dashboard analytics minimal menampilkan:

- total vehicle,
- car,
- motorcycle,
- bus,
- truck,
- estimated exposure,
- average daily traffic,
- peak hour,
- peak day,
- average speed jika tersedia,
- congestion level,
- data quality score,
- average confidence score,
- valid hours,
- missing hours,
- anomaly count.

---

### 20.2 Label Data

UI wajib menampilkan badge:

| Badge | Kapan Tampil |
|---|---|
| `Validated` | Data lolos validasi. |
| `Estimated` | Ada estimasi/backfill. |
| `Low Quality` | Data quality di bawah threshold. |
| `Anomaly Detected` | Ada anomaly. |
| `Simulation` | Data dummy/simulasi. |
| `Not Calibrated` | Speed/exposure belum valid karena kalibrasi belum selesai. |

---

### 20.3 Jangan Menipu dengan UI

Jika data quality rendah, jangan disembunyikan di tooltip kecil yang harus dicari pakai kaca pembesar warisan nenek.

Warning harus terlihat jelas di dashboard dan report.

---

## 21. Report Rules

### 21.1 Report Wajib Menampilkan Methodology

Report harus punya halaman/section metodologi yang menjelaskan:

- data source,
- camera ID,
- counting line,
- direction rule,
- vehicle class,
- confidence policy,
- data quality policy,
- estimated exposure formula,
- downtime handling,
- formula version,
- methodology version.

---

### 21.2 Data Integrity Section

Report wajib punya section data integrity:

| Metric | Contoh |
|---|---:|
| Expected Hours | 744 |
| Valid Hours | 736 |
| Missing Hours | 8 |
| Camera Uptime | 98.9% |
| Avg AI Confidence | 91.4% |
| Data Quality Score | 94/100 |
| Anomaly Count | 2 |
| Estimated Hours | 4 |

---

### 21.3 Raw Data Appendix

Raw data lengkap sebaiknya masuk:

- Excel,
- CSV,
- appendix PDF jika dibutuhkan.

PDF utama fokus ke executive summary, insight, chart, dan data integrity. Jangan menjadikan tabel hourly raksasa sebagai hidangan utama. Itu seperti menyajikan karung beras di piring makan.

---

## 22. Auto Insight Rules

### 22.1 Insight Harus Berdasarkan Data

Auto insight boleh membuat narasi seperti:

- traffic tertinggi terjadi pada hari X,
- peak hour terjadi pukul Y,
- motor mendominasi Z%,
- data quality turun pada periode tertentu,
- lokasi A lebih efisien dari CPI dibanding lokasi B.

Auto insight tidak boleh mengklaim sebab pasti jika tidak ada data pendukung.

Gunakan bahasa:

- `kemungkinan`,
- `terindikasi`,
- `berkorelasi`,
- `perlu dicek`,
- `dapat menjadi sinyal`.

Hindari bahasa:

- `pasti karena`,
- `terbukti menyebabkan`,
- `jaminan`,
- `dipastikan`.

---

### 22.2 Business Recommendation

Recommendation engine boleh memberi saran berbasis:

- total traffic,
- CPI,
- CPV,
- vehicle composition,
- peak hour,
- weekday/weekend pattern,
- data quality.

Contoh:

```text
Lokasi ini kuat untuk awareness mass-market karena total traffic tinggi dan dominasi motorcycle besar. Namun data quality pada beberapa jam malam rendah, sehingga insight malam hari perlu dibaca hati-hati.
```

---

## 23. Security dan Privacy Rules untuk AI Analytics

AI analytics dilarang melakukan:

- facial recognition,
- identifikasi wajah,
- deteksi ras/agama/atribut sensitif,
- pelacakan plat nomor untuk identitas personal,
- klaim demografi personal tanpa dasar legal,
- menyimpan raw video sembarangan tanpa retention policy,
- expose raw RTSP ke client.

Fokus sistem adalah kendaraan dan traffic, bukan identitas orang.

---

## 24. Database Requirements

Tabel yang harus mendukung dokumen ini:

- `ai_counting_rules`,
- `traffic_events`,
- `traffic_hourly_summaries`,
- `traffic_daily_summaries`,
- `traffic_monthly_summaries`,
- `data_quality_snapshots`,
- `camera_health_logs`,
- `camera_snapshots`,
- `anomaly_events`,
- `formula_versions`,
- `methodology_versions`,
- `exposure_zone_rules`,
- `vehicle_class_mappings`,
- `report_snapshots`,
- `audit_logs`.

Semua tabel operasional tenant wajib punya `tenant_id`.

---

## 25. API Requirements

Endpoint analytics harus mendukung:

- filter tenant,
- filter campaign,
- filter billboard,
- filter camera,
- filter direction,
- filter vehicle class,
- filter date range,
- data mode: raw/validated/estimated,
- quality threshold,
- anomaly include/exclude.

Endpoint minimum:

```text
GET /analytics/summary
GET /analytics/timeseries
GET /analytics/vehicle-composition
GET /analytics/hourly-heatmap
GET /analytics/location/:locationId
GET /analytics/data-quality
GET /analytics/anomalies
GET /analytics/methodology
GET /analytics/formula-version
```

Semua endpoint wajib:

- authenticated,
- role-checked,
- tenant-filtered,
- campaign-filtered untuk client,
- audited jika export/download/generate report.

---

## 26. Testing Rules

AI analytics harus dites minimal untuk:

1. counting satu arah,
2. counting dua arah,
3. multi-camera intersection,
4. duplicate tracking,
5. low confidence filtering,
6. camera offline,
7. missing hours,
8. estimated data,
9. anomaly spike,
10. anomaly drop,
11. zero traffic abnormal,
12. class distribution shift,
13. report formula snapshot,
14. tenant isolation,
15. client campaign isolation.

Tidak boleh merge fitur analytics tanpa test untuk tenant isolation dan data status.

---

## 27. Rules untuk Agent / Developer

Aturan keras:

- Jangan membuat angka traffic dummy tanpa label `simulation`.
- Jangan membuat impression seolah pasti dilihat manusia.
- Jangan menghitung kendaraan tanpa counting line.
- Jangan menghitung direction tanpa direction rule.
- Jangan tampilkan speed jika belum dikalibrasi.
- Jangan tampilkan exposure jika exposure zone belum diset.
- Jangan generate report final tanpa formula version.
- Jangan generate report final tanpa methodology version.
- Jangan sembunyikan low data quality.
- Jangan hapus confidence score dari event/summary.
- Jangan hapus data quality score dari summary/report.
- Jangan campur data raw dan validated tanpa label.
- Jangan expose raw RTSP.
- Jangan membuat Client melihat data campaign lain.
- Jangan membuat Owner Tenant punya tombol operasional.
- Jangan membuat Owner Platform SaaS menjadi admin tenant biasa.

Aturan positif:

- Selalu simpan sumber data.
- Selalu simpan status data.
- Selalu simpan quality score.
- Selalu simpan confidence score.
- Selalu simpan formula version.
- Selalu simpan methodology version.
- Selalu audit perubahan rule.
- Selalu tampilkan warning jika data lemah.
- Selalu pisahkan raw data, validated data, dan estimated data.
- Selalu desain report agar client paham, bukan tenggelam di tabel.

---

## 28. MVP Implementation Scope

### MVP Wajib

- vehicle counting,
- 4 kelas kendaraan utama,
- counting line,
- direction rule dasar,
- hourly summary,
- daily summary,
- confidence score,
- data quality score awal,
- estimated exposure formula sederhana,
- anomaly flag sederhana,
- report methodology notes,
- export PDF/Excel/CSV dengan formula version.

### Setelah MVP

- multi-camera deduplication,
- speed calibration,
- congestion formula matang,
- dwell factor,
- exposure zone polygon editor,
- recommendation engine,
- advanced anomaly detection,
- model performance monitoring.

---

## 29. Acceptance Criteria

AI analytics dianggap layak masuk production jika:

- setiap angka dashboard punya sumber data,
- setiap summary punya confidence score,
- setiap summary punya data quality score,
- setiap report punya formula version,
- setiap report punya methodology version,
- data estimated diberi label jelas,
- data low quality diberi warning,
- dashboard client tidak menampilkan klaim berlebihan,
- tenant isolation aman,
- client hanya melihat campaign miliknya,
- raw RTSP tidak bocor,
- audit log aktif untuk perubahan rule dan report generation,
- report bisa menjelaskan CPV, CPI, dan estimated exposure secara transparan.

---

## 30. Kesimpulan

AI CCTV Analytics adalah jantung angka aplikasi ini.

Dashboard boleh cantik, report boleh premium, chart boleh menari seperti kembang api malam tahun baru. Tapi jika counting, confidence, quality, exposure, dan anomaly tidak dikunci, semua itu hanya kosmetik di atas pasir basah.

Dokumen ini memastikan aplikasi tidak sekadar menghitung kendaraan, tetapi menghasilkan data yang layak dijual, layak dipahami client, dan layak dipertanggungjawabkan.

Prinsip akhirnya:

```text
Count carefully.
Label honestly.
Report transparently.
Never overclaim.
```
