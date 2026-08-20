# Eventos de Dominio

> **Qué llenar aquí:** Un evento de dominio es un hecho que ocurrió en el negocio.
> Son la columna vertebral de la comunicación asíncrona entre bounded contexts.
> El nombre SIEMPRE en pasado y en el lenguaje ubicuo del dominio.

---

## ¿Qué es un evento de dominio?

Un **Evento de Dominio** comunica que algo importante ocurrió en el negocio.
Es un mensaje inmutable que describe el hecho en pasado.

```
✓ PedidoCreado
✓ PagoRechazado
✓ UsuarioRegistrado
✓ StockAgotado

✗ CrearPedido (esto es un comando, no un evento)
✗ PedidoActualizado (demasiado genérico, ¿qué cambió?)
✗ EventoPedido (no indica qué ocurrió)
```

### Diferencia entre Comando y Evento

| Concepto | Intención | Tiempo | Puede fallar? |
|----------|-----------|--------|---------------|
| **Comando** | Instrucción para hacer algo | Presente | Sí |
| **Evento** | Notificación de algo que ocurrió | Pasado | No (ya ocurrió) |

```
Usuario → [CrearPedido] → Sistema → [PedidoCreado] → Otros contextos
            (Comando)                   (Evento)
```

---

## Catálogo de eventos

### Evento: [NombreEvento]

| Campo | Valor |
|-------|-------|
| **Nombre** | `[NombreDelEvento]` |
| **Bounded Context** | [Contexto origen] |
| **Aggregate** | [Aggregate que lo genera] |
| **Disparador** | [Qué acción de negocio genera este evento] |
| **Consumidores** | [Qué servicios/contextos escuchan este evento] |
| **Canal (topic)** | `[nombre.del.topic]` |
| **Versión del esquema** | `v1` |
| **Garantía de entrega** | At-least-once / At-most-once / Exactly-once |

**Payload (esquema JSON):**

```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440000",
  "eventType": "[NombreDelEvento]",
  "aggregateId": "550e8400-e29b-41d4-a716-446655440001",
  "aggregateType": "[NombreAggregate]",
  "occurredAt": "2024-01-15T10:30:00Z",
  "version": 1,
  "payload": {
    "[campo1]": "[tipo y descripción]",
    "[campo2]": "[tipo y descripción]"
  },
  "metadata": {
    "correlationId": "550e8400-e29b-41d4-a716-446655440002",
    "causationId": "550e8400-e29b-41d4-a716-446655440003",
    "userId": "550e8400-e29b-41d4-a716-446655440004"
  }
}
```

**Ejemplo real del payload:**

```json
{
  "eventId": "uuid-generado",
  "eventType": "[NombreDelEvento]",
  "aggregateId": "uuid-del-aggregate",
  "aggregateType": "[NombreAggregate]",
  "occurredAt": "2024-01-15T10:30:00Z",
  "version": 1,
  "payload": {
    "[campo1]": "valor de ejemplo",
    "[campo2]": 150.00
  }
}
```

**¿Qué hacen los consumidores con este evento?**

| Servicio consumidor | Acción | Idempotente? |
|--------------------|--------|--------------|
| [Servicio A] | [Actualiza su modelo de datos] | Sí — usa eventId como idempotency key |
| [Servicio B] | [Envía notificación] | Sí — verifica si notif ya fue enviada |

---

## Campos estándar de todos los eventos

Todos los eventos deben incluir estos campos en la envoltura:

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `eventId` | UUID | ID único del evento (para idempotencia) |
| `eventType` | string | Nombre del evento en PascalCase |
| `aggregateId` | UUID | ID del aggregate que generó el evento |
| `aggregateType` | string | Tipo del aggregate |
| `occurredAt` | ISO 8601 | Cuándo ocurrió el hecho de negocio |
| `version` | integer | Versión del esquema (para evolución) |
| `payload` | object | Datos del evento (específico por tipo) |
| `metadata.correlationId` | UUID | Para rastrear una transacción a través de servicios |
| `metadata.causationId` | UUID | ID del evento o comando que causó este evento |
| `metadata.userId` | UUID | Usuario que inició la cadena (si aplica) |

---

## Flujo de eventos: [Nombre del flujo]

