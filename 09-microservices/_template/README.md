# [Microservice Name]

> **Copy this folder** to document a new microservice.
> Rename the folder to `NN-service-name` (e.g.: `03-academic-service`).
> Fill in each file — delete these instructions when the document is complete.

---

## Responsibility

> [A single sentence: what this service does and what data it owns]

**Example:** "Manages authentication, sessions, and permissions for all system users."

---

## Location in the architecture

| Field | Value |
|-------|-------|
| Number in catalog | [01, 02, 03...] |
| Local port | [8001, 8002...] |
| Repository | [Repo URL] |
| Owner team | [team name] |
| DB engine | [PostgreSQL / MongoDB / Redis / none] |
| Status | [In development / In production / Deprecated] |

---

## How to run it locally

```bash
# Prerequisites
# - Docker Desktop installed
# - [other dependencies]

# Clone the repo
git clone [url]

# Environment variables
cp .env.example .env
# Edit .env with local values

# Start with Docker
docker-compose up -d

# Verify it is running
curl http://localhost:[port]/health
```

---

## Main endpoints

> [Quick list of the most important endpoints. The full contract is in `07-api/contracts/openapi/`]

| Method | Path | Description |
|--------|------|-------------|
| POST | /auth/login | Authenticate user |
| POST | /auth/refresh | Renew token |
| GET | /users/{id} | Get user |

---

## Documents for this service

- [data-model.md](./data-model.md) — Tables and DB schema
- [events.md](./events.md) — Events it publishes and consumes
- [decisions.md](./decisions.md) — Specific design decisions
- [runbook.md](./runbook.md) — How to operate in production
