# Entity Relationship Diagram (ERD) — WebGIS

**Basis Dokumen:** `docs/prd.md` v1.0, `docs/domain-model.md`
**Database:** PostgreSQL 15+ dengan PostGIS 3.3+
**Tanggal:** 2026-06-17

---

## 1. ERD Visualization

```
┌───────────────────────────────────────────────────────────────────────────┐
│                                                                           │
│                        ENTITY-RELATIONSHIP DIAGRAM                        │
│                        PostgreSQL + PostGIS                                │
│                                                                           │
└───────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────────┐
│  ┌──────────────────────────────────────────────────────────────────┐    │
│  │                        layers                                   │    │
│  │  ┌────────┬─────────────────────────────────────────────────┐   │    │
│  │  │ PK     │ id                  SERIAL              NOT NULL │   │    │
│  │  ├────────┼─────────────────────────────────────────────────┤   │    │
│  │  │ UK     │ name                VARCHAR(255)         NOT NULL│   │    │
│  │  │        │ description          TEXT                  NULL │   │    │
│  │  │        │ type                VARCHAR(50)           NOT NULL│   │    │
│  │  │        │ geom_type           VARCHAR(50)           NOT NULL│   │    │
│  │  │        │ srid                INTEGER               NOT NULL│   │    │
│  │  │        │ attribution         TEXT                   NULL │   │    │
│  │  │        │ is_public           BOOLEAN               DEFAULT│   │    │
│  │  │        │ default_visibility  BOOLEAN               DEFAULT│   │    │
│  │  │        │ z_index             INTEGER               DEFAULT│   │    │
│  │  │        │ created_at          TIMESTAMP             DEFAULT│   │    │
│  │  │        │ updated_at          TIMESTAMP             DEFAULT│   │    │
│  │  └────────┴─────────────────────────────────────────────────┘   │    │
│  │                          │                                         │    │
│  │                          │ 1 ──────── n                            │    │
│  └──────────────────────────┼────────────────────────────────────────┘    │
│                             │                                              │
│                             │ FK: layer_id                                │
│                             ▼                                              │
│  ┌──────────────────────────────────────────────────────────────────┐    │
│  │                   admin_boundaries                               │    │
│  │  ┌────────┬─────────────────────────────────────────────────┐   │    │
│  │  │ PK     │ id                  SERIAL              NOT NULL │   │    │
│  │  ├────────┼─────────────────────────────────────────────────┤   │    │
│  │  │ FK     │ layer_id            INTEGER               NOT NULL│   │    │
│  │  │ UK     │ (layer_id, code)                              │   │    │
│  │  │        │ level               VARCHAR(50)           NOT NULL│   │    │
│  │  │        │ code                VARCHAR(50)           NULL   │   │    │
│  │  │ UK     │ name                VARCHAR(255)          NOT NULL│   │    │
│  │  │ FK     │ parent_id           INTEGER               NULL   │   │    │
│  │  │        │ province_name       VARCHAR(255)          NULL   │   │    │
│  │  │        │ regency_name        VARCHAR(255)          NULL   │   │    │
│  │  │        │ district_name       VARCHAR(255)          NULL   │   │    │
│  │  │        │ population          BIGINT                NULL   │   │    │
│  │  │        │ area_km2            NUMERIC               NULL   │   │    │
│  │  │        │ geom                GEOMETRY(MultiPolygon,│   │    │
│  │  │        │                     4326)                 NOT NULL│   │    │
│  │  │        │ valid_from          TIMESTAMP             DEFAULT │   │    │
│  │  │        │ valid_to            TIMESTAMP             NULL   │   │    │
│  │  │        │ created_at          TIMESTAMP             DEFAULT │   │    │
│  │  │        │ updated_at          TIMESTAMP             DEFAULT │   │    │
│  │  └────────┴─────────────────────────────────────────────────┘   │    │
│  │                          ▲                                         │    │
│  │                          │ self-reference FK                      │    │
│  └──────────────────────────┼────────────────────────────────────────┘    │
│                             │                                              │
│                             │ FK: layer_id                                │
│                             ▼                                              │
│  ┌──────────────────────────────────────────────────────────────────┐    │
│  │                     infrastructure                                │    │
│  │  ┌────────┬─────────────────────────────────────────────────┐   │    │
│  │  │ PK     │ id                  SERIAL              NOT NULL │   │    │
│  │  ├────────┼─────────────────────────────────────────────────┤   │    │
│  │  │ FK     │ layer_id            INTEGER               NOT NULL│   │    │
│  │  │ UK     │ (layer_id, code)                              │   │    │
│  │  │        │ type                VARCHAR(100)          NOT NULL│   │    │
│  │  │        │ name                VARCHAR(255)          NULL   │   │    │
│  │  │        │ code                VARCHAR(50)           NULL   │   │    │
│  │  │        │ condition           VARCHAR(50)           NULL   │   │    │
│  │  │        │ length_km           NUMERIC               NULL   │   │    │
│  │  │        │ geom                GEOMETRY(LineString,  │   │    │
│  │  │        │                     4326)                 NOT NULL│   │    │
│  │  │        │ valid_from          TIMESTAMP             DEFAULT │   │    │
│  │  │        │ valid_to            TIMESTAMP             NULL   │   │    │
│  │  │        │ created_at          TIMESTAMP             DEFAULT │   │    │
│  │  │        │ updated_at          TIMESTAMP             DEFAULT │   │    │
│  │  └────────┴─────────────────────────────────────────────────┘   │    │
│  └──────────────────────────────────────────────────────────────────┘    │
│                                                                           │
│  ─────────────────────────────────────────────────────────────────────── │
│  TAMBAHAN UNTUK KEPERLUAN ROADMAP (Berdasarkan Gap Analysis)             │
│  ─────────────────────────────────────────────────────────────────────── │
│                                                                           │
│  ┌──────────────────────────────────────────────────────────────────┐    │
│  │                layer_attribute_definitions                        │    │
│  │  Metadata definisi atribut untuk setiap layer                      │    │
│  │  ┌────────┬─────────────────────────────────────────────────┐   │    │
│  │  │ PK     │ id                  SERIAL              NOT NULL │   │    │
│  │  ├────────┼─────────────────────────────────────────────────┤   │    │
│  │  │ FK     │ layer_id            INTEGER               NOT NULL│   │    │
│  │  │ UK     │ (layer_id, field_name)                         │   │    │
│  │  │        │ field_name          VARCHAR(100)          NOT NULL│   │    │
│  │  │        │ display_name        VARCHAR(255)          NOT NULL│   │    │
│  │  │        │ data_type           VARCHAR(50)           NOT NULL│   │    │
│  │  │        │ unit                VARCHAR(50)           NULL   │   │    │
│  │  │        │ description         TEXT                   NULL   │   │    │
│  │  │        │ is_visible_in_tooltip BOOLEAN             DEFAULT │   │    │
│  │  │        │ sort_order          INTEGER               DEFAULT │   │    │
│  │  │        │ created_at          TIMESTAMP             DEFAULT │   │    │
│  │  │        │ updated_at          TIMESTAMP             DEFAULT │   │    │
│  │  └────────┴─────────────────────────────────────────────────┘   │    │
│  └──────────────────────────────────────────────────────────────────┘    │
│                                                                           │
└────────────────────────────────────────────────────────────────────────────┘

```

