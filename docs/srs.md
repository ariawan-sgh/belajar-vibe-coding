# Software Requirements Specification (SRS) — WebGIS

**Basis Dokumen:** `docs/prd.md` v1.0, `docs/domain-model.md`, `docs/erd.md`, `docs/api-spec.md`, `docs/ui-spec.md`, `docs/map-layer-spec.md`
**Versi:** 1.0
**Tanggal:** 2026-06-17
**Status:** Approved for Development
**Pemilik:** Product Management & Architecture Team

---

## 1. Introduction

### 1.1 Purpose

Dokumen Software Requirements Specification (SRS) ini mendefinisikan kebutuhan fungsional dan non-fungsional untuk aplikasi **WebGIS**. SRS ini merupakan turunan langsung dari Product Requirements Document (PRD) dan berfungsi sebagai acuan teknis untuk tim engineering (Frontend, Backend, DevOps, QA) dalam merancang, mengembangkan, dan menguji sistem.

### 1.2 Scope

Software WebGIS mencakup:
- **Frontend:** Single Page Application (SPA) menggunakan HTML5, Tailwind CSS, Vanilla JavaScript, OpenLayers, Lit JS, dan Vite.
- **Backend:** RESTful API menggunakan Golang Fiber.
- **Database:** PostgreSQL dengan ekstensi PostGIS.
- **Deployment:** Environment staging dan production dengan CI/CD pipeline.

### 1.3 Target Audience

- Software Engineers (Frontend & Backend)
- DevOps Engineers
- QA Engineers
- Technical Architects
- Data Engineers

### 1.4 Document Conventions

| Convention | Description |
|------------|-------------|
| **MUST** | Wajib diimplementasikan untuk kelayakan sistem. |
| **SHOULD** | Disarankan untuk kepatuhan standar. |
| **MAY** | Opsional. |
| **FR-XXX** | Functional Requirement identifier. |
| **NFR-XXX** | Non-Functional Requirement identifier. |
| **API-XXX** | API Endpoint identifier. |
| **UI-XXX** | UI Component identifier. |

---

## 2. System Overview

### 2.1 System Context

```
┌─────────────────────────────────────────────────────────────────┐
│                        USER INTERACTION                          │
│                    (GIS Officer / Admin)                         │
└───────────────────────────────┬─────────────────────────────────┘
                                │ HTTPS/REST
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    WebGIS Application                            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Frontend (HTML5 + Lit JS + OpenLayers + Vite)            │  │
│  │  - Map Rendering                                          │  │
│  │  - Layer Management                                       │  │
│  │  - Hover Interaction                                      │  │
│  │  - Navigation Controls                                    │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Backend (Golang Fiber)                                   │  │
│  │  - Vector API                                             │  │
│  │  - Metadata API                                           │  │
│  │  - Health Check                                           │  │
│  └──────────────────────────────────────────────────────────┘  │
└───────────────────────────────┬─────────────────────────────────┘
                                │ SQL/Spatial Query
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│              PostgreSQL + PostGIS Database                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │  admin_      │  │   infra_     │  │   layers     │         │
│  │  boundaries  │  │  structure   │  │              │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 System Boundaries

| Component | In Scope | Out of Scope |
|-----------|----------|--------------|
| **Frontend** | Map rendering, layer control, tooltip, navigation | Authentication UI (Sprint 2), data upload UI (Sprint 2) |
| **Backend** | Vector API, metadata API, health check | Authentication (Sprint 2), file upload (Sprint 2), spatial analysis (Sprint 3) |
| **Database** | PostgreSQL + PostGIS dengan schema inti | Auth tables (Sprint 2), import logs (Sprint 2) |

---

## 3. Functional Requirements

### 3.1 FR-01: Base Map Rendering

| ID | Requirement | Priority | Description |
|----|-------------|----------|-------------|
| FR-01.1 | Tile Layer | Must | Aplikasi MUST memuat dan menampilkan base map tile dari OpenStreetMap dengan URL template `https://{a-c}.tile.openstreetmap.org/{z}/{x}/{y}.png`. |
| FR-01.2 | CRS Handling | Must | Base map menggunakan EPSG:3857 untuk rendering. Semua koordinat dari API (EPSG:4326) MUST di-transform ke EPSG:3857 sebelum di-render. |
| FR-01.3 | Zoom Range | Must | Mendukung zoom level 3 hingga 18. |
| FR-01.4 | Tile Caching | Should | Memanfaatkan browser cache untuk tiles yang sudah pernah di-load. Service worker (Workbox) SHOULD diimplementasikan untuk offline capability. |
| FR-01.5 | Tile Fallback | Should | Jika primary tile provider gagal, sistem SHOULD mencoba fallback provider (OSM Germany atau CartoDB). |
| FR-01.6 | Attribution | Must | Menampilkan `© OpenStreetMap contributors` di pojok bawah peta. |

