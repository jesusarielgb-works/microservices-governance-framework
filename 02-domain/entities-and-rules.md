# Entidades, Value Objects y Reglas de Negocio

> **Qué llenar aquí:** Los bloques de construcción del dominio siguiendo el modelo táctico del DDD.
> Este documento traduce el conocimiento del dominio (obtenido en Event Storming) a modelos de código.

> **Nota de stack:** Los conceptos de Entity, Value Object y Aggregate son independientes del lenguaje.
> Los ejemplos de código (clases, interfaces, decoradores) están escritos en pseudo-TypeScript
> para ilustrar la idea. Para ver la implementación en tu tecnología:
> [`_stacks/node-typescript.md`](../_stacks/node-typescript.md) ·
> [`_stacks/java-spring.md`](../_stacks/java-spring.md) ·
> [`_stacks/python-fastapi.md`](../_stacks/python-fastapi.md) ·
> [`_stacks/go.md`](../_stacks/go.md)

---

## Conceptos del DDD táctico

### Entidad (Entity)
Una **Entidad** es un objeto definido por su identidad, no por sus atributos.
Dos entidades son iguales si tienen el mismo ID, aunque todos sus demás atributos difieran.

```
✓ Entidad: Usuario (dos usuarios con diferente email siguen siendo distintos por su ID)
✓ Entidad: Pedido (cambia de estado pero sigue siendo el mismo pedido)
✗ No es entidad: Dinero (10 USD == 10 USD independientemente de qué billete)
```

### Value Object (VO)
Un **Value Object** es un objeto definido por sus atributos, no tiene identidad propia.
Es inmutable — si cambia un atributo, es un nuevo VO.

```
✓ Value Object: Dirección (Calle 5 #10-20, Neiva, Huila)
✓ Value Object: Dinero (USD 150.00)
✓ Value Object: Email (usuario@ejemplo.com)
✓ Value Object: RangoFecha (2024-01-01 → 2024-01-31)
```

### Aggregate
Un **Aggregate** es un cluster de entidades y VOs tratados como una unidad.
Tiene un **Aggregate Root** que es el punto de entrada — solo se puede acceder a los
objetos internos a través de la raíz.

```
Pedido (Aggregate Root)
  ├── ItemsPedido[] (Entidades dentro del aggregate)
  ├── DireccionEntrega (Value Object)
  └── TotalPedido (Value Object calculado)
```

**Regla de oro del Aggregate:** Las transacciones no cruzan fronteras de aggregates.
Si necesitas modificar dos aggregates en una operación, usa un Evento de Dominio y una Saga.

### Reglas de Negocio
Las **Reglas de Negocio** (invariantes) son las restricciones que el dominio siempre debe cumplir.
Viven en el Aggregate Root y se validan en cada operación.

---

## Entidades del sistema

### Entidad: [NombreEntidad]

**Contexto:** [Bounded Context al que pertenece]

**Descripción:** [Qué representa en el negocio, en una oración]

**Atributos:**

| Atributo | Tipo | Descripción | Obligatorio | Reglas |
|----------|------|-------------|-------------|--------|
| id | UUID | Identificador único | Sí | Generado automáticamente al crear |
| [atributo] | [tipo] | [descripción] | [Sí/No] | [validaciones] |
| createdAt | DateTime | Fecha de creación | Sí | Inmutable, asignado al crear |
| updatedAt | DateTime | Última modificación | Sí | Actualizado automáticamente |

**Ciclo de vida / Estados:**

```
[Estado A] ──(acción)──▶ [Estado B] ──(acción)──▶ [Estado C]
                                │
                          (acción)
                                ▼
                          [Estado D]
```

| Estado | Descripción | Transiciones permitidas |
|--------|-------------|------------------------|
| [BORRADOR] | Recién creado, no publicado | → ACTIVO, → CANCELADO |
| [ACTIVO] | Disponible para uso | → INACTIVO, → CANCELADO |
| [CANCELADO] | Finalizado sin completarse | Estado terminal |

**Invariantes (Reglas de negocio que SIEMPRE deben cumplirse):**

```
INV-001: [Nombre de la regla]
  - Regla: [El precio siempre debe ser mayor a 0]
  - Violación: [No se puede guardar una entidad con precio <= 0]
  - Implementación: Validar en el constructor y en el setter del atributo

INV-002: [Nombre de la regla]
  - Regla: [No se puede cambiar el dueño de una entidad una vez asignado]
  - Violación: Se lanza DomainException si se intenta cambiar ownerID
  - Implementación: El setter valida que ownerID aún sea null
```

