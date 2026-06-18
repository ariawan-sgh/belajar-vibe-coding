# Module Breakdown — WebGIS

**Basis Dokumen:** `docs/prd.md`, `docs/srs.md`, `docs/domain-model.md`, `docs/erd.md`, `docs/system-architecture.md`  
**Database:** PostgreSQL 15+ dengan PostGIS 3.3+  
**Tanggal:** 2026-06-18  
**Status:** Draft implementation breakdown

---

## 1. Prioritas

| Prioritas | Fase | Keterangan |
|---|---|---|
| `P0` | Sprint 1 | Core WebGIS: peta, layer, hover, API vector, metadata, health. |
| `P1` | Sprint 2 | Data Management & Access Control: auth, RBAC, upload, import validation. |
| `P2` | Sprint 3 | Analysis & Presentation: measure, draw, buffer, export, share view. |
| `P3` | Sprint 4 | Integration & Scale: external API sync, search, export data, mobile, time-series. |
| `TECH` | Enabler | Infrastruktur teknis yang diperlukan sebelum atau selama implementasi fitur. |

---

## 2. Tabel Modul Implementasi

| Modul | Responsibility | Dependency | API | Database Table | Priority | Complexity |
|---|---|---|---|---|---|---|
| Frontend Build Configuration | Setup Vite, TypeScript strict mode, Tailwind CSS, PostCSS, lint/test scripts. | Package manager, Node.js. | CLI: `npm run dev`, `npm run build`, `npm run lint`, `npm run test`. | Tidak ada. | TECH | Low |
| Backend Build Configuration | Setup Golang module, Fiber app skeleton, config loader, logger, test runner. | Go toolchain, PostgreSQL client library. | CLI: `go test ./...`, `go run ./server`. | Tidak ada. | TECH | Low |
| Database Migration & PostGIS Setup | Membuat schema PostgreSQL/PostGIS, extension, tabel, index, constraints, trigger `updated_at`. | PostgreSQL 15+, PostGIS 3.3+. | Tidak ada API publik; dijalankan via migration CLI. | `layers`, `admin_boundaries`, `infrastructure`, `layer_attribute_definitions`. | P0 | Medium |
| Data Seeding & Fixture Import | Mengisi data awal layer, fixture admin boundaries, fixture infrastructure, dan metadata atribut tooltip. | Database migration, dataset BPS/Kemendagri/OSM atau fixture GeoJSON. | Tidak ada API publik; dijalankan via seeder/import script. | `layers`, `admin_boundaries`, `infrastructure`, `layer_attribute_definitions`. | P0 | Medium |
| Backend Config Module | Membaca environment variable untuk database URL, CORS origin, server port, API base path, logging level. | Backend build configuration. | Internal config object; env var injection. | Tidak ada. | P0 | Low |
| Database Connection & Repository Layer | Mengelola connection pool, prepared statement, transaction helper, repository query execution. | Config module, PostgreSQL/PostGIS. | Tidak ada HTTP API; digunakan oleh service layer. | Semua tabel spasial dan metadata. | P0 | Medium |
| Layer Metadata Service | Mengelola metadata layer untuk UI dan layer switcher. | Repository layer, `layers`, `layer_attribute_definitions`. | `GET /api/v1/metadata/layers`, `GET /api/v1/metadata/attributes/:layerId`. | `layers`, `layer_attribute_definitions`. | P0 | Low |
| Admin Boundary Vector Service | Query batas administrasi berdasarkan `bbox`, `zoom_level`, `level`; validasi geometri; generalization opsional. | Repository layer, PostGIS, layer metadata. | `GET /api/v1/vector/admin-boundaries?bbox=&zoom_level=&level=`. | `admin_boundaries`, `layers`. | P0 | High |
| Infrastructure Vector Service | Query infrastruktur berdasarkan `bbox`, `zoom_level`, `type`; validasi geometri; generalization opsional. | Repository layer, PostGIS, layer metadata. | `GET /api/v1/vector/infrastructure?bbox=&zoom_level=&type=`. | `infrastructure`, `layers`. | P0 | High |
| Health Check Module | Menyediakan status aplikasi, database, PostGIS, uptime, dan dependency health. | Database connection, runtime environment. | `GET /health`. | Tidak langsung; melakukan `SELECT 1`, `SELECT postgis_version()`. | P0 | Low |
| Request Validation & Error Handling | Validasi query parameter, error response standar, request ID, HTTP status mapping. | Backend handlers, API contract. | Semua endpoint HTTP; response error format `{ "error": { ... } }`. | Tidak ada. | P0 | Medium |
| CORS, Security Headers & Rate Limiting | Mengamankan akses browser, membatasi origin, dan mencegah abuse pada API vector. | Fiber middleware, environment config. | Middleware untuk semua endpoint; rate limit pada `/api/v1/vector/*`. | Tidak ada untuk Sprint 1; rate limit dapat memakai in-memory/Redis di roadmap. | P0 | Medium |
| API Client Module | Fetch wrapper untuk metadata dan vector endpoints, retry dasar, error normalization, timeout handling. | Frontend build configuration, backend API. | Mengonsumsi semua endpoint `/api/v1/*`. | Tidak ada. | P0 | Low |
| Geometry & CRS Utility | Transform koordinat `EPSG:4326 <-> EPSG:3857`, format bbox, format coordinate 6 digit. | OpenLayers projection utility, browser API. | Utility function; digunakan oleh map, vector loader, coordinate bar. | Tidak ada. | P0 | Low |
| Frontend App Shell | Root layout full-screen, mount Lit components, manage global UI state. | Vite, Lit JS, Tailwind CSS. | Custom events: `map:ready`, `layer:toggle`, `tooltip:show`, `tooltip:hide`. | Tidak ada. | P0 | Low |
| OpenLayers Map Engine | Inisialisasi map, view, zoom range, extent Indonesia, interactions dasar. | OpenLayers, geometry utility. | Events: `moveend`, `zoomend`, `pointermove`, `click`, `viewchange`. | Tidak ada. | P0 | Medium |
| Base Map Tile Module | Menampilkan OSM tile layer, attribution, min/max zoom, tile cache browser. | OpenLayers Map Engine, OSM tile provider. | Tile URL: `https://{a-c}.tile.openstreetmap.org/{z}/{x}/{y}.png`. | Tidak ada. | P0 | Low |
| Vector Layer Factory | Membuat dan mengelola vector layer dari metadata API, styling, z-index, visibility. | Metadata API, API client, OpenLayers. | Menggunakan `GET /api/v1/metadata/layers`. | `layers`, `layer_attribute_definitions`. | P0 | Medium |
| Admin Boundary Layer Renderer | Render feature MultiPolygon admin boundaries dengan style per level. | Vector Layer Factory, Admin Boundary Vector Service. | Menggunakan `GET /api/v1/vector/admin-boundaries`. | `admin_boundaries`, `layers`. | P0 | Medium |
| Infrastructure Layer Renderer | Render feature LineString infrastruktur dengan style per type. | Vector Layer Factory, Infrastructure Vector Service. | Menggunakan `GET /api/v1/vector/infrastructure`. | `infrastructure`, `layers`. | P0 | Medium |
| BBOX Vector Loading Strategy | Memuat ulang vector data berdasarkan viewport saat `moveend`/`zoomend`. | OpenLayers Map Engine, API client, vector services. | Query param `bbox`, `zoom_level`; trigger dari map events. | `admin_boundaries`, `infrastructure`. | P0 | High |
| Layer Switcher Component | Menampilkan daftar layer, checkbox visibility, badge, dan state active layer. | Metadata API, App Shell, Vector Layer Factory. | Event: `layer:toggle`; API: `GET /api/v1/metadata/layers`. | `layers`. | P0 | Low |
| Navigation Controls Component | Zoom in/out, drag/pan, double-click zoom, reset view ke extent Indonesia. | OpenLayers Map Engine, geometry utility. | Events: `zoom:in`, `zoom:out`, `view:reset`. | Tidak ada. | P0 | Low |
| Coordinate Bar Component | Menampilkan lon/lat real-time, zoom level, jumlah layer aktif. | OpenLayers Map Engine, geometry utility. | Event: `pointermove`, `viewchange`. | Tidak ada. | P0 | Low |
| Hover Hit-Test Module | Mendeteksi feature di bawah pointer menggunakan `map.forEachFeatureAtPixel()`. | OpenLayers Map Engine, vector layers. | Event: `featurehover`, `featureout`; hit tolerance ±5 px. | Tidak ada. | P0 | Medium |
| Tooltip Component | Menampilkan atribut feature dengan positioning, flip logic, dismiss, dan format nilai. | Hover Hit-Test Module, metadata attributes. | Props/Event: `feature`, `position`, `visible`, `dismiss`. | `layer_attribute_definitions` untuk metadata tooltip. | P0 | Medium |
| Highlight Overlay Module | Mengubah style feature yang sedang di-hover dan mengembalikan style default saat keluar. | Hover Hit-Test Module, vector style factory. | Event: `featurehover`, `featureout`. | Tidak ada. | P0 | Medium |
| Loading, Error & Empty State Module | Menampilkan spinner/skeleton, toast retry, empty message jika tidak ada feature. | API Client Module, App Shell. | Events: `api:loading`, `api:error`, `api:empty`. | Tidak ada. | P0 | Low |
| Styling & Theme System | Design token warna, layer style, hover style, dark theme default, Tailwind config. | Frontend App Shell, vector layers, UI components. | Utility/style factory; tidak ada HTTP API. | Tidak ada. | P0 | Low |
| Testing & CI/CD Pipeline | Unit test, integration test, build, lint, coverage target, pipeline stage. | Frontend/backend test framework, GitHub Actions/GitLab CI. | CLI: `npm run test`, `go test ./...`, `npm run build`. | Test database opsional. | TECH | Medium |
| Docker Compose Development Stack | Menjalankan frontend, backend, PostgreSQL/PostGIS secara lokal. | Docker, PostgreSQL/PostGIS image, env file. | Local ports: frontend `5173`, backend `3000`, database `5432`. | Semua tabel database. | TECH | Medium |
| Authentication & RBAC Module | Login, JWT, role enforcement, GIS Officer read-only, System Administrator write access. | User/role schema, session/token strategy. | `POST /api/v1/auth/login`, protected endpoints dengan `Authorization: Bearer <token>`. | `users`, `roles`, `permissions`, `sessions`. | P1 | High |
| Data Management CRUD Module | Upload Shapefile/GeoJSON, validasi geometri, import log, edit layer metadata. | Auth/RBAC, OGR/shp2pgsql, import worker. | `POST /api/v1/layers/:id/import`, `PATCH /api/v1/layers/:id`, `GET /api/v1/import-logs`. | `layers`, `admin_boundaries`, `infrastructure`, `data_import_logs`. | P1 | High |
| Geometry Validation Module | Validasi `ST_IsValid`, `ST_SRID`, bounding extent Indonesia, auto-fix ringan. | Data Management CRUD, PostGIS. | Dipanggil saat import dan health/data audit. | `admin_boundaries`, `infrastructure`. | P1 | Medium |
| Advanced Analysis Module | Measure distance/area, draw/annotation, buffer/spatial analysis. | OpenLayers Draw/Measure interactions, analysis API. | `POST /api/v1/analysis/buffer`, `POST /api/v1/analysis/overlay`. | `admin_boundaries`, `infrastructure`, custom feature tables. | P2 | High |
| Export & Presentation Module | Export PDF peta, copy attribute, share view URL, legend/scale context. | Map engine, tooltip, analysis/layer state. | `GET /api/v1/export/pdf`, `POST /api/v1/export/geojson`. | `layers`, feature tables. | P2 | High |
| Search & Geocoding Module | Full-text search nama wilayah/kode dan geocoding lokasi. | PostgreSQL tsvector, external geocoder optional. | `GET /api/v1/search?q=&type=`. | `admin_boundaries`, `infrastructure`, search index. | P3 | Medium |
| External Data Integration Module | Sync data dari API eksternal, webhook notification, scheduled ingestion. | Scheduler/worker, external API contract. | Webhook endpoint, sync job API. | `data_import_logs`, external sync metadata tables. | P3 | High |
| Mobile & Responsive Enhancement Module | Tablet/mobile layout, touch interaction, responsive layer panel. | Frontend App Shell, OpenLayers interactions. | UI adaptation; tidak ada HTTP API baru. | Tidak ada. | P3 | Medium |
| MVT & Performance Optimization Module | Mengubah vector response besar menjadi Mapbox Vector Tiles untuk dataset besar. | PostGIS MVT function, OpenLayers vector tile source. | `GET /api/v1/tiles/:layer/{z}/{x}/{y}.pbf`. | `admin_boundaries`, `infrastructure`, materialized views. | P2/P3 | High |

