# Architecture Diagram

## Status

Current (canonical diagram for GeoRestore AI).

## Location

- Diagram: `docs/diagrams/GeoRestore_AI_Architecture.excalidraw` — native
  Excalidraw file (editable at <https://excalidraw.com>; JSON, version 2).
- Not a screenshot; all elements (boxes, text, arrows) are editable.
- Renderable in Obsidian with the Excalidraw plugin via `![[GeoRestore_AI_Architecture.excalidraw]]`
  when the file is copied into the vault.

## Purpose

Single architecture figure for:

- project documentation and the GitHub README
- technical presentations
- research paper "architecture" section

## Sections covered

Every box carries a title plus a detail caption (parameters, formats, values).

| # | Section | Contents |
|---|---------|----------|
| 1 | DATA SOURCES | LISS-IV (5.8 m, G/R/NIR), Sentinel-1 SAR (VV/VH), Sentinel-2, Temporal imagery, DEM, Synthetic cloud generator |
| 2 | DATA INGESTION | GeoTIFF ingestion (windowed, rasterio), Metadata extraction (CRS/geotransform/bands/DN→refl), Validation, Dataset registry (manifests, SHA-256, ds_vNNN) |
| 3 | GEO PROCESSING | CRS, Reprojection, Resampling, Band alignment, Normalization (train-split stats), Cloud mask (uint8: 0/1/2), Tiling, Patch generation (256×256, stride 192, 70/15/15) |
| 4 | ML DATASET | Train (70%), Validation (15%), Test (15%), Real-world benchmark, Synthetic benchmark |
| 5 | RESTORATION MODELS | Classical baseline (M0), U-Net (M1), Conditional GAN (M2), SAR-optical fusion (M3), Temporal model (M4), Diffusion/flow (M5) |
| 6 | LOSS FUNCTIONS | Reconstruction (masked L1), Perceptual, Adversarial, Spectral (SAM), Spatial, Temporal consistency |
| 7 | EVALUATION | PSNR, SSIM, RMSE, MAE, SAM, LPIPS, Uncertainty (MC-dropout, calibration), Downstream segmentation, Change detection, Confidence intervals, Paired track, Unpaired track |
| 8 | OUTPUT | Restored GeoTIFF (COG/BIGTIFF), Cloud mask, Uncertainty map, Metrics, Metadata (config_sha/version/seed/commit), Observation-status layer |
| 9 | PLATFORM | FastAPI, PostgreSQL/PostGIS, Redis, Worker system, MLflow, Object storage |
| 10 | FRONTEND | Upload, Map viewer, Original image, Cloud mask, Restored image, Difference map, Uncertainty map, Metrics, Download |
| 11 | DEPLOYMENT | Docker, API, Worker, Database, Object storage, GPU inference |
| 12 | CROSS-CUTTING PRINCIPLES | Config-driven runs, Mask-conditioning, RestorationModel plugin, Scene-level integrity, Deterministic seeds, One-way data flow |

## Data flow

- Main chain (solid arrows):
  `data → preprocessing → dataset → model → evaluation → output → platform`
  (DATA SOURCES → DATA INGESTION → GEO PROCESSING → ML DATASET →
  RESTORATION MODELS → EVALUATION → OUTPUT → PLATFORM →
  FRONTEND → DEPLOYMENT).
- Training path: RESTORATION MODELS → LOSS FUNCTIONS ("training signals").
- Feedback loop (blue arrow): EVALUATION → RESTORATION MODELS,
  "evaluation → experiments → improved model".

## Data categories (color legend, top right)

- **OBSERVED DATA** (green): input imagery and ground truth (data sources,
  original image).
- **RECONSTRUCTED DATA** (blue): restored model output (models, restored
  GeoTIFF / restored image).
- **UNCERTAIN DATA** (orange): low-confidence regions (uncertainty metric,
  uncertainty map).

## Maintenance

- Edit `docs/diagrams/GeoRestore_AI_Architecture.excalidraw` directly in
  Excalidraw; the file is the single source of truth.
- If a copy is used inside the vault (Obsidian Excalidraw plugin), refresh
  it from `docs/`; do not edit the vault copy.

## Related

- [[Architecture_Plan]]
- [[Overview]]
- [[Objectives]]
