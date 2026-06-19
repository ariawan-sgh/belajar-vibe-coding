# Product Requirements Document (PRD)

## WebGIS Analisis Sebaran Toko dan Pesaing

### Version

1.0

### Status

Draft

### Product Owner

Business Development Team

### Target Release

MVP v1

---

# 1. Executive Summary

WebGIS Analisis Sebaran Toko dan Pesaing adalah platform berbasis web yang membantu perusahaan retail, FMCG, distributor, franchise, dan UMKM menganalisis persebaran outlet milik sendiri maupun kompetitor menggunakan peta interaktif dan analisis spasial.

Sistem memungkinkan pengguna mengidentifikasi area yang sudah jenuh, area potensial untuk ekspansi, overlap antar toko, market coverage, serta peluang pembukaan lokasi baru berdasarkan data geografis dan demografis.

---

# 2. Problem Statement

Perusahaan sering mengalami:

* Sulit mengetahui persebaran outlet secara visual.
* Sulit mengukur jarak antar toko dan kompetitor.
* Pembukaan outlet baru berdasarkan intuisi.
* Tidak memiliki data coverage area.
* Tidak dapat mengidentifikasi wilayah yang belum terlayani.
* Sulit memantau ekspansi kompetitor.

Akibatnya:

* Kanibalisasi antar outlet.
* Penjualan tidak optimal.
* Investasi lokasi tidak efektif.
* Kehilangan peluang pasar baru.

---

# 3. Product Vision

Menjadi platform analisis lokasi berbasis GIS yang membantu perusahaan mengambil keputusan ekspansi bisnis secara berbasis data.

---

# 4. Goals

### Business Goals

* Mengurangi risiko pembukaan outlet baru.
* Meningkatkan akurasi pemilihan lokasi.
* Mempermudah analisis kompetitor.
* Menjadi platform intelligence untuk ekspansi bisnis.

### User Goals

* Melihat persebaran outlet secara real-time.
* Mengetahui area yang belum terjangkau.
* Membandingkan posisi toko dengan kompetitor.
* Menentukan lokasi potensial.

---

# 5. User Personas

## Area Manager

Tanggung Jawab:

* Mengelola outlet wilayah.

Kebutuhan:

* Melihat distribusi outlet.
* Analisis performa wilayah.

---

## Business Development

Tanggung Jawab:

* Membuka outlet baru.

Kebutuhan:

* Analisis lokasi potensial.
* Identifikasi kompetitor.

---

## Direksi

Tanggung Jawab:

* Pengambilan keputusan strategis.

Kebutuhan:

* Dashboard nasional.
* Laporan ekspansi.

---

## Franchise Owner

Kebutuhan:

* Menentukan lokasi franchise baru.
* Analisis pasar lokal.

---

# 6. Success Metrics

### Adoption

* 100 pengguna aktif bulanan.

### Efficiency

* Waktu analisis lokasi turun 70%.

### Accuracy

* 80% lokasi yang dipilih memenuhi target penjualan.

### Retention

* 60% pengguna aktif setiap bulan.

---

# 7. Scope MVP

## In Scope

### Manajemen Outlet

* Tambah outlet
* Edit outlet
* Hapus outlet
* Import CSV
* Bulk upload

### Peta Interaktif

* Zoom
* Pan
* Marker outlet
* Marker kompetitor

### Layer Management

* Outlet sendiri
* Kompetitor
* Area administrasi
* Kecamatan
* Kelurahan

### Analisis Radius

* Radius 500m
* Radius 1km
* Radius 3km
* Radius custom

### Heatmap

* Sebaran outlet
* Sebaran kompetitor

### Coverage Analysis

* Area terlayani
* Area tidak terlayani

### Dashboard

* Total outlet
* Total kompetitor
* Coverage area
* Wilayah potensial

### Reporting

* PDF Export
* Excel Export

---

# 8. Out of Scope (MVP)

* Mobile application
* AI recommendation
* Traffic prediction
* Satellite image processing
* Real-time vehicle tracking
* IoT integration

---

# 9. Functional Requirements

## FR-001 Authentication

### Description

Pengguna harus login untuk mengakses sistem.

### Acceptance Criteria

* Login berhasil.
* Logout berhasil.
* Reset password tersedia.

---

## FR-002 User Management

### Description

Admin dapat mengelola pengguna.

### Acceptance Criteria

* Create user
* Update user
* Delete user
* Assign role

---

## FR-003 Outlet Management

### Description

Mengelola data outlet perusahaan.

### Data

* Outlet ID
* Nama Outlet
* Alamat
* Latitude
* Longitude
* Tipe Outlet
* Status

### Acceptance Criteria

* CRUD outlet.
* Import CSV.

---

## FR-004 Competitor Management

### Description

Mengelola data kompetitor.

### Data

* Nama Toko
* Brand
* Latitude
* Longitude
* Kategori

