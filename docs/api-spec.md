# API Specification — WebGIS

**Basis Dokumen:** `docs/prd.md` v1.0, `docs/srs.md`, `docs/domain-model.md`
**Versi API:** `v1`
**Base URL:** `https://api.webgis.example.com`
**Tanggal:** 2026-06-17
**Status:** Approved for Implementation
**Protocol:** HTTPS (TLS 1.2+)
**Format:** JSON (GeoJSON FeatureCollection) atau Protocol Buffers (MVT)

---

## 1. API Overview

### 1.1 Purpose

API ini menyediakan akses terstruktur ke data spasial Indonesia untuk aplikasi WebGIS. Dirancang untuk performa tinggi dengan query PostGIS teroptimasi, indeks spasial GIST, dan respons sesuai standar OGC.

### 1.2 Design Principles

| Principle | Implementation |
|-----------|---------------|
| **RESTful** | Semua endpoint menggunakan HTTP methods standar (GET). |
| **Stateless** | Setiap request mandiri. Session state tidak disimpan di server. |
| **Content Negotiation** | Client dapat meminta format GeoJSON atau MVT via header `Accept`. |
| **Spatial Query Optimization** | Semua query spasial menggunakan `ST_Intersects` dengan bbox parameter. |
| **Consistent Response** | Format error dan success response konsisten across semua endpoint. |

### 1.3 Base URL & Versioning

```
Production:  https://api.webgis.example.com
Staging:     https://api-staging.webgis.example.com
Local:       http://localhost:3000

API Version: /api/v1/
```

### 1.4 Authentication & Authorization

**Sprint 1:** Public API (tidak memerlukan autentikasi).
**Sprint 2+:** JWT Bearer token untuk protected endpoints.
```
Authorization: Bearer <jwt_token>
```

### 1.5 CORS Policy

| Origin | Methods | Headers |
|--------|---------|---------|
| `https://webgis.example.com` | GET, OPTIONS | `Content-Type`, `Accept` |
| `https://staging.webgis.example.com` | GET, OPTIONS | `Content-Type`, `Accept` |

---

## 2. Common Headers

### 2.1 Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Accept` | No | `application/json` (default) atau `application/x-protobuf` untuk MVT. |
| `Content-Type` | No | Default: `application/json`. Digunakan untuk POST/PUT. |
| `X-Request-ID` | No | UUID untuk request tracing. |
| `Accept-Language` | No | `id-ID` (default) atau `en-US`. |
| `User-Agent` | No | Client info untuk logging. |

### 2.2 Response Headers

| Header | Description |
|--------|-------------|
| `Content-Type` | Sesuai format response (`application/json` atau `application/x-protobuf`). |
| `X-Request-ID` | Echo dari request atau generated UUID. |
| `X-Response-Time` | Waktu processing dalam ms (untuk debugging). |
| `Cache-Control` | `public, max-age=300, s-maxage=600` untuk vector data. |
| `Access-Control-Allow-Origin` | Sesuai CORS policy. |

---

## 3. Error Handling

### 3.1 Error Response Format

Semua error response mengikuti struktur berikut:

```json
{
  "error": {
    "code": "INVALID_BBOX",
    "message": "Bounding box format invalid.",
    "details": {
      "param": "bbox",
      "received": "invalid_format",
      "expected": "minx,miny,maxx,maxy"
    },
    "timestamp": "2026-06-17T09:03:08+07:00",
    "request_id": "abc-123-def"
  }
}
```

### 3.2 Error Codes Reference

| HTTP Status | Error Code | Description |
|-------------|------------|-------------|
| 400 | `INVALID_BBOX` | Format bbox tidak valid atau koordinat di luar range. |
| 400 | `INVALID_ZOOM` | Zoom level di luar range 3-18. |
| 400 | `INVALID_LEVEL` | Level tidak dikenali. |
| 400 | `INVALID_GEOMETRY` | Geometri request tidak valid. |
| 404 | `LAYER_NOT_FOUND` | Layer ID tidak ditemukan. |
| 422 | `GEOMETRY_INVALID` | Data spasial di database rusak (`ST_IsValid` gagal). |
| 429 | `RATE_LIMIT_EXCEEDED` | Terlalu banyak request. |
| 500 | `DATABASE_ERROR` | Koneksi database atau query PostGIS gagal. |
| 500 | `INTERNAL_ERROR` | Kesalahan server yang tidak diharapkan. |
| 503 | `SERVICE_UNAVAILABLE` | Server dalam maintenance mode. |

