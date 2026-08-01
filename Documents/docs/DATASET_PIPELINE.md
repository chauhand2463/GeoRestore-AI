# Dataset Pipeline — GeoRestore AI

Status: approved blueprint component
Parent: [[Data_Architecture]]

## 1. End-to-End Stages

| Stage | Tool (planned) | Output |
|---|---|---|
| Ingest LISS-IV | `python -m preprocessing.ingest_liss4 --src ... --out data/raw` | raw GeoTIFF + sidecar metadata |
| Validate | `src/geo` (auto) | CRS/geotransform/bands checks pass |
| Resample to grid | `python -m preprocessing.resample --grid liss4_5.8m` | `data/processed/` |
| Normalize (train-only stats) | `python -m preprocessing.normalize --stats-from train` | normalized rasters + stats JSON |
| Cloud mask v1 | `python -m masking.mask_scene --algorithm v1_threshold` | uint8 mask + params |
| Patch extraction | `python -m datasets.tile --size 256 --stride 192` | `data/patches/` |
| Scene-based split | `python -m datasets.split --rule scene_70_15_15 --buffer 1` | split manifest |
| Synthetic clouds | `python -m synthetic_cloud.generate --recipe fbm_v1 --seed 42` | synthetic pairs (patches) |
| Manifest build | `python -m datasets.build_manifest` | `manifest.json` + sha256 |

CLI names are indicative; the pipeline is config-driven and the manifest is
the single source of truth.

## 2. Cloud Masking v1 (classical)

- Threshold-based (brightness + NDVI priors), morphological cleanup
  (opening/closing), optional shadow class.
- Output: uint8 `(0=clear, 1=cloud, 2=shadow)`; confidence variant optional.
- Params frozen per dataset version.
- Learned masker deferred (Phase 2+); mask quality hook in place from v1
  (compare against hand-labeled QC set).

## 3. Patch Generation

- Default 256×256; stride 192 (25% overlap); configurable 128–512.
- Aligned to the scene grid; provenance (scene, window, split) on every
  patch.
- Drop rules: >95% cloud or >95% nodata (counts recorded in manifest).
- Channel order: `[G, R, NIR, cloud_mask]`; SAR adds `[VV, VH]`; temporal
  adds stacked dates.

## 4. Splits (leakage-free)

- **Unit: scene.** Patches inherit the scene's split.
- Ratio 70/15/15 (train/val/test), scene-level.
- Footprint overlap check: no train/test scenes within 1 tile margin;
  automated check (`datasets.split --check-leakage`) in CI.
- Stratification by region/season when known; one geographic holdout for
  generalization (E013).
- Split assignment frozen in the manifest; changing a split → new dataset
  version.

## 5. Synthetic Cloud Recipes

| Recipe | Mechanism | Severities | Use |
|---|---|---|---|
| `fbm_v1` | Perlin/FBM noise → opacity; transmittance model for thin; zero-out for thick | thin/moderate/thick | training + E004/E005 |
| `mix_v1` | real cloud layers composited onto clear scenes | n/a | training robustness |
| `bench_v1` | fixed recipes + fraction bins 0–10/10–30/30–60/60–100% | all | benchmark (E001–E005) |

All recipes are seeded per dataset version (deterministic regeneration).

## 6. Manifest Schema

See DATA_ARCHITECTURE.md §3. Required fields: dataset_version,
schema_version, sources (with licenses), processing steps, mask params,
synthetic recipes + seeds, split assignment, file list with sha256, train-only
normalization stats.

## 7. Quality Gates

- Ingestion: 100% of scenes pass CRS/geotransform/band validation.
- Mask QC: ≥1 hand-labeled scene per dataset version; report IoU.
- Split QC: zero footprint overlap across splits (automated).
- Manifest QC: all sha256 verify; stats frozen and documented.

## 8. Task Dependencies

1. `src/geo` → 2. ingestion → 3. manifests → 4. preprocessing →
5. masking → 6. tiling → 7. splits → 8. synthetic clouds → 9. ds_v001 build.