---

## 2. Table Specifications

### 2.1 Core Tables (Sprint 1)

#### 2.1.1 `layers`

**Description:** Metadata master untuk setiap layer peta yang tersedia di sistem.

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `id` | SERIAL | PRIMARY KEY | Auto-increment identifier. |
| `name` | VARCHAR(255) | NOT NULL, UNIQUE | Display name layer yang ditampilkan di UI. |
| `description` | TEXT | NULL | Deskripsi lengkap isi dan sumber data layer. |
| `type` | VARCHAR(50) | NOT NULL | ENUM: `base_map`, `administrative_boundary`, `infrastructure`, `custom`. |
| `geom_type` | VARCHAR(50) | NOT NULL | ENUM: `Point`, `LineString`, `Polygon`, `MultiPolygon`. |
| `srid` | INTEGER | NOT NULL DEFAULT 4326 | Spatial Reference System Identifier. |
| `attribution` | TEXT | NULL | Sumber data dan license text untuk ditampilkan di UI. |
| `is_public` | BOOLEAN | DEFAULT TRUE | Visibility untuk user tanpa login (Sprint 1: semua public). |
| `default_visibility` | BOOLEAN | DEFAULT TRUE | Status visible saat peta pertama kali di-load. |
| `z_index` | INTEGER | DEFAULT 0 | Urutan rendering layer (0 = paling bawah). |
| `created_at` | TIMESTAMP | DEFAULT NOW() | Timestamp pembuatan record. |
| `updated_at` | TIMESTAMP | DEFAULT NOW() | Timestamp pembaruan terakhir. |

**Indexes:**
```sql
CREATE UNIQUE INDEX idx_layers_name_unique ON layers(name);
CREATE INDEX idx_layers_type ON layers(type);
```

---

#### 2.1.2 `admin_boundaries`

