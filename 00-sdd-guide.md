# SDD Guide — Software Design Documentation

> This document explains the **approach and methodology** that governs this entire repository.
> Read it first if you are new to the project or to the methodology.

---

## What is SDD?

**Software Design Documentation** is a development approach where design documentation
**precedes and guides** implementation. It is not about documenting what was already built —
it is about designing on paper before writing code.

```
Traditional:   Code  →  Documentation (if it ever happens)
SDD:           Documentation  →  Code  →  Updated documentation
```

### The 3 SDD principles

1. **Design before code:** A reviewed and approved design document is the prerequisite
   for starting implementation. If it is not documented, it does not exist yet.

2. **Living documentation:** Documentation is updated with every change. An outdated
   document is technically incorrect — it is code with bugs, written in prose.

3. **Traceability:** Every line of code has a requirement that justifies it. Every
   requirement has a test case. Every test case has a result.

---

## SDD workflow phases

<svg viewBox="0 0 700 500" xmlns="http://www.w3.org/2000/svg" style="max-width:100%;font-family:monospace">
  <rect width="700" height="500" fill="#0d1117" rx="10"/>

  <!-- Phase 1 -->
  <rect x="30" y="20" width="640" height="80" rx="8" fill="#1c2029" stroke="#388bfd" stroke-width="1.5"/>
  <text x="50" y="44" fill="#388bfd" font-weight="bold" font-size="13">PHASE 1 — DISCOVERY</text>
  <text x="50" y="62" fill="#8b949e" font-size="12">01-context → 02-domain → 03-product</text>
  <text x="50" y="80" fill="#6e7681" font-size="11">Understand the problem before proposing solutions.</text>
  <text x="50" y="93" fill="#6e7681" font-size="10">Deliverables: overview, problem-framing, domain-map, vision</text>

  <!-- Arrow + gate -->
  <line x1="350" y1="100" x2="350" y2="125" stroke="#484f58" stroke-width="2" marker-end="url(#a)"/>
  <rect x="220" y="120" width="260" height="22" rx="4" fill="#161b22" stroke="#30363d" stroke-width="1"/>
  <text x="350" y="135" fill="#8b949e" text-anchor="middle" font-size="10">← Stakeholder validation</text>

  <!-- Phase 2 -->
  <rect x="30" y="145" width="640" height="80" rx="8" fill="#1c2029" stroke="#3fb950" stroke-width="1.5"/>
  <text x="50" y="169" fill="#3fb950" font-weight="bold" font-size="13">PHASE 2 — DEFINITION</text>
  <text x="50" y="187" fill="#8b949e" font-size="12">04-requirements → 05-architecture → 06-data → 07-api</text>
  <text x="50" y="205" fill="#6e7681" font-size="11">Define WHAT and HOW before implementing.</text>
  <text x="50" y="218" fill="#6e7681" font-size="10">Deliverables: user stories with ACs, ADRs, data models, OpenAPI contracts</text>

  <!-- Arrow + gate -->
  <line x1="350" y1="225" x2="350" y2="250" stroke="#484f58" stroke-width="2" marker-end="url(#a)"/>
  <rect x="180" y="245" width="340" height="22" rx="4" fill="#161b22" stroke="#30363d" stroke-width="1"/>
  <text x="350" y="260" fill="#8b949e" text-anchor="middle" font-size="10">← Architecture Review Board</text>

  <!-- Phase 3 -->
  <rect x="30" y="270" width="640" height="80" rx="8" fill="#1c2029" stroke="#d2a8ff" stroke-width="1.5"/>
  <text x="50" y="294" fill="#d2a8ff" font-weight="bold" font-size="13">PHASE 3 — DETAILED DESIGN</text>
  <text x="50" y="312" fill="#8b949e" font-size="12">08-uml → 09-microservices → 12-ux-ui</text>
  <text x="50" y="330" fill="#6e7681" font-size="11">Design each system component in detail.</text>
  <text x="50" y="343" fill="#6e7681" font-size="10">Deliverables: diagrams, runbooks, wireframes, design system</text>

  <!-- Arrow + gate -->
  <line x1="350" y1="350" x2="350" y2="375" stroke="#484f58" stroke-width="2" marker-end="url(#a)"/>
  <rect x="200" y="370" width="300" height="22" rx="4" fill="#161b22" stroke="#30363d" stroke-width="1"/>
  <text x="350" y="385" fill="#8b949e" text-anchor="middle" font-size="10">← Sprint kickoff (Sprint Planning)</text>

  <!-- Phase 4 -->
  <rect x="30" y="395" width="640" height="85" rx="8" fill="#1c2029" stroke="#ffa657" stroke-width="1.5"/>
  <text x="50" y="419" fill="#ffa657" font-weight="bold" font-size="13">PHASE 4 — IMPLEMENTATION (TDD) + OPERATIONS</text>
  <text x="50" y="437" fill="#8b949e" font-size="12">Code guided by design documents · tests first · 10-devops · 13-operations · 14-training</text>
  <text x="50" y="455" fill="#6e7681" font-size="11">Code + tests + updated documentation</text>
  <text x="50" y="472" fill="#6e7681" font-size="10">Gate: Code review + QA + Go/No-Go before production</text>

  <defs>
    <marker id="a" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto">
      <path d="M0,0 L0,6 L8,3 z" fill="#484f58"/>
    </marker>
  </defs>
