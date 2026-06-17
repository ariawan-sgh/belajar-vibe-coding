# Domain Model — WebGIS

**Basis Dokumen:** `docs/prd.md` v1.0
**Tanggal:** 2026-06-17

---

## 1. Ringkasan Domain

Domain WebGIS berfokus pada manajemen dan visualisasi data spasial (geografis). Sistem membagi domain menjadi empat bounded context utama:

| Bounded Context | Tanggung Jawab |
|-----------------|----------------|
| **Cartography** | Representasi visual peta, layer, tiles, styling, dan interaksi spasial. |
| **Spatial Data Management** | Penyimpanan, versi, validasi, dan query data spasial di PostgreSQL + PostGIS. |
| **Identity & Access** | Autentikasi, otorisasi, dan profil pengguna yang berinteraksi dengan sistem. |
| **Analytics & Telemetry** | Pengukuran performa peta, interaksi pengguna, dan kesehatan sistem. |

---

## 2. Entity-Relationship Overview

```
┌──────────────┐       ┌──────────────────┐       ┌──────────────────┐
│    User      │───────│    Role          │       │    Permission    │
│              │       │                  │       │                  │
│ - user_id   │       │ - role_id        │       │ - permission_id  │
│ - username  │       │ - name           │       │ - action         │
│ - email     │       │ - description    │       │ - resource       │
└──────────────┘       └──────────────────┘       └──────────────────┘
        │                       │
        │                       │
        ▼                       ▼
┌──────────────┐       ┌──────────────────┐
│   Session   │       │     Project      │
│              │       │                  │
│ - session_id│       │ - project_id     │
│ - user_id   │       │ - name           │
│ - started_at│       │ - crs            │
│ - ended_at  │       │ - extent         │
└──────────────┘       └────────┬─────────┘
                                │
                                │ 1 ──── n
                                ▼
                         ┌──────────────┐
                         │     Layer    │
                         │              │
                         │ - layer_id   │
                         │ - project_id │
                         │ - name        │
                         │ - type        │
                         │ - enabled     │
                         │ - z_index     │
                         └──────┬───────┘
                                │
                                │ 1 ──── n
                                ▼
                         ┌──────────────┐
                         │    Feature   │
                         │              │
                         │ - feature_id │
                         │ - layer_id   │
                         │ - geometry   │
                         │ - properties │
                         │ - version    │
                         │ - valid_from │
                         │ - valid_to   │
                         └──────────────┘
```

---

## 3. Core Entities & Value Objects

### 3.1 User (Aggregate Root)

**Kontek:** Identity & Access

| Atribut | Tipe | Deskripsi |
|---------|------|-----------|
| `user_id` | UUID | Identifikasi unik pengguna. |
| `username` | VARCHAR(100) | Nama pengguna unik untuk login. |
| `email` | VARCHAR(255) | Alamat email institusional. |
| `full_name` | VARCHAR(255) | Nama lengkap sesuai data HR. |
| `is_active` | BOOLEAN | Status akun aktif/nonaktif. |
| `created_at` | TIMESTAMP | Waktu pembuatan akun. |
| `updated_at` | TIMESTAMP | Waktu pembaruan terakhir. |

**Invariant (Business Rules):**
- Setiap `User` MUST ditugaskan ke minimal satu `Role`.
- `email` harus unik di dalam sistem.
- `is_active = false` melarang akses ke sistem kecuali `Role` adalah `SystemAdministrator`.

---

### 3.2 Role

| Atribut | Tipe | Deskripsi |
|---------|------|-----------|
| `role_id` | SERIAL | Primary key. |
| `name` | VARCHAR(50) | Nama role: `GISOfficer`, `SystemAdministrator`, `Viewer`. |
| `description` | TEXT | Deskripsi tanggung jawab role. |

**Kardinalitas:** User (n) ── (1) Role

**Invariant:**
- `name` harus unique.
- Role `SystemAdministrator` memiliki implicit permission `*` (wildcard) untuk manajemen data.

---

### 3.3 Permission

| Atribut | Tipe | Deskripsi |
|---------|------|-----------|
| `permission_id` | SERIAL | Primary key. |
| `action` | VARCHAR(50) | Operasi: `read`, `write`, `delete`, `admin`. |
| `resource_type` | VARCHAR(100) | Jenis sumber daya: `layer`, `feature`, `user`, `system_config`. |
| `resource_id` | UUID | Spesifik resource (nullable untuk wildcard). |

**Kardinalitas:** Role (n) ── (n) Permission (Many-to-Many)

---

### 3.4 Project

