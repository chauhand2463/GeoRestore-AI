# Risks — GeoRestore AI

Status: approved blueprint component
Parent: [[Architecture_Plan]]

## 1. Scientific & Technical Risk Register

Legend: L = likelihood (L/M/H), I = impact (L/M/H).

| ID | Risk | L | I | Mitigation | Trigger / owner |
|---|---|---|---|---|---|
| R-P1 | Cloud mask errors propagate into output and metrics | M | H | mask QC per dataset version (IoU on labeled set); mask params versioned; uncertainty layer; manual mask override in UI | mask IoU drops below threshold |
| R-P2 | No cloud-free LISS-IV GT for real validation | H | H | dual-track eval; temporal references; downstream checks (E011) | phase 2 review |
| R-P3 | SAR↔optical coregistration misalignment ruins fusion | M | H | AROSICS-style coreg + residual-shift QC gate before M3 | coreg QC gate |
| R-P4 | Overfitting to synthetic clouds | H | H | real-cloud holdout always; cloud-mixing real recipes; E016 | E011 vs E004 gap |
| R-P5 | Radiometric inconsistency between scenes/periods | M | M | per-version normalization (train-only); scaling recorded | dataset QC |
| R-P6 | OOM on large rasters | M | M | windowed ops only; BIGTIFF; memory budgets | load test |
| R-P7 | GAN instability / mode collapse | M | M | controlled protocol; EMA; checkpoints; diagnostics | loss divergence |
| R-P8 | Metric gaming (blurring → high PSNR) | M | M | joint SSIM/SAM/LPIPS; mask-aware; downstream; visual QC | review |
| R-P9 | Spectral drift in reconstructed bands | M | M | SAM loss; band-wise metrics; spectral stats | E012 |
| R-P10 | Temporal mismatch (land change between dates) | M | M | date-window constraints; document change risk | E007 |
| R-P11 | Dependency rot / lost reproducibility | M | M | lockfiles; sha manifests; containerized runs | CI failures |
| R-P12 | UI presenting reconstructions as truth | M | M | status layer; uncertainty display; disclaimers | UI review |

## 2. Hallucination Risks (Q)

| ID | Risk | Mitigation |
|---|---|---|
| R-H1 | Structural hallucination (plausible-but-wrong land cover) | uncertainty maps; downstream degradation analysis; conservative reporting |
| R-H2 | Spectral hallucination (invented spectra) | SAM loss; spectral statistics checks; band-wise metrics |
| R-H3 | Over-smoothing vs over-sharpening artifacts | joint metric reporting; visual QC set; LPIPS |
| R-H4 | Confidence illusion | calibrated uncertainty; reliability reports; never stripped from outputs |
| R-H5 | Distribution shift (season/region) | spatially held-out tests (E013); stratified experiments |

## 3. Data Leakage Risks (R)

| ID | Leak path | Control |
|---|---|---|
| R-L1 | Patch-level random splits across scenes | scene-level splits only |
| R-L2 | Spatially adjacent scenes in train and test | footprint buffer ≥1 tile; automated leakage check in CI |
| R-L3 | Normalization stats from full dataset | train-only stats, frozen per version |
| R-L4 | Masker tuned on test scenes | masker params chosen on val; frozen per version |
| R-L5 | Temporal overlap across splits | split by area; documented |
| R-L6 | Synthetic generator state shared across splits | per-version seed; region separation |
| R-L7 | Model selection on test | test once per dataset version; protocol |
| R-L8 | Nondeterministic augmentation seeds | per-sample seeds in loaders |
| R-L9 | Correlated patch metrics | per-scene aggregation + bootstrap |

## 4. Engineering & Operational Risks

| ID | Risk | Mitigation |
|---|---|---|
| R-E1 | Upload path traversal / unsafe files | sanitized names, id-derived paths, GDAL-open probe |
| R-E2 | Model artifact injection | registry-only loads; trusted checkpoints; checksums |
| R-E3 | Queue/worker failure mid-job | idempotent retries; job timeout; status audit |
| R-E4 | DB corruption / data loss | nightly dumps; tested restore; append-only manifests |
| R-E5 | Secret leakage | env-based secrets; `.env.example` only; secret scan in CI |

## 5. Monitoring Cadence

- Per experiment: bookkeeping keys verified (EXPERIMENT_MATRIX.md §3).
- Per dataset version: mask QC, split QC, sha verification.
- Per release: risk register review + security checklist (Phase 4 gate).

## 6. Open Questions (resolve before Phase 1 ends)

1. Bhoonidhi license / download terms and level (L1/L2, DN scaling).
2. Availability of temporal cloud-free LISS-IV references.
3. GPU size — affects patch/batch.
4. Phase 4 deployment target (on-prem vs cloud).
5. "Restore-all" mode needed? (default: preserve clear pixels).
