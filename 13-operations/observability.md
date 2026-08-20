# System Observability

> An unobservable system cannot be operated. Observability is not an optional feature
> added later — it is designed from the first sprint.
> The 3 pillars: Logs, Metrics, Traces.

> **Stack note:** The concepts (structured logging, RED metrics, OpenTelemetry) apply
> to any language. Code examples use Node.js ecosystem libraries
> (pino, prom-client, @opentelemetry/sdk-node). For other technologies, the equivalents are:
> - Java: Logback/SLF4J + Micrometer + OpenTelemetry Java Agent
> - Python: structlog + prometheus_client + opentelemetry-sdk
> - Go: zap + prometheus/client_golang + go.opentelemetry.io/otel

---

## The 3 pillars of observability

```
                    ┌───────────────────────────────────────────┐
                    │           Production system               │
                    │                                           │
                    │  [Service A]   [Service B]  [Service C]  │
                    └──────┬──────────────┬───────────┬─────────┘
                           │              │           │
               ┌───────────┼──────────────┼───────────┼──────────┐
               ▼           ▼              ▼           ▼          │
           [LOGS]      [METRICS]      [TRACES]   [EVENTS]       │
               │           │              │                       │
               ▼           ▼              ▼                       │
       [Elasticsearch] [Prometheus]  [Jaeger/Zipkin]             │
               │           │              │                       │
               ▼           ▼              │                       │
           [Kibana]    [Grafana]          │                       │
                           │              │                       │
                           └──────────────┘                      │
                                  │                              │
                           [ALERTS] ──────────────────────────▶│
                        [Alertmanager]                          │
                        [PagerDuty]                             │
                        [Slack]                                  │
                    └──────────────────────────────────────────-┘
```

---

## Logs

### Standard format (structured JSON)

All services must produce JSON logs with these minimum fields:

```json
{
  "timestamp": "2024-01-15T10:30:00.123Z",
  "level": "INFO",
  "service": "auth-service",
  "version": "1.3.0",
  "environment": "production",
  "correlationId": "550e8400-e29b-41d4-a716-446655440000",
  "userId": "user-uuid-if-available",
  "message": "User authenticated successfully",
  "context": {
    "userId": "uuid",
    "method": "POST",
    "path": "/auth/login",
    "statusCode": 200,
    "durationMs": 45
  }
}
```

**Required fields:**

| Field | Description |
|-------|-------------|
| `timestamp` | ISO 8601 with milliseconds and UTC timezone |
| `level` | `DEBUG`, `INFO`, `WARN`, `ERROR`, `FATAL` |
| `service` | Microservice name |
| `correlationId` | For tracing the transaction across services |
| `message` | Descriptive message, without sensitive data |

### Log levels

| Level | When to use it | Example |
|-------|---------------|---------|
| `DEBUG` | Development info. Disabled in production | "SQL query executed: SELECT..." |
| `INFO` | Normal business events | "Order #123 created" |
| `WARN` | Abnormal but not failure | "Token about to expire", "Retry #2" |
| `ERROR` | Error that requires attention | "Failed to connect to PostgreSQL" |
| `FATAL` | Error that brings down the service | "Cannot start: port in use" |

### What NOT to log

```
✗ Passwords or full tokens
✗ Credit card numbers
✗ Unmasked PII (GDPR/Data Protection)
✗ Full SQL queries with user data
✓ IDs, counts, durations, status codes
✓ First/last 4 digits of card: "****1234"
✓ Masked email: "u***@example.com"
```

### Implementation

```typescript
// Always use a configured logger, never console.log
import { logger } from '@shared/logger';

// ✓ Correct
logger.info('Order created', { orderId: order.id, customerId: order.customerId });

// ✗ Incorrect
console.log('order:', JSON.stringify(order)); // May expose sensitive data
```

---

## Metrics

### RED Metrics (Rate, Errors, Duration)

For each endpoint and each business operation, measure:

