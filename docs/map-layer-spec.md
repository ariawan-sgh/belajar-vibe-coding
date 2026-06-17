# Map & Layer Specification — WebGIS

**Basis Dokumen:** `docs/prd.md` v1.0, `docs/domain-model.md`, `docs/erd.md`, `docs/ui-spec.md`
**Tanggal:** 2026-06-17
**Status:** Approved for Development

---

## Part 1: Map Specification

### 1.1 Map Engine Overview

| Property | Specification |
|----------|---------------|
| **Engine** | OpenLayers 9.x |
| **Rendering Mode** | Canvas (default) / WebGL (optional, untuk performa tinggi) |
| **Projection (Display)** | EPSG:3857 (Web Mercator) |
| **Projection (Storage)** | EPSG:4326 (WGS84) |
| **Coordinate Transform** | Proj4js atau built-in OpenLayers `transform()` |
| **Initial View** | Center `[118.0, -2.5]` (EPSG:3857), Zoom `4` |
| **Extent Indonesia** | `[95.3, -10.9, 141.0, 5.9]` (EPSG:4326) |

### 1.2 Tile Configuration

| Property | Specification |
|----------|---------------|
| **Provider** | OpenStreetMap (default) |
| **URL Template** | `https://{a-c}.tile.openstreetmap.org/{z}/{x}/{y}.png` |
| **Min Zoom** | 3 |
| **Max Zoom** | 18 |
| **Tile Size** | 256x256 pixels |
| **Pixel Ratio** | `window.devicePixelRatio` (support Retina) |
| **Cross-Origin** | `anonymous` (untuk canvas export) |
| **Attribution** | `© OpenStreetMap contributors` |

#### Tile Fallback

| Priority | Provider | URL Template |
|----------|----------|--------------|
| 1 (Primary) | OpenStreetMap | `https://{a-c}.tile.openstreetmap.org/{z}/{x}/{y}.png` |
| 2 (Fallback) | OSM Germany | `https://tile.openstreetmap.de/{z}/{x}/{y}.png` |
| 3 (Fallback) | CartoDB Positron | `https://{a-c}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}.png` |

### 1.3 Zoom Levels & Resolution

| Zoom Level | Resolution (m/px) | Scale (1:) | Content Detail |
|-----------|-------------------|------------|----------------|
| 3 | ~152.87 km | 1:5.8E7 | Seluruh Indonesia, level provinsi |
| 4 | ~76.43 km | 1:2.9E7 | Range kepulauan, provinsi utama |
| 5 | ~38.22 km | 1:1.5E7 | Pulau besar, batas provinsi |
| 6 | ~19.11 km | 1:7.2E6 | Kabupaten besar |
| 7 | ~9.56 km | 1:3.6E6 | Kabupaten |
| 8 | ~4.78 km | 1:1.8E6 | Kecamatan |
| 9 | ~2.39 km | 1:9.0E5 | Kecamatan detail |
| 10 | ~1.19 km | 1:4.5E5 | Kelurahan/Desa |
| 11 | ~596.56 m | 1:2.3E5 | Kampung, infrastruktur besar |
| 12 | ~298.28 m | 1:1.1E5 | Infrastruktur detail |
| 13 | ~149.14 m | 5.7E4 | Jalan lokal |
| 14 | ~74.57 m | 2.8E4 | Bangunan besar |
| 15 | ~37.28 m | 1.4E4 | Area lokal |
| 16 | ~18.64 m | 7.1E3 | Plot tanah |
| 17 | ~9.32 m | 3.5E3 | Detail kecil |
| 18 | ~4.66 m | 1.8E3 | Street-level |

### 1.4 View Constraints