### 3.2 FR-02: Vector Layer Management

| ID | Requirement | Priority | Description |
|----|-------------|----------|-------------|
| FR-02.1 | Vector Layer — Admin Boundaries | Must | Memuat dan menampilkan layer batas administrasi Indonesia minimal level Provinsi dan Kabupaten. Styling: Province (stroke `#2563EB`, 2px), Regency (stroke `#3B82F6`, 1.5px). |
| FR-02.2 | Vector Layer — Infrastructure | Must | Memuat dan menampilkan layer infrastruktur: Jalan Nasional (stroke `#F59E0B`, 2px), Kereta Api (stroke `#8B5CF6`, 2px). |
| FR-02.3 | Layer Visibility Control | Must | Menyediakan kontrol checkbox untuk mengaktifkan/nonaktifkan setiap layer secara independen. Perubahan MUST terjadi dalam ≤ 1 detik. |
| FR-02.4 | Layer Ordering | Must | Urutan rendering: Base Map (z=0) → Admin Boundaries (z=1) → Infrastructure (z=2). |
| FR-02.5 | BBOX-based Loading | Must | Vector layer di-load secara incremental berdasarkan bounding box viewport saat `moveend` atau `zoomend`. |
| FR-02.6 | Geometry Simplification | Should | Backend SHOULD melakukan simplifikasi Douglas-Peucker berdasarkan zoom level untuk mengurangi vertex count pada low zoom. |

### 3.3 FR-03: Hover Interaction & Tooltip

| ID | Requirement | Priority | Description |
|----|-------------|----------|-------------|
| FR-03.1 | Mouse Move Detection | Must | Mendeteksi posisi mouse pointer di atas objek vektor menggunakan OpenLayers `map.forEachFeatureAtPixel()`. |
| FR-03.2 | Tooltip Rendering | Must | Menampilkan tooltip (Lit JS Web Component) berisi minimal 3 atribut: `name`, `code`, `area_km2` (admin) atau `name`, `type`, `length_km` (infra). |
| FR-03.3 | Tooltip Timing | Must | Tooltip muncul dalam ≤ 50ms dari mouse move. |
| FR-03.4 | Highlight Effect | Must | Feature yang di-hover menerapkan style highlight: stroke `#00FFFF`, width +1px, fill `rgba(0, 255, 255, 0.15)`. |
| FR-03.5 | Tooltip Dismissal | Must | Tooltip dan highlight dihapus dalam ≤ 100ms saat mouse meninggalkan objek. |
| FR-03.6 | Tooltip Positioning | Must | Tooltip diposisikan dekat kursor (offset 12px) dengan viewport flip logic agar tidak overflow. |
| FR-03.7 | Debounce | Must | Event `pointermove` di-debounce menggunakan `requestAnimationFrame` dengan interval maksimal 16ms. |

### 3.4 FR-04: Navigation Controls

