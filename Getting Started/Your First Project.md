---
tags:
  - type/guide
  - status/stable
created: {{date}}
---

# Your First Project

How to set up a project in the vault and get Claude Code oriented to it.

## 1. Create the Project Folder

Inside `Projects/_Active/`, create a subfolder named after your project:

```
Projects/_Active/
└── My App/
    ├── Current State.md     ← required
    ├── Decisions Log.md     ← recommended
    └── Open Questions.md    ← optional
```

You can create these manually in Obsidian or from the terminal:

```bash
mkdir -p ~/code/claude-vault/Projects/_Active/"My App"
```

See `Templates/PROJECT-PROTOCOL.md` for the full template and all file schemas.

---

## 2. Write Current State.md

This is the most important file. Claude reads it at session start to orient itself. Keep it accurate — a stale Current State is worse than none.

```markdown
---
tags:
  - project/active
  - type/current-state
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: building
---

# Current State — My App

## What This Is
One paragraph describing the project — what it does, who it's for, why it exists.

## Where We Are
Current phase, what's working, what's in progress.

## What's Done
- [x] Completed thing

## What's Still Missing
- [ ] Next thing to build

## Immediate Next Steps
1. First thing to do

## Key Decisions Made
- Why we chose X over Y
```

---

## 3. Register the Project in CLAUDE.md

Open `CLAUDE.md` and add your project to the Active Projects table:

```markdown
## Active Projects

| Project | Path | Status |
|---------|------|--------|
| My App | `Projects/_Active/My App/` | brief one-line status |
```

Claude Code loads `CLAUDE.md` automatically. From that point forward, every session starts with Claude already knowing your active projects.

---

## 4. Update index.md

Add your project to the `_Active/` list in `index.md` so the master catalog stays current.

---

## 5. Add to Roadmap.md (Recommended)

Add a row to `Projects/Roadmap.md` with the current gate or milestone. This is useful for multi-project contexts where sequencing matters.

---

## 6. Make the Vault Your Own (Git Setup)

If you used the **Use this template** button on GitHub, your vault already has its own clean repo — nothing to do here.

If you cloned manually, detach from the template and start a fresh history:

```bash
cd ~/vaults/claude-vault
rm -rf .git
git init
git add -A
git commit -m "initial vault"
git remote add origin git@github.com:<your-username>/<your-vault-repo>.git
git push -u origin main
```

From that point, end sessions with `vault-session-close` to commit and push automatically.

---

## Session Protocol

**At session start:** Claude reads `CLAUDE.md` (automatic), then reads `hot.md` for the session cache.

**At session end:** Run `vault-session-close` — updates `hot.md`, `Current State.md`, writes a session log, and pushes to git.

---

## References

- [[Templates/PROJECT-PROTOCOL]] — full file schemas and templates
- [[When Memory Systems Help]] — deciding how much infrastructure to add
- [[Deployment Patterns]] — full stack vs vault-only
