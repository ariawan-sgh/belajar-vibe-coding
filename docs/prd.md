# Product Requirements Document (PRD) — WebGIS

## 1. Executive Summary

WebGIS adalah aplikasi webGIS berbasis web yang menampilkan peta interaktif wilayah Indonesia berserta atribut detail setiap objek peta saat mouse hover. Aplikasi ini dikhususkan untuk **GIS Officer** sebagai pengguna utama, memungkinkan identifikasi dan eksplorasi spasial cepat, akurat, dan efisien dalam satu Portal terpadu.

Aplikasi dibangun dengan arsitektur modern:
- **Frontend:** HTML5, Tailwind CSS, Vanilla JavaScript, OpenLayers, Lit JS, Vite.
- **Backend:** Golang Fiber.
- **Database:** PostgreSQL dengan ekstensi PostGIS untuk penyimpanan dan query data spasial.

Dengan fondasi data spasial yang terstruktur, antarmuka responsif, dan performa tinggi, WebGIS menjadi basis digital untuk kebutuhan pemetaan dan analisis spasial di organisasi.

---

## 2. Business Goals

| ID | Business Goal | Description | Priority |
|----|---------------|-------------|----------|
| BG-01 | Akses Terpadu ke Peta Indonesia | Menyediakan satu Portal terpusat bagi GIS Officer untuk mengakses peta digital lengkap wilayah Indonesia. | High |
| BG-02 | Identifikasi Atribut Spasial Cepat | Meningkatkan efisiensi identifikasi objek peta melalui interaksi mouse hover yang real-time tanpa perlu klik atau navigasi menu tambahan. | High |
| BG-03 | Fondasi Data Spasial Terstruktur | Menyediakan struktur data spasial yang terstandardisasi dan terindeks (PostgreSQL + PostGIS) sebagai basis pengambilan keputusan berbasis lokasi. | High |
| BG-04 | Performa Rendering Peta Tinggi | Menjamin pengalaman pengguna tetap responsif dengan frame rate minimal 30fps meskipun menampilkan ribuan vektor features. | Medium |
| BG-05 | Ekstensibilitas Jangka Panjang | Membangun arsitektur modular yang memudahkan penambahan layer, integrasi data eksternal, atau fitur analisis lanjutan tanpa rewrite besar. | Medium |

---

## 3. Scope

### In Scope (Sprint 1)

| Modul | Description |
|-------|-------------|
| Base Map | Tampilan peta dasar Indonesia menggunakan OpenLayers dengan tile provider (OpenStreetMap / custom tile server). |
| Administrative Boundary Layer | Layer batas administrasi Indonesia minimal level Provinsi dan Kabupaten/Kota. |
| Infrastructure Layer | Layer infrastruktur publik primer minimal Jalan Nasional dan Jalur Kereta Api. |
| Hover Interaction | Tooltip dinamis dan highlight effect yang muncul saat mouse hover menampilkan atribut objek. |
| Layer Switcher | Kontrol untuk mengaktifkan/nonaktifkan setiap layer peta secara independen. |
| Navigation Controls | Zoom in/out, pan, dan fit-to-extent untuk wilayah Indonesia. |
| Coordinate Display | Menampilkan koordinat (longitude & latitude) posisi kursor secara real-time. |
| Responsive UI | Tampilan optimal pada desktop dengan viewport minimal 1280px. |

### Out of Scope (Sprint 1)

| Item | Reason |
|------|--------|
| Input/edit data spasial (CRUD geometry) | Membutuhkan sistem autentikasi, validasi geometry, dan manajemen hak akses. |
| Analisis spasial terdepan (buffer, overlay, spatial join) | Bergantung pada kematangan dataset dan kebutuhan spesifik. |
| Export data (Shapefile, GeoJSON, KML) | Membutuhkan library server-side processing atau client-side transformer. |
| Multi-user collaboration (comments, annotations) | Fitur opsional untuk kebutuhan tim yang belum terdefinisi. |
| Mobile-first design | Fokus awal adalah desktop untuk GIS Officer. |
| Print/export layout peta (PDF, image) | Fitur opsional untuk kebutuhan presentasi. |

