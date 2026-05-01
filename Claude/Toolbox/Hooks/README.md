---
tags:
  - type/reference
  - topic/hooks
  - status/active
created: {{date}}
---

# Hooks

Hooks are shell commands that Claude Code executes automatically in response to specific lifecycle events. Configured in `~/.claude/settings.json`.

## Available Hook Events

| Event | When it fires |
|-------|--------------|
| `PreCompact` | Before Claude Code compacts the conversation context |
| `PostCompact` | After compaction completes |
| `PreToolUse` | Before any tool call executes — can return `"defer"` to delay |
| `PostToolUse` | After any tool call completes |
| `Notification` | When Claude Code sends a notification |
| `PermissionDenied` | When a tool call is denied by permissions |
| `InstructionsLoaded` | When CLAUDE.md / rules files are loaded |
| `Stop` | When Claude finishes a response |

## Key Wired Hooks (for this vault system)

### PreCompact — hot.md update
```json
{
  "type": "command",
  "command": "echo '{\"systemMessage\": \"BEFORE COMPACTING: Overwrite hot.md with current session state (focus, what was done, open questions, where to pick up). This is the primary session cache — be specific and complete.\"}'"
}
```
**Purpose:** Injects a system message instructing Claude to write current session state to `hot.md` before context is compacted. Ensures the resume file is always current.

### PreCompact — claude-memory capture
```json
{
  "type": "command",
  "command": "<claude-memory-path>/hooks/auto-capture-precompact.py",
  "timeout": 10
}
```
**Purpose:** Captures a snapshot of the current session into the claude-memory Postgres DB before compaction. Powers session history and decision search. Requires claude-memory to be running.

## Hook Configuration Pattern

Hooks are arrays — multiple hooks can run on the same event:

```json
{
  "hooks": {
    "PreCompact": [
      {
        "hooks": [
          { "type": "command", "command": "..." },
          { "type": "command", "command": "...", "timeout": 10 }
        ]
      }
    ]
  }
}
```

> [!warning] Merge, don't replace
> When adding a new hook, always read `settings.json` first and merge into the existing hooks array. Writing the file fresh overwrites existing hooks.

## Patterns Worth Exploring

- **PostCompact** — auto-push vault to git after compaction
- **PreToolUse** — log all file edits to an audit trail
- **PostToolUse** — trigger a lint/test run after code edits
- **Stop** — run a script when Claude finishes a response

## Location

`~/.claude/settings.json` (global) or `<project>/.claude/settings.json` (project-scoped).