---

## 3. Modul MVP Sprint 1

| Modul | Priority | Complexity | Catatan |
|---|---|---|---|
| Database Migration & PostGIS Setup | P0 | Medium | Fondasi utama sebelum backend/API. |
| Data Seeding & Fixture Import | P0 | Medium | Bisa memakai fixture kecil sebelum dataset resmi tersedia. |
| Backend Config Module | P0 | Low | Diperlukan untuk env parity. |
| Database Connection & Repository Layer | P0 | Medium | Basis query aman dan parameterized. |
| Layer Metadata Service | P0 | Low | Mendukung layer switcher dan tooltip metadata. |
| Admin Boundary Vector Service | P0 | High | Query spasial utama dengan GIST index. |
| Infrastructure Vector Service | P0 | High | Query spasial utama dengan GIST index. |
| Health Check Module | P0 | Low | Endpoint monitoring dasar. |
| Request Validation & Error Handling | P0 | Medium | Wajib untuk API contract. |
| CORS, Security Headers & Rate Limiting | P0 | Medium | Minimal CORS strict untuk Sprint 1. |
| API Client Module | P0 | Low | Abstraksi fetch dan error handling. |
| Geometry & CRS Utility | P0 | Low | Kunci untuk konsistensi koordinat. |
| Frontend App Shell | P0 | Low | Root layout aplikasi. |
| OpenLayers Map Engine | P0 | Medium | Core rendering map. |
| Base Map Tile Module | P0 | Low | OSM tile rendering. |
| Vector Layer Factory | P0 | Medium | Mengubah metadata API menjadi layer runtime. |
| Admin Boundary Layer Renderer | P0 | Medium | Render MultiPolygon. |
| Infrastructure Layer Renderer | P0 | Medium | Render LineString. |
| BBOX Vector Loading Strategy | P0 | High | Penting untuk performa viewport loading. |
| Layer Switcher Component | P0 | Low | Toggle layer. |
| Navigation Controls Component | P0 | Low | Zoom/pan/reset view. |
| Coordinate Bar Component | P0 | Low | Lon/lat display. |
| Hover Hit-Test Module | P0 | Medium | Deteksi feature saat mouse hover. |
| Tooltip Component | P0 | Medium | Menampilkan atribut feature. |
| Highlight Overlay Module | P0 | Medium | Visual feedback hover. |
| Loading, Error & Empty State Module | P0 | Low | Reliability UX dasar. |
| Styling & Theme System | P0 | Low | Konsistensi visual. |
| Testing & CI/CD Pipeline | TECH | Medium | Wajib untuk production-ready. |
| Docker Compose Development Stack | TECH | Medium | Mempermudah local development. |