| Property | Value | Description |
|----------|-------|-------------|
| `minZoom` | 3 | Tidak mungkin zoom out ke level < 3 |
| `maxZoom` | 18 | Tidak mungkin zoom in ke level > 18 |
| `extent` | `[95.3, -10.9, 141.0, 5.9]` (4326) | Bounding box Indonesia (EPSG:4326), di-transform ke 3857 |
| `padding` | `[50, 50, 50, 50]` | Padding default untuk `fit()` operation |
| `duration` | `500ms` | Durasi animasi untuk `fit()` dan `animate()` |
| `centerOnRotation` | `true` | Rotasi peta mempertahankan center |
| `multiWorld` | `false` | Hanya menampilkan satu dunia (tidak repeat tiles) |

### 1.5 Interactions Configuration

| Interaction | Type | Configuration |
|-------------|------|---------------|
| **Mouse Wheel Zoom** | `MouseWheelZoom` | `duration: 250ms`, `useAnchor: true` |
| **Double Click Zoom** | `DoubleClickZoom` | `duration: 250ms` |
| **Drag Pan** | `DragPan` | `kinetic: true`, `decayRate: 0.95`, `minVelocity: 0.1` |
| **Pinch Zoom** | `PinchZoom` | `duration: 100ms` (untuk mobile future) |
| **Keyboard Zoom** | `KeyboardPan` + custom | `+`/`-` untuk zoom, arrow keys untuk pan |
| **Attribution** | `Attribution` | `collapsible: true`, `tipLabel: "Map attribution"` |

### 1.6 Layer Ordering (Z-Index)

| Z-Index | Layer | Description |
|---------|-------|-------------|
| 0 | Base Map (OSM) | Tile layer dasar |
| 1 | Administrative Boundaries | Batas provinsi, kabupaten |
| 2 | Infrastructure | Jalan, kereta api, bandara, bendungan |
| 3 | Custom Layers (Sprint 2+) | Layer yang diupload admin |
| 4 | Vector Overlay | Highlight effect, measure, draw annotations |
| 5 | Scale Bar | Scale bar OpenLayers control |
| 6 | Interaction Overlay | Crosshair cursor, coordinate overlay |

### 1.7 Coordinate Systems

#### EPSG:4326 (WGS84) — Storage & API

| Property | Value |
|----------|-------|
| **Name** | WGS 84 |
| **Code** | EPSG:4326 |
| **Unit** | Degree |
| **X-Range** | -180 to 180 |
| **Y-Range** | -90 to 90 |
| **Usage** | Database storage, API request/response, coordinate display |

#### EPSG:3857 (Web Mercator) — Display

| Property | Value |
|----------|-------|
| **Name** | WGS 84 / Pseudo-Mercator |
| **Code** | EPSG:3857 |
| **Unit** | Metre |
| **X-Range** | -20,037,508.34 to 20,037,508.34 |
| **Y-Range** | -20,037,508.34 to 20,037,508.34 |
| **Usage** | OpenLayers rendering, tile coordinates |

#### CRS Transformation Rules

| From | To | Function | Usage |
|------|----|----------|-------|
| EPSG:4326 | EPSG:3857 | `transform([lon, lat], 'EPSG:4326', 'EPSG:3857')` | Set view center from API data |
| EPSG:3857 | EPSG:4326 | `transform([x, y], 'EPSG:3857', 'EPSG:4326')` | Display coordinate in bottom bar |
| EPSG:4326 | EPSG:3857 | `ST_Transform(geom, 4326, 3857)` (PostGIS) | Backend: sebelum kirim GeoJSON ke frontend |

### 1.8 Map State Serialization (Share View)

Untuk fitur "Share View" (Sprint 3), state peta dapat di-serialisasi ke URL parameter:

```typescript
interface MapState {
  center: [number, number];      // EPSG:4326 [lon, lat]
  zoom: number;                  // 3-18
  rotation: number;              // 0 (default)
  layers: string[];              // Array of layer IDs yang aktif
}

// Serialization
const state: MapState = {
  center: [118.0, -2.5],
  zoom: 4,
  rotation: 0,
  layers: ["admin_boundaries", "infrastructure"]
};

const encoded = btoa(JSON.stringify(state));
// URL: /?map=eyJjZW50ZXIiOlsxLjgsIC0yLjVdLCAiem9vbSI6NCwg...
```