**Description:** Menyimpan data batas administrasi Indonesia (provinsi, kabupaten/kota, kecamatan, desa).

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `id` | SERIAL | PRIMARY KEY | Auto-increment identifier. |
| `layer_id` | INTEGER | FK → layers(id) ON DELETE CASCADE | Referensi ke metadata layer. |
| `level` | VARCHAR(50) | NOT NULL | ENUM: `province`, `regency`, `district`, `village`. |
| `code` | VARCHAR(50) | NULL | Kode administratif (Kemendagri/BPS). |
| `name` | VARCHAR(255) | NOT NULL | Nama wilayah administrasi. |
| `parent_id` | INTEGER | FK → admin_boundaries(id) ON DELETE SET NULL | Untuk hierarki (misal: kabupaten → provinsi). |
| `province_name` | VARCHAR(255) | NULL | Denormalisasi untuk performa queryfilter. |
| `regency_name` | VARCHAR(255) | NULL | Denormalisasi untuk performa query filter. |
| `district_name` | VARCHAR(255) | NULL | Denormalisasi untuk performa query filter. |
| `population` | BIGINT | NULL | Jumlah penduduk (data dari BPS). |
| `area_km2` | NUMERIC | NULL | Luas wilayah dalam km². |
| `geom` | GEOMETRY(MultiPolygon, 4326) | NOT NULL | Geometri batas administrasi. |
| `valid_from` | TIMESTAMP | DEFAULT NOW() | Mulai berlaku data (untuk versioning). |
| `valid_to` | TIMESTAMP | NULL | Berakhir berlaku data (null jika masih berlaku). |
| `created_at` | TIMESTAMP | DEFAULT NOW() | Timestamp import pertama. |
| `updated_at` | TIMESTAMP | DEFAULT NOW() | Timestamp pembaruan terakhir. |

**Indexes:**
```sql
-- Spatial Index (Wajib untuk performa query spasial)
CREATE INDEX idx_admin_boundaries_geom ON admin_boundaries USING GIST (geom);

-- Non-Spatial Index untuk filter sering digunakan
CREATE INDEX idx_admin_boundaries_level ON admin_boundaries (level);
CREATE INDEX idx_admin_boundaries_province ON admin_boundaries (province_name);
CREATE INDEX idx_admin_boundaries_layer_id ON admin_boundaries (layer_id);
```

**Integrity Constraints:**
```sql
-- Validasi SRID
ALTER TABLE admin_boundaries 
  ADD CONSTRAINT chk_admin_srid CHECK (ST_SRID(geom) = 4326);

-- Validasi Geometri
ALTER TABLE admin_boundaries 
  ADD CONSTRAINT chk_admin_valid CHECK (ST_IsValid(geom));

-- Validasi Koordinat Indonesia (opsional tapi disarankan)
ALTER TABLE admin_boundaries 
  ADD CONSTRAINT chk_admin_bbox CHECK (
    ST_XMin(geom) BETWEEN 95.3 AND 141.0 AND
    ST_YMin(geom) BETWEEN -10.9 AND 5.9
  );

-- Unique constraint untuk mencegah duplikasi
CREATE UNIQUE INDEX idx_admin_boundaries_layer_code 
  ON admin_boundaries(layer_id, code) 
  WHERE code IS NOT NULL;
```

---

#### 2.1.3 `infrastructure`

**Description:** Menyimpan data infrastruktur publik primer (jalan, kereta api, bandara, bendungan).

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `id` | SERIAL | PRIMARY KEY | Auto-increment identifier. |
| `layer_id` | INTEGER | FK → layers(id) ON DELETE CASCADE | Referensi ke metadata layer. |
| `type` | VARCHAR(100) | NOT NULL | ENUM: `national_road`, `railway`, `airport`, `dam`. |
| `name` | VARCHAR(255) | NULL | Nama fasilitas (misal: "Jalan Tol Jakarta-Cikampek"). |
| `code` | VARCHAR(50) | NULL | Kode identifikasi (opsional). |
| `condition` | VARCHAR(50) | NULL | Kondisi: `baik`, `sedang`, `rusak`. |
| `length_km` | NUMERIC | NULL | Panjang fasilitas dalam kilometer. |
| `geom` | GEOMETRY(LineString, 4326) | NOT NULL | Geometri garis/point infrastruktur. |
| `valid_from` | TIMESTAMP | DEFAULT NOW() | Mulai berlaku data. |
| `valid_to` | TIMESTAMP | NULL | Berakhir berlaku data. |
| `created_at` | TIMESTAMP | DEFAULT NOW() | Timestamp import pertama. |
| `updated_at` | TIMESTAMP | DEFAULT NOW() | Timestamp pembaruan terakhir. |

**Indexes:**
```sql
-- Spatial Index
CREATE INDEX idx_infrastructure_geom ON infrastructure USING GIST (geom);

-- Non-Spatial Index
CREATE INDEX idx_infrastructure_type ON infrastructure (type);
CREATE INDEX idx_infrastructure_layer_id ON infrastructure (layer_id);
```

**Integrity Constraints:**
```sql
-- Validasi SRID
ALTER TABLE infrastructure 
  ADD CONSTRAINT chk_infra_srid CHECK (ST_SRID(geom) = 4326);

-- Validasi Geometri
ALTER TABLE infrastructure 
  ADD CONSTRAINT chk_infra_valid CHECK (ST_IsValid(geom));

-- Unique constraint
CREATE UNIQUE INDEX idx_infrastructure_layer_code 
  ON infrastructure(layer_id, code) 
  WHERE code IS NOT NULL;
```

---

### 2.2 Supporting Tables (Roadmap / Sprint 2+)

#### 2.2.1 `layer_attribute_definitions`

