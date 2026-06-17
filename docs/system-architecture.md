# System Architecture — WebGIS

**Basis Dokumen:** `docs/prd.md` v1.0, `docs/product-backlog.md`  
**Versi Dokumen:** 1.0  
**Tanggal:** 2026-06-17  
**Pemilik:** Product Management Team & Engineering Lead  

---

## 1. High-Level Architecture Overview

WebGIS mengadopsi arsitektur **Client-Server monolitik terbagi (split monolith)** dengan pemisahan yang jelas antara frontend dan backend, dihubungkan melalui RESTful API. Sistem ini dirancang modular agar mudah di-scale dan di-extend di masa depan.

```
┌─────────────────────────────────────────────────────────────────┐
│                         Browser (Client)                        │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    WebGIS Application                      │  │
│  │  ┌─────────────┐  ┌──────────────┐  ┌─────────────────┐  │  │
│  │  │  Lit JS     │  │  OpenLayers  │  │  Tailwind CSS   │  │  │
│  │  │ Components  │  │    Map       │  │  (Styling)      │  │  │
│  │  └──────┬──────┘  └──────┬───────┘  └─────────────────┘  │  │
│  │         │                │                                │  │
│  │  ┌──────┴────────────────┴───────┐                      │  │
│  │  │      State Management Layer    │                      │  │
│  │  │   (Layer state, Feature state  │                      │  │
│  │  │    Tooltip state, UI state)    │                      │  │
│  │  └───────────────────────────────┘                      │  │
│  └───────────────────────────────────────────────────────────┘  │
└──────────────────────┬──────────────────────────────────────────┘
                       │ HTTPS / REST API (JSON/GeoJSON)
                       │
┌──────────────────────▼──────────────────────────────────────────┐
│                      Backend Server (Golang Fiber)               │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    API Layer                               │  │
│  │  ┌──────────────┐  ┌────────────┐  ┌───────────────────┐  │  │
│  │  │ Health       │  │ Metadata   │  │ Vector Services    │  │  │
│  │  │ Handler      │  │ Handler    │  │ (admin-boundaries, │  │  │
│  │  └──────────────┘  └────────────┘  │  infrastructure)   │  │  │
│  │                                     └───────────────────┘  │  │
│  ├───────────────────────────────────────────────────────────┤  │
│  │              Service Layer                                 │  │
│  │  ┌──────────────────┐  ┌─────────────────────────────┐   │  │
│  │  │ PostGIS Service  │  │ Tile Service (opsional MVT) │   │  │
│  │  │ (spatial query,  │  │                             │   │  │
│  │  │  bbox filtering) │  │                             │   │  │
│  │  └──────────────────┘  └─────────────────────────────┘   │  │
│  ├───────────────────────────────────────────────────────────┤  │
│  │              Middleware Layer                              │  │
│  │  CORS • Rate Limiting • Logging • Error Handling          │  │
│  └───────────────────────────────────────────────────────────┘  │
└──────────────────────┬──────────────────────────────────────────┘
                       │ PostgreSQL Driver (pgx / GORM)
                       │
┌──────────────────────▼──────────────────────────────────────────┐
│                 PostgreSQL + PostGIS Database                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Tables:                                                  │  │
│  │  • layers (metadata)                                      │  │
│  │  • admin_boundaries (MultiPolygon)                        │  │
│  │  • infrastructure (LineString)                            │  │
│  │                                                           │  │
│  │  Spatial Index: GIST on geom columns                      │  │
│  │  B-Tree Index: level, type, province_name                 │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Frontend Architecture

### 2.1 Technology Stack

| Layer | Teknologi | Versi / Catatan |
|----|-----------|-----------------|
| Runtime | Vite | Build tool & dev server |
| UI Framework | Lit JS (Web Components) | Standar Web Components, shadow DOM |
| Map Engine | OpenLayers | v8+, proyeksi EPSG:3857 |
| Styling | Tailwind CSS | Utility-first CSS via PostCSS |
| Language | TypeScript | Mode strict enabled |
| Package Manager | npm / pnpm | package.json |

### 2.2 Komponen Frontend

Aplikasi dibangun menggunakan **Web Components berbasis Lit JS**. Setiap komponen berdiri sendiri (self-contained) dengan shadow DOM untuk isolasi styling.

```
<webgis-app>                              # Root application shell
├── <webgis-map>                          # Container OpenLayers Map instance
│   ├── <webgis-layer-switcher>           # Panel kontrol layer visibility
│   ├── <webgis-tooltip>                  # Tooltip floating (atribut feature)
│   ├── <webgis-coordinate-bar>           # Bottom bar: koordinat + nama layer
│   └── <webgis-zoom-control>             # Tombol zoom + reset view
│
└── (Layers — managed oleh WebGISMap)
    ├── BaseMapLayer (OSM Tile)
    ├── AdminBoundaryLayer (Vector)
    └── InfrastructureLayer (Vector)
