---
tags:
  - type/index
created: {{date}}
status: active
---

# Vault Index

Master catalog of vault contents. Update when folders or major notes are added or removed.

---

## Vault Root Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Claude Code session instructions; read automatically on session start |
| `hot.md` | Session cache — read first every session |
| `index.md` | This file — master catalog |
| `log.md` | Append-only AI operations log |

---

## Folders

### Inbox/
Zero-friction capture zone. Drop anything here — links, ideas, screenshots, pastes. Process with `vault-ingest` skill on request.

### Projects/
All project work, split by lifecycle status.

| Subfolder | Contents |
|-----------|---------|
| `_Active/` | <!-- list active projects --> |
| `_Parked/` | <!-- list parked projects --> |
| `_Archive/` | <!-- list archived projects --> |

Root-level: `Roadmap.md`

### Research/
Background knowledge, references, reading notes.

<!-- List research subfolders here as they are created -->

### Knowledge/
Distilled, reusable technical patterns. Not project-specific.

<!-- List knowledge notes here -->

### Templates/
Reusable document templates.

<!-- List templates here -->

### Claude/
Claude's working space — not primary knowledge, but working artifacts.

| Subfolder | Contents |
|-----------|---------|
| `Session-Logs/` | Per-session summaries (YYYY-MM-DD.md) |
| `Toolbox/` | Claude Code harness reference: Agents, Commands, Hooks, MCP, Skills |
| `Scratch/` | Temporary drafts and in-progress work |
