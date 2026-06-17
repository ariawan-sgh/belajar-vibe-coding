# UI Specification — WebGIS

**Basis Dokumen:** `docs/prd.md` v1.0, `docs/prd-gap-analysis.md`
**Tanggal:** 2026-06-17
**Platform:** Web Desktop (min. 1280px width)
**Teknologi:** HTML5, Tailwind CSS, Lit JS, OpenLayers, Vite

---

## 1. Design Principles

| Principle | Application |
|-----------|-------------|
| **Function First** | Peta adalah elemen utama (100% viewport). Semua UI overlay mencegah occlusion area peta yang penting. |
| **Information Density** | GIS Officer membutuhkan banyak informasi sekaligus. Kompak tapi tidak clutter. |
| **Immediate Feedback** | Setiap interaksi (hover, klik, toggle) harus memberikan visual feedback dalam ≤ 100ms. |
| **Progressive Disclosure** | Kontrol yang jarang digunakan disembunyikan di bawah dropdown atau secondary panel. |
| **Consistency** | Semua komponen menggunakan Tailwind utility classes yang sama. Warna, spacing, dan typography konsisten. |

---

## 2. Layout Structure

### 2.1 Full-Screen Map Layout

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  [Layer Switcher]                              [Zoom Control]  [Reset View] │
│  ┌──────────────┐                                      ▲                     │
│  │ ☑ Admin       │                                      │                     │
│  │   Boundary    │                                      │                     │
│  │ ☑ Infrastruktur│          ┌─────────────────────┐   │                     │
│  │               │          │                     │   ┴                     │
│  │               │          │                     │   ─                     │
│  │               │          │      MAP AREA       │   ─                     │
│  │               │          │   (OpenLayers)      │   ─                     │
│  │               │          │                     │   ▲                     │
│  │               │          │                     │   │                     │
│  │               │          └─────────────────────┘   │                     │
│  └──────────────┘                                      │                     │
│                                                        ┴                     │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  Lon: 106.845599, Lat: -6.208763  │  Zoom: 10  │  Active: 2 layers │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Z-Index Layering

| Z-Index | Element | Description |
|---------|---------|-------------|
| 0 | Base Map Tiles | OpenLayers tile layer |
| 1 | Vector Layers | Admin boundaries, infrastructure |
| 2 | Vector Overlay | Highlight effects |
| 10 | UI Controls | Layer switcher, zoom control |
| 20 | Tooltip | Floating tooltip (highest priority) |
| 30 | Modal / Toast | Error messages, notifications |

---

## 3. Color System

### 3.1 Primary Palette

| Role | Color | Tailwind Class | Usage |
|------|-------|---------------|-------|
| Primary | `#2563EB` | `blue-600` | Active toggles, links, primary actions |
| Primary Hover | `#1D4ED8` | `blue-700` | Hover states |
| Primary Light | `#DBEAFE` | `blue-100` | Backgrounds, active indicators |

### 3.2 Semantic Colors

| Role | Color | Tailwind Class | Usage |
|------|-------|---------------|-------|
| Success | `#059669` | `emerald-600` | Valid data, success states |
| Warning | `#D97706` | `amber-600` | Caution, deprecated data |
| Error | `#DC2626` | `red-600` | Errors, invalid geometries |
| Info | `#0891B2` | `cyan-600` | Informational, highlight |

### 3.3 Layer Colors

| Layer | Fill | Stroke | Stroke Width |
|-------|------|--------|--------------|
| Administrative Boundary (Province) | `transparent` | `#2563EB` (blue-600) | 2px |
| Administrative Boundary (Kabupaten) | `transparent` | `#3B82F6` (blue-500) | 1.5px |
| Infrastructure - Road | `transparent` | `#F59E0B` (amber-500) | 2px |
| Infrastructure - Railway | `transparent` | `#8B5CF6` (violet-500) | 2px |

### 3.4 Neutral Colors

