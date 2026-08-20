# Domain Events

> **What to fill in here:** A domain event is a fact that occurred in the business.
> They are the backbone of asynchronous communication between bounded contexts.
> The name is ALWAYS in past tense and in the ubiquitous language of the domain.

---

## What is a domain event?

A **Domain Event** communicates that something important occurred in the business.
It is an immutable message that describes the fact in past tense.

```
✓ OrderCreated
✓ PaymentRejected
✓ UserRegistered
✓ StockDepleted

✗ CreateOrder (this is a command, not an event)
✗ OrderUpdated (too generic — what changed?)
✗ OrderEvent (does not indicate what occurred)
```

### Difference between Command and Event

| Concept | Intent | Tense | Can fail? |
|---------|--------|-------|-----------|
| **Command** | Instruction to do something | Present | Yes |
| **Event** | Notification of something that occurred | Past | No (it already happened) |

```
User → [CreateOrder] → System → [OrderCreated] → Other contexts
          (Command)                  (Event)
```

---

## Event catalog

### Event: [EventName]

| Field | Value |
|-------|-------|
| **Name** | `[EventName]` |
| **Bounded Context** | [Origin context] |
| **Aggregate** | [Aggregate that generates it] |
| **Trigger** | [Which business action generates this event] |
| **Consumers** | [Which services/contexts listen to this event] |
| **Channel (topic)** | `[topic.name]` |
| **Schema version** | `v1` |
| **Delivery guarantee** | At-least-once / At-most-once / Exactly-once |

**Payload (JSON schema):**

```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440000",
  "eventType": "[EventName]",
  "aggregateId": "550e8400-e29b-41d4-a716-446655440001",
  "aggregateType": "[AggregateName]",
  "occurredAt": "2024-01-15T10:30:00Z",
  "version": 1,
  "payload": {
    "[field1]": "[type and description]",
    "[field2]": "[type and description]"
  },
  "metadata": {
    "correlationId": "550e8400-e29b-41d4-a716-446655440002",
    "causationId": "550e8400-e29b-41d4-a716-446655440003",
    "userId": "550e8400-e29b-41d4-a716-446655440004"
  }
}
```

**Real payload example:**

```json
{
  "eventId": "generated-uuid",
  "eventType": "[EventName]",
  "aggregateId": "aggregate-uuid",
  "aggregateType": "[AggregateName]",
  "occurredAt": "2024-01-15T10:30:00Z",
  "version": 1,
  "payload": {
    "[field1]": "example value",
    "[field2]": 150.00
  }
}
```

**What do consumers do with this event?**

| Consuming service | Action | Idempotent? |
|------------------|--------|-------------|
| [Service A] | [Updates its data model] | Yes — uses eventId as idempotency key |
| [Service B] | [Sends notification] | Yes — checks if notification was already sent |

---

## Standard fields for all events

All events must include these fields in the envelope:

| Field | Type | Description |
|-------|------|-------------|
| `eventId` | UUID | Unique event ID (for idempotency) |
| `eventType` | string | Event name in PascalCase |
| `aggregateId` | UUID | ID of the aggregate that generated the event |
| `aggregateType` | string | Aggregate type |
| `occurredAt` | ISO 8601 | When the business fact occurred |
| `version` | integer | Schema version (for evolution) |
| `payload` | object | Event data (specific per type) |
| `metadata.correlationId` | UUID | For tracing a transaction across services |
| `metadata.causationId` | UUID | ID of the event or command that caused this event |
| `metadata.userId` | UUID | User who initiated the chain (if applicable) |

---

## Event flow: [Flow name]

> Document here the event flows for the main business processes.
> Use the Event Storming format: orange=event, blue=command, green=view/policy, yellow=aggregate.

```
[Actor]
  │
  │  [CommandA]           [CommandB]           [CommandC]
  ▼      │                    │                    │
[AggregateA]          [AggregateB]          [AggregateC]
  │                        ▲                    ▲
  │   [EventA]             │   [EventB]         │
  └──────────────────────▶│──────────────────▶│
```

### Example: Order creation flow

```
Customer
  │
  │  CreateOrder (command)
  ▼
[Aggregate: Order]
  │
  │  OrderCreated (event)
  ├──────────────────────────────────┐
  │                                   ▼
  │                          [Service: Inventory]
  │                          Decrements stock
  │                          StockReserved (event)
  │
  │  OrderCreated (event)
  └──────────────────────────────────┐
                                      ▼
                            [Service: Notifications]
                            Sends email to customer
```

---

## Schema evolution strategy

Events are contracts. Changing them in an incompatible way breaks consumers.

### What is a compatible change (does not break)?

```
✓ Add a new optional field to the payload
✓ Add a new event type
✓ Change a required field → optional
```

### What is an incompatible change (breaks)?

```
✗ Remove a field from the payload
✗ Change the type of a field (string → number)
✗ Change an optional field → required
✗ Change the event name
```

### How to evolve a schema without breaking consumers

**Strategy: Version the event**

```
Step 1: Publish EventV2 (new type with incompatible changes)
Step 2: Publish both EventV1 and EventV2 during the migration period
Step 3: Migrate consumers to V2 one by one
Step 4: Deprecate EventV1 (announce 1 sprint in advance)
Step 5: Stop publishing EventV1
```

---

## Event summary table

| Event | Origin context | Topic | Consumers | Version |
|-------|---------------|-------|-----------|---------|
| [EventA] | [ContextA] | `[topic.a]` | [SvcB, SvcC] | v1 |
| [EventB] | [ContextB] | `[topic.b]` | [SvcA] | v1 |

---

## Policies — Reactions to events

A **Policy** (or Saga step) describes what happens automatically when an event arrives.
It is the logic of "whenever X occurs, do Y".

```
Event:  OrderCreated
Policy: Whenever an OrderCreated arrives with type=URGENT,
        emit the command NotifyOperationsTeam
```

| Trigger event | Policy | Emitted command | Service |
|--------------|--------|----------------|---------|
| [EventA] | Whenever [condition], then... | [CommandB] | [ServiceX] |

---

## Resilience patterns for events

### At-least-once delivery + Idempotency

The message broker guarantees the event is delivered **at least once** but it may be
delivered more than once (in case of retries). Consumers must be **idempotent**.

```typescript
// Idempotent consumer — stores the processed eventId
async function processOrderCreatedEvent(event: OrderCreated): Promise<void> {
  // 1. Check if already processed
  if (await isEventAlreadyProcessed(event.eventId)) {
    logger.info(`Event ${event.eventId} already processed, ignoring`);
    return;
  }

  // 2. Process the event
  await updateModel(event.payload);

  // 3. Mark as processed (in the same transaction)
  await markEventProcessed(event.eventId);
}
```

### Dead Letter Queue (DLQ)

When an event fails after N retries, it goes to the DLQ.

| Configuration | Recommended value |
|--------------|------------------|
| Retries before DLQ | 3-5 |
| Backoff | Exponential (1s → 2s → 4s → 8s) |
| DLQ retention | 7 days |
| Alert | When DLQ has > 0 messages |

> See DLQ runbook in `09-microservices/services/XX-service/runbook.md`