> Documenta aquí los flujos de eventos para los procesos de negocio principales.
> Usa el formato de Event Storming: naranja=evento, azul=comando, verde=vista/policy, amarillo=aggregate.

```
[Actor]
  │
  │  [ComandoA]           [ComandoB]           [ComandoC]
  ▼      │                    │                    │
[AggregateA]          [AggregateB]          [AggregateC]
  │                        ▲                    ▲
  │   [EventoA]            │   [EventoB]        │
  └──────────────────────▶│──────────────────▶│
```

### Ejemplo: Flujo de creación de pedido

```
Cliente
  │
  │  CrearPedido (comando)
  ▼
[Aggregate: Pedido]
  │
  │  PedidoCreado (evento)
  ├──────────────────────────────────┐
  │                                   ▼
  │                          [Servicio: Inventario]
  │                          Descuenta stock
  │                          StockReservado (evento)
  │
  │  PedidoCreado (evento)
  └──────────────────────────────────┐
                                      ▼
                            [Servicio: Notificaciones]
                            Envía email al cliente
```

---

## Estrategia de evolución de esquemas

Los eventos son contratos. Cambiarlos de forma incompatible rompe a los consumidores.

### ¿Qué es un cambio compatible (no rompe)?

```
✓ Agregar un campo opcional nuevo al payload
✓ Agregar un nuevo tipo de evento
✓ Hacer un campo obligatorio → opcional
```

### ¿Qué es un cambio incompatible (rompe)?

```
✗ Eliminar un campo del payload
✗ Cambiar el tipo de un campo (string → number)
✗ Hacer un campo opcional → obligatorio
✗ Cambiar el nombre del evento
```

### Cómo evolucionar un esquema sin romper consumidores

**Estrategia: Versionar el evento**

```
Paso 1: Publicar EventoV2 (nuevo tipo con cambios incompatibles)
Paso 2: Publicar ambos EventoV1 y EventoV2 durante el período de migración
Paso 3: Migrar consumidores a V2 uno por uno
Paso 4: Deprecar EventoV1 (avisar con 1 sprint de anticipación)
Paso 5: Eliminar la publicación de EventoV1
```

---

## Tabla resumen de eventos

| Evento | Contexto origen | Topic | Consumidores | Versión |
|--------|----------------|-------|-------------|---------|
| [EventoA] | [ContextoA] | `[topic.a]` | [SvcB, SvcC] | v1 |
| [EventoB] | [ContextoB] | `[topic.b]` | [SvcA] | v1 |

---

## Políticas (Policies) — Reactions a eventos

Una **Policy** (o Saga paso) describe qué ocurre automáticamente cuando llega un evento.
Es la lógica de "siempre que X ocurra, hacer Y".

```
Evento:  PedidoCreado
Policy:  Siempre que un PedidoCreado llegue con tipo=URGENTE,
         emitir el comando NotificarEquipoOperaciones
```

| Evento disparador | Policy | Comando emitido | Servicio |
|------------------|--------|-----------------|---------|
| [EventoA] | Siempre que [condición], entonces... | [ComandoB] | [ServicioX] |

---

## Patrones de resiliencia para eventos

### At-least-once delivery + Idempotencia

El message broker garantiza que el evento se entregue **al menos una vez** pero puede
entregarse más de una vez (en caso de reintentos). Los consumidores deben ser **idempotentes**.

```typescript
// Consumidor idempotente — guarda el eventId procesado
async function procesarEventoCreado(event: PedidoCreado): Promise<void> {
  // 1. Verificar si ya fue procesado
  if (await eventoYaProcesado(event.eventId)) {
    logger.info(`Evento ${event.eventId} ya procesado, ignorando`);
    return;
  }

  // 2. Procesar el evento
  await actualizarModelo(event.payload);

  // 3. Marcar como procesado (en la misma transacción)
  await marcarEventoProcesado(event.eventId);
}
```

### Dead Letter Queue (DLQ)

Cuando un evento falla después de N reintentos, va a la DLQ.

| Configuración | Valor recomendado |
|--------------|------------------|
| Reintentos antes de DLQ | 3-5 |
| Backoff | Exponencial (1s → 2s → 4s → 8s) |
| Retención en DLQ | 7 días |
| Alerta | Cuando DLQ tenga > 0 mensajes |

> Ver runbook de DLQ en `09-microservices/services/XX-servicio/runbook.md`
