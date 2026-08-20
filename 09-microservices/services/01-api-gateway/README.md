# API Gateway

> **Punto único de entrada** al sistema. Recibe todas las peticiones del frontend y las enruta
> al microservicio correspondiente. Es el dueño autoritativo de las reglas de enrutamiento.

---

## Ubicación en la arquitectura

| Campo | Valor |
|-------|-------|
| Número en catálogo | 01 |
| Puerto local | 8080 |
| Repositorio | [URL del repo del servicio] |
| Motor de BD | — (stateless, sin BD propia) |
| Comunica con | auth-service, [otros servicios] |
| Es consumido por | Frontend web, app móvil, herramientas de terceros |

---

## Responsabilidades (qué SÍ hace este servicio)

- Enrutar peticiones HTTP al microservicio correcto según el path (`/api/v1/auth/*`, `/api/v1/[recurso]/*`)
- Verificar que el token JWT sea válido antes de reenviar la petición (delegando al auth-service)
- Adjuntar el contexto del usuario (`X-User-Id`, `X-User-Role`) como headers internos
- Rate limiting global: máximo [100] peticiones por IP por minuto
- Logs de acceso unificados con correlationId

## Fuera de su alcance (qué NO hace)

- **No maneja lógica de negocio** — solo enruta
- **No verifica permisos de recurso** — solo verifica que el JWT sea válido (autorización la hace cada servicio)
- **No almacena datos** — stateless

---

## Cómo correrlo localmente

```bash
# Desde la raíz del proyecto (levanta todos los servicios)
docker compose up -d api-gateway

# Verificar que está funcionando
curl http://localhost:8080/health
```

**Respuesta esperada:**
```json
{ "status": "ok", "timestamp": "2024-01-15T10:30:00Z" }
```

---

## Documentos relacionados

- [data-model.md](./data-model.md) — No aplica (sin BD propia)
- [events.md](./events.md) — No aplica (no emite eventos de dominio)
- [decisions.md](./decisions.md) — Decisiones de diseño del gateway
- [runbook.md](./runbook.md) — Operación en producción
- [Contrato API](../../../07-api/contracts/openapi/api-gateway.yaml)

---

## Decisiones de diseño

Ver: `09-microservices/services/01-api-gateway/decisions.md`

Las decisiones relevantes incluyen:
- **ADR-002** — Por qué se eligió [Kong / Nginx / custom Express] como base del gateway
- **ADR-003** — Estrategia de autenticación en el gateway vs. en cada servicio

---

## Patrón de enrutamiento

```
Cliente → :8080/api/v1/auth/*        → auth-service :8081
Cliente → :8080/api/v1/[recurso]/*  → [nombre]-service :808N
```

Configuración del enrutamiento: `[ruta al archivo de configuración del gateway]`
