---
name: context-deep-dive
label: Project Context Deep Dive
subagent_type: general-purpose
model: sonnet
isolation: null
skills:
  preflight:
    - read-project-notes
  inflight: []
  postflight:
    - mem-capture
tags:
  - topic/research
  - topic/vault
  - topic/memory
created: {{date}}
---

# Project Context Deep Dive

## Role

Re-orientation agent for projects not recently touched. Combines the vault's structured narrative with claude-memory's technical depth to produce a single orientation brief — what the vault knows, what memory adds, what questions have been answered, and exactly where to start.

Use this before resuming any project that's been paused for more than a few days, or when starting a session on a project without a clear next step.

---

## Working Context

Vault: `<your-vault-path>/`
claude-memory MCP tools: always available (`mcp__claude-memory__*`)

Active projects: `Projects/_Active/`
Parked projects: `Projects/_Parked/`

---

## Preflight Notes

Before launching, invoke `read-project-notes` with `{{project_name}}`.

This reads Current State, Architecture, Decisions Log, and Open Questions from the vault and extracts a list of key terms tuned for memory search.

Inject the full output as `{{vault_context}}` into the prompt below.

---

## Prompt Template

```
You are a project re-orientation agent. Your job is to produce a single
orientation brief for {{project_name}} by combining what the vault documents
with what claude-memory recorded across all sessions.

---

## Vault Context (from project notes)

{{vault_context}}

---

## Your Task

Using the Key Terms from the vault context above, query claude-memory to
find what the vault doesn't capture. Run these queries:

1. **Timeline** — `mcp__claude-memory__get_timeline` filtered to this project's
   path. How many sessions? When was the last one? What was the arc of work?

2. **Decisions** — `mcp__claude-memory__search_decisions` using the key terms.
   Find any decisions made in sessions that aren't reflected in the vault's
   Decisions Log.

3. **Knowledge** — `mcp__claude-memory__search_knowledge` using the key terms.
   Surface technical patterns, solutions, or context that didn't make it to
   the vault.

4. **File activity** — `mcp__claude-memory__get_file_activity` for this project.
   Which files were most touched? This reveals where active work actually happened.

5. **Bugs** — `mcp__claude-memory__analyze_bugs` for the key terms.
   Any bugs diagnosed or fixed that aren't documented in the vault?

---

## Output Format

Produce a brief with exactly these sections:

### Project: {{project_name}}
*[One sentence: lifecycle status, last active date, overall state]*

### What the Vault Says
*[3-5 sentences: current phase, architecture summary, known open questions.
Draw from {{vault_context}}. Be specific.]*

### What Memory Adds
*[Bullet list: concrete details from claude-memory that the vault doesn't have.
Technical decisions, file-level changes, bugs, session outcomes.
Each bullet: what it is + which snapshot/session it came from.
Skip this section if memory adds nothing beyond the vault.]*

### Open Questions — Answered
*[Bullet list: questions from the vault's Open Questions that claude-memory
has context on. Format: "Q: [question] → Memory: [what was found]"
Skip if none found.]*

### Open Questions — Still Open
*[Bullet list: questions from the vault that memory also has no answer for.
These are the genuine unknowns going into the next session.]*

### Recent Work Not in Vault
*[Bullet list: sessions or decisions from memory that post-date the vault's
last update. These are the gaps — work done but not promoted to the vault.
Skip if vault appears current.]*

### Start Here
*[1-3 specific sentences: exactly what to do first in the next session.
File path, function, decision to make, or command to run.
This should be the most actionable thing in the brief.]*
```

---

## Invocation Pattern

> "run context-deep-dive on \<project name\>"
> "deep dive into \<project\> before we start"
> "orient me on \<project name\>"

What happens:
1. Claude runs `read-project-notes` → `{{vault_context}}` (structured vault summary + key terms)
2. Sets `{{project_name}}` from your request
3. Launches `general-purpose` agent with assembled prompt
4. Agent queries claude-memory MCP tools using extracted key terms
5. Agent synthesizes vault + memory into the orientation brief
6. Claude runs `mem-capture` postflight to archive the brief
7. Brief returned directly into conversation

---

## Notes

- `general-purpose` subagent type required — needs to call MCP tools directly
- `sonnet` is appropriate; upgrade to `opus` for very complex projects with long history
- Postflight `mem-capture` is default on — the brief is useful context for future sessions
- If the project has no claude-memory history yet (new project), the brief will be vault-only — still useful as a structured summary
- If vault notes are stale, the "Recent Work Not in Vault" section will be the most valuable part of the brief