---

## 4. User Roles

### 4.1 GIS Officer (Primary User)

**Deskripsi:** Pengguna utama aplikasi yang bertugas melakukan identifikasi, eksplorasi, dan analisis awal terhadap data spasial pada peta.

**Karakteristik:**
- Memiliki pemahaman dasar tentang Sistem Informasi Geografis (SIG).
- Membutuhkan akses cepat ke informasi atribut spasial tanpa navigasi menu yang rumit.
- Menggunakan desktop workstation dengan monitor resolusi minimal 1366×768.

**Akses & Permissions:**
- Read-only access ke semua layer peta yang tersedia.
- Tidak memiliki hak untuk mengubah atau menghapus data spasial.

### 4.2 System Administrator (Secondary User — Out of Scope Sprint 1)

**Deskripsi:** Pengguna yang bertanggung jawab atas maintenance, konfigurasi, dan update data spasial dalam sistem.

**Karakteristik:**
- Memiliki hak akses penuh untuk mengelola layer, data spasial, dan konfigurasi sistem.
- Bertugas mengupload data baru, memvalidasi integritas data, dan memastikan ketersediaan sistem.

**Akses & Permissions:**
- Write access ke database (PostgreSQL + PostGIS).
- Akses ke panel administratif untuk manajemen layer dan metadata.
- **Catatan:** Fungsionalitas untuk System Administrator tidak termasuk dalam scope Sprint 1. Direncanakan pada Roadmap berikutnya.

---

## 5. User Stories

### Epic: Peta Interaktif

| ID | User Story | Acceptance Criteria |
|----|-----------|---------------------|
| US-01 | Sebagai GIS Officer, saya ingin melihat peta interaktif seluruh wilayah Indonesia, agar saya dapat menavigasi dan memahami konteks spasial dari data yang tersedia. | Peta Indonesia ter-render dalam ≤ 2 detik dengan base map dan semua layer aktif sesuai konfigurasi default. |
| US-02 | Sebagai GIS Officer, saya ingin melihat atribut detail dari setiap objek peta saat saya mengarahkan kursor (mouse hover) ke objek tersebut, agar saya bisa mendapatkan informasi cepat tanpa perlu klik. | Tooltip muncul dalam ≤ 50ms dengan minimal 3 atribut relevan (nama, kode, tipe/info lain). |
| US-03 | Sebagai GIS Officer, saya ingin dapat mengaktifkan atau menonaktifkan setiap layer peta secara independen, agar saya dapat menyesuaikan tampilan peta sesuai kebutuhan analisis saya. | Checkbox layer switcher berfungsi, layer visibility berubah dalam ≤ 1 detik tanpa reload halaman. |
| US-04 | Sebagai GIS Officer, saya ingin dapat melakukan zoom in, zoom out, dan menggeser peta (pan), agar saya dapat fokus pada area spasial tertentu yang ingin saya analisis. | Zoom dan pan berfungsi lancar dengan scroll wheel, tombol kontrol, dan drag pada peta. |
| US-05 | Sebagai GIS Officer, saya ingin dapat memulihkan tampilan peta kembali ke cakupan penuh Indonesia, agar saya dapat kembali ke pandangan awal dengan cepat. | Tombol "Reset View" menampilkan seluruh Indonesia dalam ≤ 1 detik. |
| US-06 | Sebagai GIS Officer, saya ingin objek yang sedang saya hover memiliki indikasi visual yang jelas (highlight), agar saya tahu objek mana yang sedang aktif. | Feature yang di-hover menerapkan highlight style (stroke warna berbeda, ketebalan bertambah, atau fill opacity naik). |
| US-07 | Sebagai GIS Officer, saya ingin melihat koordinat (longitude & latitude) dari posisi kursor saya di peta, agar saya dapat mencatat posisi spasial dengan akurat. | Bottom bar menampilkan koordinat real-time dalam format desimal (6 digits precision) saat mouse bergerak di atas peta. |

