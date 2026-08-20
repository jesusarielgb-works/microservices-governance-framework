# 02 — Problem Domain

> **What is this?** The mental model of the business. It is not technology — it is understanding
> the problem the system solves before writing code. This section comes from Domain-Driven Design (DDD).

## Why this section exists

The most costly mistakes in software are not bugs — they are domain misunderstandings.
When developers do not deeply understand the business:
- They create incorrect abstractions that have to be rewritten
- Names in the code do not match the business's names → permanent confusion
- Microservice boundaries are drawn incorrectly

This section captures domain knowledge **before** designing the architecture.

---

## Key concepts you must know

**Entity:** Domain object with a unique identity (e.g.: a `Student` identified by their code).

**Value Object:** Object with no identity of its own, defined by its attributes (e.g.: `Address`, `Price`).

**Aggregate:** Group of entities treated as a unit. Only the aggregate root
can be referenced from outside.

**Domain Event:** Something that occurred in the business that other parts of the system must know
(e.g.: `StudentEnrolled`, `PaymentApproved`). They are facts, stated in past tense.

**Bounded Context:** Area of the system where a particular model applies.
Each microservice generally corresponds to a bounded context.

---

## What is here and how to fill it in

### `domain-map.md` ⭐
Map of all bounded contexts and how they relate.
**Fill in:** draw the contexts as rectangles and the relationships between them
(upstream/downstream, shared kernel, anti-corruption layer).

**Format:**
```markdown
## Bounded Contexts

### [Context Name]
**Responsibility:** [what this context manages]
**Main entities:** [list]
**Owning team:** [team]

## Relationship map
[ASCII diagram or description of how the contexts relate]

| Context A | Relationship | Context B | Description |
|-----------|-------------|-----------|-------------|
| [A] | downstream-of | [B] | [A] consumes events from [B] |
```

### `entities-and-rules.md` ⭐
Catalog of entities, value objects, and business rules.
**Fill in:** for each entity: name, attributes, invariants (rules that MUST ALWAYS hold),
behaviors.

**Format:**
```markdown
## Entity: [EntityName]
**Belongs to:** [Bounded Context]
**Identifier:** [field that makes it unique]

### Attributes
| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|

### Business rules (invariants)
- [ ] [A rule that must always hold]

### Behaviors (domain methods)
- `[name()]`: [what it does]
```

### `domain-events.md` ⭐
List of all events that occur in the domain.
**Fill in:** event name (past tense), what triggers it, what data it carries, who consumes it.

**Format:**
```markdown
| Event | Triggered by | Data | Consumers | Bounded Context |
|-------|-------------|------|-----------|----------------|
| [NameInPastTense] | [action that causes it] | [event fields] | [what listens to this] | [context] |
```

---

## Correlations with other sections

| This section feeds... | Why |
|-----------------------|-----|
| `05-architecture/overview.md` | Bounded contexts → microservices |
| `06-data/models.md` | Entities → data tables/collections |
| `07-api/contracts/` | Domain events → events in async APIs |
| `09-microservices/event-catalog.md` | All domain events are registered there |
| `04-requirements/user-stories.md` | Business rules → acceptance criteria |

---

## Recommended tool: Event Storming

**Event Storming** is a domain discovery workshop with sticky notes:
1. 🟠 Orange: Domain events (past tense)
2. 🔵 Blue: Commands (what triggers the event)
3. 🟡 Yellow: Actors (who executes the command)
4. 🟣 Purple: Policies (automatic reactions)
5. 🟦 Light blue: External systems

Running an Event Storming session with the team before filling in this section saves weeks of redesign.

---

## Questions this section must answer

- What are the main business entities?
- What rules can NEVER be violated in the system?
- What important events occur in the domain?
- Where are the natural boundaries of the system (for defining microservices)?
