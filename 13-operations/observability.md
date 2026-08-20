# Observabilidad del Sistema

> Un sistema no observable no puede ser operado. La observabilidad no es un feature opcional
> que se agrega después — se diseña desde el primer sprint.
> Los 3 pilares: Logs, Métricas, Trazas.

> **Nota de stack:** Los conceptos (structured logging, RED metrics, OpenTelemetry) aplican
> a cualquier lenguaje. Los ejemplos de código usan las librerías del ecosistema Node.js
> (pino, prom-client, @opentelemetry/sdk-node). Para otras tecnologías, los equivalentes son:
> - Java: Logback/SLF4J + Micrometer + OpenTelemetry Java Agent
> - Python: structlog + prometheus_client + opentelemetry-sdk
> - Go: zap + prometheus/client_golang + go.opentelemetry.io/otel

---

## Los 3 pilares de la observabilidad

```
                    ┌───────────────────────────────────────────┐
                    │           Sistema en producción           │
                    │                                           │
                    │  [Servicio A]  [Servicio B]  [Servicio C] │
                    └──────┬──────────────┬───────────┬─────────┘
                           │              │           │
               ┌───────────┼──────────────┼───────────┼──────────┐
               ▼           ▼              ▼           ▼          │
           [LOGS]      [MÉTRICAS]     [TRAZAS]   [EVENTOS]      │
               │           │              │                       │
               ▼           ▼              ▼                       │
       [Elasticsearch] [Prometheus]  [Jaeger/Zipkin]             │
               │           │              │                       │
               ▼           ▼              │                       │
           [Kibana]    [Grafana]          │                       │
                           │              │                       │
                           └──────────────┘                      │
                                  │                              │
                           [ALERTAS] ──────────────────────────▶│
                        [Alertmanager]                          │
                        [PagerDuty]                             │
                        [Slack]                                  │
                    └──────────────────────────────────────────-┘
```

---

## Logs

### Formato estándar (JSON estructurado)

Todos los servicios deben producir logs en JSON con estos campos mínimos:

```json
{
  "timestamp": "2024-01-15T10:30:00.123Z",
  "level": "INFO",
  "service": "auth-service",
  "version": "1.3.0",
  "environment": "production",
  "correlationId": "550e8400-e29b-41d4-a716-446655440000",
  "userId": "uuid-del-usuario-si-disponible",
  "message": "Usuario autenticado correctamente",
  "context": {
    "userId": "uuid",
    "method": "POST",
    "path": "/auth/login",
    "statusCode": 200,
    "durationMs": 45
  }
}
```

**Campos obligatorios:**

| Campo | Descripción |
|-------|-------------|
| `timestamp` | ISO 8601 con milisegundos y zona UTC |
| `level` | `DEBUG`, `INFO`, `WARN`, `ERROR`, `FATAL` |
| `service` | Nombre del microservicio |
| `correlationId` | Para rastrear la transacción a través de servicios |
| `message` | Mensaje descriptivo, sin datos sensibles |

### Niveles de log

| Nivel | Cuándo usarlo | Ejemplo |
|-------|--------------|---------|
| `DEBUG` | Info de desarrollo. Desactivado en producción | "Query SQL ejecutada: SELECT..." |
| `INFO` | Eventos de negocio normales | "Pedido #123 creado" |
| `WARN` | Situación anormal pero no falla | "Token a punto de expirar", "Retry #2" |
| `ERROR` | Error que requiere atención | "Fallo al conectar a PostgreSQL" |
| `FATAL` | Error que hace caer el servicio | "No se puede iniciar: puerto en uso" |

### Lo que NO va en los logs

```
✗ Contraseñas o tokens completos
✗ Números de tarjeta de crédito
✗ PII sin enmascarar (GDPR/Habeas Data)
✗ Queries SQL completos con datos de usuarios
✓ IDs, conteos, duraciones, códigos de estado
✓ Primeros/últimos 4 dígitos de tarjeta: "****1234"
✓ Email enmascarado: "u***@ejemplo.com"
```

### Implementación

```typescript
// Usar siempre un logger configurado, nunca console.log
import { logger } from '@shared/logger';

// ✓ Correcto
logger.info('Pedido creado', { pedidoId: pedido.id, clienteId: pedido.clienteId });

// ✗ Incorrecto
console.log('pedido:', JSON.stringify(pedido)); // Puede exponer datos sensibles
```

---

## Métricas

### Métricas RED (Rate, Errors, Duration)

Para cada endpoint y cada operación de negocio, medir:

| Métrica | Tipo | Descripción |
|---------|------|-------------|
| `http_requests_total` | Counter | Total de requests por método, path, status |
| `http_request_duration_seconds` | Histogram | Latencia de cada request |
| `http_requests_in_flight` | Gauge | Requests activos en este momento |