---

## Part 2: Layer Specification

### 2.1 Layer Architecture Overview

Setiap layer di WebGIS memiliki struktur berikut:

```
Layer
├── Metadata (static / dynamic)
│   ├── id, name, description, type
│   ├── geom_type, srid, attribution
│   ├── default_visibility, z_index
│   └── attributes (dynamic definitions)
├── Source (data provider)
│   ├── type: "postgis" | "geojson" | "mvt" | "osm"
│   ├── table: (untuk postgis)
│   ├── url: (untuk geojson/mvt)
│   └── fields: array of field names
├── Style (appearance)
│   ├── default style (per geometry type)
│   ├── hover style
│   ├── selected style
│   └── scale-dependent style rules
└── Interaction (behavior)
    ├── hover: tooltip + highlight
    ├── click: select / zoom to feature
    ├── filter: by attribute value
    └── cluster: untuk point features (future)
```

### 2.2 Base Map Layer

| Property | Specification |
|----------|---------------|
| **Layer ID** | `base_map` |
| **Layer Name** | "Peta Dasar Indonesia" |
| **Layer Type** | `base_map` |
| **Geometry Type** | N/A (tile-based) |
| **Source Type** | `xyz` (OpenStreetMap tile server) |
| **Source URL** | `https://{a-c}.tile.openstreetmap.org/{z}/{x}/{y}.png` |
| **SRID** | N/A (tile sudah dalam EPSG:3857) |
| **Default Visibility** | `true` |
| **Z-Index** | `0` |
| **Attribution** | `© OpenStreetMap contributors` |
| **Opacity** | `1.0` (100%) |
| **Min Zoom** | 3 |
| **Max Zoom** | 18 |

**Style:** Tile-based, tidak memiliki style specification selain attribution.

### 2.3 Administrative Boundaries Layer

| Property | Specification |
|----------|---------------|
| **Layer ID** | `admin_boundaries` |
| **Layer Name** | "Batas Administrasi Indonesia" |
| **Layer Type** | `administrative_boundary` |
| **Geometry Type** | `MultiPolygon` |
| **Source Type** | `postgis` |
| **Source Table** | `admin_boundaries` (via view `v_admin_boundaries_hierarchy`) |
| **SRID** | 4326 |
| **Default Visibility** | `true` |
| **Z-Index** | `1` |
| **Attribution** | `© Badan Pusat Statistik / Kemendagri` |
| **Opacity** | `1.0` |

#### Attribute Definitions

| Field | Type | Tooltip | Description |
|-------|------|---------|-------------|
| `id` | integer | hidden | Internal database ID |
| `level` | string | visible | Level administrasi (province/regency/district/village) |
| `code` | string | visible | Kode Kemendagri/BPS |
| `name` | string | visible | Nama wilayah |
| `province_name` | string | visible (jika level != province) | Nama provinsi induk |
| `regency_name` | string | visible (jika level == district/village) | Nama kabupaten induk |
| `district_name` | string | visible (jika level == village) | Nama kecamatan induk |
| `population` | bigint | visible | Jumlah penduduk |
| `area_km2` | numeric | visible | Luas wilayah (km²) |

#### Style Rules

| Condition | Fill | Stroke | Stroke Width |
|-----------|------|--------|--------------|
| `level = 'province'` | `transparent` | `#2563EB` (blue-600) | `2px` |
| `level = 'regency'` | `transparent` | `#3B82F6` (blue-500) | `1.5px` |
| `level = 'district'` | `transparent` | `#60A5FA` (blue-400) | `1px` |
| `level = 'village'` | `transparent` | `#93C5FD` (blue-300) | `0.75px` |

#### Hover Style

