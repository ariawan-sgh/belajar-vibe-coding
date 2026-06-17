# PRD Gap Analysis — WebGIS

**Tanggal Review:** 2026-06-17
**Reviewer:** Senior Product Manager & Business Analyst
**Basis Review:** `docs/prd.md` v1.0

---

## Ringkasan Eksekutif

Dokumen PRD WebGIS sudah memiliki fondasi yang solid dengan cakupan fitur inti yang jelas dan arsitektur modern. Namun, terdapat **gap signifikan** pada area *data management foundation*, *error handling*, *cartographic essentials*, *operational readiness*, dan *testing strategy*. Gap ini dibagi menjadi tiga kategori berdasarkan urgensi: **Critical** (menghalangi go-live), **High Priority** (risiko tinggi jika tidak diatasi), dan **Medium/Low** (perlu klarifikasi atau roadmap).

---

## 1. Critical Gaps (Must Fix Before Launch)

### 1.1 [GAP-DATA-01] Tidak Ada Strategi Data Seeding & Initial Population
**Lokasi Gap:** Seluruh dokumen, terutama Section 9.5 (Database Schema) dan Business Workflow.
**Deskripsi:** PRD mendefinisikan tabel-tabel spasial (`admin_boundaries`, `infrastructure`) tetapi tidak menjelaskan:
- Bagaimana dataset awal di-populate (data source, format, tool ETL).
- Apakah ada script migrasi SQL atau seeder untuk data awal Batas Administrasi Indonesia.
- Proses validasi awal sebelum data masuk production (data quality check).
**Dampak:** Tim engineering tidak tahu dataset awal dari mana dan kualitasnya seperti apa, berpotensi mogok saat onboarding.
**Rekomendasi:** Tambahkan **Appendix D: Data Ingestion & Seeding Strategy** yang mencakup:
```markdown
1. Sumber data resmi: BPS, Kemendagri, atau open data Batas Administratif Indonesia.
2. Format input: Shapefile (.shp) dengan CRS EPSG:4326.
3. Tool ETL: `shp2pgsql` atau `ogr2ogr` dengan parameter `-s 4326:4326 -lco GEOMETRY_NAME=geom`.
4. Data quality gate: ST_IsValid(), ST_SRID(), duplicate check.
```

### 1.2 [GAP-DATA-02] Tidak Ada Konvensi Atribut & Metadata Layer
**Lokasi Gap:** Section 9.3 (Backend API) dan 9.5 (Database Schema).
**Deskripsi:** Endpoint `/api/v1/metadata/attributes/:layerId` dijanjikan tetapi tidak ada spesifikasi:
- Bagaimana metadata atribut (description, data type, unit of measure) disimpan dan dikelola.
- Apakah ada UI untuk admin mengelola metadata, atau hardcoded di database.
- Standar penulisan field name (snake_case vs camelCase antara frontend dan backend).
**Dampak:** Tooltip bisa menampilkan atribut yang tidak bermakna atau不一致 antara frontend dan backend.
**Rekomendasi:** Tambahkan spesifikasi atribut untuk setiap layer:
| Layer | Field Name | Tipe Data | Deskripsi | Satuan |
|-------|-----------|----------|-----------|--------|
| admin_boundaries | province_name | VARCHAR | Nama provinsi | - |
| admin_boundaries | area_km2 | NUMERIC | Luas wilayah | km² |
| infrastructure | condition | VARCHAR | Kondisi jalan | enum: baik/sedang/rusak |

### 1.3 [GAP-AUTH-03] Tidak Ada Autentikasi & Authorization Meski Ada User Role Admin
**Lokasi Gap:** Section 4.2 (System Administrator) dan Future Roadmap.
**Deskripsi:** PRD mendefinisikan role System Administrator dengan write access, tetapi:
- Tidak ada mekanisme autentikasi untuk GIS Officer maupun Admin di Sprint 1.
- Tidak jelas apakah aplikasi dibuka secara public atau dibatasi domain/internal.
- Tanpa auth, tidak ada dasar untuk SSO atau integrasi dengan sistem internal organisasi.
**Dampak:** Jika aplikasi di-deploy ke public URL, data spasial bisa diakses tanpa kontrol. Jika internal tapi tanpa SSO, manajemen credentials menyebar.
**Rekomendasi:** Even if minimal, tambahkan:
1. **Sprint 0 (pre-requisite):** Basic authentication untuk membedakan internal vs external access.
2. **Sprint 2:** JWT-based authentication sesuai roadmap, tetapi definisikan contract API-nya sekarang.