| Role | Color | Tailwind Class | Usage |
|------|-------|---------------|-------|
| Background Dark | `#1F2937` | `gray-800` | Coordinate bar, panel backgrounds |
| Background Dark Hover | `#111827` | `gray-900` | Hover states on dark bg |
| Text Primary | `#F9FAFB` | `gray-50` | Text on dark backgrounds |
| Text Secondary | `#D1D5DB` | `gray-300` | Secondary text, labels |
| Border | `#374151` | `gray-700` | Panel borders, dividers |
| White | `#FFFFFF` | `white` | Tooltip background, cards |

---

## 4. Typography

### 4.1 Font Stack

```css
font-family: 'Inter', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
```

### 4.2 Type Scale

| Name | Size | Weight | Line Height | Usage |
|------|------|--------|-------------|-------|
| `text-xs` | 12px | 400 | 16px | Timestamps, metadata labels |
| `text-sm` | 14px | 400 | 20px | Body text, tooltip content, coordinate bar |
| `text-base` | 16px | 400 | 24px | Primary text, layer names |
| `text-lg` | 18px | 500 | 28px | Panel headings |
| `text-xl` | 20px | 600 | 28px | Modal titles |
| `text-2xl` | 24px | 700 | 32px | Page title (if any) |

### 4.3 Monospace (for coordinates)

```css
font-family: 'JetBrains Mono', 'Fira Code', 'SF Mono', monospace;
```

---

## 5. Component Specifications

### 5.1 `<webgis-map>` — Root Map Container

**Location:** Full viewport (`w-screen h-screen`)

| Property | Value |
|----------|-------|
| Position | `fixed inset-0` |
| z-index | 0 |
| Background | `#1a1a2e` (dark fallback while tiles load) |

**Behavior:**
- Initialize OpenLayers Map with view `center: [118.0, -2.5]` (transformed to EPSG:3857), `zoom: 4`.
- Emit custom events: `map:pointermove`, `map:click`, `map:moveend`, `map:zoomend`.
- Handle resize observer untuk responsive adjustments.

---

### 5.2 `<webgis-layer-switcher>` — Layer Control Panel

**Location:** Top-right corner, offset 16px dari tepi.

```
┌─────────────────────────────┐
│  Layer                      │
│                             │
│  ┌────────────────────────┐ │
│  │ ☑ Batas Administrasi   │ │
│  │   [badge: 3 level]     │ │
│  ├────────────────────────┤ │
│  │ ☑ Infrastruktur        │ │
│  │   [badge: 4 tipe]      │ │
│  └────────────────────────┘ │
│                             │
│  + Tambah Layer             │
└─────────────────────────────┘
```

**Specifications:**

| Property | Value |
|----------|-------|
| Position | `absolute top-4 right-4` |
| Width | `280px` |
| Background | `bg-gray-800/95 backdrop-blur-sm` |
| Border radius | `rounded-lg` |
| Border | `border border-gray-700` |
| Shadow | `shadow-xl` |
| z-index | 10 |

**Internal Elements:**

| Element | Spec |
|---------|------|
| Header | `text-gray-50 text-lg font-semibold px-4 py-3 border-b border-gray-700` |
| Layer Item | `flex items-center gap-3 px-4 py-3 hover:bg-gray-700/50 transition-colors` |
| Checkbox | Custom styled: `w-4 h-4 rounded border-gray-600 text-blue-600 focus:ring-blue-500` |
| Layer Name | `text-sm font-medium text-gray-200` |
| Layer Badge | `text-xs px-2 py-0.5 rounded-full bg-gray-700 text-gray-400` |
| Add Button | `text-sm text-blue-400 hover:text-blue-300 px-4 py-2 border-t border-gray-700` |

**States:**
- **Default:** Header dengan background gelap.
- **Hover (item):** `bg-gray-700/50`.
- **Disabled:** `opacity-50 pointer-events-none` (jika layer unavailable).
- **Loading:** Tampilkan skeleton shimmer pada nama layer.

---

### 5.3 `<webgis-zoom-control>` — Zoom & Reset Buttons

**Location:** Bottom-right corner, di atas coordinate bar.

```
         ▲
         │
     ────┼───
         │
         ▼
```

**Specifications:**

| Property | Value |
|----------|-------|
| Position | `absolute bottom-24 right-4` |
| Direction | `flex flex-col gap-1` |
| Container | `bg-gray-800/95 backdrop-blur-sm rounded-lg border border-gray-700 shadow-xl overflow-hidden` |
| z-index | 10 |

