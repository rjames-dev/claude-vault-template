---
name: read-project-notes
label: Read Project Notes
tags:
  - type/skill
  - topic/vault
  - status/active
created: {{date}}
execution_type: read
output_format: structured
parameters:
  - name: project_name
    description: Project folder name (e.g. "My App", "Research Tool")
    required: true
prerequisites: []
---

# read-project-notes

Reads all key notes for a named project and returns structured context including extracted key terms suitable for claude-memory queries.

Used as a preflight skill in agent profiles that need project context before querying claude-memory or starting work.

**Invoke with:** preflight in agent profiles, or directly — "read project notes for \<project name\>"

---

## Steps

### 1. Locate the project folder

Check in order:
1. `Projects/_Active/{{project_name}}/`
2. `Projects/_Parked/{{project_name}}/`
3. `Projects/_Archive/{{project_name}}/`

If not found, report clearly: "Project '{{project_name}}' not found in _Active, _Parked, or _Archive."

Note the lifecycle status (Active / Parked / Archived) — include in output.

### 2. Read available notes

Read each file if it exists:

| File | Required |
|------|----------|
| `Current State.md` | Yes — core status |
| `Architecture.md` | No — include if present |
| `Decisions Log.md` | No — include last 10 entries if present |
| `Open Questions.md` | No — include all if present |

Do not read other files unless specifically relevant.

### 3. Return structured output

Format the output exactly as shown below so agent profiles can inject it cleanly via `{{vault_context}}`:

```
## Project: {{project_name}}
**Lifecycle:** Active | Parked | Archived
**Last Updated:** YYYY-MM-DD (from Current State.md frontmatter or last session log)

### Current Status
[2-4 sentence summary from Current State.md — current phase, what's done, what's next]

### Architecture Summary
[3-5 sentence summary from Architecture.md if exists, otherwise omit this section]

### Recent Decisions
[Last 5-10 entries from Decisions Log.md as a bulleted list, if exists]
[Omit section if no Decisions Log]

### Open Questions
[All entries from Open Questions.md as a bulleted list, if exists]
[Omit section if no Open Questions]

### Key Terms for Memory Search
[8-15 specific, project-vocabulary terms extracted from the above.
Focus on: technology names, component names, architectural concepts,
bug names, decision topics — the vocabulary a claude-memory semantic
search would need to surface relevant sessions.
Format as comma-separated list.]
```

---

## Notes

- If Current State.md is missing, note the gap and use whatever notes exist
- Key Terms should be specific and technical — avoid generic words like "bug" or "feature"
- Include component/file names when they appear in the notes — these are the best search anchors
