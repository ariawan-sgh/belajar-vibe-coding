# Product Backlog — WebGIS

**Basis Dokumen:** `docs/prd.md` v1.0, `docs/prd-gap-analysis.md`, `docs/domain-model.md`, `docs/srs.md`
**Metodologi:** Scrum
**Sprint Duration:** 2 minggu
**Tanggal:** 2026-06-17
**Pemilik:** Product Management

---

## 1. Epic Structure

| Epic ID | Epic Name | Deskripsi | Prioritas |
|---------|-----------|-----------|-----------|
| EPIC-01 | Spatial Data Foundation | Database schema, PostGIS setup, data seeding, indexing | P0 |
| EPIC-02 | Base Map Rendering | Peta dasar Indonesia, tile provider, CRS handling | P0 |
| EPIC-03 | Vector Layer System | Administrative boundaries, infrastructure layers, styling | P0 |
| EPIC-04 | Hover Interaction & Tooltip | Tooltip, highlight effect, hit-testing | P0 |
| EPIC-05 | Navigation & Controls | Zoom/pan, fit-to-extent, coordinate display | P0 |
| EPIC-06 | Backend Vector API | RESTful endpoints, spatial query, error handling | P0 |
| EPIC-07 | UX Polish & Reliability | Loading states, error states, attribution, caching | P0 |
| EPIC-08 | Authentication & Authorization | JWT, RBAC, login UI | P1 |
| EPIC-09 | Data Management (CRUD) | Upload spatial data, validation, import logs | P1 |
| EPIC-10 | Advanced Analysis Tools | Measure, draw/annotation, buffer | P2 |
| EPIC-11 | Export & Presentation | PDF export, copy-to-clipboard, theme toggle | P2 |
| EPIC-12 | Data Integration & Search | External API sync, webhooks, full-text search | P3 |

---

## 2. Product Backlog Items

### EPIC-01: Spatial Data Foundation

| ID | User Story | Prioritas | SP | Sprint | Dependencies |
|----|-----------|-----------|------|--------|--------------|
| PB-01 | Sebagai System, saya ingin database PostgreSQL + PostGIS berjalan dengan schema lengkap (tables, indexes, constraints), agar data spasial siap di-query. | P0 | 5 | Sprint 1 | DB server, PostGIS extension |
| PB-02 | Sebagai System, saya ingin data awal batas administrasi Indonesia (Provinsi & Kabupaten) di-seed ke database, agar layer terlihat saat aplikasi di-load. | P0 | 8 | Sprint 1 | PB-01, dataset BPS/Kemendagri |
| PB-03 | Sebagai System, saya ingin data awal infrastruktur (Jalan Nasional, Jalur Kereta Api) di-seed ke database, agar layer terlihat saat aplikasi di-load. | P0 | 5 | Sprint 1 | PB-01, dataset OSM/PUPR |

### EPIC-02: Base Map Rendering

| ID | User Story | Prioritas | SP | Sprint | Dependencies |
|----|-----------|-----------|------|--------|--------------|
| PB-04 | Sebagai GIS Officer, saya ingin peta dasar Indonesia dimuat menggunakan OpenStreetMap tiles, agar saya memiliki konteks visual wilayah. | P0 | 3 | Sprint 1 | Vite setup, OpenLayers |
| PB-05 | Sebagai System, saya ingin base map mendukung zoom level 3 hingga 18 dengan caching optimal, agar rendering tetap cepat. | P0 | 2 | Sprint 1 | PB-04 |

### EPIC-03: Vector Layer System