**Kontek:** Cartography

Merupakan wadah spasial untuk sesi kerja pengguna. Dalam konteks WebGIS, `Project` mewakili state peta pada satu waktu tertentu (center, zoom, extent, visible layers).

| Atribut | Tipe | Deskripsi |
|---------|------|-----------|
| `project_id` | UUID | Identifikasi unik project. |
| `name` | VARCHAR(255) | Nama project / sesi kerja. |
| `owner_id` | UUID | FK ke `User.user_id`. |
| `crs` | VARCHAR(50) | Proyeksi default: `EPSG:3857` atau `EPSG:4326`. |
| `center` | GEOMETRY(Point, 4326) | Titik tengah peta dalam WGS84. |
| `zoom` | INTEGER | Zoom level saat project disimpan. |
| `extent` | GEOMETRY(Polygon, 4326) | Batas koordinat yang terlihat (opsional). |
| `created_at` | TIMESTAMP | Waktu pembuatan. |
| `updated_at` | TIMESTAMP | Waktu pembaruan terakhir. |

**Invariant:**
- `zoom` MUST berada dalam rentang 3-18.
- `center` MUST valid geometri (`ST_IsValid`).

---

### 3.5 Layer (Aggregate Root)

**Kontek:** Cartography & Spatial Data Management

Layer adalah koleksi feature spasial yang memiliki styling, metadata, dan visibility state bersama.

| Atribut | Tipe | Deskripsi |
|---------|------|-----------|
| `layer_id` | SERIAL | Primary key. |
| `project_id` | UUID | FK opsional ke `Project` (null untuk global layer). |
| `name` | VARCHAR(255) | Nama layer yang ditampilkan di UI. |
| `description` | TEXT | Deskripsi isi layer. |
| `type` | ENUM | Tipe layer: `base_map`, `administrative_boundary`, `infrastructure`, `custom`. |
| `geom_type` | ENUM | Jenis geometri dominan: `Point`, `LineString`, `Polygon`, `MultiPolygon`. |
| `source` | JSONB | Konfigurasi sumber data: `{ "type": "postgis", "table": "admin_boundaries", "srid": 4326 }`. |
| `enabled` | BOOLEAN | Status visibility layer. |
| `z_index` | INTEGER | Urutan rendering (0 = paling bawah). |
| `attribution` | TEXT | Sumber data dan lisensi. |
| `style` | JSONB | Styling default: `{ "fill": "#3388ff", "stroke": "#ffffff", "width": 2 }`. |
| `created_at` | TIMESTAMP | Waktu pembuatan. |
| `updated_at` | TIMESTAMP | Waktu pembaruan terakhir. |

**Invariant:**
- `z_index` harus unik per `project_id` (untuk layer dengan `project_id` yang sama).
- `source` harus terdefinisi untuk layer selain `base_map`.
- `enabled = true` tidak menjamin feature muncul jika parent `Project` tidak aktif.

**Kardinalitas:**
- Project (1) ── (n) Layer
- LayerType (1) ── (n) Layer

---

### 3.6 Feature (Entity)

**Kontek:** Spatial Data Management

Feature adalah objek spasial tunggal di dalam layer (misal: satu provinsi, satu jalan).

| Atribut | Tipe | Deskripsi |
|---------|------|-----------|
| `feature_id` | SERIAL | Primary key. |
| `layer_id` | INTEGER | FK ke `Layer.layer_id`. |
| `external_id` | VARCHAR(255) | ID dari sumber data asli (misal: kode Kemendagri). |
| `geometry` | GEOMETRY | Geometri spasial dalam SRID layer (default 4326). |
| `properties` | JSONB | Atribut bebas: `{ "name": "DKI Jakarta", "code": "31", "area_km2": 664.01 }`. |
| `version` | INTEGER | Nomor versi untuk optimistik locking (opsional Sprint 1). |
| `valid_from` | TIMESTAMP | Mulai berlaku data. |
| `valid_to` | TIMESTAMP | Berakhir berlaku data (null jika masih berlaku). |
| `created_at` | TIMESTAMP | Waktu import pertama. |
| `updated_at` | TIMESTAMP | Waktu pembaruan terakhir. |

**Invariant:**
- `geometry` MUST valid (`ST_IsValid(geometry) = true`).
- `ST_SRID(geometry)` MUST sesuai dengan `Layer.geom_type` basis atau layer CRS.
- `external_id` + `layer_id` harus unique untuk mencegah duplikasi.
- `valid_to` = null menandakan data berlaku saat ini.