### 3.3 Error Response Examples

**400 Bad Request:**
```json
{
  "error": {
    "code": "INVALID_BBOX",
    "message": "Bounding box format invalid. Expected: minx,miny,maxx,maxy",
    "details": {
      "param": "bbox",
      "received": "106.8,-6.2",
      "expected": "4 numeric values separated by comma"
    },
    "timestamp": "2026-06-17T09:03:08+07:00",
    "request_id": "abc-123-def"
  }
}
```

**404 Not Found:**
```json
{
  "error": {
    "code": "LAYER_NOT_FOUND",
    "message": "Layer with ID 'infra' not found.",
    "details": {
      "layer_id": "infra",
      "available_layers": ["admin_boundaries", "infrastructure"]
    },
    "timestamp": "2026-06-17T09:03:08+07:00",
    "request_id": "abc-123-def"
  }
}
```

**429 Rate Limit:**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please retry after 60 seconds.",
    "details": {
      "limit": 100,
      "window": "1 minute",
      "retry_after": 60
    },
    "timestamp": "2026-06-17T09:03:08+07:00",
    "request_id": "abc-123-def"
  }
}
```

---

## 4. Endpoint: Vector — Administrative Boundaries

**API-ID:** `API-VEC-01`
**Method:** `GET`
**Endpoint:** `/api/v1/vector/admin-boundaries`
**Authentication:** Public
**Rate Limit:** 100 req/min per IP

### 4.1 Request Parameters

| Parameter | Type | Required | Description | Example |
|-----------|------|----------|-------------|---------|
| `bbox` | string | Yes | Bounding box `minx,miny,maxx,maxy` (EPSG:4326). | `95.3,-10.9,141.0,5.9` |
| `zoom_level` | integer | Yes | Zoom level saat ini (1-18). Digunakan untuk generalization. | `10` |
| `level` | string | No | Filter level administrasi. Enum: `province`, `regency`, `district`, `village`. | `province` |

### 4.2 Request Examples

**Basic:**
```
GET /api/v1/vector/admin-boundaries?bbox=95.3,-10.9,141.0,5.9&zoom_level=4
```

**With Level Filter:**
```
GET /api/v1/vector/admin-boundaries?bbox=106.7,-6.3,107.0,-6.1&zoom_level=12&level=province
```

### 4.3 Response Format (Success: 200)

**Content-Type:** `application/json`

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "id": "admin_boundary.1",
      "geometry": {
        "type": "MultiPolygon",
        "coordinates": [
          [
            [
              [106.789, -6.208],
              [106.791, -6.208],
              [106.791, -6.206],
              [106.789, -6.206],
              [106.789, -6.208]
            ]
          ]
        ]
      },
      "properties": {
        "id": 1,
        "layer_id": 1,
        "level": "province",
        "code": "31",
        "name": "DKI Jakarta",
        "province_name": "DKI Jakarta",
        "regency_name": null,
        "district_name": null,
        "population": 10562088,
        "area_km2": 664.01
      }
    }
  ],
  "metadata": {
    "zoom_level": 4,
    "feature_count": 34,
    "bbox": [95.3, -10.9, 141.0, 5.9],
    "crs": "EPSG:4326",
    "response_time_ms": 45
  }
}
```

### 4.4 Response Format (MVT)

**Content-Type:** `application/x-protobuf`

Request dengan header:
```
Accept: application/x-protobuf
```

Response adalah biner MVT (Mapbox Vector Tiles) yang berisi tiles untuk bbox yang diminta.

---

## 5. Endpoint: Vector — Infrastructure

