# GeoRestore AI

AI-based geospatial image restoration: reconstructs degraded or missing
information in satellite imagery while preserving spatial and spectral
consistency.

## Structure

- `src/` — core source code
- `models/` — model definitions
- `training/` — training code
- `inference/` — inference code
- `preprocessing/` — preprocessing code
- `evaluation/` — evaluation code
- `configs/` — YAML/JSON configurations
- `tests/` — tests
- `data/` — raw/, processed/, patches/ (not committed)
- `outputs/` — predictions/, geotiff/, visualizations/
- `knowledge/` — project knowledge base (Obsidian vault)

## Documentation

- `docs/` — project blueprint (14 docs: architecture, data, ML, models,
  evaluation, API, frontend, deployment, research, experiments, risks,
  roadmap). Canonical source of architecture.
- `ARCHITECTURE.md` — approved architecture & implementation plan.
- `AGENTS.md` — agent-facing project rules (read this first).
- `knowledge/` — Obsidian vault; `knowledge/05_Engineering/Blueprint/`
  mirrors `docs/` for vault browsing.

## Knowledge Base

Open `knowledge/` as an Obsidian vault. It holds research, remote sensing
concepts, dataset notes, model architecture, experiments, decisions,
problems, and evaluation methodology. See `knowledge/README.md` for the vault
layout.

## Conventions

- All long-term knowledge lives in `knowledge/` as Markdown, not code comments.
- No large datasets/checkpoints in the vault; use `data/`, `outputs/`, `models/`.

## Setup

TODO: environment setup, installation, and usage instructions (see
`docs/DEPLOYMENT.md` for the runtime topology).