---

## 6. Functional Requirements

### 6.1 FR-01: Base Map Rendering

| ID | Requirement | Detail |
|----|------------|--------|
| FR-01.1 | Tile Layer | Aplikasi memuat dan menampilkan base map tile dari provider yang dapat dikonfigurasi (default: OpenStreetMap atau custom tile server). |
| FR-01.2 | Coordinate Reference System | Base map menggunakan proyeksi EPSG:3857 (Web Mercator) untuk kompatibilitas dengan OpenLayers, dengan dukungan transformasi koordinat ke EPSG:4326 (WGS84). |
| FR-01.3 | Multi-resolution Tiles | Mendukung loading tiles dengan level of detail zoom level 3 hingga zoom level 18. |
| FR-01.4 | Tile Caching | Memanfaatkan browser cache dan service worker (opsional) untuk mempercepat loading tiles yang sudah pernah diakses. |

### 6.2 FR-02: Vector Layer Management

| ID | Requirement | Detail |
|----|------------|--------|
| FR-02.1 | Vector Layer — Administrative Boundary | Memuat dan menampilkan layer batas administrasi Indonesia minimal level Provinsi dan Kabupaten/Kota, dengan styling berbeda per level. |
| FR-02.2 | Vector Layer — Infrastruktur | Memuat dan menampilkan layer infrastruktur publik primer minimal Jalan Nasional dan Jalur Kereta Api. |
| FR-02.3 | Layer Visibility Control | Menyediakan kontrol checkbox atau toggle untuk mengaktifkan/menonaktifkan setiap layer secara independen. |
| FR-02.4 | Layer Ordering | Urutan rendering: base map → administrative boundary → infrastruktur. |

### 6.3 FR-03: Hover Interaction & Tooltip

| ID | Requirement | Detail |
|----|------------|--------|
| FR-03.1 | Mouse Move Detection | Mendeteksi posisi mouse pointer di atas objek vektor dan mengidentifikasi feature yang sedang di-hover. |
| FR-03.2 | Feature Hit-Test | Melakukan spatial hit-test menggunakan OpenLayers `map.forEachFeatureAtPixel()` untuk menentukan feature yang sedang di-hover. |
| FR-03.3 | Tooltip Rendering | Menampilkan tooltip (menggunakan Lit JS Web Component) berisi atribut feature pada posisi dekat kursor. |
| FR-03.4 | Highlight Effect | Feature yang di-hover menerapkan style highlight (stroke berbeda warna, ketebalan bertambah, atau fill opacity naik). |
| FR-03.5 | Tooltip Dismissal | Tooltip dan highlight dihapus saat mouse meninggalkan objek peta (mouse out). |
| FR-03.6 | Tooltip Positioning | Tooltip diposisikan sedekat mungkin dengan kursor tanpa melampaui viewport browser. Otomatis menyesuaikan posisi jika terlalu dekat tepi viewport. |

### 6.4 FR-04: Navigation Controls

| ID | Requirement | Detail |
|----|------------|--------|
| FR-04.1 | Zoom Control | Menyediakan kontrol zoom (+ / -) yang dapat diklik. |
| FR-04.2 | Mouse Wheel Zoom | Mendukung zoom menggunakan scroll wheel pada mouse. |
| FR-04.3 | Pan Control | Mendukung drag/pan menggunakan klik dan seret mouse pada peta. |
| FR-04.4 | Fit-to-Extent Button | Menyediakan tombol "Reset View" atau "Fit Indonesia" untuk memulihkan tampilan peta ke cakupan penuh Indonesia. |
| FR-04.5 | Coordinate Display | Control bar menampilkan koordinat longitude dan latitude posisi kursor secara real-time. |