---

## 4. Dependency Flow

| Urutan | Modul Dependensi | Modul Target |
|---:|---|---|
| 1 | Frontend Build Configuration, Backend Build Configuration | Semua modul aplikasi. |
| 2 | Database Migration & PostGIS Setup | Data Seeding, Repository Layer, Vector Services. |
| 3 | Data Seeding & Fixture Import | Layer Metadata Service, Vector Services, Frontend rendering. |
| 4 | Backend Config + Repository Layer | Health, Metadata, Vector API. |
| 5 | Layer Metadata Service | Vector Layer Factory, Layer Switcher, Tooltip metadata. |
| 6 | Admin/Infrastructure Vector Services | Admin Boundary Renderer, Infrastructure Renderer. |
| 7 | OpenLayers Map Engine + Base Map | Vector Layers, Navigation, Coordinate Bar, Hover. |
| 8 | Vector Layer Factory + BBOX Loading | Admin Boundary Renderer, Infrastructure Renderer. |
| 9 | Hover Hit-Test + Metadata Attribute | Tooltip + Highlight Overlay. |
| 10 | API Client + Error Handling | Loading, Error & Empty State Module. |
| 11 | Testing & CI/CD | Semua modul sebelum deploy. |

---

## 5. Catatan Arsitektur

1. Sprint 1 fokus pada GeoJSON + BBOX loading. MVT dipindahkan ke modul performa ketika dataset melebihi batas aman GeoJSON.
2. Semua geometri disimpan dalam `EPSG:4326`; rendering OpenLayers dilakukan di `EPSG:3857`.
3. Frontend tidak menyimpan state permanen pada Sprint 1. State hanya runtime map, layer visibility, tooltip, dan coordinate display.
4. Backend menggunakan Clean Architecture sederhana: handler, service, repository, model.
5. `layer_attribute_definitions` dibuat agar tooltip tidak hardcode di frontend dan dapat dikembangkan untuk custom layer di Sprint 2+.
6. Auth, CRUD data, advanced analysis, export, search, dan external integration adalah modul roadmap setelah core MVP selesai.