**API-ID:** `API-VEC-02`
**Method:** `GET`
**Endpoint:** `/api/v1/vector/infrastructure`
**Authentication:** Public
**Rate Limit:** 100 req/min per IP

### 5.1 Request Parameters

| Parameter | Type | Required | Description | Example |
|-----------|------|----------|-------------|---------|
| `bbox` | string | Yes | Bounding box `minx,miny,maxx,maxy` (EPSG:4326). | `106.7,-6.3,107.0,-6.1` |
| `zoom_level` | integer | Yes | Zoom level (1-18). | `12` |
| `type` | string | No | Filter tipe infrastruktur. Enum: `national_road`, `railway`, `airport`, `dam`. | `national_road` |

### 5.2 Request Examples

**All Infrastructure:**
```
GET /api/v1/vector/infrastructure?bbox=106.7,-6.3,107.0,-6.1&zoom_level=12
```

**Filtered by Type:**
```
GET /api/v1/vector/infrastructure?bbox=106.7,-6.3,107.0,-6.1&zoom_level=12&type=railway
```

### 5.3 Response Format (Success: 200)

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "id": "infrastructure.1",
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [106.789, -6.208],
          [106.820, -6.195],
          [106.850, -6.180]
        ]
      },
      "properties": {
        "id": 1,
        "layer_id": 2,
        "type": "national_road",
        "name": "Jalan Tol Jakarta-Cikampek",
        "code": "JT001",
        "condition": "baik",
        "length_km": 55.2
      }
    }
  ],
  "metadata": {
    "zoom_level": 12,
    "feature_count": 156,
    "bbox": [106.7, -6.3, 107.0, -6.1],
    "crs": "EPSG:4326",
    "response_time_ms": 38
  }
}
```

---

## 6. Endpoint: Metadata — Layers

**API-ID:** `API-META-01`
**Method:** `GET`
**Endpoint:** `/api/v1/metadata/layers`
**Authentication:** Public
**Rate Limit:** 30 req/min per IP

### 6.1 Request Parameters

None.

### 6.2 Response Format (Success: 200)

```json
[
  {
    "id": "admin_boundaries",
    "name": "Batas Administrasi Indonesia",
    "description": "Layer batas provinsi, kabupaten/kota, kecamatan, dan desa seluruh Indonesia",
    "type": "administrative_boundary",
    "geom_type": "MultiPolygon",
    "srid": 4326,
    "attribution": "© Badan Pusat Statistik / Kemendagri",
    "default_visibility": true,
    "z_index": 1,
    "attributes": [
      {
        "field": "name",
        "type": "varchar",
        "description": "Nama wilayah",
        "unit": null
      },
      {
        "field": "code",
        "type": "varchar",
        "description": "Kode Kemendagri",
        "unit": null
      },
      {
        "field": "area_km2",
        "type": "numeric",
        "description": "Luas wilayah",
        "unit": "km²"
      },
      {
        "field": "population",
        "type": "bigint",
        "description": "Jumlah penduduk",
        "unit": "jiwa"
      }
    ]
  },
  {
    "id": "infrastructure",
    "name": "Infrastruktur Publik",
    "description": "Layer infrastruktur publik: Jalan Nasional, Jalur Kereta Api, Bandara, Bendungan",
    "type": "infrastructure",
    "geom_type": "LineString",
    "srid": 4326,
    "attribution": "© OpenStreetMap contributors / Kementerian PUPR",
    "default_visibility": true,
    "z_index": 2,
    "attributes": [
      {
        "field": "name",
        "type": "varchar",
        "description": "Nama fasilitas",
        "unit": null
      },
      {
        "field": "type",
        "type": "varchar",
        "description": "Jenis infrastruktur",
        "unit": null
      },
      {
        "field": "condition",
        "type": "varchar",
        "description": "Kondisi",
        "unit": null
      },
      {
        "field": "length_km",
        "type": "numeric",
        "description": "Panjang fasilitas",
        "unit": "km"
      }
    ]
  }
]
```

---

## 7. Endpoint: Metadata — Layer Attributes Detail

**API-ID:** `API-META-02`
**Method:** `GET`
**Endpoint:** `/api/v1/metadata/attributes/:layerId`
**Authentication:** Public
**Rate Limit:** 30 req/min per IP

### 7.1 Request Parameters

| Parameter | Type | Required | Description | Example |
|-----------|------|----------|-------------|---------|
| `layerId` | path | Yes | ID layer (string). | `admin_boundaries` |

### 7.2 Request Example

```
GET /api/v1/metadata/attributes/admin_boundaries
```

### 7.3 Response Format (Success: 200)

```json
{
  "layer_id": "admin_boundaries",
  "layer_name": "Batas Administrasi Indonesia",
  "attributes": [
    {
      "field": "id",
      "type": "integer",
      "description": "Internal database ID",
      "unit": null,
      "example": 1,
      "is_visible_in_tooltip": false
    },
    {
      "field": "name",
      "type": "varchar",
      "description": "Nama wilayah administrasi",
      "unit": null,
      "example": "DKI Jakarta",
      "is_visible_in_tooltip": true,
      "sort_order": 1
    },
    {
      "field": "code",
      "type": "varchar",
      "description": "Kode Kemendagri",
      "unit": null,
      "example": "31",
      "is_visible_in_tooltip": true,
      "sort_order": 2
    },
    {
      "field": "level",
      "type": "varchar",
      "description": "Level administrasi",
      "unit": null,
      "example": "province",
      "is_visible_in_tooltip": true,
      "sort_order": 3
    },
    {
      "field": "area_km2",
      "type": "numeric",
      "description": "Luas wilayah",
      "unit": "km²",
      "example": 664.01,
      "is_visible_in_tooltip": true,
      "sort_order": 4
    },
    {
      "field": "population",
      "type": "bigint",
      "description": "Jumlah penduduk",
      "unit": "jiwa",
      "example": 10562088,
      "is_visible_in_tooltip": true,
      "sort_order": 5
    }
  ]
}
```

### 7.4 Response Format (Error: 404)

```json
{
  "error": {
    "code": "LAYER_NOT_FOUND",
    "message": "Layer with ID 'invalid_layer' not found.",
    "details": {
      "layer_id": "invalid_layer",
      "available_layers": ["admin_boundaries", "infrastructure"]
    },
    "timestamp": "2026-06-17T09:03:08+07:00",
    "request_id": "abc-123-def"
  }
}
```

---

## 8. Endpoint: Health Check

**API-ID:** `API-HEALTH-01`
**Method:** `GET`
**Endpoint:** `/health`
**Authentication:** Public

### 8.1 Response Format (Success: 200)

```json
{
  "status": "healthy",
  "timestamp": "2026-06-17T09:03:08+07:00",
  "version": "1.0.0",
  "dependencies": {
    "postgres": {
      "status": "connected",
      "version": "15.4",
      "connection_pool": {
        "active": 3,
        "idle": 7,
        "max": 20
      }
    },
    "postgis": {
      "status": "loaded",
      "version": "3.3.2"
    }
  },
  "system": {
    "memory_usage_mb": 128,
    "cpu_usage_percent": 12,
    "uptime_seconds": 86400
  }
}
```

### 8.2 Response Format (Degraded: 503)

```json
{
  "status": "degraded",
  "timestamp": "2026-06-17T09:03:08+07:00",
  "version": "1.0.0",
  "dependencies": {
    "postgres": {
      "status": "connected",
      "latency_ms": 250
    },
    "postgis": {
      "status": "loaded",
      "version": "3.3.2"
    }
  },
  "alerts": [
    {
      "severity": "warning",
      "message": "Database response time above threshold (> 200ms)"
    }
  ]
}
```

---

## 9. Query Parameters Reference

### 9.1 Spatial Query Parameters

| Parameter | Type | Format | Valid Range | Description |
|-----------|------|--------|-------------|-------------|
| `bbox` | string | `minx,miny,maxx,maxy` | Lon: 95.3-141.0, Lat: -10.9-5.9 | Bounding box dalam EPSG:4326. |
| `zoom_level` | integer | integer | 3-18 | Zoom level untuk generalization dan filtering. |

### 9.2 Filter Parameters

| Parameter | Type | Valid Values | Description |
|-----------|------|-------------|-------------|
| `level` | string | `province`, `regency`, `district`, `village` | Filter level administrasi. |
| `type` | string | `national_road`, `railway`, `airport`, `dam` | Filter tipe infrastruktur. |

---

## 10. Response Schemas

### 10.1 GeoJSON FeatureCollection (Standar)

```typescript
type GeoJSONFeatureCollection = {
  type: "FeatureCollection";
  features: Array<{
    type: "Feature";
    id: string;
    geometry: {
      type: "MultiPolygon" | "LineString" | "Point";
      coordinates: any;
    };
    properties: Record<string, any>;
  }>;
  metadata?: {
    zoom_level: number;
    feature_count: number;
    bbox: [number, number, number, number];
    crs: string;
    response_time_ms: number;
  };
};
```

### 10.2 Layer Metadata Schema

```typescript
type LayerMetadata = {
  id: string;
  name: string;
  description: string;
  type: "base_map" | "administrative_boundary" | "infrastructure" | "custom";
  geom_type: "Point" | "LineString" | "Polygon" | "MultiPolygon";
  srid: number;
  attribution: string;
  default_visibility: boolean;
  z_index: number;
  attributes: Array<{
    field: string;
    type: string;
    description: string;
    unit: string | null;
  }>;
};
```

### 10.3 Error Response Schema

```typescript
type ErrorResponse = {
  error: {
    code: string;
    message: string;
    details?: Record<string, any>;
    timestamp: string;
    request_id: string;
  };
};
```

---

## 11. Rate Limiting

### 11.1 Limits per Endpoint

| Endpoint | Limit | Window | Burst |
|----------|-------|--------|-------|
| `/api/v1/vector/*` | 100 requests | 1 minute | 20 |
| `/api/v1/metadata/*` | 30 requests | 1 minute | 10 |
| `/health` | Unlimited | - | - |

### 11.2 Rate Limit Headers

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1718640188
```

---

## 12. Pagination

Vector endpoints menggunakan **cursor-based pagination** untuk dataset besar.

### 12.1 Pagination Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | integer | 5000 | Maksimal features per response. Max: 10000. |
| `cursor` | string | null | Cursor untuk pagination (dari response sebelumnya). |

### 12.2 Paginated Response

```json
{
  "type": "FeatureCollection",
  "features": [ ... ],
  "pagination": {
    "limit": 5000,
    "cursor": "eyJ...",
    "has_more": true,
    "total_count": 25000
  }
}
```

---

## 13. Caching Strategy

### 13.1 Cache Headers

| Scenario | Cache-Control | Max Age |
|----------|--------------|---------|
| Vector data (GeoJSON) | `public, max-age=300, s-maxage=600` | 5 min client, 10 min CDN |
| Vector data (MVT) | `public, max-age=3600, s-maxage=86400` | 1 hour client, 1 day CDN |
| Metadata | `public, max-age=3600` | 1 hour |
| Health check | `no-cache, no-store` | - |

### 13.2 ETag Support

Responses menyertakan ETag berdasarkan:
- Hash dari query parameters.
- `last_updated` timestamp dari database.

Client dapat mengirim `If-None-Match` untuk conditional request.

---

## 14. Content Negotiation

### 14.1 Format Selection

| Accept Header | Response Format |
|--------------|-----------------|
| `application/json` | GeoJSON FeatureCollection |
| `application/x-protobuf` | Mapbox Vector Tiles (MVT) |
| `*/*` | Default: GeoJSON |

### 14.2 MVT Request Example

```
GET /api/v1/vector/admin-boundaries?bbox=106.7,-6.3,107.0,-6.1&zoom_level=12
Accept: application/x-protobuf
```

Response: Binary MVT data dengan Content-Type `application/x-protobuf`.

---

## 15. PostGIS Query Patterns

### 15.1 Spatial Query Template

```sql
-- Query dasar untuk vector endpoints
SELECT 
    id,
    layer_id,
    level,
    code,
    name,
    province_name,
    regency_name,
    district_name,
    population,
    area_km2,
    geom,
    valid_from,
    valid_to
