# Sprint Plan — WebGIS

**Basis Dokumen:** `docs/prd.md` v1.0, `docs/product-backlog.md`
**Metodologi:** Scrum (2-week sprint)
**Total Sprint:** 4 sprints (Q1–Q4 2026)
**Tanggal:** 2026-06-17
**Pemilik:** Product Management & Scrum Master

---

## 1. Sprint Overview

| Sprint | Periode | Goal | Total SP | Status |
|--------|---------|------|----------|--------|
| **Sprint 1** | 17 Jun – 30 Jun 2026 | Core Launch — Peta dasar + hover interaction + API dasar | 34 SP | Planned |
| **Sprint 2** | 01 Jul – 14 Jul 2026 | Data Management & Access Control — Auth + CRUD data | 38 SP | Planned |
| **Sprint 3** | 15 Jul – 28 Jul 2026 | Analysis & Presentation — Analysis tools + export | 36 SP | Planned |
| **Sprint 4** | 29 Jul – 11 Agu 2026 | Integration & Scale — External API + mobile + search | 39 SP | Planned |

**Total Story Points:** 147 SP
**Rata-rata per Sprint:** 36.75 SP

---

## 2. Sprint 1 — "Core Launch"

### 2.1 Sprint Details

| Field | Value |
|-------|-------|
| **Sprint Goal** | Aplikasi siap digunakan oleh 10+ GIS Officer untuk eksplorasi spasial dasar peta Indonesia dengan performa ≥ 30fps. |
| **Duration** | 2 minggu (10 hari kerja) |
| **Start Date** | 17 Juni 2026 |
| **End Date** | 30 Juni 2026 |
| **Total Story Points** | 34 SP |
| **Team Capacity** | 3 Frontend, 2 Backend, 1 DevOps/Data Engineer |
| **Velocity Target** | 34 SP (100% committed) |

### 2.2 Sprint Backlog

#### Day 1–2: Infrastructure & Database Setup

| Task ID | Description | SP | Assignee | Dependencies |
|---------|-------------|----|----------|--------------|
| PB-01 | Database schema + PostGIS setup | 5 | Backend + DevOps | DB server, PostGIS extension |
| PB-02 | Seed data batas administrasi (Provinsi & Kabupaten) | 8 | Backend + Data Engineer | PB-01, dataset BPS/Kemendagri |
| PB-03 | Seed data infrastruktur (Jalan Nasional, Kereta Api) | 5 | Backend + Data Engineer | PB-01, dataset OSM/PUPR |

#### Day 3–4: Backend API Development

| Task ID | Description | SP | Assignee | Dependencies |
|---------|-------------|----|----------|--------------|
| PB-14 | API `/api/v1/vector/admin-boundaries` | 8 | Backend | PB-01, PB-02 |
| PB-15 | API `/api/v1/vector/infrastructure` | 5 | Backend | PB-01, PB-03 |
| PB-16 | API `/api/v1/metadata/layers` | 3 | Backend | PB-01 |
| PB-17 | API `/health` endpoint | 1 | Backend | - |

#### Day 5–7: Frontend Core Development

| Task ID | Description | SP | Assignee | Dependencies |
|---------|-------------|----|----------|--------------|
| PB-04 | Base map OSM rendering | 3 | Frontend | Vite setup, OpenLayers |
| PB-05 | Zoom level 3–18 + caching | 2 | Frontend | PB-04 |
| PB-06 | Layer batas administrasi dengan styling | 5 | Frontend + Backend | PB-02, PB-14 |
| PB-07 | Layer infrastruktur dengan styling | 3 | Frontend + Backend | PB-03, PB-15 |
| PB-08 | Layer switcher (enable/disable) | 3 | Frontend | PB-06, PB-07 |

#### Day 8–10: Interaction & Polish

| Task ID | Description | SP | Assignee | Dependencies |
|---------|-------------|----|----------|--------------|
| PB-09 | Tooltip hover dengan atribut | 5 | Frontend | PB-06, PB-07 |
| PB-10 | Highlight effect pada hover | 3 | Frontend | PB-09 |
| PB-11 | Zoom/pan navigation controls | 3 | Frontend | PB-04 |
| PB-12 | Reset View / Fit Indonesia | 2 | Frontend | PB-06 |
| PB-13 | Coordinate bar (Lon/Lat) | 2 | Frontend | PB-04 |
| PB-18 | Loading state UI | 3 | Frontend | PB-14, PB-15 |
| PB-19 | Error state + retry | 2 | Frontend | Error handling |
| PB-20 | Map attribution display | 1 | Frontend | Gap analysis |

