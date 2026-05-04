---
tags:
  - project/active
  - type/architecture
created: 2026-05-04
updated: 2026-05-04
---

# Architecture — compose-orientation

## System Map

```
┌─────────────────────────────────────────────────────────────────┐
│                      USER'S BROWSER                             │
│                                                                 │
│   v2-frontend (Agora)  ─── Next.js 15, React 18, MUI 6         │
│   Port 3000 (dev) / Azure Container Apps (prod)                 │
│                                                                 │
│   BFF API Routes:                                               │
│     /api/auth/*        → Auth0 (external)                       │
│     /api/hasura        → Hasura GraphQL (external)              │
│     /api/logos/*       → Flask API (Logos backend)              │
└────────────────────┬──────────────────────────────┬────────────┘
                     │                              │
                     │ GraphQL                      │ REST (SSE, upload, export)
                     ▼                              ▼
           ┌─────────────────┐          ┌───────────────────────┐
           │  Hasura GraphQL │          │   "Logos" Flask API   │
           │  (external svc) │          │   port 5050           │
           │                 │          │                       │
           │  Tables:        │          │  EITHER:              │
           │  users          │          │  socratics-v2-proto/  │
           │  projects       │          │  backend/app.py       │
           │  documents      │          │                       │
           │  fm_components  │          │  OR (production):     │
           │  builds         │          │  v2-backend/          │
           │  line_items     │          │  services/api/app.py  │
           │  taxonomy       │          └──────────┬────────────┘
           │  periods        │                     │
           └─────────────────┘                     │ enqueues jobs
                                                   ▼
                                     ┌─────────────────────────┐
                                     │  Azure Service Bus      │
                                     │                         │
                                     │  compute-jobs topic     │
                                     │    └ compute-workers    │
                                     │  ingest-jobs topic      │
                                     │    └ ingest-workers     │
                                     └──────────┬──────────────┘
                                                │ consumes
                              ┌─────────────────┴────────────────┐
                              │                                   │
                              ▼                                   ▼
                   ┌──────────────────┐              ┌───────────────────┐
                   │  Compute Worker  │              │   Ingest Worker   │
                   │  (v2-backend)    │              │   (v2-backend)    │
                   │                  │              │                   │
                   │  Kinds:          │              │  Pipeline:        │
                   │  - entity_resolve│              │  1. Download blob │
                   │  - assign_children              │  2. Extract XLSX  │
                   │  - reclassify   │              │  3. Classify      │
                   │  - export_xlsx  │              │  4. POST BFF      │
                   └────────┬─────────┘              └────────┬──────────┘
                            │                                 │
                            └──────────────┬──────────────────┘
                                           │ reads/writes via
                                           ▼
                              ┌─────────────────────────┐
                              │   Azure CosmosDB        │
                              │   socratics-documents   │
                              │                         │
                              │   financial_models      │
                              │   document_versions     │
                              │   audit_events          │
                              │   jobs                  │
                              └─────────────────────────┘
                              
                              ┌─────────────────────────┐
                              │   Azure Blob Storage    │
                              │   socratics-sources     │
                              │   XLSX uploads          │
                              └─────────────────────────┘
                              
                              ┌─────────────────────────┐
                              │   Redis                 │
                              │   DB 1: rate limits     │
                              │   DB 2: quota rollups   │
                              └─────────────────────────┘
                              
                              ┌─────────────────────────┐
                              │   Event Hubs /          │
                              │   Log Analytics         │
                              │   UsageEvents (billing) │
                              └─────────────────────────┘
```

---

## The Three Repos and Their Roles

### 1. socratics-v2-prototype
**Role:** Self-contained MVP prototype, demo deployments, Vercel preview builds.

**What it is:** A monorepo containing both a Flask backend and a Next.js 15 frontend. Fully independent — no code dependency on v2-backend or v2-frontend. This is the proof-of-concept that defined the product workflow; it still runs as a demo and Vercel preview environment.

