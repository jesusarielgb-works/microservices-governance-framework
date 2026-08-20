# 07 — API Contracts

> **What is this?** The formal specification of how microservices communicate with each other
> and with the outside world. The contract is the promise a service makes to its consumers.

## Why contracts are critical in microservices

In a monolith, changing a function is easy: the compiler tells you what you break.
In microservices, a change to an API can silently break other services in production.

**Contract-first development:** design the API before implementing it. This way the team
can work in parallel (frontend, backend, other services) with an agreed contract.

---

## What is here and how to fill it in

### `guidelines.md` ⭐
The project's REST standards. **The entire team must follow this before designing an endpoint.**

**Key topics to define:**
```markdown
## Versioning
- URL: /api/v1/resources (version in the URL)
- Header: Accept: application/vnd.api+json;version=1

## Endpoint naming
- Plural for collections: GET /users
- Nouns, not verbs: GET /users/{id}/orders (not: GET /getUserOrders)
- snake_case or kebab-case for URLs

## Pagination
- ?page=1&limit=20 (offset-based)
- ?cursor=xyz (cursor-based for large volumes)

## Standard responses
| Code | When to use it |
|------|----------------|
| 200 | Success with body |
| 201 | Successful creation |
| 204 | Success without body |
| 400 | Client error (validation) |
| 401 | Not authenticated |
| 403 | Not authorized |
| 404 | Resource not found |
| 422 | Unprocessable entity |
| 500 | Server error |

## Error format
{
  "error": "VALIDATION_ERROR",
  "message": "The email field is required",
  "details": [{"field": "email", "message": "required"}]
}
```

### `authentication.md` ⭐
The system's authentication and authorization strategy.
**Fill in:** what mechanism (JWT, OAuth2, API Key), authentication flow, expiration policies,
refresh token handling, RBAC (roles and permissions).

### `contracts/openapi/` ⭐⭐
One `.yaml` file per microservice with the OpenAPI 3.0 contract.

**File name:** `service-name.yaml`

**Minimum structure for each contract:**
```yaml
openapi: 3.0.3
info:
  title: [Service Name] API
  version: 1.0.0
  description: [What this service does]

servers:
  - url: http://localhost:8080/api/v1
    description: Local

paths:
  /resources:
    get:
      summary: List resources
      tags: [Resources]
      parameters:
        - name: page
          in: query
          schema:
            type: integer
      responses:
        '200':
          description: List of resources
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ResourceList'

components:
  schemas:
    Resource:
      type: object
      required: [id, name]
      properties:
        id:
          type: string
          format: uuid
        name:
          type: string
  
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - bearerAuth: []
```

### `_shared.yaml`
Reusable schemas and components across all contracts (pagination, errors, etc.)

---

## Recommended tools

- **Swagger Editor** — Online editor for OpenAPI
- **Redocly** — Generates HTML documentation from the YAML (already configured in `redocly.yaml`)
- **Postman / Insomnia** — For testing the contracts
- **OpenAPI Generator** — Generates client/server code from the YAML

---

## Correlations with other sections

| This section is fed by... | And feeds into... |
|---------------------------|-------------------|
| `04-requirements/functional.md` → what it must do | Endpoints implementing each FR |
| `06-data/models.md` → what data exists | Schemas in the contracts |
| `02-domain/domain-events.md` | Event publishing endpoints |
| Contracts here | `09-microservices/[service]/` that implements them |
| Contracts here | `11-quality/testing-strategy.md` → contract tests |

---

## Questions this section must answer

- What is each endpoint called and what does it do?
- What data does it receive and what does it return?
- How does the client authenticate?
- What errors can the service return?
- How do I version the API without breaking consumers?