### 2.3 Sprint 1 Timeline

```
Week 1
┌─────────────────────────────────────────────────────────────────┐
│ Day 1 (Mon) │ Day 2 (Tue) │ Day 3 (Wed) │ Day 4 (Thu) │ Day 5 (Fri) │
├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
│ PB-01       │ PB-02       │ PB-14       │ PB-15       │ PB-04       │
│ PB-03       │             │ PB-16       │ PB-17       │ PB-05       │
│             │             │             │             │             │
│ *Infra      │ *Data       │ *API Dev    │ *API Dev    │ *Frontend   │
│  Setup      │  Seeding    │             │             │  Core       │
└─────────────────────────────────────────────────────────────────┘

Week 2
┌─────────────────────────────────────────────────────────────────┐
│ Day 6 (Mon) │ Day 7 (Tue) │ Day 8 (Wed) │ Day 9 (Thu) │ Day 10 (Fri)│
├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
│ PB-06       │ PB-09       │ PB-11       │ PB-18       │ Sprint      │
│ PB-07       │ PB-10       │ PB-12       │ PB-19       │ Review &    │
│ PB-08       │             │ PB-13       │ PB-20       │ Retro       │
│             │             │             │             │             │
│ *Vector UI  │ *Interaction│ *Navigation │ *Polish &   │ *Demo to    │
│  & Styling  │  & Tooltip  │  Controls   │  Bug Fix    │  Stakeholder│
└─────────────────────────────────────────────────────────────────┘
```

### 2.4 Sprint 1 Milestones

| Milestone | Deliverable | Due Date |
|-----------|-------------|----------|
| **M1** | Database schema + seeded data | Day 2 (18 Jun) |
| **M2** | Backend API endpoints live | Day 4 (20 Jun) |
| **M3** | Frontend map rendering + layers | Day 7 (23 Jun) |
| **M4** | Tooltip + navigation complete | Day 9 (25 Jun) |
| **M5** | Sprint Review & UAT | Day 10 (30 Jun) |

### 2.5 Sprint 1 Risks & Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Dataset batas administrasi tidak tersedia dalam format siap pakai | High | Medium | Gunakan backup: download dari GitHub indonesiaopen/batas-administratif |
| Performa rendering lambat saat 5000+ features | High | Low | Optimasi: GIST index, geometry simplification, debounce tooltip |
| OpenLayers learning curve untuk junior dev | Medium | Medium | Pair programming dengan senior, Dokumentasi internal |
| CORS issue saat development | Medium | Low | Setup CORS middleware di awal, whitelist localhost |

---

## 3. Sprint 2 — "Data Management & Access Control"

### 3.1 Sprint Details

| Field | Value |
|-------|-------|
| **Sprint Goal** | GIS Officer dapat login dan mengelola data spasial tanpa bantuan engineering; data management terotomatisasi. |
| **Duration** | 2 minggu (10 hari kerja) |
| **Start Date** | 01 Juli 2026 |
| **End Date** | 14 Juli 2026 |
| **Total Story Points** | 38 SP |
| **Team Capacity** | 3 Frontend, 2 Backend, 1 DevOps/Data Engineer |
| **Velocity Target** | 38 SP (maintain atau improve dari Sprint 1) |

### 3.2 Sprint Backlog

#### Day 1–3: Authentication & Authorization

| Task ID | Description | SP | Assignee | Dependencies |
|---------|-------------|----|----------|--------------|
| PB-21 | JWT Authentication system | 8 | Backend | Database users table |
| PB-22 | RBAC (GISOfficer, SystemAdministrator) | 5 | Backend | PB-21 |
| PB-23 | Login UI (form + session management) | 3 | Frontend | PB-21 |

#### Day 4–7: Data Management CRUD