| ID | User Story | Prioritas | SP | Sprint | Dependencies |
|----|-----------|-----------|------|--------|--------------|
| PB-06 | Sebagai GIS Officer, saya ingin melihat layer batas administrasi (Provinsi, Kabupaten) dengan styling berbeda per level, agar mudah dibedakan. | P0 | 5 | Sprint 1 | PB-02, API vektor |
| PB-07 | Sebagai GIS Officer, saya ingin melihat layer infrastruktur (Jalan Nasional, Kereta Api) dengan warna berbeda per tipe, agar mudah dikenali. | P0 | 3 | Sprint 1 | PB-03, API vektor |
| PB-08 | Sebagai GIS Officer, saya ingin mengaktifkan/nonaktifkan layer secara independen melalui layer switcher, agar saya bisa fokus ke data yang relevan. | P0 | 3 | Sprint 1 | PB-06, PB-07 |

### EPIC-04: Hover Interaction & Tooltip

| ID | User Story | Prioritas | SP | Sprint | Dependencies |
|----|-----------|-----------|------|--------|--------------|
| PB-09 | Sebagai GIS Officer, saya ingin melihat tooltip dengan atribut objek saat mouse hover, agar saya dapat mendapatkan informasi cepat tanpa klik. | P0 | 5 | Sprint 1 | PB-06, PB-07 |
| PB-10 | Sebagai GIS Officer, saya ingin objek yang di-hover memiliki highlight visual, agar saya tahu objek mana yang sedang aktif. | P0 | 3 | Sprint 1 | PB-09 |

### EPIC-05: Navigation & Controls

| ID | User Story | Prioritas | SP | Sprint | Dependencies |
|----|-----------|-----------|------|--------|--------------|
| PB-11 | Sebagai GIS Officer, saya ingin melakukan zoom in/out dan pan dengan scroll wheel dan tombol kontrol, agar saya bisa navigasi peta dengan nyaman. | P0 | 3 | Sprint 1 | PB-04 |
| PB-12 | Sebagai GIS Officer, saya ingin tombol "Reset View" untuk menampilkan seluruh Indonesia, agar saya bisa kembali ke view awal dengan cepat. | P0 | 2 | Sprint 1 | PB-06 |
| PB-13 | Sebagai GIS Officer, saya ingin melihat koordinat (Lon/Lat) posisi kursor secara real-time di bottom bar, agar saya bisa mencatat posisi spasial. | P0 | 2 | Sprint 1 | PB-04 |

### EPIC-06: Backend Vector API

| ID | User Story | Prioritas | SP | Sprint | Dependencies |
|----|-----------|-----------|------|--------|--------------|
| PB-14 | Sebagai System, saya ingin API endpoint `/api/v1/vector/admin-boundaries` mengembalikan GeoJSON/MVT dengan query PostGIS teroptimasi, agar frontend dapat memuat layer. | P0 | 8 | Sprint 1 | PB-01, PB-02 |
| PB-15 | Sebagai System, saya ingin API endpoint `/api/v1/vector/infrastructure` mengembalikan GeoJSON/MVT dengan query PostGIS teroptimasi, agar frontend dapat memuat layer. | P0 | 5 | Sprint 1 | PB-01, PB-03 |
| PB-16 | Sebagai System, saya ingin API endpoint `/api/v1/metadata/layers` mengembalikan metadata layer yang tersedia, agar frontend dapat menampilkan layer switcher. | P0 | 3 | Sprint 1 | PB-01 |
| PB-17 | Sebagai System, saya ingin API endpoint `/health` untuk health check, agar monitoring dapat mendeteksi downtime. | P0 | 1 | Sprint 1 | - |

### EPIC-07: UX Polish & Reliability

| ID | User Story | Prioritas | SP | Sprint | Dependencies |
|----|-----------|-----------|------|--------|--------------|
| PB-18 | Sebagai System, saya ingin loading state ditampilkan saat data dimuat, agar user tahu sistem sedang bekerja. | P0 | 3 | Sprint 1 | PB-14, PB-15 |
| PB-19 | Sebagai System, saya ingin error state ditampilkan jika API gagal, agar user dapat mengambil tindakan (retry). | P0 | 2 | Sprint 1 | Error handling |
| PB-20 | Sebagai System, saya ingin map attribution ditampilkan sesuai license, agar tidak melanggar ketentuan penggunaan data. | P0 | 1 | Sprint 1 | Gap analysis GAP-ATTR-06 |