```typescript
// Implementación con prom-client (Node.js)
import { Counter, Histogram, Registry } from 'prom-client';

const requestCounter = new Counter({
  name: 'http_requests_total',
  help: 'Total de HTTP requests',
  labelNames: ['method', 'path', 'status'],
});

const requestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duración de HTTP requests en segundos',
  labelNames: ['method', 'path', 'status'],
  buckets: [0.01, 0.05, 0.1, 0.3, 0.5, 1, 2, 5], // segundos
});
```

### Métricas de negocio (USE method para recursos)

| Métrica de negocio | Tipo | Descripción |
|-------------------|------|-------------|
| `pedidos_creados_total` | Counter | Pedidos creados exitosamente |
| `pedidos_fallidos_total` | Counter | Pedidos que fallaron (por motivo) |
| `valor_pedido_cop` | Histogram | Distribución de valor de pedidos |

### Endpoint de métricas

```
GET /metrics
```

Expone métricas en formato Prometheus (text/plain).
Solo accesible desde la red interna, no expuesto al API Gateway.

---

## Trazas distribuidas

El tracing permite seguir una request a través de múltiples servicios.

### Propagación del contexto

```
Request externa
     │
     ▼
[API Gateway]  genera trace-id: abc123, span-id: 001
     │         Headers: traceparent: 00-abc123-001-01
     │
     ▼
[Auth Service]  crea span hijo: abc123 / 002
     │
     ▼
[Pedido Service]  crea span hijo: abc123 / 003
     │
     ▼
[BD PostgreSQL]  crea span hijo: abc123 / 004 (instrumentación automática)
```

### Implementación con OpenTelemetry

```typescript
// Instrumentación automática (configurar al inicio del servicio)
import { NodeSDK } from '@opentelemetry/sdk-node';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';

const sdk = new NodeSDK({
  traceExporter: new OTLPTraceExporter({
    url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT,
  }),
  instrumentations: [
    // Instrumenta automáticamente: HTTP, Express, PostgreSQL, Redis
    getNodeAutoInstrumentations(),
  ],
});

sdk.start();
```

### Crear spans manuales para lógica de negocio

```typescript
import { trace } from '@opentelemetry/api';

const tracer = trace.getTracer('auth-service');

async function autenticarUsuario(email: string): Promise<Token> {
  return tracer.startActiveSpan('autenticar-usuario', async (span) => {
    try {
      span.setAttributes({ 'user.email_domain': email.split('@')[1] });
      const usuario = await usuarioRepo.buscarPorEmail(email);
      span.setAttributes({ 'user.found': !!usuario });
      // ... lógica ...
      return token;
    } catch (error) {
      span.recordException(error);
      span.setStatus({ code: SpanStatusCode.ERROR });
      throw error;
    } finally {
      span.end();
    }
  });
}
```

---

## Dashboards de Grafana

### Dashboard principal — Vista ejecutiva

Panels requeridos:
1. **Tasa de errores** (%) — últimos 30 min
2. **Latencia P95** (ms) — por servicio
3. **Throughput** (RPS) — por servicio
4. **Disponibilidad** (%) — vs SLO
5. **Instancias por servicio** — para detectar scaling events

### Dashboard por servicio

Cada servicio debe tener su propio dashboard con:
1. RED metrics (Rate, Errors, Duration)
2. Métricas de JVM / Node.js (GC, heap, CPU)
3. Pool de conexiones de BD (activas, en espera, máximo)
4. Métricas de mensajes (publicados / consumidos / DLQ)

---

## Alertas

### Reglas de alerta estándar

| Alerta | Condición | Severidad | Notifica a |
|--------|-----------|-----------|-----------|
| `HighErrorRate` | Error rate > 5% durante 5 min | P1 | PagerDuty + Slack |
| `HighLatency` | P95 > 500ms durante 5 min | P2 | Slack #alerts |
| `ServiceDown` | Health check falla > 1 min | P1 | PagerDuty |
| `DLQNotEmpty` | DLQ tiene > 0 mensajes | P2 | Slack #alerts |
| `LowDiskSpace` | Disco > 80% | P2 | Slack #alerts |
| `PodCrashLooping` | Pod reinicia > 3 veces en 10 min | P1 | PagerDuty |

```yaml
# prometheus-rules.yaml
groups:
  - name: service-alerts
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Alta tasa de errores en {{ $labels.service }}"
          description: "Error rate: {{ $value | humanizePercentage }}"
```

---

## Correlaciones

- Runbook de respuesta a alertas → `09-microservices/services/XX/runbook.md`
- SLOs y Error Budget → `13-operations/README.md`
- Incidents → `13-operations/incident-management.md`
- Cómo agregar métricas de negocio → `09-microservices/services/XX/README.md`
