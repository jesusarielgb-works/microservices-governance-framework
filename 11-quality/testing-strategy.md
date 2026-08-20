# Testing Strategy

> Defines what, how much, and with which tools to test at each layer.
> Complements the TDD guide (`tdd-guide.md`) that explains the process.
> This document is the project's quality contract.

---

## Project testing pyramid

```
                 /\
                /  \
               /    \
              /  E2E  \     ← Few, slow, expensive but high-value
             /  [5%]   \
            /────────────\
           /  Integration  \
          /    [25%]        \  ← Test real integration (DB, broker, external APIs)
         /──────────────────\
        /   Contract Tests    \
       /      [20%]            \  ← Verify the OpenAPI contract is fulfilled
      /────────────────────────\
     /       Unit Tests         \
    /          [50%]             \  ← Many, fast, test business logic
   /────────────────────────────\
```

**Rule:** More tests at the bottom = more maintainable system at lower cost.
Inverting the pyramid (more E2E than unit tests) makes CI slow and fragile.

---

## Tier 1: Unit Tests

**Objective:** Test business logic in complete isolation.

| Aspect | Value |
|--------|-------|
| **What they test** | Aggregates, Value Objects, Domain Services, use cases |
| **Isolation** | Complete — no real calls to DB or external services |
| **Speed** | < 5ms per test |
| **Target coverage** | ≥ 80% of lines (≥ 90% in `src/domain/`) |
| **Tools** | Jest, Vitest, JUnit 5, pytest |
| **Doubles** | Fakes, Stubs, Spies (see `tdd-guide.md`) |
| **When they run** | On every `git commit` (pre-commit hook) and in CI |

**Folder structure:**

```
tests/
└── unit/
    ├── domain/
    │   ├── [Aggregate].spec.ts
    │   └── value-objects/
    │       └── [VO].spec.ts
    └── application/
        └── [UseCase].spec.ts
```

**Unit test example (see complete guide in `tdd-guide.md`):**

```typescript
// tests/unit/domain/Order.spec.ts
it('total should be the sum of items × quantity', () => {
  const items = [
    new OrderItem(prod1Id, 2, new Money(100, 'USD')),
    new OrderItem(prod2Id, 1, new Money(50, 'USD')),
  ];
  const order = Order.create(customerId, items);
  expect(order.total).toEqual(new Money(250, 'USD'));
});
```

---

## Tier 2: Integration Tests

**Objective:** Verify that the adapters work correctly with real systems.

| Aspect | Value |
|--------|-------|
| **What they test** | Repositories vs real DB, publishers vs real broker |
| **Isolation** | Low — use real DB and broker in Docker |
| **Speed** | 50ms – 2s per test |
| **Target coverage** | Happy path + 2-3 error cases per adapter |
| **Tools** | Jest + Testcontainers, Spring Boot Test |
| **Setup** | `docker compose up -d test-db test-broker` |
| **When they run** | In CI, NOT in pre-commit (too slow) |

**Setup with Testcontainers:**

```typescript
// tests/integration/setup.ts
import { PostgreSqlContainer } from '@testcontainers/postgresql';

let pgContainer: StartedPostgreSqlContainer;

beforeAll(async () => {
  pgContainer = await new PostgreSqlContainer('postgres:15')
    .withDatabase('test_db')
    .withUsername('test')
    .withPassword('test')
    .start();

  process.env.DATABASE_URL = pgContainer.getConnectionUri();

  // Apply migrations
  await runMigrations(process.env.DATABASE_URL);
}, 60_000); // 60s timeout for image pull

afterAll(async () => {
  await pgContainer.stop();
});
```

---

## Tier 3: Contract Tests (Consumer-Driven Contract Testing)

**Objective:** Verify that the OpenAPI contract (what the service documents) matches what the service actually implements.

| Aspect | Value |
|--------|-------|
| **What they test** | HTTP API vs OpenAPI contract |
| **Tools** | Pact, Dredd, supertest + openapi-validator |
| **When they run** | In CI before pushing to staging |

**Validation with openapi-validator:**

