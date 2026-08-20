# Guía TDD — Test-Driven Development

> El TDD no es sobre testing — es sobre **diseño**. Escribir el test primero te fuerza a pensar
> en la interfaz antes que en la implementación. El resultado: código más simple, más desacoplado
> y con una suite de pruebas que documenta el comportamiento del sistema.

> **Nota de stack:** Los principios y ciclos de TDD descritos aquí aplican a cualquier lenguaje.
> Los ejemplos de código concretos (test runner, librerías de mocks, comandos) están en:
> - Node.js + TypeScript → [`_stacks/node-typescript.md`](../_stacks/node-typescript.md)
> - Java + Spring Boot → [`_stacks/java-spring.md`](../_stacks/java-spring.md)
> - Python + FastAPI → [`_stacks/python-fastapi.md`](../_stacks/python-fastapi.md)
> - Go → [`_stacks/go.md`](../_stacks/go.md)

---

## El ciclo Red-Green-Refactor

```
        ┌──────────────────────────────────────────────────────────┐
        │                                                          │
        ▼                                                          │
   ┌─────────┐                                                     │
   │   RED   │  Escribe el test más pequeño que puede fallar.      │
   │  🔴     │  NO implementes nada todavía.                       │
   └────┬────┘  El test debe fallar por la razón correcta.         │
        │                                                          │
        ▼                                                          │
   ┌─────────┐                                                     │
   │  GREEN  │  Escribe el código MÍNIMO para que el test pase.    │
   │  🟢     │  No busques elegancia aquí. Solo haz que pase.      │
   └────┬────┘                                                     │
        │                                                          │
        ▼                                                          │
   ┌──────────────┐                                                │
   │   REFACTOR   │  Mejora el código sin cambiar el comportamiento│
   │  ♻️          │  Los tests deben seguir en verde.              │
   └──────────────┘                                                │
        │                                                          │
        └──────────────────────────────────────────────────────────┘
```

**La regla de los 3 momentos:**
1. `RED`: El test falla — confirma que el test puede detectar el bug
2. `GREEN`: El test pasa — el código hace lo mínimo necesario
3. `REFACTOR`: El código es limpio — sin duplicación, bien nombrado

---

## Los 3 tipos de testeo (FIRST)

Los buenos tests son:

| Letra | Principio | Descripción |
|-------|-----------|-------------|
| **F** | Fast | Se ejecutan en milisegundos, no segundos |
| **I** | Isolated | No dependen de otros tests ni de orden de ejecución |
| **R** | Repeatable | El mismo resultado siempre, independiente del ambiente |
| **S** | Self-validating | Pass / Fail sin interpretación manual |
| **T** | Timely | Escritos ANTES del código, no después |

---

## Test Doubles: la taxonomía completa

Cuando el dominio necesita colaboradores externos (repositorios, APIs), los reemplazamos
en los tests con doubles. No todos los doubles son iguales:

### 1. Dummy
No hace nada. Se pasa para satisfacer una firma pero nunca se llama.

```typescript
const dummyLogger = {} as Logger; // Nunca se llama, solo llena el constructor
const useCase = new CrearPedidoUseCase(repo, eventPublisher, dummyLogger);
```

### 2. Stub
Devuelve respuestas hardcodeadas. Sin verificación de llamadas.

```typescript
class StubInventarioRepository implements InventarioRepositoryPort {
  async verificarDisponibilidad(productoId: ProductoId): Promise<boolean> {
    return true; // Siempre disponible — control del escenario de test
  }
}
```

### 3. Fake
Implementación real pero simplificada. Tiene estado, funciona correctamente pero de forma ligera.

```typescript
class InMemoryPedidoRepository implements PedidoRepositoryPort {
  private readonly store = new Map<string, Pedido>();

  async guardar(pedido: Pedido): Promise<void> {
    this.store.set(pedido.id.value, pedido);
  }

  async buscarPorId(id: PedidoId): Promise<Pedido | null> {
    return this.store.get(id.value) ?? null;
  }

  // Útil para aserciones en tests
  get todos(): Pedido[] {
    return Array.from(this.store.values());
  }
}
```

### 4. Spy
Registra las llamadas que recibe. Puedes verificar si fue llamado y con qué argumentos.

```typescript
class SpyEventPublisher implements EventPublisherPort {
  readonly eventosPublicados: DomainEvent[] = [];

  async publicar(evento: DomainEvent): Promise<void> {
    this.eventosPublicados.push(evento);
  }
}

// En el test:
expect(spyPublisher.eventosPublicados).toHaveLength(1);
expect(spyPublisher.eventosPublicados[0]).toBeInstanceOf(PedidoCreado);
```

### 5. Mock
Tiene expectativas pre-programadas. Falla si no se llama como se espera.

```typescript
// Jest mock
const mockRepo = {
  guardar: jest.fn().mockResolvedValue(undefined),
  buscarPorId: jest.fn().mockResolvedValue(null),
};

// Después del test:
expect(mockRepo.guardar).toHaveBeenCalledTimes(1);
expect(mockRepo.guardar).toHaveBeenCalledWith(expect.objectContaining({
  clienteId: clienteIdEsperado,
}));
```