**Description:** Metadata definisi atribut untuk setiap layer. Digunakan untuk tooltip dynamic dan form input di masa depan.

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `id` | SERIAL | PRIMARY KEY | Auto-increment identifier. |
| `layer_id` | INTEGER | FK → layers(id) ON DELETE CASCADE | Referensi ke layer. |
| `field_name` | VARCHAR(100) | NOT NULL | Nama field di JSONB properties. |
| `display_name` | VARCHAR(255) | NOT NULL | Nama yang ditampilkan di UI (misal: "Luas Wilayah"). |
| `data_type` | VARCHAR(50) | NOT NULL | ENUM: `string`, `number`, `boolean`, `date`. |
| `unit` | VARCHAR(50) | NULL | Satuan (misal: `km²`, `km`, `m`). |
| `description` | TEXT | NULL | Deskripsi atribut untuk tooltip atau help text. |
| `is_visible_in_tooltip` | BOOLEAN | DEFAULT TRUE | Tampilkan di tooltip saat hover. |
| `sort_order` | INTEGER | DEFAULT 0 | Urutan tampil di tooltip. |
| `created_at` | TIMESTAMP | DEFAULT NOW() | - |
| `updated_at` | TIMESTAMP | DEFAULT NOW() | - |

**Unique Constraint:**
```sql
CREATE UNIQUE INDEX idx_layer_attr_unique 
  ON layer_attribute_definitions(layer_id, field_name);
```

---

#### 2.2.2 `users` *(Sprint 2 — Authentication)*

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `user_id` | UUID | PRIMARY KEY DEFAULT gen_random_uuid() | Identifikasi unik pengguna. |
| `username` | VARCHAR(100) | NOT NULL, UNIQUE | Nama pengguna untuk login. |
| `email` | VARCHAR(255) | NOT NULL, UNIQUE | Alamat email institusional. |
| `full_name` | VARCHAR(255) | NULL | Nama lengkap. |
| `password_hash` | VARCHAR(255) | NOT NULL | Hash password (bcrypt/argon2). |
| `role_id` | INTEGER | FK → roles(id) | Role pengguna. |
| `is_active` | BOOLEAN | DEFAULT TRUE | Status akun. |
| `last_login` | TIMESTAMP | NULL | Waktu login terakhir. |
| `created_at` | TIMESTAMP | DEFAULT NOW() | - |
| `updated_at` | TIMESTAMP | DEFAULT NOW() | - |

---

#### 2.2.3 `roles` *(Sprint 2)*

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `role_id` | SERIAL | PRIMARY KEY | - |
| `name` | VARCHAR(50) | NOT NULL, UNIQUE | `GISOfficer`, `SystemAdministrator`, `Viewer`. |
| `description` | TEXT | NULL | Deskripsi tanggung jawab. |

---

#### 2.2.4 `data_import_logs` *(Sprint 2 — Audit)*

| Column | Data Type | Constraints | Description |
|--------|-----------|-------------|-------------|
| `import_id` | SERIAL | PRIMARY KEY | - |
| `layer_id` | INTEGER | FK → layers(id) | Layer yang di-import. |
| `file_name` | VARCHAR(255) | NOT NULL | Nama file asli. |
| `file_format` | VARCHAR(50) | NOT NULL | `shapefile`, `geojson`, `csv`. |
| `record_count` | INTEGER | NOT NULL | Jumlah features yang di-import. |
| `status` | VARCHAR(50) | NOT NULL | `success`, `partial`, `failed`. |
| `error_message` | TEXT | NULL | Detail error jika gagal. |
| `imported_by` | UUID | FK → users(user_id) | User yang melakukan import. |
| `started_at` | TIMESTAMP | DEFAULT NOW() | - |
| `finished_at` | TIMESTAMP | NULL | - |

---

## 3. Relationship Summary

| Table A | Table B | Relationship | Cardinality | Foreign Key |
|---------|---------|--------------|-------------|-------------|
| `layers` | `admin_boundaries` | Parent-Child | 1 → n | `admin_boundaries.layer_id` → `layers.id` |
| `layers` | `infrastructure` | Parent-Child | 1 → n | `infrastructure.layer_id` → `layers.id` |
| `admin_boundaries` | `admin_boundaries` | Self-Reference (Hierarchy) | 1 → n | `admin_boundaries.parent_id` → `admin_boundaries.id` |
| `layers` | `layer_attribute_definitions` | Parent-Child | 1 → n | `layer_attribute_definitions.layer_id` → `layers.id` |
| `users` | `roles` | Many-to-One | n → 1 | `users.role_id` → `roles.id` |
| `layers` | `data_import_logs` | Parent-Child | 1 → n | `data_import_logs.layer_id` → `layers.id` |
| `users` | `data_import_logs` | Parent-Child | 1 → n | `data_import_logs.imported_by` → `users.user_id` |

---

## 4. Complete SQL Schema

