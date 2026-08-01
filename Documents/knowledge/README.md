# Knowledge Base — GeoRestore AI

This folder is an Obsidian vault: open this directory (`knowledge/`) as a
vault in Obsidian and every note becomes linkable, searchable, and graphable.

## Layout

| Folder | Contents |
|---|---|
| `00_Project/` | Overview, objectives, requirements, roadmap, [[Architecture_Plan]] |
| `01_Research/` | Papers, literature review, existing methods, research gaps |
| `02_Remote_Sensing/` | Sensors, spectral bands, georeferencing, GeoTIFF |
| `03_Dataset/` | Dataset notes, collection, preprocessing, cloud masking, patches |
| `04_Model/` | Architecture, generator, discriminator, losses, GAN/diffusion, spectral consistency |
| `05_Experiments/` | Experiment logs: `Experiment_<number>.md` |
| `05_Engineering/` | Tooling, infrastructure, deployment; `Blueprint/` mirrors `docs/` |
| `06_Decisions/` | Recorded architectural/dataset/technology decisions |
| `07_Problems/` | Bugs, failed experiments, solutions |
| `08_Evaluation/` | Metrics (PSNR, SSIM, SAM, spectral) and evaluation protocol |

The project blueprint lives in `docs/` (canonical) and is mirrored to
`05_Engineering/Blueprint/` — edit `docs/`, refresh the mirror.

## Rules

- Experiments go in `05_Experiments/Experiment_<number>.md` with: objective,
  hypothesis, dataset, preprocessing, model, configuration, training setup,
  metrics, results, observations, conclusion, next action.
- Decisions are recorded in `06_Decisions/` before/while being implemented.
- Failed experiments are never deleted; they belong in `07_Problems/`.
- Notes are interconnected with Obsidian wikilinks; link targets must match note
  filenames exactly.
- Long-term knowledge lives here as Markdown — not in code comments.
- No raw imagery, GeoTIFFs, checkpoints, or generated images in the vault;
  those go in `data/`, `outputs/`, `models/`.
