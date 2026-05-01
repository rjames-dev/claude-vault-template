---
tags:
  - type/guide
  - topic/system
  - status/active
created: {{date}}
---

# System Guide — Vault + Claude Memory

A living reference for the combined vault + claude-memory system. Update this document when new skills, profiles, or capabilities are added.

---

## The Two-Layer System

```
┌──────────────────────────────────────────────────────┐
│  LAYER 1 — claude-memory (Deep Archive)              │
│                                                      │
│  Auto-captured on every compaction                   │
│  Semantic search across all sessions                 │
│  Decisions, bugs, file activity, timelines           │
│  Never loses anything                                │
└─────────────────────┬────────────────────────────────┘
                      │  summary-agent promotes automatically
                      │  PreCompact hook captures automatically
                      ▼
┌──────────────────────────────────────────────────────┐
│  LAYER 2 — Obsidian Vault (Living Knowledge)         │
│                                                      │
│  Human-readable narrative and project structure      │
│  Current State, Architecture, Decisions, Questions   │
│  Git-backed — portable, versioned, browsable         │
│  Planning layer — where ideas become direction       │
└─────────────────────┬────────────────────────────────┘
                      │  CLAUDE.md + hot.md orient every session
                      │  read-project-notes primes memory queries
                      ▼
┌──────────────────────────────────────────────────────┐
│  Claude Code Session                                 │
│                                                      │
│  CLAUDE.md auto-loaded at start                      │
│  hot.md read first → immediate orientation           │
│  MCP tools query both layers in the same session     │
└──────────────────────────────────────────────────────┘
```

**The key insight:** claude-memory is wide and automatic — it captures everything but has no narrative thread. The vault is structured and human-readable but only as current as someone bothered to write. Together they are complete. The bridge between them is mostly automatic, but you can also traverse it deliberately using agent profiles and MCP queries.

---

## What Happens Without You Doing Anything

Most of the system is self-maintaining. On every compaction:

**1. hot.md is updated (PreCompact hook)**
Claude writes the current session state to `hot.md` before compacting — current focus, what was just done, open questions, where to pick up next. This is the primary session cache read at the start of every future session.

**2. claude-memory captures the session (PreCompact hook)**
`auto-capture-precompact.py` sends the conversation snapshot to the claude-memory pipeline: structured summary, embeddings, metadata (decisions, bugs, file mentions, tags).

**3. Vault notes are updated (summary-agent)**
After every qualifying compaction, a Haiku agent runs fire-and-forget and writes to the vault:
- `Claude/Session-Logs/YYYY-MM-DD.md` — session log entry
- `Projects/_Active/<name>/Decisions Log.md` — architectural decisions
- `Projects/_Active/<name>/Open Questions.md` — unresolved issues
- `Knowledge/Learnings Log.md` — errors and patterns

**You don't need to do anything for any of this.** It runs on every compact.

> [!note] When no compact fires
> Short sessions that end without a compact won't trigger the above.
> Run `vault-session-close` manually to push the vault and update hot.md as a fallback.

---

## Session Workflows

### Starting a Session
1. CLAUDE.md is loaded automatically — no action needed
2. Read `hot.md` — one file, complete context, you're oriented

That's it. If you're resuming a project after significant time away, also run:
> "run context-deep-dive on \<project name\>"

### During a Session
Work normally. Compactions handle capture and vault updates automatically.

To capture something ad hoc (link, idea, pasted content, screenshot):
1. Drop it in `Inbox/` **or** paste directly into the conversation
2. Say: `run vault-ingest`

### Ending a Session
If a compact fired during the session — hot.md and vault are already updated. Just commit and push:
> "run vault-session-close"

If no compact fired — vault-session-close handles everything: hot.md update, Current State, session log, git push.

---

## Agent Profiles

Profiles are named, reusable agent configurations. Invoke by name in natural language. Claude reads the profile, runs preflight skills, assembles the prompt, and launches the agent.

Profiles live in: `Claude/Toolbox/Agents/Profiles/`

---

### `feature-brief` — Pre-Coding Context Brief

**When to use:** Before starting any non-trivial feature or bug fix. Prevents re-discovering context you've already mapped.

**What it does:**
- Preflight: queries claude-memory for relevant history on the topic
- Launches an Explore agent to map the codebase area
- Returns: what exists, how it's wired, known state from memory, where to start, watch-outs

**Invocation:**
> "spin up the feature-brief agent for \<topic\> in \<repo/dir\>"

---

### `context-deep-dive` — Project Re-Orientation

**When to use:** Resuming a project after days or weeks away. Combining the vault's narrative with memory's technical depth into a single brief.

**What it does:**
- Preflight: reads vault notes for the project (Current State, Architecture, Decisions, Open Questions) and extracts key terms
- Launches a general-purpose agent that queries claude-memory with those terms: timeline, decisions, knowledge, file activity, bugs
- Synthesizes: what the vault says, what memory adds, which open questions have answers, recent work not yet in vault, where to start
- Postflight: archives the brief to claude-memory

**Invocation:**
> "run context-deep-dive on \<project name\>"

**Output sections:**
- What the Vault Says
- What Memory Adds
- Open Questions — Answered
- Open Questions — Still Open
- Recent Work Not in Vault
- **Start Here** ← the most important section

---

## Vault Skills

Skills are step-by-step workflows Claude executes in the current session. No subagent spawned.

Skills live in: `Claude/Toolbox/Skills/`

---

### `vault-session-close` — End of Session

**When to use:** When ending a session, especially one without a compact.