```

**Kontrak Komponen:**

| Komponen | Props / Attributes | Events | Deskripsi |
|----------|-------------------|--------|-----------|
| `<webgis-map>` | `center`, `zoom`, `layers` | `featurehover`, `featureout`, `layertoggle` | Root map. Mengelola instance OpenLayers, layer lifecycle, dan event bus. |
| `<webgis-layer-switcher>` | `layers-config` | `layer:toggle` | Menampilkan daftar layer dengan checkbox enable/disable. |
| `<webgis-tooltip>` | `feature`, `position`, `visible` | `dismiss` | Tooltip dinamis yang menampilkan atribut feature. |
| `<webgis-coordinate-bar>` | `coordinate`, `active-layer` | — | Bottom bar untuk koordinat lon/lat. |
| `<webgis-zoom-control>` | `zoom-level`, `extent` | `zoom:in`, `zoom:out`, `view:reset` | Kontrol zoom dan tombol reset extent. |

### 2.3 Struktur File Frontend

```
src/
├── index.html                    # Entry HTML, mount <webgis-app>
├── src/
│   ├── main.ts                   # Vite entry point
│   ├── app.ts                    # <webgis-app> root component
│   │
│   ├── components/
│   │   ├── WebGISMap.ts          # <webgis-map> — OpenLayers orchestration
│   │   ├── LayerSwitcher.ts      # <webgis-layer-switcher>
│   │   ├── Tooltip.ts            # <webgis-tooltip>
│   │   ├── CoordinateBar.ts      # <webgis-coordinate-bar>
│   │   └── ZoomControl.ts        # <webgis-zoom-control>
│   │
│   ├── layers/
│   │   ├── BaseMapLayer.ts       # OSM Tile Layer wrapper
│   │   ├── AdminBoundaryLayer.ts # Vector Layer — batas admin
│   │   └── InfrastructureLayer.ts # Vector Layer — infrastruktur
│   │
│   ├── styles/
│   │   └── main.css              # Tailwind directives + custom overrides
│   │
│   └── utils/
│       ├── map-utils.ts          # Coordinate transform (4326 ↔ 3857)
│       ├── api.ts                # Fetch wrapper, error handling
│       └── debounce.ts           # Debounce utility untuk tooltip
│
├── public/
│   └── favicon.ico
│
├── tailwind.config.js            # Konfigurasi Tailwind (dark mode, dll)
├── postcss.config.js             # PostCSS config untuk Tailwind
├── tsconfig.json                 # TypeScript strict mode
└── package.json                  # Dependencies & scripts
```

### 2.4 State Management

Frontend menggunakan **state management sederhana berbasis Lit JS reactive properties** tanpa library eksternal.

**State Terbagi menjadi:**

1. **Map State** — dikelola di dalam `<webgis-map>`:
   - `center: [number, number]` — koordinat pusat peta (EPSG:3857)
   - `zoom: number` — level zoom saat ini
   - `activeLayers: string[]` — daftar layer yang aktif
   - `hoveredFeature: Feature | null` — feature yang sedang di-hover

2. **UI State** — dikelola di komponen induk:
   - `tooltipVisible, tooltipPosition, tooltipContent`
   - `coordinateDisplay`
   - `isLoading, error`

**Data Flow (Unidirectional):**

```
User Action (hover, zoom, toggle)
    │
    ▼
