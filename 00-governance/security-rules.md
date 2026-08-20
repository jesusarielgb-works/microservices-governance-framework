# Reglas Técnicas de Seguridad

> Controles técnicos obligatorios que aplican a todo el código del proyecto.
> Estas reglas complementan la política de seguridad (`security-policy.md`) con
> prácticas concretas de implementación.

---

## OWASP Top 10 — Controles por categoría

### A01 — Control de Acceso Roto (Broken Access Control)

```typescript
// ❌ MAL — confiar en datos del frontend
const userId = req.body.userId;

// ✅ BIEN — extraer del token JWT verificado
const userId = req.user.sub; // req.user viene del middleware de autenticación
```

**Reglas:**
- Todo endpoint protegido DEBE tener el middleware de autenticación aplicado
- Los permisos se verifican en el Use Case, no en el Controller
- Un recurso solo se devuelve si el usuario tiene permiso de `[recurso]:read`
- Las acciones de escritura requieren permiso de `[recurso]:write` o `[recurso]:delete`

### A02 — Fallas Criptográficas

**Reglas:**
- Contraseñas: usar **bcrypt** con cost factor ≥ 12. Nunca MD5 o SHA-1 para contraseñas
- JWTs: firmar con RS256 (asimétrico). Nunca HS256 en producción con secreto débil
- Datos sensibles en tránsito: HTTPS obligatorio en todos los ambientes excepto local
- Datos sensibles en reposo: cifrar con AES-256-GCM los campos marcados como PII
- Nunca loguear contraseñas, tokens ni datos de tarjetas de crédito

### A03 — Inyección (Injection)

**SQL:**
```typescript
// ❌ MAL — concatenación directa
const user = await db.query(`SELECT * FROM users WHERE id = ${userId}`);

// ✅ BIEN — query parametrizada
const user = await db.query('SELECT * FROM users WHERE id = $1', [userId]);

// ✅ BIEN — ORM con parámetros tipados
const user = await userRepository.findOne({ where: { id: userId } });
```

**Reglas:**
- Queries parametrizadas SIEMPRE. Cero strings concatenados en SQL
- Validar y sanitizar todos los inputs con una librería de validación (Zod, Joi, class-validator)
- En GraphQL: limitar profundidad de queries con `graphql-depth-limit`

### A04 — Diseño Inseguro

- Toda HU que exponga datos del usuario debe pasar por revisión de privacidad
- Los endpoints de consulta masiva tienen paginación obligatoria (máximo [100] registros por página)
- No exponer IDs internos secuenciales; usar UUIDs

### A05 — Configuración de Seguridad Incorrecta

```
# Lista de verificación por ambiente
□ Stack traces NO visibles en producción
□ Headers de seguridad configurados (Helmet.js o equivalente):
  - X-Content-Type-Options: nosniff
  - X-Frame-Options: DENY
  - Content-Security-Policy definida
  - Strict-Transport-Security en producción
□ Puertos innecesarios cerrados
□ Credenciales de desarrollo NO en producción
```

### A06 — Componentes Vulnerables

**Reglas:**
- Ejecutar `npm audit` (o equivalente) antes de cada release
- Vulnerabilidades **Critical/High** bloquean el deploy
- Renovar dependencias en cada sprint (al menos una vez)
- No usar versiones `latest` sin fijar en `package.json`; usar versiones exactas o rangos conservadores

### A07 — Fallas de Identificación y Autenticación

- JWT con expiración máxima de **1 hora** para access tokens
- Refresh tokens con expiración de **[7 días / 30 días]** y rotación en cada uso
- Rate limiting en `/auth/login`: máximo [10] intentos por IP en 5 minutos
- Bloqueo de cuenta después de [5] intentos fallidos consecutivos

### A08 — Fallas de Integridad de Software y Datos

- Verificar checksum de imágenes Docker antes de usar en producción
- Los webhooks de terceros deben verificar firma criptográfica
- Validar que los mensajes del broker (Kafka/RabbitMQ) tienen el schema esperado

### A09 — Fallas de Registro y Monitoreo

- Toda autenticación fallida debe loguearse con IP, timestamp y user-agent
- Loguear operaciones de borrado con quién, cuándo y qué se borró
- Los logs de seguridad se retienen mínimo **90 días**
- Alertas automáticas configuradas para:
  - Más de [50] errores 401/403 en 5 minutos
  - Acceso a un recurso desde un país inesperado (si aplica)

### A10 — Server-Side Request Forgery (SSRF)

- Las URLs que se construyen con input del usuario DEBEN validarse contra una lista de dominios permitidos
- No hacer fetch a IPs privadas (192.168.x.x, 10.x.x.x, 127.x.x.x) desde el servidor

---

## Manejo de inputs del usuario

```typescript
// Ejemplo con Zod — siempre validar en la capa de Controller/Adapter
const CreateUserSchema = z.object({
  email: z.string().email().max(255),
  name: z.string().min(1).max(100).trim(),
  role: z.enum(['ADMIN', 'USER', 'VIEWER']),
});

// El resultado es tipado y sanitizado
const parsed = CreateUserSchema.parse(req.body);
```

**Regla:** Todos los inputs externos (HTTP body, query params, path params, mensajes del broker)
pasan por un schema de validación antes de llegar al dominio.

---

## Manejo de errores seguro

```typescript
// ❌ MAL — expone detalles internos
res.status(500).json({ error: error.message, stack: error.stack });

// ✅ BIEN — mensaje genérico + traceId para correlación interna
res.status(500).json({
  error: 'INTERNAL_SERVER_ERROR',
  message: 'Error interno del servidor',
  traceId: req.headers['x-trace-id'],
});
```

---

## Correlaciones

- Política de seguridad (gestión, acceso, vault) → `00-governance/security-policy.md`
- Threat model del sistema → `05-architecture/security-threat-model.md`
- Autenticación y JWT → `07-api/authentication.md`
- Observabilidad y logs de seguridad → `13-operations/observability.md`
