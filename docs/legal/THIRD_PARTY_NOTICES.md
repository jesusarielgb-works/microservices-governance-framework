# Third-Party Notices
## Microservices Governance Framework (MGF)

This document maps every third-party concept, method or standard referenced in
the MGF to its source, its nature, and the original contribution of the author
on top of it within the framework.

Its purpose is to delimit with precision what belongs to third parties and what
constitutes the author's original expression — a requirement for any honest
registration of a work that builds on pre-existing knowledge.

> **Principle:** The ideas, methods and standards listed below are not objects of
> copyright in themselves. They belong to their authors or to the public domain.
> The MGF does not claim ownership of any of them. What the MGF protects is the
> **selection, coordination, arrangement and expression** of these elements in a
> new and original structure.

---

## Inventory of third-party concepts

### 1. Domain-Driven Design (DDD)

| Field | Value |
|-------|-------|
| **Source** | Eric Evans — *Domain-Driven Design: Tackling Complexity in the Heart of Software*, 2003 |
| **Nature** | Methodology / body of ideas (not protectable as such) |
| **MGF sections** | `02-domain/` |

**Author's original contribution:**
The MGF selects a subset of DDD concepts (Bounded Contexts, Entities, Value
Objects, Domain Events, Aggregates) and prescribes **how they are to be
documented** in a governance framework — which artefacts to produce, in what
format, with what naming and with what dependency on the product and architecture
sections. The application of DDD to a documentation governance model, with
explicit rules for how the domain layer feeds the architecture layer, is original.

---

### 2. C4 Model

| Field | Value |
|-------|-------|
| **Source** | Simon Brown — c4model.com |
| **Nature** | Diagramming notation / method (not protectable as such) |
| **MGF sections** | `08-uml/`, `08-uml/diagrams/source/c4-container-example.md` |

**Author's original contribution:**
The MGF selects C4 as the prescribed diagramming notation and integrates it into
the documentation governance: the example in `c4-container-example.md` is
written by the author, and the rules governing when and how C4 diagrams must be
produced (diagram-index, exports directory, Mermaid rendering) are original
editorial and structural decisions.

---

### 3. Hexagonal Architecture (Ports & Adapters)

| Field | Value |
|-------|-------|
| **Source** | Alistair Cockburn — "Hexagonal architecture", 2005 |
| **Nature** | Architectural pattern (not protectable as such) |
| **MGF sections** | `05-architecture/hexagonal-architecture.md`, `_stacks/` |

**Author's original contribution:**
The MGF takes the hexagonal pattern and translates it into **governance rules**:
how a service must be structured to be considered compliant with the framework,
what the folder layout looks like in each supported technology stack (Go, Java,
Node/TS, Python), and how the primary/secondary adapter distinction maps to the
service template. The written guide at `05-architecture/hexagonal-architecture.md`
is an original expression of the author.

---

### 4. GoF Design Patterns

| Field | Value |
|-------|-------|
| **Source** | Gamma, Helm, Johnson, Vlissides — *Design Patterns: Elements of Reusable Object-Oriented Software*, 1994 |
| **Nature** | Patterns catalogue (not protectable as such) |
| **MGF sections** | `05-architecture/pattern-guide.md` |

**Author's original contribution:**
The `pattern-guide.md` selects a subset of GoF patterns (Factory Method, Strategy,
Observer, Decorator, among others) and presents them alongside microservices-
specific patterns (Saga, CQRS, Circuit Breaker, Outbox, etc.) in a unified
catalogue. The selection, the arrangement in a single document, the "when to use
/ when NOT to use" structure for each pattern, and the domain-specific examples
are original written expression of the author.

---

### 5. Test-Driven Development (TDD)

| Field | Value |
|-------|-------|
| **Source** | Kent Beck — *Test-Driven Development: By Example*, 2002 |
| **Nature** | Development discipline / method (not protectable as such) |
| **MGF sections** | `11-quality/tdd-guide.md`, `11-quality/testing-strategy.md` |

**Author's original contribution:**
The TDD guide (`11-quality/tdd-guide.md`, 14.67 KB) is an original written
document that applies the TDD discipline to the microservices context of the
framework: red-green-refactor cycle, test pyramid adapted to microservices,
integration with the hexagonal architecture's port/adapter boundary, and rules
for what constitutes a passing quality gate in the framework's Definition of Done.

---

### 6. OpenAPI Specification

| Field | Value |
|-------|-------|
| **Source** | Linux Foundation / OpenAPI Initiative — openapis.org |
| **Nature** | Open standard (not protectable as such) |
| **MGF sections** | `07-api/contracts/openapi/` |

**Author's original contribution:**
The four YAML contracts in the repository (`_shared.yaml`, `_template-service.yaml`,
`api-gateway.yaml`, `auth-service.yaml`) are original written expressions of
the author using the OpenAPI standard as the encoding format. The structure of
`_shared.yaml` (reusable components), the template in `_template-service.yaml`
(prescriptive contract scaffold for any new service), and the two example
contracts are not reproductions of the standard — they are original works written
in compliance with it.

---

### 7. Conventional Commits

| Field | Value |
|-------|-------|
| **Source** | Conventional Commits specification — conventionalcommits.org |
| **Nature** | Open community convention (not protectable as such) |
| **MGF sections** | `00-governance/git-conventions.md` |

**Author's original contribution:**
The MGF adopts Conventional Commits as its prescribed commit format and extends
it with project-specific rules: which commit types are mandatory in the framework,
what goes in the scope field for each section type, and how commit messages
integrate with the Definition of Done. The governance rules in
`00-governance/git-conventions.md` are an original expression of the author on
top of the underlying convention.

---

## Summary table

| Concept | Source | Nature | Author's added expression |
|---------|--------|--------|--------------------------|
| DDD | Eric Evans (2003) | Method | How to document the domain layer in a governance framework |
| C4 Model | Simon Brown | Notation | Governance rules for diagram production; C4 example |
| Hexagonal Architecture | A. Cockburn (2005) | Pattern | Multi-stack governance rules; written guide |
| GoF Patterns | Gamma et al. (1994) | Catalogue | Selected subset + microservices patterns + original examples |
| TDD | Kent Beck (2002) | Discipline | TDD guide applied to microservices + quality gate rules |
| OpenAPI | Linux Foundation | Standard | Four original contracts + reusable template |
| Conventional Commits | Community | Convention | Extended governance rules for the framework |

---

*Prepared on 26 August 2026 as part of the DNDA registration process.*
*Technical-legal instrument. Registrability is determined by the DNDA.*