Event Handler (di dalam komponen)
    │
    ▼
Update State (Lit reactive property)
    │
    ▼
Render (Lit render() → shadow DOM update)
    │
    ▼
OpenLayers API call (jika perlu, e.g., setLayerVisibility)
```

---

## 3. Backend Architecture

### 3.1 Technology Stack

| Layer | Teknologi | Versi / Catatan |
|----|-----------|-----------------|
| Runtime | Golang Fiber | v2+, HTTP framework cepat & ringan |
| Database Driver | pgx | PostgreSQL driver native |
| Spatial Query | raw SQL + PostGIS functions | Tanpa ORM untuk performa optimal |
| Config | Viper | Environment-based config |
| Logging | Zap (uber-go/zap) | Structured logging |
| API Docs | Swagger (opsional) | OpenAPI spec generation |

### 3.2 Arsitektur Backend

Backend mengadopsi pola **Clean Architecture** sederhana dengan pemisahan Concern:

```
┌───────────────────────────────────────────────────────────────┐
│                     HTTP Layer (Fiber)                         │
│                  Route Registration + Middleware                │
└───────────────────────────┬───────────────────────────────────┘
                            │
┌───────────────────────────▼───────────────────────────────────┐
│                      Handler Layer                             │
│  • vector_handler.go      → parse req, validate params        │
│  • metadata_handler.go    → serve layer metadata              │
│  • health_handler.go      → health check                      │
└───────────────────────────┬───────────────────────────────────┘
                            │
┌───────────────────────────▼───────────────────────────────────┐
│                     Service Layer                              │
│  • postgis_service.go     → build spatial query               │
│                             (ST_Intersects, bbox, zoom filter) │
│  • tile_service.go        → MVT generation (opsional)         │
└───────────────────────────┬───────────────────────────────────┘
                            │
┌───────────────────────────▼───────────────────────────────────┐
│                     Repository / SQL Layer                     │
│  • Parameterized queries                                      │
│  • ST_Transform, ST_Intersects, ST_AsGeoJSON                  │
│  • Connection pool management                                  │
└───────────────────────────┬───────────────────────────────────┘
                            │
┌───────────────────────────▼───────────────────────────────────┐
│                  PostgreSQL + PostGIS                           │
└───────────────────────────────────────────────────────────────┘
```

### 3.3 Struktur File Backend

```
server/
├── main.go                    # Fiber app init, middleware, routes
├── handlers/
│   ├── vector_handler.go      # GET /api/v1/vector/*
│   ├── metadata_handler.go    # GET /api/v1/metadata/*
│   └── health_handler.go      # GET /health
├── services/
│   ├── postgis_service.go     # Spatial query builder
│   └── tile_service.go        # MVT tile generation (Sprint 2+)
├── models/
│   ├── layer.go               # Layer metadata struct
│   └── feature.go             # Feature response struct
├── middleware/
│   ├── cors.go                # CORS policy
│   ├── rate_limit.go          # Rate limiting (Sprint 2)
│   └── logger.go              # Structured logging
├── config/
│   └── config.go              # Environment config
├── go.mod
└── go.sum
```

### 3.4 API Endpoints

| Method | Endpoint | Deskripsi | Response Format |
|--------|----------|-----------|----------------|
| GET | `/api/v1/vector/admin-boundaries` | Ambil batas administrasi | GeoJSON FeatureCollection |
| GET | `/api/v1/vector/infrastructure` | Ambil data infrastruktur | GeoJSON FeatureCollection |
| GET | `/api/v1/metadata/layers` | Daftar layer tersedia | JSON array |
| GET | `/api/v1/metadata/attributes/:layerId` | Daftar atribut layer | JSON array |
| GET | `/health` | Health check | JSON `{ "status": "ok" }` |

**Query Parameters untuk Vector Endpoints:**

| Parameter | Tipe | Deskripsi | Wajib |
|-----------|------|-----------|-------|
| `bbox` | string | Bounding box (`minLon,minLat,maxLon,maxLat`) | Ya |
| `zoom_level` | integer | Level zoom (3–18) | Ya |
| `level` | string | Filter level admin (province/regency) | Tidak |
| `type` | string | Filter tipe infrastruktur | Tidak |

---

## 4. Database Architecture

### 4.1 Schema Overview

Database menggunakan **PostgreSQL dengan ekstensi PostGIS** untuk mendukung tipe data geometri dan operasi spasial.

```
PostgreSQL + PostGIS
├── Extension: postgis
├── Table: layers (metadata layer)
├── Table: admin_boundaries (batas administrasi)
│   └── Index: GIST (geom), B-Tree (level, province_name)
└── Table: infrastructure (infrastruktur)
    └── Index: GIST (geom), B-Tree (type)
