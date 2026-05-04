---
tags:
  - project/active
  - type/open-questions
created: 2026-05-04
---

# Open Questions — compose-orientation

*Unresolved questions, blockers, and decisions pending.*

---

## Architecture

- [ ] **Where is the infrastructure repo?** — v2-backend's design authority is `../infrastructure/docs/Multidoc Integration/`. This repo is not in the v2/ folder. It contains the target architecture document, compute plane addendum, and migration roadmap — critical reading for understanding v2-backend's intended direction. Need to locate it.

- [ ] **Which Flask backend does v2-frontend currently point to in prod?** — The "Logos backend" proxied by v2-frontend's `/api/logos/*` routes is either `socratics-v2-prototype/backend` or `v2-backend/services/api`. Both expose identical endpoints. The answer determines how far along the migration actually is. Check `NEXT_PUBLIC_LOGOS_URL` value in prod Vercel/Azure env config.

- [ ] **Is v2-backend/services/api currently deployed anywhere?** — The prototype backend is deployed on Azure Container Apps (`demos` resource group). Does v2-backend's API service have its own deployment? No deployment workflow was found in v2-backend (unlike the prototype which has `deploy-backend.yml`).

## v2-backend Specifics

- [ ] **Yoshi classifier — what is it?** — Referenced throughout `ingest_worker` as the classifier for the ingest pipeline ("Yoshi HTTP contract TBD"). The contract (endpoint, auth, failure semantics) is an open decision in `services/ingest_worker/DEFINITIONS.md`. Is Yoshi an internal service or a third-party vendor?

- [ ] **`packages/evaluator` scope** — Placeholder for server-side math evaluator (target §10.7). When is this prioritized? The compute plane invariant says structural writes must recompute via evaluator before patching — until this is implemented, that invariant cannot be satisfied.

- [ ] **Cosmos `jobs` container** — Referenced in env template and Cosmos schema but not created by `dev/cosmos/seed_cosmos.py`. Is this a deliberate defer or an oversight?

- [ ] **Ingest worker blob authentication** — `DEFINITIONS.md` decision 2: should ingest worker use managed identity or SAS URL validation? Affects security posture and Azure identity configuration.

## v2-frontend Specifics

- [ ] **Full Hasura GraphQL schema** — Inferred from v2-frontend types and service files but no `.graphql` schema file was found in any of the three repos. The authoritative schema is presumably in the infrastructure repo or the Hasura console. Need this to understand the full data model.

- [ ] **WebSocket subscriptions** — v2-frontend has `graphql-ws` installed but BFF pattern blocks client-side WebSocket to Hasura. Comment says "no WebSocket subscriptions (BFF limitation; future enhancement)." Is real-time data (project state changes) currently handled by polling? How frequently?

- [ ] **"Logos" URL construction in prod** — v2-frontend's logos proxy derives the Logos URL from the request host: `{protocol}://logos.{host}/api/v1`. Does a `logos.` subdomain exist in all environments (dev, staging, prod)? Or is it overridden by `NEXT_PUBLIC_LOGOS_URL`?

## Migration & Sequencing

- [ ] **Phase 3+ API retirement timeline** — When does the Flask API (v2-backend/services/api) retire in favor of the Next.js BFF? This is referenced in v2-backend docs but not scheduled. Understanding this affects whether to invest in the Flask API or skip to the BFF.

- [ ] **socratics-v2-prototype end-of-life** — The prototype is a demo/preview vehicle. Is there a plan to retire it, or does it continue as a lightweight demo track separate from the production platform?

- [ ] **Bicep templates for v2-backend** — The prototype's `deploy-backend.yml` references `backend/infrastructure/bicep/main.bicep` but these may not exist yet. Does v2-backend have its own IaC, or does that live in the infrastructure repo?