---

## 2. High Priority Gaps (Significant Risk)

### 2.1 [GAP-ERR-04] Tidak Ada Requirement untuk Error, Empty, & Loading States
**Lokasi Gap:** Section 6 (Functional Requirements).
**Deskripsi:** Tidak ada spesifikasi untuk:
- **Loading:** Apa yang ditampilkan saat base map tiles atau vector layer sedang dimuat (spinner, skeleton, progress bar)?
- **Empty state:** Apa yang ditampilkan jika query spatial mengembalikan 0 features (misal zoom terlalu dalam).
- **Error state:** Apa yang ditampilkan jika backend API gagal (500, timeout, CORS error). Apakah ada toast notification atau retry mechanism?
- **Fallback:** Apa yang terjadi jika base map tile provider down? Apakah ada fallback provider?
**Dampak:** Pengalaman pengguna (UX) akan terasa tidak profesional saat ada masalah jaringan atau API. GIS Officer mungkin mengira peta "rusak" padahal hanya loading atau error.
**Rekomendasi:** Tambahkan FR baru:
```markdown
### 6.6 FR-06: Loading, Error & Empty States
| ID | Requirement | Detail |
|----|------------|--------|
| FR-06.1 | Loading Indicator | Menampilkan indikator loading saat base map tiles atau vector layer sedang di-fetch. |
| FR-06.2 | Empty State Message | Menampilkan pesan informatif jika tidak ada feature dalam viewport saat ini. |
| FR-06.3 | Error Handling | Menampilkan toast notification jika API request gagal, dengan opsi retry. |
| FR-06.4 | Fallback Tile Provider | Jika primary tile provider gagal, fallback ke secondary provider. |
```

### 2.2 [GAP-CART-05] Tidak Ada Legend & Scale Bar
**Lokasi Gap:** Section 9.1 (Frontend Components).
**Deskripsi:** PRDDefines komponen untuk zoom control, layer switcher, tooltip, dan coordinate bar — tetapi **tidak ada legend** dan **tidak ada scale bar**. Kedua elemen ini adalah standar cartographic untuk aplikasi GIS.
- **Legend:** Memberikan penjelasan warna, simbol, dan garis untuk setiap layer.
- **Scale bar:** Memberikan konteks jarak pada peta saat berbagai zoom level.
**Dampak:** Pengguna tidak tahu apakah warna merah adalah "Jalan Nasional dalam kondisi rusak" atau "Batas provinsi". Tanpa scale bar, tidak ada konteks jarak spasial.
**Rekomendasi:** Tambahkan komponen baru:
| Komponen | Teknologi | Deskripsi |
|----------|----------|-----------|
| `<webgis-legend>` | Lit JS | Panel atau overlay yang menampilkan penjelasan simbol, warna, dan garis untuk setiap layer aktif. |
| `<webgis-scale-bar>` | Lit JS | Bar skal visual di pojok peta yang menunjukkan jarak pada zoom level saat ini. |

### 2.3 [GAP-ATTR-06] Tidak Ada Requirement untuk Map Attribution
**Lokasi Gap:** Section 6.1 (Base Map Rendering) dan 9.5 (Database Schema — kolom `attribution`).
**Deskripsi:**
- Kolom `attribution` ada di tabel `layers` tetapi tidak ada FR yang menspesifikasikan cara menampilkannya di UI.
- Base map dari OSM memerlukan attribusi yang sesuai dengan license (© OpenStreetMap contributors).
- Data vektor (Batas Administrasi) kemungkinan berasal dari sumber terbuka yang juga memerlukan attribution.
**Dampak:** Risiko pelanggaran lisensi (legal risk) dan kurangnya transparansi sumber data.
**Rekomendasi:** Tambahkan spesifikasi di Section 6.1:
- Semua layer (base map dan vector) harus menampilkan attribution sesuai license.
- Untuk OSM, gunakan kontrol attribution bawaan OpenLayers atau custom component.
- Attribution ditampilkan di pojok bawah peta, sesuai standar OSM.