FROM admin_boundaries
WHERE 
    layer_id = (SELECT id FROM layers WHERE name = 'Batas Administrasi Indonesia')
    AND valid_to IS NULL
    AND ST_Intersects(
        geom,
        ST_MakeEnvelope(:xmin, :ymin, :xmax, :ymax, 4326)
    )
    AND (:level IS NULL OR level = :level)
ORDER BY name ASC
LIMIT :limit OFFSET :offset;
```

### 15.2 Geometry Simplification Query

```sql
-- Simplifikasi geometri berdasarkan zoom level
SELECT 
    id,
    name,
    ST_SimplifyPreserveTopology(geom, :tolerance) AS geom,
    area_km2
FROM admin_boundaries
WHERE 
    ST_Intersects(geom, ST_MakeEnvelope(:xmin, :ymin, :xmax, :ymax, 4326))
    AND zoom_level < 10
LIMIT :limit;
```

**Tolerance by Zoom Level:**

| Zoom Level | Tolerance (degrees) | Approximation |
|-----------|---------------------|---------------|
| 3-5 | 0.1 | ~11 km |
| 6-8 | 0.01 | ~1.1 km |
| 9-11 | 0.001 | ~110 m |
| 12-14 | 0.0001 | ~11 m |
| 15-18 | 0 (no simplification) | Original detail |

---

## 16. API Handler Implementation (Golang Fiber)

### 16.1 Vector Handler

```go
// handlers/vector_handler.go
func (h *VectorHandler) GetAdminBoundaries(c *fiber.Ctx) error {
    bbox := c.Query("bbox")
    zoomLevel := c.QueryInt("zoom_level", 0)
    level := c.Query("level")

    if err := validateBBox(bbox); err != nil {
        return c.Status(fiber.StatusBadRequest).JSON(ErrorResponse{
            Code:    "INVALID_BBOX",
            Message: err.Error(),
        })
    }

    features, err := h.postgisService.QueryAdminBoundaries(bbox, zoomLevel, level)
    if err != nil {
        return c.Status(fiber.StatusInternalServerError).JSON(ErrorResponse{
            Code:    "DATABASE_ERROR",
            Message: "Failed to query spatial data",
        })
    }

    return c.JSON(GeoJSONResponse{
        Type:     "FeatureCollection",
        Features: features,
    })
}
```

### 16.2 Metadata Handler

```go
// handlers/metadata_handler.go
func (h *MetadataHandler) GetLayers(c *fiber.Ctx) error {
    layers, err := h.postgisService.GetAllLayers()
    if err != nil {
        return c.Status(fiber.StatusInternalServerError).JSON(ErrorResponse{
            Code:    "DATABASE_ERROR",
            Message: "Failed to fetch layers",
        })
    }
    return c.JSON(layers)
}
```

---

## 17. Testing Specifications

### 17.1 API Test Matrix

| Test ID | Endpoint | Test Case | Expected Result |
|---------|----------|-----------|----------------|
| API-TC-01 | `/vector/admin-boundaries` | Valid bbox, no filter | 200, FeatureCollection dengan features |
| API-TC-02 | `/vector/admin-boundaries` | Invalid bbox format | 400, `INVALID_BBOX` |
| API-TC-03 | `/vector/admin-boundaries` | Zoom level out of range | 400, `INVALID_ZOOM` |
| API-TC-04 | `/vector/admin-boundaries` | Invalid level value | 400, `INVALID_LEVEL` |
| API-TC-05 | `/vector/admin-boundaries` | Layer exists but empty bbox | 200, empty FeatureCollection |
| API-TC-06 | `/metadata/layers` | Normal request | 200, array of layer objects |
| API-TC-07 | `/metadata/attributes/:id` | Valid layer ID | 200, attribute definitions |
| API-TC-08 | `/metadata/attributes/:id` | Invalid layer ID | 404, `LAYER_NOT_FOUND` |
| API-TC-09 | `/health` | Normal request | 200, health status |
| API-TC-10 | All endpoints | Rate limit exceeded | 429, `RATE_LIMIT_EXCEEDED` |

### 17.2 Performance Test Targets

| Metric | Target | Test Method |
|--------|--------|-------------|
| P50 Response Time | ≤ 100ms | Automated load test (k6/Artillery) |
| P95 Response Time | ≤ 200ms | Automated load test |
| P99 Response Time | ≤ 500ms | Automated load test |
| Throughput | 100 req/sec sustained | Load test |
| Max Concurrent | 50 concurrent users | Load test |

---

## 18. OpenAPI 3.0 Specification

```yaml
openapi: 3.0.3
info:
  title: WebGIS API
  description: RESTful API untuk aplikasi WebGIS - akses data spasial Indonesia
  version: 1.0.0
  contact:
    name: API Support

