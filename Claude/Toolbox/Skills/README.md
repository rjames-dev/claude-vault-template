---
tags:
  - type/reference
  - topic/skills
  - status/active
created: {{date}}
---

# Skills

Two complementary skill systems, serving different purposes.

---

## System 1 — Vault-Native Skills (portable)

Skill definitions stored as markdown files in this folder. Claude reads them at invocation time via the `Read` tool — no installation, no DB, no machine setup. Portable via `git clone`.

### How They Work

1. User says "run vault-ingest" or "lint the vault" or invokes via agent profile preflight
2. Claude reads the skill file from this folder
3. Executes the steps described in the file using available tools
4. Returns output to the conversation

### Skill File Schema

```yaml
---
name: skill-slug          # kebab-case, unique identifier
label: Human Label        # displayed name
execution_type:           # read | bash | mcp | multi
  # read  — Claude uses Read/Grep/Glob tools
  # bash  — Claude runs shell commands via Bash tool
  # mcp   — Claude calls MCP tools directly
  # multi — combination of the above
output_format:            # structured | prose | raw
  # structured — key/value or table (best for preflight injection into agent prompts)
  # prose      — narrative (best for human reading)
  # raw        — unprocessed tool output
parameters:               # named inputs this skill accepts (optional)
  - name: param_name
    description: what it does
    required: true | false
prerequisites: []         # other skills that must run first
tags: []
created: YYYY-MM-DD
---
```

### Vault-Native Skills

| Skill | Execution | Output | Purpose |
|-------|-----------|--------|---------|
| [`vault-session-close`](vault-session-close.md) | multi | prose | End-of-session: update hot.md (fallback), Current State, session log, git push |
| [`vault-ingest`](vault-ingest.md) | read | prose | Process Inbox/ items or raw material into vault notes |
| [`vault-lint`](vault-lint.md) | bash+read | structured | Health check: broken links, missing frontmatter, stale notes |
| [`read-project-notes`](read-project-notes.md) | read | structured | Read all key notes for a project; extract key terms for memory search |

### Use in Agent Profiles

Vault-native skills are the building blocks for agent profiles in `Claude/Toolbox/Agents/Profiles/`. Profiles declare which skills to run as preflight (before agent launch), injecting results into the agent prompt.

```yaml
skills:
  preflight:
    - read-project-notes   # result injected as {{vault_context}}
  postflight:
    - mem-capture          # archive the agent's output
```

---

## System 2 — claude-memory DB Skills (semantic trigger)

Skills registered in the `skills_agents` Postgres table. Matched semantically to user requests via Ollama embeddings. Fire automatically when a conversation matches a trigger phrase.

### How They Work

1. Skill registered in DB with trigger phrases
2. Trigger phrases embedded via Ollama
3. When user request matches a trigger semantically, the skill fires
4. Skill's script/prompt is injected into the conversation via `Skill` tool

> [!note] This system requires claude-memory running locally. See [[../System-Guide]] for setup.

---

## Comparison

| | Vault-Native | claude-memory DB |
|--|--|--|
| Storage | Markdown file in this folder | Postgres `skills_agents` table |
| Portability | Git clone only | Machine-local (requires DB + Ollama) |
| Invocation | Named ("run vault-lint") or agent preflight | Semantic trigger auto-match |
| Analytics | None | Usage tracked in DB |
| Best for | Repeatable vault workflows, agent building blocks | Contextual auto-detection |

---

## Promotion Path

```
Vault-native skill (this folder)
  → use it a few times, refine
  → if semantic auto-trigger is valuable: register in claude-memory DB
  → if explicit slash invocation is valuable: add to ~/.claude/commands/
```
