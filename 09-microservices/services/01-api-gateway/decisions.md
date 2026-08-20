# Decisiones técnicas — API Gateway

> Decisiones de diseño internas de este servicio que no están cubiertas por los ADRs
> de arquitectura global. Cada decisión documenta el contexto, la elección y las consecuencias.

---

## Decisión: Autenticación en gateway vs. en cada servicio

**Fecha:** [fecha de la decisión]
**Contexto:** Hay dos estrategias para verificar tokens JWT: (A) el gateway verifica el token
y pasa el contexto del usuario como headers; (B) cada microservicio verifica su propio token.

**Decisión:** Estrategia A — el gateway verifica el JWT (firma + expiración + blacklist)
y adjunta `X-User-Id` y `X-User-Role` como headers internos. Los microservicios confían
en estos headers sin re-verificar el token.

**Consecuencias:**
- Los microservicios son más simples (no necesitan librería JWT)
- El gateway es el único punto que necesita acceso a la clave pública JWT
- Si el gateway es comprometido, todos los servicios son vulnerables — mitigado con TLS interno
- Los headers internos NO deben estar disponibles desde fuera del gateway (nginx/proxy stripea los headers X-User-* entrantes)

---

## Decisión: Rate limiting por IP vs. por usuario

**Fecha:** [fecha de la decisión]
**Contexto:** El rate limiting se puede aplicar a la IP de origen o al usuario autenticado.

**Decisión:** Rate limiting por IP para endpoints públicos (login, register), por user-id para
endpoints autenticados.

**Consecuencias:**
- El rate limit por IP no penaliza a usuarios de la misma red (NAT corporativa)
- El rate limit por user-id requiere que el token esté verificado antes de aplicar el límite
- Implementación: bucket en Redis con key `rate:{ip}:{path}` o `rate:{userId}:{path}`

---

## Decisión: Implementación del gateway

**Fecha:** [fecha de la decisión]
**Contexto:** Opciones: (A) Kong/NGINX Plus (solución gestionada), (B) Express/Fastify custom,
(C) Envoy como sidecar.

**Decisión:** [A / B / C] — [razón]

**Consecuencias:** [trade-offs de la decisión]

---

## Correlaciones

- ADRs de arquitectura global → `05-architecture/decisions/records/`
- Runbook del gateway → `runbook.md`