| Property | Value |
|----------|-------|
| Fill | `rgba(37, 99, 235, 0.15)` |
| Stroke | `#00FFFF` (cyan-400) |
| Stroke Width | +1px dari default |

### 2.4 Infrastructure Layer

| Property | Specification |
|----------|---------------|
| **Layer ID** | `infrastructure` |
| **Layer Name** | "Infrastruktur Publik" |
| **Layer Type** | `infrastructure` |
| **Geometry Type** | `LineString` (default), `Point` (airport/dam) |
| **Source Type** | `postgis` |
| **Source Table** | `infrastructure` (via view `v_infrastructure_active`) |
| **SRID** | 4326 |
| **Default Visibility** | `true` |
| **Z-Index** | `2` |
| **Attribution** | `© OpenStreetMap contributors / Kementerian PUPR` |
| **Opacity** | `1.0` |

#### Attribute Definitions

| Field | Type | Tooltip | Description |
|-------|------|---------|-------------|
| `id` | integer | hidden | Internal database ID |
| `type` | string | visible | Jenis infrastruktur (national_road, railway, airport, dam) |
| `name` | string | visible | Nama fasilitas |
| `code` | string | visible | Kode identifikasi |
| `condition` | string | visible | Kondisi (baik/sedang/rusak) |
| `length_km` | numeric | visible | Panjang fasilitas (km) |

#### Style Rules

| Type | Stroke | Stroke Width | Icon (future) |
|------|--------|--------------|---------------|
| `national_road` | `#F59E0B` (amber-500) | `2px` | 🛣 |
| `railway` | `#8B5CF6` (violet-500) | `2px` | 🚂 |
| `airport` | `#EF4444` (red-500) | `3px` | ✈️ |
| `dam` | `#0891B2` (cyan-600) | `3px` | 🏗

### 2.5 Custom Layers (Sprint 2+)

| Property | Specification |
|----------|---------------|
| **Layer ID** | `custom_{uuid}` |
| **Layer Name** | User-defined |
| **Layer Type** | `custom` |
| **Geometry Type** | Dinamis (Point/LineString/Polygon/MultiPolygon) |
| **Source Type** | `postgis` (setelah upload) atau `geojson` (client-side) |
| **Source Table** | `infrastructure` (generic) atau tabel custom |
| **Default Visibility** | `false` (tidak aktif saat dibuat) |
| **Z-Index** | `3` (di atas layer default) |
| **Attribution** | User-defined |

#### Default Style untuk Custom Layers

| Geometry Type | Fill | Stroke | Stroke Width |
|---------------|------|--------|--------------|
| Point | `#6366F1` (indigo-500) r=6px | `#FFFFFF` width 2px | - |
| LineString | `transparent` | `#6366F1` | `2px` |
| Polygon | `rgba(99, 102, 241, 0.3)` | `#6366F1` | `2px` |

### 2.6 Future Layers (Roadmap)

| Layer ID | Layer Name | Geometry Type | Source | Sprint Target |
|----------|-----------|--------------|--------|---------------|
| `rivers` | Sungai | LineString | OSM / Daftar Sungai Indonesia | Sprint 2 |
| `lakes` | Danau | Polygon | OSM / DEMNAS | Sprint 2 |
| `mountains` | Gunung | Point | GADM / BGW/ | Sprint 2 |
| `landfills` | TPA / SPAL | Point | PTPP / DLH | Sprint 2 |
| `conservation` | Kawasan Konservasi | Polygon | KemenLHK / GTK | Sprint 2 |
| `heatmap` | Heatmap (Kepadatan) | Point (cluster) | Custom | Sprint 3 |
| `time_series` | Time-Series | Dynamic | Historical data | Sprint 4 |

---

## Part 3: Layer Data Model

### 3.1 Layer Configuration Schema (JSON)

