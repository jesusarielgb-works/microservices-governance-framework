# Arquitectura Hexagonal (Ports & Adapters)

> La arquitectura hexagonal, propuesta por Alistair Cockburn, organiza un servicio de forma
> que el **dominio de negocio sea completamente independiente** de la tecnología que lo rodea.
> La base de datos, el framework web, el message broker — todos son detalles intercambiables.
> Lo que importa es la lógica de negocio, que vive en el centro.

> **Nota de stack:** Los conceptos de este documento son válidos para cualquier lenguaje.
> Los ejemplos de código y la estructura de carpetas específica de tu tecnología están en:
> - Node.js + TypeScript → [`_stacks/node-typescript.md`](../_stacks/node-typescript.md)
> - Java + Spring Boot → [`_stacks/java-spring.md`](../_stacks/java-spring.md)
> - Python + FastAPI → [`_stacks/python-fastapi.md`](../_stacks/python-fastapi.md)
> - Go → [`_stacks/go.md`](../_stacks/go.md)

---

## El problema que resuelve

```
❌ Arquitectura tradicional en capas:

  [Controller HTTP]
       ↓
  [Service]
       ↓
  [Repository]
       ↓
  [Base de datos]

Problema: El "Service" mezcla lógica de negocio con llamadas a frameworks.
Si cambias el framework, rompes el negocio. Si quieres testear el negocio,
necesitas simular la base de datos.
```

```
✓ Arquitectura Hexagonal:

  [HTTP Controller]  [CLI]  [Test]  ← Adaptadores Primarios (entran al hexágono)
          │            │      │
          └────────────┴──────┘
                       │
                 [Puerto de Entrada]  ← Interface que define la API del dominio
                       │
               ┌───────────────┐
               │               │
               │    DOMINIO    │  ← Lógica de negocio pura, sin dependencias externas
               │               │
               └───────────────┘
                       │
                 [Puerto de Salida]  ← Interface que el dominio necesita del mundo exterior
                       │
          ┌────────────┴──────┐
          │                   │
  [Adaptador BD]  [Adaptador Kafka]  ← Adaptadores Secundarios (salen del hexágono)
```

---

## Estructura de carpetas

```
src/
├── domain/                          # El hexágono — sin frameworks, sin dependencias externas
│   ├── [aggregate]/
│   │   ├── [Aggregate].ts           # Aggregate Root con invariantes
│   │   ├── [Aggregate]Id.ts         # Value Object para el ID
│   │   ├── events/
│   │   │   └── [EventoOcurrido].ts  # Eventos de dominio
│   │   ├── services/
│   │   │   └── [DomainService].ts   # Lógica que no pertenece a una entidad
│   │   └── ports/                   # Interfaces (puertos) — contratos abstractos
│   │       ├── in/
│   │       │   └── [UseCasePort].ts # Puerto de entrada: contrato del caso de uso
│   │       └── out/
│   │           └── [RepoPort].ts    # Puerto de salida: contrato del repositorio
│   └── shared/
│       └── value-objects/           # VOs compartidos entre aggregates
│           ├── Email.ts
│           └── Dinero.ts
│
├── application/                     # Casos de uso — orquesta el dominio
│   └── [aggregate]/
│       ├── [CrearXxxUseCase].ts     # Implementa el puerto de entrada
│       └── dtos/
│           ├── [CrearXxxRequest].ts
│           └── [CrearXxxResponse].ts
│
├── infrastructure/                  # Todo lo externo al hexágono
│   ├── adapters/
│   │   ├── in/                      # Adaptadores primarios — reciben llamadas externas
│   │   │   ├── http/
│   │   │   │   ├── [XxxController].ts
│   │   │   │   └── [XxxRouter].ts
│   │   │   └── messaging/
│   │   │       └── [XxxEventConsumer].ts
│   │   └── out/                     # Adaptadores secundarios — llaman al exterior
│   │       ├── persistence/
│   │       │   └── [XxxRepositoryImpl].ts   # Implementa el puerto de salida
│   │       ├── messaging/
│   │       │   └── [XxxEventPublisher].ts
│   │       └── external/
│   │           └── [ExternalApiAdapter].ts
│   └── config/
│       ├── database.ts
│       └── container.ts             # Inyección de dependencias (IoC)
│
└── main.ts                          # Bootstrap — conecta adaptadores con puertos
```

---

## Los Puertos

Los puertos son **interfaces** (contratos abstractos). El dominio los define;
los adaptadores los implementan.

### Puerto de entrada (Driving Port)

Define lo que el dominio puede hacer — su API pública desde el punto de vista del exterior.