| ID | Requirement | Priority | Description |
|----|-------------|----------|-------------|
| FR-04.1 | Zoom Control | Must | Tombol zoom in/out dengan increment 1 zoom level per klik. |
| FR-04.2 | Mouse Wheel Zoom | Must | Scroll zoom dengan momentum (kinetic decay 0.95, durasi 500ms). |
| FR-04.3 | Pan Control | Must | Drag/pan dengan klik dan seret mouse. |
| FR-04.4 | Double-click Zoom | Must | Double-click = zoom in pada titik klik. Shift+double-click = zoom out. |
| FR-04.5 | Fit-to-Extent | Must | Tombol "Reset View" menerapkan `view.fit(extent_indo, {padding: [50,50,50,50], duration: 500})`. Extent: `[95.3, -10.9, 141.0, 5.9]` (EPSG:4326). |
| FR-04.6 | Coordinate Display | Must | Menampilkan koordinat Lon/Lat secara real-time di bottom bar dalam format desimal (6 digits). |
| FR-04.7 | Keyboard Navigation | Should | Shortcut: `+`/`-` untuk zoom, `0` untuk reset view, `Esc` untuk tutup tooltip. |

### 3.5 FR-05: Backend Vector API

| ID | Requirement | Priority | Description |
|----|-------------|----------|-------------|
| FR-05.1 | Admin Boundaries Endpoint | Must | `GET /api/v1/vector/admin-boundaries` dengan parameters `bbox`, `zoom_level`, `level`. Response: GeoJSON FeatureCollection atau MVT. |
| FR-05.2 | Infrastructure Endpoint | Must | `GET /api/v1/vector/infrastructure` dengan parameters `bbox`, `zoom_level`, `type`. Response: GeoJSON FeatureCollection atau MVT. |
| FR-05.3 | Spatial Query | Must | Query menggunakan `ST_Intersects(geom, ST_MakeEnvelope(:xmin, :ymin, :xmax, :ymax, 4326))` dengan parameter binding. |
| FR-05.4 | Metadata Layers Endpoint | Must | `GET /api/v1/metadata/layers` mengembalikan array metadata layer (id, name, type, geom_type, srid, attribution, default_visibility, z_index, attributes). |
| FR-05.5 | Layer Attributes Endpoint | Must | `GET /api/v1/metadata/attributes/:layerId` mengembalikan definisi atribut untuk layer tertentu. |
| FR-05.6 | Health Check Endpoint | Must | `GET /health` mengembalikan status sistem (postgres, postgis, uptime). |
| FR-05.7 | Response Time | Must | P95 response time ≤ 200ms untuk vector endpoints. |
| FR-05.8 | Error Handling | Must | Semua error response mengikuti format standar: `{"error": {"code": "...", "message": "...", "details": {...}, "timestamp": "...", "request_id": "..."}}`. |

### 3.6 FR-06: UX Polish & Reliability

| ID | Requirement | Priority | Description |
|----|-------------|----------|-------------|
| FR-06.1 | Loading State | Must | Menampilkan spinner atau skeleton saat base map atau vector layer sedang di-load. |
| FR-06.2 | Error State | Must | Menampilkan toast notification jika API request gagal, dengan tombol "Retry". |
| FR-06.3 | Empty State | Should | Menampilkan pesan informatif jika query spasial mengembalikan 0 features. |
| FR-06.4 | Map Attribution | Must | Menampilkan attribution sesuai license untuk base map dan setiap vector layer. |

---

## 4. Non-Functional Requirements

### 4.1 Performance

| ID | Metric | Target | Measurement Method |
|----|--------|--------|-------------------|
| NFR-P01 | First Contentful Paint (FCP) | ≤ 2.0 detik | Lighthouse CI |
| NFR-P02 | Time to Interactive (TTI) | ≤ 4.0 detik | Lighthouse CI |
| NFR-P03 | Map Render FPS | ≥ 30 fps | Custom FPS meter |
| NFR-P04 | Tooltip Latency (p95) | ≤ 50ms | `performance.now()` measurement |
| NFR-P05 | API Response Time (p95) | ≤ 200ms | Backend access log / APM |
| NFR-P06 | End-to-End Query Time | ≤ 500ms | Backend tracing |