**Backend** (`backend/`):
- Flask 3.1.3 + Python 3.12
- Port 5050
- State: JSON files in `backend/data/` (ephemeral, gitignored)
- LLM: Anthropic SDK, `claude-haiku-4-5-20251001`
- Deployment: Azure Container Apps (`demos` resource group, `socraticsprodacr.azurecr.io`)
- CI: `.github/workflows/deploy-backend.yml` (manual, `workflow_dispatch`)

**Frontend** (`frontend/`):
- Next.js 15.2.4, React 18, MUI 6, TypeScript
- Port 3000
- Auth: Auth0 (optional for Compose-only preview — `PUBLIC_PREVIEW_MODE=true` skips auth)
- Deployment: Vercel
- CI: `.github/workflows/frontend-build.yml` (auto on push/PR)

**Workflow:** 3-step UI — Upload XLSX → AI classify/merge → Review/Export

**Key seam:** `frontend/src/components/multidoc/backendApi.ts` — sole file calling backend. All calls go through Next.js routes at `/api/compose/*` which proxy to Flask.

---

### 2. v2-backend
**Role:** Production backend. Phase 0 landing zone — structure is complete, kind handlers are stubbed. This repo will replace socratics-v2-prototype's Flask backend.

**What it contains:**

| Component | Type | Status | Purpose |
|-----------|------|--------|---------|
| `services/api` | Flask HTTP | Transitional | Same 17 endpoints as prototype; retires into Next.js BFF in Phase 3+ |
| `services/compute_worker` | Service Bus consumer | Wired, stubbed | Async compute jobs (entity_resolve, assign_children, etc.) |
| `services/ingest_worker` | Service Bus consumer | Wired, stubbed | Per-file ingest pipeline |
| `packages/llm_wrapper` | Python package | Complete | Sole LLM import point; rate limit + quota + UsageEvent emission |
| `packages/queue_client` | Python package | Complete | Producer + Consumer for Service Bus; JobEnvelope wire format |
| `packages/document_client` | Python package | Complete | Cosmos reads/writes; etag patches; BFF internal client |
| `packages/rate_limit` | Python package | Complete | Redis token bucket |
| `packages/multidoc` | Python package | Complete | Shared extract/classify/merge pipeline logic |
| `packages/evaluator` | Stub | Placeholder | Server-side math evaluator |
| `packages/job_envelope` | Stub | Placeholder | Cosmos job state machine |

**Design authority:** `../infrastructure/docs/Multidoc Integration/` — three key documents:
- `task_3.1_target_architecture.md` — target state (§§2, 3, 5, 6, 7, 10)
- `task_3.1_addendum_compute_plane.md` — compute plane spec (§§8.3–8.17)
- `task_3.2_migration_roadmap.md` — Phase 0 → production migration

**7 Load-Bearing Invariants** (from `docs/REPO_ORIENTATION.md`):
1. `llm_wrapper` is the ONLY place that imports provider SDKs — CI enforces
2. Every LLM call emits a UsageEvent — always, no sampling
3. Document writes use etag-protected patches (`If-Match`)
4. Workers enqueue their own successors — no change-feed watcher
5. Structural writes recompute via `packages/evaluator` before patching
6. No cross-partition queries on hot paths
7. Three audit tiers stay separate (correctness / metering / security)

**Local dev:** `docker compose up` brings up Redis, Service Bus emulator (sbedge + servicebus), CosmosDB emulator. Seed with `dev/cosmos/seed_cosmos.py`.

---

### 3. v2-frontend (Agora)
**Role:** Production frontend. Consumes Hasura GraphQL for data and the Flask API ("Logos backend") for file/AI operations.

**Tech stack:** Next.js 15.2.4, React 18.3.1, MUI 6, Redux Toolkit, Apollo Client, Auth0, RxJS, Sentry, Amplitude

