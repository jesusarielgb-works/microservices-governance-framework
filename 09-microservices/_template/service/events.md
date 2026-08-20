# Events — [Service Name]

> Events are how microservices communicate asynchronously.
> Document here all events that this service **publishes** and **consumes**.

---

## Events this service PUBLISHES

> These are the events this service emits when something important occurs.
> Other services can subscribe to them.

| Event | Topic/Exchange | When it is emitted | Payload (key fields) |
|-------|---------------|--------------------|---------------------|
| [NameInPastTense] | [topic.name] | [condition that triggers it] | `{field1, field2}` |

### Event schema: [EventName]

```json
{
  "eventId": "uuid",
  "eventType": "[EventName]",
  "timestamp": "2024-01-01T12:00:00Z",
  "version": "1.0",
  "source": "[service-name]",
  "payload": {
    "[field1]": "[type and description]",
    "[field2]": "[type and description]"
  }
}
```

---

## Events this service CONSUMES

> These are events from other services that this service reacts to.

| Event | Published by | Topic/Exchange | What this service does when it receives it |
|-------|-------------|----------------|-------------------------------------------|
| [EventName] | [source service] | [topic] | [action taken] |

---

## Delivery guarantees

| Guarantee | Value | Implication |
|-----------|-------|-------------|
| At-least-once | [Yes/No] | Consumers must be idempotent |
| At-most-once | [Yes/No] | Some events may be lost |
| Exactly-once | [Yes/No] | More expensive, more reliable |

---

## Error handling

- **Dead Letter Queue:** [name of the DLQ where failed events go]
- **Retries:** [N retries with exponential backoff]
- **Alerts:** [when the team is alerted of an event in the DLQ]
