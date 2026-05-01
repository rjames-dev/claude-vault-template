---
tags:
  - type/strategy
  - topic/claude-code
  - status/draft
created: {{date}}
---

# Claude Environment Strategy — Dynamic Workbench

## The Problem

The `.claude/` folder tends to accumulate noise over time — commands installed for one project that persist long after the work is done, hooks that made sense in one context but add overhead in another. A cluttered workbench slows things down and makes it harder to know what's actually relevant right now.

## The Mental Model

```
Toolbox (vault)          — the shelf: everything known, researched, available
.claude/ (active)        — the workbench: only what's needed for current work
```

The Toolbox is permanent and grows over time. The workbench is curated and should reflect the current project context.

## Design Goals

1. **Low noise** — `.claude/` contains only tools relevant to active work
2. **Easy activation** — loading a project's tools should take one command
3. **Easy cleanup** — closing a project should leave no residue
4. **Vault-driven** — the vault is already the source of truth for active projects; the environment should follow from it
5. **No magic** — the mechanism should be transparent and easy to reason about

---

## Options Under Consideration

### Option A — Manual Profile Scripts
Each project has `activate.sh` / `deactivate.sh` scripts that copy/remove files from `~/.claude/commands/`.

**Pros:** Simple, explicit, no dependencies
**Cons:** Manual step to remember, scripts to maintain per project

---

### Option B — Symlinks
`~/.claude/commands/` holds symlinks into Toolbox source files.
Adding = `ln -s`, removing = `rm`.

**Pros:** Source files stay in Toolbox, changes propagate automatically, easy to audit what's active vs stored
**Cons:** Still manual, symlinks can confuse some tools

---

### Option C — Project Manifest + Sync Script (Preferred Direction)
Each `Projects/<Name>/` folder optionally contains a `claude-tools.json` listing which commands and hooks should be active for that project. A sync script reads the active projects from `CLAUDE.md` and reconciles `~/.claude/` accordingly.

```json
// Projects/_Active/MyProject/claude-tools.json (example)
{
  "commands": [
    "my-project-deploy",
    "my-project-status"
  ],
  "hooks": {
    "PreCompact": ["session-state-myproject"]
  }
}
```

**Pros:** Vault-native, ties into existing project structure, could be automated via a hook or session-start skill, auditable
**Cons:** More design work upfront, sync script needs building

**Activation flow:**
```
session starts
  → read CLAUDE.md active projects
  → for each active project, read claude-tools.json
  → union of all tool lists → install into ~/.claude/
  → on project close → remove project's tools from ~/.claude/
```

---

## Open Design Questions

1. **Conflict resolution** — what if two active projects want different versions of the same command?
2. **Hook management** — hooks are in `settings.json`, not individual files. Merging/removing requires careful JSON handling.
3. **Session-start integration** — natural trigger point, but auto-activate or just surface a prompt?
4. **Granularity** — workspace-level (always active) vs project-level (only when open).
5. **Deactivation trigger** — removing from CLAUDE.md active list? Explicit command? Manual?
6. **`.claude/` baseline** — some things should always be present and never touched by project activation.

---

## Relationship to Existing Infrastructure

| Component | Role in this strategy |
|-----------|----------------------|
| `Claude/Toolbox/` | Source of all available tools — the shelf |
| `Projects/<Name>/claude-tools.json` | Per-project tool manifest (to be created) |
| `~/.claude/commands/` | Active workbench — synced from manifests |
| `~/.claude/settings.json` | Active hooks — merged from manifests |
| `CLAUDE.md` active projects list | Drives what gets activated |

---

## Next Steps (when ready to build)

- [ ] Decide on Option A / B / C or a hybrid
- [ ] Resolve open design questions above
- [ ] Prototype with one project
- [ ] Build the sync script
- [ ] Wire into session-start or a dedicated `/env-sync` command
- [ ] Document the full pattern back here once proven

---

## Related

- [[README]] — Toolbox overview and promotion path
- [[Hooks/README]] — hook merge pattern
- [[Commands/README]] — commands catalogue
- [[Skills/README]] — skills system