### 2.4 [GAP-COORD-07] Inkonsistensi Penanganan Coordinate Reference System (CRS)
**Lokasi Gap:** Appendix B poin 1 vs Section 6.1.2.
**Deskripsi:** Ada dua pernyataan yang saling bertentangan:
- *Appendix B poin 1:* "Selalu transformasi EPSG:4326 → EPSG:3857 di sisi backend (`ST_Transform(geom, 3857)`) sebelum diserialisasi ke GeoJSON."
- *Section 6.1.2:* "Base map menggunakan proyeksi EPSG:3857 ... dengan dukungan transformasi koordinat ke EPSG:4326 (WGS84)."
- Namun, tidak ada spesifikasi CRS untuk vector data yang disimpan di database (schema menulis `GEOMETRY(MultiPolygon, 4326)`).
- Jika data disimpan di 4326 tetapi di-transform ke 3857 di backend, maka query spatial (`bbox`) juga harus di-transform.
**Dampak:** Potensi mismatch koordinat, feature tidak muncul, atau query tidak efisien.
**Rekomendasi:** Chọn satu strategi dan konsistenkan seluruh dokumen:
- **Opsi A (Rekomendasi):** Simpan semua geometri di database dalam EPSG:4326. Backend menerima bbox dalam EPSG:4326 dari frontend (karena OpenLayers mengirim koordinat viewport dalam proyeksi peta). Query PostGIS menggunakan `ST_Transform(geom, 4326)` untuk hit-test atau kirim bbox dalam 3857 dan transform di query.
- Dokumentasikan alur CRS secara eksplisit di Section 6.1.2.

### 2.5 [GAP-PERF-08] Tidak Ada Strategi untuk Generalization / Clustering
**Lokasi Gap:** Section 6.5 (Responsive & Performance) dan 11 (KPI).
**Deskripsi:** KR FPS 30 fps diatur untuk 5.000 features, tetapi:
- Tidak ada mekanisme untuk menangani jumlah features yang lebih besar (misal 50.000+ saat zoom out ke seluruh Indonesia).
- Tidak ada generalisasi (simplification) geometri untuk low zoom level.
- Tidak ada spatial clustering untuk point features (jika nanti ada layer titik).
- Tidak ada batas maksimal features per response.
**Dampak:** Performa akan anjlok saat user zoom out ke tingkat nasional karena ribuan features dimuat sekaligus.
**Rekomendasi:** Tambahkan requirement:
- Backend harus mendukung `simplification` parameter (epsilon Douglas-Peucker) berdasarkan zoom level.
- Frontend harus melakukan client-side clustering atau vector tile aggregation untuk >10.000 features.
- Response API dibatasi maksimal 10.000 features per request (batching atau pagination jika perlu).

---

## 3. Medium / Low Priority Gaps (Clarification & Roadmap)

### 3.1 [GAP-SEARCH-09] Tidak Ada Search / Geocoding
**Lokasi Gap:** Seluruh dokumen.
**Deskripsi:** GIS Officer mungkin perlu mencari lokasi tertentu (nama provinsi, kota, koordinat). Saat ini hanya navigasi manual (pan/zoom).
**Rekomendasi:**
- **Sprint 1:** Tambahkan search sederhana menggunakan Nominatim (OSM) atau library geocoding ringan.
- **Sprint 2+:** Tambahkan search pada atribut spasial (cari berdasarkan kode atau nama wilayah).

### 3.2 [GAP-API-10] Tidak Ada Spesifikasi Error Response & Rate Limiting
**Lokasi Gap:** Section 9.3 (Backend API).
**Deskripsi:**
- Tidak ada spesifikasi format response ketika terjadi error (structure JSON, HTTP status code).
- Tidak ada rate limiting untuk mencegah abuse API.
- Tidak ada pagination mechanism untuk response yang besar.
**Rekomendasi:** Tambahkan ke Section 9.3 atau buat Section baru `9.5 API Standards`:
- Semua error response mengikuti format: `{"error": {"code": "string", "message": "string", "details": {}}}`
- Rate limiting: 100 requests/minute per IP (adjustable).
- Max response size: 10MB.