```json
{
  "layer_id": "string (unique identifier)",
  "name": "string (display name)",
  "description": "string",
  "type": "base_map | administrative_boundary | infrastructure | custom",
  "geom_type": "Point | LineString | Polygon | MultiPolygon",
  "source": {
    "type": "postgis | geojson | mvt | xyz",
    "table": "string (untuk postgis)",
    "url": "string (untuk geojson/mvt/xyz)",
    "fields": ["string (array of field names)"],
    "params": {
      "bbox": "string format",
      "zoom_level": "integer"
    }
  },
  "style": {
    "type": "style",
    "rules": [
      {
        "condition": "level == 'province'",
        "style": {
          "stroke": "#2563EB",
          "stroke_width": 2,
          "fill": "transparent"
        }
      }
    ],
    "default_style": {
      "stroke": "#3B82F6",
      "stroke_width": 1.5,
      "fill": "transparent"
    },
    "hover_style": {
      "stroke": "#00FFFF",
      "stroke_width": 3,
      "fill": "rgba(0, 255, 255, 0.15)"
    }
  },
  "interaction": {
    "hover": {
      "enabled": true,
      "tooltip_fields": ["name", "code", "area_km2"],
      "highlight": true
    },
    "click": {
      "enabled": true,
      "action": "zoom_to | select | none"
    },
    "filter": {
      "enabled": true,
      "fields": ["level", "type", "province_name"]
    }
  },
  "visibility": {
    "default": true,
    "min_zoom": 3,
    "max_zoom": 18
  },
  "z_index": 1,
  "attribution": "string",
  "metadata": {
    "created_by": "string",
    "created_at": "ISO8601",
    "updated_at": "ISO8601",
    "record_count": "integer",
    "crs": "EPSG:4326"
  }
}
```

### 3.2 OpenLayers Layer Factory Pattern

```typescript
interface LayerConfig {
  id: string;
  name: string;
  type: LayerType;
  source: VectorSource | TileSource;
  style: StyleFunction | Style;
  zIndex: number;
  visible: boolean;
  minZoom?: number;
  maxZoom?: number;
}

class LayerFactory {
  static createBaseMapLayer(): TileLayer<OSMSource> {
    return new TileLayer({
      source: new OSM({
        url: 'https://{a-c}.tile.openstreetmap.org/{z}/{x}/{y}.png',
        attributions: '© OpenStreetMap contributors',
        crossOrigin: 'anonymous',
      }),
      zIndex: 0,
      visible: true,
    });
  }

  static createVectorLayer(config: LayerConfig): VectorLayer {
    const vectorSource = new VectorSource({
      format: new GeoJSON(),
      url: config.source.url,
      strategy: bboxStrategy, // load on moveend
    });

    return new VectorLayer({
      source: vectorSource,
      style: config.style,
      zIndex: config.zIndex,
      visible: config.visible,
      minZoom: config.minZoom,
      maxZoom: config.maxZoom,
    });
  }
}
```

---

## Part 4: Styling Specification

### 4.1 Color Palette Reference

| Role | Color | Hex | Tailwind |
|------|-------|-----|----------|
| Primary | Blue 600 | `#2563EB` | `blue-600` |
| Primary Hover | Blue 700 | `#1D4ED8` | `blue-700` |
| Primary Light | Blue 100 | `#DBEAFE` | `blue-100` |
| Highlight | Cyan 400 | `#00FFFF` | `cyan-400` |
| Success | Emerald 600 | `#059669` | `emerald-600` |
| Warning | Amber 500 | `#F59E0B` | `amber-500` |
| Error | Red 600 | `#DC2626` | `red-600` |
| Info | Cyan 600 | `#0891B2` | `cyan-600` |
| Infrastructure - Road | Amber 500 | `#F59E0B` | `amber-500` |
| Infrastructure - Railway | Violet 500 | `#8B5CF6` | `violet-500` |
| Panel Background | Gray 800 | `#1F2937` | `gray-800` |
| Panel Hover | Gray 700 | `#374151` | `gray-700` |
| Text on Dark | Gray 50 | `#F9FAFB` | `gray-50` |
| Text Muted | Gray 400 | `#9CA3AF` | `gray-400` |
| Tooltip Background | White | `#FFFFFF` | `white` |

