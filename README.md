# Claude Vault — Obsidian Vault Template

A pre-configured Obsidian vault that pairs with [claude-memory](https://github.com/rjames-dev/claude-memory) to give Claude Code persistent, structured memory across sessions. The vault provides the human-readable narrative layer; claude-memory provides the deep technical archive.

**Not sure if you need all of this?** Read [`Getting Started/When Memory Systems Help.md`](Getting%20Started/When%20Memory%20Systems%20Help.md) first — the vault alone (no claude-memory) is already useful for many workflows.

## Getting Started

Three guides are included in the `Getting Started/` folder:

| Guide | What it covers |
|---|---|
| [When Memory Systems Help](Getting%20Started/When%20Memory%20Systems%20Help.md) | Honest assessment of when each layer earns its complexity |
| [Deployment Patterns](Getting%20Started/Deployment%20Patterns.md) | Vault only vs full stack vs production server — which fits you |
| [Your First Project](Getting%20Started/Your%20First%20Project.md) | How to set up a project folder and get Claude oriented to it |

## Setup

```bash
git clone https://github.com/rjames-dev/obsidian-vault-template ~/code/claude-vault
```

Then open Obsidian → **Open folder as vault** → point to `~/code/claude-vault`.

> **Important:** Detach from the template remote and push to your own private repository:
> ```bash
> cd ~/code/claude-vault
> git remote remove origin
> git remote add origin git@github.com:<you>/<your-vault>.git
> git push -u origin main
> ```

## Folder Structure

```
claude-vault/
  hot.md              ← session cache (read first every session)
  index.md            ← master catalog of vault contents
  log.md              ← append-only AI operations log
  CLAUDE.md           ← auto-loaded by Claude Code; session protocol
  Inbox/              ← zero-friction capture; process with vault-ingest
  Projects/
    _Active/          ← current work
    _Parked/          ← paused, will resume
    _Archive/         ← done or abandoned
    Roadmap.md        ← project sequence and dependencies
  Research/           ← background knowledge by topic
  Knowledge/          ← reusable technical patterns
    Learnings Log.md  ← auto-populated errors and lessons
  Templates/          ← PRD, project protocol
  Claude/
    Session-Logs/     ← per-session summaries (auto-written by summary-agent)
    Toolbox/          ← harness reference: agents, skills, hooks, MCP, commands
      Skills/         ← vault-native skill files
      Agents/
        Profiles/     ← agent profile files
    Scratch/          ← temp drafts (gitignored)
  Getting Started/    ← guides for new users
```

## How It Works

### Session Start
1. `CLAUDE.md` is loaded automatically by Claude Code
2. Claude reads `hot.md` — one file, complete context, immediately oriented
3. Claude reads `Projects/_Active/<name>/Current State.md` for the active project

### During a Session
- Drop things in `Inbox/` and say "run vault-ingest" to turn them into proper notes
- Compactions auto-update `hot.md` and (if claude-memory is running) capture the session

### Session End
Run "vault-session-close" — updates `hot.md`, `Current State.md`, writes a session log, and pushes to git.

## Vault Skills

Four vault-native skills are included:

| Skill | Purpose |
|-------|---------|
| `vault-session-close` | End-of-session: update hot.md, current state, session log, git push |
| `vault-ingest` | Process Inbox/ items or pasted content into proper vault notes |
| `vault-lint` | Monthly health check: broken links, stale notes, missing frontmatter |
| `read-project-notes` | Structured project context (used by agent profiles) |

Skills live in `Claude/Toolbox/Skills/`. Invoke by name: "run vault-ingest", "lint the vault".

## Agent Profiles

Two agent profiles are included in `Claude/Toolbox/Agents/Profiles/`:

| Profile | Purpose |
|---------|---------|
| `feature-brief` | Pre-coding context brief — explores codebase + queries memory history |
| `context-deep-dive` | Project re-orientation — synthesizes vault + claude-memory into a single brief |

## What Gets Committed

- All `.md` notes
- `.obsidian/` config (plugins, theme, hotkeys) — keeps settings in sync across machines
- `CLAUDE.md`, `hot.md`, `index.md`, `log.md`

**Not committed:** `workspace.json` (machine-specific), `Claude/Scratch/` contents, `.DS_Store`