### EPIC-08: Authentication & Authorization

| ID | User Story | Prioritas | SP | Sprint | Dependencies |
|----|-----------|-----------|------|--------|--------------|
| PB-21 | Sebagai Admin, saya ingin sistem autentikasi JWT, agar hanya user yang terdaftar yang dapat mengakses aplikasi. | P1 | 8 | Sprint 2 | Database users table |
| PB-22 | Sebagai Admin, saya ingin role-based access control (RBAC) dengan role `GISOfficer` dan `SystemAdministrator`, agar hak akses terbatas sesuai tanggung jawab. | P1 | 5 | Sprint 2 | PB-21 |
| PB-23 | Sebagai GIS Officer, saya ingin login dengan username/password, agar saya dapat mengakses aplikasi. | P1 | 3 | Sprint 2 | PB-21 |

### EPIC-09: Data Management (CRUD)

| ID | User Story | Prioritas | SP | Sprint | Dependencies |
|----|-----------|-----------|------|--------|--------------|
| PB-24 | Sebagai Admin, saya ingin upload data spasial (Shapefile/GeoJSON) melalui UI, agar saya dapat menambahkan layer baru tanpa SQL manual. | P1 | 13 | Sprint 2 | PB-21, OGR tooling |
| PB-25 | Sebagai Admin, saya ingin validasi otomatis `ST_IsValid` dan `ST_SRID` saat import, agar data yang masuk ke database berkualitas baik. | P1 | 5 | Sprint 2 | PB-24 |
| PB-26 | Sebagai Admin, saya ingin melihat log import (sukses/gagal, feature count, error message), agar saya bisa tracking kegagalan import. | P1 | 3 | Sprint 2 | PB-24 |
| PB-27 | Sebagai System, saya ingin request rate limiting (100 req/min), agar API tidak dibombardir oleh abuse. | P1 | 3 | Sprint 2 | Middleware |
| PB-28 | Sebagai System, saya ingin structured logging untuk semua request backend, agar debugging mudah dilakukan. | P1 | 2 | Sprint 2 | Logging library |
| PB-29 | Sebagai Admin, saya ingin mengedit metadata layer (nama, tipe, default visibility, z-index), agar saya dapat mengelola layer tanpa restart aplikasi. | P1 | 5 | Sprint 2 | PB-21 |
| PB-30 | Sebagai Admin, saya ingin menambahkan layer baru (Sungai, Danau, Gunung, TPA, Kawasan Konservasi), agar data spasial lebih kaya. | P1 | 8 | Sprint 2 | PB-24, PB-29 |

### EPIC-10: Advanced Analysis Tools

| ID | User Story | Prioritas | SP | Sprint | Dependencies |
|----|-----------|-----------|------|--------|--------------|
| PB-31 | Sebagai GIS Officer, saya ingin tool pengukuran jarak dan luas area di peta, agar saya dapat menghitung dimensi spasial secara cepat. | P2 | 8 | Sprint 3 | OpenLayers interaction |
| PB-32 | Sebagai GIS Officer, saya ingin menggambar anotasi (point, line, polygon) di peta dengan atribut, agar saya bisa menandai area penting. | P2 | 13 | Sprint 3 | OpenLayers Draw interaction |
| PB-33 | Sebagai GIS Officer, saya ingin membuat buffer (kaersalahan) dari objek spasial, agar saya bisa menganalisis pengaruh jarak. | P2 | 8 | Sprint 3 | PostGIS ST_Buffer |

### EPIC-11: Export & Presentation

