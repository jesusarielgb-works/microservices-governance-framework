# Traceability Matrix

> Traceability connects every line of code to its business justification.
> It allows answering: "Why does this function exist?" and "Which HU covers this part of the system?"
> It also identifies: unimplemented requirements and code without a requirement (possible technical debt).

---

## How to use this matrix

```
Requirement → HU → Test Case → Implementation → Service

If a requirement has no HU: it is not planned
If a HU has no test case: it has no completeness criterion
If a test case has no implementation: there is test technical debt
If there is code without an HU: possible gold-plating or bug introduced without a story
```

---

## FR → HU → Test → Service matrix

| FR ID | FR Description | HU(s) | Tests that verify it | Service | Status |
|-------|---------------|-------|---------------------|---------|--------|
| FR-001 | [System allows user registration] | HU-001 | `auth.register.spec.ts` | auth-service | ✅ Done |
| FR-002 | [System allows authentication] | HU-002 | `auth.login.spec.ts` | auth-service | ✅ Done |
| FR-003 | [System allows creating orders] | HU-003, HU-004 | `order.create.spec.ts` | order-service | 🟡 In progress |
| FR-004 | [System sends email notifications] | HU-010 | `notifications.spec.ts` | notification-service | 🔴 Pending |
| FR-00X | [description] | [HU-00X] | [test file] | [service] | [status] |

---

## NFR → Validation matrix

| NFR ID | Description | How it is validated | Tool | Status |
|--------|-------------|-------------------|------|--------|
| NFR-001 | P95 < 300ms | Load test in staging | k6 | ✅ Validated |
| NFR-002 | 99.9% availability | SLO monitoring | Grafana | 🟡 Monitoring |
| NFR-004 | JWT authentication | Security contract test | Postman + OWASP ZAP | 🔴 Pending |

---

## Inverse traceability: HU → FR

| HU | Title | FR(s) it implements | Sprint |
|----|-------|---------------------|--------|
| HU-001 | [User registration] | FR-001 | Sprint 1 |
| HU-002 | [Login] | FR-002 | Sprint 1 |
| HU-003 | [Create basic order] | FR-003 | Sprint 2 |

---

## Status legend

| Status | Meaning |
|--------|---------|
| ✅ Done | Implemented, tested, and in production |
| 🟡 In progress | Under development in the current sprint |
| 🔴 Pending | In the backlog, not started |
| ⏸ Blocked | Has an external blocker |
| ❌ Cancelled | Removed from scope |

---

## Identified gaps (requirements without coverage)

> This section is updated automatically or manually when reviewing the matrix.
> A gap is: an FR without an HU, or an HU without a test, or a test without implementation.

| Gap type | Description | Required action | Owner | Date |
|----------|-------------|----------------|-------|------|
| FR without HU | FR-00X has no associated HU | Create HU in the backlog | Product Owner | [date] |
| HU without test | HU-00X has no acceptance test | Add test before the next sprint | QA / Dev | [date] |

---

## How to maintain this matrix

1. When an HU is created: add the row in the FR → HU → Test → Service section
2. When a test is written: note the file in the "Tests that verify it" column
3. When an HU is completed: change the status to ✅
4. At each Sprint Planning: review gaps and assign actions

---

## Correlations

- User Stories → `04-requirements/user-stories.md`
- Non-Functional Requirements → `04-requirements/non-functional.md`
- Testing strategy → `11-quality/testing-strategy.md`
- DoD that determines when an HU is Done → `00-governance/definition-of-done.md`