### 4.2 Style Composition Rules

| Layer Type | Fill | Stroke | Stroke Width | Hover Fill | Hover Stroke | Hover Width |
|-----------|------|--------|--------------|------------|--------------|-------------|
| Admin Province | transparent | `blue-600` | 2px | `rgba(37,99,235,0.15)` | `cyan-400` | 3px |
| Admin Regency | transparent | `blue-500` | 1.5px | `rgba(37,99,235,0.15)` | `cyan-400` | 2.5px |
| Admin District | transparent | `blue-400` | 1px | `rgba(37,99,235,0.15)` | `cyan-400` | 2px |
| Admin Village | transparent | `blue-300` | 0.75px | `rgba(37,99,235,0.15)` | `cyan-400` | 1.5px |
| Road (national) | transparent | `amber-500` | 2px | `rgba(245,158,11,0.2)` | `cyan-400` | 4px |
| Railway | transparent | `violet-500` | 2px | `rgba(139,92,246,0.2)` | `cyan-400` | 4px |
| Airport | transparent | `red-500` | 3px | `rgba(239,68,68,0.2)` | `cyan-400` | 4px |
| Dam | transparent | `cyan-600` | 3px | `rgba(8,145,178,0.2)` | `cyan-400` | 4px |

### 4.3 Zoom-Dependent Styling

| Zoom Level Range | Admin Stroke Width | Infrastructure Stroke Width | Detail Level |
|-----------------|-------------------|---------------------------|--------------|
| 3-5 | Province only (2px) | Hide infra | National overview |
| 6-8 | Province (2px), Regency (1.5px) | Show major roads/railways | Province detail |
| 9-11 | Province, Regency, District (1px) | Show all roads/railways | Kabupaten detail |
| 12-14 | + Village (0.75px) | Show airports, dams | Kecamatan detail |
| 15-18 | All levels full detail | All features | Street level |

---

## Part 5: Performance Specification

### 5.1 Rendering Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| FPS (idle) | ≥ 30 fps | Custom FPS counter |
| FPS (pan/zoom) | ≥ 30 fps | OpenLayers frame monitor |
| FCP | ≤ 2 detik | Lighthouse |
| TTI | ≤ 4 detik | Lighthouse |
| Tooltip latency | ≤ 50ms | `performance.now()` measurement |

### 5.2 Optimization Strategies

| Strategy | Implementation | Trigger |
|----------|---------------|---------|
| **Geometry Simplification** | `ST_SimplifyPreserveTopology(geom, tolerance)` di backend | Zoom level < 12 |
| **BBOX-based Loading** | Fetch hanya features dalam viewport | `moveend` event |
| **Debounced Tooltip** | `requestAnimationFrame` + 16ms debounce | `pointermove` event |
| **Tile Caching** | Browser cache + Service Worker (opsional) | Automatic |
| **Layer Clustering** | `ol/source/Cluster` untuk point features | > 1000 points |
| **WebGL Renderer** | `ol/renderer/WebGL` (opt-in) | > 10,000 features |

### 5.3 Simplification Tolerance by Zoom

| Zoom Level | Tolerance (degrees) | Approx. Distance | Vertex Reduction |
|-----------|---------------------|-----------------|-----------------|
| 3-5 | 0.01 | ~1.1 km | 80% reduction |
| 6-8 | 0.001 | ~110 m | 50% reduction |
| 9-11 | 0.0001 | ~11 m | 20% reduction |
| 12-14 | 0.00001 | ~1.1 m | 5% reduction |
| 15-18 | 0 (no simplification) | Original | 0% |

---

## Part 6: Attribute Configuration

### 6.1 Tooltip Display Rules

