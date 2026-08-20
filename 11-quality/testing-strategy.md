# Estrategia de Testing

> Define qué, cuánto y con qué herramientas probar en cada capa.
> Complementa la guía TDD (`tdd-guide.md`) que explica el proceso.
> Este documento es el contrato de calidad del proyecto.

---

## Pirámide de testing del proyecto

```
                 /\
                /  \
               /    \
              /  E2E  \     ← Pocos, lentos, costosos pero de alto valor
             /  [5%]   \
            /────────────\
           /  Integration  \
          /    [25%]        \  ← Prueban la integración real (BD, broker, APIs externas)
         /──────────────────\
        /   Contract Tests    \
       /      [20%]            \  ← Verifican que el contrato OpenAPI se cumple
      /────────────────────────\
     /       Unit Tests         \
    /          [50%]             \  ← Muchos, rápidos, prueban la lógica de negocio
   /────────────────────────────\
```

**Regla:** Más tests abajo = sistema más mantenible y de menor costo.
Invertir la pirámide (más E2E que unitarios) hace el CI lento y frágil.

---

## Tier 1: Tests Unitarios

**Objetivo:** Probar la lógica de negocio en completo aislamiento.

| Aspecto | Valor |
|---------|-------|
| **Qué prueban** | Aggregates, Value Objects, Domain Services, casos de uso |
| **Aislamiento** | Completo — ninguna llamada real a BD o servicios externos |
| **Velocidad** | < 5ms por test |
| **Cobertura objetivo** | ≥ 80% de líneas (≥ 90% en `src/domain/`) |
| **Herramientas** | Jest, Vitest, JUnit 5, pytest |
| **Dobles** | Fakes, Stubs, Spies (ver `tdd-guide.md`) |
| **Cuándo corren** | En cada `git commit` (pre-commit hook) y en CI |

**Estructura de carpetas:**

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

**Ejemplo de test unitario (ver guía completa en `tdd-guide.md`):**

```typescript
// tests/unit/domain/Pedido.spec.ts
it('el total debe ser la suma de items × cantidad', () => {
  const items = [
    new ItemPedido(prod1Id, 2, new Dinero(100, 'COP')),
    new ItemPedido(prod2Id, 1, new Dinero(50, 'COP')),
  ];
  const pedido = Pedido.crear(clienteId, items);
  expect(pedido.total).toEqual(new Dinero(250, 'COP'));
});
```

---

## Tier 2: Tests de Integración

**Objetivo:** Verificar que los adaptadores funcionan correctamente con sistemas reales.

| Aspecto | Valor |
|---------|-------|
| **Qué prueban** | Repositorios vs BD real, publishers vs broker real |
| **Aislamiento** | Bajo — usan BD y broker reales en Docker |
| **Velocidad** | 50ms – 2s por test |
| **Cobertura objetivo** | Paths feliz + 2-3 casos de error por adaptador |
| **Herramientas** | Jest + Testcontainers, Spring Boot Test |
| **Setup** | `docker compose up -d test-db test-broker` |
| **Cuándo corren** | En CI, NO en pre-commit (demasiado lentos) |

**Setup con Testcontainers:**

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

  // Aplicar migraciones
  await runMigrations(process.env.DATABASE_URL);
}, 60_000); // 60s timeout para pull de imagen

afterAll(async () => {
  await pgContainer.stop();
});
```

---

## Tier 3: Tests de Contrato (Consumer-Driven Contract Testing)

**Objetivo:** Verificar que el contrato OpenAPI (lo que el servicio documenta) coincide con lo que el servicio implementa realmente.

| Aspecto | Valor |
|---------|-------|
| **Qué prueban** | API HTTP vs contrato OpenAPI |
| **Herramientas** | Pact, Dredd, supertest + openapi-validator |
| **Cuándo corren** | En CI antes de subir a staging |

**Validación con openapi-validator:**

```typescript
// tests/contract/api-contract.spec.ts
import { createOpenAPIValidatorMiddleware } from 'express-openapi-validator';

const app = buildApp();
app.use(createOpenAPIValidatorMiddleware({
  apiSpec: './07-api/contracts/openapi/[servicio].yaml',
  validateRequests: true,
  validateResponses: true,
}));

