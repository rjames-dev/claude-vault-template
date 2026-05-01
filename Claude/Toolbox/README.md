---
tags:
  - type/index
  - topic/claude-code
created: {{date}}
---

# Claude Toolbox

Living reference for Claude Code capabilities, patterns, and techniques.
Updated as new things are discovered, tested, and refined.

## Promotion Path

```
Toolbox (research + reference)
  → .claude/ folder (active, installed, running)
```

When a pattern proves genuinely useful in practice, it graduates:
- Commands → `~/.claude/commands/<name>.md`
- Hooks → `~/.claude/settings.json` hooks array
- Skills → vault-native skill files (this folder) or claude-memory skills system
- MCP servers → `~/.claude.json` mcpServers config

## Contents

| Folder | What lives here |
|--------|----------------|
| [[Agents/README]] | Subagent types, patterns, when to use each |
| [[Commands/README]] | Slash commands — built-in, custom, installed |
| [[Hooks/README]] | Hook events, wired hooks, patterns |
| [[MCP/README]] | MCP servers, tools, integration patterns |
| [[Skills/README]] | Vault-native skill system — skill files and schema |
| [[System-Guide]] | Full system guide: vault + claude-memory two-layer architecture |
| [[Environment-Strategy]] | Dynamic workbench — keeping .claude/ lean and project-relevant |
