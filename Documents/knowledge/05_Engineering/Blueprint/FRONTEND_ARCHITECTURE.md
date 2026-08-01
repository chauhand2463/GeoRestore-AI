# Frontend Architecture — GeoRestore AI

Status: approved blueprint component
Parent: [[Architecture_Plan]]

## 1. Stack

Next.js (App Router) + TypeScript + MapLibre GL (GIS viewer) + TanStack
Query (server state) + zustand (client state).

## 2. Application Structure

```
web/
  app/
    page.tsx              # landing: upload + recent jobs
    scenes/[id]/page.tsx  # scene workspace (layers, compare, metrics)
    models/page.tsx       # model registry viewer
    jobs/[id]/page.tsx    # job detail + artifacts
  components/
    RasterLayer.tsx       # MapLibre raster overlay
    LayerSwitcher.tsx     # original / mask / restored / uncertainty / diff
    CompareSlider.tsx     # before-after swipe
    MetricsTable.tsx      # per-band + mask-aware metrics
    UploadDropzone.tsx
    JobProgress.tsx
  lib/
    api.ts                # typed API client (fetch)
    types.ts              # API schema types (mirror pydantic)
```

## 3. Pages / Views

| View | Function |
|---|---|
| Upload | drag-drop GeoTIFF, size/type pre-check, metadata inspection (CRS, bands, stats) |
| Scene workspace | layer switcher: original / cloud mask / restored / uncertainty / diff; compare slider |
| Model selection | list from `GET /api/v1/models`; version + status + metrics shown |
| Run restore | choose scene + model + options → job → progress |
| Metrics | PSNR/SSIM/RMSE/MAE/SAM per band + mask-aware table; simple charts |
| History | jobs list with model + dataset version + artifacts |
| Download | restored / mask / uncertainty GeoTIFFs |

## 4. Raster Display Strategy (v1)

- Server generates previews: downsampled WebP/PNG of COG overviews.
- MapLibre renders previews as raster overlays (no client-side GDAL).
- Full slippy-tile pipeline deferred to Phase 4.

## 5. Data Flow

```
Upload → POST /api/v1/cloud-detect|restore → {job_id}
→ poll GET /api/v1/jobs/{id} (TanStack Query, backoff)
→ artifacts URLs → RasterLayer overlays + MetricsTable
```

## 6. Non-Goals (initial phases)

- Client-side raster processing.
- Offline/PWA, mobile apps, real-time streaming.
- Full tile service (Phase 4 minimal).
- Visual polish beyond functional correctness (priority order).

## 7. Conventions

- Types mirror the API schemas (generated from OpenAPI when feasible).
- No business logic in the frontend; all logic in the API.
- Accessibility basics: keyboard navigation, ARIA labels on tools.