```typescript
// src/domain/pedido/ports/in/CrearPedidoPort.ts
export interface CrearPedidoPort {
  ejecutar(request: CrearPedidoRequest): Promise<CrearPedidoResponse>;
}
```

### Puerto de salida (Driven Port)

Define lo que el dominio necesita del mundo exterior — sin saber cómo se implementa.

```typescript
// src/domain/pedido/ports/out/PedidoRepositoryPort.ts
export interface PedidoRepositoryPort {
  guardar(pedido: Pedido): Promise<void>;
  buscarPorId(id: PedidoId): Promise<Pedido | null>;
  buscarPorCliente(clienteId: ClienteId): Promise<Pedido[]>;
}

// src/domain/pedido/ports/out/EventPublisherPort.ts
export interface EventPublisherPort {
  publicar(evento: DomainEvent): Promise<void>;
}
```

---

## Los Adaptadores

### Adaptador Primario — HTTP Controller

El controlador HTTP traduce la solicitud HTTP al caso de uso del dominio.

```typescript
// src/infrastructure/adapters/in/http/PedidoController.ts
import { CrearPedidoPort } from '@domain/pedido/ports/in/CrearPedidoPort';

@Controller('/pedidos')
export class PedidoController {
  constructor(
    // Inyecta el puerto, NO la implementación concreta
    private readonly crearPedido: CrearPedidoPort,
  ) {}

  @Post('/')
  async crear(@Body() body: CrearPedidoHttpRequest): Promise<void> {
    // Traduce HTTP request → DTO del dominio
    const request = new CrearPedidoRequest(body.clienteId, body.items);
    // Llama al caso de uso a través del puerto
    const response = await this.crearPedido.ejecutar(request);
    return response;
  }
}
```

### Adaptador Secundario — Repository

El repositorio implementa el puerto de salida. El dominio no sabe que existe PostgreSQL.

```typescript
// src/infrastructure/adapters/out/persistence/PedidoRepositoryImpl.ts
import { PedidoRepositoryPort } from '@domain/pedido/ports/out/PedidoRepositoryPort';

export class PedidoRepositoryImpl implements PedidoRepositoryPort {
  constructor(private readonly db: DatabaseConnection) {}

  async guardar(pedido: Pedido): Promise<void> {
    // Traduce Aggregate → fila de base de datos
    await this.db.query(
      'INSERT INTO pedidos (id, cliente_id, estado, total) VALUES ($1, $2, $3, $4)',
      [pedido.id.value, pedido.clienteId.value, pedido.estado, pedido.total.amount],
    );
  }

  async buscarPorId(id: PedidoId): Promise<Pedido | null> {
    const row = await this.db.queryOne('SELECT * FROM pedidos WHERE id = $1', [id.value]);
    if (!row) return null;
    // Traduce fila de base de datos → Aggregate
    return PedidoMapper.toDomain(row);
  }
}
```

---

## El Caso de Uso (Application Service)

El caso de uso orquesta el dominio. Usa puertos de entrada y salida. No contiene lógica de negocio — esa vive en el Aggregate.

```typescript
// src/application/pedido/CrearPedidoUseCase.ts
import { CrearPedidoPort } from '@domain/pedido/ports/in/CrearPedidoPort';
import { PedidoRepositoryPort } from '@domain/pedido/ports/out/PedidoRepositoryPort';
import { EventPublisherPort } from '@domain/pedido/ports/out/EventPublisherPort';

export class CrearPedidoUseCase implements CrearPedidoPort {
  constructor(
    private readonly pedidoRepo: PedidoRepositoryPort,
    private readonly eventPublisher: EventPublisherPort,
  ) {}

  async ejecutar(request: CrearPedidoRequest): Promise<CrearPedidoResponse> {
    // 1. Crear el aggregate (la lógica de negocio vive AQUÍ, en el dominio)
    const pedido = Pedido.crear(request.clienteId, request.items);

    // 2. Persistir (a través del puerto — el caso de uso no sabe qué BD se usa)
    await this.pedidoRepo.guardar(pedido);

    // 3. Publicar eventos de dominio (a través del puerto)
    for (const evento of pedido.domainEvents) {
      await this.eventPublisher.publicar(evento);
    }

    return new CrearPedidoResponse(pedido.id.value);
  }
}
```

---

## La Regla de Dependencia

> **Las dependencias siempre apuntan hacia adentro.**
> El dominio no importa nada de la aplicación ni de la infraestructura.
> La infraestructura importa del dominio (pero nunca al revés).

```
infrastructure/ → application/ → domain/
                                    ↑
                         NO puede importar nada de application/ ni infrastructure/
```