### ¿Cuándo usar cada uno?

| Double | Cuándo usarlo |
|--------|--------------|
| Dummy | El colaborador no importa en este test |
| Stub | Controlas el escenario de entrada (qué devuelve un colaborador) |
| Fake | Tests de integración ligeros — el comportamiento importa |
| Spy | Verificas que algo fue llamado (efecto de salida) |
| Mock | Verificas tanto el comportamiento de entrada como de salida |

**Preferencia:** Fake > Stub/Spy > Mock. Los mocks son frágiles — se rompen si refactorizas la implementación interna.

---

## TDD por capa (con Arquitectura Hexagonal)

### Capa 1: Dominio — Pruebas unitarias de aggregates

Son las pruebas más valiosas. Testean la lógica de negocio pura.
**Sin mocks de infraestructura.** No hay base de datos. No hay HTTP.

```typescript
// tests/unit/domain/Pedido.spec.ts
describe('Pedido — invariantes', () => {
  describe('crear()', () => {
    it('falla si no hay ítems (INV-001)', () => {
      expect(() => Pedido.crear(clienteId, [])).toThrow('INV-001');
    });

    it('falla si un ítem tiene cantidad cero', () => {
      const itemInvalido = new ItemPedido(productoId, 0, precio);
      expect(() => Pedido.crear(clienteId, [itemInvalido])).toThrow();
    });

    it('calcula el total correctamente', () => {
      const items = [
        new ItemPedido(producto1, 2, new Dinero(100, 'COP')), // 200
        new ItemPedido(producto2, 1, new Dinero(50, 'COP')),  // 50
      ];
      const pedido = Pedido.crear(clienteId, items);
      expect(pedido.total).toEqual(new Dinero(250, 'COP'));
    });

    it('emite el evento PedidoCreado al crearse', () => {
      const pedido = Pedido.crear(clienteId, [itemValido]);
      expect(pedido.domainEvents).toHaveLength(1);
      expect(pedido.domainEvents[0]).toBeInstanceOf(PedidoCreado);
    });
  });

  describe('confirmar()', () => {
    it('solo puede confirmarse si está en estado PENDIENTE (INV-002)', () => {
      const pedido = Pedido.crear(clienteId, [itemValido]);
      pedido.confirmar();
      expect(() => pedido.confirmar()).toThrow('INV-002');
    });

    it('cambia estado a CONFIRMADO', () => {
      const pedido = Pedido.crear(clienteId, [itemValido]);
      pedido.confirmar();
      expect(pedido.estado).toBe(EstadoPedido.CONFIRMADO);
    });
  });
});
```

**Paso TDD:**
1. 🔴 Escribe `it('falla si no hay ítems', ...)` — falla porque `Pedido.crear` no existe
2. 🟢 Implementa `Pedido.crear` con la validación mínima
3. ♻️ Refactoriza el mensaje de error para ser más descriptivo

---

### Capa 2: Aplicación — Pruebas de casos de uso

Testean la orquestación. Usan Fakes para repositorios y Spies para publishers.

```typescript
// tests/unit/application/CrearPedidoUseCase.spec.ts
describe('CrearPedidoUseCase', () => {
  let pedidoRepo: InMemoryPedidoRepository;
  let eventPublisher: SpyEventPublisher;
  let useCase: CrearPedidoUseCase;

  beforeEach(() => {
    pedidoRepo = new InMemoryPedidoRepository();
    eventPublisher = new SpyEventPublisher();
    useCase = new CrearPedidoUseCase(pedidoRepo, eventPublisher);
  });

  it('guarda el pedido en el repositorio', async () => {
    const request = new CrearPedidoRequest(clienteId, [itemValido]);
    await useCase.ejecutar(request);
    expect(pedidoRepo.todos).toHaveLength(1);
  });

  it('publica el evento PedidoCreado', async () => {
    await useCase.ejecutar(new CrearPedidoRequest(clienteId, [itemValido]));
    expect(eventPublisher.eventosPublicados[0]).toBeInstanceOf(PedidoCreado);
  });

  it('retorna el ID del pedido creado', async () => {
    const response = await useCase.ejecutar(new CrearPedidoRequest(clienteId, [itemValido]));
    expect(response.pedidoId).toBeDefined();
    expect(typeof response.pedidoId).toBe('string');
  });

  it('propaga el error del dominio si los ítems están vacíos', async () => {
    await expect(
      useCase.ejecutar(new CrearPedidoRequest(clienteId, []))
    ).rejects.toThrow('INV-001');
  });
});
```

---

### Capa 3: Infraestructura — Pruebas de integración

Testean que los adaptadores interactúan correctamente con sistemas externos.
**Usan la base de datos real** (en un contenedor Docker local).