| Task ID | Description | SP | Assignee | Dependencies |
|---------|-------------|----|----------|--------------|
| PB-24 | Upload spatial data (Shapefile/GeoJSON) | 13 | Frontend + Backend | PB-21, OGR tooling |
| PB-25 | Auto geometry validation on import | 5 | Backend | PB-24 |
| PB-26 | Import log tracking UI | 3 | Backend + Frontend | PB-24 |
| PB-29 | Edit layer metadata UI | 5 | Frontend + Backend | PB-21 |

#### Day 8–10: Enhancement & Polish

| Task ID | Description | SP | Assignee | Dependencies |
|---------|-------------|----|----------|--------------|
| PB-27 | API rate limiting (100 req/min) | 3 | Backend | Middleware |
| PB-28 | Structured backend logging | 2 | Backend | Logging library |
| PB-30 | Tambah layer baru (Sungai, Danau, Gunung, TPA, Konservasi) | 8 | Frontend + Backend | PB-24, PB-29 |

### 3.3 Sprint 2 Timeline

```
Week 1
┌─────────────────────────────────────────────────────────────────┐
│ Day 1 (Mon) │ Day 2 (Tue) │ Day 3 (Wed) │ Day 4 (Thu) │ Day 5 (Fri) │
├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
│ PB-21       │ PB-22       │ PB-23       │ PB-24       │ PB-24       │
│ (JWT)       │ (RBAC)      │ (Login UI)  │ (Upload     │ (Upload     │
│             │             │             │  UI)        │  Backend)   │
└─────────────────────────────────────────────────────────────────┘

Week 2
┌─────────────────────────────────────────────────────────────────┐
│ Day 6 (Mon) │ Day 7 (Tue) │ Day 8 (Wed) │ Day 9 (Thu) │ Day 10 (Fri)│
├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
│ PB-25       │ PB-26       │ PB-27       │ PB-29       │ PB-30       │
│ (Validation│ (Import Log │ (Rate Limit │ (Meta Edit   │ (New Layers │
│  + Fix)     │  UI)        │  + Logging) │  UI)         │  + Demo)    │
└─────────────────────────────────────────────────────────────────┘
```

### 3.4 Sprint 2 Milestones

| Milestone | Deliverable | Due Date |
|-----------|-------------|----------|
| **M1** | JWT Auth + Login UI | Day 3 (03 Jul) |
| **M2** | RBAC role enforcement | Day 3 (03 Jul) |
| **M3** | Upload spatial data feature | Day 7 (07 Jul) |
| **M4** | Import validation + logging | Day 8 (08 Jul) |
| **M5** | New layers (Sungai, Danau, dll) | Day 10 (14 Jul) |

---

## 4. Sprint 3 — "Analysis & Presentation"

### 4.1 Sprint Details

| Field | Value |
|-------|-------|
| **Sprint Goal** | WebGIS berkembang menjadi tool analisis spasial yang dapat menghasilkan output presentasi. |
| **Duration** | 2 minggu (10 hari kerja) |
| **Start Date** | 15 Juli 2026 |
| **End Date** | 28 Juli 2026 |
| **Total Story Points** | 36 SP |

### 4.2 Sprint Backlog

| Task ID | Description | SP | Assignee | Dependencies |
|---------|-------------|----|----------|--------------|
| PB-31 | Measure tool (jarak, luas area) | 8 | Frontend | OpenLayers interaction |
| PB-32 | Drawing & annotation tools | 13 | Frontend | OpenLayers Draw |
| PB-33 | Spatial buffer tool | 8 | Backend + Frontend | PostGIS ST_Buffer |
| PB-34 | Export peta ke PDF | 13 | Frontend | jsPDF + dom-to-image |
| PB-35 | Copy atribut to clipboard | 2 | Frontend | Tooltip enhancement |
| PB-36 | Dark/Light theme toggle | 5 | Frontend | Tailwind dark mode |
| PB-37 | Share view URL | 5 | Frontend | URL state serialization |

**Catatan:** PB-34 (PDF export 13 SP) adalah item terberat. Jika terdesak, dapat di-defer ke Sprint 4 atau di-split menjadi MVP (tanpa legend) dan full version.

### 4.3 Sprint 3 Timeline

