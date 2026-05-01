---
tags:
  - type/reference
  - topic/agents
  - status/active
created: {{date}}
---

# Agent Profiles

Named, reusable agent configurations stored in the vault. Each profile defines an agent's role, the subagent type to use, which skills to invoke at each phase, and a prompt template.

## How to Invoke

During a coding session, say:

> "spin up the `feature-brief` agent for [topic]"
> "run context-deep-dive on [project]"
> "launch `[profile-name]` with context: [description]"

Claude reads the profile, executes preflight skills, assembles the prompt, and launches the agent.

---

## Profile Schema

```yaml
---
name: slug-name              # kebab-case identifier
label: Human Label           # displayed name
subagent_type: Explore       # general-purpose | Explore | Plan | claude-code-guide
model: sonnet                # sonnet | opus | haiku (optional, omit to inherit)
isolation: worktree          # optional — only for agents that write code
skills:
  preflight:                 # skills Claude invokes BEFORE launching agent
    - skill-name             # result is injected into agent prompt
  inflight:                  # skills the agent MAY invoke during its run
    - skill-name             # list here for agent awareness (general-purpose only)
  postflight:                # skills Claude invokes AFTER agent returns
    - skill-name             # e.g. mem-capture to archive the output
tags: [topic/example]
created: YYYY-MM-DD
---
```

---

## Skill Phases Explained

| Phase | Who runs it | Purpose |
|---|---|---|
| **preflight** | Main Claude | Gather context before launch — inject into prompt |
| **inflight** | The agent itself | Agent calls `Skill` tool during its run (general-purpose only) |
| **postflight** | Main Claude | Process or archive the agent's result |

Note: `Explore` and `Plan` agents are read-only and cannot invoke skills. Only `general-purpose` agents support inflight skill use.

---

## Profile Body Sections

Each profile should contain:

- **Role** — what this agent does in one paragraph
- **Working Context** — relevant paths, repos, services the agent should know
- **Preflight Notes** — how to use the preflight skill results in the prompt
- **Prompt Template** — the full prompt sent to the agent; use `{{variable}}` for values substituted at invocation time
- **Invocation Pattern** — the natural language phrase that triggers this profile

---

## Available Profiles

| Profile | Type | Skills | Purpose |
|---|---|---|---|
| [[feature-brief]] | Explore | mem-search (pre) | Pre-coding context brief for any topic/area |
| [[context-deep-dive]] | general-purpose | read-project-notes (pre), mem-capture (post) | Project re-orientation: vault + claude-memory synthesized |

---

## Promotion Path

```
Draft profile here
  → Use it a few times, refine the prompt template
  → If high-value, register as a skill in claude-memory
    (so it can be semantically triggered, not just named)
```