```sql
-- ============================================================================
-- WebGIS Database Schema
-- PostgreSQL 15+ with PostGIS 3.3+
-- ============================================================================

-- Enable PostGIS extension
CREATE EXTENSION IF NOT EXISTS postgis;

-- Enable UUID extension (untuk user ID)
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- ============================================================================
-- TABLE: layers
-- Metadata master untuk setiap layer peta
-- ============================================================================
CREATE TABLE layers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    type VARCHAR(50) NOT NULL CHECK (type IN (
        'base_map', 
        'administrative_boundary', 
        'infrastructure', 
        'custom'
    )),
    geom_type VARCHAR(50) NOT NULL CHECK (geom_type IN (
        'Point', 
        'LineString', 
        'Polygon', 
        'MultiPolygon'
    )),
    srid INTEGER NOT NULL DEFAULT 4326,
    attribution TEXT,
    is_public BOOLEAN DEFAULT TRUE,
    default_visibility BOOLEAN DEFAULT TRUE,
    z_index INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Indexes untuk layers
CREATE UNIQUE INDEX idx_layers_name_unique ON layers(name);
CREATE INDEX idx_layers_type ON layers(type);

-- Trigger untuk updated_at
CREATE TRIGGER trg_layers_updated_at 
  BEFORE UPDATE ON layers 
  FOR EACH ROW EXECUTE FUNCTION update_modified_column();

-- ============================================================================
-- TABLE: admin_boundaries
-- Data batas administrasi Indonesia
-- ============================================================================
CREATE TABLE admin_boundaries (
    id SERIAL PRIMARY KEY,
    layer_id INTEGER NOT NULL REFERENCES layers(id) ON DELETE CASCADE,
    level VARCHAR(50) NOT NULL CHECK (level IN (
        'province', 
        'regency', 
        'district', 
        'village'
    )),
    code VARCHAR(50),
    name VARCHAR(255) NOT NULL,
    parent_id INTEGER REFERENCES admin_boundaries(id) ON DELETE SET NULL,
    province_name VARCHAR(255),
    regency_name VARCHAR(255),
    district_name VARCHAR(255),
    population BIGINT,
    area_km2 NUMERIC,
    geom GEOMETRY(MultiPolygon, 4326) NOT NULL,
    valid_from TIMESTAMP DEFAULT NOW(),
    valid_to TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Spatial Index (Wajib untuk performa query spasial)
CREATE INDEX idx_admin_boundaries_geom ON admin_boundaries USING GIST (geom);

-- Non-Spatial Index
CREATE INDEX idx_admin_boundaries_level ON admin_boundaries (level);
CREATE INDEX idx_admin_boundaries_province ON admin_boundaries (province_name);
CREATE INDEX idx_admin_boundaries_layer_id ON admin_boundaries (layer_id);
CREATE INDEX idx_admin_boundaries_valid_to ON admin_boundaries (valid_to) 
  WHERE valid_to IS NULL;

-- Integrity Constraints
ALTER TABLE admin_boundaries 
  ADD CONSTRAINT chk_admin_srid CHECK (ST_SRID(geom) = 4326);
ALTER TABLE admin_boundaries 
  ADD CONSTRAINT chk_admin_valid CHECK (ST_IsValid(geom));
ALTER TABLE admin_boundaries 
  ADD CONSTRAINT chk_admin_bbox CHECK (
    ST_XMin(geom) BETWEEN 95.3 AND 141.0 AND
    ST_YMin(geom) BETWEEN -10.9 AND 5.9
  );

-- Unique constraint untuk mencegah duplikasi
CREATE UNIQUE INDEX idx_admin_boundaries_layer_code 
  ON admin_boundaries(layer_id, code) 
  WHERE code IS NOT NULL;

-- Trigger updated_at
CREATE TRIGGER trg_admin_boundaries_updated_at 
  BEFORE UPDATE ON admin_boundaries 
  FOR EACH ROW EXECUTE FUNCTION update_modified_column();

-- ============================================================================
-- TABLE: infrastructure
-- Data infrastruktur publik (jalan, kereta api, bandara, bendungan)
-- ============================================================================
CREATE TABLE infrastructure (
    id SERIAL PRIMARY KEY,
    layer_id INTEGER NOT NULL REFERENCES layers(id) ON DELETE CASCADE,
    type VARCHAR(100) NOT NULL CHECK (type IN (
        'national_road', 
        'railway', 
        'airport', 
        'dam'
    )),
    name VARCHAR(255),
    code VARCHAR(50),
    condition VARCHAR(50) CHECK (condition IN ('baik', 'sedang', 'rusak')),
    length_km NUMERIC,
    geom GEOMETRY(LineString, 4326) NOT NULL,
    valid_from TIMESTAMP DEFAULT NOW(),
    valid_to TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Spatial Index
CREATE INDEX idx_infrastructure_geom ON infrastructure USING GIST (geom);

-- Non-Spatial Index
CREATE INDEX idx_infrastructure_type ON infrastructure (type);
CREATE INDEX idx_infrastructure_layer_id ON infrastructure (layer_id);
CREATE INDEX idx_infrastructure_valid_to ON infrastructure (valid_to) 
  WHERE valid_to IS NULL;

-- Integrity Constraints
ALTER TABLE infrastructure 
  ADD CONSTRAINT chk_infra_srid CHECK (ST_SRID(geom) = 4326);
ALTER TABLE infrastructure 
  ADD CONSTRAINT chk_infra_valid CHECK (ST_IsValid(geom));
ALTER TABLE infrastructure 
  ADD CONSTRAINT chk_infra_bbox CHECK (
    ST_XMin(geom) BETWEEN 95.3 AND 141.0 AND
    ST_YMin(geom) BETWEEN -10.9 AND 5.9
  );

-- Unique constraint
CREATE UNIQUE INDEX idx_infrastructure_layer_code 
  ON infrastructure(layer_id, code) 
  WHERE code IS NOT NULL;

-- Trigger updated_at
CREATE TRIGGER trg_infrastructure_updated_at 
  BEFORE UPDATE ON infrastructure 
  FOR EACH ROW EXECUTE FUNCTION update_modified_column();

-- ============================================================================
-- TABLE: layer_attribute_definitions (Roadmap - Sprint 2+)
-- Metadata definisi atribut untuk setiap layer
-- ============================================================================
CREATE TABLE layer_attribute_definitions (
    id SERIAL PRIMARY KEY,
    layer_id INTEGER NOT NULL REFERENCES layers(id) ON DELETE CASCADE,
    field_name VARCHAR(100) NOT NULL,
    display_name VARCHAR(255) NOT NULL,
    data_type VARCHAR(50) NOT NULL CHECK (data_type IN (
        'string', 
        'number', 
        'boolean', 
        'date'
    )),
    unit VARCHAR(50),
    description TEXT,
    is_visible_in_tooltip BOOLEAN DEFAULT TRUE,
    sort_order INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE UNIQUE INDEX idx_layer_attr_unique 
  ON layer_attribute_definitions(layer_id, field_name);

CREATE TRIGGER trg_layer_attr_updated_at 
  BEFORE UPDATE ON layer_attribute_definitions 
  FOR EACH ROW EXECUTE FUNCTION update_modified_column();

-- ============================================================================
-- TABLE: users (Roadmap - Sprint 2+)
-- ============================================================================
CREATE TABLE users (
    user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(100) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL UNIQUE,
    full_name VARCHAR(255),
    password_hash VARCHAR(255) NOT NULL,
    role_id INTEGER NOT NULL REFERENCES roles(role_id),
    is_active BOOLEAN DEFAULT TRUE,
    last_login TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TRIGGER trg_users_updated_at 
  BEFORE UPDATE ON users 
  FOR EACH ROW EXECUTE FUNCTION update_modified_column();

-- ============================================================================
-- TABLE: roles (Roadmap - Sprint 2+)
-- ============================================================================
CREATE TABLE roles (
    role_id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE CHECK (name IN (
        'GISOfficer', 
        'SystemAdministrator', 
        'Viewer'
    )),
    description TEXT
);

-- ============================================================================
-- TABLE: data_import_logs (Roadmap - Sprint 2+)
-- ============================================================================
CREATE TABLE data_import_logs (
    import_id SERIAL PRIMARY KEY,
    layer_id INTEGER NOT NULL REFERENCES layers(id),
    file_name VARCHAR(255) NOT NULL,
    file_format VARCHAR(50) NOT NULL,
    record_count INTEGER NOT NULL,
    status VARCHAR(50) NOT NULL CHECK (status IN (
        'success', 
        'partial', 
        'failed'
    )),
    error_message TEXT,
    imported_by UUID REFERENCES users(user_id),
    started_at TIMESTAMP DEFAULT NOW(),
    finished_at TIMESTAMP
);

CREATE INDEX idx_import_logs_layer_id ON data_import_logs(layer_id);
CREATE INDEX idx_import_logs_status ON data_import_logs(status);

-- ============================================================================
-- HELPER FUNCTION: updated_at trigger
-- ============================================================================
CREATE OR REPLACE FUNCTION update_modified_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- ============================================================================
-- VIEW: v_active_layers ( untuk query cepat layer yang aktif )
-- ============================================================================
CREATE OR REPLACE VIEW v_active_layers AS
SELECT 
    l.id,
    l.name,
    l.description,
    l.type,
    l.geom_type,
    l.srid,
    l.attribution,
    l.z_index,
    l.default_visibility
FROM layers l
WHERE l.is_public = TRUE
  AND l.default_visibility = TRUE
ORDER BY l.z_index ASC;

-- ============================================================================
-- VIEW: v_admin_boundaries_hierarchy ( untuk query hierarki admin )
-- ============================================================================
CREATE OR REPLACE VIEW v_admin_boundaries_hierarchy AS
SELECT 
    a.id,
    a.layer_id,
    a.level,
    a.code,
    a.name,
    a.parent_id,
    a.province_name,
    a.regency_name,
    a.district_name,
    a.population,
    a.area_km2,
    a.geom,
    a.valid_from,
    a.valid_to,
    p.name AS parent_name
FROM admin_boundaries a
LEFT JOIN admin_boundaries p ON a.parent_id = p.id
WHERE a.valid_to IS NULL;

-- ============================================================================
-- VIEW: v_infrastructure_active ( untuk query infrastruktur aktif )
-- ============================================================================
CREATE OR REPLACE VIEW v_infrastructure_active AS
SELECT 
    i.id,
    i.layer_id,
    i.type,
    i.name,
    i.code,
    i.condition,
    i.length_km,
    i.geom,
    l.name AS layer_name,
    l.attribution
FROM infrastructure i
JOIN layers l ON i.layer_id = l.id
WHERE i.valid_to IS NULL
  AND l.is_public = TRUE;

-- ============================================================================
-- SAMPLE DATA: Insert base layers
-- ============================================================================
INSERT INTO layers (name, description, type, geom_type, srid, attribution, default_visibility, z_index) 
VALUES 
    ('Batas Administrasi Indonesia', 
     'Layer batas provinsi, kabupaten/kota, kecamatan, dan desa seluruh Indonesia', 
     'administrative_boundary', 
     'MultiPolygon', 
     4326, 
     '© Badan Pusat Statistik / Kemendagri', 
     TRUE, 
     1),
    ('Infrastruktur Publik', 
     'Layer infrastruktur publik: Jalan Nasional, Jalur Kereta Api, Bandara, Bendungan', 
     'infrastructure', 
     'LineString', 
     4326, 
     '© OpenStreetMap contributors / Kementerian PUPR', 
     TRUE, 
     2);

-- ============================================================================
-- GRANTS ( sesuaikan dengan user aplikasi )
-- ============================================================================
-- CREATE USER webgis_app WITH PASSWORD 'secure_password';
-- GRANT CONNECT ON DATABASE webgis TO webgis_app;
-- GRANT USAGE ON SCHEMA public TO webgis_app;
-- GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO webgis_app;
-- GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO webgis_app;
-- GRANT EXECUTE ON FUNCTION update_modified_column() TO webgis_app;
```

