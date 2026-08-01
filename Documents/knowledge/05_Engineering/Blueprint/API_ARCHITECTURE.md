# API Architecture — GeoRestore AI

Status: approved blueprint component
Parent: [[Architecture_Plan]]

## 1. Stack

FastAPI + uvicorn + pydantic v2, Redis + arq jobs, PostGIS, artifact store
(`outputs/`). Versioned namespace `/api/v1`. OpenAPI docs auto-generated.

## 2. Endpoints (v1)

| Method | Path | Purpose | Returns |
|---|---|---|---|
| GET | `/api/v1/health` | liveness + versions | `{status, services}` |
| POST | `/api/v1/cloud-detect` | cloud mask job | `{job_id}` |
| POST | `/api/v1/restore` | restore job | `{job_id}` |
| POST | `/api/v1/evaluate` | paired or no-reference eval | `{job_id}` |
| POST | `/api/v1/change-detect` | (deferred, Phase 3 optional) | `{job_id}` |
| GET | `/api/v1/models` | registry listing | `[{id, name, version, status, metrics}]` |
| GET | `/api/v1/jobs/{job_id}` | status + progress + artifact URIs | job record |
| GET | `/api/v1/jobs/{job_id}/artifacts/{kind}` | download artifact | file |

Small-patch debug endpoints (synchronous) allowed for UI experimentation.

## 3. Job Lifecycle

```
queued → running → succeeded | failed | cancelled
```

- `POST` validates input (pydantic), creates job row, enqueues; returns 201
  with `job_id`.
- Worker (arq) executes; updates progress + status; stores artifact URIs and
  error (sanitized traceback) on failure.
- Retries: idempotent (params keyed by job_id); timeout per job type.
- Polling: UI polls `GET /api/v1/jobs/{id}` (TanStack Query, backoff).

## 4. Example Payloads

### POST /api/v1/restore

```json
{
  "scene_id": "s001",
  "model": {"name": "m2_cgan", "version": "1.0"},
  "mask": {"algorithm": "v1_threshold", "params": {}},
  "options": {"preserve_clear_pixels": true, "output_formats": ["cog"]}
}
```

### Response (201)

```json
{"job_id": "9f3a...", "status": "queued", "poll": "/api/v1/jobs/9f3a..."}
```

### GET /api/v1/jobs/{id} (succeeded)

```json
{
  "job_id": "9f3a...", "type": "restore", "status": "succeeded",
  "progress": 1.0, "created_at": "...", "finished_at": "...",
  "artifacts": {
    "restored": "/api/v1/jobs/9f3a.../artifacts/restored",
    "cloud_mask": ".../artifacts/cloud_mask",
    "uncertainty": ".../artifacts/uncertainty",
    "status_layer": ".../artifacts/status_layer",
    "metrics": ".../artifacts/metrics",
    "metadata": ".../artifacts/metadata"
  }
}
```

## 5. Errors

- `400` validation (pydantic), `401/403` auth, `404` unknown job/model,
  `409` job conflict, `413` upload too large, `500` internal.
- Error body: `{"error": {"code": "...", "message": "...", "job_id": "..."}}`.

## 6. Upload Rules

- GeoTIFF: extension + TIFF magic + GDAL-open probe; size limits (config);
  sanitized filenames; storage paths derived from DB ids (never user input).
- Chunked upload deferred to Phase 4.

## 7. Auth & Limits (Phase 1 minimal, Phase 4 full)

- Phase 1: API keys + rate limiting (per key).
- Phase 4: JWT/OIDC optional; job ownership checks; audit trail.
- CORS locked to web origin; no direct exposure of `outputs/` static.

## 8. Conventions

- All schemas pydantic v2; OpenAPI docs; semantic versioning of the API.
- Structured JSON logs with `request_id` and `job_id`.
- DB writes via app role only (least privilege).

See DEPLOYMENT.md for runtime topology.
