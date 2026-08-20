# TDD Guide — Test-Driven Development

> TDD is not about testing — it is about **design**. Writing the test first forces you to think
> about the interface before the implementation. The result: simpler, more decoupled code
> with a test suite that documents system behavior.

> **Stack note:** The TDD principles and cycles described here apply to any language.
> Concrete code examples (test runner, mock libraries, commands) are in:
> - Node.js + TypeScript → [`_stacks/node-typescript.md`](../_stacks/node-typescript.md)
> - Java + Spring Boot → [`_stacks/java-spring.md`](../_stacks/java-spring.md)
> - Python + FastAPI → [`_stacks/python-fastapi.md`](../_stacks/python-fastapi.md)
> - Go → [`_stacks/go.md`](../_stacks/go.md)

---

## The Red-Green-Refactor cycle

```
        ┌──────────────────────────────────────────────────────────┐
        │                                                          │
        ▼                                                          │
   ┌─────────┐                                                     │
   │   RED   │  Write the smallest test that can fail.             │
   │  🔴     │  Do NOT implement anything yet.                     │
   └────┬────┘  The test must fail for the right reason.           │
        │                                                          │
        ▼                                                          │
   ┌─────────┐                                                     │
   │  GREEN  │  Write the MINIMUM code to make the test pass.      │
   │  🟢     │  Do not aim for elegance here. Just make it pass.   │
   └────┬────┘                                                     │
        │                                                          │
        ▼                                                          │
   ┌──────────────┐                                                │
   │   REFACTOR   │  Improve code without changing behavior.       │
   │  ♻️          │  Tests must remain green.                      │
   └──────────────┘                                                │
        │                                                          │
        └──────────────────────────────────────────────────────────┘
```

**The rule of 3 moments:**
1. `RED`: The test fails — confirms the test can detect the bug
2. `GREEN`: The test passes — the code does the bare minimum needed
3. `REFACTOR`: The code is clean — no duplication, well named

---

## The 3 testing principles (FIRST)

Good tests are:

| Letter | Principle | Description |
|--------|-----------|-------------|
| **F** | Fast | Run in milliseconds, not seconds |
| **I** | Isolated | Do not depend on other tests or execution order |
| **R** | Repeatable | Same result every time, regardless of environment |
| **S** | Self-validating | Pass / Fail without manual interpretation |
| **T** | Timely | Written BEFORE the code, not after |

---

## Test Doubles: the complete taxonomy

When the domain needs external collaborators (repositories, APIs), we replace them
in tests with doubles. Not all doubles are the same:

### 1. Dummy
Does nothing. Passed to satisfy a signature but never called.

```typescript
const dummyLogger = {} as Logger; // Never called, just fills the constructor
const useCase = new CreateOrderUseCase(repo, eventPublisher, dummyLogger);
```

### 2. Stub
Returns hardcoded responses. No call verification.

```typescript
class StubInventoryRepository implements InventoryRepositoryPort {
  async checkAvailability(productId: ProductId): Promise<boolean> {
    return true; // Always available — controls the test scenario
  }
}
```

### 3. Fake
Real but simplified implementation. Has state, works correctly but lightweight.

```typescript
class InMemoryOrderRepository implements OrderRepositoryPort {
  private readonly store = new Map<string, Order>();

  async save(order: Order): Promise<void> {
    this.store.set(order.id.value, order);
  }

  async findById(id: OrderId): Promise<Order | null> {
    return this.store.get(id.value) ?? null;
  }

  // Useful for assertions in tests
  get all(): Order[] {
    return Array.from(this.store.values());
  }
}
```

### 4. Spy
Records the calls it receives. You can verify if it was called and with what arguments.

```typescript
class SpyEventPublisher implements EventPublisherPort {
  readonly publishedEvents: DomainEvent[] = [];

  async publish(event: DomainEvent): Promise<void> {
    this.publishedEvents.push(event);
  }
}

// In the test:
expect(spyPublisher.publishedEvents).toHaveLength(1);
expect(spyPublisher.publishedEvents[0]).toBeInstanceOf(OrderCreated);
```

### 5. Mock
Has pre-programmed expectations. Fails if not called as expected.

```typescript
// Jest mock
const mockRepo = {
  save: jest.fn().mockResolvedValue(undefined),
  findById: jest.fn().mockResolvedValue(null),
};

// After the test:
expect(mockRepo.save).toHaveBeenCalledTimes(1);
expect(mockRepo.save).toHaveBeenCalledWith(expect.objectContaining({
  customerId: expectedCustomerId,
}));
```

