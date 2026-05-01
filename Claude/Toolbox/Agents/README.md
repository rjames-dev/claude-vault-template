---
tags:
  - type/reference
  - topic/agents
  - status/active
created: {{date}}
---

# Agents

Subagents are specialized Claude instances launched via the `Agent` tool. Each type has a specific role, toolset, and use case.

## Available Agent Types

### `general-purpose`
Full tool access. Best for complex multi-step tasks, open-ended research, or anything that might require multiple rounds of searching and reasoning.
**Use when:** task is too open-ended for a simple Grep/Glob, or requires autonomous decision-making across many steps.

### `Explore`
Read-only tools (no Edit/Write). Specialized for codebase exploration. Supports thoroughness levels: `quick`, `medium`, `very thorough`.
**Use when:** you need to understand how something works, find files by pattern, or answer architectural questions without modifying anything.

### `Plan`
Software architect agent. Designs implementation plans, identifies critical files, considers trade-offs. Returns step-by-step plans.
**Use when:** about to implement something non-trivial and want to align on approach before writing code.

### `claude-code-guide`
Specialized for questions about Claude Code itself — CLI features, hooks, slash commands, MCP servers, settings, IDE integrations, Agent SDK, API.
**Use when:** user asks "how do I..." or "can Claude..." about Claude tooling. Check if one is already running before spawning a new one.

## Key Patterns

### Parallel agents
When tasks are independent, launch multiple agents in one message:
```
Agent(Explore, "find all API endpoints") + Agent(Explore, "find all tests")
```

### Background agents
For long-running tasks where you don't need the result immediately:
```
Agent(general-purpose, "...", run_in_background=true)
```

### Worktree isolation
For agents that will make code changes, use `isolation: "worktree"` to give them an isolated git branch. Worktree is auto-cleaned if no changes are made.

### Foreground vs background
- **Foreground:** you need results before proceeding (research, planning)
- **Background:** genuinely independent work (validation, testing)

## Agent Profiles

Named agent configurations stored in the vault. Each profile combines a subagent type, a prompt template, and a skill pipeline (preflight / inflight / postflight). Profiles let you invoke a configured agent by name during a session rather than describing it from scratch.

**Location:** [[Profiles/README]]

**Invocation:** "spin up the `feature-brief` agent for [topic]"

| Profile | Type | Purpose |
|---|---|---|
| [[Profiles/feature-brief]] | Explore | Pre-coding context brief — mem-search + codebase explore |
| [[Profiles/context-deep-dive]] | general-purpose | Project re-orientation: vault + claude-memory synthesized |

---

## When NOT to Use Agents

- Finding a specific file → use `Glob` directly
- Finding a class/function definition → use `Grep` directly
- Reading a known file path → use `Read` directly
- Simple 1-2 file searches → use tools directly

Agents add latency and context overhead. Default to direct tools first.
