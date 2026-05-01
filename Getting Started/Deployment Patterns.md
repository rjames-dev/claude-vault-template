---
tags:
  - type/guide
  - status/stable
created: {{date}}
---

# Deployment Patterns

Three configurations, from simplest to most complete. Start with Pattern 1 and add layers when you feel the need.

---

## Pattern 1 — Vault Only (Most Users)

**What it is:** The vault as a git-backed knowledge base, paired with `CLAUDE.md` for session orientation and `hot.md` as the session cache. No additional services required.

**What you get:**
- Claude Code reads `CLAUDE.md` automatically at session start
- `hot.md` gives instant re-orientation every session
- Project notes give you persistent context across sessions and machines
- `git pull` on any machine restores full knowledge base instantly
- Vault skills (vault-session-close, vault-ingest, vault-lint) work out of the box

**Setup:**
1. Clone this template and open in Obsidian
2. Edit `CLAUDE.md` → add your active projects
3. Create a project folder: `Projects/_Active/<Your Project>/Current State.md`
4. Detach from the template remote and push to your own private repo:
   ```bash
   git remote remove origin
   git remote add origin git@github.com:<you>/<your-vault>.git
   git push -u origin main
   ```

**Cost:** Near zero — just Obsidian and a private git repo.

---

## Pattern 2 — Full Dev Stack

**What it is:** Pattern 1 plus claude-memory (auto-capture) and a summary-agent that auto-populates vault from sessions.

**What you get:**
- Every Claude Code session captured automatically via PreCompact hook
- PostgreSQL + pgvector for semantic search across all session history
- `hot.md` updated automatically on every compact (no manual end-of-session step)
- Summary agent writes session logs, decisions, and learnings to the vault
- Agent profiles (`context-deep-dive`, `feature-brief`) work with full memory queries

**Additional requirements:**
- Docker (for PostgreSQL, pgvector, Ollama)
- Anthropic API key (for Claude-generated summaries)
- ~2GB RAM for the Docker stack

**Setup:**
1. Complete Pattern 1
2. Clone and run [claude-memory](https://github.com/rjames-dev/claude-memory)
3. Run `scripts/setup-env.sh` — vault path, DB password, API key
4. Run `setup-hooks.sh` — installs PreCompact capture hook and hot.md directive

**When to choose this:** You work across multiple long sessions, return to projects after gaps, or want an automatically maintained knowledge base.

---

## Pattern 3 — Production Server with Help Vault

**What it is:** Pattern 2 running on a server, with a second vault serving as a user-facing help system for an application.

**What you get:**
- **Dev vault** — private session context, same as Pattern 2
- **Help vault** — application documentation served by headless Obsidian as a wiki-style help system, linked from or embedded in your application UI
- Linux user/group isolation on the help vault — restricted contributors can add/edit help content without system access

**Additional requirements:**
- Linux server with Docker
- Headless Obsidian via KasmVNC
- Separate vault repos: one private (dev), one for help content

**When to choose this:** You're building an application and want Obsidian-powered help documentation that your team (or an AI) can maintain as living notes.

---

## Choosing the Right Pattern

| Situation | Pattern |
|---|---|
| One machine, active project, short gaps | 1 |
| Multiple machines or long gaps between sessions | 1 or 2 |
| Many projects, want auto-captured history | 2 |
| Building an app that needs a help system | 3 |
| Server deployment with team contribution | 3 |

## References

- [[When Memory Systems Help]] — deeper explanation of when each layer earns its cost
- [[Your First Project]] — setting up a project in the vault