**Kardinalitas:**
- Layer (1) ── (n) Feature (satu layer mengandung ratusan ribu features)

---

### 3.7 Tile (Value Object)

**Kontek:** Cartography

Representasi satu tile gambar peta dasar.

| Atribut | Tipe | Deskripsi |
|---------|------|-----------|
| `z` | INTEGER | Zoom level. |
| `x` | INTEGER | Column index. |
| `y` | INTEGER | Row index. |
| `url` | STRING | URL tile (misal: `https://tile.osm.org/4/8/4.png`). |
| `crs` | STRING | Proyeksi tile: `EPSG:3857`. |

**Catatan:** Tile tidak dipersist di database, hanya transit di cache browser/CDN.

---

### 3.8 CoordinateSystem (Value Object)

**Kontek:** Cartography — Spesifikasi sistem referensi koordinat yang digunakan sistem.

| Atribut | Tipe | Deskripsi |
|---------|------|-----------|
| `srid` | INTEGER | EPSG code (contoh: 4326, 3857). |
| `name` | STRING | Nama proyeksi (contoh: "WGS 84", "Web Mercator"). |
| `unit` | STRING | Satuan: `degree`, `metre`. |
| `extent_wgs84` | EXTENT | `[min_lon, min_lat, max_lon, max_lat]` untuk bounding box dunia. |

**Kardinalitas:** Satu sistem menggunakan minimal 2 CRS: `EPSG:4326` (storage) dan `EPSG:3857` (display).

---

### 3.9 Session (Aggregate Root)

**Kontek:** Analytics & Telemetry

Merekam sesi penggunaan aplikasi oleh seorang User.

| Atribut | Tipe | Deskripsi |
|---------|------|-----------|
| `session_id` | UUID | Identifikasi sesi. |
| `user_id` | UUID | FK ke `User.user_id`. |
| `started_at` | TIMESTAMP | Waktu login / buka aplikasi. |
| `ended_at` | TIMESTAMP | Waktu logout / tutup aplikasi (null jika masih aktif). |
| `device_info` | JSONB | `{ "browser": "Chrome", "version": "120", "os": "Windows" }`. |
| `actions` | JSONB | Array aksi: `[ { "type": "zoom", "timestamp": "...", "to_zoom": 10 }, ... ]` (opsional). |

---

## 4. Enumerations

### 4.1 LayerType

| Nilai | Deskripsi |
|-------|-----------|
| `base_map` | Tile layer dasar (OSM, custom). |
| `administrative_boundary` | Batas administrasi (provinsi, kabupaten, dll). |
| `infrastructure` | Infrastruktur: jalan, kereta api, bandara, bendungan. |
| `custom` | Layer tambahan yang diupload admin. |

### 4.2 FeatureStatus

| Nilai | Deskripsi |
|-------|-----------|
| `active` | Feature berlaku dan ditampilkan. |
| `deprecated` | Feature replaced oleh versi baru, tidak ditampilkan. |
| `draft` | Feature belum siap untuk production. |

### 4.3 ActionType (untuk Audit / Telemetry)

| Nilai | Deskripsi |
|-------|-----------|
| `view_loaded` | Peta pertama kali dimuat. |
| `zoom` | Perubahan zoom level. |
| `pan` | Perubahan posisi peta (pan). |
| `hover_feature` | Mouse hover ke feature. |
| `toggle_layer` | Menyalakan/mematikan layer. |
| `fit_extent` | Reset view ke extent tertentu. |

---

## 5. Relationship Detail & Cardinality

| Dari | Ke | Kardinalitas | Deskripsi |
|------|----|--------------|-----------|
| `User` | `Role` | Many-to-One | Satu user memiliki satu role utama. |
| `Role` | `Permission` | Many-to-Many | Role aggregates banyak permission. |
| `User` | `Project` | One-to-Many | Satu user bisa punya banyak project. |
| `Project` | `Layer` | One-to-Many | Satu project memiliki banyak layer. |
| `Layer` | `Feature` | One-to-Many | Satu layer mengandung banyak feature spasial. |
| `Layer` | `LayerType` | Many-to-One | Setiap layer bergantung pada satu tipe. |
| `User` | `Session` | One-to-Many | Satu user bisa punya banyak session. |
| `Layer` | `CRS` | Many-to-One | Layer disimpan dalam satu CRS (umumnya EPSG:4326). |

---

## 6. State Machines

### 6.1 Layer Lifecycle

