## What does this PR do?

> Briefly describe the change. One or two sentences a reviewer can read in 10 seconds.

---

## Type of change

- [ ] New feature (feat)
- [ ] Bug fix (fix)
- [ ] Refactor without behavior change (refactor)
- [ ] Documentation (docs)
- [ ] Infrastructure / CI / configuration (chore)
- [ ] Tests (test)

---

## Related HU(s)

Closes: #[issue or HU number]

---

## Changes included

> List the relevant files/modules that changed and why. Focus on the "why", not the "what" — the diff already shows the what.

- `[file or module]` — [reason for change]
- `[file or module]` — [reason for change]

---

## Tests performed

- [ ] Unit tests written / updated and passing (`npm test`)
- [ ] Integration tests passing (if applicable)
- [ ] Manually tested the main flow locally
- [ ] Tested the most likely error cases

### Test cases executed

| Scenario | Expected result | Passed? |
|----------|----------------|---------|
| [main flow] | [result] | ✅ / ❌ |
| [error case] | [result] | ✅ / ❌ |

---

## Documentation checklist

- [ ] If endpoint behavior changes → OpenAPI contract updated
- [ ] If data model changes → `06-data/models.md` or `services/NN/data-model.md` updated
- [ ] If domain changes (entities, rules, events) → `02-domain/` updated
- [ ] If an architectural decision was made → ADR written in `05-architecture/decisions/records/`
- [ ] If there is a new service → `09-microservices/service-catalog.md` updated

---

## Definition of Done

- [ ] Code reviewed by at least 1 person
- [ ] Unit tests written (coverage ≥ 80% on modified files)
- [ ] No linter warnings in modified files
- [ ] Documentation updated (see checklist above)
- [ ] PR deploys successfully to the staging environment (pipeline green)

See full DoD: [`00-governance/definition-of-done.md`](../00-governance/definition-of-done.md)

---

## Notes for the reviewer

> Is there anything you'd like the reviewer to pay special attention to? Any debatable technical decision?
> Any context the diff doesn't show?

[Write here or remove this section if not applicable]
