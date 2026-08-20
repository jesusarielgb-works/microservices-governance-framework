# 03 — Product Definition

> **What is this?** The answer to "what are we going to build?". It is not "how" — that comes in
> architecture. Here the validated problem, product vision, and build plan are defined.

## Why this section exists

Without a clear product definition:
- The team builds features nobody asked for
- Scope grows out of control (scope creep)
- There is no way to know whether the project was successful

This section is the contract between the team and stakeholders about **what will be built and why**.

---

## What is here and how to fill it in

### `problem-framing.md` ⭐ (Start here)
Articulates the problem before proposing solutions.
**Fill in:** who has the problem, exactly what pain, evidence of the problem, how they solve it today.

**Format:**
```markdown
## The problem
**Who has it?** [Profile of the affected user]
**What problem do they have?** [Description of the pain, specific]
**When does it occur?** [Situation that triggers the problem]
**What is the impact?** [Concrete consequence: time, money, frustration]
**How do they solve it today?** [Current workaround and why it is insufficient]

## Why it is worth solving
[Justification for the value of building this system]
```

### `discovery-brief.md`
Findings from user research.
**Fill in:** interviews conducted, insights found, assumptions validated and invalidated.

### `vision.md` ⭐
The product's north star in 1-2 sentences.
**Fill in:** format "For [user], who [need], [system name] is a [product type]
that [key benefit]. Unlike [alternative], our product [differentiator]."

### `roadmap.md`
Delivery plan over time.
**Fill in:** milestones per quarter/sprint, which features go into each phase.

**Format:**
```markdown
## Phase 1 — MVP (Sprint 1-3)
- [Critical feature 1]
- [Critical feature 2]

## Phase 2 — Iteration (Sprint 4-6)
- [Improvements based on feedback]
```

### `product-backlog.md` ⭐
Prioritized list of everything that must be built.
**Fill in:** using the `_template-backlog.md` template. Order by user value.

### `_template-prd.md`
Complete Product Requirements Document.
**Use when:** you need to formalize requirements for an external stakeholder or academic delivery.

### `_template-discovery-brief.md`
Template for documenting user research.

### `_template-problem-framing.md`
Structured template for framing the problem.

### `_template-backlog.md`
Template for initial backlog user stories.

---

## User Story format

```markdown
## HU-[SERVICE]-[NNN]: [Title]
**As** [user role]
**I want** [action they want to perform]
**So that** [benefit they receive]

### Acceptance criteria
- [ ] AC1: Given [context], when [action], then [expected result]
- [ ] AC2: ...

### Technical notes
[Constraints or implementation considerations]

**Estimation:** [SP]  **Priority:** [High/Medium/Low]
```

---

## Correlations with other sections

| This section feeds... | Why |
|-----------------------|-----|
| `04-requirements/` | Backlog HUs are formalized as requirements |
| `02-domain/` | Problem framing reveals domain entities |
| `15-project-control/technical-backlog.md` | Technical debt identified during definition |

---

## Questions this section must answer

- What problem exactly are we solving?
- What does product success look like?
- What do we build first and why?
- What do we NOT build in this cycle?
