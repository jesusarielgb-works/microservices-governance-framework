# Auth Service

> **Autoridad de identidad** del sistema. Gestiona autenticación, emisión de JWT y permisos RBAC.
> Es el dueño autoritativo de las entidades `User` y `Role`.

---

## Ubicación en la arquitectura

| Campo | Valor |
|-------|-------|
| Número en catálogo | 02 |
| Puerto local | 8081 |
| Repositorio | [URL del repo del servicio] |
| Motor de BD | PostgreSQL (usuarios y roles) + Redis (sesiones y token blacklist) |
| Comunica con | — (no llama a otros microservicios) |
| Es consumido por | api-gateway (verificación de tokens), todos los demás servicios |

---

## Responsabilidades (qué SÍ hace este servicio)

- Registrar nuevos usuarios y hashear contraseñas con bcrypt (cost 12)
- Autenticar usuarios y emitir access tokens (JWT, RS256, 1h) y refresh tokens
- Verificar tokens: validar firma, expiración y que no estén en la blacklist
- Gestionar roles y permisos (RBAC): asignar, revocar, consultar
- Invalidar tokens (logout): agregar a blacklist en Redis hasta expiración

## Fuera de su alcance (qué NO hace)

- **No maneja información del perfil de negocio** (nombre de empresa, preferencias de app) — ese dato pertenece al servicio que lo necesita
- **No envía emails** — publica el evento `user.registered` y el notification-service lo escucha
- **No autoriza acceso a recursos específicos** — solo informa los permisos; cada servicio hace la verificación final

---

## Cómo correrlo localmente

```bash
# Levantar con sus dependencias (PostgreSQL + Redis)
docker compose up -d auth-service

# Verificar
curl http://localhost:8081/health

# Crear usuario de prueba
curl -X POST http://localhost:8081/api/v1/auth/register \
  -H 'Content-Type: application/json' \
  -d '{"email": "dev@example.com", "password": "DevPass123!", "name": "Dev User"}'
```

---

## Documentos relacionados

- [data-model.md](./data-model.md) — Tablas `users`, `roles`, `user_roles`
- [events.md](./events.md) — `user.registered`, `user.login_failed`, `user.password_changed`
- [decisions.md](./decisions.md) — Decisiones de diseño del servicio de autenticación
- [runbook.md](./runbook.md) — Operación: rotación de claves, reseteo de contraseña, token blacklist
- [Contrato API](../../../07-api/contracts/openapi/auth-service.yaml)

---

## Estructura del JWT emitido

```json
{
  "sub": "550e8400-e29b-41d4-a716-446655440000",
  "email": "usuario@example.com",
  "roles": ["USER"],
  "permissions": ["appointment:read", "appointment:write"],
  "iat": 1705316400,
  "exp": 1705320000
}
```

**Algoritmo:** RS256 (clave pública disponible en `GET /api/v1/auth/jwks`)

---

## Variables de ambiente requeridas

Ver `.env.example` en la raíz para los nombres de las variables.
Los valores reales están en el vault del ambiente correspondiente.

| Variable | Propósito |
|----------|-----------|
| `APP_AUTH_DATABASE_URL` | Conexión a PostgreSQL |
| `APP_AUTH_REDIS_URL` | Conexión a Redis (token blacklist) |
| `APP_AUTH_JWT_PRIVATE_KEY` | Clave privada RS256 para firmar tokens |
| `APP_AUTH_JWT_PUBLIC_KEY` | Clave pública RS256 para verificar tokens |
| `APP_AUTH_BCRYPT_ROUNDS` | Cost factor de bcrypt (default: 12) |