```
Week 1
┌─────────────────────────────────────────────────────────────────┐
│ Day 1 (Mon) │ Day 2 (Tue) │ Day 3 (Wed) │ Day 4 (Thu) │ Day 5 (Fri) │
├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
│ PB-31       │ PB-31       │ PB-32       │ PB-32       │ PB-32       │
│ (Measure    │ (Measure    │ (Drawing    │ (Drawing    │ (Drawing    │
│  Setup)     │  Logic)     │  Setup)     │  Logic)     │  Polish)    │
└─────────────────────────────────────────────────────────────────┘

Week 2
┌─────────────────────────────────────────────────────────────────┐
│ Day 6 (Mon) │ Day 7 (Tue) │ Day 8 (Wed) │ Day 9 (Thu) │ Day 10 (Fri)│
├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
│ PB-33       │ PB-34       │ PB-35       │ PB-36       │ PB-37       │
│ (Buffer)    │ (PDF Export │ (Copy       │ (Theme       │ (Share URL  │
│             │  Setup)     │  Clipboard) │  Toggle)     │  + Demo)    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Sprint 4 — "Integration & Scale"

### 5.1 Sprint Details

| Field | Value |
|-------|-------|
| **Sprint Goal** | WebGIS terhubung dengan data eksternal dan mendukung akses mobile dasar. |
| **Duration** | 2 minggu (10 hari kerja) |
| **Start Date** | 29 Juli 2026 |
| **End Date** | 11 Agustus 2026 |
| **Total Story Points** | 39 SP |

### 5.2 Sprint Backlog

| Task ID | Description | SP | Assignee | Dependencies |
|---------|-------------|----|----------|--------------|
| PB-40 | Full-text search atribut spasial | 8 | Backend + Frontend | PostgreSQL tsvector |
| PB-41 | Export data (GeoJSON, SHP, CSV, KML) | 8 | Backend + Frontend | File generation library |
| PB-38 | Integrasi API eksternal (BPS/BMKG) | 8 | Backend | Webhook scheduling |
| PB-39 | Webhook notification | 5 | Backend | PB-38 |
| PB-42 | Time-series layer support | 13 | Frontend + Backend | Time-series schema |
| PB-43 | Responsive tablet/mobile | 8 | Frontend | Tailwind responsive |
| PB-44 | Async task queue (Celery/RQ) | 8 | Backend + DevOps | Redis |

**Catatan:** Total SP Sprint 4 adalah 58 SP. Jika team velocity adalah ~37 SP/sprint, prioritaskan PB-40, PB-41, PB-43 (must-have untuk Q4) dan defer PB-38, PB-39, PB-42, PB-44 ke Sprint 5 atau roadmap.

### 5.3 Sprint 4 Timeline

```
Week 1
┌─────────────────────────────────────────────────────────────────┐
│ Day 1 (Mon) │ Day 2 (Tue) │ Day 3 (Wed) │ Day 4 (Thu) │ Day 5 (Fri) │
├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
│ PB-40       │ PB-40       │ PB-41       │ PB-41       │ PB-43       │
│ (Search     │ (Search     │ (Export     │ (Export     │ (Mobile     │
│  Backend)   │  Frontend)  │  Backend)   │  Frontend)  │  Responsive)│
└─────────────────────────────────────────────────────────────────┘

Week 2
┌─────────────────────────────────────────────────────────────────┐
│ Day 6 (Mon) │ Day 7 (Tue) │ Day 8 (Wed) │ Day 9 (Thu) │ Day 10 (Fri)│
├─────────────┼─────────────┼─────────────┼─────────────┼─────────────┤
│ PB-38       │ PB-39       │ PB-42       │ PB-44       │ Sprint      │
│ (External   │ (Webhook)   │ (Time-      │ (Async      │ Review &    │
│  API Sync)  │             │  series)    │  Queue)     │ Demo        │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Sprint Ceremonies Schedule

### 6.1 Ceremony Overview

| Ceremony | Duration | Frequency | Participants | Time |
|----------|----------|-----------|--------------|------|
| **Sprint Planning** | 2 jam | Sprint start | Full team + PO | Day 1, 09:00–11:00 WIB |
| **Daily Standup** | 15 menit | Setiap hari | Full team | 09:00–09:15 WIB |
| **Backlog Refinement** | 1 jam | Setiap hari (Selasa & Kamis) | PO + Dev Team | 10:00–11:00 WIB |
| **Sprint Review** | 1 jam | Sprint end | Full team + Stakeholders | Day 10, 14:00–15:00 WIB |
| **Sprint Retrospective** | 45 menit | Sprint end | Full team + SM | Day 10, 15:15–16:00 WIB |
| **Technical Sync** | 30 menit | Weekly (Rabu) | Dev Team only | 10:00–10:30 WIB |

