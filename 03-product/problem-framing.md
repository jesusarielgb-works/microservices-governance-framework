# Problem Framing — Problem Definition

> **Why this document exists:** Before designing solutions, the team must be
> aligned on the problem it solves. This document captures that alignment.
> A well-defined problem is already halfway to a solution.

---

## 1. The problem in one sentence

> Complete this template:

**[User segment]** who **[usage context]** struggle with **[pain/problem]**
because **[root cause]**, resulting in **[quantifiable impact]**.

**Example:**
> **Mid-sized company inventory operators** who **manage catalogs of more than 500 products**
> struggle with **stock control across multiple warehouses** because **current systems
> do not support real-time synchronization**, resulting in **15% of orders with stock errors
> and 3 hours of manual correction work per day**.

---

## 2. Affected users

| Segment | Description | Estimated size | Priority |
|---------|-------------|---------------|---------|
| [Segment A] | [Who they are, what they do] | [N users] | High |
| [Segment B] | [Who they are] | [N users] | Medium |

### Jobs-to-be-done (JTBD)

> What job is the user trying to do when they "hire" our product?

**When** [situation / context],
**I want** [motivation / what they are trying to achieve],
**so that** [expected outcome / benefit].

---

## 3. Evidence of the problem

> The problem must be real. Document the evidence you have.

| Evidence type | Source | Date | Key finding |
|--------------|--------|------|------------|
| User interviews | [N] interviews with [profile] | [date] | [what they said] |
| Support data | Support tickets | [period] | [% of tickets on this topic] |
| Benchmarking | [Competitors / market] | [date] | [how others solve it] |
| Direct observation | [Shadowing / field research] | [date] | [what was observed] |

---

## 4. Current user solution (and its problems)

> How does the user solve the problem today?

| Current solution | Limitations | Cost/Friction |
|-----------------|------------|--------------|
| [Excel / manual process] | [Does not scale, errors, slow] | [X hours/day] |
| [Legacy system] | [No API, no integration] | [Y errors/week] |

---

## 5. Solution hypothesis

> This is the first draft of the solution direction. It is not a commitment.

**We believe that** [describe the high-level solution]
**for** [the user segment],
**will achieve** [the expected benefit].
**We will know we succeeded when** [specific metric].

---

## 6. Success metrics (North Star)

| Metric | Current baseline | 6-month target | How to measure it |
|--------|----------------|---------------|-------------------|
| [Business metric 1] | [current value] | [target value] | [instrument] |
| [Adoption metric] | [current value] | [target value] | [instrument] |

**North Star Metric:** [The single metric that best captures the value delivered]

---

## 7. Hypothesis risks

| Risk | Probability | Impact | Experiment to validate |
|------|------------|--------|----------------------|
| [Users will not adopt the change] | High | High | [Pilot with N users] |
| [The problem is not as frequent as we think] | Medium | High | [Support log analysis] |

---

## 8. Out of scope (we do not solve)

> Explicitly define which related problems you are NOT solving in this version.
> This prevents scope creep.

- [Related problem that is out of scope: why]
- [Feature users ask for but we are not doing now: reason]

---

## Correlations

- Product vision → `03-product/vision.md`
- HUs that implement this solution → `04-requirements/user-stories.md`
- Detailed KPIs → `13-operations/README.md`
