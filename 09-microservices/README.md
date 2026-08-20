# 09 — Microservices

> **What is this?** The individual documentation for each microservice in the system.
> This is the most active section of the repository — it is updated with every significant change.

## Structure of each microservice

Each service has its own folder in `services/` with this structure:

```
services/
└── 01-service-name/
    ├── README.md              ← Description, responsibility, how to run it
    ├── data-model.md          ← The service's own data model
    ├── events.md              ← Events it publishes and consumes
    ├── decisions.md           ← Design decisions specific to the service
    ├── runbook.md             ← How to operate the service in production
    └── components/            ← If the service has sub-components
        └── [component]/
            ├── README.md
            └── contract.md
```

**Use `_template/service/` as a starting point for each new service.**

---

## Cross-cutting documents (apply to ALL services)

### `service-catalog.md` ⭐ (Start here)
Registry of all microservices in the system.
**Fill in:** when you define microservices. Update when you add/remove/rename.

**Format:**
```markdown
| # | Service | Responsibility | Local port | Repo | Status |
|---|---------|----------------|------------|------|--------|
| 01 | iam | Authentication and authorization | 8001 | [repo] | ✅ In production |
| 02 | reference-data | Master data for the system | 8002 | [repo] | 🚧 In development |
```

### `service-boundary-rules.md` ⭐
The rules that define where each service ends.
**Fill in:** criteria for deciding if something belongs to service A or B. These rules prevent
services from overlapping or leaving responsibilities in limbo.

**Include:**
- How do we decide boundaries? (by domain, by team, by data lifecycle)
- What to do when something "could go in either one"?
- The "database" rule: if two things share a DB, they are the same service or there is a design error

### `communication-patterns.md` ⭐
How services communicate with each other.
**Fill in:** what uses REST (synchronous), what uses events (asynchronous), why, concrete examples.

**Patterns to document:**
- Request/Response (REST/gRPC): when to use it, when not to
- Events/Messages (RabbitMQ/Kafka): when to use it, how delivery is guaranteed
- API Gateway: what goes through it, what does not
- Service Discovery: how services find each other

### `event-catalog.md` ⭐
Catalog of all system events.
**Fill in:** name, payload, who publishes, who consumes, on which topic/exchange.

**Format:**
```markdown
| Event | Payload (key fields) | Published by | Consumed by | Topic/Exchange |
|-------|---------------------|--------------|-------------|----------------|
| UserCreated | userId, email, role | iam | actors, audit | users.events |
```

### `data-ownership-matrix.md` ⭐
Who is the authoritative owner of each piece of data.
**Fill in:** for each business entity, which service has the "official version".

**Format:**
```markdown
| Entity | Owner service | Others that have a copy | How it is synchronized |
|--------|--------------|------------------------|----------------------|
| User | iam | actors (basic data) | UserCreated event |
```

### `dependency-map.md`
Dependency map between services.
**Fill in:** graph (can be ASCII) showing which service depends on which.
Red alert: if the graph has cycles → there is a design problem.

### `storage-and-documents.md`
How the system handles file storage.
**Fill in:** which service handles files, where they are stored (S3, MinIO, local), naming policy.

### `service-readiness-checklist.md`
Checklist that every service must meet before going to production.

---

## Correlations with other sections

| This section is fed by... | And feeds into... |
|---------------------------|-------------------|
| `02-domain/domain-map.md` → bounded contexts | One per context |
| `05-architecture/overview.md` → defined services | README of each service |
| `07-api/contracts/openapi/` → service contract | `services/[service]/` implements it |
| `06-data/models.md` → data model | `services/[service]/data-model.md` |
| `services/[service]/runbook.md` | `13-operations/` consolidates them |

---

## Service numbering

Number services to indicate implicit dependencies:
services with lower numbers tend to be more fundamental.

Suggested convention:
- `01` = IAM / Security (all depend on this)
- `02` = Reference / master data
- `03-0N` = Domain services
- `0N+1` = Support services (documents, notifications, audit)