servers:
  - url: https://api.webgis.example.com
    description: Production
  - url: https://api-staging.webgis.example.com
    description: Staging
  - url: http://localhost:3000
    description: Local Development

security:
  - BearerAuth: []

paths:
  /api/v1/vector/admin-boundaries:
    get:
      operationId: getAdminBoundaries
      summary: Mengambil data batas administrasi
      tags: [Vector]
      parameters:
        - name: bbox
          in: query
          required: true
          schema:
            type: string
            example: "95.3,-10.9,141.0,5.9"
        - name: zoom_level
          in: query
          required: true
          schema:
            type: integer
            minimum: 3
            maximum: 18
        - name: level
          in: query
          schema:
            type: string
            enum: [province, regency, district, village]
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/GeoJSONFeatureCollection'
            application/x-protobuf:
              schema:
                type: string
                format: binary
        '400':
          $ref: '#/components/responses/BadRequest'
        '500':
          $ref: '#/components/responses/InternalError'

  /api/v1/vector/infrastructure:
    get:
      operationId: getInfrastructure
      summary: Mengambil data infrastruktur
      tags: [Vector]
      parameters:
        - name: bbox
          in: query
          required: true
          schema:
            type: string
        - name: zoom_level
          in: query
          required: true
          schema:
            type: integer
            minimum: 3
            maximum: 18
        - name: type
          in: query
          schema:
            type: string
            enum: [national_road, railway, airport, dam]
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/GeoJSONFeatureCollection'
        '400':
          $ref: '#/components/responses/BadRequest'
        '500':
          $ref: '#/components/responses/InternalError'

  /api/v1/metadata/layers:
    get:
      operationId: getLayers
      summary: Mendapatkan metadata semua layer
      tags: [Metadata]
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/LayerMetadata'
        '500':
          $ref: '#/components/responses/InternalError'

  /api/v1/metadata/attributes/{layerId}:
    get:
      operationId: getLayerAttributes
      summary: Mendapatkan definisi atribut untuk layer tertentu
      tags: [Metadata]
      parameters:
        - name: layerId
          in: path
          required: true
          schema:
            type: string
            example: admin_boundaries
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/LayerAttributeDetail'
        '404':
          $ref: '#/components/responses/NotFound'
        '500':
          $ref: '#/components/responses/InternalError'

  /health:
    get:
      operationId: healthCheck
      summary: Health check endpoint
      tags: [System]
      responses:
        '200':
          description: Healthy
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/HealthResponse'
        '503':
          description: Degraded
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/HealthResponse'

