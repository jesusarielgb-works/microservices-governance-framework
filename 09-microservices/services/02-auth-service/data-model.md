# Modelo de Datos — Auth Service

> El Auth Service es el **dueño autoritativo** de la identidad del usuario.
> Ningún otro servicio lee directamente estas tablas. Si necesitan datos del usuario,
> los obtienen del token JWT o consultan vía API.

---

## Motor de base de datos

**Motor principal:** PostgreSQL — para usuarios, roles y tokens de actualización.
**Motor de caché:** Redis — para la blacklist de tokens y contadores de intentos fallidos.

**Justificación PostgreSQL:** Transacciones ACID son críticas: no puede existir un usuario
sin rol asignado, y no pueden perderse registros de intentos de login fallidos.

**Justificación Redis:** La verificación del token blacklist ocurre en cada request al gateway.
Redis soporta búsquedas O(1) con TTL automático, sin necesidad de jobs de limpieza.

---

## Esquema PostgreSQL

### Tabla: `users`

**Propósito:** Credenciales de autenticación. Solo los datos necesarios para autenticar — perfil de aplicación va en el servicio de negocio correspondiente.

| Campo | Tipo | Nullable | Descripción | Restricciones |
|-------|------|----------|-------------|---------------|
| id | UUID | No | Identificador único | PK, gen_random_uuid() |
| email | VARCHAR(255) | No | Email del usuario | UNIQUE, NOT NULL |
| password_hash | VARCHAR(255) | No | Hash bcrypt (cost 12) | NOT NULL |
| email_verified | BOOLEAN | No | Si el email fue verificado | DEFAULT false |
| failed_attempts | INT | No | Intentos de login fallidos | DEFAULT 0 |
| locked_until | TIMESTAMPTZ | Sí | Bloqueado hasta esta fecha | NULL = no bloqueado |
| created_at | TIMESTAMPTZ | No | Fecha de registro | DEFAULT NOW() |
| updated_at | TIMESTAMPTZ | No | Última modificación | DEFAULT NOW() |
| deleted_at | TIMESTAMPTZ | Sí | Soft delete | NULL = activo |

**Índices:**
| Nombre | Campos | Justificación |
|--------|--------|---------------|
| idx_users_email | email | Búsqueda por email en login (principal lookup) |
| idx_users_deleted_at | deleted_at | Filtrar usuarios activos |

### Tabla: `roles`

**Propósito:** Catálogo de roles del sistema.

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| id | UUID | No | PK |
| name | VARCHAR(50) | No | Nombre del rol (UNIQUE): ADMIN, USER, VIEWER |
| description | TEXT | Sí | Descripción legible del rol |
| created_at | TIMESTAMPTZ | No | DEFAULT NOW() |

### Tabla: `user_roles`

**Propósito:** Relación many-to-many entre usuarios y roles.

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| user_id | UUID | No | FK → users.id ON DELETE CASCADE |
| role_id | UUID | No | FK → roles.id ON DELETE RESTRICT |
| assigned_at | TIMESTAMPTZ | No | Cuándo se asignó el rol |
| assigned_by | UUID | Sí | FK → users.id — quién asignó el rol |

**PK compuesta:** (user_id, role_id)

### Tabla: `refresh_tokens`

**Propósito:** Tokens de actualización activos. Permite rotación y revocación.

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| id | UUID | No | PK |
| user_id | UUID | No | FK → users.id ON DELETE CASCADE |
| token_hash | VARCHAR(255) | No | Hash SHA-256 del token (nunca el token real) |
| expires_at | TIMESTAMPTZ | No | Expiración del refresh token |
| revoked_at | TIMESTAMPTZ | Sí | NULL = activo, fecha = revocado |
| created_at | TIMESTAMPTZ | No | DEFAULT NOW() |
| user_agent | TEXT | Sí | Para mostrar sesiones activas al usuario |

---

## Esquema Redis

| Key pattern | Tipo | TTL | Propósito |
|-------------|------|-----|-----------|
| `blacklist:{jti}` | String | Hasta expiración del JWT | Tokens revocados (logout) |
| `attempts:{email}` | String (int) | 5 minutos | Contador de intentos fallidos |
| `locked:{email}` | String | Hasta unlock | Email bloqueado temporalmente |

---

## Migraciones

**Herramienta:** [Flyway / Alembic / golang-migrate — según el stack del proyecto]
**Ubicación de scripts:** `src/migrations/` (ver guía en `_stacks/[tu-stack].md`)

**Política:** Todas las migraciones son forward-only en producción. Los rollbacks de datos se hacen con migraciones adicionales, no revirtiendo scripts.

---

## Correlaciones

- Runbook del servicio → `runbook.md`
- Eventos que emite → `events.md`
- Política de seguridad y JWT → `00-governance/security-policy.md`
