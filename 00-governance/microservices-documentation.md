# Per-Microservice Documentation Standard

> Defines exactly what documents each microservice must have, who writes them,
> when they are created, and when they must be updated. Non-compliance blocks the merge.

---

## Required structure for each service

Each microservice lives in `09-microservices/services/NN-service-name/` and MUST have:

```
09-microservices/services/NN-service-name/
├── README.md         ⭐ REQUIRED from Sprint 1
├── data-model.md     ⭐ REQUIRED before creating migrations
├── events.md         ⭐ REQUIRED if the service emits/consumes events
├── decisions.md      🔵 RECOMMENDED — internal technical decisions of the service
└── runbook.md        🟢 REQUIRED before first deploy to staging
```

And its OpenAPI contract in:
```
07-api/contracts/openapi/service-name.yaml   ⭐ REQUIRED if it exposes REST endpoints
```

---

## README.md — Service technical sheet

**When to create it:** At the start of the sprint where the service is created
**Owner:** Developer assigned to the service
**Update when:** Responsibility, ports, or dependencies between services change

Minimum content (use `_template/service/README.md`):

| Section | What it must say |
|---------|-----------------|
| Responsibility | One sentence: what it does and what data it is the authoritative owner of |
| Architecture location | Port, repository, DB engine, who it communicates with |
| Responsibilities (what it DOES) | List of concrete responsibilities |
| Out of scope (what it does NOT do) | What it delegated and to whom |
| How to run locally | Exact commands, must work |
| Related documents | Links to the other files of the service |

---

## data-model.md — Service data model

**When to create it:** Before the first migration script
**Owner:** Developer assigned to the service
**Update when:** A table/collection is created or modified

Minimum content:
- ER diagram (Mermaid) of the service's tables
- Description of each table with its columns, types, constraints, and purpose
- Justification of the chosen DB engine (PostgreSQL, MongoDB, Redis, etc.)
- Migration strategy (Flyway, Liquibase, or manual scripts)

**Rule:** A field whose reason for existing is not obvious MUST have a comment in the diagram.

---

## events.md — Service event catalog

**When to create it:** When the service publishes or consumes its first domain event
**Owner:** Developer assigned to the service
**Update when:** An event is added, modified, or removed

Minimum content:
- Table of published events: name, topic/exchange, when it is emitted, schema
- Table of consumed events: name, which service it comes from, what action it triggers
- Payload schema (can reference the OpenAPI or Avro contract)

See standard event structure: `02-domain/domain-events.md`

---

## decisions.md — Service technical decisions

**When to create it:** When the team makes a non-obvious technical decision about the service
**Owner:** Whoever made the decision
**Update when:** A new decision is made or a previous one is revoked

Recommended format: miniADR (without the full rigor of an architecture ADR):
```markdown
### Decision: [short name]
**Date:** [date]
**Context:** [what problem was being solved]
**Decision:** [what was decided]
**Consequences:** [known trade-offs]
```

---

## runbook.md — Service operations manual

**When to create it:** Before the first deploy to staging
**Owner:** Responsible developer + DevOps
**Update when:** A new operational issue is discovered or a procedure changes

Minimum content:
- How to verify the service is healthy (health check, key metrics)
- Known symptoms and their causes: "If you see X, the problem is Y, the solution is Z"
- How to perform a service rollback
- How to run DB migrations
- Configured alerts and what to do when they fire

---

## OpenAPI Contract

**When to create it:** Before implementing the service's first endpoint (API-first)
**Owner:** Developer assigned to the service
**Update when:** An endpoint is added, modified, or removed

**API-First Rule:** The contract is written BEFORE the code. Contract tests validate
that the code fulfills the contract, not the other way around.

Use the template: `07-api/contracts/openapi/_template-service.yaml`

---

## How to add a new microservice

1. Copy `09-microservices/_template/service/` → `09-microservices/services/NN-name/`
2. Update `09-microservices/service-catalog.md` with the new service's entry
3. Update `09-microservices/dependency-map.md` (or create it if it doesn't exist)
4. Copy `07-api/contracts/openapi/_template-service.yaml` → `07-api/contracts/openapi/service-name.yaml`
5. Create a PR with at least the README.md and the sketched API contract

---

## Correlations

- Service template → `09-microservices/_template/service/`
- Service catalog → `09-microservices/service-catalog.md`
- API contracts → `07-api/contracts/openapi/`
- General documentation rules → `00-governance/documentation-rules.md`