**Ejemplo en código (TypeScript/Java):**

```typescript
// TypeScript — Entity con invariantes
class Pedido {
  private constructor(
    private readonly id: PedidoId,
    private estado: EstadoPedido,
    private items: ItemPedido[],
    private total: Dinero,
  ) {}

  static crear(items: ItemPedido[]): Pedido {
    if (items.length === 0) {
      throw new DomainException('INV-001: Un pedido debe tener al menos un ítem');
    }
    const total = items.reduce((sum, item) => sum.sumar(item.subtotal), Dinero.cero('COP'));
    return new Pedido(PedidoId.nuevo(), EstadoPedido.PENDIENTE, items, total);
  }

  confirmar(): void {
    if (this.estado !== EstadoPedido.PENDIENTE) {
      throw new DomainException('INV-002: Solo se puede confirmar un pedido PENDIENTE');
    }
    this.estado = EstadoPedido.CONFIRMADO;
    // Registrar evento de dominio
    this.addEvent(new PedidoConfirmadoEvent(this.id, this.total));
  }
}
```

---

## Value Objects del sistema

### Value Object: [NombreVO]

**Descripción:** [Qué representa]

**Atributos:**

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| [campo1] | [tipo] | [descripción] |

**Reglas de validación:**

```
- [El email debe tener formato válido: texto@dominio.extensión]
- [La extensión debe ser de al menos 2 caracteres]
```

**Ejemplo:**

```typescript
// Value Object — Inmutable, validado en el constructor
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

## Aggregates del sistema

### Aggregate: [NombreAggregate]

**Aggregate Root:** [NombreEntidadRaíz]

**Entidades internas:**
- [EntidadInterna1] — [por qué está dentro del aggregate]
- [EntidadInterna2] — [por qué está dentro del aggregate]

**Value Objects:**
- [VO1], [VO2]

**Invariantes del Aggregate:**

```
AGGR-INV-001: La suma de items.subtotal debe igualar a aggregate.total
AGGR-INV-002: No se puede agregar un item si el pedido está en estado CONFIRMADO
AGGR-INV-003: No puede haber dos items con el mismo productoId
```

**¿Por qué estos objetos forman un aggregate?**
> [Explicación de por qué estos objetos deben mantenerse consistentes como una unidad.
> Ej: "Un Pedido y sus Items deben ser consistentes siempre — no puede existir un Item
> sin su Pedido, y el total del Pedido siempre debe reflejar la suma de los Items."]

---

## Tabla resumen de bloques tácticos

| Nombre | Tipo | Bounded Context | Aggregate Root? |
|--------|------|----------------|----------------|
| [Entidad A] | Entity | [Contexto A] | Sí |
| [Entidad B] | Entity | [Contexto A] | No (dentro de A) |
| [VO: Email] | Value Object | Compartido | N/A |
| [VO: Dinero] | Value Object | Compartido | N/A |
| [Servicio X] | Domain Service | [Contexto B] | N/A |

---

## Domain Services

Un **Domain Service** es lógica de negocio que no pertenece naturalmente a ninguna entidad.
Úsalo cuando:
- La operación involucra múltiples entidades o aggregates
- Sería antinatural que la operación pertenezca a una sola entidad
- La lógica no necesita un estado propio

```typescript
// Domain Service — Sin estado, orquesta lógica entre entidades
class ServicioCalculoPrecio {
  calcularTotal(items: ItemPedido[], descuentos: Descuento[], impuestos: Impuesto[]): Dinero {
    const subtotal = items.reduce((sum, item) => sum.sumar(item.subtotal), Dinero.cero('COP'));
    const conDescuento = descuentos.reduce((total, d) => d.aplicar(total), subtotal);
    const conImpuestos = impuestos.reduce((total, imp) => imp.aplicar(total), conDescuento);
    return conImpuestos;
  }
}
```

---

## Correlación con el código

| Artefacto de dominio | Paquete / folder en código | Archivo |
|---------------------|---------------------------|---------|
| Aggregate Root `Pedido` | `src/domain/pedido/` | `Pedido.ts` |
| Value Object `Email` | `src/domain/shared/value-objects/` | `Email.ts` |
| Domain Service `ServicioCalculoPrecio` | `src/domain/pedido/services/` | `ServicioCalculoPrecio.ts` |
| Repositorio `PedidoRepository` | `src/domain/pedido/ports/` | `PedidoRepository.ts` |

> Ver estructura hexagonal en `05-architecture/hexagonal-architecture.md`