### 6.5 FR-05: Responsive & Performance

| ID | Requirement | Detail |
|----|------------|--------|
| FR-05.1 | Viewport Minimum | Konten ditampilkan secara optimal pada viewport minimal 1280px width. |
| FR-05.2 | Render Performance | Menjaga frame rate minimum 30 fps saat menggeser atau zoom peta dengan 5.000+ features aktif. |
| FR-05.3 | Debounced Tooltip | Pembaruan tooltip pada mouse move di-debounce (interval maksimal 16ms, ~60fps) untuk mencegah rendering berlebihan. |

---

## 7. Non Functional Requirements

### 7.1 Performance

| ID | Requirement | Target |
|----|------------|--------|
| NFR-01 | First Contentful Paint (FCP) | ≤ 2 detik pada koneksi 4G stabil. |
| NFR-02 | Time to Interactive (TTI) | ≤ 4 detik pada koneksi 4G stabil. |
| NFR-03 | Map Render FPS | Minimum 30 fps saat pan/zoom dengan 5.000+ features. |
| NFR-04 | Tooltip Response Latency | ≤ 50ms dari mouse move ke tooltip muncul. |
| NFR-05 | Backend API Response Time | ≤ 200ms untuk query atribut spasial (p95). |

### 7.2 Reliability & Availability

| ID | Requirement | Target |
|----|------------|--------|
| NFR-06 | Uptime | 99.5% selama jam kerja (08:00–17:00 WIB, Senin–Jumat). |
| NFR-07 | Database Backup | Harian automated backup PostgreSQL + PostGIS dengan retention 30 hari. |

### 7.3 Security

| ID | Requirement | Target |
|----|------------|--------|
| NFR-08 | Data Exposure | Hanya data spasial non-sensitive di-expose melalui API public. Tidak ada PII pada response API. |
| NFR-09 | CORS Configuration | Backend mengimplementasikan CORS policy yang hanya mengizinkan origin domain yang terdaftar. |
| NFR-10 | Input Sanitization | Semua parameter query dari frontend disanitasi pada backend. PostGIS query menggunakan parameterized queries. |

### 7.4 Maintainability & Extensibility

| ID | Requirement | Target |
|----|------------|--------|
| NFR-11 | Code Organization | Frontend menggunakan struktur berbasis komponen dengan file modular. |
| NFR-12 | CI/CD Pipeline | Deployment melalui pipeline otomatis yang menjalankan lint, build, dan tests sebelum deploy. |
| NFR-13 | Documentation | Semua API endpoint terdokumentasi dalam file docs/api.md atau menggunakan OpenAPI/Swagger. |

### 7.5 Browser Compatibility

| ID | Requirement | Target |
|----|------------|--------|
| NFR-14 | Supported Browsers | Google Chrome 120+, Microsoft Edge 120+, Mozilla Firefox 121+, Safari 17+. |
| NFR-15 | No IE11 Support | Internet Explorer 11 tidak didukung. |

---

## 8. Business Workflow

### Flow 1: Identifikasi dan Eksplorasi Objek Peta

```
1. GIS Officer membuka aplikasi WebGIS di browser.
   ↓
2. Sistem memuat base map Indonesia (OpenStreetMap tiles) + vector layers default.
   ↓
3. GIS Officer melihat peta lengkap dengan layer switcher, zoom controls, dan coordinate bar.
   ↓
4. GIS Officer menavigasi peta (scroll zoom, drag pan, klik tombol kontrol).
   ↓
5. GIS Officer mengarahkan kursor ke objek peta.
   ↓
6. Sistem mendeteksi feature di bawah kursor (hit-test), menampilkan tooltip dengan atribut,
   dan memberikan highlight visual pada feature.
   ↓
7. GIS Officer membaca atribut dari tooltip (nama, kode, tipe, info lain).
   ↓
8. (Opsional) GIS Officer menonaktifkan/mengaktifkan layer via layer switcher.
   ↓
9. Proses selesai — GIS Officer menutup session atau memfokuskan area lain.
```

