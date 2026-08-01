# Model Roadmap — GeoRestore AI

Status: approved blueprint component
Parent: [[Architecture_Plan]]

## 1. Principle

Progression, not jumps. Each model is a plugin behind the
`RestorationModel` interface (see SYSTEM_ARCHITECTURE.md §4.1). A model is
added only when its prerequisites (data, gates) exist.

## 2. Model Progression

| ID | Model | Input | Output | Loss | Uncertainty | Status | Gate to proceed |
|---|---|---|---|---|---|---|---|
| M0 | Classical inpainting (Telea/NS, GDAL fill) | G,R,NIR + mask | image | n/a | none | planned (Phase 1) | — |
| M1 | U-Net (ResNet-18/34 encoder, 4–5 levels) | G,R,NIR + mask | image | masked L1 + SSIM | MC-dropout | planned (Phase 1) | E001 baseline numbers |
| M2 | Conditional GAN (STGAN-CR-style generator + PatchGAN) | G,R,NIR + mask | image | L1 + LPIPS + adv + SAM | MC-dropout | planned (Phase 2) | beats M1 on cloud-region metrics (non-overlapping CI) or documented negative |
| M3 | SAR-optical fusion (dual encoder + cross-attention) | + VV, VH | image | same as M2 | MC-dropout | planned (Phase 3) | coregistration QC passed; beats M2 on cloud-region metrics or documented negative |
| M4 | Temporal (multi-date stack encoder) | + temporal clear frames | image | same as M2 | MC-dropout | planned (Phase 3) | temporal corpus exists |
| M5 | Diffusion / flow (experimental) | any | image | DDPM / rectified-flow | inherent samples | deferred (Phase 3, optional) | M2 stable; research-only |

## 3. Interface Contract

```python
class Prediction:
    values: np.ndarray        # (C, H, W) float32, inverse-normalized
    uncertainty: np.ndarray   # (H, W) float32
    meta: dict                # model, version, config_sha, timings

class RestorationModel(Protocol):
    name: str
    input_modality: str       # "optical" | "sar_optical" | "temporal"
    def predict(self, patch, cloud_mask, context: dict) -> Prediction: ...
```

Registration via config:

```yaml
# configs/model/stgan_cr.yaml
model:
  id: m2_cgan
  class: models.cgan.STGANCR
  input_modality: optical
  patch_size: 256
  generator: {base_channels: 64, depth: 4}
  discriminator: {type: patchgan, n_layers: 3}
  uncertainty: {method: mc_dropout, passes: 16, dropout_rate: 0.1}
```

## 4. Shared Modeling Decisions

- Mask is an input channel **and** a loss-weight map.
- Clear pixels preserved by default in restored output (identity path);
  "restore-all" mode only for experiments.
- Per-band z-score normalization (train-only stats), inverse on output.
- All bands processed jointly; per-band metrics mandatory.
- Downstream of every model: mask-aware evaluation (EVALUATION.md).

## 5. Decision Gates

1. M2 enters training only after M1 results exist (E001) — baselines anchor
   claims.
2. M3 enters training only after SAR↔optical coregistration quality gate
   passes (residual shift within tolerance on a QC set).
3. M5 (diffusion) is research-only until M2/M3 are registered; never a
   default pipeline model.
4. A model is promoted to "production candidate" only with: registered
   artifact, config_sha, dataset_version, seeds, documented eval report.

## 6. What Is NOT a Model in This Roadmap

- Cloud masker: separate component (`src/masking`), versioned independently.
- Synthetic cloud generator: data component, not a model.
- Evaluation: pipeline component, not a model.
