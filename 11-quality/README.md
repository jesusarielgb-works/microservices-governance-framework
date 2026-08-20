# 11 — Quality

> **What is this?** How the team ensures the system works correctly
> and keeps working as it evolves. Tests, code review, and metrics.

## The testing pyramid

```
          /\
         /  \
        / E2E \      ← Few, slow, expensive (end-to-end tests)
       /--------\
      / Integra-  \   ← Some, moderate (test interaction between parts)
     /  tion       \
    /--------------\
   / Unit Tests     \  ← Many, fast, cheap (test a function/class)
  /------------------\
```

In microservices, add a layer: **Contract Tests**.
They verify that the consumer and provider of an API agree on the contract.

---

## What is here and how to fill it in

### `testing-strategy.md` ⭐
The complete testing strategy for the project.
**Fill in:** what types of tests are done, with what tool, minimum expected coverage,
which go in the CI/CD pipeline, which are manual.

**Format:**
```markdown
## Types of tests

### Unit
- **Tool:** [Jest / JUnit / pytest / etc.]
- **Minimum coverage:** [80%]
- **What they test:** functions, classes, business logic in isolation
- **What they do NOT test:** DB, network, external services

### Integration
- **Tool:** [Testcontainers + JUnit / pytest]
- **What they test:** service + real DB (in container), without external services
- **When they run:** on every PR

### Contract (Contract Testing)
- **Tool:** [Pact / Spring Cloud Contract]
- **What they test:** that the producer fulfills what the consumer expects
- **When they run:** on every PR of the producer and consumer

### E2E (End-to-End)
- **Tool:** [Playwright / Cypress / k6]
- **What they test:** complete user flows with the entire system running
- **When they run:** before each deploy to staging

## Minimum coverage per service
| Service | Unit | Integration | Contract |
|---------|------|-------------|---------|
| iam | 85% | Required | Required |
| [rest] | 80% | Required | Recommended |
```

### `code-review.md`
Team code review guide.
**Fill in:** what to verify in a PR (beyond "it works"), list of required checks,
how to give constructive feedback, maximum time to review a PR.

---

## Correlations with other sections

| This section is fed by... | And feeds into... |
|---------------------------|-------------------|
| `04-requirements/traceability-matrix.md` → what to test | Test cases covering each FR |
| `07-api/contracts/` → API contracts | Contract tests |
| `00-governance/definition-of-done.md` → what every PR must have | Code review checklist |
| `10-devops/ci-cd.md` | Which tests go in which pipeline step |

---

## Questions this section must answer

- What types of tests does the team write?
- How much code coverage is required?
- What does a reviewer check in a Pull Request?
- How do we know a change did not break anything?
