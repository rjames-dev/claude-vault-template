---
tags:
  - type/reference
  - topic/mcp
  - status/active
created: {{date}}
---

# MCP Servers

Model Context Protocol servers expose tools to Claude Code. Configured in `~/.claude.json` under `mcpServers`.

## MCP Configuration Pattern

```json
// ~/.claude.json
{
  "mcpServers": {
    "server-name": {
      "command": "node",
      "args": ["/path/to/server.js"],
      "env": {
        "SOME_VAR": "value"
      }
    }
  }
}
```

## Active MCP Servers (add yours here)

| Server | Purpose | Connection |
|--------|---------|------------|
| <!-- server name --> | <!-- purpose --> | <!-- stdio / sse --> |

## claude-memory (if installed)

**Purpose:** Session memory, search, and analytics across all conversations.
**Connection:** stdio → `node <claude-memory-path>/mcp-server/src/server.js`

Tools available:

| Tool | Purpose |
|------|---------|
| `search_memory` | Semantic search across past conversations |
| `search_knowledge` | Search curated knowledge base |
| `search_decisions` | Find architectural decisions |
| `search_exact_phrase` | Exact phrase search |
| `get_snapshot` | Retrieve a specific snapshot by ID |
| `get_timeline` | Session timeline and activity |
| `get_file_activity` | File mention heatmap |
| `get_project_stats` | Per-project activity stats |
| `analyze_bugs` | Bug pattern analysis |

## Useful MCP Servers to Explore

| Server | Purpose |
|--------|---------|
| `filesystem` | Direct file access with structured tools |
| `github` | GitHub API (issues, PRs, repos) |
| `postgres` | Direct DB querying via natural language |
| `brave-search` | Web search |
| `puppeteer` | Browser automation |
| `obsidian-mcp-server` | Live read/write access to Obsidian vault |

## Notes

- MCP tools appear as deferred tools in Claude Code — use `ToolSearch` to fetch their schema before the first call in a session
- Tools run with the permissions of the process that launched Claude Code
- MCP servers configured in `~/.claude.json` are available in every session; project-scoped servers go in `<project>/.claude.json`