components:
  schemas:
    GeoJSONFeatureCollection:
      type: object
      required: [type, features]
      properties:
        type:
          type: string
          example: "FeatureCollection"
        features:
          type: array
          items:
            $ref: '#/components/schemas/Feature'
        metadata:
          type: object
          properties:
            zoom_level:
              type: integer
            feature_count:
              type: integer
            bbox:
              type: array
              items:
                type: number
              minItems: 4
              maxItems: 4
            crs:
              type: string
              example: "EPSG:4326"

    Feature:
      type: object
      required: [type, geometry, properties]
      properties:
        type:
          type: string
          example: "Feature"
        id:
          type: string
        geometry:
          $ref: '#/components/schemas/Geometry'
        properties:
          type: object

    Geometry:
      type: object
      required: [type, coordinates]
      properties:
        type:
          type: string
          enum: [Point, LineString, Polygon, MultiPolygon]
        coordinates:
          type: array

    LayerMetadata:
      type: object
      required: [id, name, type, geom_type]
      properties:
        id:
          type: string
        name:
          type: string
        description:
          type: string
        type:
          type: string
          enum: [base_map, administrative_boundary, infrastructure, custom]
        geom_type:
          type: string
          enum: [Point, LineString, Polygon, MultiPolygon]
        srid:
          type: integer
          default: 4326
        attribution:
          type: string
        default_visibility:
          type: boolean
          default: true
        z_index:
          type: integer
          default: 0
        attributes:
          type: array
          items:
            $ref: '#/components/schemas/AttributeDefinition'

    AttributeDefinition:
      type: object
      required: [field, type, description]
      properties:
        field:
          type: string
        type:
          type: string
          enum: [string, number, boolean, date]
        description:
          type: string
        unit:
          type: string
          nullable: true

    LayerAttributeDetail:
      type: object
      required: [layer_id, attributes]
      properties:
        layer_id:
          type: string
        layer_name:
          type: string
        attributes:
          type: array
          items:
            $ref: '#/components/schemas/AttributeDetail'

    AttributeDetail:
      type: object
      required: [field, type, description]
      properties:
        field:
          type: string
        type:
          type: string
        description:
          type: string
        unit:
          type: string
          nullable: true
        example:
          type: any
          nullable: true
        is_visible_in_tooltip:
          type: boolean
          default: true
        sort_order:
          type: integer
          default: 0

    HealthResponse:
      type: object
      required: [status, timestamp, version]
      properties:
        status:
          type: string
          enum: [healthy, degraded]
        timestamp:
          type: string
          format: date-time
        version:
          type: string
        dependencies:
          type: object
        system:
          type: object
        alerts:
          type: array
          items:
            type: object

    ErrorResponse:
      type: object
      required: [error]
      properties:
        error:
          type: object
          required: [code, message, timestamp, request_id]
          properties:
            code:
              type: string
            message:
              type: string
            details:
              type: object
            timestamp:
              type: string
              format: date-time
            request_id:
              type: string

  responses:
    BadRequest:
      description: Invalid request parameters
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    InternalError:
      description: Server error
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'

  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

