# Claude Context — Vault Root

Loaded automatically every session. Keep it concise.

## Session Protocol

**Start:**
1. Read `hot.md` — single-file session cache, gets you up to speed immediately
2. Read `Projects/_Active/<name>/Current State.md` for any project you'll be working on

**End:**
Run the `vault-session-close` skill (or read `Claude/Toolbox/Skills/vault-session-close.md` for manual steps).

---

## Vault Structure

```
claude-vault/
  hot.md              ← session cache (read first)
  index.md            ← master catalog
  log.md              ← append-only operations log
  Inbox/              ← zero-friction capture; process with vault-ingest
  Projects/
    _Active/          ← current work
    _Parked/          ← paused, will resume
    _Archive/         ← done or abandoned
  Research/           ← background knowledge by topic
  Knowledge/          ← reusable technical patterns
  Templates/          ← PRD, project protocol
  Claude/
    Session-Logs/     ← per-session summaries
    Toolbox/          ← harness reference: agents, skills, hooks, MCP, commands
    Scratch/          ← temp drafts
```

---

## Active Projects

| Project | Path | Status |
|---------|------|--------|
<!-- Add active projects here: | Project Name | `Projects/_Active/Name/` | brief status | -->

See `Projects/Roadmap.md` for full project sequence and dependencies.

---

## Formatting Conventions

- YAML frontmatter on all notes: `tags`, `created`, `status`
- Headings for structure — not bold text as pseudo-headings
- Wikilinks `[[Note Name]]` for internal links
- Nested tags: `#project/active`, `#status/draft`, `#type/log`
- Callouts `> [!note]` for highlighted info

---

## Skills

Vault skills live in `Claude/Toolbox/Skills/`. Read the skill file to execute.

| Skill | When to use |
|-------|-------------|
| `vault-session-close.md` | End of every session |
| `vault-ingest.md` | Processing Inbox/ items or raw materials into notes |
| `vault-lint.md` | Monthly health check — broken links, stale notes, missing frontmatter |
| `read-project-notes.md` | Get structured project context (used by agent profiles) |