### When to use each one?

| Double | When to use it |
|--------|---------------|
| Dummy | The collaborator does not matter in this test |
| Stub | You control the input scenario (what a collaborator returns) |
| Fake | Lightweight integration tests — behavior matters |
| Spy | You verify that something was called (output effect) |
| Mock | You verify both input and output behavior |

**Preference:** Fake > Stub/Spy > Mock. Mocks are fragile — they break if you refactor the internal implementation.

---

## TDD by layer (with Hexagonal Architecture)

### Layer 1: Domain — Unit tests for aggregates

These are the most valuable tests. They test pure business logic.
**No infrastructure mocks.** No database. No HTTP.

```typescript
// tests/unit/domain/Order.spec.ts
describe('Order — invariants', () => {
  describe('create()', () => {
    it('fails if there are no items (INV-001)', () => {
      expect(() => Order.create(customerId, [])).toThrow('INV-001');
    });

    it('fails if an item has zero quantity', () => {
      const invalidItem = new OrderItem(productId, 0, price);
      expect(() => Order.create(customerId, [invalidItem])).toThrow();
    });

    it('calculates total correctly', () => {
      const items = [
        new OrderItem(product1, 2, new Money(100, 'USD')), // 200
        new OrderItem(product2, 1, new Money(50, 'USD')),  // 50
      ];
      const order = Order.create(customerId, items);
      expect(order.total).toEqual(new Money(250, 'USD'));
    });

    it('emits the OrderCreated event on creation', () => {
      const order = Order.create(customerId, [validItem]);
      expect(order.domainEvents).toHaveLength(1);
      expect(order.domainEvents[0]).toBeInstanceOf(OrderCreated);
    });
  });

  describe('confirm()', () => {
    it('can only be confirmed if in PENDING state (INV-002)', () => {
      const order = Order.create(customerId, [validItem]);
      order.confirm();
      expect(() => order.confirm()).toThrow('INV-002');
    });

    it('changes status to CONFIRMED', () => {
      const order = Order.create(customerId, [validItem]);
      order.confirm();
      expect(order.status).toBe(OrderStatus.CONFIRMED);
    });
  });
});
```

**TDD step:**
1. 🔴 Write `it('fails if there are no items', ...)` — fails because `Order.create` does not exist
2. 🟢 Implement `Order.create` with the minimum validation
3. ♻️ Refactor the error message to be more descriptive

---

### Layer 2: Application — Use case tests

Test orchestration. Use Fakes for repositories and Spies for publishers.

```typescript
// tests/unit/application/CreateOrderUseCase.spec.ts
describe('CreateOrderUseCase', () => {
  let orderRepo: InMemoryOrderRepository;
  let eventPublisher: SpyEventPublisher;
  let useCase: CreateOrderUseCase;

  beforeEach(() => {
    orderRepo = new InMemoryOrderRepository();
    eventPublisher = new SpyEventPublisher();
    useCase = new CreateOrderUseCase(orderRepo, eventPublisher);
  });

  it('saves the order in the repository', async () => {
    const request = new CreateOrderRequest(customerId, [validItem]);
    await useCase.execute(request);
    expect(orderRepo.all).toHaveLength(1);
  });

  it('publishes the OrderCreated event', async () => {
    await useCase.execute(new CreateOrderRequest(customerId, [validItem]));
    expect(eventPublisher.publishedEvents[0]).toBeInstanceOf(OrderCreated);
  });

  it('returns the ID of the created order', async () => {
    const response = await useCase.execute(new CreateOrderRequest(customerId, [validItem]));
    expect(response.orderId).toBeDefined();
    expect(typeof response.orderId).toBe('string');
  });

  it('propagates the domain error if items are empty', async () => {
    await expect(
      useCase.execute(new CreateOrderRequest(customerId, []))
    ).rejects.toThrow('INV-001');
  });
});
```

---

### Layer 3: Infrastructure — Integration tests

Test that adapters interact correctly with external systems.
**Use the real database** (in a local Docker container).