### 3.3 [GAP-TEST-11] Tidak Ada Testing Strategy yang Eksplisit
**Lokasi Gap:** Section 7.4 (Maintainability) — NFR-12.
**Deskripsi:** CI/CD disebutkan tetapi tidak ada requirement untuk:
- Test coverage minimum (unit test, integration test).
- Testing framework yang digunakan (Jest/Vitest untuk frontend, testify untuk Go).
- E2E testing atau visual regression testing untuk peta.
**Rekomendasi:** Tambahkan NFR baru atau klarifikasi NFR-12:
- Frontend: Vitest + Testing Library, coverage ≥ 70%.
- Backend: testify, coverage ≥ 70%.
- E2E (opsional): Playwright untuk smoke test UI.

### 3.4 [GAP-DEPLOY-12] Tidak Ada Deployment & Infrastructure Architecture
**Lokasi Gap:** Seluruh dokumen.
**Deskripsi:** Tidak ada spesifikasi:
- Bagaimana aplikasi di-deploy (Docker, Kubernetes, VPS, PaaS).
- Environment variables management (database credentials, CORS origins).
- CDN untuk tiles atau static assets.
- Infrastructure cost estimation.
**Rekomendasi:** Tambahkan `Appendix C: Deployment Architecture`:
- Frontend: Static assets hosted di Vercel/Netlify atau Nginx reverse proxy.
- Backend: Docker container, deploy ke EC2/Cloud Run/DigitalOcean App Platform.
- Database: RDS PostgreSQL + PostGIS atau self-managed PostgreSQL.
- Redis: Opsional untuk caching API response.

### 3.5 [GAP-MONITOR-13] Tidak Ada Logging & Monitoring
**Lokasi Gap:** Section 7.2 (Reliability).
**Deskripsi:** Uptime dan backup disebutkan tetapi tidak ada:
- Application logging (structured logs untuk request/response).
- Error tracking (Sentry, Rollbar) untuk frontend dan backend.
- Metric collection (Prometheus + Grafana atau simple dashboard) untuk FPS, API latency.
- Alerting mechanism jika API down atau DB replication lag.
**Rekomendasi:** Tambahkan requirement di Section 7.2:
- Structured JSON logging untuk backend.
- Frontend error boundary dengan reporting ke backend.
- Daily uptime check menggunakan UptimeRobot atau sejenisnya.

### 3.6 [GAP-ACCESS-14] Tidak Ada Keyboard Navigation & Accessibility
**Lokasi Gap:** Section 7.5 (Browser Compatibility).
**Deskripsi:** Hanya browser support yang disebutkan. Tidak ada:
- Keyboard shortcut untuk zoom (+/-) atau fit-to-extent.
- ARIA labels untuk controls.
- Focus management untuk tooltip.
- Screen reader support untuk feature attributes.
**Dampak:** Aplikasi tidak ramah aksesibilitas dan tidak memenuhi standar WCAG 2.1 (jika organisasi memerlukannya).
**Rekomendasi:** Tambahkan NFR baru:
- Semua interactive controls dapat diakses via keyboard.
- Tooltip menyertainya ARIA live region atau role="tooltip".

### 3.7 [GAP-LEGAL-15] Data Licensing & Terms of Service
**Lokasi Gap:** Seluruh dokumen.
**Deskripsi:** Tidak ada klausul tentang:
- Sumber data (siapa pemilik, license: ODbL untuk OSM, CC-BY untuk dataset pemerintah).
- Terms of service untuk penggunaan aplikasi.
- Data privacy policy (meskipun data spasial publik, tetap perlu disclaimer).
**Rekomendasi:** Tambahkan `Appendix E: Legal & Licensing`:
- Base map: OpenStreetMap (ODbL).
- Administrative boundaries: Sesuaikan dengan license data sumber (misal CC-BY 4.0 dari Kemendagri).
- Infrastructure: OpenStreetMap atau dataset pemerintah.

