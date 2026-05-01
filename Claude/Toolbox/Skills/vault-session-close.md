---
name: vault-session-close
label: Vault Session Close
tags:
  - type/skill
  - topic/vault
  - status/active
created: {{date}}
execution_type: multi
prerequisites: []
---

# vault-session-close

End-of-session vault maintenance skill. Runs when a session is wrapping up and work has been done in the vault or on a project.

**Invoke with:** "close the session", "wrap up", "end of session", or explicitly "run vault-session-close"

---

## Steps

### 1. Update hot.md (if no compact fired this session)

The PreCompact hook updates hot.md automatically whenever a compact fires. Only do this manually if the session is ending without a compact having occurred.

If needed, overwrite `hot.md` (vault root) with:

```
## Last Updated
YYYY-MM-DD

## Current Focus
[What is the primary thing being worked on right now]

## What Was Just Done
[Bulleted list of concrete changes made this session]

## Active Projects
[Current one-line status for each active project touched this session]

## Open Questions
[Any unresolved decisions or blockers surfaced this session]

## Next Session Picks Up From
[Exactly where to resume — be specific enough that no re-reading is needed]
```

### 2. Update Current State.md (if project work happened)

For each project touched this session, read its `Projects/_Active/<name>/Current State.md` and update to reflect:
- Current phase / milestone
- What was completed
- What is next
- Any decisions made

### 3. Append session log

Write a session summary to `Claude/Session-Logs/YYYY-MM-DD.md` (create file if it doesn't exist for today, append if it does).

Include: what was worked on, decisions made, outcomes, files changed.

### 4. Quick vault lint

Run the broken-frontmatter and stale-Current-State checks from `vault-lint` (checks 2 and 3 only — fast, no wikilink traversal):

```bash
find . -name "*.md" -not -path "*/.git/*" -not -path "*/.obsidian/*" | while read f; do
  head -1 "$f" | grep -q "^---$" || echo "missing frontmatter: $f"
done

find Projects/_Active -name "Current State.md" -mtime +30 | while read f; do
  echo "stale (>30d): $f"
done
```

Surface any findings to the user before proceeding. Skip full vault-lint unless issues are found.

### 5. Rebuild knowledge graph (incremental, if graphify installed)

```bash
graphify . --update
```

This re-extracts only changed files and merges them into `graphify-out/graph.json`. Skip this step if graphify is not installed — errors are non-fatal.

### 6. Commit and push vault

```bash
git add -A
git commit -m "session: YYYY-MM-DD — <one-line summary>"
git push
```

### 7. Remind user

Output: "Vault pushed. Remember to `git pull` on other machines before editing in Obsidian."

---

## Notes

- Skip step 1 if a compact already fired this session (hot.md was auto-updated by the PreCompact hook)
- Skip step 2 if the session was research/planning only with no project state changes
- Step 4 (lint) is a quick check — if it finds nothing, say so and move on
- Step 5 (graphify) is incremental — cheap if few files changed. If it errors, skip and proceed to step 6
- If git push fails due to conflicts, surface the error rather than silently failing
