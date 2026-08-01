# Data Architecture — GeoRestore AI

Status: approved blueprint component
Parent: [[Architecture_Plan]]

## 1. Data Catalog

| Source | Format | Use | Band set | Notes |
|---|---|---|---|---|
| LISS-IV (Bhoonidhi) | GeoTIFF | primary optical | G, R, NIR | ~5.8 m; license terms apply |
| Sentinel-1 | GRD GeoTIFF | SAR auxiliary | VV, VH | all-weather structure; coregistration needed |
| Sentinel-2 | GeoTIFF | auxiliary optical / cloud-free refs | 10 m bands | cross-scale study |
| DEM (SRTM/ALOS) | GeoTIFF | optional channels | elevation | optional |
| Temporal stack | GeoTIFF | temporal fusion | LISS-IV | same-scene cloud-free dates |

## 2. Directory Layout

```
data/
  raw/            # immutable sources (never written after ingestion)
  processed/      # normalized, resampled, coregistered per dataset version
  patches/        # tiles + masks + split manifests
  benchmarks/     # synthetic benchmark manifests + fixtures
  metadata/       # manifests.json, stats, processing logs
outputs/
  predictions/    # per-job model outputs (restored rasters)
  geotiff/        # final GeoTIFF deliverables (COG)
  visualizations/ # previews, comparison images
models/           # checkpoints (also mirrored to MLflow artifacts)
```

Rule: `data/` and `outputs/` are **outside git** (see `.gitignore`).
`data/raw/` is append-only.

## 3. Versioning Model

- **Dataset version** = folder + `manifest.json` (single source of truth).
- A version is immutable once created. Any change (masker, normalization,
  synthetic recipe, split, new scenes) → new version.
- Manifest content:

```json
{
  "dataset_version": "ds_v001",
  "schema_version": 1,
  "created_at": "2026-08-01T00:00:00Z",
  "sources": {
    "liss4": {"dir": "data/raw/liss4_2026q3", "license": "bhoonidhi-terms", "count": 24},
    "sentinel1": {"dir": "data/raw/s1_2026q3", "count": 0}
  },
  "processing": [
    {"step": "resample", "target_grid": "liss4_5.8m", "params": {}},
    {"step": "normalize", "mode": "zscore", "stats_origin": "train_only"}
  ],
  "mask": {"algorithm": "v1_threshold", "params": {"min_ndvi": 0.0, "radius": 3}},
  "synthetic": {"algorithm": "fbm_v1", "seed": 42, "severities": ["thin", "moderate", "thick"]},
  "splits": {"rule": "scene_level_70_15_15", "train": ["s001", "s002", "..."], "val": ["..."], "test": ["..."]},
  "files": [{"path": "patches/train/s001/000_000.npz", "sha256": "..."}],
  "stats": {"bands": ["G", "R", "NIR"], "mean": [..], "std": [..], "origin": "train"}
}
```

- **Normalization stats** are computed from the train split only and frozen
  in the manifest.

## 4. Integrity

- SHA-256 recorded for every file in manifests and in PostGIS.
- Ingestion validates: TIFF magic, GDAL-open, CRS present, geotransform
  sane, band count matches expected.
- Reproducibility keys: dataset_version + config_sha + model_version + seed.

## 5. Geospatial Conventions

- Validate CRS + geotransform on every open (`src/geo`).
- All processing happens on an explicit target grid (default: LISS-IV
  5.8 m grid of the reference scene, chosen at dataset-version creation).
- Rasters keep native CRS; footprints in EPSG:4326 (PostGIS).
- Band order + scaling (DN → reflectance) recorded in manifest; never
  assumed.
- Outputs: Cloud-Optimized GeoTIFF (tiled 256×256, overviews, DEFLATE or
  ZSTD, BIGTIFF when >4 GB).
- Windowed I/O only; never materialize full rasters in memory.
- Cloud masks: uint8 (0 clear, 1 cloud, optional 2 shadow); mask algorithm
  + params stored beside the mask.

## 6. Patch Convention

- Default patch size 256×256; configurable 128–512.
- Stride 192 (25% overlap); overlap blending uses distance-weighted seams.
- Channel order: `[G, R, NIR, cloud_mask]` (optical); SAR fusion adds
  `[VV, VH]`; temporal adds stacked dates.
- Patches with >95% cloud or >95% nodata dropped (count recorded).
- Every patch has provenance: scene id, window coords, split, mask
  algorithm, synthetic recipe + seed.

## 7. Synthetic Cloud Generation (controlled supervised training)

Versioned + seeded generators:

1. **FBM/PF fractal clouds** — Perlin/FBM noise → opacity; physics-informed
   transmittance for thin clouds; thick clouds zero the signal.
2. **Cloud mixing** — real cloud layers from cloudy scenes composited onto
   clear scenes (LISS-IV or Sentinel-2 cloud scenes).
3. **Controlled recipes** — severity classes (thin/moderate/thick) and cloud
   fraction bins (0–10, 10–30, 30–60, 60–100%).

Used for training (paired GT) and benchmark evaluation. Real-cloud evaluation
never uses synthetic GT.

## 8. Expected Scale (indicative)

- Phase 1: ≥20 LISS-IV scenes → ≥5k train patches.
- Phase 2: 40–100 scenes → 20–50k patches (synthetic + real).
- Phase 3: +S1/S2 paired regions (100–300 scenes if available).

## 9. Data Handling Rules

- `data/raw/` append-only; derived dirs regenerable from manifests.
- No large files in `knowledge/` or git.
- Licenses recorded per source; Bhoonidhi terms respected.