### Flow 2: Data Management (System Administrator)

```
1. SysAdmin mendefinisikan dataset spasial baru (format: GeoJSON, Shapefile, atau tabular).
   ↓
2. Import data ke PostgreSQL + PostGIS menggunakan shp2pgsql / ogr2ogr / COPY.
   ↓
3. Validasi integritas geometri (ST_IsValid, ST_SRID = 4326).
   ↓
4. Backend menyediakan endpoint API (GeoJSON atau MVT) dengan query PostGIS teroptimasi.
   ↓
5. Frontend consuming API — fetch data pada map interaction dan render sebagai Vector Layer.
```

---

## 9. Feature Breakdown

### 9.1 Frontend Components

| Komponen | Teknologi | Deskripsi |
|----------|----------|-----------|
| `<webgis-map>` | Lit JS + OpenLayers | Root component. Menginisialisasi instance OpenLayers Map, mengelola layer, dan event bus untuk interaksi. |
| `<webgis-layer-switcher>` | Lit JS | Panel kontrol overlay untuk mengaktifkan/nonaktifkan setiap layer. |
| `<webgis-tooltip>` | Lit JS | Tooltip floating yang menampilkan atribut feature. Diposisikan secara dinamis dengan flip logic. |
| `<webgis-coordinate-bar>` | Lit JS | Bottom bar yang menampilkan koordinat (lon/lat) dan nama layer aktif. |
| `<webgis-zoom-control>` | Lit JS | Tombol zoom in/out dan tombol fit-to-extent. |

### 9.2 Frontend File Structure

```
src/
├── index.html
├── src/
│   ├── main.ts                 # Vite entry, mount komponen
│   ├── components/
│   │   ├── WebGISMap.ts        # Root map component (Lit JS + OpenLayers)
│   │   ├── LayerSwitcher.ts    # Layer switcher control
│   │   ├── Tooltip.ts          # Tooltip component
│   │   ├── CoordinateBar.ts    # Coordinate display bar
│   │   └── ZoomControl.ts      # Zoom buttons
│   ├── layers/
│   │   ├── BaseMapLayer.ts     # OSM tile layer wrapper
│   │   ├── AdminBoundaryLayer.ts
│   │   └── InfrastructureLayer.ts
│   ├── styles/
│   │   └── main.css            # Tailwind directives + custom CSS
│   └── utils/
│   │   ├── map-utils.ts        # Coordinate transform helpers
│   │   └── api.ts              # Fetch wrapper dengan error handling
├── tailwind.config.js
├── postcss.config.js
├── tsconfig.json
└── package.json
```

### 9.3 Backend API Endpoints (Golang Fiber)

| Method | Endpoint | Deskripsi |
|--------|----------|-----------|
| GET | `/api/v1/vector/admin-boundaries` | Mengembalikan feature batas administrasi. Parameter: `bbox`, `zoom_level`, `level`. |
| GET | `/api/v1/vector/infrastructure` | Mengembalikan feature infrastruktur. Parameter: `bbox`, `zoom_level`, `type`. |
| GET | `/api/v1/metadata/layers` | Metadata layer yang tersedia (nama, tipe, SRS, atribut). |
| GET | `/api/v1/metadata/attributes/:layerId` | Daftar atribut beserta tipe data dan deskripsi untuk suatu layer. |
| GET | `/health` | Health check endpoint. |

### 9.4 Backend File Structure (Golang Fiber)

