# Domain Map — Bounded Contexts

> **What to fill in here:** The domain map is the central DDD (Domain-Driven Design) artifact.
> It defines the system's boundaries and how they relate to each other.
> Build it first with the team and domain experts in an Event Storming session.

## Before filling in this document: Event Storming

**Event Storming** is a collaborative workshop for modeling the domain before writing code.
It lasts 2–4 hours with the whole team (dev + PO + business expert).

**Materials:** Long wall, 4-color sticky notes, markers.

**Standard colors:**
| Color | Represents | Example |
|-------|-----------|---------|
| 🟠 Orange | **Domain events** (something that happened, past tense) | `AppointmentScheduled`, `PaymentReceived` |
| 🔵 Blue | **Commands** (action that triggers the event) | `ScheduleAppointment`, `ProcessPayment` |
| 🟡 Yellow | **Actors** (who executes the command) | `Patient`, `Doctor`, `Admin` |
| 🩷 Pink | **External systems** or integration points | `Payment Gateway`, `Email SMTP` |

**Session steps:**
1. (30 min) Post all events that occur in the business, in chronological order, on the wall
2. (30 min) Identify which command or actor triggers each event
3. (45 min) Group related events — each group is a candidate Bounded Context
4. (30 min) Draw relationships between Bounded Contexts (who depends on whom)
5. (30 min) Discuss the resulting map and agree on names

**Result:** The session output directly feeds the 3 documents in `02-domain/`:
- Identified events → `domain-events.md`
- Entities and their rules → `entities-and-rules.md`
- Bounded Contexts and their map → this document

---

---

## 1. Domain overview

> One paragraph of context about the business and what problem the system solves.
> Write it without technical terms — it must be readable by a business expert.

```
[Describe the business domain here. E.g.: "The system manages the complete cycle of
[X] reservations, from the customer's request through to confirmation and billing."]
```

---

## 2. Identified Bounded Contexts

A **Bounded Context** is the explicit boundary within which a particular domain model
has consistent meaning. Each bounded context has its own Ubiquitous Language.

> **Signs of a good bounded context:**
> - Has a clear responsible team
> - Has its own database
> - Can be deployed independently
> - The same term in two different contexts can mean different things

### Bounded Context: [Name — e.g.: User Management]

| Field | Value |
|-------|-------|
| **Name** | [ContextName] |
| **Responsibility** | [What this context captures in one sentence] |
| **Owning team** | [Responsible team or person] |
| **Microservice(s)** | [service-name, service2-name] |
| **Database** | [PostgreSQL / MongoDB / Redis / etc.] |
| **Ubiquitous Language** | [Key business terms in this context] |

**Context-specific terms (Ubiquitous Language):**

| Term | Meaning in THIS context | Different in another context? |
|------|------------------------|-------------------------------|
| [User] | [Person with an active account] | [Yes — in Billing it's "Client"] |
| [Account] | [Set of credentials] | [No] |

---

### Bounded Context: [Name — e.g.: Order Management]

| Field | Value |
|-------|-------|
| **Name** | [ContextName] |
| **Responsibility** | |
| **Owning team** | |
| **Microservice(s)** | |
| **Database** | |
| **Ubiquitous Language** | |

---

## 3. Context Map

The Context Map shows relationships between bounded contexts. Relationships define
how contexts communicate and who holds the "power" in the integration.

```
┌─────────────────────┐        ┌──────────────────────┐
│  [Context A]        │        │  [Context B]         │
│                     │──────▶│                      │
│  Domain:            │  D→C  │  Domain:             │
│  [responsibility]   │       │  [responsibility]    │
└─────────────────────┘       └──────────────────────┘
                                        │
                               D→C      │
                                        ▼
                              ┌──────────────────────┐
                              │  [Context C]         │
                              │                      │
                              │  [responsibility]    │
                              └──────────────────────┘
```

### Context relationship types

| Type | Symbol | Description | Example |
|------|--------|-------------|---------|
| **Upstream → Downstream** | `U → D` | U provides, D consumes. D depends on U. | Auth → Orders |
| **Shared Kernel** | `SK` | Two teams share part of the model | Shared User ID |
| **Customer/Supplier** | `C/S` | Supplier (U) negotiates with Customer (D) | Inventory → Sales |
| **Conformist** | `CONF` | D adopts U's model without negotiating | Legacy integration |
| **Anti-Corruption Layer** | `ACL` | D translates U's model to protect itself | Gateway → External API |
| **Open Host Service** | `OHS` | U publishes a published protocol | Event Bus, REST API |
| **Published Language** | `PL` | Explicit shared language | OpenAPI spec, events |

### Relationships table

| Context A | Relationship | Context B | Communication channel | Contract |
|-----------|-------------|-----------|----------------------|---------|
| [Context A] | U → D | [Context B] | REST / Event | OpenAPI / AsyncAPI |
| [Context B] | ACL | [Context C] | Adapter | Internal interface |

---

## 4. Core Domain, Supporting, Generic

DDD classifies subdomains by their strategic value:

| Type | Description | Investment | Example |
|------|-------------|-----------|---------|
| **Core Domain** | Where the business competitive advantage lies. What differentiates us. | MAXIMUM — build, don't buy | Matching algorithm |
| **Supporting Subdomain** | Necessary for the core but not differentiating. Can be outsourced. | MEDIUM | Order management |
| **Generic Subdomain** | Commodity. Off-the-shelf solution exists. | MINIMUM — buy/use OSS | Authentication, emails |

### Classification of this project's bounded contexts

| Bounded Context | Type | Justification |
|----------------|------|---------------|
| [Context A] | Core | [why it is the heart of the business] |
| [Context B] | Supporting | [why it supports without being differentiating] |
| [Context C] | Generic | [standard solution available] |

---

## 5. Modeling decisions

### How were these decisions made?

> Document the Event Storming session or the process you used to arrive at the map.
> If you changed the map, explain why.

- **Event Storming session:** [date], [participants]
- **Tool used:** [Miro / Lucidchart / physical whiteboard]
- **Map iterations:** v1 (date), v2 (date)

### Key decisions and discarded alternatives

| Decision | Discarded alternative | Reason |
|----------|----------------------|--------|
| [Separate Context A and B] | [Have them in one] | [Business logic is different and they evolve at different rates] |

---

## 6. How to update this map

1. Before adding a new microservice, verify whether it belongs to an existing bounded context.
2. If a context's ubiquitous language is changing, review whether the context should be split.
3. Run an Event Storming session every time the domain changes significantly.
4. The context map MUST be synchronized with the C4 system-level diagram (`05-architecture/overview.md`).

> **Important correlation:** The bounded contexts in this document →
> Microservices in `09-microservices/service-catalog.md` →
> C4 diagrams in `08-uml/` →
> Service separation ADRs in `05-architecture/decisions/`
