# 04 — Requirements

> **What is this?** The formal specification of what the system must do.
> Functional: what it does. Non-functional: how well it does it.

## Why this section exists

Requirements are the contract between the team and the client/stakeholder.
Without them:
- There is no way to verify whether the system is complete
- Scope changes have no baseline for comparison
- Tests have no success criterion

---

## Types of requirements

### Functional (FR)
Describe **what the system does**: functions, behaviors, data transformations.
*Example: "The system must allow the user to recover their password via email."*

### Non-functional (NFR)
Describe **how it does it**: quality, performance, availability, security.
*Example: "The system must respond in less than 200ms for 95% of requests."*

NFRs are usually harder to meet than FRs and are ignored more frequently. **They are equally important.**

---

## What is here and how to fill it in

### `functional.md` ⭐
List of all the system's functional requirements.
**Fill in:** numbered, with the module/service they belong to, source (originating HU), priority.

**Format:**
```markdown
| ID | Module | Description | Source (HU) | Priority |
|----|--------|-------------|------------|---------|
| FR-001 | [Service] | The system must [do something] | HU-XXX-001 | High |
```

### `non-functional.md` ⭐
Quality, performance, and technical constraint requirements.
**Fill in:** by category (performance, availability, security, scalability, etc.)

**Format:**
```markdown
## Performance
| ID | Requirement | Metric | How to verify |
|----|------------|--------|--------------|
| NFR-001 | Response time | p95 < 200ms | Load test with K6 |

## Availability
| ID | Requirement | Metric | How to verify |
|----|------------|--------|--------------|
| NFR-010 | Uptime | 99.9% monthly | Production monitoring |

## Security
| ID | Requirement | Description |
|----|------------|-------------|
| NFR-020 | Authentication | JWT with 1-hour expiration |
```

### `user-stories.md`
Formalized user stories (coming from the `03-product/` backlog).
**Fill in:** with As/I want/So that format + verifiable acceptance criteria.

### `traceability-matrix.md` ⭐
Table that connects: HU → Requirement → Test case.
**Fill in:** when you have requirements and tests defined. Allows coverage verification.

**Format:**
```markdown
| HU | FR/NFR | Description | Test case | Status |
|----|--------|-------------|----------|--------|
| HU-IAM-001 | FR-001 | Login with email | TC-001 | ✅ |
```

### `_template-hu.md`
Template for a complete User Story with acceptance criteria.

### `_template-nfr.md`
Template for specifying non-functional requirements with their verification metrics.

---

## Correlations with other sections

| This section feeds... | Why |
|-----------------------|-----|
| `05-architecture/` | Performance/availability NFRs guide architectural decisions |
| `11-quality/testing-strategy.md` | Each FR must have at least one test case |
| `09-microservices/` | FRs are grouped by responsible service |
| `07-api/` | Integration FRs → endpoints in API contracts |
| `15-project-control/risks.md` | Very demanding NFRs usually generate technical risks |

---

## Common mistakes to avoid

❌ **"The system must be fast"** → Not measurable. Better: "p95 < 200ms"

❌ **"The system must be secure"** → Not verifiable. Better: "Authentication with JWT, tokens expire in 1h"

❌ Writing requirements that describe the solution instead of the problem.

✅ A good requirement is: **specific, measurable, achievable, relevant, and verifiable**.

---

## Questions this section must answer

- What must the system do for each type of user?
- With what speed, availability, and security?
- Which requirement originates each test case?
- Are all requirements covered by tests?