it('POST /pedidos devuelve 201 con el esquema correcto', async () => {
  const response = await request(app)
    .post('/pedidos')
    .set('Authorization', `Bearer ${testJWT}`)
    .send(validPedidoPayload);

  expect(response.status).toBe(201);
  // La validación del esquema de respuesta la hace el middleware OpenAPI
});
```

---

## Tier 4: Tests End-to-End (E2E)

**Objetivo:** Verificar flujos completos de usuario como lo haría el usuario final.

| Aspecto | Valor |
|---------|-------|
| **Qué prueban** | Flujos críticos de negocio de extremo a extremo |
| **Aislamiento** | Ninguno — ambiente real (staging o ambiente E2E dedicado) |
| **Velocidad** | 5s – 60s por test |
| **Herramientas** | Playwright (UI), K6 (API E2E), Cypress |
| **Cuándo corren** | En CI solo en el pipeline de staging, no en cada PR |
| **Scope** | Solo flujos críticos — registrar + crear pedido + confirmar pago |

**Flujos E2E prioritarios:**

| # | Flujo | Servicios involucrados |
|---|-------|----------------------|
| 1 | Registro de usuario y login | auth-service |
| 2 | [Flujo principal de negocio] | [servicios] |
| 3 | [Flujo de error crítico] | [servicios] |

---

## Tests de Performance

**Objetivo:** Verificar que los RNFs de rendimiento se cumplen bajo carga.

| Aspecto | Valor |
|---------|-------|
| **Herramienta** | k6 |
| **Cuándo corren** | En el pipeline de staging (no en cada PR) |
| **Criterio de falla** | Si P95 > 300ms o error rate > 1% |

```javascript
// tests/performance/load-test.js (k6)
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 100 },  // Rampa a 100 usuarios
    { duration: '60s', target: 100 },  // Mantener 100 usuarios
    { duration: '30s', target: 0 },    // Bajar
  ],
  thresholds: {
    'http_req_duration': ['p(95)<300'],  // P95 < 300ms
    'http_req_failed': ['rate<0.01'],    // < 1% de errores
  },
};

export default function () {
  const res = http.get('http://localhost:8080/[endpoint-critico]', {
    headers: { 'Authorization': `Bearer ${__ENV.TEST_TOKEN}` },
  });
  check(res, { 'status is 200': (r) => r.status === 200 });
}
```

---

## Configuración del CI por pipeline

```yaml
# .github/workflows/ci.yml (simplificado)
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

  # Solo en push a main/develop:
  performance-tests:
    if: github.ref == 'refs/heads/main'
    steps:
      - run: k6 run tests/performance/load-test.js
```

---

## Cobertura de código — Umbrales por capa

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

**Regla de la cobertura:** Si un PR baja la cobertura global, falla el CI.
La cobertura no puede bajar — si hay código sin tests, los tests deben agregarse en el mismo PR.

---

## Archivos de fixtures y test data

```
tests/
├── fixtures/
│   ├── pedido.fixture.ts       # Builders de objetos de test
│   ├── usuario.fixture.ts
│   └── db-seeds/
│       └── test-data.sql       # Datos base para tests de integración
├── helpers/
│   ├── auth.helper.ts          # Generar JWT de test
│   └── database.helper.ts      # Reset de BD entre tests
```

**Fixture Builder pattern:**

```typescript
// tests/fixtures/pedido.fixture.ts
export class PedidoFixture {
  static crearValido(overrides: Partial<PedidoData> = {}): Pedido {
    return Pedido.crear(
      overrides.clienteId ?? new ClienteId('cliente-test-uuid'),
      overrides.items ?? [ItemPedidoFixture.unItem()],
    );
  }

  static enEstadoConfirmado(): Pedido {
    const pedido = PedidoFixture.crearValido();
    pedido.confirmar();
    return pedido;
  }
}
```

---

## Correlaciones

- Guía TDD completa → `11-quality/tdd-guide.md`
- Definition of Done (cobertura requerida) → `00-governance/definition-of-done.md`
- Contratos OpenAPI que se validan → `07-api/contracts/openapi/`
- Hexagonal Architecture (facilita el testing) → `05-architecture/hexagonal-architecture.md`
- Pipeline de CI/CD → `10-devops/README.md`
