# Decisiones técnicas — Auth Service

> Decisiones de diseño del servicio de autenticación que complementan los ADRs globales.

---

## Decisión: Algoritmo de firma JWT — RS256 vs HS256

**Fecha:** [fecha]
**Contexto:** JWT puede firmarse con HMAC-SHA256 (HS256, clave simétrica compartida) o
RSA-SHA256 (RS256, par de claves pública/privada).

**Decisión:** RS256 — clave privada solo en auth-service, clave pública disponible en
`GET /api/v1/auth/jwks` para que cualquier servicio pueda verificar tokens sin llamar al auth-service.

**Consecuencias:**
- Cada servicio puede verificar tokens de forma independiente (sin latencia de red)
- La rotación de claves es operativamente más compleja (hay que distribuir la nueva clave pública)
- El JWKS endpoint permite rotación gradual con múltiples claves activas simultáneamente

---

## Decisión: Refresh tokens en base de datos vs. stateless

**Fecha:** [fecha]
**Contexto:** Los refresh tokens pueden ser stateless (JWT largo) o stateful (ID en BD).

**Decisión:** Stateful — el refresh token es un UUID almacenado en la tabla `refresh_tokens`.
Solo el hash SHA-256 se persiste (nunca el valor real).

**Consecuencias:**
- Revocación inmediata posible (logout de dispositivo específico, suspensión de cuenta)
- Requiere un lookup a PostgreSQL en cada uso del refresh token
- La BD de refresh_tokens puede convertirse en bottleneck si hay millones de sesiones activas

---

## Decisión: Dónde vive la verificación de permisos por recurso

**Fecha:** [fecha]
**Contexto:** Los permisos granulares (ej: "solo puede ver sus propias citas") pueden vivir
en auth-service o en cada servicio de negocio.

**Decisión:** Auth-service gestiona roles (ADMIN, USER, VIEWER). Los permisos granulares
por recurso son responsabilidad de cada microservicio de negocio.

**Razón:** Auth-service no conoce el dominio de cada servicio. Centralizar permisos granulares
crearía un acoplamiento bidireccional y haría al auth-service dependiente de todos los demás.

---

## Decisión: Política de bloqueo de cuentas

**Fecha:** [fecha]
**Decisión:** [N] intentos fallidos → bloqueo por [M] minutos. El contador se resetea
en login exitoso. El bloqueo se almacena en Redis (TTL = M minutos, autolimpieza).

**Consecuencias:**
- Un atacante puede hacer DoS a un usuario específico forzando el bloqueo
- Mitigación: el bloqueo gradual (5 min → 30 min → 24 hours) reduce el impacto

---

## Correlaciones

- ADRs de arquitectura global → `05-architecture/decisions/records/`
- Reglas de seguridad técnica → `00-governance/security-rules.md`
- Modelo de datos → `data-model.md`
