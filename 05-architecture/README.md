# 05 — Architecture

> **What is this?** The system's design decisions: how it is organized, why,
> what alternatives were evaluated, and how it is deployed. ADRs are the treasure of this section.

## Why this section exists

A system's architecture is the set of decisions that are hard to change later.
Documenting them has three benefits:
1. **New team members** understand the system without having to ask everything from scratch
2. **The team** does not repeat already-resolved discussions
3. **Years later**, everyone remembers why each decision was made

---

## What is here and how to fill it in

### `overview.md` ⭐ (Start here)
High-level view of the complete system.
**Fill in:** C4 Level 1 (System) and Level 2 (Container) diagram, list of microservices with
each one's responsibility, how they communicate (sync/async), technologies per layer.

**Recommended format:**
```markdown
## Architecture diagram
[ASCII diagram, Mermaid, or reference to image in assets/]

## Microservices
| Service | Responsibility | Technology | DB |
|---------|--------------|------------|-----|
| [name] | [what it does] | [stack] | [engine] |

## Communication patterns
- Sync: [what uses REST between which services]
- Async: [what uses events/messages between which services]
- Gateway: [how external requests arrive]
```

### `deployment.md` ⭐
How the system is deployed in each environment.
**Fill in:** infrastructure diagram, what goes in Docker/K8s, network configuration, hardware requirements.

### `cross-cutting.md`
Concerns that apply to all microservices.
**Fill in:** standard logging, distributed tracing, centralized configuration, feature flags,
error handling, retry policies.

### `pattern-guide.md`
Catalog of design patterns used in the project.
**Fill in:** for each pattern: name, when to use it, when NOT to use it, concrete example from the project.

### `security-threat-model.md`
Security threat analysis of the system.
**Fill in:** using the STRIDE methodology: Spoofing, Tampering, Repudiation, Information Disclosure,
Denial of Service, Elevation of Privilege. For each threat: implemented mitigation.

### `decisions/` ⭐⭐ — Architecture Decision Records (ADRs)

#### What is an ADR?
A record of ONE important architectural decision: what was decided, why, what alternatives
were evaluated, and what the consequences are. They are **short documents** (1-2 pages).

**When to create an ADR:**
- When choosing a message broker (RabbitMQ vs Kafka vs Redis Streams)
- When deciding the database strategy (one per service vs shared)
- When choosing a communication pattern (REST vs gRPC vs events)
- When choosing an authentication library
- Any decision that, if changed, requires significant refactoring

**When NOT to create an ADR:**
- Day-to-day operational decisions
- Things that can be changed easily without systemic impact

**Use `decisions/_template-adr.md`**

**Typical ADR examples:**
```
ADR-001-documentation-language.md  → Why English for all documentation
ADR-002-auth-strategy.md           → Why JWT and not sessions
ADR-003-database-per-service.md    → Why separate DB per service
ADR-004-api-gateway.md             → Why Kong and not custom NGINX
```

---

## Correlations with other sections

| This section is fed by... | And feeds... |
|--------------------------|-------------|
| `02-domain/domain-map.md` → bounded contexts | `09-microservices/` → one service per context |
| `04-requirements/non-functional.md` → NFRs | Decisions about technology and scale |
| ADRs chosen here | `09-microservices/` implements the decided patterns |
| `deployment.md` | `10-devops/environments.md` |

---

## The 5 most common architecture mistakes

1. **Microservices too small** — If a "service" cannot exist independently, it is not a microservice.
2. **Shared database** — Destroys service independence. Each service, its own DB.
3. **Only synchronous communication** — For non-urgent operations, async events scale better.
4. **No API Gateway** — Exposing microservices directly to the frontend creates coupling.
5. **No documented decisions** — In 6 months nobody remembers why X was chosen.

---

## Questions this section must answer

- How is the system organized into large blocks?
- Why was each key technology chosen?
- What alternatives were evaluated and why were they discarded?
- How is the system deployed?
- What patterns does the team apply and how?