### 6.2 Definition of Ready (DoR)

Sebelum task bisa masuk sprint, harus memenuhi:
- [ ] User Story sudah di-refine dan memiliki acceptance criteria yang terukur.
- [ ] Dependency identifikasi dan tersedia.
- [ ] Estimasi story points sudah diberikan oleh team.
- [ ] Design/UI spec sudah ready (jika涉及 UI).
- [ ] Test approach sudah didefinisikan.

### 6.3 Definition of Done (DoD)

Sebelum task bisa di-mark "Done", harus memenuhi:
- [ ] Code sudah di-review minimal 1 reviewer.
- [ ] Unit test coverage ≥ 70%.
- [ ] Kode sudah di-merge ke branch `main` (atau `develop` jika menggunakan GitFlow).
- [ ] Manual testing sesuai acceptance criteria.
- [ ] Dokumentasi API/component sudah di-update.
- [ ] No critical/high bugs.
- [ ] Deployed ke staging environment dan di-verify oleh PO.

---

## 7. Team Capacity & Velocity

### 7.1 Team Composition

| Role | Nama | Capacity (hari/sprint) | Skills |
|------|------|------------------------|--------|
| **Product Owner** | [Nama PO] | 10 hari | Business analysis, stakeholder management |
| **Scrum Master** | [Nama SM] | 10 hari | Agile coaching, impediment removal |
| **Senior Frontend** | [Nama] | 10 hari | Lit JS, OpenLayers, Tailwind, Vite |
| **Junior Frontend** | [Nama] | 10 hari | HTML, CSS, JavaScript dasar |
| **Senior Backend** | [Nama] | 10 hari | Golang Fiber, PostgreSQL, PostGIS |
| **Junior Backend** | [Nama] | 10 hari | Golang dasar, REST API |
| **DevOps/Data Engineer** | [Nama] | 10 hari | Docker, PostgreSQL, ETL, Geo data |

### 7.2 Velocity Tracking

| Sprint | Committed SP | Completed SP | Velocity | Notes |
|--------|-------------|--------------|----------|-------|
| Sprint 1 | 34 | - | - | Planned |
| Sprint 2 | 38 | - | - | Planned |
| Sprint 3 | 36 | - | - | Planned |
| Sprint 4 | 39 | - | - | Planned |
| **Total** | **147** | **-** | **-** | - |

**Target Velocity:** 37 SP/sprint (rata-rata dari 4 sprint)

### 7.3 Capacity per Role per Sprint

| Role | Sprint 1 | Sprint 2 | Sprint 3 | Sprint 4 |
|------|----------|----------|----------|----------|
| Senior Frontend | 20 SP | 16 SP | 21 SP | 21 SP |
| Junior Frontend | 10 SP | 12 SP | 10 SP | 10 SP |
| Senior Backend | 18 SP | 21 SP | 13 SP | 21 SP |
| Junior Backend | - | 8 SP | 8 SP | 8 SP |
| DevOps/Data Engineer | 13 SP | 8 SP | 5 SP | 13 SP |

---

## 8. Dependencies & Critical Path

### 8.1 Dependency Map