**What it does:**
1. Updates `hot.md` if no compact fired (fallback — hook handles this automatically on compacts)
2. Updates `Projects/_Active/<name>/Current State.md` for any project touched
3. Appends session log to `Claude/Session-Logs/YYYY-MM-DD.md`
4. Quick vault lint (frontmatter and stale Current State checks only)
5. Incremental graphify update (if graphify is installed)
6. `git add -A && git commit && git push` in the vault
7. Reminds you to `git pull` on other machines

**Invocation:**
> "run vault-session-close" / "close the session" / "wrap up"

---

### `vault-ingest` — Process Raw Materials

**When to use:** When you have something to add to the vault — a link, a pasted doc, a screenshot, an Inbox item — and want it turned into a proper vault note.

**What it does:**
1. Reads the source material
2. Determines the right destination folder
3. Writes a note with frontmatter, structure, summary paragraph, wikilinks
4. Deletes the Inbox source if applicable
5. Appends an entry to `log.md`

**Invocation:**
> "run vault-ingest" / "ingest this" / "add this to the vault: \<content\>"

---

### `vault-lint` — Vault Health Check

**When to use:** Monthly, or before a session where vault accuracy matters.

**What it checks:**
1. Broken wikilinks
2. Missing frontmatter
3. Stale Current State files (not updated in 30+ days)
4. Inbox items older than 7 days
5. Orphaned notes (no incoming wikilinks)
6. Notes missing `status:` field

**Invocation:**
> "run vault-lint" / "lint the vault" / "check vault health"

---

### `read-project-notes` — Read Project Vault Notes

**When to use:** Primarily as a preflight skill in agent profiles. Can also be used directly to get a structured project summary.

**What it does:** Finds the project folder across `_Active/`→`_Parked/`→`_Archive/`, reads key notes, returns structured output including key terms tuned for claude-memory queries.

**Invocation:**
> "read project notes for \<project name\>"

---

## Searching Your History (claude-memory)

claude-memory MCP tools are always available. Claude can call them directly — just describe what you're looking for.

| I want to... | Say this |
|---|---|
| Find a past decision | "search memory for decisions about \<topic\>" |
| See project session history | "get the timeline for \<project\>" |
| Find how a bug was fixed | "analyze bugs related to \<component\>" |
| Find a technical pattern | "search memory for \<topic\>" |
| Use vault notes to search deeper | "Read \<project\> Current State, then search memory for \<topic\>" |
| Find exact text from a past session | "search memory for the exact phrase '\<phrase\>'" |

---

## Vault Map

```
claude-vault/
  CLAUDE.md          ← loaded every session; session protocol, vault structure, active projects
  hot.md             ← session cache; read first every session; auto-updated on compact
  index.md           ← master catalog of vault contents
  log.md             ← append-only record of significant AI actions in the vault
  Inbox/             ← zero-friction capture; process with vault-ingest

  Projects/
    _Active/         ← current work
    _Parked/         ← paused, will resume
    _Archive/        ← done or abandoned
    Roadmap.md       ← project sequence and dependencies

  Research/          ← background knowledge by topic

  Knowledge/         ← reusable technical patterns

  Templates/         ← PRD template, project protocol

  Claude/
    Session-Logs/    ← per-session summaries (written by summary-agent + manually)
    Toolbox/         ← this guide lives here; skill and agent reference
      Skills/        ← vault-native skill files
      Agents/
        Profiles/    ← agent profile files
    Scratch/         ← temporary drafts
```

**Project folder conventions:**
Each project folder contains some combination of: `Current State.md` (required), `Architecture.md`, `Decisions Log.md`, `Open Questions.md`, plus project-specific notes.

---

## Quick Reference

| I want to... | Do this |
|---|---|
| Orient at session start | Read `hot.md` (auto-loaded context) |
| Resume a project after time away | `run context-deep-dive on <project>` |
| Get codebase context before coding | `spin up the feature-brief agent for <topic> in <repo>` |
| Capture something quickly | Drop in `Inbox/`, then `run vault-ingest` |
| Process pasted content into a note | Paste it, then `run vault-ingest` |
| End a session and push the vault | `run vault-session-close` |
| Check vault health | `run vault-lint` |
| Find a past decision | `search memory for decisions about <topic>` |
| See project session history | `get the timeline for <project>` |
| Find how a bug was fixed | `analyze bugs related to <component>` |
| Find a technical pattern | `search memory for <topic>` |
| See what skills are available | Read `Claude/Toolbox/Skills/README.md` |
| See what agent profiles exist | Read `Claude/Toolbox/Agents/Profiles/README.md` |

---

## Adding to This System

### New Skill
1. Create `Claude/Toolbox/Skills/<name>.md` using the schema in `Skills/README.md`
2. Add to the skills table in `Skills/README.md`
3. Add a row to the Quick Reference table above

### New Agent Profile
1. Create `Claude/Toolbox/Agents/Profiles/<name>.md` using the schema in `Profiles/README.md`
2. Add to the Available Profiles table in `Profiles/README.md`
3. Add invocation pattern and description to the Agent Profiles section above
4. Add a row to the Quick Reference table above

### New Project
1. Create `Projects/_Active/<name>/` with at minimum `Current State.md`
2. Add to vault `CLAUDE.md` active projects table
3. Add to `index.md` `_Active/` list

---

## Related

- [[Claude/Toolbox/Skills/README]] — full skill system documentation and schema
- [[Claude/Toolbox/Agents/Profiles/README]] — agent profile schema and available profiles
- `hot.md` — always the freshest context on current session state