---

## 5. ERD Diagram (Mermaid Format)

Untuk dokumentasi yang mendukung Mermaid (GitHub, GitLab, Notion), gunakan diagram berikut:

```mermaid
erDiagram
    layers ||--o{ admin_boundaries : contains
    layers ||--o{ infrastructure : contains
    layers ||--o{ layer_attribute_definitions : defines
    admin_boundaries ||--o{ admin_boundaries : "parent-child"
    users ||--|| roles : has
    layers ||--o{ data_import_logs : tracks
    
    layers {
        serial id PK
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
        serial id PK
        integer layer_id FK
        varchar(50) level
        varchar(50) code UK
        varchar(255) name
        integer parent_id FK
        varchar(255) province_name
        varchar(255) regency_name
        varchar(255) district_name
        bigint population
        numeric area_km2
        geometry(MultiPolygon,4326) geom
        timestamp valid_from
        timestamp valid_to
        timestamp created_at
        timestamp updated_at
    }
    
    infrastructure {
        serial id PK
        integer layer_id FK
        varchar(100) type
        varchar(255) name
        varchar(50) code
        varchar(50) condition
        numeric length_km
        geometry(LineString,4326) geom
        timestamp valid_from
        timestamp valid_to
        timestamp created_at
        timestamp updated_at
    }
    
    layer_attribute_definitions {
        serial id PK
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
    
    users {
        uuid user_id PK
        varchar(100) username UK
        varchar(255) email UK
        varchar(255) full_name
        varchar(255) password_hash
        integer role_id FK
        boolean is_active
        timestamp last_login
        timestamp created_at
        timestamp updated_at
    }
    
    roles {
        serial role_id PK
        varchar(50) name UK
        text description
    }
    
    data_import_logs {
        serial import_id PK
        integer layer_id FK
        varchar(255) file_name
        varchar(50) file_format
        integer record_count
        varchar(50) status
        text error_message
        uuid imported_by FK
        timestamp started_at
        timestamp finished_at
    }
```

