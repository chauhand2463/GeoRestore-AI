# Blueprint — Project Documentation Mirror

This folder mirrors `docs/` (the canonical blueprint) into the vault for
browsing in Obsidian.

**Canonical location: `docs/` at the repo root. Edit there; refresh this
mirror afterward:**

```powershell
Copy-Item -Path 'docs/*.md' -Destination 'knowledge/05_Engineering/Blueprint/' -Force
```

## Index

Note: wikilinks below use vault paths to stay unambiguous; the only
basename that collides with another vault note is `ROADMAP`
(`docs/ROADMAP.md` ↔ `00_Project/Roadmap.md`).

| Doc | Covers |
|---|---|
| [[05_Engineering/Blueprint/PROJECT_VISION]] | mission, research question, principles, non-goals, success criteria |
| [[05_Engineering/Blueprint/SYSTEM_ARCHITECTURE]] | layered architecture, components, interfaces, dependency graph |
| [[05_Engineering/Blueprint/DATA_ARCHITECTURE]] | data catalog, layout, versioning, manifests, geospatial conventions |
| [[05_Engineering/Blueprint/ML_ARCHITECTURE]] | training pipeline, config contract, determinism, uncertainty, registry |
| [[05_Engineering/Blueprint/MODEL_ROADMAP]] | M0–M5 progression, gates, plugin interface |
| [[05_Engineering/Blueprint/DATASET_PIPELINE]] | ingestion → masks → patches → splits → synthetic clouds |
| [[05_Engineering/Blueprint/EVALUATION]] | mask-aware metrics, stratification, statistics, protocol versioning |
| [[05_Engineering/Blueprint/API_ARCHITECTURE]] | endpoints, jobs, schemas, errors, auth |
| [[05_Engineering/Blueprint/FRONTEND_ARCHITECTURE]] | Next.js GIS UI, layers, data flow |
| [[05_Engineering/Blueprint/DEPLOYMENT]] | compose stack, env vars, CI, backup, runbook |
| [[05_Engineering/Blueprint/RESEARCH_PLAN]] | research questions, evidence standards, knowledge workflow |
| [[05_Engineering/Blueprint/EXPERIMENT_MATRIX]] | E001–E016 matrix, bookkeeping keys, sequencing |
| [[05_Engineering/Blueprint/RISKS]] | risk registers (scientific, hallucination, leakage, engineering) |
| [[05_Engineering/Blueprint/ROADMAP]] | phases, DoD, milestones, first-20 tasks |

Related: [[Architecture_Plan]] (approved plan), [[Roadmap]] (vault roadmap).