```
Sprint 1
┌───────────────────────────────────────────────────────────────────┐
│  PB-01 (DB Setup)                                                  │
│      │                                                              │
│      ├──▶ PB-02 (Seed Admin)                                       │
│      │         │                                                    │
│      │         └──▶ PB-06 (Admin Layer UI)                         │
│      │                   │                                          │
│      │                   └──▶ PB-09 (Tooltip)                      │
│      │                                                              │
│      ├──▶ PB-03 (Seed Infra)                                       │
│      │         │                                                    │
│      │         └──▶ PB-07 (Infra Layer UI)                         │
│      │                   │                                          │
│      │                   └──▶ PB-08 (Layer Switcher)               │
│      │                                                              │
│      └──▶ PB-14, PB-15, PB-16 (API Endpoints)                      │
│                │                                                    │
│                └──▶ PB-04, PB-05 (Base Map)                        │
│                          │                                          │
│                          └──▶ PB-11, PB-12, PB-13 (Navigation)    │
└───────────────────────────────────────────────────────────────────┘

Sprint 2
┌───────────────────────────────────────────────────────────────────┐
│  PB-21 (JWT Auth)                                                  │
│      │                                                              │
│      ├──▶ PB-22 (RBAC)                                             │
│      │         │                                                    │
│      │         └──▶ PB-23 (Login UI)                               │
│      │                                                              │
│      └──▶ PB-24 (Upload Data)                                      │
│                │                                                    │
│                ├──▶ PB-25 (Validation)                             │
│                │         │                                          │
│                │         └──▶ PB-26 (Import Log)                   │
│                │                                                    │
│                └──▶ PB-29 (Edit Metadata)                          │
│                          │                                          │
│                          └──▶ PB-30 (New Layers)                   │
└───────────────────────────────────────────────────────────────────┘
```

### 8.2 Critical Path

1. **PB-01** (DB Setup) → **PB-02/PB-03** (Data Seeding) → **PB-06/PB-07** (Vector Layers) → **PB-09** (Tooltip) → **Sprint 1 Go-Live**
2. **PB-21** (JWT Auth) → **PB-22** (RBAC) → **PB-24** (Upload) → **PB-29** (Edit Metadata) → **PB-30** (New Layers) → **Sprint 2 Complete**

---

## 9. Risk Register & Mitigation

| ID | Risk | Probability | Impact | Sprint | Mitigation | Owner |
|----|------|-------------|--------|--------|------------|-------|
| R-01 | Dataset batas administrasi tidak tersedia | Medium | High | 1 | Gunakan dataset alternatif dari GitHub indonesiaopen | Data Engineer |
| R-02 | Performa rendering < 30fps pada 5000+ features | Low | High | 1 | Optimasi: GIST index, geometry simplification, debounce | Backend + Frontend |
| R-03 | OpenLayers learning curve untuk junior dev | Medium | Medium | 1 | Pair programming, dokumentasi internal | Senior Frontend |
| R-04 | Dataset OSM terlalu besar untuk di-seed | Medium | Medium | 1 | Pre-filter data: hanya Jalan Nasional & Kereta Api | Data Engineer |
| R-05 | JWT implementation complexity | Low | Medium | 2 | Gunakan library golang-jwt/jwt,模板 siap pakai | Backend |
| R-06 | Upload Shapefile berukuran besar (> 100MB) | Medium | Medium | 2 | Implementasi chunked upload + progress bar | Backend + Frontend |
| R-07 | MVT generation lambat di backend | Medium | Medium | 3 | Pre-generate tiles atau gunakan materialized view | Backend |
| R-08 | Time-series data growth tidak terduga | Low | Medium | 4 | Implementasi partitioning by date | Backend |

---

## 10. Success Criteria per Sprint

### 10.1 Sprint 1 Success Criteria

- [ ] Aplikasi dapat diakses di `http://localhost:5173` (Vite dev server).
- [ ] Peta Indonesia ter-render dengan base map OSM.
- [ ] Layer batas administrasi (Provinsi, Kabupaten) terlihat dan bisa di-toggle.
- [ ] Layer infrastruktur (Jalan Nasional, Kereta Api) terlihat dan bisa di-toggle.
- [ ] Tooltip muncul saat hover dengan atribut: `name`, `code`, `area_km2` / `length_km`.
- [ ] Highlight effect diterapkan saat hover.
- [ ] Zoom in/out dan pan berfungsi.
- [ ] Tombol Reset View menampilkan seluruh Indonesia.
- [ ] Coordinate bar menampilkan Lon/Lat real-time.
- [ ] API endpoints mengembalikan data valid (GeoJSON).
- [ ] FCP ≤ 2 detik, FPS ≥ 30 pada 5000 features.
- [ ] Tidak ada console error pada Chrome, Firefox, Safari.

### 10.2 Sprint 2 Success Criteria

