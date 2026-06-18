# ERD PostgreSQL/PostGIS — WebGIS

**Basis Dokumen:** `docs/prd.md` v1.0  
**Database:** PostgreSQL 15+ dengan PostGIS 3.3+  
**Tanggal:** 2026-06-18  
**Status:** Draft technical design untuk Sprint 1 Core Foundation

---

## 1. Ringkasan Desain

ERD ini memodelkan kebutuhan Sprint 1 dari PRD:

- Metadata layer peta.
- Data batas administrasi Indonesia.
- Data infrastruktur publik primer.
- Validasi geometri PostGIS.
- Indeks spasial dan indeks filter.
- Relasi hierarki antar batas administrasi.
- Ekstensi roadmap ringan untuk metadata atribut tooltip.

### Cakupan utama

| Area | Tabel |
|---|---|
| Layer metadata | `layers` |
| Batas administrasi | `admin_boundaries` |
| Infrastruktur | `infrastructure` |
| Metadata atribut layer | `layer_attribute_definitions` |

---

## 2. Mermaid ER Diagram

```mermaid
erDiagram
    layers ||--o{ admin_boundaries : contains
    layers ||--o{ infrastructure : contains
    layers ||--o{ layer_attribute_definitions : defines
    admin_boundaries ||--o{ admin_boundaries : "parent-child hierarchy"

    layers {
        integer id PK
        varchar(255) name UK
        text description
        varchar(50) type
        varchar(50) geom_type
        integer srid
        text attribution
        boolean is_public
        boolean default_visibility
        integer z_index
        timestamp created_at
        timestamp updated_at
    }

    admin_boundaries {
        integer id PK
        integer layer_id FK
        varchar(50) level
        varchar(50) code
        varchar(255) name
        integer parent_id FK
        varchar(255) province_name
        varchar(255) regency_name
        varchar(255) district_name
        bigint population
        numeric area_km2
        geometry MultiPolygon geom
        timestamp valid_from
        timestamp valid_to
        timestamp created_at
        timestamp updated_at
    }

    infrastructure {
        integer id PK
        integer layer_id FK
        varchar(100) type
        varchar(255) name
        varchar(50) code
        varchar(50) condition
        numeric length_km
        geometry LineString geom
        timestamp valid_from
        timestamp valid_to
        timestamp created_at
        timestamp updated_at
    }

    layer_attribute_definitions {
        integer id PK
        integer layer_id FK
        varchar(100) field_name
        varchar(255) display_name
        varchar(50) data_type
        varchar(50) unit
        text description
        boolean is_visible_in_tooltip
        integer sort_order
        timestamp created_at
        timestamp updated_at
    }
```

---

## 3. Entity Definitions

### 3.1 `layers`

Tabel metadata master untuk setiap layer yang tersedia di WebGIS.

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| `id` | `SERIAL` | `PRIMARY KEY` | Identifier layer. |
| `name` | `VARCHAR(255)` | `NOT NULL, UNIQUE` | Nama layer yang ditampilkan di UI/API. |
| `description` | `TEXT` | Nullable | Deskripsi isi layer. |
| `type` | `VARCHAR(50)` | `NOT NULL` | `base_map`, `administrative_boundary`, `infrastructure`, `custom`. |
| `geom_type` | `VARCHAR(50)` | `NOT NULL` | `Point`, `LineString`, `Polygon`, `MultiPolygon`, atau `NULL` untuk tile. |
| `srid` | `INTEGER` | `DEFAULT 4326` | Spatial Reference System Identifier. |
| `attribution` | `TEXT` | Nullable | Sumber data dan lisensi. |
| `is_public` | `BOOLEAN` | `DEFAULT TRUE` | Dapat diakses tanpa autentikasi pada Sprint 1. |
| `default_visibility` | `BOOLEAN` | `DEFAULT TRUE` | Layer aktif saat aplikasi pertama kali dimuat. |
| `z_index` | `INTEGER` | `DEFAULT 0` | Urutan rendering layer. |
| `created_at` | `TIMESTAMP` | `DEFAULT NOW()` | Waktu pembuatan. |
| `updated_at` | `TIMESTAMP` | `DEFAULT NOW()` | Waktu pembaruan terakhir. |

---

### 3.2 `admin_boundaries`