| Rule | Specification |
|------|---------------|
| **Max Attributes** | 6 atribut per tooltip (exclude internal ID) |
| **Sort Order** | Sesuai `sort_order` di `layer_attribute_definitions` |
| **Hidden Fields** | Field dengan `is_visible_in_tooltip = false` tidak ditampilkan |
| **Formatting** | Number: `toLocaleString('id-ID')`. Area: `{value} km²`. Population: `{value} jiwa`. |
| **Null Handling** | Tampilkan `"-"` untuk nilai NULL |

### 6.2 Coordinate Display Format

| Format | Usage | Example |
|--------|-------|---------|
| Decimal Degrees | Default | `106.845599, -6.208763` |
| Precision | 6 digits | `-6.208763` |
| Separator | `|` | `Lon: 106.845599 \| Lat: -6.208763` |
| Monospace Font | Yes | JetBrains Mono / Fira Code |
| Update Frequency | Real-time (raframe) | ≤ 16ms |

---

## Part 7: Metadata Layer Schema

### 7.1 Default Layer Metadata (Seed Data)

```sql
INSERT INTO layers (name, description, type, geom_type, srid, attribution, default_visibility, z_index) 
VALUES 
    (
        'Batas Administrasi Indonesia', 
        'Layer batas provinsi, kabupaten/kota, kecamatan, dan desa seluruh Indonesia', 
        'administrative_boundary', 
        'MultiPolygon', 
        4326, 
        '© Badan Pusat Statistik / Kemendagri', 
        TRUE, 
        1
    ),
    (
        'Infrastruktur Publik', 
        'Layer infrastruktur publik: Jalan Nasional, Jalur Kereta Api, Bandara, Bendungan', 
        'infrastructure', 
        'LineString', 
        4326, 
        '© OpenStreetMap contributors / Kementerian PUPR', 
        TRUE, 
        2
    ),
    (
        'Peta Dasar Indonesia', 
        'Base map tile dari OpenStreetMap', 
        'base_map', 
        null, 
        null, 
        '© OpenStreetMap contributors', 
        TRUE, 
        0
    );
```

### 7.2 Layer Metadata via API Response

```json
{
  "id": "admin_boundaries",
  "name": "Batas Administrasi Indonesia",
  "description": "Layer batas provinsi, kabupaten/kota, kecamatan, dan desa",
  "type": "administrative_boundary",
  "geom_type": "MultiPolygon",
  "srid": 4326,
  "attribution": "© Badan Pusat Statistik / Kemendagri",
  "default_visibility": true,
  "z_index": 1,
  "attributes": [
    {"field": "name", "type": "varchar", "description": "Nama wilayah", "unit": null, "is_visible_in_tooltip": true, "sort_order": 1},
    {"field": "code", "type": "varchar", "description": "Kode Kemendagri", "unit": null, "is_visible_in_tooltip": true, "sort_order": 2},
    {"field": "level", "type": "varchar", "description": "Level administrasi", "unit": null, "is_visible_in_tooltip": true, "sort_order": 3},
    {"field": "area_km2", "type": "numeric", "description": "Luas wilayah", "unit": "km²", "is_visible_in_tooltip": true, "sort_order": 4},
    {"field": "population", "type": "bigint", "description": "Jumlah penduduk", "unit": "jiwa", "is_visible_in_tooltip": true, "sort_order": 5}
  ]
}
```

---

## Part 8: Implementation Notes

### 8.1 OpenLayers Map Initialization

