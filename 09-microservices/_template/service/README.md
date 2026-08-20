# [Service Name]

> Replace this README with the specific service's documentation.
> Delete the instructions in `[brackets]` when they are complete.

---

## Responsibility

> [One sentence: what this service does. What data is its authoritative owner.]

---

## Location in the architecture

| Field | Value |
|-------|-------|
| Number in catalog | [NN] |
| Local port | [80XX] |
| Repository | [URL] |
| DB engine | [PostgreSQL / MongoDB / Redis / —] |
| Communicates with | [list of other services it consumes] |
| Consumed by | [list of services/frontends that call it] |

---

## Responsibilities (what this service DOES)

- [Responsibility 1]
- [Responsibility 2]

## Out of scope (what it does NOT do)

- [What it delegated to another service and which one]

---

## How to run it locally

```bash
# From the project root
docker compose up -d [service-name]

# Verify
curl http://localhost:[port]/health
```

---

## Related documents

- [data-model.md](./data-model.md) — DB schema
- [events.md](./events.md) — Events it publishes and consumes
- [decisions.md](./decisions.md) — Internal design decisions
- [runbook.md](./runbook.md) — Operation in production
- [API Contract](../../../07-api/contracts/openapi/[service-name].yaml)
