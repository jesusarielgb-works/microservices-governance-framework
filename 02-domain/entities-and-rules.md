# Entities, Value Objects, and Business Rules

> **What to fill in here:** The building blocks of the domain following the DDD tactical model.
> This document translates domain knowledge (obtained in Event Storming) into code models.

> **Stack note:** The concepts of Entity, Value Object, and Aggregate are language-independent.
> Code examples (classes, interfaces, decorators) are written in pseudo-TypeScript
> to illustrate the idea. To see the implementation in your technology:
> [`_stacks/node-typescript.md`](../_stacks/node-typescript.md) ·
> [`_stacks/java-spring.md`](../_stacks/java-spring.md) ·
> [`_stacks/python-fastapi.md`](../_stacks/python-fastapi.md) ·
> [`_stacks/go.md`](../_stacks/go.md)

---

## Tactical DDD concepts

### Entity
An **Entity** is an object defined by its identity, not its attributes.
Two entities are equal if they have the same ID, even if all their other attributes differ.

```
✓ Entity: User (two users with different emails are still distinct by their ID)
✓ Entity: Order (changes state but remains the same order)
✗ Not an entity: Money (10 USD == 10 USD regardless of which bill)
```

### Value Object (VO)
A **Value Object** is an object defined by its attributes; it has no identity of its own.
It is immutable — if an attribute changes, it is a new VO.

```
✓ Value Object: Address (5th Street #10-20, Neiva, Huila)
✓ Value Object: Money (USD 150.00)
✓ Value Object: Email (user@example.com)
✓ Value Object: DateRange (2024-01-01 → 2024-01-31)
```

### Aggregate
An **Aggregate** is a cluster of entities and VOs treated as a unit.
It has an **Aggregate Root** which is the entry point — internal objects can only be
accessed through the root.

```
Order (Aggregate Root)
  ├── OrderItems[] (Entities inside the aggregate)
  ├── DeliveryAddress (Value Object)
  └── OrderTotal (Calculated Value Object)
```

**Golden rule of the Aggregate:** Transactions do not cross aggregate boundaries.
If you need to modify two aggregates in one operation, use a Domain Event and a Saga.

### Business Rules
**Business Rules** (invariants) are the constraints the domain must always satisfy.
They live in the Aggregate Root and are validated on every operation.

---

## System entities

### Entity: [EntityName]

**Context:** [Bounded Context it belongs to]

**Description:** [What it represents in the business, in one sentence]

**Attributes:**

| Attribute | Type | Description | Required | Rules |
|-----------|------|-------------|---------|-------|
| id | UUID | Unique identifier | Yes | Auto-generated on creation |
| [attribute] | [type] | [description] | [Yes/No] | [validations] |
| createdAt | DateTime | Creation date | Yes | Immutable, set on creation |
| updatedAt | DateTime | Last modification | Yes | Updated automatically |

**Lifecycle / States:**

```
[State A] ──(action)──▶ [State B] ──(action)──▶ [State C]
                               │
                          (action)
                               ▼
                          [State D]
```

| State | Description | Allowed transitions |
|-------|-------------|---------------------|
| [DRAFT] | Just created, not published | → ACTIVE, → CANCELLED |
| [ACTIVE] | Available for use | → INACTIVE, → CANCELLED |
| [CANCELLED] | Finished without completing | Terminal state |

**Invariants (Business rules that MUST ALWAYS hold):**

```
INV-001: [Rule name]
  - Rule: [Price must always be greater than 0]
  - Violation: [An entity with price <= 0 cannot be saved]
  - Implementation: Validate in the constructor and in the attribute setter

INV-002: [Rule name]
  - Rule: [The owner of an entity cannot be changed once assigned]
  - Violation: DomainException is thrown if an attempt is made to change ownerID
  - Implementation: The setter validates that ownerID is still null
```

**Code example (TypeScript/Java):**

