# Eventos — API Gateway

> El API Gateway **no emite eventos de dominio**. Su responsabilidad es de infraestructura
> (enrutamiento, autenticación, rate limiting), no de negocio.
>
> Los eventos de dominio (cosas que le importan al negocio) los emiten los servicios detrás del gateway.

---

## Eventos publicados

**Ninguno.** El API Gateway no publica al message broker.

---

## Eventos consumidos

**Ninguno.** El API Gateway no suscribe al message broker.

---

## Logs de acceso (no son eventos de dominio)

El gateway sí genera **logs de acceso** estructurados por cada petición HTTP:

```json
{
  "timestamp": "2024-01-15T10:30:00.123Z",
  "level": "info",
  "type": "access_log",
  "method": "POST",
  "path": "/api/v1/auth/login",
  "status": 200,
  "durationMs": 45,
  "ip": "192.168.1.100",
  "correlationId": "550e8400-e29b-41d4-a716-446655440000",
  "upstreamService": "auth-service",
  "upstreamStatus": 200
}
```

Estos logs van a la plataforma de observabilidad (`13-operations/observability.md`) pero
**no se publican al broker de mensajes** — no son eventos de negocio.

---

## Correlaciones

- Observabilidad y logs → `13-operations/observability.md`
- Eventos de negocio de los servicios → `09-microservices/services/02-auth-service/events.md`