---

## 6. Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        DATA FLOW: Query Spasial                          │
└─────────────────────────────────────────────────────────────────────────┘

┌──────────────┐     ┌──────────────┐     ┌──────────────────────────────┐
│   Frontend   │────▶│   Backend    │────▶│        PostgreSQL            │
│  (Browser)   │     │   (Golang    │     │         + PostGIS            │
│              │     │    Fiber)    │     │                              │
│  ┌────────┐ │     │  ┌────────┐  │     │  ┌────────────────────────┐  │
│  │ Map    │ │     │  │ Vector │  │     │  │  Spatial Query Engine  │  │
│  │ View   │ │     │  │Handler │  │     │  │  (PostGIS Functions)   │  │
│  └───┬────┘ │     │  └───┬────┘  │     │  └───────────┬────────────┘  │
│      │      │     │      │       │     │              │               │
│      │ bbox │     │      │       │     │              │               │
│      │ zoom │     │      │ bbox  │     │  ┌───────────▼────────────┐  │
│      │ layer│     │      │ zoom  │     │  │                       │  │
│      ▼      │     │      ▼       │     │  │   1. ST_Intersects     │  │
│  ┌────────┐ │     │  ┌────────┐  │     │  │   2. ST_Transform      │  │
│  │Fetch   │ │     │  │PostGIS │  │     │  │   3. ST_Simplify       │  │
│  │API     │ │◀────│  │Service │  │◀────│  │                       │  │
│  └────────┘ │     │  └────────┘  │     │  └───────────────────────┘  │
│              │     │              │     │                              │
│  ┌────────┐ │     │              │     │  ┌────────────────────────┐  │
│  │Tooltip │ │     │              │     │  │                       │  │
│  │Render  │ │     │              │     │  │  Tables:               │  │
│  └────────┘ │     │              │     │  │  • layers              │  │
│              │     │              │     │  │  • admin_boundaries    │  │
│              │     │              │     │  │  • infrastructure      │  │
└──────────────┘     └──────────────┘     └──────────────────────────────┘

