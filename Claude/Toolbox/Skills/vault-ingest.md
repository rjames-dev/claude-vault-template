---
name: vault-ingest
label: Vault Ingest
tags:
  - type/skill
  - topic/vault
  - status/active
created: {{date}}
execution_type: read
prerequisites: []
---

# vault-ingest

Processes any raw material into a proper vault note: correct folder placement, YAML frontmatter, heading structure, summary paragraph, and wikilinks to related notes.

**Invoke with:** "ingest this", "add this to the vault", "process the inbox", "turn this into a vault note"

---

## Input Sources

- A file in `Inbox/` (most common)
- Pasted text in the conversation
- A URL (fetch content first, then process)
- A screenshot or image (extract text/content, then process)

---

## Steps

### 1. Understand the material

Read or receive the source content. Determine:
- What type of content is it? (research, project note, reference, decision, log)
- What project or topic does it belong to?
- Does a related note already exist that this should be merged into?

### 2. Choose the destination

| Content type | Destination |
|-------------|-------------|
| Research / background knowledge | `Research/<topic>/` |
| Active project note | `Projects/_Active/<name>/` |
| Technical pattern / reusable insight | `Knowledge/` |
| Template | `Templates/` |
| Temporary / unclear | `Inbox/` (leave for now, flag to user) |

### 3. Write the note

Structure:
```markdown
---
tags:
  - <appropriate tags>
created: YYYY-MM-DD
status: draft | active
source: <url or "conversation" or "inbox">
---

# <Descriptive Title>

<One-paragraph summary of what this note contains and why it matters.>

---

## <Section 1>
...

## <Section 2>
...

## Related
- [[Related Note 1]]
- [[Related Note 2]]
```

Tag guidelines:
- Type: `type/research`, `type/reference`, `type/decision`, `type/log`
- Topic: `topic/<area>` (e.g., `topic/claude`, `topic/docker`, `topic/finance`)
- Project: `project/<name>` if project-specific

### 4. Clean up the source

- If source was an `Inbox/` file: delete it after the note is created
- If source was pasted text: confirm with user before discarding
- If source was a URL: keep the URL in the `source:` frontmatter field

### 5. Update log.md

Append a one-line entry to `log.md` (vault root):
```
- YYYY-MM-DD: Ingested "<note title>" → <destination path>
```

---

## Notes

- Prefer merging into an existing note over creating a new one when the content is closely related
- If the right destination is ambiguous, ask the user before writing
- Use wikilinks generously — connecting notes is more valuable than a standalone note
