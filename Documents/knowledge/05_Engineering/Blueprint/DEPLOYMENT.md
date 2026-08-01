# Deployment — GeoRestore AI

Status: approved blueprint component
Parent: [[Architecture_Plan]]

## 1. Environments

| Env | Purpose | Form |
|---|---|---|
| dev | local development | docker compose (full stack) or bare process |
| research | single GPU host | docker compose + training CLI |
| production | Phase 4 | compose, hardened; no K8s |

## 2. Dev Stack (docker-compose.yml)

| Service | Image/Base | Ports | Notes |
|---|---|---|---|
| api | python:3.11 | 8000 | FastAPI/uvicorn |
| worker | python:3.11 | — | arq worker; GPU passthrough for inference |
| redis | redis:7 | 6379 | queue + optional preview cache |
| postgis | postgis/postgis:16-3.4 | 5432 | app schema + migrations |
| mlflow | python:3.11 | 5000 | tracking + registry |
| web | node:20 | 3000 | Next.js |
| nginx | nginx:alpine | 80 | reverse proxy (optional in dev) |

Volumes: `data/`, `outputs/`, `models/`, `mlflow-artifacts/` persisted;
`postgres_data` volume.

## 3. Environment Variables (example set)

```
POSTGRES_URI=postgresql://app:***@postgis:5432/georestore
REDIS_URI=redis://redis:6379/0
MLFLOW_TRACKING_URI=http://mlflow:5000
DATA_ROOT=/data
OUTPUTS_ROOT=/outputs
MODELS_ROOT=/models
API_KEYS=...            # comma-separated (Phase 1)
RATE_LIMIT_PER_KEY=120  # per minute
MAX_UPLOAD_BYTES=2147483648
```

Secrets never committed; `.env.example` only.

## 4. Dependency Groups (pyproject)

| Group | Contents |
|---|---|
| geo | rasterio, GDAL (pygdal), numpy, scikit-image, opencv-headless |
| ml | torch, torchvision, albumentations, lpips (optional) |
| api | fastapi, uvicorn, pydantic, pydantic-settings, arq, redis |
| db | psycopg (async), geoalchemy2 (optional) |
| dev | pytest, ruff, mypy, pre-commit |

Lockfile committed; CI runs `pip-audit`/`trivy`.

## 5. CI Pipeline

1. ruff lint, mypy typecheck
2. pytest (unit + geo round-trip + leakage checks)
3. manifest verification on a small fixture dataset
4. build images (optional, Phase 4)

## 6. Health Checks & Observability

- `/api/v1/health` checks: DB ping, Redis ping, queue health.
- Structured JSON logs with `request_id`/`job_id`.
- Phase 4: Prometheus metrics + Grafana dashboards (optional).
- Job timeouts and retry budgets per type.

## 7. Backup & Restore

- PostGIS: nightly `pg_dump` (tested restore monthly).
- Artifacts: rsync/object-store copy of `outputs/`, `models/`, `data/`.
- Manifests are part of git history (JSON) — dataset metadata survives.

## 8. Deployment Runbook (Phase 4 gate)

1. `docker compose build && docker compose up -d`
2. Run migrations (`api` startup or separate command)
3. Smoke test: health, upload small patch, run restore, download artifact
4. Load test: ≥10 concurrent restore jobs
5. Security checklist (AGENTS.md + RISKS.md)
6. Backup verified; monitoring dashboards live

## 9. Scaling Path (documented, not built initially)

- More workers → horizontal inference scaling.
- Object storage for artifacts; tile cache in Redis.
- K8s only if deployment needs outgrow compose.