| ID | User Story | Prioritas | SP | Sprint | Dependencies |
|----|-----------|-----------|------|--------|--------------|
| PB-34 | Sebagai GIS Officer, saya ingin export peta ke PDF dengan legend dan scale bar, agar saya bisa membuat laporan presentasi. | P2 | 13 | Sprint 3 | Print library (jsPDF + dom-to-image) |
| PB-35 | Sebagai GIS Officer, saya ingin meng-copy atribut feature ke clipboard, agar saya bisa memindahkan data ke spreadsheet/ dokumen lain. | P2 | 2 | Sprint 3 | Tooltip enhancement |
| PB-36 | Sebagai GIS Officer, saya ingin toggle tema gelap/terang, agar saya bisa bekerja dalam kondisi cahaya berbeda. | P2 | 5 | Sprint 3 | Tailwind dark mode |
| PB-37 | Sebagai GIS Officer, saya ingin share view peta (URL dengan zoom, center, layer aktif), agar saya bisa mengirimkan konteks peta ke rekan kerja. | P2 | 5 | Sprint 3 | URL state serialization |

### EPIC-12: Data Integration & Search

| ID | User Story | Prioritas | SP | Sprint | Dependencies |
|----|-----------|-----------|------|--------|--------------|
| PB-38 | Sebagai System, saya ingin integrasi dengan REST API eksternal (BPS, BMKG, OpenData Indonesia) untuk auto-sync data spasial, agar data selalu terbaru. | P3 | 8 | Sprint 4 | Webhook scheduling |
| PB-39 | Sebagai Admin, saya ingin webhook notification ketika terjadi perubahan data, agar saya selalu tahu status dataset. | P3 | 5 | Sprint 4 | PB-38 |
| PB-40 | Sebagai GIS Officer, saya ingin full-text search berdasarkan nama/nama wilayah/code, agar saya dapat menemukan lokasi dengan cepat. | P3 | 8 | Sprint 4 | PostgreSQL tsvector |
| PB-41 | Sebagai GIS Officer, saya ingin export data spasial (GeoJSON, Shapefile, CSV, KML), agar saya bisa menganalisis lebih lanjut di tools lain. | P3 | 8 | Sprint 4 | File generation library |
| PB-42 | Sebagai GIS Officer, saya ingin time-series layer support (animasi data berdasarkan waktu), agar saya bisa melihat perubahan spasial sepanjang masa. | P3 | 13 | Sprint 4 | Time-series schema |

---

## 3. Sprint Allocation

### Sprint 1 (Q1 2026) — "Core Foundation"
**Target:** 34 SP

| ID | User Story | SP | Epic |
|----|-----------|------|------|
| PB-01 | Database schema + PostGIS setup | 5 | EPIC-01 |
| PB-02 | Seed data batas administrasi | 8 | EPIC-01 |
| PB-03 | Seed data infrastruktur | 5 | EPIC-01 |
| PB-04 | Base map OSM rendering | 3 | EPIC-02 |
| PB-05 | Zoom level 3-18 + caching | 2 | EPIC-02 |
| PB-06 | Layer batas administrasi dengan styling | 5 | EPIC-03 |
| PB-07 | Layer infrastruktur dengan styling | 3 | EPIC-03 |
| PB-08 | Layer switcher (enable/disable) | 3 | EPIC-03 |
| PB-09 | Tooltip hover dengan atribut | 5 | EPIC-04 |
| PB-10 | Highlight effect pada hover | 3 | EPIC-04 |
| PB-11 | Zoom/pan navigation controls | 3 | EPIC-05 |
| PB-12 | Reset View / Fit Indonesia | 2 | EPIC-05 |
| PB-13 | Coordinate bar (Lon/Lat) | 2 | EPIC-05 |
| PB-14 | API vector admin-boundaries | 8 | EPIC-06 |
| PB-15 | API vector infrastructure | 5 | EPIC-06 |
| PB-16 | API metadata layers | 3 | EPIC-06 |
| PB-17 | API health check | 1 | EPIC-06 |
| PB-18 | Loading state UI | 3 | EPIC-07 |
| PB-19 | Error state + retry | 2 | EPIC-07 |
| PB-20 | Map attribution display | 1 | EPIC-07 |