```typescript
// tests/integration/PedidoRepositoryImpl.spec.ts
describe('PedidoRepositoryImpl (integración con PostgreSQL)', () => {
  let db: DatabaseConnection;
  let repo: PedidoRepositoryImpl;

  beforeAll(async () => {
    db = await createTestDatabaseConnection(); // Base de datos de test en Docker
    await db.migrate(); // Aplica migraciones
  });

  afterAll(async () => {
    await db.close();
  });

  beforeEach(async () => {
    await db.query('TRUNCATE TABLE pedidos CASCADE');
    repo = new PedidoRepositoryImpl(db);
  });

  it('guarda y recupera un pedido correctamente', async () => {
    const pedidoOriginal = Pedido.crear(clienteId, [itemValido]);
    await repo.guardar(pedidoOriginal);

    const recuperado = await repo.buscarPorId(pedidoOriginal.id);

    expect(recuperado).not.toBeNull();
    expect(recuperado!.id.value).toBe(pedidoOriginal.id.value);
    expect(recuperado!.total).toEqual(pedidoOriginal.total);
  });
});
```

---

### Capa 4: API — Pruebas de contrato (Consumer-Driven Contract Testing)

Verifican que el contrato OpenAPI se cumple en la implementación real.

```typescript
// Con Pact o supertest + OpenAPI
describe('POST /pedidos — contrato', () => {
  it('responde 201 con el id del pedido', async () => {
    const response = await request(app)
      .post('/pedidos')
      .set('Authorization', `Bearer ${testToken}`)
      .send({
        clienteId: 'cliente-uuid',
        items: [{ productoId: 'prod-uuid', cantidad: 1, precio: { amount: 100, currency: 'COP' } }],
      });

    expect(response.status).toBe(201);
    expect(response.body.pedidoId).toMatch(UUID_REGEX);
  });

  it('responde 400 si el body está malformado', async () => {
    const response = await request(app)
      .post('/pedidos')
      .set('Authorization', `Bearer ${testToken}`)
      .send({ clienteId: 'no-es-uuid' }); // items faltante

    expect(response.status).toBe(400);
    expect(response.body.error).toBe('VALIDATION_ERROR');
  });
});
```

---

## Cobertura de código

La cobertura es una señal, no un objetivo en sí mismo.

| Métrica | Objetivo mínimo | Notas |
|---------|----------------|-------|
| Líneas — dominio | 90%+ | El core del negocio debe estar muy cubierto |
| Líneas — aplicación | 80%+ | Casos de uso deben estar cubiertos |
| Líneas — infraestructura | 60%+ | Tests de integración cubren los paths principales |
| Ramas (branches) | 75%+ | Los if/else deben tener tests para ambos caminos |

**Lo que la cobertura NO dice:**
- No dice si los tests son significativos
- No dice si cubres los edge cases correctos
- Un test que solo ejecuta líneas sin aserciones puede dar 100% pero no prueba nada

---

## TDD con Value Objects

Los VOs son los objetos más fáciles de probar con TDD. Empieza por ellos.

```typescript
// 🔴 Primero el test
describe('Email', () => {
  it('crea un email válido', () => {
    expect(() => new Email('usuario@ejemplo.com')).not.toThrow();
  });

  it('rechaza email sin @', () => {
    expect(() => new Email('noesvalido')).toThrow();
  });

  it('normaliza a minúsculas', () => {
    const email = new Email('USUARIO@EJEMPLO.COM');
    expect(email.toString()).toBe('usuario@ejemplo.com');
  });

  it('dos emails con el mismo valor son iguales', () => {
    const e1 = new Email('test@test.com');
    const e2 = new Email('test@test.com');
    expect(e1.equals(e2)).toBe(true);
  });
});

// 🟢 Implementación mínima para pasar los tests
class Email {
  private readonly value: string;

  constructor(email: string) {
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
      throw new DomainException(`Email inválido: ${email}`);
    }
    this.value = email.toLowerCase();
  }

  toString(): string { return this.value; }
  equals(other: Email): boolean { return this.value === other.value; }
}
```

---

## Orden de implementación TDD en un sprint

1. **Escribir los tests de dominio primero** (Aggregates, VOs, reglas de negocio)
2. **Implementar el dominio** hasta que los tests pasen
3. **Escribir los tests del caso de uso** con Fakes de los puertos
4. **Implementar el caso de uso**
5. **Escribir los tests de integración** para los adaptadores
6. **Implementar los adaptadores** (repositorio, publisher, controlador HTTP)
7. **Escribir el test de contrato** del endpoint HTTP
8. **Refactorizar** en cualquier momento mientras los tests están en verde

---

## Configuración del stack de testing

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

## Correlaciones

- Arquitectura Hexagonal (facilita el TDD) → `05-architecture/hexagonal-architecture.md`
- Pirámide de testing → `11-quality/README.md`
- Estrategia de testing por tipo → `11-quality/testing-strategy.md`
- Definition of Done (cobertura requerida) → `00-governance/definition-of-done.md`
