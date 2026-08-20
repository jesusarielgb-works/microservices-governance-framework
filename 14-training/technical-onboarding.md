# Technical Onboarding

> Welcome to the team. This document is your guide for the first days.
> Goal: you can make your first commit in 3 days.
> If anything in this document is unclear or outdated, fix it yourself — that is your first contribution.

---

## Day 1 — Setup and context

### Morning: Access and environment

- [ ] Get access to the repository (confirm with the Tech Lead)
- [ ] Get access to Slack / Teams (team channel: `#[channel-name]`)
- [ ] Get access to Jira / Linear / GitHub Projects
- [ ] Configure your local environment following `10-devops/local-setup.md`
- [ ] Verify that `curl http://localhost:8080/health` responds `{"status": "ok"}`

### Afternoon: Read core documentation

Read in this order — each builds on the previous:

1. `00-sdd-guide.md` — How the team works (30 min)
2. `01-context/overview.md` — What we are building (20 min)
3. `02-domain/domain-map.md` — The business domain (30 min)
4. `05-architecture/overview.md` — How it is built (30 min)
5. `00-governance/git-conventions.md` — How we manage code (20 min)

### Day 1 meetings

- [ ] Meet & greet with the team
- [ ] 1:1 with the Tech Lead (30 min) — project context and your responsibilities
- [ ] Product demo (if there is a recording, watch it beforehand)

---

## Day 2 — Understand the domain

### Read domain and requirements documentation

- [ ] `02-domain/entities-and-rules.md` — Entities and business rules
- [ ] `02-domain/domain-events.md` — System events
- [ ] `04-requirements/user-stories.md` — HUs for the current sprint
- [ ] `01-context/glossary.md` — Project terms

### Explore the code

- [ ] Clone the repository for the service you will work on
- [ ] Read `09-microservices/services/[your-service]/README.md`
- [ ] Follow the folder structure — it should match `05-architecture/hexagonal-architecture.md`
- [ ] Run the tests: `npm run test:unit` — all must be green
- [ ] Run the service locally and test the main endpoints

### Day 2 meeting

- [ ] Domain walkthrough session with the Tech Lead or a senior developer (1 hour)
  - Ask them to explain the main business flow
  - Take notes on terms you do not know — add them to the glossary

---

## Day 3 — First contribution

### Your first task

The Tech Lead will assign you a small task labeled `good-first-issue`:
- It should be a small bug fix or documentation improvement
- The goal is to learn the workflow, not the complexity of the task

### TDD workflow for your first task

1. Read the ticket and the acceptance criteria
2. Write the test that verifies the criterion (`🔴 RED`)
3. Implement the minimum code to make the test pass (`🟢 GREEN`)
4. Refactor if necessary (`♻️ REFACTOR`)
5. Create the PR following the conventions in `00-governance/git-conventions.md`

### Checklist before opening the PR

- [ ] `npm run test:unit` passes (green)
- [ ] `npm run lint` passes (0 errors)
- [ ] The commit title follows `type(scope): description`
- [ ] The branch is named `feat/HU-XXX-description` or `fix/BUG-XXX-description`

---

## Week 1 — Go deeper

| Day | Activity |
|-----|----------|
| 4 | Code review (participate in the team's code review — observe first) |
| 5 | Participate in the Daily Standup with something concrete to report |
| 5 | Read `05-architecture/pattern-guide.md` — the patterns we use |
| 5 | Read `11-quality/tdd-guide.md` and `11-quality/testing-strategy.md` in full |

---

## Week 2 — Guided independence

- [ ] Complete your first HU independently
- [ ] Actively participate in a code review
- [ ] Read `07-api/README.md` and understand an OpenAPI contract for a service you use
- [ ] Participate in the sprint Retrospective

---

## Architecture: The 5 concepts you must understand first

Before writing code, understand these 5 concepts from the project's architecture:

### 1. Bounded Contexts
Each service corresponds to a Bounded Context of the domain.
See: `02-domain/domain-map.md`

### 2. Hexagonal Architecture
The domain is the center. Nothing from the framework enters the domain.
See: `05-architecture/hexagonal-architecture.md`

### 3. Ports and Adapters
Interfaces live in the domain. Implementations in infrastructure.
See: `05-architecture/hexagonal-architecture.md`

### 4. The flow of a request
```
HTTP Request
  → [Controller] (adapter in)
  → [UseCase] (application)
  → [Aggregate] (domain — business logic lives here)
  → [Repository] (port → adapter out)
  → Database
```

### 5. Domain events
Services communicate via events, not direct calls (when possible).
See: `02-domain/domain-events.md`

---

## Frequently asked questions (FAQ)

**Why don't I use `console.log` in tests?**
Because it pollutes the output. Use `logger.debug()` from the configured logger.

**Can I commit directly to `main`?**
No. Everything goes through a PR with at least 1 approval.

**Can I change the database schema directly?**
No. Every change goes as a versioned migration. See `06-data/models.md`.

**How do I know if my API change breaks consumers?**
Run the contract tests: `npm run test:contract`. If they pass, the contract is maintained.

**Where do I ask for help if I am stuck?**
1. Search this documentation first
2. Ask in `#[team-channel]`
3. Do not wait more than 1 hour before asking for help — the team's time is valuable

**What do I do if I find incorrect or outdated documentation?**
Fix it and open a PR. Documentation is code.

---

## Additional resources

| Resource | URL / Location | Purpose |
|----------|---------------|---------|
| Git guide | `00-governance/git-conventions.md` | Commit and branch conventions |
| Project ADRs | `05-architecture/decisions/` | Understand decisions made and why |
| Service runbook | `09-microservices/services/[your-service]/runbook.md` | Operate and troubleshoot |
| [External documentation] | [URL] | [purpose] |

---

## Correlations

- Technical setup → `10-devops/local-setup.md`
- TDD process → `11-quality/tdd-guide.md`
- Code conventions → `00-governance/git-conventions.md`
- Your service → `09-microservices/services/[name]/`