- [ ] User dapat login dengan username/password.
- [ ] Role-based access: GISOfficer vs Admin.
- [ ] Admin dapat upload Shapefile/GeoJSON melalui UI.
- [ ] Validasi geometri (`ST_IsValid`) berjalan saat import.
- [ ] Import log menampilkan status sukses/gagal.
- [ ] Admin dapat edit metadata layer (nama, tipe, visibility).
- [ ] Layer baru (Sungai, Danau, Gunung, TPA, Konservasi) dapat ditambahkan.
- [ ] API rate limiting aktif (100 req/min).
- [ ] Structured logging berjalan untuk semua request.

### 10.3 Sprint 3 Success Criteria

- [ ] Measure tool dapat mengukur jarak dan luas area.
- [ ] Drawing tool dapat membuat point, line, polygon.
- [ ] Buffer tool menghasilkan kaersalahan spasial dari objek.
- [ ] Export PDF peta dengan legend dan scale bar.
- [ ] Copy atribut ke clipboard berfungsi.
- [ ] Dark/Light theme toggle berfungsi.
- [ ] Share view menghasilkan URL dengan state peta.

### 10.4 Sprint 4 Success Criteria

- [ ] Full-text search berdasarkan nama wilayah berfungsi.
- [ ] Export data ke GeoJSON, Shapefile, CSV, KML berfungsi.
- [ ] Data eksternal (BPS, BMKG) ter-sync otomatis.
- [ ] Webhook notification dikirim saat data berubah.
- [ ] Aplikasi responsif pada tablet (viewport 768px+).
- [ ] Async task queue berjalan untuk import besar.

---

## 11. Metrics & Monitoring

### 11.1 Sprint Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Sprint Velocity | ≥ 30 SP/sprint | Completed SP / Sprint |
| Task Completion Rate | ≥ 85% | Completed tasks / Committed tasks |
| Bug Escape Rate | ≤ 5% | Bugs found in production / Total bugs |
| Code Review Coverage | 100% | PRs dengan minimal 1 approval |
| Test Coverage | ≥ 70% | Unit test coverage (frontend + backend) |

### 11.2 Product Metrics (Tracked per Sprint)

| Metric | Sprint 1 Target | Sprint 2 Target | Sprint 3 Target | Sprint 4 Target |
|--------|-----------------|-----------------|-----------------|-----------------|
| FCP | ≤ 2.0 detik | ≤ 1.8 detik | ≤ 1.5 detik | ≤ 1.2 detik |
| FPS | ≥ 30 fps | ≥ 35 fps | ≥ 40 fps | ≥ 45 fps |
| API P95 | ≤ 200ms | ≤ 150ms | ≤ 100ms | ≤ 80ms |
| Uptime | 99% | 99.2% | 99.5% | 99.5% |

---

## 12. Appendix

### 12.1 Glossary

| Term | Definition |
|------|------------|
| **SP (Story Points)** | Unit estimasi relatif untuk kompleksitas, effort, dan risk sebuah user story. |
| **Velocity** | Rata-rata story points yang diselesaikan per sprint. |
| **DoR (Definition of Ready)** | Kriteria minimal sebelum task bisa diambil ke sprint. |
| **DoD (Definition of Done)** | Kriteria minimal sebelum task bisa di-mark "Done". |
| **GIST Index** | Generalized Search Tree index untuk kolom geometri PostGIS. |
| **MVT** | Mapbox Vector Tiles — format binary untuk vektor tiles. |
| **ST_Intersects** | Fungsi PostGIS untuk mengecek apakah dua geometri saling bersinggungan. |
| **CRS** | Coordinate Reference System — sistem referensi koordinat spasial. |

### 12.2 References

- `docs/prd.md` — Product Requirements Document
- `docs/product-backlog.md` — Product Backlog
- `docs/srs.md` — Software Requirements Specification
- `docs/domain-model.md` — Domain Model
- `docs/erd.md` — Entity Relationship Diagram
- `docs/api-spec.md` — API Specification
- `docs/ui-spec.md` — UI Specification
- `docs/prd-gap-analysis.md` — PRD Gap Analysis

---

*Document Owner: Product Management & Scrum Master*
*Last Review: 2026-06-17*
*Next Review: 2026-06-24 (Sprint 1 Day 1)*
*Distribution: Engineering Team, Stakeholders*
