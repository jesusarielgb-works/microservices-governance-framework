# User Stories — Backlog

> **What to fill in here:** The product's User Story backlog.
> Each HU uses the standard format with Acceptance Criteria in Given/When/Then.
> Refined (Ready) HUs go to the sprint. Unrefined ones are epics or ideas.

---

## Backlog status

| Cut | Sprint | Total HUs | Refined | In progress | Completed |
|-----|--------|-----------|---------|-------------|-----------|
| Cut 1 | Sprint 1-2 | [N] | [N] | [N] | [N] |
| Cut 2 | Sprint 3-4 | [N] | [N] | [N] | [N] |

---

## Epics

| ID | Epic | Description |
|----|------|-------------|
| EP-001 | [Epic name] | [Brief description of the epic's objective] |
| EP-002 | [Name] | [Description] |

---

## User Stories

### HU-001 — [Descriptive name] {#HU-001}

**Epic:** EP-00X

> **As** [user role]
> **I want** [action / feature]
> **so that** [benefit / value received]

**Acceptance Criteria:**

```gherkin
Scenario 1: [Scenario name — happy path]
  Given [initial context]
  When  [user action]
  Then  [expected result]
  And   [additional condition if applicable]

Scenario 2: [Scenario name — edge case / error]
  Given [context]
  When  [action]
  Then  [error result, e.g.: validation message is shown]
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [1 / 2 / 3 / 5 / 8 / 13] |
| Priority | [Must Have / Should Have / Could Have] |
| Target sprint | Sprint [N] |
| Assigned to | [Name] |
| Status | [Backlog / Ready / In Progress / Done] |
| Dependencies | [HU-00X, HU-00Y] |
| Affected service(s) | [service-name] |

---

### HU-002 — [Descriptive name] {#HU-002}

**Epic:** EP-00X

> **As** [role]
> **I want** [action]
> **so that** [benefit]

**Acceptance Criteria:**

```gherkin
Scenario 1: [Happy path]
  Given [context]
  When  [action]
  Then  [result]

Scenario 2: [Error case]
  Given [context]
  When  [invalid action]
  Then  error "[error code]" is shown with message "[message]"
```

| Field | Value |
|-------|-------|
| Story Points | [N] |
| Priority | [Must Have] |
| Target sprint | Sprint [N] |
| Status | [Backlog] |

---

## Rules for writing HUs

### 1. The role matters
Do not write "As a user" — that says nothing. Use the specific role:
```
✓ As a system administrator
✓ As a registered customer
✓ As an inventory operator
✗ As a user
✗ As a person
```

### 2. The benefit justifies the work
The "so that" must describe a business benefit, not redescribe the action:
```
✓ so that I can manage my orders without calling support
✗ so that I can see my orders (this only describes the feature)
```

### 3. ACs are verifiable
Each AC must be verifiable manually or automatable as a test:
```
✓ Then the system shows a message "Order #123 confirmed"
✓ Then the confirmation email arrives in less than 30 seconds
✗ Then the system works well (not verifiable)
✗ Then the user is satisfied (not verifiable)
```

### 4. One HU = one unit of value
If the HU has 15 ACs, it is probably 3 HUs.
The team must be able to complete it in one sprint (maximum 2 weeks).

---

## Ready-to-copy HU template

```markdown
### HU-00X — [Name] {#HU-00X}

**Epic:** EP-00X

> **As** [role]
> **I want** [action]
> **so that** [benefit]

**Acceptance Criteria:**

\```gherkin
Scenario 1: [name]
  Given [context]
  When  [action]
  Then  [result]
\```

| Field | Value |
|-------|-------|
| Story Points | |
| Priority | |
| Target sprint | |
| Status | Backlog |
| Dependencies | |
```

---

## Correlations

- Full template with DoD checklist → `04-requirements/_template-hu.md`
- Non-functional requirements → `04-requirements/non-functional.md`
- Traceability matrix → `04-requirements/traceability-matrix.md`
- API contracts derived from these HUs → `07-api/contracts/openapi/`