```
server/
├── main.go                    # Fiber app initialization, middleware, route registration
├── handlers/
│   ├── vector_handler.go      # Handler untuk vektor layer endpoints
│   ├── metadata_handler.go    # Handler untuk metadata endpoints
│   └── health_handler.go      # Health check
├── services/
│   ├── postgis_service.go     # Business logic: query PostGIS, format response
│   └── tile_service.go        # Tile generation (MVT) bila diperlukan
├── models/
│   ├── layer.go
│   └── feature.go
├── middleware/
│   └── cors.go
├── config/
│   └── config.go
├── go.mod
└── go.sum
```

### 9.5 Database Schema (PostgreSQL + PostGIS)

```sql
-- Enable PostGIS extension
CREATE EXTENSION IF NOT EXISTS postgis;

-- Tabel metadata layer
CREATE TABLE layers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    type VARCHAR(50) NOT NULL,
    geom_type VARCHAR(50) NOT NULL,
    srid INTEGER NOT NULL DEFAULT 4326,
    attribution TEXT,
    is_public BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Tabel batas administrasi
CREATE TABLE admin_boundaries (
    id SERIAL PRIMARY KEY,
    layer_id INTEGER REFERENCES layers(id),
    level VARCHAR(50) NOT NULL,
    code VARCHAR(50),
    name VARCHAR(255) NOT NULL,
    parent_id INTEGER REFERENCES admin_boundaries(id),
    province_name VARCHAR(255),
    regency_name VARCHAR(255),
    district_name VARCHAR(255),
    population BIGINT,
    area_km2 NUMERIC,
    geom GEOMETRY(MultiPolygon, 4326) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Spasial index
CREATE INDEX idx_admin_boundaries_geom ON admin_boundaries USING GIST (geom);
CREATE INDEX idx_admin_boundaries_level ON admin_boundaries (level);
CREATE INDEX idx_admin_boundaries_province ON admin_boundaries (province_name);

-- Tabel infrastruktur
CREATE TABLE infrastructure (
    id SERIAL PRIMARY KEY,
    layer_id INTEGER REFERENCES layers(id),
    type VARCHAR(100) NOT NULL,
    name VARCHAR(255),
    code VARCHAR(50),
    condition VARCHAR(50),
    length_km NUMERIC,
    geom GEOMETRY(LineString, 4326) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Spasial index
CREATE INDEX idx_infrastructure_geom ON infrastructure USING GIST (geom);
CREATE INDEX idx_infrastructure_type ON infrastructure (type);
```

---

## 10. Acceptance Criteria

| ID | Acceptance Criteria | Test Method |
|----|---------------------|-------------|
| AC-01 | Ketika WebGIS di-load di browser (Chrome 120+, koneksi 4G), peta Indonesia terlihat dalam ≤ 2 detik (FCP). Tile ter-render tanpa artefak visual. Zoom level 3–18 berfungsi tanpa error. | Automated: Lighthouse audit + Manual: visual inspection pada berbagai zoom level. |
| AC-02 | Ketika layer "Batas Administrasi" aktif, batas provinsi dan kabupaten/kota terlihat jelas dengan styling berbeda per level. Hover ke batas menampilkan tooltip: Nama Provinsi, Kode, Luas (km²). | Manual: aktifkan layer, hover ke beberapa provinsi, verifikasi tooltip. |
| AC-03 | Ketika layer "Infrastruktur" aktif, Jalan Nasional dan Jalur Kereta Api terlihat sebagai line features dengan warna berbeda. Hover menampilkan: Nama, Jenis, Kondisi, Panjang (km). | Manual: aktifkan layer, hover ke feature, verifikasi tooltip. |
| AC-04 | Tooltip muncul dalam ≤ 50ms saat mouse diarahkan ke objek vektor, menampilkan minimal 3 atribut. Tooltip hilang dalam ≤ 100ms saat mouse meninggalkan objek. Highlight effect diterapkan. | Automated: custom performance timer + Manual: tes dengan DevTools Performance tab. |
| AC-05 | Saat checkbox layer di-uncheck, feature layer hilang dari peta dalam ≤ 1 detik. Saat dicek kembali, feature muncul tanpa reload halaman. | Manual: toggle layer on/off multiple kali, measure waktu dengan stopwatch atau console timer. |
| AC-06 | Setelah klik tombol "Reset View", peta menampilkan seluruh Indonesia dalam ≤ 1 detik. Tidak ada spatial reference mismatch atau feature yang hilang. | Manual: klik reset, verify extent dan visibility semua layer aktif. |
| AC-07 | Dengan 5.000 vector features aktif, aksi zoom dan pan tetap ≥ 30 fps. Tidak ada jerky animation. Tooltip tidak memperlambat rendering. | Automated: custom FPS counter + Manual: DevTools Performance profiling. |
| AC-08 | Saat mouse bergerak di atas peta, koordinat di bottom bar ter-update real-time dalam format desimal (6 digits). Konsisten dengan posisi mouse (maksimal ± 0.001°). | Manual: compare koordinat dengan known reference point (misal: Monas: -6.175392, 106.827153). |
| AC-09 | GET `/api/v1/vector/admin-boundaries?level=province&bbox=...` mengembalikan JSON valid (GeoJSON FeatureCollection atau MVT binary) dengan ≤ 200ms response time (p95). Semua geometri valid. | Automated: Postman/Newman test + SQL: `SELECT id, ST_IsValid(geom) FROM admin_boundaries WHERE NOT ST_IsValid(geom)`. |
| AC-10 | Aplikasi berfungsi tanpa console error pada Chrome 120+, Edge 120+, Firefox 121+, dan Safari 17+. | Manual: cross-browser testing pada masing-masing browser. |