### 3.8 [GAP-COPY-16] Fitur Copy to Clipboard untuk Atribut
**Lokasi Gap:** Section 6.3 (Hover Interaction & Tooltip).
**Deskripsi:** Tooltip menampilkan atribut tetapi tidak ada cara untuk menyalin nilai (misal: kode wilayah, koordinat) ke clipboard.
**Rekomendasi:** Tambahkan FR-03.7:
- Tooltip menyediakan tombol "Copy" untuk menyalin atribut tertentu (nama, kode, koordinat) ke clipboard.

---

## 4. Summary Table Gaps

| ID | Kategori | Deskripsi Singkat | Prioritas |
|----|---------|-------------------|-----------|
| GAP-DATA-01 | Data Management | Tidak ada strategi seeding & initial population | **Critical** |
| GAP-DATA-02 | Data Management | Tidak ada konvensi atribut & metadata | **Critical** |
| GAP-AUTH-03 | Security | Tidak ada auth meski ada role admin | **Critical** |
| GAP-ERR-04 | UX/UI | Tidak ada error/empty/loading states | **High** |
| GAP-CART-05 | Cartography | Tidak ada legend & scale bar | **High** |
| GAP-ATTR-06 | Legal | Tidak ada map attribution requirement | **High** |
| GAP-COORD-07 | Data/CRS | Inkonsistensi transformasi koordinat | **High** |
| GAP-PERF-08 | Performance | Tidak ada generalization/clustering strategy | **High** |
| GAP-SEARCH-09 | UX/UI | Tidak ada search/geocoding | Medium |
| GAP-API-10 | Backend | Tidak ada error response & rate limiting | Medium |
| GAP-TEST-11 | Engineering | Tidak ada testing strategy eksplisit | Medium |
| GAP-DEPLOY-12 | Engineering | Tidak ada deployment architecture | Medium |
| GAP-MONITOR-13 | Engineering | Tidak ada logging & monitoring | Medium |
| GAP-ACCESS-14 | Accessibility | Tidak ada keyboard navigation & ARIA | Medium |
| GAP-LEGAL-15 | Legal | Tidak ada data licensing & ToS | Low |
| GAP-COPY-16 | UX/UI | Tidak ada copy-to-clipboard di tooltip | Low |

---

## 5. Rekomendasi Prioritas Perbaikan

### Before Sprint 1 Kickoff
1. **Resolve GAP-DATA-01 & GAP-DATA-02** — Finalisasi dataset sumber, ETL script, dan metadata atribut sebelum development dimulai.
2. **Fix GAP-COORD-07** — Konsolidasikan strategi CRS dan dokumentasikan di Section 6.1.2.
3. **Add GAP-AUTH-03** — Minimal basic auth untuk internal access.

### During Sprint 1
4. **Add GAP-ERR-04** — Implementasi loading/error/empty states.
5. **Add GAP-CART-05** — Tambahkan komponen legend & scale bar.
6. **Add GAP-ATTR-06** — Tambahkan attribution requirement.

### Post-Sprint 1 (Sprint 2+)
7. **Address GAP-PERF-08** — Generalization & clustering untuk performa.
8. **Address GAP-SEARCH-09** — Search/geocoding.
9. **Address remaining Medium/Low gaps** sesuai prioritas bisnis.

---

## 6. Catatan Tambahan

- **Konsistensi Naming Convention:** Pastikan penamaan field dan konvensi coding (snake_case vs camelCase) konsisten antara frontend (JavaScript/TypeScript) dan backend (Go).
- **API Versioning:** Endpoint menggunakan `/api/v1/` — bagus. Tambahkan strategi deprecation dan migration guide untuk versi mendatang.
- **Data Versioning:** Tidak ada mekanisme versioning untuk dataset spasial. Pertimbangkan menambahkan kolom `version` atau `valid_from` / `valid_to` jika data akan sering berubah.
- **Multi-tenancy:** Tidak dibutuhkan untuk Sprint 1, tetapi penting untuk diantisipasi jika organisasi ingin menampilkan data dari multiple wilayah atau cabang.

---

*Document Owner: Product Management & Business Analysis Team*
*Status: Draft — Open for Review*
*Next Review: Setelah finalisasi dataset dan before Sprint 1 kickoff*