```

---

## 7. Indexing Strategy

| Table | Index Type | Columns | Purpose |
|-------|-----------|---------|---------|
| `admin_boundaries` | GIST | `geom` | Spatial query: ST_Intersects, ST_Within |
| `infrastructure` | GIST | `geom` | Spatial query: ST_Intersects, ST_Within |
| `admin_boundaries` | B-Tree | `level` | Filter by administrasi level (provinsi/kabupaten) |
| `admin_boundaries` | B-Tree | `province_name` | Filter by province name |
| `infrastructure` | B-Tree | `type` | Filter by infrastruktur type (road/railway) |
| `admin_boundaries` | B-Tree | `layer_id` | Join dengan tabel layers |
| `infrastructure` | B-Tree | `layer_id` | Join dengan tabel layers |
| `layer_attribute_definitions` | B-Tree | `(layer_id, field_name)` | Unique constraint + lookup |
| `users` | B-Tree | `email` | Login lookup |
| `users` | B-Tree | `username` | Login lookup |
| `data_import_logs` | B-Tree | `layer_id` | Audit trail per layer |
| `data_import_logs` | B-Tree | `status` | Filter import errors |

---

## 8. Maintenance Considerations

### 8.1 Vacuum & Analyze

```sql
-- Jadwalkan vacuum analyse secara periodik untuk tabel spasial besar
VACUUM ANALYZE admin_boundaries;
VACUUM ANALYZE infrastructure;
```

### 8.2 Geometry Validation Query

```sql
-- Cek integritas geometri secara berkala
SELECT 
    id, 
    ST_IsValid(geom) AS is_valid, 
    ST_IsValidReason(geom) AS invalid_reason
FROM admin_boundaries
WHERE NOT ST_IsValid(geom);

SELECT 
    id, 
    ST_IsValid(geom) AS is_valid, 
    ST_IsValidReason(geom) AS invalid_reason
FROM infrastructure
WHERE NOT ST_IsValid(geom);
```

### 8.3 Data Retention Policy

```sql
-- Contoh: Arsip data yang sudah tidak berlaku > 2 tahun
-- (Opsional, untuk table historical versioning)
INSERT INTO admin_boundaries_archive 
SELECT * FROM admin_boundaries 
WHERE valid_to < NOW() - INTERVAL '2 years';

DELETE FROM admin_boundaries 
WHERE valid_to < NOW() - INTERVAL '2 years';
```

---

## 9. Catatan untuk Development Team

1. **GIST Index adalah Wajib:** Tanpa GIST index pada kolom `geom`, query `ST_Intersects` pada dataset besar (> 100.000 features) akan sangat lambat (> 5 detik).

2. **SRID Konsisten:** SelaluNomor 4326 untuk storage. Backend bertanggung jawab transformasi ke EPSG:3857 saat diperlukan untuk OpenLayers.

3. **Denormalisasi Disengaja:** Kolom `province_name`, `regency_name`, `district_name` dikembalikan untuk menghindari join berulang pada query admin boundaries.

4. **Soft Delete:** Gunakan `valid_to` timestamp sebagai soft delete. Jangan melakukan hard delete kecuali untuk cleanup archive.

5. **Updated_at Trigger:** Semua tabel memiliki trigger untuk `updated_at`. Jangan update field ini secara manual.

6. **JSONB Migration Path:** Untuk Sprint 1, properti disimpan di kolom terpisah. Untuk Sprint 2+, pertimbangkan migrasi ke `properties JSONB` untuk fleksibilitas.

7. **UUID untuk User:** Menggunakan `uuid-ossp` extension. Pastikan `uuid-ossp` ter-install di database.

8. **Connection Pooling:** Untuk produksi, pertimbangkan menggunakan PgBouncer untuk connection pooling jika concurrent users > 50.

---

*Document Owner: Product Management & Business Analysis Team*
*Last Review: 2026-06-17*
*Next Review: 2026-07-01*
*Related Documents:* `docs/prd.md`, `docs/domain-model.md`, `docs/prd-gap-analysis.md`
