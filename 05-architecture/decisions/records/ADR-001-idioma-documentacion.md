# ADR-001 — Documentation Language

| Field | Value |
|-------|-------|
| **ID** | ADR-001 |
| **Date** | 2026-08-20 |
| **Status** | Accepted |
| **Author** | Jesús Ariel González Bonilla |

---

## Context

The software industry standard — Stack Overflow, library documentation, technical articles,
frameworks, and open-source tooling — operates in English. Mixing languages across artifacts
(Spanish docs, English code, Spanish API descriptions) creates a cognitive translation
boundary that slows onboarding, increases errors in naming, and makes it harder to search
for help online.

A single, clear rule established from day one prevents inconsistencies: mismatched variable
names, Spanish comments in English code, and conflicting terminology between documentation
and implementation.

---

## Evaluated alternatives

### Alternative A — Everything in English (CHOSEN)
- **Pros:** Industry standard; easy to hire external developers; libraries and frameworks are in English; technical reference documentation is in English; eliminates the translation boundary between docs and code; GitHub, Stack Overflow, and AI tools all work best with English content
- **Cons:** May require slightly more effort from team members who are not native English speakers

### Alternative B — Everything in Spanish
- **Pros:** Natural communication with Spanish-speaking clients; business domain names preserved exactly
- **Cons:** Uncomfortable mix with language keywords (if, for, return, etc.); inconsistent with the library ecosystem; external contributors cannot participate

### Alternative C — Split by layer (discarded)
- **Pros:** Each artifact uses the most natural language for its audience
- **Cons:** Requires strict discipline and explicit rules; harder to explain to new team members; creates a permanent translation boundary between docs (Spanish) and code (English); domain terms accumulate two canonical names

---

## Decision

**Alternative A:** Use English for all documentation and code.

| Artifact | Language | Reason |
|----------|----------|--------|
| Variables, functions, classes in code | English | Consistency with libraries and frameworks |
| Table and column names in DB | English | Coherence with the code that maps them |
| Commits (Conventional Commits) | English | Established standard, readable on GitHub |
| Git branch names | English | Consistent with commits |
| Markdown documentation | English | Eliminates the translation boundary; searchable |
| OpenAPI contracts (descriptions) | English | Readable by any future contributor |
| End-user error messages | English (or localized) | Localization layer handles language at runtime |
| Internal system logs | English | Facilitates search in library documentation and alerts |
| ADRs and technical documentation | English | Single source of truth, no translation boundary |

---

## Consequences

**Positive:**
- A single language across all artifacts eliminates the cognitive translation boundary
- New team members have one clear rule from day 1
- External contributors and AI tools work without friction
- Domain terms have one canonical form (the English one in the code)

**Negative:**
- Team members who are less confident in English need to invest more initially
- Some business terms may require careful translation decisions

**Mitigation:**
- Maintain a domain glossary with the canonical English translation for each business term: `01-context/glossary.md`
- When a term has a debatable translation, document it in the glossary before using it in code

---

## References

- Team documentation conventions → `00-governance/documentation-rules.md`
- Domain term glossary → `01-context/glossary.md`
