# Risk Register

> Unmitigated risks become problems. This register allows the team to
> anticipate, not just react.
> Review and update at each retrospective or when there is a significant project change.

---

## Probability × impact matrix

```
IMPACT
  │
  │ HIGH  │  [Mitigate]    │  [Avoid]       │
  │       │  Low           │  High          │
  │       │  Probability   │  Probability   │
  │───────────────────────────────────────
  │ MEDIUM│  [Accept]      │  [Mitigate]    │
  │       │  with monitoring│              │
  │───────────────────────────────────────
  │ LOW   │  [Accept]      │  [Accept]      │
  │       │                │               │
  └───────────────────────────────────────
              LOW              HIGH
                      PROBABILITY
```

**Response strategies:**
- **Avoid:** Change the plan so the risk cannot materialize
- **Mitigate:** Reduce the probability or impact
- **Transfer:** Pass the risk to another party (insurance, provider, contract)
- **Accept:** Acknowledge the risk and have a contingency plan

---

## Active risk register

### R-001 — [Risk name]

| Field | Value |
|-------|-------|
| **ID** | R-001 |
| **Category** | [Technical / Business / Team / External] |
| **Description** | [What could go wrong] |
| **Probability** | [High / Medium / Low] |
| **Impact** | [High / Medium / Low] |
| **Risk level** | [Critical / High / Medium / Low] |
| **Strategy** | [Mitigate / Avoid / Transfer / Accept] |
| **Mitigation plan** | [What we do to reduce probability or impact] |
| **Contingency plan** | [What we do if the risk materializes] |
| **Trigger** | [Warning signal that the risk is about to occur] |
| **Owner** | [Name of the person responsible for monitoring this risk] |
| **Review date** | [date] |
| **Status** | [Active / Mitigated / Occurred / Closed] |

---

### R-002 — [Risk name]

| Field | Value |
|-------|-------|
| **ID** | R-002 |
| **Category** | [Technical] |
| **Description** | [E.g.: Dependency X may not be available by the integration date] |
| **Probability** | [Medium] |
| **Impact** | [High] |
| **Risk level** | [High] |
| **Strategy** | [Mitigate] |
| **Mitigation plan** | [Start conversation with provider 6 weeks ahead; design mock/stub just in case] |
| **Contingency plan** | [Develop with mock, launch without the integration, add it in the next iteration] |
| **Trigger** | [No response from provider in 2 weeks] |
| **Owner** | [Name] |
| **Review date** | [date] |
| **Status** | [Active] |

---

## Common technical risks in microservices

These risks apply to almost all microservices projects. Evaluate which ones apply:

| Risk | Typical probability | Standard mitigation |
|------|--------------------|--------------------|
| Failure cascade (one service down brings everything down) | Medium | Circuit Breaker, timeout, fallback |
| Data inconsistency between services | High | Saga, Outbox, idempotent events |
| Network latency in synchronous calls | High | gRPC, cache, async where possible |
| Lost messages in the broker | Medium | At-least-once + idempotent consumers |
| Schema evolution breaks consumers | High | Version events, compatible changes first |
| Technical debt accumulation | High | DoD with minimum coverage, regular reviews |
| Premature over-engineering | Medium | Start simple, YAGNI, measure before optimizing |
| Sensitive data exposure in logs | Medium | Logging policy, PII masking, SAST |
| Configuration drift between environments | High | Infrastructure as Code, environment variables |

---

## Closed risks / Lessons learned

| ID | Risk | Outcome | Lesson |
|----|------|---------|--------|
| R-00X | [name] | [Occurred / Did not occur] | [What we learned] |

---

## Template for adding a new risk

```markdown
### R-00X — [Name]

| Field | Value |
|-------|-------|
| **ID** | R-00X |
| **Category** | |
| **Description** | |
| **Probability** | |
| **Impact** | |
| **Risk level** | |
| **Strategy** | |
| **Mitigation plan** | |
| **Contingency plan** | |
| **Trigger** | |
| **Owner** | |
| **Review date** | |
| **Status** | Active |
```

---

## Correlations

- External dependencies → `15-project-control/dependencies.md`
- Related open questions → `15-project-control/open-questions.md`
- Architectural decisions that mitigated risks → `05-architecture/decisions/`
