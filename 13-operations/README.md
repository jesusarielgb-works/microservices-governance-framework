# 13 — Operaciones

> **¿Qué es esto?** Cómo se opera el sistema en producción: cómo se detectan problemas,
> cómo se responde a incidentes y cómo se garantizan los compromisos de disponibilidad.

## Por qué existe esta sección

Un sistema que "funciona" en staging pero es imposible de operar en producción no sirve.
Las preguntas clave de operaciones son:
- ¿Sabemos cuándo algo falla ANTES de que los usuarios llamen?
- ¿Sabemos exactamente qué hacer cuando algo falla a las 3am?
- ¿Tenemos compromisos formales de disponibilidad?

---

## Conceptos clave: SLA, SLO, SLI

**SLI (Service Level Indicator):** métrica que mides — ej: porcentaje de solicitudes exitosas.

**SLO (Service Level Objective):** el objetivo para ese SLI — ej: "99.9% de solicitudes exitosas en 30 días".

**SLA (Service Level Agreement):** el contrato con el cliente sobre esos objetivos, con consecuencias.

**Error Budget:** cuánto puedes "fallar" dentro del SLO. Si el SLO es 99.9%, tienes 43.8 minutos/mes de downtime disponible.

---

## Qué hay aquí y cómo llenarlo

### `observability.md` ⭐
Cómo el sistema hace visible su estado interno.
**Los 3 pilares de observabilidad:**
- **Logs:** qué registra cada servicio, en qué formato (JSON estructurado recomendado), dónde van
- **Métricas:** qué mide cada servicio (latencia, error rate, throughput), con qué herramienta (Prometheus)
- **Trazas:** cómo se rastrea una solicitud que pasa por múltiples servicios (OpenTelemetry, Jaeger)

**Llena:**
```markdown
## Logs
- Formato: JSON estructurado con campos: timestamp, service, level, traceId, message, [datos contextuales]
- Herramienta: [ELK Stack / Loki + Grafana / CloudWatch]
- Retención: [30 días en caliente, 1 año en frío]

## Métricas
- Herramienta: Prometheus + Grafana
- Dashboard principal: [URL]
- Métricas obligatorias por servicio:
  - http_request_duration_seconds (histograma)
  - http_requests_total (contador por status code)
  - [métricas de negocio específicas]

## Trazas distribuidas
- Herramienta: OpenTelemetry + Jaeger
- Cómo propagar el traceId entre servicios: header X-Trace-Id
```

### `incident-management.md` ⭐
Proceso de respuesta a incidentes.
**Llena:** cómo se clasifica la severidad (P0/P1/P2/P3), quién responde, canal de comunicación,
cuándo escalar, proceso de post-mortem.

**Severidades:**
```markdown
| Nivel | Descripción | SLA de respuesta | SLA de resolución |
|-------|-------------|-----------------|-------------------|
| P0 | Sistema caído completamente | 5 min | 1 hora |
| P1 | Funcionalidad crítica degradada | 15 min | 4 horas |
| P2 | Funcionalidad no crítica afectada | 1 hora | 24 horas |
| P3 | Problema menor, workaround disponible | 4 horas | 1 semana |
```

### `backup-and-recovery.md`
Estrategia de respaldo y recuperación de datos.
**Llena:** frecuencia de backups, dónde se almacenan, cómo se prueban, tiempo objetivo de recuperación (RTO/RPO).

---

## Correlaciones con otras secciones

| Esta sección se alimenta de... | Y alimenta a... |
|-------------------------------|-----------------|
| `09-microservices/[servicio]/runbook.md` | Runbooks específicos consolidados aquí |
| `05-architecture/overview.md` | Qué monitorear en cada servicio |
| `10-devops/environments.md` | Configuración de alertas por ambiente |
| Incidentes documentados aquí | `15-project-control/technical-backlog.md` → mejoras post-incidente |

---

## Preguntas que esta sección debe responder

- ¿Cómo sabemos que el sistema falla antes de que los usuarios lo reporten?
- ¿Qué hacemos exactamente cuando falla el servicio X a las 3am?
- ¿Cuánto downtime podemos tener mensualmente?
- ¿Cómo recuperamos los datos si la BD falla?