```

### 4.2 Tabel dan Relasi

```sql
-- Metadata Layer
layers
├── id (PK, SERIAL)
├── name, description, type, geom_type
├── srid (default: 4326)
├── attribution, is_public
└── timestamps

admin_boundaries
├── id (PK, SERIAL)
├── layer_id → layers.id (FK)
├── level (province / regency)
├── code, name, parent_id (hierarchical)
├── province_name, regency_name, district_name
├── population, area_km2
├── geom (MultiPolygon, SRID 4326)
└── timestamps

infrastructure
├── id (PK, SERIAL)
├── layer_id → layers.id (FK)
├── type, name, code, condition
├── length_km
├── geom (LineString, SRID 4326)
└── timestamps
```

### 4.3 Indeks Spasial

Indeks sangat penting untuk performa query spasial pada dataset besar:

```sql
-- Spasial index (GIST) — WAJIB ada untuk query performan
CREATE INDEX idx_admin_boundaries_geom ON admin_boundaries USING GIST (geom);
CREATE INDEX idx_infrastructure_geom ON infrastructure USING GIST (geom);

-- B-Tree index untuk filter non-spasial
CREATE INDEX idx_admin_boundaries_level ON admin_boundaries (level);
CREATE INDEX idx_admin_boundaries_province ON admin_boundaries (province_name);
CREATE INDEX idx_infrastructure_type ON infrastructure (type);
```

### 4.4 Query Spasial Contoh

Query untuk mengambil batas administrasi berdasarkan bounding box:

```sql
SELECT 
    id, name, level, code, province_name, area_km2,
    ST_AsGeoJSON(ST_Transform(geom, 4326)) as geom
FROM admin_boundaries
WHERE 
    level = $1
    AND ST_Intersects(
        ST_Transform(geom, 4326),
        ST_MakeEnvelope($2, $3, $4, $5, 4326)
    );
```

**Catatan Penting:**
- Selalu lakukan `ST_Transform(geom, 4326)` saat serialisasi ke GeoJSON agar koordinat konsisten.
- Tanpa GIST index, query `ST_Intersects` pada dataset besar (> 10.000 features) dapat mencapai > 5 detik.
- Query di parameterize (prepared statement) untuk mencegah SQL injection.

---

## 5. Data Flow & Interaction Patterns

### 5.1 Initial Load Flow

```
Browser
    │
    ▼ (1) Load index.html + Vite bundle
    │
    ▼ (2) <webgis-map> init OpenLayers Map instance
    │   - Set view: center Indonesia, zoom 5, CRS EPSG:3857
    │   - Add BaseMapLayer (OSM tiles)
    │
    ▼ (3) Fetch layer metadata: GET /api/v1/metadata/layers
    │
    ▼ (4) Untuk setiap layer aktif default:
    │   - Fetch vector data: GET /api/v1/vector/{layer}?bbox=...&zoom_level=...
    │   - Parse GeoJSON response
    │   - Add Vector Layer ke OpenLayers Map
    │
    ▼ (5) Terapkan styling berdasarkan layer type
    │
    ▼ (6) Siap —-map interaktif
