# 01 — Project Context

> **What is this?** The "why" of the system. Anyone new must be able to read this
> folder and understand what problem the project solves, what it includes, and what it does NOT include.

## Why this section exists

Before designing anything, the team needs to agree on:
- What problem are we solving?
- For whom?
- What is in scope and what is out of scope?
- What does each term we use mean?

Without this, each team member works with different assumptions and the project fragments.

---

## What is here and how to fill it in

### `overview.md` ⭐
Executive description of the system in maximum 1 page.
**Fill in:** system name, problem it solves, main users, key technologies,
current status (under construction / in production / legacy).

**Suggested format:**
```markdown
## What is [System Name]?
[2-3 sentences: what it is and what it's for]

## Problem it solves
[The user's pain before this system]

## Main users
- [Role 1]: [what they do in the system]
- [Role 2]: [what they do in the system]

## Technology stack
- Backend: [language/framework]
- Database: [engine]
- Infrastructure: [Docker/K8s/Cloud]
```

### `scope.md` ⭐
System boundaries: what it does and what it does NOT do.
**Fill in:** explicit list of what is INSIDE and OUTSIDE the MVP scope and future versions.
This prevents scope creep (the system that grows without control).

**Format:**
```markdown
## In scope (MVP)
- [Feature 1]
- [Feature 2]

## Out of scope (MVP)
- [What we deliberately do NOT do]

## Candidates for future versions
- [What might come later]
```

### `glossary.md` ⭐
Dictionary of the project domain.
**Fill in:** all technical and business terms used in the project, with their exact definition.
If two people define "client" differently, the system will have bugs.

**Format:**
```markdown
| Term | Definition | Synonyms | Notes |
|------|-----------|----------|-------|
| [Term] | [Precise definition in the context of this system] | [if any] | [if applicable] |
```

### `_template-project-profile.md`
Project technical sheet for internal records.
**Fill in:** when the project is formalized (official name, tech lead, dates, stakeholders).

### `_template-scope-declaration.md`
Formal scope declaration template for presentations or deliverables.

---

## Correlations with other sections

| If you change this... | Also review... |
|-----------------------|----------------|
| The problem described in `overview.md` | Product vision in `03-product/vision.md` |
| The scope in `scope.md` | Requirements in `04-requirements/`, PRD in `03-product/` |
| A term in `glossary.md` | Every document where that term appears |

---

## Recommended fill order

1. `overview.md` — 30 minutes with the full team
2. `scope.md` — 1 hour of discussion (the most valuable thing you can do at the start)
3. `glossary.md` — grows throughout the project, start with 10 key terms

---

## Questions this section must answer

- What does this system exist for?
- Who are the users?
- What does the system NOT do?
- What does [term X] mean in this project?
