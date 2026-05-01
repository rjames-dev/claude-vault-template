---
tags:
  - type/reference
  - topic/commands
  - status/active
created: {{date}}
---

# Commands

Slash commands are reusable prompt templates invoked with `/command-name`. Custom commands live as markdown files in `~/.claude/commands/`.

## Built-in Commands

Key commands by category:

| Command | Purpose |
|---------|---------|
| `/clear` | Clear conversation history |
| `/compact [instructions]` | Manually trigger context compaction |
| `/context` | Visualize context usage with optimization suggestions |
| `/memory` | View and manage memory files |
| `/cost` | Show token usage for current session |
| `/model [model]` | Switch model (e.g. `/model opus`) |
| `/effort [level]` | Set model effort: low, medium, high, max, auto |
| `/fast [on\|off]` | Toggle fast mode |
| `/plan [desc]` | Enter plan mode |
| `/ultraplan <prompt>` | Draft a plan for review before execution |
| `/diff` | Interactive diff viewer for uncommitted changes |
| `/stats` | Daily usage, session history, streaks |
| `/usage` | Plan limits and rate limit status |
| `/mcp` | Manage MCP server connections |
| `/hooks` | View hook configurations |
| `/skills` | List available skills |
| `/tasks` | List and manage background tasks |

## Custom Commands (add yours here)

Place `.md` files in `~/.claude/commands/` — the filename becomes the slash command.

| Command | Purpose |
|---------|---------|
| <!-- /your-command --> | <!-- description --> |

## Creating a Custom Command

Create a `.md` file in `~/.claude/commands/`:

```markdown
# Command Name

Description of what this command does.

## Instructions

The prompt text that gets injected when /name is invoked.
Can reference $ARGUMENTS for any text passed after the command name.
```

The filename becomes the slash command: `my-command.md` → `/my-command`.

## Vault-as-Commands-Source Pattern

You can symlink this folder as the project-level commands directory so vault command files are available as slash commands:

```bash
ln -sf <vault-path>/Claude/Toolbox/Commands <project-path>/.claude/commands
```

Command files placed here are immediately available when Claude Code is launched from that project. The vault becomes the source of truth — commands travel with the repo.