| Metric | Type | Description |
|--------|------|-------------|
| `http_requests_total` | Counter | Total requests by method, path, status |
| `http_request_duration_seconds` | Histogram | Latency of each request |
| `http_requests_in_flight` | Gauge | Active requests at this moment |

```typescript
// Implementation with prom-client (Node.js)
import { Counter, Histogram, Registry } from 'prom-client';

const requestCounter = new Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'path', 'status'],
});

const requestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration in seconds',
  labelNames: ['method', 'path', 'status'],
  buckets: [0.01, 0.05, 0.1, 0.3, 0.5, 1, 2, 5], // seconds
});
```

### Business metrics (USE method for resources)

| Business metric | Type | Description |
|----------------|------|-------------|
| `orders_created_total` | Counter | Orders successfully created |
| `orders_failed_total` | Counter | Orders that failed (by reason) |
| `order_value_usd` | Histogram | Distribution of order values |

### Metrics endpoint

```
GET /metrics
```

Exposes metrics in Prometheus format (text/plain).
Only accessible from the internal network, not exposed to the API Gateway.

---

## Distributed traces

Tracing allows following a request across multiple services.

### Context propagation

```
External request
     │
     ▼
[API Gateway]  generates trace-id: abc123, span-id: 001
     │         Headers: traceparent: 00-abc123-001-01
     │
     ▼
[Auth Service]  creates child span: abc123 / 002
     │
     ▼
[Order Service]  creates child span: abc123 / 003
     │
     ▼
[PostgreSQL DB]  creates child span: abc123 / 004 (automatic instrumentation)
```

### Implementation with OpenTelemetry

```typescript
// Automatic instrumentation (configure at service startup)
import { NodeSDK } from '@opentelemetry/sdk-node';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';

const sdk = new NodeSDK({
  traceExporter: new OTLPTraceExporter({
    url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT,
  }),
  instrumentations: [
    // Automatically instruments: HTTP, Express, PostgreSQL, Redis
    getNodeAutoInstrumentations(),
  ],
});

sdk.start();
```

### Create manual spans for business logic

```typescript
import { trace } from '@opentelemetry/api';

const tracer = trace.getTracer('auth-service');

async function authenticateUser(email: string): Promise<Token> {
  return tracer.startActiveSpan('authenticate-user', async (span) => {
    try {
      span.setAttributes({ 'user.email_domain': email.split('@')[1] });
      const user = await userRepo.findByEmail(email);
      span.setAttributes({ 'user.found': !!user });
      // ... logic ...
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

## Grafana Dashboards

### Main dashboard — Executive view

Required panels:
1. **Error rate** (%) — last 30 min
2. **P95 latency** (ms) — per service
3. **Throughput** (RPS) — per service
4. **Availability** (%) — vs SLO
5. **Instances per service** — to detect scaling events

### Per-service dashboard

Each service must have its own dashboard with:
1. RED metrics (Rate, Errors, Duration)
2. JVM / Node.js metrics (GC, heap, CPU)
3. DB connection pool (active, waiting, maximum)
4. Message metrics (published / consumed / DLQ)

---

## Alerts

### Standard alert rules

| Alert | Condition | Severity | Notifies |
|-------|-----------|----------|---------|
| `HighErrorRate` | Error rate > 5% for 5 min | P1 | PagerDuty + Slack |
| `HighLatency` | P95 > 500ms for 5 min | P2 | Slack #alerts |
| `ServiceDown` | Health check fails > 1 min | P1 | PagerDuty |
| `DLQNotEmpty` | DLQ has > 0 messages | P2 | Slack #alerts |
| `LowDiskSpace` | Disk > 80% | P2 | Slack #alerts |
| `PodCrashLooping` | Pod restarts > 3 times in 10 min | P1 | PagerDuty |

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
          summary: "High error rate in {{ $labels.service }}"
          description: "Error rate: {{ $value | humanizePercentage }}"
```

---

## Correlations

- Alert response runbook → `09-microservices/services/XX/runbook.md`
- SLOs and Error Budget → `13-operations/README.md`
- Incidents → `13-operations/incident-management.md`
- How to add business metrics → `09-microservices/services/XX/README.md`