```typescript
// TypeScript — Entity with invariants
class Order {
  private constructor(
    private readonly id: OrderId,
    private status: OrderStatus,
    private items: OrderItem[],
    private total: Money,
  ) {}

  static create(items: OrderItem[]): Order {
    if (items.length === 0) {
      throw new DomainException('INV-001: An order must have at least one item');
    }
    const total = items.reduce((sum, item) => sum.add(item.subtotal), Money.zero('COP'));
    return new Order(OrderId.new(), OrderStatus.PENDING, items, total);
  }

  confirm(): void {
    if (this.status !== OrderStatus.PENDING) {
      throw new DomainException('INV-002: Only a PENDING order can be confirmed');
    }
    this.status = OrderStatus.CONFIRMED;
    // Record domain event
    this.addEvent(new OrderConfirmedEvent(this.id, this.total));
  }
}
```

---

## System Value Objects

### Value Object: [VOName]

**Description:** [What it represents]

**Attributes:**

| Attribute | Type | Description |
|-----------|------|-------------|
| [field1] | [type] | [description] |

**Validation rules:**

```
- [Email must have a valid format: text@domain.extension]
- [The extension must be at least 2 characters]
```

**Example:**

```typescript
// Value Object — Immutable, validated in the constructor
class Email {
  private readonly value: string;

  constructor(email: string) {
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
      throw new DomainException(`Invalid email: ${email}`);
    }
    this.value = email.toLowerCase();
  }

  toString(): string { return this.value; }

  equals(other: Email): boolean { return this.value === other.value; }
}
```

---

## System Aggregates

### Aggregate: [AggregateName]

**Aggregate Root:** [RootEntityName]

**Internal entities:**
- [InternalEntity1] — [why it is inside the aggregate]
- [InternalEntity2] — [why it is inside the aggregate]

**Value Objects:**
- [VO1], [VO2]

**Aggregate invariants:**

```
AGGR-INV-001: The sum of items.subtotal must equal aggregate.total
AGGR-INV-002: An item cannot be added if the order is in CONFIRMED status
AGGR-INV-003: No two items can have the same productId
```

**Why do these objects form an aggregate?**
> [Explanation of why these objects must be kept consistent as a unit.
> E.g.: "An Order and its Items must always be consistent — an Item cannot exist
> without its Order, and the Order total must always reflect the sum of the Items."]

---

## Summary table of tactical building blocks

| Name | Type | Bounded Context | Aggregate Root? |
|------|------|----------------|----------------|
| [Entity A] | Entity | [Context A] | Yes |
| [Entity B] | Entity | [Context A] | No (inside A) |
| [VO: Email] | Value Object | Shared | N/A |
| [VO: Money] | Value Object | Shared | N/A |
| [Service X] | Domain Service | [Context B] | N/A |

---

## Domain Services

A **Domain Service** is business logic that does not naturally belong to any entity.
Use it when:
- The operation involves multiple entities or aggregates
- It would be unnatural for the operation to belong to a single entity
- The logic does not need its own state

```typescript
// Domain Service — Stateless, orchestrates logic between entities
class PriceCalculationService {
  calculateTotal(items: OrderItem[], discounts: Discount[], taxes: Tax[]): Money {
    const subtotal = items.reduce((sum, item) => sum.add(item.subtotal), Money.zero('COP'));
    const withDiscount = discounts.reduce((total, d) => d.apply(total), subtotal);
    const withTaxes = taxes.reduce((total, tax) => tax.apply(total), withDiscount);
    return withTaxes;
  }
}
```

---

## Correlation with code

| Domain artifact | Package / folder in code | File |
|----------------|--------------------------|------|
| Aggregate Root `Order` | `src/domain/order/` | `Order.ts` |
| Value Object `Email` | `src/domain/shared/value-objects/` | `Email.ts` |
| Domain Service `PriceCalculationService` | `src/domain/order/services/` | `PriceCalculationService.ts` |
| Repository `OrderRepository` | `src/domain/order/ports/` | `OrderRepository.ts` |

> See hexagonal structure in `05-architecture/hexagonal-architecture.md`