```

### 5.2 Hover Interaction Flow

```
User hover di atas feature vektor
    │
    ▼ (1) OpenLayers: pointermove event
    │   - Ambil pixel position
    │   - map.forEachFeatureAtPixel(pixel, hitTestFn)
    │
    ▼ (2) Hit-test return feature + layer
    │   - Jika ada feature:
    │     a. Terapkan highlight style (stroke warna, ketebalan)
    │     b. Extract atribut dari feature properties
    │     c. Kirim event 'featurehover' ke <webgis-tooltip>
    │   - Jika tidak ada:
    │     a. Hapus highlight dari feature sebelumnya
    │     b. Sembunyikan tooltip
    │
    ▼ (3) <webgis-tooltip> update:
    │   - Set position (dekat kursor, dengan viewport flip logic)
    │   - Render atribut (nama, kode, tipe)
    │
    ▼ (4) Mouse keluar dari feature:
    │   - Reset style feature ke default
    │   - Sembunyikan tooltip
```

### 5.3 Layer Toggle Flow

```
User klik checkbox di <webgis-layer-switcher>
    │
    ▼ (1) Event 'layer:toggle' dipancarkan
    │
    ▼ (2) <webgis-map> handler:
    │   - Update state: activeLayers[]
    │   - map.getLayers().forEach(layer => {
    │       layer.setVisible(layerId in activeLayers)
    │     })
    │
    ▼ (3) OpenLayers re-render hanya layer yang berubah
    │   - Tanpa reload halaman
    │   - Tanpa refetch data dari API
```

---

## 6. Security Architecture

### 6.1 CORS Policy

Backend menerapkan CORS strict policy:

| Aspek | Konfigurasi |
|-------|-------------|
| Allowed Origins | Hanya domain terdaftar (misal: `https://gis.internal.company.com`) |
| Allowed Methods | `GET`, `OPTIONS` |
| Allowed Headers | `Content-Type`, `Authorization` |
| Max Age | 86400 detik (1 hari) |

### 6.2 Input Sanitization

Semua parameter query dari frontend divalidasi dan disanitasi:

| Vector Query | Sanitization |
|--------------|--------------|
| `bbox` | Validasi format (4 angka, rangeValid) |
| `zoom_level` | Validasi range (3–18), integer only |
| `level`, `type` | Whitelist enum validation |
| `layerId` (metadata) | UUID / integer validation |

**SQL Injection Prevention:**
- Menggunakan **parameterized queries** (prepared statements) untuk semua query PostGIS.
- Tidak ada string concatenation atau template literal untuk query SQL.

### 6.3 Data Exposure Policy

| Layer | Akses | Catatan |
|-------|-------|---------|
| Base Map | Public | OpenStreetMap tiles, lisensi ODbL |
| Admin Boundaries | Public | Data spasial non-sensitive |
| Infrastructure | Public | Data spasial non-sensitive |
| Metadata | Public | Informasi layer schema |

Tidak ada PII (Personally Identifiable Information) yang di-expose melalui API public.

### 6.4 Keamanan Jangka Panjang (Sprint 2+)

| Fitur | Deskripsi |
|-------|-----------|
| JWT Authentication | Token-based auth untuk akses aplikasi |
| RBAC | Role `GISOfficer` (read-only) dan `SystemAdministrator` (full access) |
| Rate Limiting | 100 req/min per pengguna/IP |

---

## 7. Deployment Architecture

