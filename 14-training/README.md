# 14 — Training

> **What is this?** Manuals for the different profiles that interact with the system:
> end users, administrators, and new team developers.

---

## What is here and how to fill it in

### `user-manual.md`
Manual for the system's end user.
**Fill in:** at the end of the project, when the screens are stable.
**Audience:** non-technical person who uses the system in their daily work.
**Include:** how to perform the main tasks, screenshots, what to do if something fails.

### `admin-manual.md`
Manual for the system administrator.
**Fill in:** system configurations, user and permission management, periodic tasks.
**Audience:** technical person who configures and maintains the system (not necessarily a developer).

### `technical-onboarding.md` ⭐
Guide for a new developer to contribute to the project.
**Fill in:** from the first day of the project.
**Audience:** new developer with general experience but no knowledge of this project.

**Minimum content:**
```markdown
## Day 1 — Setup
1. Read [00-governance/git-conventions.md] — 30 min
2. Start the system locally ([10-devops/local-setup.md]) — 2 hours
3. Read the architectural overview ([05-architecture/overview.md]) — 1 hour
4. Run the tests and make sure everything passes — 30 min

## Week 1 — Understand the domain
- Read [01-context/overview.md] and [02-domain/domain-map.md]
- Review the service catalog [09-microservices/service-catalog.md]
- Read the ADR for the service you are going to work on

## First contribution
1. Find a task labeled "good first issue" in the backlog
2. Follow the Git flow defined in [00-governance/git-conventions.md]
3. Request a code review from [team member]

## Frequently asked questions
- **How do I add a new endpoint?** → See [07-api/guidelines.md], then the corresponding service in [09-microservices/services/]
- **How do I add a DB migration?** → See [06-data/migration-strategy.md]
- **How do I run just one service?** → See [10-devops/local-setup.md#individual-services]
```

---

## Correlations

- `10-devops/local-setup.md` → first step of technical onboarding
- `00-governance/` → rules the new team member must know
- `09-microservices/` → destination of most work