Tabel batas administrasi Indonesia untuk Provinsi, Kabupaten/Kota, Kecamatan, dan Desa.

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| `id` | `SERIAL` | `PRIMARY KEY` | Identifier feature. |
| `layer_id` | `INTEGER` | `FK -> layers.id` | Metadata layer batas administrasi. |
| `level` | `VARCHAR(50)` | `NOT NULL` | `province`, `regency`, `district`, `village`. |
| `code` | `VARCHAR(50)` | Nullable | Kode BPS/Kemendagri. |
| `name` | `VARCHAR(255)` | `NOT NULL` | Nama wilayah. |
| `parent_id` | `INTEGER` | `FK -> admin_boundaries.id` | Relasi hierarki, contoh: Kabupaten → Provinsi. |
| `province_name` | `VARCHAR(255)` | Nullable | Denormalisasi nama provinsi untuk performa. |
| `regency_name` | `VARCHAR(255)` | Nullable | Denormalisasi nama kabupaten/kota. |
| `district_name` | `VARCHAR(255)` | Nullable | Denormalisasi nama kecamatan. |
| `population` | `BIGINT` | Nullable | Jumlah penduduk. |
| `area_km2` | `NUMERIC` | Nullable | Luas wilayah dalam km². |
| `geom` | `GEOMETRY(MultiPolygon, 4326)` | `NOT NULL` | Geometri batas administrasi. |
| `valid_from` | `TIMESTAMP` | `DEFAULT NOW()` | Awal masa berlaku data. |
| `valid_to` | `TIMESTAMP` | Nullable | Akhir masa berlaku data; `NULL` berarti aktif. |
| `created_at` | `TIMESTAMP` | `DEFAULT NOW()` | Waktu import pertama. |
| `updated_at` | `TIMESTAMP` | `DEFAULT NOW()` | Waktu pembaruan terakhir. |

---

### 3.3 `infrastructure`

Tabel infrastruktur publik primer, mencakup Jalan Nasional, Jalur Kereta Api, Bandara, dan Bendungan.

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| `id` | `SERIAL` | `PRIMARY KEY` | Identifier feature. |
| `layer_id` | `INTEGER` | `FK -> layers.id` | Metadata layer infrastruktur. |
| `type` | `VARCHAR(100)` | `NOT NULL` | `national_road`, `railway`, `airport`, `dam`. |
| `name` | `VARCHAR(255)` | Nullable | Nama fasilitas. |
| `code` | `VARCHAR(50)` | Nullable | Kode identifikasi fasilitas. |
| `condition` | `VARCHAR(50)` | Nullable | `baik`, `sedang`, `rusak`. |
| `length_km` | `NUMERIC` | Nullable | Panjang fasilitas dalam kilometer. |
| `geom` | `GEOMETRY(LineString, 4326)` | `NOT NULL` | Geometri infrastruktur. |
| `valid_from` | `TIMESTAMP` | `DEFAULT NOW()` | Awal masa berlaku data. |
| `valid_to` | `TIMESTAMP` | Nullable | Akhir masa berlaku data; `NULL` berarti aktif. |
| `created_at` | `TIMESTAMP` | `DEFAULT NOW()` | Waktu import pertama. |
| `updated_at` | `TIMESTAMP` | `DEFAULT NOW()` | Waktu pembaruan terakhir. |

---

### 3.4 `layer_attribute_definitions`

Tabel metadata atribut untuk menampilkan tooltip dan dokumentasi API layer.

| Kolom | Tipe | Constraint | Keterangan |
|---|---|---|---|
| `id` | `SERIAL` | `PRIMARY KEY` | Identifier definisi atribut. |
| `layer_id` | `INTEGER` | `FK -> layers.id` | Layer pemilik atribut. |
| `field_name` | `VARCHAR(100)` | `NOT NULL` | Nama field data, contoh: `name`, `code`, `area_km2`. |
| `display_name` | `VARCHAR(255)` | `NOT NULL` | Label tampilan di tooltip, contoh: `Luas Wilayah`. |
| `data_type` | `VARCHAR(50)` | `NOT NULL` | `string`, `number`, `boolean`, `date`. |
| `unit` | `VARCHAR(50)` | Nullable | Satuan, contoh: `km²`, `km`, `jiwa`. |
| `description` | `TEXT` | Nullable | Deskripsi atribut. |
| `is_visible_in_tooltip` | `BOOLEAN` | `DEFAULT TRUE` | Menentukan field yang ditampilkan pada tooltip. |
| `sort_order` | `INTEGER` | `DEFAULT 0` | Urutan tampilan atribut. |
| `created_at` | `TIMESTAMP` | `DEFAULT NOW()` | Waktu pembuatan. |
| `updated_at` | `TIMESTAMP` | `DEFAULT NOW()` | Waktu pembaruan terakhir. |

---

## 4. Relasi dan Kardinalitas

| Relasi | Kardinalitas | Keterangan |
|---|---|---|
| `layers` → `admin_boundaries` | 1 → many | Satu metadata layer dapat memiliki banyak feature batas administrasi. |
| `layers` → `infrastructure` | 1 → many | Satu metadata layer dapat memiliki banyak feature infrastruktur. |
| `layers` → `layer_attribute_definitions` | 1 → many | Satu layer dapat memiliki banyak definisi atribut. |
| `admin_boundaries` → `admin_boundaries` | 1 → many | Relasi hierarki parent-child untuk wilayah administrasi. |

---

## 5. Indeks

### 5.1 `layers`

```sql
CREATE UNIQUE INDEX idx_layers_name_unique
    ON layers(name);

CREATE INDEX idx_layers_type
    ON layers(type);

CREATE INDEX idx_layers_default_visibility
    ON layers(default_visibility, z_index);
```

