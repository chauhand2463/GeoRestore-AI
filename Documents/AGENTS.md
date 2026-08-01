# GeoRestore AI — Agent Instructions

## 1. Project Purpose

GeoRestore AI is an AI-based geospatial image restoration system that
reconstructs degraded or missing information in satellite imagery while
preserving spatial and spectral consistency. The flagship module is
CloudClear-L4: cloud removal and surface reconstruction for LISS-IV
satellite imagery (~5.8 m, G/R/NIR). It is a research-grade,
production-oriented platform, not a notebook demo.

Authoritative references:

- `docs/` — the project blueprint (14 documents). **Canonical source of
  architecture.**
- `knowledge/` — Obsidian vault (research, experiments, decisions,
  problems).
- `knowledge/00_Project/Architecture_Plan.md` — approved architecture plan
  (standalone copy at repo root `ARCHITECTURE.md`).
- `knowledge/05_Engineering/Blueprint/` — mirror of `docs/` for vault
  browsing (`docs/` is canonical; mirrors are refreshed from it, never
  edited directly).

Before any major task, check the relevant doc under `docs/` and the
relevant notes under `knowledge/`.

## 2. Architecture Rules

- Patches are the unit of learning; **scenes are the unit of integrity**
  (splits, stats, versioning are scene-level).
- All models implement the single plugin interface `RestorationModel`
  (`src/models/interface.py`). The pipeline never imports concrete model
  classes directly.
- Clear pixels are preserved by default in restored output; only masked
  regions are replaced.
- Every restore output = restored raster + cloud mask + uncertainty +
  observation-status layer + processing metadata.
- Config-driven everything: no experiment, job, or inference run without a
  resolved config hash.
- Two evaluation tracks always: paired (synthetic-cloud benchmark,
  mask-aware) and unpaired (real clouds, no-reference).
- No RAG / vector DB / embeddings. No Kubernetes. No distributed training
  (initial phases). No new heavyweight technology without a decision note
  in `knowledge/06_Decisions/`.
- Data flow is one-way: `data/raw → data/processed → data/patches →
  training/evaluation`. Never write into `data/raw/`.

## 3. ML Engineering Rules

- Model progression only: M0 classical → M1 U-Net → M2 conditional GAN →
  M3 SAR-optical fusion → M4 temporal → M5 diffusion (experimental). Do not
  jump to complex models before their gates (see `docs/MODEL_ROADMAP.md`).
- Mask-conditioning everywhere: the cloud mask is both an input channel and
  a loss-weight map.
- Composite loss from registered parts (masked L1, perceptual, adversarial,
  SAM spectral) with config weights.
- Deterministic: seed torch/numpy/python/albumentations/synthetic clouds;
  record seed + GPU/driver/CUDA in run metadata.
- Checkpoint by best val metric; record best-by-PSNR and best-by-SAM
  separately. Test set is used once per dataset version.
- Uncertainty (MC-dropout baseline) is a first-class output; calibration
  report required before a model is a production candidate.
- Train on synthetic + real data; keep a real-cloud holdout that never
  enters training.

## 4. Geospatial Rules

- Validate CRS + geotransform + band count on every raster open (rasterio).
- Never silently reproject or resample: the target grid must be explicit
  and recorded.
- Outputs are Cloud-Optimized GeoTIFFs (tiled, overviews, compression,
  BIGTIFF when >4 GB) unless otherwise specified.
- Band order and scaling (DN → reflectance) come from the dataset manifest;
  never assumed.
- Windowed raster I/O only; never materialize full rasters in memory.
- Cloud masks are uint8 (0=clear, 1=cloud, 2=shadow); mask algorithm and
  params stored with the mask.
- Normalization stats are computed from the train split only and frozen per
  dataset version.
- Footprints in EPSG:4326; rasters keep native CRS.

## 5. Testing Requirements

- Before every commit/PR: `ruff`, `mypy`, `pytest` (all run in CI on push).
- Mandatory test classes:
  - geo round-trip (CRS/geotransform preserved, band count)
  - manifest hashing and immutability checks
  - split leakage checks (no footprint overlap across splits)
  - evaluation-library correctness on tiny fixtures
  - API validation (bad uploads rejected, job lifecycle)
- A change touching a pipeline must add or update a test in the same
  change. No test, no merge.

## 6. Reproducibility Requirements

- Every run records: `config_sha`, `dataset_version`, `model_version`,
  `seed`, code commit, library versions (lockfile), GPU/driver.
- Dataset versions are immutable (append-only); any change → new version.
- Every `Experiment_NNN.md` cites `mlflow_run_id` + the four
  reproducibility keys; an experiment note without them is invalid.
