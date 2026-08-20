# Contributing Guide

> Welcome to the team. This guide explains how to make your first contribution
> and the rules that apply to all changes in this repository.

---

## Before you start

1. Read `00-governance/README.md` — team rules
2. Read `00-governance/git-conventions.md` — how to work with Git
3. Make sure your local environment is set up: `10-devops/local-setup.md`
4. Understand the system domain: `02-domain/domain-map.md`

---

## Workflow

```
1. Pick a User Story from the sprint (status: Ready)
2. Create your branch from develop
3. Implement using TDD (Red → Green → Refactor)
4. Update affected documentation
5. Open a Pull Request using the template
6. PR is reviewed and merged by the Tech Lead
```

### Branch naming

```
feat/HU-[service]-[number]-short-description
fix/HU-[service]-[number]-short-description
chore/short-description
docs/short-description
```

Examples:
```
feat/HU-AUTH-001-jwt-login
fix/HU-SCHEDULING-003-schedule-overlap
docs/adr-002-database-engine
```

---

## Pull Request process

1. **Open the PR** against `develop` (never directly against `main`)
2. **Fill in the PR template** completely (`/.github/pull_request_template.md`)
3. **Assign reviewers:** at least 1 (preferably the Tech Lead or service owner)
4. **CI must be green** — do not request review with a failing pipeline
5. **Do not force-push** to a branch whose PR already has comments — create new commits

### What blocks a merge

- Failing CI pipeline (lint, tests, build)
- Fewer than 1 approval
- Documentation not updated (API, data model, domain)
- Test coverage below the project minimum (see `11-quality/tdd-guide.md`)

---

## Code style

- Follow the linting and formatting conventions defined in your stack guide (`_stacks/[your-stack].md`). Do not modify linter configuration without an ADR.
- Conventional Commits are mandatory: `type(scope): description`
- One commit = one logical unit of change. Do not commit "wip" or "fixing stuff"

### Commit types

| Type | When to use |
|------|-------------|
| `feat` | New functionality |
| `fix` | Bug fix |
| `refactor` | Code improvement without behavior change |
| `test` | Add or improve tests |
| `docs` | Documentation only |
| `chore` | Build, CI, dependencies, config |
| `perf` | Performance improvement |

---

## TDD — Test-Driven Development

**Team rule:** User Stories are implemented using TDD. No exceptions.

```
1. Write the test that describes the behavior (RED — fails)
2. Write the minimum code to pass the test (GREEN — passes)
3. Refactor without breaking tests (REFACTOR)
```

See full guide: `11-quality/tdd-guide.md`

---

## Documentation

If your change:
- Adds/modifies an endpoint → update the OpenAPI contract
- Changes a data model → update the service's `data-model.md`
- Makes an architecture decision → write an ADR
- Adds a new microservice → update `service-catalog.md`

---

## Questions?

- Check the documentation in this repo — it's probably already answered there
- Ask in the daily stand-up or in the team channel
- If you think something is missing from the documentation, document it yourself (that is the spirit of this project)
