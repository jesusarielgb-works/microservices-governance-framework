# API Gateway

> **Single entry point** to the system. Receives all frontend requests and routes them
> to the corresponding microservice. It is the authoritative owner of the routing rules.

---

## Location in the architecture

| Field | Value |
|-------|-------|
| Number in catalog | 01 |
| Local port | 8080 |
| Repository | [Service repo URL] |
| DB engine | — (stateless, no own DB) |
| Communicates with | auth-service, [other services] |
| Consumed by | Web frontend, mobile app, third-party tools |

---

## Responsibilities (what this service DOES)

- Route HTTP requests to the correct microservice based on the path (`/api/v1/auth/*`, `/api/v1/[resource]/*`)
- Verify that the JWT token is valid before forwarding the request (delegating to auth-service)
- Attach user context (`X-User-Id`, `X-User-Role`) as internal headers
- Global rate limiting: maximum [100] requests per IP per minute
- Unified access logs with correlationId

## Out of scope (what it does NOT do)

- **Does not handle business logic** — only routes
- **Does not verify resource permissions** — only verifies the JWT is valid (authorization is done by each service)
- **Does not store data** — stateless

---

## How to run it locally

```bash
# From the project root (starts all services)
docker compose up -d api-gateway

# Verify it is working
curl http://localhost:8080/health
```

**Expected response:**
```json
{ "status": "ok", "timestamp": "2024-01-15T10:30:00Z" }
```

---

## Related documents

- [data-model.md](./data-model.md) — Not applicable (no own DB)
- [events.md](./events.md) — Not applicable (does not emit domain events)
- [decisions.md](./decisions.md) — Gateway design decisions
- [runbook.md](./runbook.md) — Operation in production
- [API Contract](../../../07-api/contracts/openapi/api-gateway.yaml)

---

## Design decisions

See: `09-microservices/services/01-api-gateway/decisions.md`

Relevant decisions include:
- **ADR-002** — Why [Kong / Nginx / custom Express] was chosen as the gateway base
- **ADR-003** — Authentication strategy at the gateway vs. in each service

---

## Routing pattern

```
Client → :8080/api/v1/auth/*        → auth-service :8081
Client → :8080/api/v1/[resource]/*  → [name]-service :808N
```

Routing configuration: `[path to gateway configuration file]`
