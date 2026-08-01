# Experiment Matrix — GeoRestore AI

Status: approved blueprint component
Parent: [[Research_Plan]]

## 1. Matrix (E001–E016)

| ID | Experiment | Objective / Hypothesis | Models | Track | Metrics | Depends on | Gate / Decision |
|---|---|---|---|---|---|---|---|
| E001 | Baseline sweep | Classical floor vs U-Net | M0, M1 | paired | PSNR/SSIM/SAM mask-aware | ds_v001, eval lib | anchors all claims |
| E002 | cGAN vs U-Net | Adversarial training helps LISS-IV | M1, M2 | paired | full set | E001 | M2 promoted if CI-separated |
| E003 | Loss ablation | Contribution of L_perc / L_spec / L_adv | M2 variants | paired | full set | E002 | final loss weights |
| E004 | Cloud severity | Thin/moderate/thick behavior | M1, M2 | paired+stratified | by severity | E002 | documented limits |
| E005 | Cloud fraction | 0–10/10–30/30–60/60–100% behavior | M1, M2 | paired+stratified | by bin | E002 | documented limits |
| E006 | SAR fusion ablation | Value of VV/VH | M2, M3 | paired | full set | coreg QC, M3 | M3 promoted if better |
| E007 | Temporal ablation | Value of temporal clear frames | M2, M4 | paired | full set | temporal corpus | M4 decision |
| E008 | Diffusion experiment | Feasibility vs cGAN (research-only) | M5 | paired | full set + sampling | M2 stable | research record |
| E009 | Uncertainty calibration | MC-dropout calibration quality | M1, M2 | paired/unpaired | calibration | eval lib | release gate input |
| E010 | Downstream utility | Segmentation before/after restoration | M2 best | downstream | task accuracy | downstream tool | Q3 answer |
| E011 | Real-cloud validation | No-reference suite on real clouds | M1, M2, M3 | unpaired | no-ref suite | real holdout | Q-real answer |
| E012 | Spectral fidelity | Band-wise + NDVI preservation | M1, M2 | paired | band stats, NDVI | E002 | spectral claim |
| E013 | Geographic generalization | Held-out region/season | M2 | paired+unpaired | by region | ds with regions | generalization claim |
| E014 | Dataset version delta | Effect of dataset improvements | M2 | paired | full set | ds_v002 | dataset decisions |
| E015 | Mask sensitivity | Impact of mask errors on output | M2 | paired | full set vs mask corruption | E002 | robustness claim |
| E016 | Synthetic→real gap | Quantify domain gap | M2, M3 | mixed | E011 + E004 | E011 | research record |

## 2. Priority & Sequencing

- **Core path**: E001 → E002 → E003 → E004 → E005 (research prototype).
- **Trust**: E009 (alongside Phase 2).
- **Multimodal claims**: E006, E007 (Phase 3).
- **Utility claim**: E010 (Phase 2 end / Phase 3).
- **Exploratory**: E008, E016 (research-only).
- **Dataset-driven**: E014 (whenever a new dataset version lands).

## 3. Run Bookkeeping (mandatory per run)

- config_sha (canonical config hash)
- dataset_version (ds_vNNN)
- model_version (name-major.minor)
- seed
- mlflow_run_id
- protocol_version (ev_v1)

An experiment note missing these keys is not a valid record.

## 4. What Constitutes a Result

- Metrics table per EVALUATION.md §10 template (mask-aware regions, per-band
  appendix, CI).
- Visual QC samples (fixed set per experiment, stored in outputs/
  visualizations).
- Interpretation + next action in the Experiment note.

## 5. Negative Results

Any experiment that fails to show improvement is documented, not deleted:
`knowledge/07_Problems/Failed_Experiments.md` + the experiment note's
conclusion section. Matrix status column tracks them.
