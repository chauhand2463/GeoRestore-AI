# Research Plan — GeoRestore AI

Status: approved blueprint component
Parent: [[Architecture_Plan]]

## 1. Research Questions

| ID | Question | Experiments | Track |
|---|---|---|---|
| Q1 | Reconstruction quality in cloud regions, per band, per severity | E001–E005 | paired |
| Q2 | Spectral fidelity | E002, E003, E012 | paired |
| Q3 | Downstream utility | E010 | downstream |
| Q4 | Value of SAR / temporal data | E006, E007 | paired |
| Q5 | Uncertainty trustworthiness | E009 | paired/unpaired |

## 2. Methodological Commitments

1. **Dual tracks** — synthetic paired benchmark (controlled, GT-known) and
   real-cloud evaluation (no-reference). Both reported; never conflated.
2. **Baselines first** — classical (M0) and U-Net (M1) anchor every later
   claim.
3. **Leakage-free** — scene-level splits, buffer checks, train-only stats
   (see DATASET_PIPELINE.md).
4. **Statistical rigor** — per-scene aggregation, bootstrap CI, paired
   tests; "better" defined operationally (EVALUATION.md §5).
5. **Negative results are knowledge** — documented in `knowledge/07_Problems/`
   and in experiment notes; never deleted.
6. **No SOTA claims** — claims are always relative to our own controlled
   baselines.

## 3. Evidence Standards

| Claim type | Minimum evidence |
|---|---|
| Model A beats model B | non-overlapping CIs or paired test (p<0.05), same dataset version + protocol |
| Restoration helps downstream | measured downstream delta (E010 protocol) |
| Fusion helps | fusion ablation vs optical-only on same data |
| Uncertainty is trustworthy | calibration report (reliability, coverage) |
| Real-world performance | unpaired track + documented domain-gap analysis (E016) |

## 4. Knowledge Workflow

Every experiment produces:

1. Config + dataset version + seeds (reproducibility keys).
2. MLflow run (machine-readable).
3. `knowledge/05_Experiments/Experiment_NNN.md` (AGENTS.md template) with
   run_id + keys + interpretation.
4. Decision: continue / adjust / stop — recorded in the note's "Next action".

Papers are recorded in `knowledge/01_Research/` per AGENTS.md research rules
(paper, contribution, method, limitations, relation to GeoRestore).

## 5. Decision Gates

- **Phase gate** (section: see ROADMAP.md): all DoD items must pass.
- **Model gate** (MODEL_ROADMAP.md §5): baselines before GAN; coregistration
  QC before fusion; diffusion research-only.
- **Release gate** (Phase 4): calibration + downstream + security checklist.

## 6. Publication & Ethics

- Open data policy: manifests + code public; raw data per license.
- Reconstructed outputs labeled as model-generated; no misuse in
  decision-critical contexts without uncertainty disclosure.
- Record ethics note in `knowledge/00_Project/` if scope expands.

## 7. Research vs Engineering Split

| Research | Engineering |
|---|---|
| model selection, losses, SAM loss weighting | config system, API, jobs, PostGIS |
| synthetic cloud realism | ingestion, manifests, tiling |
| evaluation protocol, uncertainty calibration | MLflow wiring, CI |
| fusion/temporal/diffusion studies | deployment, backups |

Both roles share the same repo and rules; the classification guides effort,
not hierarchy.
