# Guía de Patrones de Diseño y Microservicios

> Este documento es el catálogo de patrones del proyecto.
> Para cada patrón: cuándo usarlo, cuándo NO, y ejemplo de implementación.
> Los patrones no son recetas — son herramientas. Úsalos cuando el problema lo requiere.

> **Nota de stack:** Las descripciones y diagramas son agnósticos de tecnología.
> Los fragmentos de código ilustrativos usan pseudo-TypeScript como lenguaje de referencia
> por su cercanía a la sintaxis de pseudocódigo. Para ver la implementación concreta en tu stack:
> [`_stacks/node-typescript.md`](../_stacks/node-typescript.md) ·
> [`_stacks/java-spring.md`](../_stacks/java-spring.md) ·
> [`_stacks/python-fastapi.md`](../_stacks/python-fastapi.md) ·
> [`_stacks/go.md`](../_stacks/go.md)

---

## Índice

**Patrones de diseño (GoF y SOLID)**
1. [Patrones Creacionales](#creacionales)
2. [Patrones Estructurales](#estructurales)
3. [Patrones de Comportamiento](#comportamiento)

**Patrones de microservicios**
4. [Decomposición del sistema](#descomposicion)
5. [Comunicación entre servicios](#comunicacion)
6. [Resiliencia](#resiliencia)
7. [Datos y consistencia](#datos)
8. [Observabilidad](#observabilidad)

---

## Patrones de diseño (GoF) {#creacionales}

### 1. Factory Method

**Problema:** Quieres crear objetos sin exponer la lógica de creación ni acoplar el código al tipo concreto.

**Cuándo usarlo:**
- Cuando el tipo exacto del objeto a crear no se conoce hasta runtime
- Cuando la creación tiene lógica compleja (validaciones, configuración)

**Ejemplo en dominio:**

```typescript
// Factory Method — dentro del Aggregate Root
class Pedido {
  // En vez de new Pedido(...), usamos un factory method
  static crear(clienteId: ClienteId, items: ItemPedido[]): Pedido {
    if (items.length === 0) throw new DomainException('INV-001');
    return new Pedido(PedidoId.nuevo(), clienteId, items, EstadoPedido.PENDIENTE);
  }

  static reconstituir(data: PedidoData): Pedido {
    // Para reconstruir desde la base de datos
    return new Pedido(new PedidoId(data.id), new ClienteId(data.clienteId), ...);
  }
}
```

---

### 2. Builder

**Problema:** Un objeto tiene muchos parámetros opcionales y la construcción se vuelve ilegible.

**Cuándo usarlo:** Objetos de configuración complejos, test data builders.

```typescript
// Builder — especialmente útil para tests
const pedido = new PedidoBuilder()
  .conCliente('cliente-id-123')
  .conItem(producto1, cantidad: 2)
  .conItem(producto2, cantidad: 1)
  .conDireccion('Calle 5 #10-20, Neiva')
  .enEstado(EstadoPedido.CONFIRMADO)
  .build();
```

---

### 3. Singleton (con precaución)

**Problema:** Una clase debe tener exactamente una instancia.

**Cuándo usarlo:** Conexiones a BD, registros de configuración.

**ADVERTENCIA:** El Singleton dificulta las pruebas. Preferir inyección de dependencias.

```typescript
// ✓ Mejor: Singleton gestionado por el contenedor DI, no por la clase misma
// En el contenedor (NestJS, tsyringe, etc.):
container.registerSingleton(DatabaseConnection, DatabaseConnectionImpl);
```

---

### 4. Adapter (Patrón estructural)

**Problema:** Quieres usar una clase existente pero su interfaz no coincide con la que necesitas.

**Cuándo usarlo:** Integración con APIs externas, librerías de terceros.

```typescript
// El dominio define la interface que necesita
interface PaymentGatewayPort {
  cobrar(monto: Dinero, tarjeta: DatosToken): Promise<ResultadoCobro>;
}

// El adaptador traduce al API externa
class StripePaymentAdapter implements PaymentGatewayPort {
  constructor(private stripe: Stripe) {}

  async cobrar(monto: Dinero, tarjeta: DatosToken): Promise<ResultadoCobro> {
    // Traduce el modelo del dominio → modelo de Stripe
    const charge = await this.stripe.charges.create({
      amount: monto.toCentavos(),
      currency: monto.currency,
      source: tarjeta.token,
    });
    // Traduce el resultado de Stripe → modelo del dominio
    return new ResultadoCobro(charge.id, charge.status === 'succeeded');
  }
}
```

---

### 5. Decorator

**Problema:** Quieres agregar comportamiento a un objeto sin modificarlo ni heredar de él.

**Cuándo usarlo:** Logging, caching, validación, rate limiting alrededor de casos de uso.

```typescript
// Decorator de caché alrededor del repositorio
class CachedPedidoRepository implements PedidoRepositoryPort {
  constructor(
    private readonly repo: PedidoRepositoryPort,
    private readonly cache: CachePort,
  ) {}

  async buscarPorId(id: PedidoId): Promise<Pedido | null> {
    const cached = await this.cache.get(`pedido:${id.value}`);
    if (cached) return PedidoMapper.toDomain(cached);

    const pedido = await this.repo.buscarPorId(id);
    if (pedido) await this.cache.set(`pedido:${id.value}`, pedido, TTL_5_MINUTES);
    return pedido;
  }
}
```

---

### 6. Observer / Event Bus interno

**Problema:** Un objeto necesita notificar a otros sin conocerlos directamente.

**Cuándo usarlo:** Para publicar eventos de dominio después de persistir el aggregate.

```typescript
// El Aggregate acumula eventos — el UseCase los publica
class Pedido {
  private readonly _events: DomainEvent[] = [];

  confirmar(): void {
    // ... lógica de negocio ...
    this._events.push(new PedidoConfirmado(this.id));
  }

  get domainEvents(): DomainEvent[] {
    return [...this._events];
  }

  clearEvents(): void {
    this._events.length = 0;
  }
}
```

---

### 7. Strategy

**Problema:** Quieres intercambiar algoritmos en tiempo de ejecución.

**Cuándo usarlo:** Estrategias de descuento, algoritmos de cálculo, métodos de pago.

```typescript
interface EstrategiaDescuento {
  calcular(subtotal: Dinero, usuario: Usuario): Dinero;
}

class DescuentoEstudiante implements EstrategiaDescuento {
  calcular(subtotal: Dinero, usuario: Usuario): Dinero {
    return subtotal.multiplicar(0.15); // 15% descuento
  }
}

class DescuentoEmpresarial implements EstrategiaDescuento {
  calcular(subtotal: Dinero, usuario: Usuario): Dinero {
    return subtotal.multiplicar(0.20); // 20% descuento
  }
}
```

---

### 8. Template Method

**Problema:** Un algoritmo tiene una estructura fija pero algunos pasos varían.

**Cuándo usarlo:** Flujos de proceso con variaciones (exportar a CSV, Excel, PDF).

```typescript
abstract class ExportadorReporte {
  // Template Method — estructura fija
  async exportar(datos: DatosReporte): Promise<Buffer> {
    const validados = await this.validar(datos);
    const transformados = await this.transformar(validados);
    const buffer = await this.generar(transformados);
    await this.registrarExportacion(datos.usuarioId);
    return buffer;
  }

  protected abstract transformar(datos: DatosReporte): Promise<DatosTransformados>;
  protected abstract generar(datos: DatosTransformados): Promise<Buffer>;
  
  // Pasos con implementación por defecto (pueden sobrescribirse)
  protected async validar(datos: DatosReporte): Promise<DatosReporte> { return datos; }
  protected async registrarExportacion(userId: UserId): Promise<void> {}
}
```

---

## Patrones de Microservicios

### Descomposición {#descomposicion}

#### API Gateway

**Problema:** Los clientes necesitan llamar a múltiples servicios para obtener una respuesta.

```
                    ┌─────────────────┐
Móvil ──────────▶  │                 │ ──▶ [Servicio A]
Web ────────────▶  │   API Gateway   │ ──▶ [Servicio B]
IoT ────────────▶  │                 │ ──▶ [Servicio C]
                    └─────────────────┘
                         Hace:
                    - Routing
                    - Auth/AuthZ
                    - Rate limiting
                    - SSL termination
                    - Request aggregation
```

**Cuándo usarlo:** Siempre, en arquitecturas de microservicios es esencial.

**Herramientas:** Kong, AWS API Gateway, NGINX, Traefik, Spring Cloud Gateway.

---

#### Backend for Frontend (BFF)

**Problema:** El móvil y el web necesitan datos con formato muy diferente pero comparten el mismo API.

```
Mobile ──▶ [BFF Mobile]  ──▶ Servicios internos
Web    ──▶ [BFF Web]     ──▶ Servicios internos
Alexa  ──▶ [BFF Voice]   ──▶ Servicios internos
```

**Cuándo usarlo:** Cuando los clientes tienen necesidades muy diferentes. Con moderación — cada BFF es una API que mantener.

---

#### Strangler Fig (Migración incremental)

**Problema:** Necesitas migrar un monolito a microservicios sin reescribirlo de golpe.

```
Fase 1:  Cliente → Monolito (100% tráfico)
Fase 2:  Cliente → API Gateway → Monolito (70%) + Nuevo Servicio (30%)
Fase 3:  Cliente → API Gateway → Nuevo Servicio (100%) — monolito retirado
```

**Cómo:** El API Gateway desvía tráfico gradualmente al nuevo servicio mientras el monolito sigue funcionando.

---

### Comunicación entre servicios {#comunicacion}

#### Síncronoː REST / gRPC

| Aspecto | REST | gRPC |
|---------|------|------|
| Protocolo | HTTP/1.1 o HTTP/2 | HTTP/2 |
| Serialización | JSON (legible) | Protocol Buffers (eficiente) |
| Tipado | Manual con OpenAPI | Automático con .proto |
| Streaming | No nativo | Sí (unidireccional y bidireccional) |
| Uso recomendado | APIs públicas, comunicación externa | Comunicación interna entre servicios |

**Cuándo usar comunicación síncrona:**
- Cuando necesitas la respuesta inmediatamente (consultas, UI)
- Operaciones de baja latencia que el usuario espera

---

#### Asíncronoː Message Broker (Kafka / RabbitMQ)

```
[Servicio A] ──publica──▶ [Topic/Queue] ──consume──▶ [Servicio B]
                                                       [Servicio C]
```

**Cuándo usar comunicación asíncrona:**
- Cuando la operación no requiere respuesta inmediata
- Cuando quieres desacoplar productores de consumidores
- Para procesar en background (emails, notificaciones, reportes)
- Para garantizar entrega (la BD del broker es durable)

---

### Resiliencia {#resiliencia}

#### Circuit Breaker

**Problema:** Un servicio lento o fallando hace que el tuyo también falle (cascada de fallos).

```
Estado CLOSED (normal):
  Llamadas pasan → si N fallos consecutivos → pasar a OPEN

Estado OPEN (cortocircuito):
  Llamadas bloqueadas inmediatamente (fail fast) → después de T segundos → HALF-OPEN

Estado HALF-OPEN (prueba):
  Permite 1 llamada → si falla: volver a OPEN | si pasa: volver a CLOSED
```

```typescript
// Con opossum o resilience4j
const circuit = new CircuitBreaker(servicioExterno.llamar, {
  timeout: 3000,           // Timeout por llamada
  errorThresholdPercentage: 50, // % de errores para abrir
  resetTimeout: 30000,     // Tiempo en OPEN antes de intentar HALF-OPEN
});

circuit.fallback(() => ({ cached: true, data: ultimoCacheConfiable }));
```

---

#### Retry con Backoff Exponencial

**Problema:** Fallos transitorios (red inestable, servicio reiniciando).

```typescript
async function conRetry<T>(
  fn: () => Promise<T>,
  opciones = { intentos: 3, backoffBase: 1000 }
): Promise<T> {
  for (let intento = 1; intento <= opciones.intentos; intento++) {
    try {
      return await fn();
    } catch (err) {
      if (intento === opciones.intentos) throw err;
      const delay = opciones.backoffBase * Math.pow(2, intento - 1); // 1s, 2s, 4s
      await sleep(delay + Math.random() * 100); // Jitter para evitar thundering herd
    }
  }
}
```

---

### Datos y consistencia {#datos}

#### Database per Service

**Regla:** Cada microservicio tiene su propia base de datos. Ningún servicio accede directamente
a la base de datos de otro.

```
✓ Correcto:
  Servicio A → Base de datos A
  Servicio B → Base de datos B

✗ Incorrecto:
  Servicio A → Base de datos B (JOIN directo)
```

**¿Cómo comparto datos entonces?** Con APIs o eventos, nunca con SQL directo.

---

#### Saga (Transacciones distribuidas)

**Problema:** Una transacción de negocio abarca múltiples servicios y no puedes usar una transacción ACID distribuida.

```
Saga Coreografiada (via eventos):

  [Pedidos]                [Inventario]           [Pagos]
     │ PedidoCreado            │                     │
     │ ─────────────────────▶  │                     │
     │                    StockReservado             │
     │ ◀─────────────────────  │                     │
     │ PedidoStockConfirmado                         │
     │ ─────────────────────────────────────────▶   │
     │                                         PagoAprobado
     │ ◀─────────────────────────────────────────   │
```

**Compensaciones:** Si un paso falla, ejecuta transacciones compensadoras en orden inverso.

```
Paso 1: Reservar stock         → Compensación: Liberar stock
Paso 2: Debitar pago           → Compensación: Reembolsar
Paso 3: Confirmar pedido       → Compensación: Cancelar pedido
```

---

#### CQRS (Command Query Responsibility Segregation)

**Problema:** La lógica para escribir datos es muy diferente de la lógica para leerlos.
Un modelo único fuerza compromisos subóptimos para ambos.

```
Escritura (Commands):                    Lectura (Queries):
  POST /pedidos                            GET /pedidos?clienteId=X
       │                                         │
       ▼                                         ▼
  [Command Handler]                       [Query Handler]
       │                                         │
       ▼                                         ▼
  [Aggregate]                            [Read Model / Projection]
       │                                 (desnormalizado, optimizado para lectura)
       ▼
  [Event Store / Write DB]
       │
       ▼ (actualiza la lectura via eventos)
  [Read DB]
```

**Cuándo usarlo:** Cuando el volumen de lecturas es mucho mayor que el de escrituras, o cuando las consultas son muy complejas de hacer sobre el modelo de escritura.

**Precaución:** Aumenta la complejidad. No siempre vale la pena.

---

#### Outbox Pattern (Transaccional)

**Problema:** Necesitas garantizar que cuando guardas en la base de datos, también publicas el evento — sin riesgo de publicarlo dos veces o no publicarlo si hay un fallo.

```
❌ Sin Outbox (puede perder eventos):
  BEGIN TRANSACTION
    INSERT INTO pedidos ...
  COMMIT
  // Si el sistema cae aquí, el evento se pierde
  publishEvent(PedidoCreado)

✓ Con Outbox (atómico):
  BEGIN TRANSACTION
    INSERT INTO pedidos ...
    INSERT INTO outbox (event_type, payload, published) VALUES ('PedidoCreado', '...', false)
  COMMIT
  // Proceso separado lee outbox y publica
  // Si falla la publicación, el outbox sigue teniendo el evento
```

```sql
-- Tabla outbox
CREATE TABLE outbox (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  event_type  VARCHAR(100) NOT NULL,
  payload     JSONB NOT NULL,
  created_at  TIMESTAMPTZ DEFAULT NOW(),
  published_at TIMESTAMPTZ,
  published   BOOLEAN DEFAULT false
);

-- Índice para el Relay (proceso que publica eventos pendientes)
CREATE INDEX idx_outbox_unpublished ON outbox (created_at) WHERE published = false;
```

---

#### Event Sourcing

**Problema:** Necesitas auditoría completa, reproducir el estado del sistema en cualquier punto del tiempo, o reconstruir proyecciones.

```
Tradicional:   BD guarda estado actual → "Un pedido vale $150"
Event Sourcing: BD guarda eventos       → "PedidoCreado($100) + DescuentoAplicado($-30) + ItemAgregado($80)"

Para saber el estado actual: reproduces todos los eventos en orden.
```

**Cuándo usarlo:** Auditoría financiera, debugging avanzado, sistemas donde el historial importa.

**Cuándo NO usarlo:** La mayoría de casos. Agrega complejidad significativa. No es la solución por defecto.

---

### Observabilidad {#observabilidad}

#### Sidecar Pattern

**Problema:** Quieres agregar capacidades de observabilidad, configuración, o red a un servicio sin modificar su código.

```
Pod de Kubernetes:
  ┌──────────────────────────────┐
  │  [Servicio A]               │
  │  [Sidecar: Envoy/Istio]    │  ← Maneja TLS, métricas, service mesh
  │  [Sidecar: Filebeat]       │  ← Recolecta logs
  └──────────────────────────────┘
```

---

## Cuándo NO usar cada patrón

| Patrón | No usarlo cuando... |
|--------|---------------------|
| CQRS | El modelo de lectura y escritura son similares. Solo agrega complejidad. |
| Event Sourcing | No necesitas historial completo. Es difícil de implementar y mantener. |
| Saga | La transacción cabe en un solo servicio. Usa una transacción ACID simple. |
| Circuit Breaker | La llamada es interna al mismo servicio. No vale el overhead. |
| BFF | Los clientes tienen necesidades similares. Un API Gateway estándar es suficiente. |

---

## Patrones adoptados en este proyecto

> **Llenar con las decisiones de tu proyecto específico.**
> Para cada patrón: decide si se adopta, documenta el ADR que justifica la decisión,
> y enlaza la sección de este mismo documento donde aprendiste cuándo usarlo.

| Patrón | ¿Adoptado? | Justificación / ADR |
|--------|------------|---------------------|
| API Gateway | [Sí / No — ver ADR-NNN] | [Razón breve] |
| Database per Service | [Sí / No — ver ADR-NNN] | [Razón breve] |
| Circuit Breaker | [Sí / No — ver ADR-NNN] | [Razón breve] |
| Saga (coreografiada) | [Sí / No — ver ADR-NNN] | [Razón breve] |
| Outbox Pattern | [Sí / No — ver ADR-NNN] | [Razón breve] |
| CQRS | [Sí / No — ver ADR-NNN] | [Razón breve] |
| Event Sourcing | [Sí / No — ver ADR-NNN] | [Razón breve] |
| BFF | [Sí / No — ver ADR-NNN] | [Razón breve] |

---

## Correlaciones

- Hexagonal Architecture → `05-architecture/hexagonal-architecture.md`
- ADR de decisiones de patrones → `05-architecture/decisions/`
- Implementación de Saga → `09-microservices/services/XX/events.md`
- Runbook de Circuit Breaker → `09-microservices/services/XX/runbook.md`
- Outbox en el modelo de datos → `06-data/models.md`
