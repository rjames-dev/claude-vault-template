---
tags:
  - type/template
  - topic/projects
created: {{date}}
---

# Project Protocol

Standard folder structure and file conventions for new projects. Copy into `Projects/_Active/<name>/` when starting a project.

---

## Folder Structure

```
Projects/_Active/<Project Name>/
  Current State.md      ← required; Claude reads this at session start
  Architecture.md       ← optional; system design, component map
  Decisions Log.md      ← optional; architecture decisions, newest first
  Open Questions.md     ← optional; unresolved issues and blockers
```

---

## Current State.md Template

```markdown
---
tags:
  - project/active
  - type/current-state
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: building | planning | blocked | complete
---

# Current State — <Project Name>

## What This Is
One paragraph: what the project does, who it's for, why it exists.

## Where We Are
Current phase, what's working, what's in progress.

## What's Done
- [x] Completed thing
- [x] Another completed thing

## What's Still Missing
- [ ] Next thing to build
- [ ] Open question to resolve

## Immediate Next Steps
1. First thing to do
2. Second thing to do

## Key Decisions Made
- Why we chose X over Y
- Why the architecture is structured this way
```

---

## Decisions Log.md Template

```markdown
---
tags:
  - project/active
  - type/decisions-log
created: YYYY-MM-DD
---

# Decisions Log — <Project Name>

*Architecture and implementation decisions, newest first.*

## YYYY-MM-DD
- Chose X over Y because Z
- Decided not to implement W — added complexity with no clear benefit
```

---

## Open Questions.md Template

```markdown
---
tags:
  - project/active
  - type/open-questions
created: YYYY-MM-DD
---

# Open Questions — <Project Name>

*Unresolved questions, blockers, and decisions pending.*

- [ ] Question or blocker — context about why it's open
```

---

## Registering a New Project

1. Create the folder: `Projects/_Active/<name>/`
2. Write `Current State.md` (use template above)
3. Add to `CLAUDE.md` active projects table
4. Add to `index.md` under `_Active/`
5. Add to `Projects/Roadmap.md`