---

## 11. KPI dan Success Metrics

### 11.1 Product KPIs

| KPI | Metric | Target (Bulan 1) | Target (Kuartal 1) | Pengukuran |
|-----|--------|-------------------|--------------------|-----------|
| KPI-01 | Page Load Time (FCP) | ≤ 2 detik | ≤ 1.5 detik | Web Vitals (Lighthouse / Chrome User Experience Report) |
| KPI-02 | Map Render FPS | ≥ 30 fps | ≥ 45 fps | OpenLayers frame monitor / custom performance logger |
| KPI-03 | Tooltip Response P95 | ≤ 50ms | ≤ 30ms | Custom client-side timer |
| KPI-04 | API Response P95 | ≤ 200ms | ≤ 100ms | Backend access log / APM |
| KPI-05 | Uptime | 99% | 99.5% | Monitoring server / UptimeRobot |

### 11.2 Business Impact KPIs

| KPI | Metric | Target (Kuartal 1) | Pengukuran |
|-----|--------|--------------------|-----------|
| BI-01 | Task Completion Rate (GIS lookup) | ≥ 95% task berhasil diselesaikan dalam ≤ 3 menit | User interview + usage analytics |
| BI-02 | Layer Activation Rate | ≥ 80% GIS Officer mengaktifkan ≥ 2 layer dalam satu session | Analytics event: layer_toggle |
| BI-03 | Bug Rate | ≤ 5 open bugs kritikal per sprint | GitHub Issues / Project board |

### 11.3 Adoption KPIs

| KPI | Metric | Target (Kuartal 1) | Pengukuran |
|-----|--------|--------------------|-----------|
| AD-01 | Active Users (Weekly) | ≥ 10 GIS Officer | Auth log / analytics |
| AD-02 | Session Duration | Rata-rata ≥ 5 menit per session | Analytics session timer |
| AD-03 | Return Rate | ≥ 60% pengguna kembali dalam 7 hari | Cohort analysis |

---

## 12. Future Roadmap

### Q1 2026 — Sprint 1: Core (Current PRD Scope)

- [x] Peta dasar Indonesia (base map + vector layers).
- [x] Hover tooltip interaktif dengan highlight.
- [x] Layer switcher, zoom/pan controls, coordinate display.
- [x] Backend API + PostgreSQL + PostGIS dasar.
- [x] CI/CD, testing, dan dokumentasi API.