### Inversión de dependencia (DI) en la práctica

```typescript
// ✓ Correcto — dominio define la interface, infraestructura la implementa
// En domain/:
export interface PedidoRepositoryPort { ... }

// En infrastructure/:
export class PedidoRepositoryImpl implements PedidoRepositoryPort { ... }

// En el bootstrap (main.ts), la implementación concreta se inyecta:
const pedidoRepo = new PedidoRepositoryImpl(dbConnection);
const crearPedidoUseCase = new CrearPedidoUseCase(pedidoRepo, eventPublisher);
const pedidoController = new PedidoController(crearPedidoUseCase);
```

---

## Ventajas para TDD

La arquitectura hexagonal es ideal para TDD porque:

1. **El dominio es testeable sin mocks de frameworks.** No necesitas levantar un servidor
   ni una base de datos para probar la lógica de negocio.

2. **Los puertos de salida se pueden fakeear fácilmente.** En los tests, usas un
   repositorio en memoria (Fake) en vez del real.

3. **Las invariantes son explícitas** y se testean en aislamiento.

```typescript
// Test unitario del dominio — cero dependencias externas
describe('Pedido', () => {
  it('no puede crearse sin ítems', () => {
    expect(() => Pedido.crear(clienteId, [])).toThrow('INV-001');
  });

  it('al confirmar cambia estado a CONFIRMADO', () => {
    const pedido = Pedido.crear(clienteId, [itemValido]);
    pedido.confirmar();
    expect(pedido.estado).toBe(EstadoPedido.CONFIRMADO);
  });

  it('al confirmar emite evento PedidoConfirmado', () => {
    const pedido = Pedido.crear(clienteId, [itemValido]);
    pedido.confirmar();
    expect(pedido.domainEvents).toContainEqual(expect.any(PedidoConfirmadoEvent));
  });
});

// Test del caso de uso con repositorio FAKE (no mock de BD real)
describe('CrearPedidoUseCase', () => {
  it('guarda el pedido y publica el evento', async () => {
    const fakePedidoRepo = new InMemoryPedidoRepository();
    const fakeEventPublisher = new InMemoryEventPublisher();
    const useCase = new CrearPedidoUseCase(fakePedidoRepo, fakeEventPublisher);

    await useCase.ejecutar(new CrearPedidoRequest(clienteId, [itemValido]));

    expect(fakePedidoRepo.pedidos).toHaveLength(1);
    expect(fakeEventPublisher.eventos).toContainEqual(expect.any(PedidoCreado));
  });
});
```

> Ver guía completa de TDD en `11-quality/tdd-guide.md`

---

## Checklist de Arquitectura Hexagonal

Al revisar un PR o nuevo servicio, verifica:

- [ ] `domain/` no tiene imports de `infrastructure/` ni de `application/`
- [ ] `domain/` no tiene imports de frameworks (Express, NestJS, TypeORM, etc.)
- [ ] Toda interfaz de repositorio vive en `domain/ports/out/`
- [ ] Toda interfaz de caso de uso vive en `domain/ports/in/`
- [ ] Los mappers (`toDomain` / `toPersistence`) viven en `infrastructure/`, no en `domain/`
- [ ] Los DTOs de la API HTTP viven en `infrastructure/adapters/in/http/`, no en `domain/`
- [ ] Existe un test unitario para cada invariante del Aggregate

---

## Errores comunes (anti-patrones)

| Anti-patrón | Por qué es malo | Solución |
|------------|-----------------|---------|
| `import { Repository } from 'typeorm'` en el dominio | Acopla el dominio a TypeORM | Definir interface port propia |
| Lógica de negocio en el Controller | Si cambias el endpoint, cambia el negocio | Mover al Aggregate |
| Repository que devuelve DTOs en vez de Aggregates | El dominio no puede validar invariantes | Usar Mapper para reconstruir el Aggregate |
| Caso de uso con 15 dependencias | Probablemente hace demasiado | Dividir en casos de uso más pequeños |
| `any` en las interfaces de los puertos | Pierdes el contrato tipado | Tipado explícito siempre |

---

## Referencias y correlaciones

- Bounded Contexts → `02-domain/domain-map.md`
- Entidades e invariantes → `02-domain/entities-and-rules.md`
- Eventos de dominio → `02-domain/domain-events.md`
- Patrones complementarios (CQRS, Event Sourcing, Saga) → `05-architecture/pattern-guide.md`
- TDD aplicado a la arquitectura hexagonal → `11-quality/tdd-guide.md`
- Template de servicio con estructura hexagonal → `09-microservices/_template/service/`
