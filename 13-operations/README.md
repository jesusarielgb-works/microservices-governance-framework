# 13 — Operations

> **What is this?** How the system is operated in production: how problems are detected,
> how incidents are responded to, and how availability commitments are guaranteed.

## Why this section exists

A system that "works" in staging but is impossible to operate in production is useless.
The key operations questions are:
- Do we know when something fails BEFORE users call?
- Do we know exactly what to do when something fails at 3am?
- Do we have formal availability commitments?

---

## Key concepts: SLA, SLO, SLI

**SLI (Service Level Indicator):** the metric you measure — e.g.: percentage of successful requests.

**SLO (Service Level Objective):** the target for that SLI — e.g.: "99.9% of successful requests in 30 days".

**SLA (Service Level Agreement):** the contract with the customer about those targets, with consequences.

**Error Budget:** how much you can "fail" within the SLO. If the SLO is 99.9%, you have 43.8 minutes/month of available downtime.

---

## What is here and how to fill it in

### `observability.md` ⭐
How the system makes its internal state visible.
**The 3 pillars of observability:**
- **Logs:** what each service records, in what format (structured JSON recommended), where they go
- **Metrics:** what each service measures (latency, error rate, throughput), with what tool (Prometheus)
- **Traces:** how a request that passes through multiple services is tracked (OpenTelemetry, Jaeger)

**Fill in:**
```markdown
## Logs
- Format: structured JSON with fields: timestamp, service, level, traceId, message, [contextual data]
- Tool: [ELK Stack / Loki + Grafana / CloudWatch]
- Retention: [30 days hot, 1 year cold]

## Metrics
- Tool: Prometheus + Grafana
- Main dashboard: [URL]
- Required metrics per service:
  - http_request_duration_seconds (histogram)
  - http_requests_total (counter by status code)
  - [specific business metrics]

## Distributed traces
- Tool: OpenTelemetry + Jaeger
- How to propagate traceId between services: X-Trace-Id header
```

### `incident-management.md` ⭐
Incident response process.
**Fill in:** how severity is classified (P0/P1/P2/P3), who responds, communication channel,
when to escalate, post-mortem process.

**Severities:**
```markdown
| Level | Description | Response SLA | Resolution SLA |
|-------|-------------|-------------|----------------|
| P0 | System completely down | 5 min | 1 hour |
| P1 | Critical functionality degraded | 15 min | 4 hours |
| P2 | Non-critical functionality affected | 1 hour | 24 hours |
| P3 | Minor issue, workaround available | 4 hours | 1 week |
```

### `backup-and-recovery.md`
Data backup and recovery strategy.
**Fill in:** backup frequency, where they are stored, how they are tested, target recovery time (RTO/RPO).

---

## Correlations with other sections

| This section is fed by... | And feeds into... |
|---------------------------|-------------------|
| `09-microservices/[service]/runbook.md` | Specific runbooks consolidated here |
| `05-architecture/overview.md` | What to monitor in each service |
| `10-devops/environments.md` | Alert configuration per environment |
| Incidents documented here | `15-project-control/technical-backlog.md` → post-incident improvements |

---

## Questions this section must answer

- How do we know the system is failing before users report it?
- What exactly do we do when service X fails at 3am?
- How much downtime can we have per month?
- How do we recover data if the DB fails?