**Button Specs:**

| Button | Icon | Tooltip | Size |
|--------|------|---------|------|
| Zoom In | `+` | "Zoom In" (Ctrl + Scroll) | `w-10 h-10` |
| Zoom Out | `−` | "Zoom Out" (Ctrl + Scroll) | `w-10 h-10` |
| Reset View | [Indonesia icon / home] | "Reset View ke Indonesia" | `w-10 h-10` |

**Button States:**
- **Default:** `bg-gray-800 text-gray-300 hover:bg-gray-700 hover:text-white`
- **Active/Pressed:** `bg-gray-700 text-white`
- **Disabled (min/max zoom):** `opacity-40 cursor-not-allowed`
- **Focus:** `ring-2 ring-blue-500 ring-offset-2 ring-offset-gray-800`

**Icon Style:** Menggunakan SVG inline atau Lucide icons, `w-5 h-5`, stroke width 2.

---

### 5.4 `<webgis-tooltip>` — Feature Tooltip

**Location:** Floating, mengikuti posisi mouse dengan offset 12px.

```
┌──────────────────────────────┐
│  DKI Jakarta                 │  ← Header (name, bold, larger)
│  ─────────────────────────── │
│  Kode     : 31               │  ← Key-value pairs
│  Level    : Province         │
│  Luas     : 664 km²          │
│  Penduduk : 10.562.088       │
│                              │
│  ┌────────────────────────┐  │
│  │ Copy                    │  │ ← Action footer
│  └────────────────────────┘  │
└──────────────────────────────┘
```

**Specifications:**

| Property | Value |
|----------|-------|
| Position | `fixed` (di-set via JS berdasarkan mouse position) |
| Max Width | `320px` |
| Background | `bg-white` |
| Border radius | `rounded-lg` |
| Shadow | `shadow-2xl` |
| Border | `border border-gray-200` |
| z-index | 20 |
| Font | Inter, 14px |

**Internal Elements:**

| Element | Spec |
|---------|------|
| Header (Feature Name) | `text-gray-900 font-semibold text-base px-4 pt-3 pb-2` |
| Divider | `h-px bg-gray-200 mx-4` |
| Attribute Row | `flex justify-between px-4 py-1.5 text-sm` |
| Key | `text-gray-500` |
| Value | `text-gray-900 font-medium` |
| Action Footer | `border-t border-gray-100 px-4 py-2 mt-1` |
| Copy Button | `text-xs text-blue-600 hover:text-blue-800 flex items-center gap-1` |

**Positioning Logic:**
- Default: offset `(mouseX + 12px, mouseY + 12px)`.
- Jika `tooltipRight > viewportWidth`: flip ke kiri (`mouseX - tooltipWidth - 12px`).
- Jika `tooltipBottom > viewportHeight`: flip ke atas (`mouseY - tooltipHeight - 12px`).
- Transition: `transition-all duration-75 ease-out` (smooth follow).

**States:**
- **Visible:** `opacity-100 pointer-events-auto`.
- **Hidden:** `opacity-0 pointer-events-none` (tidak di-remove dari DOM untuk performa).
- **Loading:** Skeleton shimmer (opsional, untuk tooltip yang fetch data tambahan).

---

### 5.5 `<webgis-coordinate-bar>` — Bottom Status Bar

**Location:** Bottom of viewport, full width.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 🖱 Lon: 106.845599 | Lat: -6.208763  │  Zoom: 10/18  │  2 layers active   │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Specifications:**

| Property | Value |
|----------|-------|
| Position | `fixed bottom-0 left-0 right-0` |
| Height | `32px` (h-8) |
| Background | `bg-gray-800/95 backdrop-blur-sm` |
| Border top | `border-t border-gray-700` |
| z-index | 10 |

**Internal Layout:**

| Element | Spec |
|---------|------|
| Container | `flex items-center justify-between px-4 h-full` |
| Coordinate Group | `flex items-center gap-4` |
| Coordinate Label | `text-xs text-gray-400 uppercase tracking-wider` |
| Coordinate Value | `text-xs font-mono text-gray-200` (JetBrains Mono) |
| Separator | `w-px h-4 bg-gray-600` |
| Zoom Level | `text-xs text-gray-400` |
| Active Layers Count | `text-xs text-gray-400` |