```typescript
// tests/contract/api-contract.spec.ts
import { createOpenAPIValidatorMiddleware } from 'express-openapi-validator';

const app = buildApp();
app.use(createOpenAPIValidatorMiddleware({
  apiSpec: './07-api/contracts/openapi/[service].yaml',
  validateRequests: true,
  validateResponses: true,
}));

it('POST /orders returns 201 with the correct schema', async () => {
  const response = await request(app)
    .post('/orders')
    .set('Authorization', `Bearer ${testJWT}`)
    .send(validOrderPayload);

  expect(response.status).toBe(201);
  // Response schema validation is done by the OpenAPI middleware
});
```

---

## Tier 4: End-to-End Tests (E2E)

**Objective:** Verify complete user flows as the end user would experience them.

| Aspect | Value |
|--------|-------|
| **What they test** | Critical business flows end-to-end |
| **Isolation** | None — real environment (staging or dedicated E2E environment) |
| **Speed** | 5s – 60s per test |
| **Tools** | Playwright (UI), K6 (API E2E), Cypress |
| **When they run** | In CI only in the staging pipeline, not on every PR |
| **Scope** | Critical flows only — register + create order + confirm payment |

**Priority E2E flows:**

| # | Flow | Services involved |
|---|------|------------------|
| 1 | User registration and login | auth-service |
| 2 | [Main business flow] | [services] |
| 3 | [Critical error flow] | [services] |

---

## Performance Tests

**Objective:** Verify that NFR performance requirements are met under load.

| Aspect | Value |
|--------|-------|
| **Tool** | k6 |
| **When they run** | In the staging pipeline (not on every PR) |
| **Failure criterion** | If P95 > 300ms or error rate > 1% |

```javascript
// tests/performance/load-test.js (k6)
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 100 },  // Ramp up to 100 users
    { duration: '60s', target: 100 },  // Hold 100 users
    { duration: '30s', target: 0 },    // Ramp down
  ],
  thresholds: {
    'http_req_duration': ['p(95)<300'],  // P95 < 300ms
    'http_req_failed': ['rate<0.01'],    // < 1% errors
  },
};

export default function () {
  const res = http.get('http://localhost:8080/[critical-endpoint]', {
    headers: { 'Authorization': `Bearer ${__ENV.TEST_TOKEN}` },
  });
  check(res, { 'status is 200': (r) => r.status === 200 });
}
```

---

## CI configuration per pipeline

```yaml
# .github/workflows/ci.yml (simplified)
jobs:
  unit-tests:
    steps:
      - run: npm run test:unit -- --coverage
      - run: npm run lint

  integration-tests:
    services:
      postgres:
        image: postgres:15
      redis:
        image: redis:7
    steps:
      - run: npm run test:integration

  contract-tests:
    steps:
      - run: npm run test:contract

  # Only on push to main/develop:
  performance-tests:
    if: github.ref == 'refs/heads/main'
    steps:
      - run: k6 run tests/performance/load-test.js
```

---

## Code coverage — Thresholds per layer

```json
// jest.config.json
{
  "coverageThreshold": {
    "global": {
      "lines": 80,
      "branches": 75,
      "functions": 80,
      "statements": 80
    },
    "./src/domain/": {
      "lines": 90,
      "branches": 85
    }
  }
}
```

**Coverage rule:** If a PR lowers the global coverage, CI fails.
Coverage cannot go down — if there is uncovered code, tests must be added in the same PR.

---

## Fixture and test data files

```
tests/
├── fixtures/
│   ├── order.fixture.ts       # Test object builders
│   ├── user.fixture.ts
│   └── db-seeds/
│       └── test-data.sql       # Base data for integration tests
├── helpers/
│   ├── auth.helper.ts          # Generate test JWT
│   └── database.helper.ts      # Reset DB between tests
```

**Fixture Builder pattern:**

```typescript
// tests/fixtures/order.fixture.ts
export class OrderFixture {
  static createValid(overrides: Partial<OrderData> = {}): Order {
    return Order.create(
      overrides.customerId ?? new CustomerId('customer-test-uuid'),
      overrides.items ?? [OrderItemFixture.oneItem()],
    );
  }

  static inConfirmedState(): Order {
    const order = OrderFixture.createValid();
    order.confirm();
    return order;
  }
}
```

---

## Correlations

- Complete TDD guide → `11-quality/tdd-guide.md`
- Definition of Done (required coverage) → `00-governance/definition-of-done.md`
- OpenAPI contracts to validate → `07-api/contracts/openapi/`
- Hexagonal Architecture (facilitates testing) → `05-architecture/hexagonal-architecture.md`
- CI/CD pipeline → `10-devops/README.md`
