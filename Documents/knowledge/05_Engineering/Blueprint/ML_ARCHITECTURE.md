# ML Architecture — GeoRestore AI

Status: approved blueprint component
Parent: [[Architecture_Plan]]

## 1. Training Pipeline Stages

```
dataset version → loaders → model build → composite loss → train loop → validation → logging → registration
```

1. **Data**: loaders from manifests; fixed augmentation (rotation/flip/scale
   via albumentations with mask sync; no elastic distortions that break
   geospatial correspondence).
2. **Model**: built from registry via `configs/model/*.yaml`; init recorded.
3. **Loss**: registered components with weights from config:

| Loss | Role | Notes |
|---|---|---|
| `L_rec` | masked L1/L2 | mask-weighted; cloud regions weighted higher |
| `L_perc` | perceptual | LPIPS or VGG features |
| `L_adv` | adversarial | PatchGAN / hinge |
| `L_spec` | spectral | SAM (spectral angle) across bands |
| `L_edge` | structure (optional) | gradient-domain |

4. **Train loop**: deterministic seeding; AMP optional; grad clipping; EMA
   optional; checkpoint best-by-PSNR and best-by-SAM separately.
5. **Validation**: mask-aware metrics on val scenes only.
6. **Logging**: MLflow run = params (config hash), metrics (per epoch, per
   band, mask-aware), artifacts (checkpoint, config, val visualizations).
7. **Registration**: model + config + dataset version + metrics → registry.

## 2. Config-Driven Contract

Every run resolves exactly one effective config:

```
dataset_version: ds_v001
model: stgan_cr
train:
  patch_size: 256
  batch_size: 16
  epochs: 100
  lr: 0.0002
  seed: 42
loss_weights: {rec: 10, perc: 1, adv: 0.5, spec: 5}
eval: {protocol_version: ev_v1, mask_aware: true}
```

`config_sha = sha256(canonical_json(config))` — the run's identity.

No experiment without a config hash.

## 3. Determinism & Reproducibility

- Seed torch, numpy, python, albumentations, and the synthetic-cloud
  generator (per version).
- Record: GPU model, CUDA version, lockfile, commit, config_sha,
  dataset_version, model_version, seed — all in MLflow tags and the
  Experiment note.
- CUDA nondeterminism: document if observed; never silently accepted.

## 4. Uncertainty

- Baseline: MC-dropout (T forward passes, mean = prediction, variance =
  uncertainty).
- Optional: heteroscedastic head (later).
- Calibration: compare predicted uncertainty vs actual error on synthetic
  pairs; reliability diagrams; report correlation.
- Uncertainty is a first-class output (raster + UI layer); never stripped.

## 5. Model Registry

- MLflow registered models keyed by `(name, version)`.
- Registry metadata: config_sha, dataset_version, metrics, status
  (staging/production), artifact URI.
- Deployment loads only registered artifacts.

## 6. Discipline Rules

- Val is for selection; test runs once per dataset version.
- Clear pixels preserved by default; mask-weighted losses.
- Report best-by-PSNR and best-by-SAM; selection criterion stated in config.
- Real-cloud holdout never enters training.

## 7. Hardware Assumptions

- Phase 1–3: single GPU (≥8 GB VRAM recommended; batch/patch adjusted).
- Phase 4: GPU workers scale horizontally behind the queue.
- CPU fallback for classical baselines and geo jobs.

## 8. What ML Architecture Is NOT

- Not a DSL: configs are YAML validated by pydantic.
- Not a distributed framework: no DeepSpeed in initial phases.
- Not a meta-framework: one plugin interface only (see SYSTEM_ARCHITECTURE).