**Content:**
- **Left section:** `Lon: {value}` | `Lat: {value}` — update real-time pada `pointermove`.
- **Center section (opsional):** Nama layer yang sedang di-hover.
- **Right section:** `Zoom: {level}/18` | `{count} layers active`.

---

## 6. Interaction Patterns

### 6.1 Hover Interaction (Tooltip + Highlight)

```
┌─────────────┐
│             │
│   MAP       │
│             │
│    ┌─────┐  │
│    │ Hover│←── Mouse enters feature
│    │Area  │
│    └─────┘  │
│      │      │
│      ▼      │
│  ┌────────┐ │
│  │TOOLTIP │ │ ← Appears within 50ms
│  └────────┘ │
└─────────────┘
```

**Sequence:**

| Step | Event | Action | Timing |
|------|-------|--------|--------|
| 1 | `pointermove` | Debounce 16ms via `requestAnimationFrame` | ≤ 16ms |
| 2 | Hit test | `map.forEachFeatureAtPixel(pixel, hitTest, { hitTolerance: 5 })` | ≤ 5ms |
| 3 | Hover start | Apply highlight style, position tooltip, show tooltip | ≤ 50ms total |
| 4 | `pointermove` (still hovering) | Update tooltip position only (no re-render) | ≤ 10ms |
| 5 | `pointerleave` feature | Remove highlight, hide tooltip | ≤ 100ms |

**Highlight Style:**

| Geometry Type | Default Style | Highlight Style |
|--------------|--------------|-----------------|
| Polygon (Admin Boundary) | `fill: transparent`, `stroke: #2563EB`, `width: 2px` | `fill: rgba(37, 99, 235, 0.15)`, `stroke: #00FFFF`, `width: 3px` |
| Line (Infrastructure) | `stroke: #F59E0B`, `width: 2px` | `stroke: #00FFFF`, `width: 4px` |

**Tooltip Transition:**
- Show: `opacity-0 → opacity-100` dalam 75ms.
- Move: `transition-all duration-75 ease-out`.
- Hide: `opacity-100 → opacity-0` dalam 75ms, lalu `pointer-events: none`.

---

### 6.2 Layer Toggle Interaction

```
[ ☑ Batas Administrasi ]    →    [ ☐ Batas Administrasi ]
         (checked)                      (unchecked)
         Layer visible                  Layer hidden
```

**Behavior:**
1. User klik checkbox.
2. Checkbox animasi: `transition-all duration-150`.
3. Layer visibility berubah: `layer.setVisible(boolean)`.
4. Vector features hilang/muncul dengan fade transition (opsional, menggunakan OpenLayers `postrender`).
5. Layer switcher update badge count.

**Accessibility:**
- Checkbox memiliki `aria-label` dan `role="switch"`.
- Status di-announce via `aria-live="polite"`.

---

### 6.3 Navigation Interactions

| Interaction | Input | Result |
|------------|-------|--------|
| Zoom In | Scroll up / Klik `+` | `view.setZoom(current + 1)` |
| Zoom Out | Scroll down / Klik `−` | `view.setZoom(current - 1)` |
| Pan | Klik + drag | `view.setCenter()` dengan momentum |
| Double Click Zoom | Double click pada peta | `view.setZoom(current + 1)` pada titik klik |
| Reset View | Klik tombol reset | `view.fit(extent_indo, { padding: [50,50,50,50] })` |

**Zoom Limits:** Min `3`, Max `18`.
**Momentum:** OpenLayers built-in kinetic panning (durasi 500ms, decay rate 0.95).

---

## 7. States

### 7.1 Loading States

| Scenario | UI Treatment |
|----------|-------------|
| Initial Page Load | Full-screen dark backdrop (`#1a1a2e`) dengan spinner centered. Spinner: `w-12 h-12 border-4 border-blue-600 border-t-transparent rounded-full animate-spin`. |
| Tile Loading | OSM tile loading di-handle oleh OpenLayers. Jika tile gagal, tampilkan placeholder gray tile. |
| Vector Layer Loading | Layer switcher menampilkan skeleton shimmer pada nama layer. Badge menampilkan "Loading..." |
| Tooltip Loading (opsional) | Tampilkan pulse animation pada teks atribut selama data di-fetch. |