### 4.2 Scalability

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-S01 | Concurrent Users | 50 concurrent GIS Officer tanpa degradation. |
| NFR-S02 | Database Connections | Connection pool 20-50 connections. |
| NFR-S03 | Payload Size | Maksimal 5MB per GeoJSON response. |

### 4.3 Reliability

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-R01 | Uptime | 99.5% selama jam kerja (08:00-17:00 WIB, Senin-Jumat). |
| NFR-R02 | Data Backup | Harian backup PostgreSQL + continuous WAL archiving. Retention 30 hari. |
| NFR-R03 | Error Recovery | Recovery dalam ≤ 5 menit setelah failure. |

### 4.4 Security

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-SEC01 | Input Sanitization | Semua parameter query divalidasi dan di-sanitize. Menggunakan parameterized queries untuk PostGIS. |
| NFR-SEC02 | CORS Policy | Hanya mengizinkan origin yang terdaftar dalam environment variable. |
| NFR-SEC03 | Rate Limiting | Maksimal 100 requests/minute per IP (vector endpoints). |
| NFR-SEC04 | Data Exposure | Hanya data non-sensitive di-expose. Tidak ada PII. |
| NFR-SEC05 | HTTPS | Semua komunikasi menggunakan TLS 1.2+. |

### 4.5 Maintainability

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-M01 | Code Modularity | Komponen Lit JS maksimal 300 lines per file. |
| NFR-M02 | API Documentation | Semua endpoint terdokumentasi menggunakan OpenAPI 3.0. |
| NFR-M03 | Database Migrations | Menggunakan structured migration tool. |
| NFR-M04 | Logging | Structured JSON logging untuk backend. |

### 4.6 Portability

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-PORT01 | Containerization | Dapat dijalankan melalui Docker Compose untuk development. |
| NFR-PORT02 | Environment Parity | Environment variable untuk semua konfigurasi. |

### 4.7 Browser Compatibility

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-B01 | Supported Browsers | Chrome 120+, Edge 120+, Firefox 121+, Safari 17+. |
| NFR-B02 | No IE11 | Internet Explorer 11 tidak didukung. |

---

## 5. Interface Requirements

### 5.1 User Interface Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| UI-01 | Full-screen map layout dengan peta sebagai elemen utama (100% viewport). | Must |
| UI-02 | Layer Switcher di pojok kanan atas (top-right overlay). | Must |
| UI-03 | Zoom Control di pojok kanan bawah (di atas coordinate bar). | Must |
| UI-04 | Coordinate Bar di bagian bawah (bottom status bar), height 32px. | Must |
| UI-05 | Tooltip floating mengikuti mouse dengan offset 12px. | Must |
| UI-06 | Menggunakan Tailwind CSS utility classes. Font: Inter (sans), JetBrains Mono (mono). | Must |
| UI-07 | Dark theme default. Light mode opsional untuk Sprint 3. | Must |

### 5.2 Software Interfaces

#### 5.2.1 External: Base Map Tile Provider

| Item | Specification |
|------|--------------|
| Protocol | HTTP/2 |
| URL Template | `https://{a-c}.tile.openstreetmap.org/{z}/{x}/{y}.png` |
| Max Zoom | 18 |
| Attribution | Wajib ditampilkan |
| Fallback | OSM Germany: `https://tile.openstreetmap.de/{z}/{x}/{y}.png` |

#### 5.2.2 Internal: Backend API

| Item | Specification |
|------|--------------|
| Protocol | HTTPS (TLS 1.2+) |
| Base URL | `/api/v1/` |
| Content-Type | `application/json` (GeoJSON), `application/x-protobuf` (MVT) |
| CORS | Whitelist origin terdaftar |
| Compression | Gzip/Brotli untuk response > 1KB |

---

## 6. Data Requirements

### 6.1 Data Sources

