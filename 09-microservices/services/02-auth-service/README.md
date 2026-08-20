# Auth Service

> **Identity authority** of the system. Manages authentication, JWT issuance, and RBAC permissions.
> It is the authoritative owner of the `User` and `Role` entities.

---

## Location in the architecture

| Field | Value |
|-------|-------|
| Number in catalog | 02 |
| Local port | 8081 |
| Repository | [Service repo URL] |
| DB engine | PostgreSQL (users and roles) + Redis (sessions and token blacklist) |
| Communicates with | — (does not call other microservices) |
| Consumed by | api-gateway (token verification), all other services |

---

## Responsibilities (what this service DOES)

- Register new users and hash passwords with bcrypt (cost 12)
- Authenticate users and issue access tokens (JWT, RS256, 1h) and refresh tokens
- Verify tokens: validate signature, expiration, and that they are not on the blacklist
- Manage roles and permissions (RBAC): assign, revoke, query
- Invalidate tokens (logout): add to blacklist in Redis until expiration

## Out of scope (what it does NOT do)

- **Does not handle business profile information** (company name, app preferences) — that data belongs to the service that needs it
- **Does not send emails** — publishes the `user.registered` event and notification-service listens to it
- **Does not authorize access to specific resources** — only informs permissions; each service performs the final verification

---

## How to run it locally

```bash
# Start with its dependencies (PostgreSQL + Redis)
docker compose up -d auth-service

# Verify
curl http://localhost:8081/health

# Create a test user
curl -X POST http://localhost:8081/api/v1/auth/register \
  -H 'Content-Type: application/json' \
  -d '{"email": "dev@example.com", "password": "DevPass123!", "name": "Dev User"}'
```

---

## Related documents

- [data-model.md](./data-model.md) — Tables `users`, `roles`, `user_roles`
- [events.md](./events.md) — `user.registered`, `user.login_failed`, `user.password_changed`
- [decisions.md](./decisions.md) — Design decisions for the authentication service
- [runbook.md](./runbook.md) — Operation: key rotation, password reset, token blacklist
- [API Contract](../../../07-api/contracts/openapi/auth-service.yaml)

---

## Emitted JWT structure

```json
{
  "sub": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "roles": ["USER"],
  "permissions": ["appointment:read", "appointment:write"],
  "iat": 1705316400,
  "exp": 1705320000
}
```

**Algorithm:** RS256 (public key available at `GET /api/v1/auth/jwks`)

---

## Required environment variables

See `.env.example` at the root for the variable names.
Real values are in the vault of the corresponding environment.

| Variable | Purpose |
|----------|---------|
| `APP_AUTH_DATABASE_URL` | PostgreSQL connection |
| `APP_AUTH_REDIS_URL` | Redis connection (token blacklist) |
| `APP_AUTH_JWT_PRIVATE_KEY` | RS256 private key for signing tokens |
| `APP_AUTH_JWT_PUBLIC_KEY` | RS256 public key for verifying tokens |
| `APP_AUTH_BCRYPT_ROUNDS` | bcrypt cost factor (default: 12) |
