---
tags:
  - project/active
  - type/decisions-log
created: 2026-05-04
---

# Decisions Log — compose-orientation

*Architecture and implementation decisions, newest first.*

---

## 2026-05-04

### socratics-v2-prototype is intentionally standalone
The prototype shares no code with v2-backend or v2-frontend. It was a fast-moving MVP that defined the product workflow. Keeping it isolated means it can be deployed for demos and Vercel previews without depending on production infrastructure.

### v2-backend/services/api exposes the same 17 endpoints as the prototype backend
This is the transition strategy: keep the API contract stable while replacing internals (JSON files → CosmosDB, direct Anthropic calls → llm_wrapper with rate limiting and metering). The Flask API is labeled "transitional" and is planned to retire in Phase 3+ when the Next.js BFF takes over.

### llm_wrapper is the sole LLM import point (enforced by CI lint)
All LLM calls go through `packages/llm_wrapper`. This is a load-bearing invariant enforced by `tests/test_no_direct_provider_imports.py`. Rationale: every call must emit a UsageEvent, apply rate limiting, and enforce tenant quota. Centralizing this makes those guarantees unconditional.

### Workers write to Cosmos via BFF internal endpoints, not directly
Compute and ingest workers POST to `/api/document/<project_id>/*` on the Flask API rather than writing directly to CosmosDB. Rationale (addendum §8.11.2): keeps all document mutation logic in one place (the API), enables etag-based optimistic concurrency to be enforced consistently, and avoids workers needing Cosmos credentials.

### v2-frontend uses BFF pattern (no client-side JWTs)
All backend calls from the browser go through Next.js API routes, which add auth headers server-side. The browser never sees a JWT. Rationale: Auth0 session is httpOnly cookie; Hasura admin secret stays server-side; Logos URL/key stays server-side.

### Statement type names differ across the boundary
Frontend uses `IS` / `BS`; backends use `pl` / `balance_sheet`. The translation happens in the Next.js BFF layer. This was presumably a naming choice made in the prototype era that was not worth breaking to fix.

### UsageEvents go to Event Hubs or Log Analytics, never CosmosDB
Billing metering data is write-heavy, query-intensive, and immutable — poor fit for CosmosDB's per-request cost model. Event Hubs/Log Analytics are the telemetry backbone. This is enforced by the llm_wrapper's sink architecture (load-bearing invariant #7).

### Workers enqueue their own successors (no orchestrator)
The compute plane uses explicit successor enqueuing in handler code rather than a workflow orchestrator or change-feed watcher. Rationale (addendum §8.6): simpler mental model, no additional service dependency, job lineage tracked via `parent_job_id` in JobEnvelope.
