---
tags:
  - project/active
  - type/current-state
created: 2026-05-04
updated: 2026-05-04
status: building
---

# Current State — compose-orientation

## What This Is

A deep orientation into the three codebases that make up the Socratics V2 financial consolidation platform: `socratics-v2-prototype`, `v2-backend`, and `v2-frontend`. The goal is a complete, navigable understanding of what each repo does, how they interconnect, and the migration path from prototype to production — so any future session can pick up with full context in seconds.

## Where We Are

Orientation complete. Architecture, data flows, integration contracts, and open questions are all documented. The three repos exist at different stages of maturity: the prototype is a complete self-contained MVP; v2-backend is a Phase 0 production landing zone (wire observable, kind handlers stubbed); v2-frontend (Agora) is the full production frontend. A fourth repo — `infrastructure` — contains the design authority and IaC, but lives outside this folder at `../infrastructure/`.

## What's Done

- [x] Full exploration of socratics-v2-prototype (all backend + frontend files, env vars, API contract, deployment config)
- [x] Full exploration of v2-backend (all services + packages, design invariants, Cosmos/Service Bus topology, env vars)
- [x] Full exploration of v2-frontend (all routes, services, state shape, auth flow, BFF pattern, env vars)
- [x] Cross-repo interconnection map built (see Architecture.md)
- [x] Migration path identified (prototype → v2-backend production replacement)
- [x] Naming mismatch documented (IS/BS vs pl/balance_sheet)
- [x] Open questions catalogued (see Open Questions.md)

## What's Still Missing

- [ ] Locate and read the `infrastructure` repo (design authority for v2-backend, IaC, Hasura schema)
- [ ] Confirm which Flask backend v2-frontend currently points to in prod (prototype vs v2-backend)
- [ ] Understand the Yoshi classifier contract (referenced in v2-backend ingest_worker but TBD)
- [ ] Understand the Hasura schema in full (inferred from v2-frontend types; no GraphQL schema file found)
- [ ] Verify Azure deployment topology across all three repos (endpoints, Container App names, VNet config)

## Immediate Next Steps

1. Find the `infrastructure` repo — it is the design authority for everything in v2-backend
2. Confirm current prod routing: does `NEXT_PUBLIC_LOGOS_URL` in v2-frontend point to socratics-v2-prototype or v2-backend/services/api?
3. Read `task_3.1_target_architecture.md` and `task_3.1_addendum_compute_plane.md` in the infrastructure repo

## Key Decisions Made

- socratics-v2-prototype is intentionally self-contained — not a dependency of v2-backend or v2-frontend, but the seed from which both grew
- v2-backend's API service (Flask, port 5050) exposes the same 17 endpoints as the prototype — it is the production replacement, not a separate service
- v2-frontend uses a strict BFF pattern: all backend calls go through Next.js API routes, no JWTs exposed to the browser
- The "Logos backend" in v2-frontend = the Flask API (either prototype or v2-backend/services/api depending on environment)
- Statement type naming differs across the boundary: frontend uses `IS`/`BS`, backends use `pl`/`balance_sheet`; the Next.js BFF handles translation
