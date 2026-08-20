# Documentation Rules

> These rules determine how documentation is written, organized, and maintained in this project.
> Documentation that does not follow these rules may be rejected in code review.

---

## Core principle

> **"Documentation is code. If it's not up to date, it's broken."**

Every HU that modifies system behavior MUST include updating the affected documents.
The DoD requires it.

---

## Language

| Artifact | Language |
|----------|----------|
| Source code (variables, functions, classes) | English |
| Code comments | English |
| Commits | English (Conventional Commits) |
| Branch names | English |
| Markdown documentation | English |
| OpenAPI contracts (descriptions) | English |
| Error messages returned to frontend | English (or localized) |
| Internal system logs | English |

> **Rule:** Once the language for each category is chosen, it is binding for the entire project.
> Mixing languages in the same category is grounds for PR rejection.

---

## File structure

```
Each section has its README.md that explains the folder's purpose.
Content documents use kebab-case.md (e.g.: domain-map.md, risk-register.md).
Templates are prefixed with _ to appear first (e.g.: _template-hu.md, _template-adr.md).
ADRs are numbered sequentially: ADR-001-short-title.md.
```

---

## What to document and what NOT to

### DO document

| What | Where |
|------|-------|
| Non-obvious architectural decisions | `05-architecture/decisions/records/ADR-NNN.md` |
| Business rules and domain invariants | `02-domain/entities-and-rules.md` |
| API contracts for each service | `07-api/contracts/openapi/[service].yaml` |
| Data model changes | `06-data/models.md` |
| Operational procedures | `13-operations/` |
| Identified risks | `15-project-control/risks.md` |

### DO NOT document

- What the code already says clearly (do not repeat in comments what can be read in the code)
- Temporary decisions or experiments that will be reverted
- Implementation details of external libraries (those have their own documentation)
- Change history (that's what git log is for)

---

## Owners per section

| Section | Owner | Review frequency |
|---------|-------|-----------------|
| `00-governance/` | Tech Lead | Start of each sprint |
| `02-domain/` | Tech Lead + PO | When the domain changes |
| `04-requirements/` | Product Owner | Each sprint |
| `05-architecture/` | Tech Lead | Each design decision |
| `07-api/contracts/` | Service-owning developer | Each API change |
| `09-microservices/` | Service-owning developer | Each release |
| `13-operations/` | DevOps / On-call | After each incident |
| `15-project-control/` | Tech Lead | Weekly review |

---

## Document format

### Headings
- `# H1` — only one per file; it is the title
- `## H2` — main sections
- `### H3` — subsections
- Do not use H4 or deeper; if you need it, the document has too much hierarchy

### Tables
Use tables for comparisons, registers, and matrices. Do not use tables for simple lists.

### Code
Always use code blocks with the language specified:
````
```typescript
const x = 1;
```
````

### Template instructions
Blocks marked `> [!NOTE] INSTRUCTIONS` indicate the document is an unfilled template.
Remove them when the document is complete.

---

## Update process

1. The developer identifies which documents their change affects
2. Updates the documents together with the code (same PR)
3. The reviewer verifies the documentation is up to date
4. If the PR closes a HU that had API impact → the OpenAPI contract must be updated

---

## Correlations

- Git conventions → `00-governance/git-conventions.md`
- Per-microservice documentation standard → `00-governance/microservices-documentation.md`
- Definition of Done (docs as part of DoD) → `00-governance/definition-of-done.md`