### 7.2 Empty States

| Scenario | UI Treatment |
|----------|-------------|
| No features in viewport | Tooltip tidak muncul. Coordinate bar menampilkan: "Tidak ada objek di area ini". |
| All layers disabled | Base map tetap terlihat. Layer switcher menampilkan empty state: "Aktifkan layer untuk melihat data". |
| API returns empty GeoJSON | Map menampilkan base map saja. Toast notification: "Tidak ada data spasial untuk area ini." |

### 7.3 Error States

| Scenario | UI Treatment |
|----------|-------------|
| API Error (500, 404, timeout) | Toast notification merah di pojok atas: `"Gagal memuat data: [error code]. Coba lagi."` dengan tombol "Retry". |
| Invalid Geometry | Tooltip menampilkan: `"Data tidak valid. Hubungi administrator."` dalam warna merah. |
| Tile Provider Down | Fallback tile (jika dikonfigurasi) atau gray placeholder. Toast: "Base map tidak tersedia. Menggunakan fallback." |
| CORS Error | Full-page error: "Aplikasi tidak dapat terhubung ke server. Periksa koneksi jaringan." |

**Toast Component Spec:**
- Position: `fixed top-4 right-4`.
- Animation: `translate-x-0 → translate-x-full` untuk enter/exit.
- Auto-dismiss: 5 detik.
- Manual dismiss: Klik X atau klik di luar.

---

## 8. Spacing & Sizing

### 8.1 Spacing Scale (Tailwind)

| Token | Value | Usage |
|-------|-------|-------|
| `p-1` | 4px | Icon padding, tight spacing |
| `p-2` | 8px | Button padding, compact items |
| `p-3` | 12px | Default component padding |
| `p-4` | 16px | Panel padding, section spacing |
| `gap-1` | 4px | Icon-text gap |
| `gap-2` | 8px | List item gap |
| `gap-3` | 12px | Form field gap |
| `gap-4` | 16px | Section gap |

### 8.2 Component Sizing

| Component | Width | Height | Min/Max |
|-----------|-------|--------|---------|
| Layer Switcher | 280px | auto | max-h-[80vh] dengan scroll |
| Zoom Control | 40px | auto | — |
| Tooltip | 280px | auto | max-h-[60vh], max-w-[320px] |
| Coordinate Bar | 100% | 32px | — |
| Checkbox | 16px | 16px | — |
| Button (kecil) | 32px | 32px | Zoom buttons |
| Icon | 20px | 20px | Standard icon size |

---

## 9. Motion & Animation

### 9.1 Transitions

| Element | Property | Duration | Easing |
|---------|----------|----------|--------|
| Tooltip | opacity, transform | 75ms | ease-out |
| Layer Switcher | opacity (show/hide) | 150ms | ease-in-out |
| Button | background-color, color | 150ms | ease-in-out |
| Checkbox | all | 150ms | ease-in-out |
| Toast | transform (slide) | 300ms | ease-out |

### 9.2 Motion Principles

- **Respect `prefers-reduced-motion`:** Jika user mengaktifkan reduced motion di OS, semua transition harus di-disable atau dikurangi durasinya.
- **No auto-playing animations:** Tidak ada animasi yang berjalan terus-menerus tanpa interaksi user.
- **Motion dengan purpose:** Hanya gunakan motion untuk: show/hide, state change, dan spatial transition (tooltip follow).

---

## 10. Icons

### 10.1 Icon Library

Menggunakan **Lucide React/JS** (atau equivalente vanilla SVG). Semua icon dalam stroke width 2, size 20px default.

### 10.2 Icon Mapping