**Key architectural patterns:**
- **BFF (Backend-for-Frontend):** All external calls proxied through Next.js API routes. No JWTs exposed to browser.
- **Apollo Client → Hasura:** GraphQL queries proxied through `/api/hasura`, which adds `x-hasura-role`, `x-hasura-user-id`, `x-hasura-tenant-id` headers
- **Logos backend:** File operations (upload, export) and AI chat (SSE) proxied through `/api/logos/*`
- **Observable-based AI chat:** `SocratesService.chat()` returns `Observable<string>` via RxJS; streams via SSE until `"SOCRATES_END"`
- **IndexedDB persistence:** Redux state (except upload, chat, onboarding) persisted to IndexedDB via redux-persist

**Route structure:** `/projects/[pid]/financial-model` is the main workspace. Sub-routes handle validation, documents, reported items, unclassified items, settings, debug.

**Auth flow:**
1. Auth0 Authorization Code Flow (OIDC)
2. httpOnly session cookie (no client-side JWT)
3. `/api/auth/me` resolves identity: Auth0 sub → Hasura user lookup → auto-provision if new
4. Middleware protects `/projects/*`

**Deployment:** Azure Container Apps (`docker-compose.yml` maps port 3010 → 3000). Separate DEV and PROD GitHub Actions workflows with `vars.DEV_*` / `vars.PROD_*` env split.

---

## Cross-Repo Interconnections

### Connection 1: v2-frontend → "Logos" Flask API

The v2-frontend's `/api/logos/*` BFF routes proxy to `{protocol}://logos.{host}/api/v1`. In production this is a subdomain of the app host. This Flask API is either:
- **Dev/prototype environments:** `socratics-v2-prototype/backend/app.py`
- **Production (target):** `v2-backend/services/api/app.py`

Both expose identical endpoint paths. The v2-frontend's `NEXT_PUBLIC_LOGOS_URL` / `NEXT_PUBLIC_BACKEND_URL` env vars control which one it hits.

**Endpoints consumed:**
| v2-frontend route | → Flask endpoint | Purpose |
|-------------------|-----------------|---------|
| `/api/logos/[pid]/upload` | `POST /api/upload-extract` | XLSX ingest |
| `/api/logos/[pid]/chat` | SSE stream | AI chat |
| `/api/logos/[pid]/export` | `GET /api/export/all` | XLSX download |

### Connection 2: v2-backend replaces socratics-v2-prototype backend

v2-backend/services/api exposes the same 17 endpoints as socratics-v2-prototype/backend. The prototype is the seed; v2-backend is the production replacement. Migration is tracked in the infrastructure repo's Phase roadmap.

**Endpoint overlap (both expose):**
- `POST /api/upload-extract`
- `POST /api/classify-inline` (calls Claude)
- `POST /api/confirm-mapping`
- `GET /api/merged`
- `POST /api/assign-children` (calls Claude)
- `POST /api/edits` / `GET /api/edits` / `POST /api/edits/reset`
- `GET /api/export/all`
- `GET /api/health`
- + more

**Key difference:** v2-backend adds `POST /api/chat` (synchronous LLM via llm_wrapper), Service Bus enqueuing, CosmosDB persistence, and the worker BFF internal endpoints.

### Connection 3: Statement type naming translation

| Frontend label | Backend value | Meaning |
|---------------|---------------|---------|
| `IS` | `pl` | Income Statement / Profit & Loss |
| `BS` | `balance_sheet` | Balance Sheet |

Translation happens at the Next.js BFF layer (socratics-v2-prototype's `/api/compose/*` route handlers and v2-frontend's `/api/logos/*` route handlers).

### Connection 4: Shared API contract document

Both `socratics-v2-prototype/backend/docs/compose_integration.md` and `v2-backend/services/api/docs/compose_integration.md` document the same 3-phase frontend integration contract. They are maintained in parallel during the transition.

---

## Shared Infrastructure (External to These Repos)