| Dataset | Source | Format | CRS | Update Frequency |
|---------|--------|--------|-----|-------------------|
| Batas Administrasi | BPS / Kemendagri | Shapefile (.shp) | EPSG:4326 | Annual |
| Jalan Nasional | OSM / PUPR | GeoJSON / Shapefile | EPSG:4326 | Ad-hoc |
| Jalur Kereta Api | OSM / Kemenhub | GeoJSON / Shapefile | EPSG:4326 | Ad-hoc |

### 6.2 Database Requirements

- **PostgreSQL 15+** dengan **PostGIS 3.3+**
- Semua geometri disimpan dalam `GEOMETRY(..., 4326)`
- GIST index pada kolom `geom` untuk setiap tabel spasial
- B-Tree index pada kolom filter (`level`, `type`, `province_name`)
- CHECK constraints: `ST_SRID(geom) = 4326`, `ST_IsValid(geom)`
- Indonesian coordinate bounds validation: `lon 95.3-141.0`, `lat -10.9-5.9`

---

## 7. System Architecture

### 7.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              CLIENT (Browser)                            │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │                     WebGIS Application (SPA)                     │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │  │
│  │  │   UI Layer  │  │  Map Engine │  │   State Management       │  │  │
│  │  │  (Tailwind) │  │ (OpenLayers)│  │  (Lit Reactive)          │  │  │
│  │  └─────────────┘  └─────────────┘  └─────────────────────────┘  │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │  │
│  │  │Lit Components│ │ Vector Tiles│  │   HTTP Client            │  │  │
│  │  │ (Tooltip,   │ │  / Layers   │  │   (Fetch API)            │  │  │
│  │  │ Switcher)   │  │             │  │                          │  │  │
│  │  └─────────────┘  └─────────────┘  └─────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────────────────┘  │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │ HTTPS/REST
                                ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                              BACKEND (Golang Fiber)                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────────────┐  │
│  │   Router    │  │  Handlers   │  │      Services / Business Logic    │  │
│  │ (Fiber App) │  │ (HTTP, CORS)│  │ (PostGIS Query, Validation)       │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────────────┘  │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │ SQL/Spatial Query
                                ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        DATABASE (PostgreSQL + PostGIS)                   │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │         admin_boundaries, infrastructure, layers                  │   │
│  │         Spasial Index: GIST(geom), B-Tree(columns)                │   │
│  └──────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

### 7.2 Component Responsibilities

| Component | Responsibility | Technology |
|-----------|----------------|-----------|
| `<webgis-map>` | Init OpenLayers Map, manage view, emit map events | Lit JS + OpenLayers |
| `<webgis-layer-switcher>` | Toggle layer visibility | Lit JS |
| `<webgis-tooltip>` | Render floating tooltip with feature attributes | Lit JS |
| `<webgis-coordinate-bar>` | Display cursor coordinates real-time | Lit JS |
| `<webgis-zoom-control>` | Zoom in/out and reset view buttons | Lit JS |
| Vector Handler | Receive bbox + params, query PostGIS, return GeoJSON/MVT | Golang Fiber |
| PostGIS Service | Construct spatial queries with ST_Intersects, ST_Transform | PostgreSQL + PostGIS |

---

## 8. API Specifications Summary

### 8.1 Endpoint Overview

| API-ID | Method | Endpoint | Description | Auth | Rate Limit |
|--------|--------|----------|-------------|------|-----------|
| API-VEC-01 | GET | `/api/v1/vector/admin-boundaries` | Data batas administrasi | Public | 100 req/min |
| API-VEC-02 | GET | `/api/v1/vector/infrastructure` | Data infrastruktur | Public | 100 req/min |
| API-META-01 | GET | `/api/v1/metadata/layers` | Metadata semua layer | Public | 30 req/min |
| API-META-02 | GET | `/api/v1/metadata/attributes/:layerId` | Atribut detail layer | Public | 30 req/min |
| API-HEALTH-01 | GET | `/health` | Health check | Public | Unlimited |