### Acceptance Criteria

* CRUD kompetitor.
* Import data.

---

## FR-005 GIS Map Visualization

### Description

Menampilkan outlet pada peta.

### Acceptance Criteria

* Marker tampil.
* Popup informasi.
* Layer aktif/nonaktif.

---

## FR-006 Radius Analysis

### Description

Menghitung area jangkauan outlet.

### Acceptance Criteria

* Radius tampil.
* Overlap area terlihat.
* Dapat mengubah radius.

---

## FR-007 Coverage Analysis

### Description

Mengukur wilayah yang terlayani.

### Output

* Persentase coverage.
* Luas coverage.

---

## FR-008 Competitor Analysis

### Description

Membandingkan outlet dan pesaing.

### Output

* Jarak outlet ke kompetitor.
* Jumlah kompetitor dalam radius.

---

## FR-009 Heatmap Analysis

### Description

Visualisasi kepadatan outlet.

### Acceptance Criteria

* Heatmap aktif.
* Intensitas berdasarkan jumlah titik.

---

## FR-010 Potential Area Detection

### Description

Menampilkan wilayah yang belum memiliki outlet.

### Output

* Daftar area potensial.
* Ranking area.

---

## FR-011 Dashboard Analytics

### Widget

* Total outlet
* Outlet aktif
* Outlet per wilayah
* Kompetitor per wilayah
* Coverage %

---

## FR-012 Reporting

### Format

* PDF
* Excel

### Isi

* Peta
* Statistik
* Analisis

---

# 10. Non Functional Requirements

## Performance

* Peta terbuka < 3 detik.
* Dashboard < 2 detik.
* Query spasial < 5 detik.

---

## Availability

* Uptime 99%.

---

## Security

* JWT Authentication.
* HTTPS.
* Role Based Access Control.

---

## Scalability

* Minimal 100.000 titik lokasi.
* Minimal 1.000 pengguna aktif.

---

# 11. GIS Analysis Module

## Spatial Query

* Within Radius
* Intersect
* Buffer
* Nearest Neighbor

## Spatial Statistics

* Density Analysis
* Coverage Analysis
* Clustering

## Market Analysis

* White Spot Analysis
* Competitive Density
* Saturation Analysis

---

# 12. Data Sources

## Internal

* Data outlet perusahaan
* Data penjualan

## External

* Data administrasi wilayah
* OpenStreetMap
* Data kependudukan
* Data POI

---

# 13. User Flow

## Analisis Lokasi Baru

1. Login
2. Buka peta
3. Pilih wilayah
4. Aktifkan layer kompetitor
5. Analisis radius
6. Analisis coverage
7. Lihat rekomendasi area
8. Export laporan

---

# 14. Technical Architecture

## Frontend

* Solid JS
* TypeScript
* OpenLayers
* Tailwind 5

## Backend

* Golang
* Go Fiber

## Database

* PostgreSQL
* PostGIS

---

# 15. Role Matrix

| Feature         | Admin | Manager | Analyst |
| --------------- | ----- | ------- | ------- |
| User Management | Yes   | No      | No      |
| Outlet CRUD     | Yes   | Yes     | No      |
| Competitor CRUD | Yes   | Yes     | No      |
| Analysis        | Yes   | Yes     | Yes     |
| Export          | Yes   | Yes     | Yes     |

---

# 16. Risks

| Risk                          | Impact | Mitigation                 |
| ----------------------------- | ------ | -------------------------- |
| Data tidak akurat             | Tinggi | Validasi lokasi            |
| Data kompetitor tidak lengkap | Sedang | Integrasi sumber eksternal |
| Query GIS lambat              | Tinggi | Spatial indexing           |
| Data duplikat                 | Sedang | Data cleansing             |

---

# 17. Future Roadmap

## Phase 2

* AI Location Recommendation
* Forecast Revenue
* Demographic Analysis
* Mobile Application

## Phase 3

* Machine Learning Site Selection
* Real-time Competitor Monitoring
* Market Expansion Simulation
* Predictive Analytics

---

# Definition of Success

Pengguna dapat menentukan lokasi outlet baru berdasarkan data spasial, persebaran kompetitor, dan cakupan pasar dengan waktu analisis kurang dari 15 menit dan tingkat akurasi keputusan yang lebih tinggi dibanding proses manual.

Jika tujuan Anda adalah membangun produk yang benar-benar siap dijadikan SaaS komersial, langkah berikutnya adalah membuat **PRD v2 Enterprise** yang menambahkan modul:

* White Spot Analysis
* Demographic Intelligence
* Traffic & Mobility Analysis
* AI Site Recommendation
* Catchment Area Analysis (5, 10, 15 menit)
* Market Potential Scoring
* Multi-tenant SaaS Architecture

Karena fitur-fitur tersebut biasanya menjadi pembeda utama antara WebGIS biasa dan platform location intelligence kelas enterprise.