```typescript
// tests/integration/OrderRepositoryImpl.spec.ts
describe('OrderRepositoryImpl (integration with PostgreSQL)', () => {
  let db: DatabaseConnection;
  let repo: OrderRepositoryImpl;

  beforeAll(async () => {
    db = await createTestDatabaseConnection(); // Test database in Docker
    await db.migrate(); // Apply migrations
  });

  afterAll(async () => {
    await db.close();
  });

  beforeEach(async () => {
    await db.query('TRUNCATE TABLE orders CASCADE');
    repo = new OrderRepositoryImpl(db);
  });

  it('saves and retrieves an order correctly', async () => {
    const originalOrder = Order.create(customerId, [validItem]);
    await repo.save(originalOrder);

    const retrieved = await repo.findById(originalOrder.id);

    expect(retrieved).not.toBeNull();
    expect(retrieved!.id.value).toBe(originalOrder.id.value);
    expect(retrieved!.total).toEqual(originalOrder.total);
  });
});
```

---

### Layer 4: API — Contract tests (Consumer-Driven Contract Testing)

Verify that the OpenAPI contract is fulfilled in the real implementation.

```typescript
// With Pact or supertest + OpenAPI
describe('POST /orders — contract', () => {
  it('responds 201 with the order id', async () => {
    const response = await request(app)
      .post('/orders')
      .set('Authorization', `Bearer ${testToken}`)
      .send({
        customerId: 'customer-uuid',
        items: [{ productId: 'prod-uuid', quantity: 1, price: { amount: 100, currency: 'USD' } }],
      });

    expect(response.status).toBe(201);
    expect(response.body.orderId).toMatch(UUID_REGEX);
  });

  it('responds 400 if the body is malformed', async () => {
    const response = await request(app)
      .post('/orders')
      .set('Authorization', `Bearer ${testToken}`)
      .send({ customerId: 'not-a-uuid' }); // items missing

    expect(response.status).toBe(400);
    expect(response.body.error).toBe('VALIDATION_ERROR');
  });
});
```

---

## Code coverage

Coverage is a signal, not an end in itself.

| Metric | Minimum target | Notes |
|--------|---------------|-------|
| Lines — domain | 90%+ | The business core must be well covered |
| Lines — application | 80%+ | Use cases must be covered |
| Lines — infrastructure | 60%+ | Integration tests cover the main paths |
| Branches | 75%+ | if/else must have tests for both paths |

**What coverage does NOT tell you:**
- It does not say whether tests are meaningful
- It does not say whether you cover the right edge cases
- A test that only executes lines without assertions can give 100% but tests nothing

---

## TDD with Value Objects

VOs are the easiest objects to test with TDD. Start with them.

```typescript
// 🔴 Test first
describe('Email', () => {
  it('creates a valid email', () => {
    expect(() => new Email('user@example.com')).not.toThrow();
  });

  it('rejects email without @', () => {
    expect(() => new Email('notvalid')).toThrow();
  });

  it('normalizes to lowercase', () => {
    const email = new Email('USER@EXAMPLE.COM');
    expect(email.toString()).toBe('user@example.com');
  });

  it('two emails with the same value are equal', () => {
    const e1 = new Email('test@test.com');
    const e2 = new Email('test@test.com');
    expect(e1.equals(e2)).toBe(true);
  });
});

// 🟢 Minimum implementation to pass the tests
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

## TDD implementation order in a sprint

1. **Write domain tests first** (Aggregates, VOs, business rules)
2. **Implement the domain** until the tests pass
3. **Write use case tests** with Fakes for the ports
4. **Implement the use case**
5. **Write integration tests** for the adapters
6. **Implement the adapters** (repository, publisher, HTTP controller)
7. **Write the contract test** for the HTTP endpoint
8. **Refactor** at any point while the tests are green

---

## Testing stack configuration

```json
// package.json
{
  "scripts": {
    "test": "jest",
    "test:unit": "jest --testPathPattern='tests/unit'",
    "test:integration": "jest --testPathPattern='tests/integration' --runInBand",
    "test:coverage": "jest --coverage",
    "test:watch": "jest --watch"
  },
  "jest": {
    "testEnvironment": "node",
    "coverageThreshold": {
      "global": {
        "lines": 80,
        "branches": 75
      },
      "./src/domain/": {
        "lines": 90
      }
    }
  }
}
```

---

## Correlations

- Hexagonal Architecture (facilitates TDD) → `05-architecture/hexagonal-architecture.md`
- Testing pyramid → `11-quality/README.md`
- Testing strategy by type → `11-quality/testing-strategy.md`
- Definition of Done (required coverage) → `00-governance/definition-of-done.md`
