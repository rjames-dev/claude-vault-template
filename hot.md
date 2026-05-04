---
tags:
  - type/session-cache
updated: 2026-05-04
---

# Hot Cache

**Read this file first every session.** Updated automatically by the PreCompact hook on every compact, and manually by vault-session-close if a session ends without a compact.

---

## Last Updated
2026-05-04

## Current Focus
compose-orientation — building complete understanding of the three V2 repos and their interconnections.

## What Was Just Done
- Reviewed claude-vault-template (the vault itself)
- Did full deep exploration of all three repos: socratics-v2-prototype, v2-backend, v2-frontend
- Created compose-orientation project in Projects/_Active/ with Current State, Architecture, Decisions Log, and Open Questions
- Registered project in CLAUDE.md, index.md, and Roadmap.md

## Active Projects
- **compose-orientation** — orientation complete; infrastructure repo not yet located; 12 open questions catalogued

## Open Questions
- Where is the `infrastructure` repo? (design authority for v2-backend; lives at `../infrastructure/` relative to v2-backend)
- Which Flask backend does v2-frontend currently point to in prod — prototype or v2-backend/services/api?
- What is Yoshi classifier? (referenced in v2-backend ingest_worker, contract TBD)

## Next Session Picks Up From
Read `Projects/_Active/compose-orientation/Architecture.md` for full system map. The natural next step is locating the infrastructure repo and reading the three design authority documents: `task_3.1_target_architecture.md`, `task_3.1_addendum_compute_plane.md`, and `task_3.2_migration_roadmap.md`. Then confirm prod routing by checking live env config for `NEXT_PUBLIC_LOGOS_URL` in v2-frontend.