### 7.1 Environment

| Environment | URL (Contoh) | Purpose |
|-------------|--------------|---------|
| Development | `http://localhost:5173` | Lokal dev (Vite dev server) |
| Staging | `https://staging-gis.internal.company.com` | QA & UAT |
| Production | `https://gis.internal.company.com` | Live production |

### 7.2 Deployment Stack (Rekomendasi)

```
┌───────────────────────────────────────────────────────────────────┐
│                        Load Balancer / Reverse Proxy              │
│                         (Nginx / Cloudflare)                      │
└──────────────────────────┬────────────────────────────────────────┘
                           │
              ┌────────────┴────────────┐
              │                         │
       ┌──────▼──────┐           ┌──────▼──────┐
       │   Frontend  │           │   Backend   │
       │  (Static:   │           │  (Golang     │
       │   Nginx /   │           │   Fiber)     │
       │   CDN)      │           │              │
       └─────────────┘           └──────┬───────┘
                                         │
                                ┌────────▼────────┐
                                │   PostgreSQL    │
                                │   + PostGIS     │
                                │   (RDS / self-  │
                                │    hosted)      │
                                └─────────────────┘
```

### 7.3 CI/CD Pipeline

Pipeline otomatis melalui GitHub Actions / GitLab CI:

```
Code Push → Lint (ESLint + golangci-lint) → Unit Test → Build → Deploy
                                                              │
                                                    ┌─────────▼──────────┐
                                                    │  Staging Deploy    │
                                                    │  (auto)            │
                                                    └─────────┬──────────┘
                                                              │
                                                    Manual Approval by PO
                                                              │
                                                    ┌─────────▼──────────┐
                                                    │  Production Deploy │
                                                    │  (manual trigger)  │
                                                    └────────────────────┘
```

---

## 8. Performance Architecture

### 8.1 Frontend Performance

| Area | Strategi |
|------|----------|
| Map Rendering | OpenLayers WebGL renderer (Sprint 2+) |
| Vector Features | Clipping & filtering berdasarkan zoom level (LOD) |
| Tile Caching | Browser cache + Service Worker |
| Tooltip | Debounced 16ms (~1 frame) |
| Bundle Size | Code splitting, lazy load komponen |

### 8.2 Backend Performance

| Area | Strategi |
|------|----------|
| Spatial Query | GIST index + bbox pre-filtering |
| Query Param | Parameterized prepared statements |
| Connection Pool | pgx connection pool (max 20 connections) |
| Response Format | GeoJSON untuk < 10k features, MVT untuk > 10k |
| Caching (Sprint 3+) | Redis cache untuk metadata & popular queries |

### 8.3 Database Performance

| Area | Strategi |
|------|----------|
| Index | GIST pada semua kolom `geom` |
| Query Plan | `EXPLAIN ANALYZE` rutin untuk monitoring |
| Partitioning (Future) | Partition tabel besar berdasarkan `level` atau `type` |

---

## 9. Monitoring & Observability

### 9.1 Monitoring Targets

| Tier | Target | Tool (Rekomendasi) |
|------|--------|-------------------|
| Uptime | 99.5% (jam kerja) | UptimeRobot / Pingdom |
| API Response | p95 ≤ 200ms | Prometheus + Grafana |
| Database | Query p95 ≤ 100ms, connection health | pg_stat_statements + Grafana |
| Frontend | FCP ≤ 2s, FPS ≥ 30 | Lighthouse CI + custom FPS monitor |
| Error Rate | ≤ 5 critical bugs per sprint | Sentry / GitHub Issues |

### 9.2 Logging Strategy

```
[Timestamp] [Level] [Component] [Message] [Context]
2026-06-17T09:30:00Z INFO  vector_handler GET /api/v1/vector/admin-boundaries bbox=95.0,10.0,140.0,6.0 duration=45ms
2026-06-17T09:30:01Z ERROR postgis_service Query failed: ST_Intersects timeout duration=5000ms
```