```typescript
import Map from 'ol/Map';
import View from 'ol/View';
import TileLayer from 'ol/layer/Tile';
import VectorLayer from 'ol/layer/Vector';
import OSM from 'ol/source/OSM';
import VectorSource from 'ol/source/Vector';
import GeoJSON from 'ol/format/GeoJSON';
import { bboxStrategy } from 'ol/loadingstrategy';

const transform = (coord: number[], from: string, to: string) => {
  return transform(coord, from, to);
};

const map = new Map({
  target: 'map-container',
  layers: [
    new TileLayer({
      source: new OSM({
        url: 'https://{a-c}.tile.openstreetmap.org/{z}/{x}/{y}.png',
        attributions: '© OpenStreetMap contributors',
        crossOrigin: 'anonymous',
      }),
      zIndex: 0,
    }),
    // Vector layers added dynamically
  ],
  view: new View({
    center: transform([118.0, -2.5], 'EPSG:4326', 'EPSG:3857'),
    zoom: 4,
    minZoom: 3,
    maxZoom: 18,
    extent: transform([95.3, -10.9, 141.0, 5.9], 'EPSG:4326', 'EPSG:3857'),
    showFullExtent: true,
  }),
  controls: [],
});
```

### 8.2 BBOX-based Vector Loading Strategy

```typescript
const adminBoundarySource = new VectorSource({
  format: new GeoJSON(),
  url: (extent) => {
    const [minx, miny, maxx, maxy] = transform(extent, 'EPSG:3857', 'EPSG:4326');
    return `/api/v1/vector/admin-boundaries?bbox=${minx},${miny},${maxx},${maxy}&zoom_level=${currentZoom}&level=province`;
  },
  strategy: bboxStrategy,
});
```

### 8.3 Style Function dengan Zoom Dependency

```typescript
function getAdminBoundaryStyle(feature: Feature, resolution: number) {
  const level = feature.get('level');
  const zoom = map.getView().getZoom();
  
  let strokeColor: string;
  let strokeWidth: number;
  
  switch (level) {
    case 'province':
      strokeColor = '#2563EB';
      strokeWidth = 2;
      break;
    case 'regency':
      strokeColor = '#3B82F6';
      strokeWidth = 1.5;
      break;
    case 'district':
      strokeColor = '#60A5FA';
      strokeWidth = 1;
      break;
    default:
      strokeColor = '#93C5FD';
      strokeWidth = 0.75;
  }
  
  // Hide village level saat zoom < 12
  if (level === 'village' && zoom < 12) {
    return undefined; // tidak render
  }
  
  return new Style({
    stroke: new Stroke({ color: strokeColor, width: strokeWidth }),
    fill: new Fill({ color: 'transparent' }),
  });
}
```

---

## 9. Appendix

### 9.1 Key Terms

| Term | Definition |
|------|-----------|
| **Tile** | Potongan 256x256px dari peta besar, terdiri dari gambar raster |
| **Vector Layer** | Layer yang menyimpan data spasial sebagai koordinat (bukan gambar) |
| **BBOX** | Bounding box — kotak pembatas yang mendefinisikan area peta yang terlihat |
| **Z-Index** | Urutan lapisan rendering (z-index lebih tinggi = di atas) |
| **SRID** | Spatial Reference System Identifier — kode unik untuk sistem proyeksi |
| **GIST Index** | Generalized Search Tree — index spasial PostgreSQL untuk query geometri cepat |
| **MVT** | Mapbox Vector Tiles — format binary untuk vektor tiles |
| **Hit-test** | Proses mendeteksi apakah mouse mengarah ke feature spasial |

### 9.2 Related Documents

| Document | Location | Purpose |
|----------|----------|---------|
| Product Requirements Document | `docs/prd.md` | Business goals, scope, user stories |
| Domain Model | `docs/domain-model.md` | Entity definitions, bounded contexts |
| ERD | `docs/erd.md` | Database schema, relationships |
| API Specification | `docs/api-spec.md` | Backend endpoint specifications |
| UI Specification | `docs/ui-spec.md` | Component specifications, styling |
| PRD Gap Analysis | `docs/prd-gap-analysis.md` | Identified gaps & recommendations |

---

*Document Owner: Product Management & Frontend Architecture Team*
*Last Review: 2026-06-17*
*Next Review: 2026-07-01*