**Milestone:** Aplikasi siap digunakan untuk eksplorasi spasial oleh GIS Officer di lingkungan internal.

---

### Q2 2026 — Sprint 2: Edit & Data Management

- [ ] Authentication & Authorization (JWT, RBAC).
- [ ] CRUD endpoint untuk mengupload dan mengelola data spasial.
- [ ] UI manajemen layer untuk System Administrator.
- [ ] Multi-layer support — menambahkan layer Sungai, Danau, Gunung, TPA, Kawasan Konservasi.
- [ ] Geometry validation otomatis (ST_IsValid, auto-fix ringan).

**Milestone:** GIS Officer dapat mengimpor, mengelola, dan memperbarui data spasial tanpa bantuan tim engineering.

---

### Q3 2026 — Sprint 3: Advanced Interaction & Analysis

- [ ] Spatial Analysis Tools: buffer, spatial join, nearest neighbor.
- [ ] Drawing & Annotation tools.
- [ ] Measure tools (jarak, luas area).
- [ ] Print/export layout (PDF peta).
- [ ] Dark/Light theme toggle.
- [ ] Share view (URL dengan state peta).

**Milestone:** WebGIS dapat digunakan untuk analisis spasial mendasar dan produksi output presentasi.

---

### Q4 2026 — Sprint 4: Data Integration & Platform Expansion

- [ ] Integrasi REST API eksternal (BPS, BMKG, OpenData Indonesia).
- [ ] API Webhook untuk notifikasi perubahan data.
- [ ] Mobile-responsive untuk tablet GIS Officer di lapangan.
- [ ] Export data (GeoJSON, Shapefile, CSV, KML).
- [ ] Full-text search pada atribut spasial.
- [ ] Time-series layer support.

**Milestone:** WebGIS berkembang menjadi platform GIS terpadu dengan integrasi data otomatis dan dukungan mobile.

---

### Beyond Q4 2026 — Long-term Vision

- [ ] Machine Learning integration untuk prediksi spasial.
- [ ] 3D terrain visualization (CesiumJS integration).
- [ ] Collaborative features: real-time annotations, shared map sessions.
- [ ] Plugin ecosystem untuk custom analysis tools.
- [ ] Integration dengan drone/satellite imagery processing pipeline.

---

## Appendix

### Appendix A: Referensi Teknis

| Komponen | Referensi |
|---------|-----------|
| OpenLayers | https://openlayers.org/docs |
| Lit JS | https://lit.dev/docs |
| Tailwind CSS | https://tailwindcss.com/docs |
| Vite | https://vitejs.dev/guide |
| Golang Fiber | https://docs.gofiber.io |
| PostgreSQL + PostGIS | https://postgis.net/documentation |

### Appendix B: Catatan untuk Development Team

1. **Proyeksi Koordinat:** Selalu transformasi EPSG:4326 → EPSG:3857 di sisi backend (`ST_Transform(geom, 3857)`) sebelum diserialisasi ke GeoJSON.
2. **Spatial Index:** Wajib GIST index pada kolom `geom`.Tanpa index, query `ST_Intersects` pada dataset besar dapat mencapai > 5 detik.
3. **Debouncing Tooltip:** Interval maksimal 16ms (~1 frame). Hindari debounce > 100ms.
4. **MVT vs GeoJSON:** Untuk data > 10.000 features per layer, gunakan Mapbox Vector Tiles (MVT) untuk mengurangi ukuran payload.
5. **Validasi Geometri:** Jalankan `SELECT id, ST_IsValid(geom) FROM <table> WHERE NOT ST_IsValid(geom)` secara berkala.

---

*Document Owner: Product Management Team*
*Last Review: 2026-06-17*
*Next Review: 2026-07-01*
*Distribution: Engineering Lead, GIS Officer Representative, Stakeholders*