---

## 10. Technology Decision Rationale

| Keputusan | Opsi yang Ditinjau | Alasan Pemilihan |
|-----------|--------------------|--------------------|
| Frontend Map Engine | OpenLayers vs Leaflet vs Mapbox GL JS | OpenLayers: performa tinggi, kompatibilitas EPSG:3857, support MVT, zero-cost license. |
| Frontend UI Framework | Lit JS vs React vs Vue | Lit JS: Web Components standard, ringan, shadow DOM isolation, tidak lock-in framework besar. |
| Backend Framework | Fiber vs Gin vs Echo | Fiber: performa terbaik di benchmark, API mirip Express.js, mudah dipelajari. |
| Database | PostgreSQL+PostGIS vs MongoDB vs MySQL | PostGIS: standar industri untuk spasial, query kompleks, indexing GIST, ekosistem mature. |
| Build Tool | Vite vs Webpack vs esbuild | Vite: HMR cepat, tree-shaking, output optima, konfigurasi minimal. |
| Styling | Tailwind CSS vs CSS Modules vs styled-components | Tailwind: konsistensi UI, utility-first, dark mode support, bundle size kecil. |
| Geo Format | GeoJSON vs MVT | GeoJSON untuk Sprint 1 (< 10k features), MVT untuk Sprint 3+ (dataset besar). |

---

## 11. Constraints & Assumptions

### 11.1 Constraints

- Aplikasi hanya mendukung desktop (viewport minimal 1280px) pada Sprint 1.
- User hanya GIS Officer dengan akses read-only pada Sprint 1.
- Sistem Administrator dan fitur CRUD hanya tersedia mulai Sprint 2.
- Data awal di-seed manual via SQL/sql file (bukan UI upload).

### 11.2 Assumptions

- Browser target mendukung Web Components (Chrome 120+, Edge 120+, Firefox 121+, Safari 17+).
- Dataset awal batas administrasi dan infrastruktur tersedia dalam format SHP/GeoJSON.
- PostgreSQL + PostGIS ter-install dan di-maintain oleh tim ops/infra.
- Koneksi internet stabil untuk loading base map tiles (OSM).
- Tidak ada requirement real-time editing kolaboratif pada Sprint 1.

---

## 12. Architecture Evolution Roadmap

| Sprint | Arsitektur Change | Alasan |
|-------|-------------------|--------|
| Sprint 1 | Baseline: Split monolith (frontend Vite + backend Fiber + PostGIS) | MVP, core functionality |
| Sprint 2 | Tambah: JWT auth, rate limiting, structured logging | Security & reliability |
| Sprint 3 | Tambah: MVT tile generation, Redis caching | Performa dataset besar |
| Sprint 4 | Tambah: External API integration, webhooks | Data integration |
| Beyond | Opsi: Microservices, message queue, 3D engine | Scale & advanced features |

---

## 13. Risk & Mitigasi

| Risk | Impact | Probability | Mitigasi |
|------|--------|------------|---------|
| Performa menurun saat > 10k features | High | Medium | Implementasi MVT tiles (Mapbox Vector Tiles) pada Sprint 3. |
| Query PostGIS lambat tanpa index | High | Low | Enforce GIST index creation + query monitoring. |
| Browser compatibility issue (Safari) | Medium | Low | Test cross-browser, gunakan polyfills jika perlu. |
| Vendor lock-in pada base map provider | Low | Low | Abstraksi BaseMapLayer untuk mudah ganti provider. |
| Data import yang tidak valid geometri | Medium | Medium | Validasi `ST_IsValid` + `ST_SRID` otomatis pada backend. |

---

*Document Owner: Engineering Lead & Product Management Team*  
*Last Review: 2026-06-17*  
*Next Review: 2026-07-01 (Sprint 1 Retrospective)*  
*Distribution: Engineering Team, GIS Officer Representative, Stakeholders*