### 8.2 Common Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `bbox` | string | Yes | `minx,miny,maxx,maxy` (EPSG:4326) |
| `zoom_level` | integer | Yes | Zoom level 3-18 |
| `level` | string | No | `province`, `regency`, `district`, `village` |
| `type` | string | No | `national_road`, `railway`, `airport`, `dam` |

### 8.3 Error Codes

| HTTP Status | Error Code | Description |
|-------------|------------|-------------|
| 400 | `INVALID_BBOX` | Format bbox tidak valid |
| 400 | `INVALID_ZOOM` | Zoom level di luar range 3-18 |
| 400 | `INVALID_LEVEL` | Level tidak dikenali |
| 404 | `LAYER_NOT_FOUND` | Layer ID tidak ditemukan |
| 422 | `GEOMETRY_INVALID` | Data spasial rusak (ST_IsValid gagal) |
| 429 | `RATE_LIMIT_EXCEEDED` | Too many requests |
| 500 | `DATABASE_ERROR` | Koneksi database atau query gagal |
| 500 | `INTERNAL_ERROR` | Kesalahan server |

---

## 9. Acceptance Criteria Summary

| ID | Acceptance Criteria | Test Method |
|----|---------------------|-------------|
| AC-01 | Peta Indonesia ter-render dalam ≤ 2 detik (FCP). Zoom level 3-18 berfungsi. | Automated: Lighthouse + Manual: visual inspection |
| AC-02 | Layer batas administrasi aktif dengan styling berbeda per level. Hover menampilkan tooltip: Nama, Kode, Luas (km²). | Manual: hover test |
| AC-03 | Layer infrastruktur aktif dengan warna berbeda per tipe. Hover menampilkan: Nama, Jenis, Kondisi, Panjang (km). | Manual: hover test |
| AC-04 | Tooltip muncul ≤ 50ms, hilang ≤ 100ms. Highlight effect diterapkan. | Automated: timer + DevTools Performance |
| AC-05 | Layer visibility berubah ≤ 1 detik tanpa reload halaman. | Manual: toggle test |
| AC-06 | Reset View menampilkan seluruh Indonesia dalam ≤ 1 detik. | Manual: click test |
| AC-07 | Zoom/pan tetap ≥ 30 fps dengan 5.000+ features. | Automated: FPS counter |
| AC-08 | Coordinate bar menampilkan Lon/Lat real-time dengan presisi 6 digit. | Manual: compare dengan reference point |
| AC-09 | API response GeoJSON valid, P95 ≤ 200ms. Geometri valid. | Automated: Postman/Newman + SQL check |
| AC-10 | Tidak ada console error pada Chrome 120+, Edge 120+, Firefox 121+, Safari 17+. | Manual: cross-browser testing |

---

## 10. Testing Requirements

### 10.1 Test Levels

| Level | Scope | Tools |
|-------|-------|-------|
| **Unit Test** | Frontend components, backend handlers, PostGIS queries | Vitest (frontend), testify (Go) |
| **Integration Test** | API endpoints, database queries | Postman/Newman, Go test |
| **E2E Test** | Full user journey: load map, hover, toggle layer, zoom | Playwright (optional) |
| **Performance Test** | FPS, API latency, FCP | Lighthouse, custom FPS meter, k6 |
| **Cross-browser Test** | Chrome, Edge, Firefox, Safari | Manual + BrowserStack |

### 10.2 Coverage Targets

| Component | Target Coverage |
|-----------|-----------------|
| Frontend (Lit JS components) | ≥ 70% |
| Backend (Handlers + Services) | ≥ 70% |
| PostGIS Queries | 100% (via integration test) |

---

## 11. Deployment Requirements

### 11.1 Environment Configuration

| Environment | URL | Purpose |
|-------------|-----|---------|
| Development | `http://localhost:5173` (frontend), `http://localhost:3000` (backend) | Local development |
| Staging | `https://staging.webgis.example.com` | UAT, QA testing |
| Production | `https://webgis.example.com` | Live user access |