| Service | Who uses it | Notes |
|---------|-------------|-------|
| Auth0 | v2-frontend, socratics-v2-prototype/frontend | `AUTH0_*` env vars |
| Hasura GraphQL | v2-frontend | Primary data store for Agora; schema not in these repos |
| Azure CosmosDB | v2-backend | `socratics-documents` database; 4 containers |
| Azure Service Bus | v2-backend | `compute-jobs`, `ingest-jobs` topics |
| Azure Blob Storage | v2-backend, socratics-v2-prototype (optional) | `socratics-sources` container |
| Azure Container Registry | socratics-v2-prototype | `socraticsprodacr.azurecr.io` |
| Azure Event Hubs / Log Analytics | v2-backend | UsageEvents telemetry (billing metering) |
| Redis | v2-backend | Rate limiting (DB 1), quota rollups (DB 2) |
| Anthropic API | socratics-v2-prototype, v2-backend | `ANTHROPIC_API_KEY` |
| Vercel | socratics-v2-prototype/frontend | Preview + demo deploys |
| Sentry | v2-frontend (org: socraticsai, project: agora) | Error tracking |
| Amplitude | v2-frontend | Analytics + session replay |

---

## Key File Locations (Quick Reference)

### socratics-v2-prototype
| File | Purpose |
|------|---------|
| `backend/app.py` | All 17 Flask routes |
| `backend/classify.py` | Claude API integration + retries |
| `backend/docs/compose_integration.md` | Frontend integration contract |
| `backend/.env.template` | All backend env vars |
| `frontend/src/components/multidoc/backendApi.ts` | **Sole integration seam** |
| `frontend/src/components/multidoc/MultiDocMerger.tsx` | Top-level orchestrator |
| `frontend/.env.template` | All frontend env vars |

### v2-backend
| File | Purpose |
|------|---------|
| `services/api/app.py` | All 17 Flask routes (production version) |
| `packages/llm_wrapper/__init__.py` | `llm_call`, `LLMContext`, `LLMResponse` |
| `packages/queue_client/__init__.py` | `Producer`, `Consumer`, `JobEnvelope` |
| `packages/document_client/client.py` | CosmosDB reads/writes/patches |
| `packages/document_client/bff_internal.py` | `BffWorkerClient` (workers → API) |
| `schemas/usage_event.schema.json` | UsageEvent billing contract |
| `docker-compose.yml` | Local emulator stack |
| `.env.template` | All 72 env vars |
| `docs/REPO_ORIENTATION.md` | 7 load-bearing invariants |

### v2-frontend
| File | Purpose |
|------|---------|
| `src/app/api/hasura/route.ts` | GraphQL BFF proxy |
| `src/app/api/logos/` | Logos backend BFF proxy routes |
| `src/app/api/auth/me/route.ts` | User identity endpoint |
| `src/lib/identity.ts` | Auth0 → Hasura user resolution |
| `src/services/hasura.service.ts` | Apollo Client setup |
| `src/services/socrates.service.ts` | AI chat Observable |
| `src/store/store.ts` | Redux store + IndexedDB persistence |
| `src/config.ts` | Config loader (env vars → config object) |
| `src/middleware.ts` | Auth0 route protection |

---

## Migration Roadmap (Summary)

```
Phase 0 (current): Landing zone
  - Wire observable end-to-end (complete)
  - llm_wrapper, queue_client, document_client, rate_limit implemented
  - Kind handlers wired but stubbed (DLQ on call)

Phase 1+: Kind implementation
  - Implement compute_worker kinds
  - Implement ingest_worker pipeline
  - Yoshi classifier integration

Phase 3+: API retirement
  - Flask API (v2-backend/services/api) retires
  - Replaced by Next.js BFF (v2-frontend) + direct Cosmos/Service Bus
  - v2-frontend takes ownership of BFF layer

Source: ../infrastructure/docs/Multidoc Integration/task_3.2_migration_roadmap.md
```