| Icon Name | Usage | Component |
|-----------|-------|-----------|
| `Plus` | Tambah layer button | Layer Switcher |
| `Check` / `Square` | Checkbox kontrol | Layer Switcher |
| `ZoomIn` | Zoom in button | Zoom Control |
| `ZoomOut` | Zoom out button | Zoom Control |
| `Home` / `Maximize` | Reset view button | Zoom Control |
| `MapPin` | Coordinate label prefix | Coordinate Bar |
| `Layers` | Layer switcher header icon | Layer Switcher |
| `Copy` | Copy attribute action | Tooltip |
| `CheckCircle` | Valid geometry indicator | Tooltip |
| `AlertCircle` | Error indicator | Error State |
| `Loader` | Loading spinner | Loading State |
| `X` | Close / dismiss | Toast, Tooltip |
| `Info` | Attribution info | Attribution Control |

---

## 11. Accessibility (A11Y)

### 11.1 WCAG 2.1 Compliance

| Requirement | Implementation |
|-------------|---------------|
| Keyboard Navigation | Semua kontrol dapat diakses via Tab. Focus indicators menggunakan `focus:ring-2 focus:ring-blue-500`. |
| Screen Reader | Komponen menggunakan semantic HTML: `role="tooltip"`, `aria-label`, `aria-live`. |
| Color Contrast | Semua teks memiliki contrast ratio ≥ 4.5:1 terhadap background (WCAG AA). |
| Focus Management | Tooltip tidak mencuri focus kecuali dengan klik. Keyboard users bisa menutup tooltip dengan Escape. |
| Reduced Motion | `@media (prefers-reduced-motion: reduce)` menonaktifkan transisi. |

### 11.2 Keyboard Shortcuts (Sprint 1 Minimal)

| Shortcut | Action |
|----------|--------|
| `+` / `=` | Zoom in |
| `-` | Zoom out |
| `0` (zero) | Reset view ke Indonesia |
| `Esc` | Tutup tooltip (jika terbuka) |
| `Tab` | Navigasi antar kontrol |

---

## 12. Responsive & Breakpoints

### 12.1 Breakpoint Strategy

Sprint 1 adalah **desktop-first**. Viewport minimum: **1280px**.

| Breakpoint | Range | Behavior |
|-----------|-------|---------|
| `xl` | ≥ 1280px | Full layout (default). |
| `lg` | 1024px – 1279px | Layer switcher collapse ke icon-only mode. Tooltip max-width: 260px. |
| `< lg` | < 1024px | TIDAK DIDUKUNG di Sprint 1. Tampilkan banner: "Aplikasi ini dioptimalkan untuk desktop 1280px+." |

### 12.2 Responsive Adjustments (Desktop Only)

| Condition | Adjustment |
|-----------|-----------|
| Viewport ≤ 1366px | Layer switcher width: `256px` (dari 280px). |
| Viewport ≤ 1280px | Tooltip max-width: `260px`. Coordinate bar font size: `text-xs`. |
| High DPI (Retina) | Tile resolution: `@2x` tiles jika tersedia dari provider. OpenLayers `tilePixelRatio` di-set ke `window.devicePixelRatio`. |

---

## 13. Assets & Resources

### 13.1 Base Map Tiles

| Item | Specification |
|------|--------------|
| Provider | OpenStreetMap (default) |
| URL Template | `https://{a-c}.tile.openstreetmap.org/{z}/{x}/{y}.png` |
| Attribution | `© OpenStreetMap contributors` |
| Max Zoom | 19 (tiles tersedia sampai zoom 18) |
| Fallback | `https://tile.openstreetmap.de/{z}/{x}/{y}.png` |

### 13.2 Logo & Branding

| Asset | Spec |
|-------|------|
| App Name | **WebGIS** — ditampilkan di browser tab title. |
| Favicon | `favicon.ico` di folder `public/`. |
| Splash Screen (opsional) | SVG logo dengan background `#1a1a2e` selama initial load (max 2 detik). |

---

## 14. Tailwind Configuration

### 14.1 Custom Colors (tailwind.config.js)

```javascript
colors: {
  'map': {
    'ocean': '#aadaff',
    'land': '#f2efe9',
    'border-admin': '#2563EB',
    'highlight': '#00FFFF',
    'road': '#F59E0B',
    'railway': '#8B5CF6',
  },
  'panel': {
    'bg': 'rgba(31, 41, 55, 0.95)',
    'border': '#374151',
    'hover': '#374151',
  }
}
```

### 14.2 Custom Font Family

