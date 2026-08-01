# Roadmap — GeoRestore AI

Status: approved blueprint component
Parent: [[Architecture_Plan]], knowledge note: [[Roadmap]] (vault)

## 1. Phases

| Phase | Name | Duration (indicative) | Scope |
|---|---|---|---|
| 0 | Foundations | ~2 weeks | repo skeleton, configs, geo I/O, dataset tooling, versioning, CI |
| 1 | MVP Baseline | ~3–4 weeks | ingestion→synthetic clouds; M0 + M1; training; eval lib; API + jobs; basic web; E001 |
| 2 | Research Prototype | ~6–8 weeks | M2 cGAN; E001–E005; MLflow; uncertainty (E009); downstream (E010); full GeoTIFF outputs |
| 3 | Multimodal System | ~8–12 weeks | S1/S2 ingestion + coreg; M3 fusion (E006); temporal M4 (E007); diffusion M5 (E008); generalization (E013) |
| 4 | Production Platform | ~4–6 weeks | auth, tiling, monitoring, backups, load test, security review, runbook |

Gates: phases advance only when the prior phase's DoD passes (below).

## 2. Definition of Done

| Phase | DoD |
|---|---|
| 0 | pyproject lock; ruff+mypy+pytest green; config loader + hash; geo I/O round-trip tests; manifest tool; CI on push |
| 1 | ≥20 LISS-IV scenes versioned; masks generated; ≥5k train patches; M0+M1 train; eval lib tested (PSNR/SSIM/RMSE/MAE/SAM, mask-aware); API: upload/cloud-detect/restore/jobs/models; async restore end-to-end → GeoTIFF + mask + metadata; web: upload→view→download; E001 documented |
| 2 | M2 trained + registered; E001–E005 with keys; MLflow UI usable; calibration report (E009); downstream eval (E010); outputs incl. uncertainty + status layers; Experiment notes per template |
| 3 | S1/S2 pipeline with coreg QC; M3 beats optical-only on cloud-region metrics or documented negative; E006/E007 documented; M5 experiment documented (even negative); E013 done |
| 4 | auth + rate limiting; monitoring; backup/restore tested; ≥10 concurrent jobs load test; security checklist passed; runbook; release notes; architecture review |

## 3. Milestones

- **M1 (MVP)**: restore job on a real GeoTIFF via web UI, metrics shown.
- **M2 (Research)**: cGAN vs U-Net result with CIs (E002) + calibration.
- **M3 (Multimodal)**: fusion result (E006) + temporal (E007).
- **M4 (Production)**: deployment runbook + release.

## 4. Task Breakdown — First 20 Implementation Tasks (dependency-ordered)

| # | Task | Depends on |
|---|---|---|
| 1 | Repo skeleton: pyproject, ruff/mypy/pytest, CI, .env.example | — |
| 2 | Config system: pydantic-settings + YAML loader + config hash | 1 |
| 3 | `src/geo`: rasterio wrapper (open/validate/COG write) + tests | 2 |
| 4 | LISS-IV ingestion CLI (Bhoonidhi → `data/raw` + sidecar) | 3 |
| 5 | Dataset manifest tool (sha256, schema version, stats, immutable) | 4 |
| 6 | Preprocessing: resample/normalize (train-only stats), band checks | 3, 5 |
| 7 | Cloud masking v1 (threshold + morphological cleanup) + QC hook | 6 |
| 8 | Patch extraction: grid tiling + overlap + provenance | 6, 7 |
| 9 | Scene-based split tool + leakage checks (footprint buffer) | 8 |
| 10 | Synthetic cloud module v1 (FBM + mixing, seeded, severity labels) | 8 |
| 11 | Dataset v001 build: 20 scenes → patches + masks + manifest + splits | 5–10 |
| 12 | M1 U-Net (mask-conditioned) + config | 2 |
| 13 | M0 classical baselines (Telea/NS + GDAL fill wrappers) | 3 |
| 14 | Training loop v1 (seeded, checkpointing, val, logging) | 2, 12 |
| 15 | Evaluation lib: PSNR/SSIM/RMSE/MAE/SAM per-band + mask-aware + CI | 3 |
| 16 | MLflow integration (run = config hash, dataset version, seed) | 14, 15 |
| 17 | API skeleton: health, models, jobs CRUD, upload validation | 2, 3 |
| 18 | Async job runner (Redis + arq): restore pipeline as job | 16, 17 |
| 19 | Inference engine: windowed tiling → predict → blend → GeoTIFF + mask + uncertainty + metadata | 12, 15, 18 |
| 20 | Web v1: upload, model select, job poll, result view, metrics, download | 17, 18, 19 |

Task 20 = MVP milestone gate.

## 5. Parallelization

- Data stack (tasks 3–10) and model stack (12–16) proceed in parallel after
  task 2/3.
- Evaluation lib (15) independent of models (needs only data types).
- Service stack (17–19) depends on interfaces, not internals.

## 6. Cadence & Reporting

- Each task lands with tests (AGENTS.md §5) and a git commit.
- Every experiment closes with an Experiment note + MLflow run.
- Phase reviews at each gate; risk register updated (RISKS.md §5).

## 7. Definition of Done for the Whole Project (Phase 4 end)

Platform: documented, deployable, backed up, security-reviewed. Science:
E001–E007, E009–E013 recorded with keys; open questions answered; research
question answered with qualified, provenance-carrying results.
