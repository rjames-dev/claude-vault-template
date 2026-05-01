---
tags:
  - type/project
  - topic/obsidian
  - topic/claude-code
  - topic/mcp
created: 2026-04-23
status: planning
---

# Recommended Implementation Addons

*Best-fit stack for coding teams using Claude Code + Obsidian. Based on [[Research/Obsidian Enhancement Projects]].*

---

## Stack Overview

```
Shared Obsidian Vault (git repo)
        │
        ├── Obsidian Git          ← syncs vault across team
        ├── obsidian-mcp-server   ← Claude Code reads/writes vault
        ├── Dataview              ← team dashboards from structured notes
        └── Smart Connections     ← semantic search across team knowledge

Per Developer
        └── claude-mem            ← captures sessions → writes to shared vault
```

---

## Components

### Foundation

**[Obsidian Git](https://github.com/Vinzent03/obsidian-git)** — 10.6k stars
The non-negotiable starting point for team use. Without it, the vault is siloed per developer. With it, the vault becomes a shared git repo — everyone pulls session logs, architecture decisions, and project status automatically.

---

### Claude Code ↔ Vault Bridge

**[obsidian-mcp-server](https://github.com/cyanheads/obsidian-mcp-server)** — 465 stars
MCP server giving Claude Code live read/write access to the vault. Preferred over alternatives for team use due to JWT/OAuth auth support — important when multiple developers are connecting. Features atomic frontmatter management, tag tools, regex search, and soft-delete.

*Alternative:* [mcp-obsidian](https://github.com/MarkusPfundstein/mcp-obsidian) (3.5k stars) — simpler, more widely adopted, good for smaller teams without auth requirements.

---

### Memory Layer

**[claude-mem](https://github.com/thedotmack/claude-mem)** — 66k stars
Per-developer session capture, routed into the shared vault via the MCP server above. Each developer's Claude Code sessions — decisions, solutions, rationale — accumulate in the team vault automatically. Community standard for Claude Code persistent memory.

---

### Knowledge Querying

**[Dataview](https://github.com/blacksmithgu/obsidian-dataview)** — 8.8k stars
Turns Claude-written frontmatter into live team dashboards — open questions across projects, decision logs, project status. Pairs naturally with structured notes Claude Code writes into the vault.

**[Smart Connections](https://github.com/brianpetro/obsidian-smart-connections)** — 4.9k stars
Semantic search across the full vault. Surfaces related notes as developers work — past decisions, similar problems solved, relevant architecture notes.

---

### Optional / High Value

**[Claudian](https://github.com/YishenTu/claudian)** — 9k stars
Embeds Claude Code directly inside Obsidian — inline editing with diff preview, `@mention` vault files, plan mode, MCP support. Collapses the vault and Claude Code into a single environment. High value but requires a workflow shift from the team.

---

## Key Insight

The vault is the team's shared brain, Obsidian Git keeps it in sync, and the MCP server is what lets Claude Code participate in it. `claude-mem` is the pipeline that feeds session knowledge into that shared brain automatically.

---

## Implementation Order

- [ ] Set up shared Obsidian vault as a git repo
- [ ] Install and configure Obsidian Git (auto-commit/push schedule)
- [ ] Install obsidian-mcp-server + Obsidian Local REST API plugin
- [ ] Add MCP server to each developer's Claude Code config
- [ ] Install claude-mem per developer, configure to write to vault
- [ ] Install Dataview and create project dashboard templates
- [ ] Install Smart Connections for semantic search
- [ ] Evaluate Claudian for team adoption

---

## Related

- [[Research/Obsidian Enhancement Projects]]
- [[Getting Started/Deployment Patterns]]