```javascript
fontFamily: {
  'sans': ['Inter', 'system-ui', 'sans-serif'],
  'mono': ['JetBrains Mono', 'Fira Code', 'monospace'],
}
```

### 14.3 Custom Animations

```javascript
extend: {
  animation: {
    'pulse-slow': 'pulse 3s cubic-bezier(0.4, 0, 0.6, 1) infinite',
    'shimmer': 'shimmer 2s linear infinite',
  },
  keyframes: {
    shimmer: {
      '0%': { backgroundPosition: '-200px 0' },
      '100%': { backgroundPosition: 'calc(200px + 100%) 0' },
    }
  }
}
```

---

## 15. Component File Structure (Updated)

```
src/
├── index.html
├── src/
│   ├── main.ts
│   ├── components/
│   │   ├── WebGISMap.ts
│   │   ├── LayerSwitcher.ts
│   │   ├── Tooltip.ts
│   │   ├── CoordinateBar.ts
│   │   ├── ZoomControl.ts
│   │   ├── Toast.ts              # NEW: Error/notification toast
│   │   ├── EmptyState.ts         # NEW: Empty state message
│   │   └── LoadingOverlay.ts     # NEW: Loading indicator
│   ├── layers/
│   │   ├── BaseMapLayer.ts
│   │   ├── AdminBoundaryLayer.ts
│   │   └── InfrastructureLayer.ts
│   ├── styles/
│   │   ├── main.css              # Tailwind directives + custom CSS
│   │   └── map-styles.css        # OpenLayers style overrides
│   ├── utils/
│   │   ├── map-utils.ts
│   │   └── api.ts
│   └── types/
│       ├── layer.ts
│       ├── feature.ts
│       └── api.ts
├── tailwind.config.js
├── postcss.config.js
├── tsconfig.json
└── package.json
```

---

## 16. Implementation Notes

### 16.1 Lit JS Styling Strategy

- Setiap komponen Lit menggunakan `static styles = css` dengan Tailwind utility classes.
- Untuk styling dinamis (highlight, tooltip position), gunakan CSS custom properties:

```css
:root {
  --highlight-stroke: #00FFFF;
  --highlight-fill: rgba(0, 255, 255, 0.15);
  --tooltip-offset: 12px;
}
```

### 16.2 OpenLayers Style Integration

```typescript
// Highlight style untuk polygon
const highlightStyle = new Style({
  stroke: new Stroke({ color: '#00FFFF', width: 3 }),
  fill: new Fill({ color: 'rgba(0, 255, 255, 0.15)' }),
});

// Default style untuk admin boundary
const adminStyle = (level: string) => {
  const width = level === 'province' ? 2 : 1.5;
  return new Style({
    stroke: new Stroke({ color: '#2563EB', width }),
    fill: new Fill({ color: 'transparent' }),
  });
};
```

### 16.3 Tooltip Positioning Algorithm

```typescript
function positionTooltip(
  mouseX: number, 
  mouseY: number, 
  tooltipWidth: number, 
  tooltipHeight: number,
  viewportWidth: number,
  viewportHeight: number
): { x: number; y: number } {
  const offset = 12;
  let x = mouseX + offset;
  let y = mouseY + offset;

  if (x + tooltipWidth > viewportWidth - 16) {
    x = mouseX - tooltipWidth - offset;
  }
  if (y + tooltipHeight > viewportHeight - 48) {
    y = mouseY - tooltipHeight - offset;
  }

  return { x: Math.max(8, x), y: Math.max(8, y) };
}
```

### 16.4 Performance: Avoid Layout Thrashing

- Tooltip positioning menggunakan `getBoundingClientRect()` sekali per frame (via `requestAnimationFrame`).
- Jangan baca `offsetWidth/offsetHeight` di dalam event handler yang sering di-trigger (seperti `pointermove`). Cache ukuran tooltip saat pertama render.
- Gunakan `will-change: transform, opacity` pada tooltip untuk GPU acceleration.

---

## 17. Design Tokens Summary

