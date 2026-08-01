# Project Vision — GeoRestore AI

Status: approved blueprint component
Parent: [[Architecture_Plan]] (repo root: `ARCHITECTURE.md`)

## 1. What We Are Building

GeoRestore AI is a **research-grade, production-oriented platform** for
satellite image restoration and Earth observation. The flagship module is
**CloudClear-L4**: generative AI-based cloud removal and surface
reconstruction for LISS-IV satellite imagery (~5.8 m, G/R/NIR bands).

It is a platform, not a notebook: modular code, versioned data, tracked
experiments, an API, a GIS web interface, and a living knowledge base.

## 2. Core Research Question

> How effectively can generative and multimodal models reconstruct
> cloud-obscured high-resolution LISS-IV imagery while preserving spatial
> structure, spectral fidelity, temporal consistency, and downstream
> geospatial utility?

Decomposed, measurable sub-questions:

| ID | Question | Evidence |
|---|---|---|
| Q1 | How close is the reconstruction inside cloud regions, per band, per cloud severity? | mask-aware PSNR/SSIM/SAM per band, stratified |
| Q2 | Is spectral fidelity preserved? | band statistics, SAM, NDVI preservation |
| Q3 | Does restoration help or hurt downstream EO use? | segmentation/classification before vs after |
| Q4 | What does SAR/temporal data add over optical-only? | fusion ablations (E006, E007) |
| Q5 | How trustworthy are the uncertainty estimates? | calibration of MC-dropout vs actual errors |

## 3. Design Principles

Priority order (hard constraint):

```
correctness > reproducibility > scientific validity > maintainability > scalability > visual polish
```

Consequences:

1. Geospatial correctness first: CRS/geotransform validation on every I/O,
   no silent resampling, provenance on every artifact.
2. Reproducibility: config hash + dataset version + seed + model version on
   every run; immutable dataset versions.
3. Scientific validity: leakage-free splits, mask-aware metrics, dual
   evaluation tracks, statistical rigor.
4. Maintainability: small typed modules, one plugin interface, config-driven
   everything.
5. Scalability: async jobs, windowed raster I/O, patch parallelism — designed
   in, optimized later.
6. Visual polish last.

## 4. Scope

In scope (initial phases):

- LISS-IV ingestion (Bhoonidhi), Sentinel-1 SAR and Sentinel-2 auxiliary.
- Geospatial processing: CRS validation, reprojection, resampling, band
  alignment, normalization, cloud masking, tiling, splits, synthetic clouds.
- Cloud restoration progression: classical → U-Net → cGAN → SAR-optical
  fusion → experimental diffusion.
- Remote-sensing-specific quality evaluation (mask-aware, stratified,
  uncertainty-aware).
- Downstream validation (land-cover segmentation).
- Platform: FastAPI, async jobs, PostGIS, MLflow, Next.js GIS frontend.
- Restored GeoTIFF + cloud mask + uncertainty + observation-status +
  processing metadata + model version as outputs.

Out of scope (initially): see section 5 and ROADMAP.md.

## 5. Non-Goals (Phase 0–2)

- RAG / vector databases / embeddings for the knowledge base.
- Diffusion/flow models before cGAN and fusion are proven.
- Kubernetes, KServe/TorchServe, distributed training.
- Multi-tenant auth/OIDC/SSO.
- Full slippy-tile streaming (minimal in Phase 4).
- DVC / Airflow / Prefect orchestrators.
- Change detection endpoint (deferred).
- SOTA benchmark racing, model compression, quantization, mobile apps.

## 6. Success Criteria

Measurable exit criteria per phase (details in ROADMAP.md):

- **Phase 1 (MVP)**: end-to-end restore job on a real GeoTIFF; classical +
  U-Net baselines; mask-aware evaluation; Experiment E001 documented.
- **Phase 2 (Research)**: cGAN beats U-Net on cloud-region metrics with
  non-overlapping confidence intervals (or documented negative result);
  E001–E005 complete; uncertainty calibrated.
- **Phase 3 (Multimodal)**: fusion model ≥ optical-only on cloud-region
  metrics with statistical significance, or documented negative result.
- **Phase 4 (Production)**: deployment runbook, security review, load test,
  backup/restore verified.

Scientific credibility rule: **no SOTA claims; negative results are
publishable; every number carries provenance.**

## 7. Roles & Responsibilities (within sessions)

- **Data engineer**: ingestion, manifests, preprocessing, split integrity.
- **ML engineer**: models, losses, training, uncertainty.
- **Evaluation engineer**: metrics, protocols, downstream validation.
- **Backend engineer**: API, jobs, storage, PostGIS.
- **Frontend engineer**: Next.js GIS UI.
- **Research lead**: experiment matrix, interpretation, knowledge base.

Any single session may combine roles; the rules in `AGENTS.md` apply to all.

## 8. Constraints & Assumptions

- Python 3.11+, PyTorch 2.x, rasterio/GDAL, PostGIS, Redis, MLflow.
- Single GPU host assumed until Phase 4.
- Bhoonidhi license terms govern LISS-IV data; record in dataset manifest.
- Ground truth for real clouds is scarce or absent → dual-track evaluation.