- Lockfile committed; no unpinned dependencies.
- Checkpoints live in `models/`/MLflow artifacts — never in `knowledge/`.

## 7. Data Handling Rules

- `data/raw/` is immutable input; `processed/` and `patches/` are derived;
  all three are outside git (see `.gitignore`).
- No large files in `knowledge/` or git.
- Manifests (JSON) are the single source of truth for dataset content,
  splits, stats, and recipes.
- SHA-256 recorded for every stored artifact (manifests + DB).
- Licensing: record source + license in the manifest; respect Bhoonidhi
  terms.

## 8. Naming Conventions

- Python: snake_case modules, PascalCase classes; packages under `src/`.
- Configs: `<domain>_<purpose>.yaml` — `dataset_*`, `model_*`, `train_*`,
  `eval_*` under `configs/`.
- Dataset versions: `ds_vNNN` (e.g., `ds_v001`). Models: `<name>-<major>.<minor>`.
- Experiments: `knowledge/05_Experiments/Experiment_<NNN>.md` (leading zeros).
- Model IDs M0–M5; experiment IDs E001–E016 (see `docs/EXPERIMENT_MATRIX.md`).
- Artifact files: `{job_id}_{kind}.tif` under `outputs/` (predictions/,
  geotiff/, visualizations/).
- DB ids are uuids; tables: `users, scenes, cloud_masks, patches,
  dataset_versions, models, evaluations, jobs, job_outputs`.
- Patch default: 256×256, stride 192; split 70/15/15; channels
  `[G, R, NIR, cloud_mask]`, SAR adds `[VV, VH]`.

## 9. Dependency Rules

- Minimal, justified dependencies; each new one recorded in
  `docs/DEPLOYMENT.md` or a decision note.
- Dependency groups in pyproject: `geo`, `ml`, `api`, `db`, `dev`.
- Use GDAL/rasterio for geospatial work — do not reimplement in pure Python.
- No new framework without a recorded architecture decision.

## 10. Rules Against Unnecessary Abstractions

- One plugin abstraction only: `RestorationModel`. No DI frameworks, no
  meta-engines, no generic "pipeline" classes.
- Prefer plain functions + small dataclasses; add classes only for real
  state or polymorphism.
- No speculative generality: build for the known roadmap (M0–M5, the 20
  tasks in `docs/ROADMAP.md`), not hypothetical needs.
- Configs are YAML + pydantic models; no DSLs, no code generation.
- Single source of truth: shared constants (patch size, stride, split
  ratio, endpoints) live in configs, not scattered literals.

## 11. Rules Against Fake Scientific Claims

- Never claim state-of-the-art without a controlled, comparable evaluation.
- Always distinguish synthetic-cloud results from real-cloud results;
  report both separately.
- Report negative results; failed experiments are project knowledge
  (`knowledge/07_Problems/`).
- Metrics are reported per scene with confidence intervals; no
  cherry-picked scenes, no RGB-only metrics (per-band reporting is
  mandatory — RGB SSIM hides NIR errors).
- Never present model output as measurement: reconstructed pixels are
  labeled as generated and carry uncertainty.
- Every claim cites `config_sha` + `dataset_version` + `seed`.
- No "better" without non-overlapping CIs or a paired test at p<0.05.

## 12. Knowledge Base (vault) Rules

The knowledge base is an Obsidian vault at `knowledge/`:

- **Before making decisions** on model architecture, dataset strategy,
  preprocessing, losses, evaluation metrics, or geospatial processing,
  check the relevant notes under `knowledge/`.
- **Architecture decisions** are recorded in `knowledge/06_Decisions/`.
  Do not silently replace an existing decision; if a better approach is
  found: explain the reason, record the new decision, record what was
  replaced, and record expected trade-offs.
- **Experiments**: every meaningful experiment documented in
  `knowledge/05_Experiments/Experiment_<NNN>.md` with Objective,
  Hypothesis, Dataset, Preprocessing, Model, Configuration, Training
  setup, Metrics, Results, Observations, Conclusion, Next action.
- **Failed experiments are never deleted**; document in
  `knowledge/07_Problems/`.
- **Research**: when using a paper, record the paper, its contribution,
  method, limitations, and relation to GeoRestore AI
  (`knowledge/01_Research/`).
- **Data and outputs**: never store large datasets, checkpoints, or
  generated images in the vault. Store descriptions/metadata only; actual
  data goes in `data/`, outputs in `outputs/`, models in `models/`.
- **Conventions**: long-term knowledge lives in `knowledge/` as Markdown,
  not code comments. Notes use Obsidian wikilinks (`[[Note_Name]]`); link
  targets must match note filenames exactly. `docs/` is mirrored to
  `knowledge/05_Engineering/Blueprint/` — edit `docs/`, refresh the mirror.
