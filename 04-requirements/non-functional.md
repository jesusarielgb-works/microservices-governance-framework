# Non-Functional Requirements (NFR)

> NFRs define the **qualities of the system** — not what it does but how well it does it.
> The golden rule: every NFR must have a metric. "The system must be fast" is not an NFR.
> "The P99 latency of the /orders endpoint must be < 200ms under 500 RPS load" is.

---

## How to write a measurable NFR?

| Bad | Good |
|-----|------|
| "The system must be fast" | "P95 latency must be < 300ms under 1000 concurrent RPS" |
| "The system must be secure" | "All endpoints require a valid JWT; tokens expire in 1 hour" |
| "The system must scale" | "The system must support up to 5000 concurrent users without degradation" |
| "The system must be available" | "Availability SLO: 99.9% monthly (maximum 44 min downtime/month)" |

---

## NFR-001: Performance

| Attribute | Metric | Test condition |
|-----------|--------|---------------|
| P95 latency — critical endpoints | < 300ms | Under [N] RPS load |
| P99 latency — critical endpoints | < 500ms | Under [N] RPS load |
| P95 latency — non-critical endpoints | < 1000ms | Normal load |
| Minimum throughput | [N] RPS | Without degradation |
| Service startup time | < 30 seconds | Cold start |

**Defined critical endpoints:**
- `POST /[resource]` — [justification for why it is critical]
- `GET /[resource]/:id` — [justification]

**Load testing tools:**
- k6, Apache JMeter, Locust, Gatling

**Where is it validated?** CI/CD in the staging pipeline before production.

---

## NFR-002: Availability

| Environment | SLO | Maintenance window | Max downtime/month |
|------------|-----|-------------------|-------------------|
| Production | 99.9% | Sundays 2am-4am | 44 minutes |
| Staging | 95% | No restriction | 36 hours |

**Monthly error budget in production:** 44 minutes
**Error Budget policy:** If > 50% of the error budget is consumed in the first half of the month,
feature deploys are frozen until the next month and stability is prioritized.

**Health checks:**
- `GET /health` — Liveness: responds 200 if the process is alive
- `GET /health/ready` — Readiness: responds 200 only if it can process traffic (DB connected, dependencies OK)

---

## NFR-003: Scalability

| Scenario | Expected behavior |
|---------|------------------|
| Gradual load growth | Horizontal auto-scaling activated when CPU > 70% |
| Sudden spike (Black Friday, etc.) | System scales in < 2 minutes |
| Load reduction | Scale-down without interrupting active traffic |
| Horizontal scaling limit | Up to [N] instances per service |

**Strategy:** Stateless horizontal scaling — each instance does not store state in memory.
State goes in Redis (sessions, cache) or PostgreSQL (persistent data).

---

## NFR-004: Security

### Authentication and Authorization
- All private endpoints require a valid JWT in the `Authorization: Bearer <token>` header
- JWT tokens expire in **1 hour**
- Refresh tokens valid for **7 days**
- RBAC (Role-Based Access Control): roles defined in `00-governance/security-policy.md`

### Data transmission
- HTTPS mandatory in production (TLS 1.2+)
- HTTP only in local development

### Sensitive data
- Passwords: hashing with bcrypt (cost factor ≥ 12) or Argon2id
- PII (personal data): encrypted at rest
- Secrets/keys: only in environment variables or vault, **never in code**

### OWASP Top 10
Code must be reviewed against the OWASP Top 10 on each release.
Tools: SAST (SonarQube/Snyk), dependency scanning, DAST in staging.

### Regulatory compliance
- [GDPR / Habeas Data / PCI-DSS / etc.] — as applicable to the project

---

## NFR-005: Observability

| Pillar | Requirement | Tool |
|--------|------------|------|
| Logs | Structured JSON format + Correlation ID | Winston / Logback |
| Metrics | RED (Rate, Errors, Duration) per endpoint | Prometheus + Grafana |
| Traces | End-to-end distributed traces | OpenTelemetry + Jaeger |
| Alerts | Alert in < 5 min when SLI violates SLO | Alertmanager / PagerDuty |

**Correlation ID:** Each external request generates a UUID correlationId propagated in all logs and spans of that transaction.

---

## NFR-006: Maintainability

| Metric | Target |
|--------|--------|
| Test coverage | ≥ 80% of lines (≥ 90% in the domain) |
| Cyclomatic complexity | ≤ 10 per function |
| Technical debt | Resolution time < 1 sprint from registration |
| Onboarding time | A new dev can deploy locally in < 1 hour following `10-devops/local-setup.md` |
| Average build time | < 5 minutes in CI |

---

## NFR-007: Portability

- All services are deployed as Docker images
- Images work in any environment with Kubernetes 1.28+
- No service depends on the host operating system
- Environment variables are the only source of environment-specific configuration

---

## NFR-008: Disaster Recovery (DR / Recovery)

| Scenario | RTO (Recovery Time Objective) | RPO (Recovery Point Objective) |
|---------|------------------------------|-------------------------------|
| Single service failure | < 2 minutes (K8s restart) | 0 (stateless) |
| Primary database failure | < 5 minutes (failover to replica) | < 1 second (synchronous replication) |
| Availability zone loss | < 15 minutes | < 5 minutes |
| Full region disaster | < 4 hours (DR in secondary region) | < 1 hour |

---

## NFR priority matrix

| NFR | Priority (P1/P2/P3) | Validated in CI? | Owner |
|-----|---------------------|-----------------|-------|
| Performance | P1 | Yes (k6 in staging) | [Tech Lead] |
| Availability | P1 | Yes (health checks) | [DevOps] |
| Security | P1 | Yes (SAST + OWASP) | [Security] |
| Scalability | P2 | Manual (quarterly) | [DevOps] |
| Observability | P1 | Yes (smoke test in CI) | [Tech Lead] |
| Maintainability | P2 | Yes (coverage in CI) | [Team] |

---

## Correlations

- Detailed SLOs and SLAs → `13-operations/README.md`
- Pipeline that validates NFRs → `10-devops/README.md`
- Incidents related to NFR violations → `13-operations/incident-management.md`
- Security checklist → `00-governance/security-policy.md`