### 11.2 CI/CD Pipeline

| Stage | Action |
|-------|--------|
| **Build** | `npm run build` (frontend), `go build` (backend) |
| **Lint** | `npm run lint` (frontend), `golangci-lint` (backend) |
| **Test** | `npm run test` (frontend), `go test ./...` (backend) |
| **Deploy Staging** | Auto-deploy ke staging pada merge ke `develop` |
| **Deploy Production** | Manual approval deploy ke `main` |

### 11.3 Infrastructure

| Component | Specification |
|-----------|--------------|
| **Frontend Hosting** | Vercel / Netlify / Nginx reverse proxy |
| **Backend Hosting** | Docker container di EC2 / Cloud Run / DO App Platform |
| **Database** | RDS PostgreSQL + PostGIS atau self-managed PostgreSQL |
| **Redis** | Opsional untuk caching API response dan session store |
| **CDN** | CloudFlare / CloudFront untuk static assets dan tiles |

---

## 12. Traceability Matrix

| PRD Section | SRS Section | Coverage |
|-------------|-------------|----------|
| Executive Summary | Section 1-2 | Covered |
| Business Goals (BG-01 s/d BG-05) | Section 2, 4.1-4.7 | Covered |
| Scope (In/Out) | Section 2.2 | Covered |
| User Roles | Section 2.3 | Covered |
| User Stories (US-01 s/d US-07) | Section 3.1-3.6 | Covered |
| Functional Requirements (FR-01 s/d FR-06) | Section 3 | Covered |
| Non-Functional Requirements | Section 4 | Covered |
| Business Workflow | Section 7 | Covered |
| Feature Breakdown | Section 7.2 | Covered |
| Acceptance Criteria | Section 9 | Covered |
| KPI & Success Metrics | Section 4.1-4.7 | Covered |
| Future Roadmap | Section 3 (Sprint 2-4) | Covered |

---

## 13. Glossary

| Term | Definition |
|------|------------|
| **GeoJSON** | Format JSON untuk encoding data spasial. |
| **MVT** | Mapbox Vector Tiles — format binary untuk vektor tiles. |
| **PostGIS** | Ekstensi spasial untuk PostgreSQL. |
| **GIST Index** | Generalized Search Tree index untuk query spasial cepat. |
| **ST_Intersects** | Fungsi PostGIS untuk mengecek kesinggungan geometri. |
| **ST_Transform** | Fungsi PostGIS untuk transformasi koordinat antar SRID. |
| **EPSG:4326** | WGS 84 — sistem koordinat geografis (degree). |
| **EPSG:3857** | Web Mercator — sistem koordinat untuk web mapping. |
| **Lit JS** | Web Component library untuk membuat custom elements. |
| **OpenLayers** | Library JavaScript untuk rendering peta interaktif. |
| **Tailwind CSS** | Utility-first CSS framework. |
| **Vite** | Build tool untuk frontend JavaScript. |
| **Golang Fiber** | Web framework untuk Golang. |
| **BBOX** | Bounding box — kotak pembatas area peta. |
| **Hit-test** | Proses deteksi feature spasial di bawah kursor mouse. |

---

## 14. Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| **Product Owner** | [Nama PO] | | 2026-06-17 |
| **Tech Lead** | [Nama TL] | | 2026-06-17 |
| **QA Lead** | [Nama QA] | | 2026-06-17 |
| **DevOps Lead** | [Nama DevOps] | | 2026-06-17 |

---

*Document Owner: Product Management & Architecture Team*
*Last Review: 2026-06-17*
*Next Review: 2026-07-01*
*Related Documents:* `docs/prd.md`, `docs/domain-model.md`, `docs/erd.md`, `docs/api-spec.md`, `docs/ui-spec.md`, `docs/map-layer-spec.md`, `docs/product-backlog.md`, `docs/sprint-backlog.md`, `docs/prd-gap-analysis.md`
