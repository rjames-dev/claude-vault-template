---
name: vault-lint
label: Vault Lint
tags:
  - type/skill
  - topic/vault
  - status/active
created: {{date}}
execution_type: bash+read
prerequisites: []
---

# vault-lint

Vault health check. Scans for structural issues, stale content, and missing metadata. Run periodically (monthly recommended) or before a major session.

**Invoke with:** "lint the vault", "check vault health", "find broken links", "run vault-lint"

---

## Checks

### 1. Broken wikilinks

Find `[[wikilinks]]` in vault notes that don't resolve to an existing file.

```bash
# Find all wikilinks
grep -r '\[\[' . \
  --include="*.md" \
  --exclude-dir=".git" \
  --exclude-dir=".obsidian" \
  -h | grep -oP '\[\[([^\]|#]+)' | sed 's/\[\[//' | sort -u
```

For each extracted link, check if a matching `.md` file exists anywhere in the vault.

### 2. Missing frontmatter

Find `.md` files that don't contain a YAML frontmatter block (no `---` at line 1).

```bash
find . -name "*.md" \
  -not -path "*/.git/*" \
  -not -path "*/.obsidian/*" | while read f; do
    head -1 "$f" | grep -q "^---$" || echo "$f"
done
```

### 3. Stale Current State files

Find `Current State.md` files in `_Active/` projects not modified in the last 30 days.

```bash
find Projects/_Active -name "Current State.md" -mtime +30
```

These may indicate projects that are actually stalled and should move to `_Parked/`.

### 4. Inbox items older than 7 days

```bash
find Inbox -not -name ".gitkeep" -mtime +7
```

### 5. Orphaned notes

Find notes with no incoming wikilinks (excluding root-level and index files). These may be candidates for archival or consolidation.

For each note, search for its filename (without .md) as a wikilink target across all other notes.

### 6. Notes missing `status:` frontmatter field

Notes with frontmatter but no `status:` field — useful to identify notes that were never properly classified.

---

## Output Format

Report grouped by check:

```
## Vault Lint Report — YYYY-MM-DD

### Broken Wikilinks (N found)
- [[Link Name]] referenced in: path/to/note.md

### Missing Frontmatter (N found)
- path/to/note.md

### Stale Current State Files (N found)
- Projects/_Active/ProjectName/Current State.md (last modified: YYYY-MM-DD)

### Inbox Items > 7 Days (N found)
- Inbox/filename.md (age: N days)

### Orphaned Notes (N found)
- path/to/note.md

### Missing Status Field (N found)
- path/to/note.md
```

End with: `Total issues: N`

---

## Notes

- Don't auto-fix issues — report and let the user decide
- Broken links and stale Current State files are the highest-priority issues
- Orphan check is approximate — Obsidian may surface more accurate backlink counts