```
                ┌───────────────────────────────────────┐
                │                                      │
                ▼                                      │
     ┌────────────────┐                               │
     │ draft          │  (enabled=true, publish)      │
     │ - default vis: false                               │
     │ - z_index: null │  ───────────────────────────► │
     └──────┬─────────┘                               │
            │                                          │
            │ (edit / update data)                     │
            │                                          │
            ▼                                          │
     ┌────────────────┐                               │
     │ active         │  (enabled=true, z_index set)  │
     │ - default vis: true                                │
     │ - data valid   │                               │
     └──────┬─────────┘                               │
            │                                          │
            │ (disable)                                │
            │                                          │
            ▼                                          │
     ┌────────────────┐                               │
     │ disabled       │  (enabled=false)             │
     │ - tetap ada di │                               │
     │   metadata      │  ───────────────────────────► │
     └──────┬─────────┘                               │
            │                                          │
            │ (remove / archive)                      │
            │                                          │
            ▼                                          │
     ┌────────────────┐                               │
     │ archived       │                               │
     │ - soft delete  │                               │
     │ - tidak di-query│                              │
     └────────────────┘                               │
                │                                      │
                └──────────────────────────────────────┘
```

### 6.2 Feature Lifecycle

```
     ┌─────────────┐
     │ imported    │  (ST_IsValid = true, valid_from = now())
     │ - siap query │
     └──────┬──────┘
            │
            │ (sesuai kebijakan retention / update data)
            │
            ▼
     ┌─────────────┐
     │ active      │  (valid_to IS NULL, visible)
     └──────┬──────┘
            │
            │ (data diperbarui: perubahan batas wilayah, dll)
            │
            ▼
     ┌─────────────┐
     │ superseded  │  (valid_to = now(), version++)
     │ - history    │
     └──────┬──────┘
            │
            │ (masa retensi habis / purge policy)
            │
            ▼
     ┌─────────────┐
     │ purged      │  (hard delete atau archive ke cold storage)
     └─────────────┘
```

### 6.3 Session Lifecycle

```
     ┌─────────────┐
     │ created     │  (user login / buka aplikasi)
     └──────┬──────┘
            │
            │ (user aktif berinteraksi)
            │
            ▼
     ┌─────────────┐
     │ active      │  (event: zoom, pan, hover, toggle layer)
     └──────┬──────┘
            │
            │ (timeout: 30 menit tanpa aktivitas)
            │ ATAU
            │ (user logout)
            │
            ▼
     ┌─────────────┐
     │ ended       │  (ended_at diisi, assets released)
     └─────────────┘
```

---

## 7. Domain Services

Domain Services berisi logic bisnis yang tidak secara natural milik satu Entity/Value Object.

| Service | Metode | Deskripsi |
|---------|--------|-----------|
| `CoordinateTransformService` | `transform(geom, fromSrid, toSrid)` | Transformasi koordinat antar SRID. Memanfaatkan PostGIS `ST_Transform` atau proj4js di frontend. |
| `SpatialQueryService` | `queryIntersects(layerId, bbox, zoomLevel)` | Menjalankan query spasial: `ST_Intersects(geom, ST_MakeEnvelope(...))`. |
| `GeometryValidationService` | `validate(geom)` | Mengecek `ST_IsValid(geom)`, `ST_SRID(geom)`, dan coordinate bounds Indonesia. |
| `LayerZIndexService` | `resolveZIndex(projectId)` | Menentukan urutan rendering layer dalam satu project (menangani konflik `z_index`). |
| `FeatureStyleService` | `resolveStyle(feature, layerStyle, hoverState)` | Menghitung style final: gabungkan default layer style + highlight state + scale-dependent style. |
| `AttributionService` | `getAttribution(layerIds)` | Menggabungkan attribution dari semua layer aktif dan base map untuk ditampilkan di UI. |

---

## 8. Aggregates & Boundaries

### 8.1 Aggregate: Layer

**Root Entity:** `Layer`
**Invariants di-protect:**
- `z_index` unik per `project_id`.
- `enabled` status konsisten dengan metadata visibility.

**Operasi Atomic:**
- `Layer.enable()` / `Layer.disable()`: mengubah visibility seluruh feature di layer.
- `Layer.updateStyle(newStyle)`: mengganti default style untuk semua features di layer.

### 8.2 Aggregate: Feature

**Root Entity:** `Feature`
**Invariants di-protect:**
- `geometry` selalu valid.
- `external_id` unik per `layer_id`.
- Hanya satu `Feature` aktif (valid_to IS NULL) per `external_id` per waktu.

**Operasi Atomic:**
- `Feature.supersede(newGeometry, newProperties)`: menutup feature saat ini (set `valid_to = now()`) dan membuat feature baru dengan versi +1.
- `Feature.archive()`: soft delete.