### 5.2 `admin_boundaries`

```sql
CREATE INDEX idx_admin_boundaries_geom
    ON admin_boundaries USING GIST (geom);

CREATE INDEX idx_admin_boundaries_level
    ON admin_boundaries(level);

CREATE INDEX idx_admin_boundaries_parent_id
    ON admin_boundaries(parent_id);

CREATE INDEX idx_admin_boundaries_province_name
    ON admin_boundaries(province_name);

CREATE INDEX idx_admin_boundaries_layer_id
    ON admin_boundaries(layer_id);

CREATE INDEX idx_admin_boundaries_active
    ON admin_boundaries(valid_to)
    WHERE valid_to IS NULL;

CREATE UNIQUE INDEX idx_admin_boundaries_layer_code_unique
    ON admin_boundaries(layer_id, code)
    WHERE code IS NOT NULL;
```

### 5.3 `infrastructure`

```sql
CREATE INDEX idx_infrastructure_geom
    ON infrastructure USING GIST (geom);

CREATE INDEX idx_infrastructure_type
    ON infrastructure(type);

CREATE INDEX idx_infrastructure_layer_id
    ON infrastructure(layer_id);

CREATE INDEX idx_infrastructure_active
    ON infrastructure(valid_to)
    WHERE valid_to IS NULL;

CREATE UNIQUE INDEX idx_infrastructure_layer_code_unique
    ON infrastructure(layer_id, code)
    WHERE code IS NOT NULL;
```

### 5.4 `layer_attribute_definitions`

```sql
CREATE UNIQUE INDEX idx_layer_attribute_definitions_layer_field_unique
    ON layer_attribute_definitions(layer_id, field_name);

CREATE INDEX idx_layer_attribute_definitions_tooltip
    ON layer_attribute_definitions(layer_id, sort_order)
    WHERE is_visible_in_tooltip = TRUE;
```

---

## 6. Integrity Constraints

```sql
ALTER TABLE layers
    ADD CONSTRAINT chk_layers_type
    CHECK (type IN ('base_map', 'administrative_boundary', 'infrastructure', 'custom'));

ALTER TABLE layers
    ADD CONSTRAINT chk_layers_geom_type
    CHECK (geom_type IN ('Point', 'LineString', 'Polygon', 'MultiPolygon') OR geom_type IS NULL);

ALTER TABLE layers
    ADD CONSTRAINT chk_layers_srid
    CHECK (srid = 4326 OR srid = 3857);

ALTER TABLE admin_boundaries
    ADD CONSTRAINT chk_admin_level
    CHECK (level IN ('province', 'regency', 'district', 'village'));

ALTER TABLE admin_boundaries
    ADD CONSTRAINT chk_admin_srid
    CHECK (ST_SRID(geom) = 4326);

ALTER TABLE admin_boundaries
    ADD CONSTRAINT chk_admin_valid_geometry
    CHECK (ST_IsValid(geom));

ALTER TABLE admin_boundaries
    ADD CONSTRAINT chk_admin_indonesia_extent
    CHECK (
        ST_XMin(geom) BETWEEN 95.3 AND 141.0
        AND ST_YMin(geom) BETWEEN -10.9 AND 5.9
    );

ALTER TABLE infrastructure
    ADD CONSTRAINT chk_infrastructure_type
    CHECK (type IN ('national_road', 'railway', 'airport', 'dam'));

ALTER TABLE infrastructure
    ADD CONSTRAINT chk_infrastructure_condition
    CHECK (condition IS NULL OR condition IN ('baik', 'sedang', 'rusak'));

ALTER TABLE infrastructure
    ADD CONSTRAINT chk_infrastructure_srid
    CHECK (ST_SRID(geom) = 4326);

ALTER TABLE infrastructure
    ADD CONSTRAINT chk_infrastructure_valid_geometry
    CHECK (ST_IsValid(geom));

ALTER TABLE infrastructure
    ADD CONSTRAINT chk_infrastructure_indonesia_extent
    CHECK (
        ST_XMin(geom) BETWEEN 95.3 AND 141.0
        AND ST_YMin(geom) BETWEEN -10.9 AND 5.9
    );

ALTER TABLE layer_attribute_definitions
    ADD CONSTRAINT chk_layer_attribute_data_type
    CHECK (data_type IN ('string', 'number', 'boolean', 'date'));
```

---

## 7. Catatan Desain

1. Semua geometri disimpan dalam `EPSG:4326` untuk konsistensi storage, API, dan validasi PostGIS.
2. OpenLayers me-render dalam `EPSG:3857`; transformasi koordinat dilakukan di sisi frontend.
3. `valid_to IS NULL` digunakan sebagai soft-delete dan penanda data aktif.
4. `parent_id` pada `admin_boundaries` mendukung hierarki wilayah administrasi.
5. `layer_attribute_definitions` dibuat sebagai tabel metadata agar tooltip tidak hardcode di frontend.
6. `layers` tetap menyimpan metadata base map, meskipun base map OSM tidak memiliki geometri PostGIS.
