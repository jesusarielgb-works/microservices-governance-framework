# Definition of Ready (DoR)

> A User Story is **Ready** when the entire team can start it in the next sprint
> without needing to resolve fundamental questions mid-sprint.
> If a story doesn't meet this DoR, it goes back to refinement.

---

## DoR checklist

Before moving a User Story to "Ready for Sprint", verify:

### Clarity

- [ ] The story is written in the format: **As [role], I want [action], so that [benefit]**
- [ ] The role is specific (not "as a user" — "as an authenticated buyer")
- [ ] The expected benefit is clear and verifiable

### Acceptance Criteria

- [ ] There are at least 2 acceptance criteria written in **Given / When / Then** format
- [ ] The criteria cover the happy path AND the main error cases
- [ ] The criteria are testable (it is possible to write an automated test for each one)
- [ ] There are no ambiguous criteria ("the response should be fast" is not valid)

### Dependencies

- [ ] All external dependencies (other services, APIs, data) are identified
- [ ] Blocking dependencies are resolved OR a workaround is defined
- [ ] If it depends on another story, that story is already Done or In Progress

### Estimation

- [ ] The team has estimated the story (story points or t-shirt size)
- [ ] There is agreement that the story fits in one sprint
- [ ] If it's too large, it has been broken down into smaller stories

### Technical readiness

- [ ] The necessary accesses and environments are available
- [ ] The API contracts (OpenAPI) are defined if the story involves new endpoints
- [ ] There is a definition of the data model if there are DB changes
- [ ] The impact on other services is identified

### Non-functional requirements

- [ ] Performance requirements are specified (if applicable)
- [ ] Security requirements are considered (authentication, authorization, validations)
- [ ] Observability requirements are included (logs, metrics, traces)

---

## Common reasons a story is NOT ready

| Problem | What to do |
|---------|-----------|
| Unclear requirements | Schedule a 30-min refinement session with the PO |
| Missing acceptance criteria | PO adds criteria before the next sprint |
| Unknown dependencies | Tech Lead reviews and documents dependencies |
| Too large (> 8 SP) | Break it down into smaller stories |
| No access to test environment | DevOps generates credentials before sprint |
| Unclear API contract | Agree on contract (OpenAPI) before starting |

---

## DoR vs DoD

| | Definition of Ready (DoR) | Definition of Done (DoD) |
|-|--------------------------|--------------------------|
| **When** | Before starting the story | After finishing the story |
| **Who verifies** | Team in planning/refinement | Team in review |
| **Purpose** | Ensure the team can start without blockers | Ensure the increment is shippable |

---

## Correlations

- Full DoD → `00-governance/definition-of-done.md`
- User Story template → `04-requirements/_template-hu.md`
- User Stories backlog → `04-requirements/user-stories.md`