---

## 19. Catatan untuk Development Team

1. **CRS Consistency:** Semua koordinat dalam request dan response menggunakan EPSG:4326 (WGS84). Backend bertanggung jawab transformasi dari/to EPSG:3857.

2. **ST_Intersects Performance:** Gunakan `ST_MakeEnvelope` untuk bbox, bukan `ST_Envelope(geom) && bbox`. Index GIST pada `geom` column adalah wajib.

3. **Geometry Simplification:** Implementasikan `ST_SimplifyPreserveTopology` berdasarkan zoom level. Jangan simplify untuk zoom >= 12.

4. **Response Size Limit:** Maksimal 10MB per GeoJSON response. Untuk dataset lebih besar, client harus menggunakan pagination atau MVT format.

5. **Parameterized Queries:** Semua query PostGIS menggunakan prepared statements atau parameter binding untuk mencegah SQL injection.

6. **CORS Headers:** Implementasikan CORS middleware di Fiber dengan whitelist origin yang strict.

7. **Request ID:** Generate `X-Request-ID` di setiap request untuk distributed tracing.

8. **Health Check Depth:** Health check melakukan lightweight query (`SELECT 1`) ke database, bukan full table scan.

9. **MVT Fallback:** Client yang支持 MVT harus mencoba Accept header `application/x-protobuf` terlebih dahulu sebelum fallback ke GeoJSON.

---

*Document Owner: Product Management & Backend Architecture Team*
*Last Review: 2026-06-17*
*Next Review: 2026-07-01*
*Related Documents:* `docs/prd.md`, `docs/srs.md`, `docs/domain-model.md`, `docs/erd.md`
