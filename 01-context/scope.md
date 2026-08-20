# System Scope

> **Why this document exists:** Scope prevents scope creep and aligns expectations.
> It is equally important to define what the system does NOT do as what it does.
> Review this document at the start of each planning cycle.

---

## In Scope

What the system **DOES build and maintain**:

### MVP Features

| # | Feature | Description | Responsible service |
|---|---------|-------------|---------------------|
| 1 | [Feature A] | [Brief description] | [service-name] |
| 2 | [Feature B] | [Brief description] | [service-name] |
| 3 | [Feature C] | [Brief description] | [service-name] |

### Included integrations

| External system | Integration type | Purpose |
|----------------|-----------------|---------|
| [System A] | REST API / Webhook / SDK | [purpose] |
| [System B] | SFTP / Database | [purpose] |

### Environments being built

| Environment | Purpose |
|-------------|---------|
| Local | Development on the developer's machine |
| Development (dev) | Continuous integration and development testing |
| Staging | Pre-production, PO acceptance testing |
| Production | Production environment |

---

## Out of Scope

What the system **does NOT build** in this version and why:

| # | What is out of scope | Reason | Future version? |
|---|---------------------|--------|----------------|
| 1 | [Feature X] | [Out of budget / Not MVP / Uses external system] | Yes — H2 2024 |
| 2 | [Integration with Y] | [Provider has no public API yet] | Pending provider |
| 3 | [Module Z] | [Another team builds it] | N/A |

### What another system / team handles (and why not us)

| Feature | Who builds it | Why not us |
|---------|--------------|-----------|
| [SSO Authentication] | Central IAM team | Reuse existing implementation |
| [Financial reports] | BI system / Analytics team | Outside the core domain |

---

## Scope assumptions

> These assumptions are taken to be true. If they change, the scope must be renegotiated.

| # | Assumption | Consequence if false |
|---|-----------|---------------------|
| 1 | External system [X] has an available REST API | We would have to build the integration differently |
| 2 | Users use [language / device / etc.] | The UX design would change |
| 3 | Initial data volume is < [N] records | The database strategy might change |

---

## Constraints

| Type | Description |
|------|-------------|
| **Time** | [MVP must be ready in X weeks / by date Y] |
| **Budget** | [N development hours / X USD of infrastructure] |
| **Technology** | [Must use the corporate stack: Java + PostgreSQL] |
| **Regulatory** | [Must comply with X regulation / certification] |
| **Team** | [N developers available] |

---

## External dependencies

| Dependency | Team / Provider | Required date | Status |
|-----------|----------------|--------------|--------|
| API of [System X] | [Team name] | [date] | 🟢 Available |
| Credentials for [Provider Y] | [Contact] | [date] | 🟡 In progress |
| [Infrastructure Z] | DevOps | [date] | 🔴 Pending |

---

## How to update the scope

The scope can change, but the change has a process:

1. Document the proposed change in this file
2. Evaluate the impact on schedule and effort
3. Obtain approval from the Product Owner and Tech Lead
4. Update the roadmap in `03-product/vision.md`
5. Create or update HUs in `04-requirements/user-stories.md`

---

## Correlations

- Vision and roadmap → `03-product/vision.md`
- Term glossary → `01-context/glossary.md`
- System overview → `01-context/overview.md`
- Scope-related risks → `15-project-control/risks.md`
