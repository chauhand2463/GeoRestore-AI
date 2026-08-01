# Evaluation — GeoRestore AI

Status: approved blueprint component
Parent: [[Architecture_Plan]]

## 1. Two Tracks

| Track | Ground truth | Purpose | Headline metrics |
|---|---|---|---|
| Paired (synthetic clouds / temporal GT) | available | benchmark, ablations, model selection | mask-aware PSNR/SSIM/SAM per band |
| Unpaired (real clouds) | none | real-world trust | no-reference suite + uncertainty + downstream |

**Generated pixels are never ground truth.** Every output ships with
mask + uncertainty + observation-status layer.

## 2. Metrics (paired track)

Computed per band AND mean:

- PSNR (dB, higher better)
- SSIM (0–1, higher better)
- RMSE
- MAE
- SAM (degrees, lower better)
- LPIPS (optional module, lower better)

Per-band reporting is mandatory — RGB-collapsed SSIM hides NIR errors that
drive NDVI (see AGENTS.md fake-claims rules).

## 3. Mask-Aware Regions

Metrics reported separately on:

1. Clear region (outside cloud mask)
2. **Cloud region** (inside mask) — headline number
3. Boundary band (8 px erosion/dilation of mask edge)

## 4. Stratification

- By cloud fraction bin: 0–10 / 10–30 / 30–60 / 60–100%.
- By severity class (synthetic): thin / moderate / thick.
- By region/season when the corpus allows (E013).

## 5. Statistical Rigor

- Aggregate per **scene** (never per patch).
- Report mean ± bootstrap 95% CI.
- Model comparison: paired tests (Wilcoxon signed-rank) on per-scene
  metrics; "better" requires non-overlapping CI or a positive paired test.
- Test set is used once per dataset version.

## 6. Unpaired Track (no-reference suite)

- Band histogram / distribution distance (reconstructed vs clear context).
- Local gradient coherence across the mask boundary.
- Spectral angle computed on clear pixels only; statistics continuity.
- Texture statistics sanity (no over-smoothing).
- Observation-status consistency (clear pixels unchanged by default).

## 7. Uncertainty Calibration

- Errors (from synthetic pairs) vs predicted uncertainty: correlation,
  reliability diagrams, coverage of confidence intervals.
- Reported per model; calibration is a release gate for production
  candidates.

## 8. Downstream Validation

- Fixed downstream model (land-cover segmentation, pretrained or trained
  on a frozen split) run on: cloudy input → restored output → (clear
  reference if available).
- Report accuracy delta; never claim "restoration improves X" without this
  measurement. E010 owns this protocol.

## 9. Protocol Versioning

- `protocol_version` (e.g., `ev_v1`) increments on any metric/decision
  change; recorded per evaluation.
- Results are comparable only within a protocol version.

## 10. Reporting Template (per experiment)

```markdown
## Metrics (protocol ev_v1, dataset ds_v001)
| Region | PSNR | SSIM | RMSE | MAE | SAM |
| Clear | .. | .. | .. | .. | .. |
| Cloud | .. | .. | .. | .. | .. |
| Boundary | .. | .. | .. | .. | .. |
Per-band appendix: table per band with the same regions.
Bootstrap CI: ±.. (95%, per scene). Paired test vs baseline: p=..
```

## 11. Forbidden Practices

- Cherry-picking scenes.
- Reporting synthetic numbers as real-world performance.
- RGB-only metrics.
- Dropping uncertainty from deliverables.
- Comparing runs with different dataset versions or protocol versions as
  if equal.