**Subtotal:** 83 SP (Prioritaskan PB-01 s/d PB-13 untuk MVP, sisanya stretch)

### Sprint 2 (Q2 2026) — "Data Management & Access Control"
**Target:** 38 SP

| ID | User Story | SP | Epic |
|----|-----------|------|------|
| PB-21 | JWT Authentication | 8 | EPIC-08 |
| PB-22 | RBAC (GISOfficer, SystemAdministrator) | 5 | EPIC-08 |
| PB-23 | Login UI | 3 | EPIC-08 |
| PB-24 | Upload spatial data (Shapefile/GeoJSON) | 13 | EPIC-09 |
| PB-25 | Auto geometry validation on import | 5 | EPIC-09 |
| PB-26 | Import log tracking UI | 3 | EPIC-09 |
| PB-27 | API rate limiting (100 req/min) | 3 | EPIC-07 |
| PB-28 | Structured backend logging | 2 | EPIC-07 |
| PB-29 | Edit layer metadata UI | 5 | EPIC-09 |
| PB-30 | Tambah layer baru (Sungai, Danau, dll) | 8 | EPIC-09 |

### Sprint 3 (Q3 2026) — "Analysis & Presentation"
**Target:** 36 SP

| ID | User Story | SP | Epic |
|----|-----------|------|------|
| PB-31 | Measure tool (jarak, luas area) | 8 | EPIC-10 |
| PB-32 | Drawing & annotation tools | 13 | EPIC-10 |
| PB-33 | Spatial buffer tool | 8 | EPIC-10 |
| PB-34 | Export peta ke PDF | 13 | EPIC-11 |
| PB-35 | Copy atribut to clipboard | 2 | EPIC-11 |
| PB-36 | Dark/Light theme toggle | 5 | EPIC-11 |
| PB-37 | Share view URL | 5 | EPIC-11 |

### Sprint 4 (Q4 2026) — "Integration & Scale"
**Target:** 39 SP

| ID | User Story | SP | Epic |
|----|-----------|------|------|
| PB-38 | Integrasi API eksternal (BPS/BMKG) | 8 | EPIC-12 |
| PB-39 | Webhook notification | 5 | EPIC-12 |
| PB-40 | Full-text search atribut spasial | 8 | EPIC-12 |
| PB-41 | Export data (GeoJSON, SHP, CSV, KML) | 8 | EPIC-12 |
| PB-42 | Time-series layer support | 13 | EPIC-12 |

---

## 4. Backlog Summary

| Priority | Count | Total SP | Sprint Allocation |
|----------|-------|----------|-------------------|
| **P0** | 20 | 83 SP | Sprint 1 (MVP) |
| **P1** | 10 | 55 SP | Sprint 2 |
| **P2** | 7 | 54 SP | Sprint 3 |
| **P3** | 5 | 42 SP | Sprint 4 |
| **Total** | **42** | **234 SP** | 4 Sprints |

---

## 5. Definition of Ready (DoR)

Sebelum item masuk sprint, harus:
- [ ] User Story sudah di-refine dengan acceptance criteria yang terukur
- [ ] Dependencies sudah diidentifikasi dan tersedia
- [ ] Estimasi story points sudah diberikan oleh team
- [ ] Design/UI spec sudah finalized (jika涉及 UI)
- [ ] Test strategy sudah didefinisikan

## 6. Definition of Done (DoD)

Sebelum item di-mark "Done":
- [ ] Code sudah di-review minimal 1 reviewer
- [ ] Unit test coverage ≥ 70%
- [ ] Kode sudah di-merge ke branch `main`
- [ ] Manual testing sesuai acceptance criteria sudah di-verify
- [ ] Dokumentasi sudah di-update
- [ ] No critical/high bugs
- [ ] Deployed ke staging dan di-verify oleh PO

---

*Document Owner: Product Management*
*Last Review: 2026-06-17*
*Next Review: 2026-06-24 (Sprint 1 Kickoff)*