</svg>

---

## Recommended fill-in order for a new project

### Week 1 — Context and Domain
1. `01-context/overview.md` — What are we building?
2. `01-context/scope.md` — What are we NOT building?
3. `02-domain/domain-map.md` — What are the bounded contexts?
4. `02-domain/entities-and-rules.md` — What are the entities and rules?
5. `02-domain/domain-events.md` — What events occur?
6. `01-context/glossary.md` — First draft with 15–20 terms

### Week 2 — Product and Requirements
7. `03-product/problem-framing.md` — Validate the problem
8. `03-product/vision.md` — Define the north star
9. `04-requirements/user-stories.md` — First 10–15 MVP user stories
10. `04-requirements/non-functional.md` — NFRs with measurable metrics

### Week 2–3 — Architecture
11. `05-architecture/overview.md` — C4 diagram and service list
12. `05-architecture/decisions/records/ADR-001-*.md` — First architecture decision
13. `06-data/models.md` — Initial data schema per service
14. `07-api/contracts/openapi/` — API contracts (contract-first)

### Week 3–4 — Detailed design
15. `09-microservices/service-catalog.md` — Full service catalog
16. `09-microservices/services/01-[service]/` — README + data-model + events per service
17. `08-uml/diagrams/source/` — Sequence diagram for critical flows
18. `12-ux-ui/navigation-map.md` + wireframes for main screens

### Sprint 1 onwards — TDD Implementation
19. `10-devops/local-setup.md` — Do this before writing any code
20. `11-quality/testing-strategy.md` — Do this before the first sprint
21. Implementation following TDD flow (see `11-quality/tdd-guide.md`)
22. `13-operations/observability.md` — Before the first deploy

---

## Review gates

| Gate | When | What is reviewed | Who approves |
|------|------|-----------------|--------------|
| **Domain Review** | After `02-domain/` | Did we capture the domain correctly? | Domain Expert + Tech Lead |
| **Architecture Review** | After ADRs and `05-architecture/` | Does the architecture satisfy NFRs? | Tech Lead + Team |
| **API Review** | Before implementing each service | Is the contract correct and consistent? | API consumers |
| **Sprint Demo** | At the end of each sprint | Does the software meet the ACs? | Product Owner |
| **Go/No-Go** | Before production | DoD, DoR, NFRs all satisfied? | Tech Lead + PO |

---

## The living documents rule

```
If the code changed but the document did not → The document is BROKEN
If the document says X but the code does Y  → The document is a LIE
```

**Responsibility:** whoever opens the PR that changes system behavior
is responsible for updating the corresponding documentation.