---

## 9. Anti-Corruption Layer (ACL)

Sistem WebGIS berinteraksi dengan sumber data eksternal yang mungkin memiliki model domain berbeda. ACL memastikan model internal tidak tercemar oleh model eksternal.

| Eksternal System | Model Eksternal | ACL Adapter | Model Internal |
|------------------|-----------------|-------------|----------------|
| **OSM / Overpass API** | Node, Way, Relation (XML/JSON) | `OSMFeatureAdapter` | `Feature` dengan `geom_type: LineString/Polygon` |
| **BPS / Kemendagri** | Shapefile (.shp) dengan atribut kemendagri | `ShapefileImporter` | `Feature` dengan `properties` ter-normalisasi |
| **Tile Provider (OSM)** | XYZ Tile scheme | `TileURLBuilder` | `Tile` value object |

**Contoh Transformasi:**
- OSM `Way` dengan `highway=primary` → `Feature.layer_id = infrastructure`, `Feature.properties = { "type": "national_road", "name": "...", "condition": null }`.

---

## 10. Domain Events

| Event | Data | Trigger |
|-------|------|---------|
| `LayerEnabled` | `{ layer_id, project_id, timestamp }` | GIS Officer mengaktifkan layer. |
| `LayerDisabled` | `{ layer_id, project_id, timestamp }` | GIS Officer menonaktifkan layer. |
| `FeatureHovered` | `{ feature_id, layer_id, cursor_position, timestamp }` | Mouse hover ke feature. |
| `ZoomLevelChanged` | `{ old_zoom, new_zoom, center, timestamp }` | Perubahan zoom atau pan. |
| `DataImported` | `{ layer_id, feature_count, source_file, timestamp }` | SysAdmin meng-upload data baru. |
| `GeometryInvalid` | `{ feature_id, layer_id, validation_error }` | Validasi geometri gagal saat import. |
| `SessionStarted` | `{ session_id, user_id, timestamp }` | User membuka aplikasi. |
| `SessionEnded` | `{ session_id, user_id, duration_seconds }` | User menutup aplikasi. |

---

## 11. Integrity Constraints Summary

| Constraint | Scope | Implementation |
|------------|-------|----------------|
| Coordinate Range Indonesia | `Feature.geometry` | `CHECK (ST_XMin(geom) BETWEEN 95.3 AND 141.0 AND ST_YMin(geom) BETWEEN -10.9 AND 5.9)` |
| Unique Layer per Project | `Layer` | `UNIQUE(project_id, name)` |
| Unique External ID per Layer | `Feature` | `UNIQUE(layer_id, external_id)` |
| Valid Geometry | `Feature.geometry` | `CHECK (ST_IsValid(geom))` |
| Valid SRID | `Feature.geometry` | `CHECK (ST_SRID(geom) = 4326)` |
| Active Feature | `Feature` | `CHECK (valid_to IS NULL)` untuk constraint "satu versions aktif per external_id". |
| Zoom Range | `Project.zoom` | `CHECK (zoom BETWEEN 3 AND 18)` |
| Z-Index Uniqueness | `Layer` per project | Aplikasi enforce uniqueness sebelum insert/update. |

---

## 12. Catatan untuk Engineering

1. **SRID Dominan:** Semua operasi spasial di database menggunakan EPSG:4326. Transformasi ke EPSG:3857 hanya terjadi di boundary frontend (OpenLayers) atau query conditional dengan `ST_Transform`.
2. **JSONB Schema Drift:** Kolom `properties` dan `style` menggunakan JSONB. Hindari hardcode key names di seluruh codebase. Buat constant atau enum untuk field names yang umum digunakan.
3. **Geometry Complexity:** MultiPolygon untuk batas administrasi bisa sangat kompleks (ribuan vertex). Backend SHOULD menjalankan `ST_SimplifyPreserveTopology(geom, tolerance)` berdasarkan zoom level.
4. **Soft Delete Pattern:** Untuk `Feature`, gunakan `valid_to` timestamp sebagai soft delete agar history spasial tetap terjaga untuk audit.
5. **Project State Serialization:** Untuk fitur "Share View" (roadmap), `Project` dapat diserialisasi ke JSON base64 dan di-pass sebagai URL parameter. Pastikan `Project.center` dan `Project.zoom` konsisten dalam CRS mana pun.

---

*Document Owner: Product Management & Business Analysis Team*
*Last Review: 2026-06-17*
*Next Review: 2026-07-01*