```
=== SPACING ===
panel-padding: 16px (p-4)
panel-gap: 8px (gap-2)
control-offset: 16px (dari tepi viewport)

=== TYPOGRAPHY ===
font-family: Inter
font-mono: JetBrains Mono
base-size: 16px (text-base)
small-size: 14px (text-sm)
tiny-size: 12px (text-xs)

=== COLORS ===
primary: #2563EB (blue-600)
highlight: #00FFFF (cyan-400)
panel-bg: rgba(31, 41, 55, 0.95)
panel-border: #374151 (gray-700)
tooltip-bg: #FFFFFF
text-on-dark: #F9FAFB (gray-50)
text-muted: #9CA3AF (gray-400)

=== SHADOWS ===
panel-shadow: shadow-xl (0 20px 25px -5px rgba(0,0,0,0.3))
tooltip-shadow: shadow-2xl (0 25px 50px -12px rgba(0,0,0,0.4))

=== BORDERS ===
panel-radius: rounded-lg (8px)
tooltip-radius: rounded-lg (8px)
button-radius: rounded-md (6px)

=== Z-INDEX ===
base-map: 0
vector-layer: 1
vector-highlight: 2
ui-controls: 10
tooltip: 20
modal-toast: 30
```

---

## 18. Mockup Wireframes (ASCII)

### 18.1 Full Screen Layout

```
╔══════════════════════════════════════════════════════════════════════════════╗
║  ┌─────────────┐                                          [▲][─][▼][⌂]    ║
║  │ ☑ Batas     │                                                            ║
║  │   Admin     │                                                            ║
║  │ ☑ Infrastruktur│                                                         ║
║  │             │                                                            ║
║  └─────────────┘                                                            ║
║                                                                              ║
║                                                                              ║
║                                                                              ║
║                            [  MAP AREA  ]                                   ║
║                                                                              ║
║                                                                              ║
║                                                                              ║
║                                                                              ║
║                                                                              ║
║                                                              [Zoom Controls] ║
║                                                                              ║
╠══════════════════════════════════════════════════════════════════════════════╣
║  📍 Lon: 106.845599 │ Lat: -6.208763  │  Zoom: 10/18  │  2 layers active   ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

### 18.2 Tooltip Close-Up

```
                     ┌──────────────────────┐
                     │ DKI Jakarta      [×] │
                     │──────────────────────│
                     │ Kode       : 31      │
                     │ Level      : Province│
                     │ Luas       : 664 km² │
                     │ Penduduk   : 10.5 Jt │
                     │──────────────────────│
                     │ [📋 Copy Attributes]  │
                     └──────────────────────┘
                           ▲
                           │ mouse position
```

### 18.3 Layer Switcher Close-Up

```
┌─────────────────────────┐
│ 🗂 Layer                │
│─────────────────────────│
│ ☑ Batas Administrasi    │
│   Province, Kabupaten   │
│                         │
│ ☑ Infrastruktur         │
│   Jalan, Kereta Api     │
│                         │
│─────────────────────────│
│  + Tambah Layer Baru    │
└─────────────────────────┘
```

---

## 19. Catatan untuk Development Team

1. **Tailwind JIT:** Pastikan `content` di `tailwind.config.js` mencakup semua file `.ts` di `src/` agar utility classes ter-generate.
2. **OpenLayers CSS:** Include `ol.css` dan `ol-layerswitcher.css` (jika digunakan) di `index.html` atau import di `main.ts`.
3. **Lit JS Scoped Styles:** Setiap komponen Lit menggunakan `static styles = css` — Jangan gunakan global CSS untuk styling komponen.
4. **Tooltip Performance:** Jangan render tooltip untuk setiap `pointermove`. Gunakan `requestAnimationFrame` dan debounce 16ms.
5. **Coordinate Precision:** Format koordinat dengan `toFixed(6)` untuk Lon/Lat. Gunakan `Intl.NumberFormat` untuk format angka besar (populasi).
6. **Dark Mode:** Default menggunakan dark theme. Light mode dijadwalkan di Q3 (Roadmap).
7. **Font Loading:** Preconnect ke Google Fonts (Inter, JetBrains Mono) di `index.html` head.

---

*Document Owner: Product Management & UX/UI Design Team*
*Last Review: 2026-06-17*
*Next Review: 2026-07-01*
*Related Documents:* `docs/prd.md`, `docs/prd-gap-analysis.md`, `docs/domain-model.md`, `docs/erd.md`
