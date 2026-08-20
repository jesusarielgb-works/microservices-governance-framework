# Política de Seguridad

> La seguridad no es una feature — es una propiedad del sistema que se construye desde
> el primer día. Este documento define las prácticas obligatorias.
> Cualquier desviación debe ser aprobada explícitamente por el Tech Lead.

---

## Principios de seguridad

1. **Defense in Depth:** Múltiples capas de seguridad. Si una falla, las otras contienen el daño.
2. **Least Privilege:** Cada componente tiene solo los permisos mínimos necesarios.
3. **Fail Secure:** En caso de error, el sistema deniega el acceso, no lo permite.
4. **Security by Design:** Los controles de seguridad se diseñan desde el inicio, no se agregan al final.
5. **Zero Trust:** Verificar siempre, nunca confiar implícitamente, incluso dentro de la red interna.

---

## Autenticación

### JWT (JSON Web Tokens)

| Propiedad | Valor obligatorio |
|-----------|------------------|
| Algoritmo de firma | RS256 (asimétrico) o HS256 con secret de 256+ bits |
| Expiración del access token | 1 hora (`exp`) |
| Expiración del refresh token | 7 días |
| Claims obligatorios | `sub` (userId), `iat`, `exp`, `jti` (unique ID del token) |
| Almacenamiento en cliente | `httpOnly cookie` (web) o Keychain/Keystore (móvil) |

**Claims prohibidos en el payload:**
- Contraseñas
- Datos de tarjetas
- PII completo (solo el ID del usuario)

### Refresh Token

- Almacenado en la base de datos (con hash bcrypt)
- Rotación obligatoria en cada uso (un refresh token = un uso)
- Invalidación en logout y en cambio de contraseña
- Invalidación de TODOS los tokens activos si se detecta uso de token revocado

---

## Autorización

### RBAC (Role-Based Access Control)

| Rol | Descripción | Permisos |
|-----|-------------|---------|
| `SUPER_ADMIN` | Administrador técnico del sistema | Todos |
| `ADMIN` | Administrador de negocio | [definir] |
| `OPERATOR` | Operador con permisos de escritura | [definir] |
| `VIEWER` | Solo lectura | [definir] |
| `[ROL_PERSONALIZADO]` | [descripción] | [definir] |

**Modelo de permisos:**

```
Permiso: [recurso]:[acción]

Ejemplos:
  pedidos:create
  pedidos:read
  pedidos:update
  pedidos:delete
  usuarios:read
  reportes:export
```

**Validación:**
- El API Gateway valida el JWT (firma y expiración)
- Cada servicio valida los permisos del rol para la operación específica
- Los roles se incluyen en el JWT como claim `roles: ["OPERATOR", "VIEWER"]`

---

## Comunicación segura

### Transmisión

- **HTTPS obligatorio** en todos los ambientes excepto local
- TLS 1.2 mínimo; TLS 1.3 recomendado
- Certificados: Let's Encrypt (staging) / CA corporativa (producción)
- HSTS activado en producción

### Comunicación interna entre servicios

- mTLS para comunicación servicio-a-servicio en producción (si es posible con service mesh)
- Bearer token o API key interna para servicios que no soporten mTLS

---

## Manejo de secretos

```
✗ NUNCA en el código fuente
✗ NUNCA en .env commiteado
✗ NUNCA en logs
✗ NUNCA en mensajes de error al cliente
✓ Variables de ambiente (inyectadas por el orchestrador)
✓ Vault (HashiCorp Vault, AWS Secrets Manager, etc.)
✓ Kubernetes Secrets (encriptados con KMS)
```

**Rotación de secretos:**
- API keys: cada 90 días
- Certificados TLS: 60 días antes del vencimiento
- Passwords de BD: cada 6 meses o inmediatamente si hay sospecha de compromiso

---

## Validación y sanitización de entradas

### Reglas generales

1. **Nunca confíes en el input del usuario.** Valida en el borde (controller) antes de procesar.
2. **Whitelist, no blacklist.** Define lo que se permite, no solo lo que se prohíbe.
3. **Rechaza temprano.** Si el input es inválido, responde 400 y no proceses más.

### SQL Injection — Prevención

```typescript
// ✗ VULNERABLE
const result = await db.query(`SELECT * FROM users WHERE email = '${userInput}'`);

// ✓ SEGURO — Usar parámetros preparados siempre
const result = await db.query('SELECT * FROM users WHERE email = $1', [userInput]);
```

### XSS — Prevención

```typescript
// ✗ VULNERABLE — renderizar HTML sin escapar
element.innerHTML = userProvidedContent;

// ✓ SEGURO — usar textContent o sanitizar
element.textContent = userProvidedContent;
// o con librería: DOMPurify.sanitize(userProvidedContent)
```

### Validación con Zod / Joi

```typescript
// Schema de validación explícito en el controller
const CrearPedidoSchema = z.object({
  clienteId: z.string().uuid(),
  items: z.array(z.object({
    productoId: z.string().uuid(),
    cantidad: z.number().int().positive().max(1000),
    precio: z.object({
      amount: z.number().positive(),
      currency: z.enum(['COP', 'USD']),
    }),
  })).min(1).max(50),
});
```

---

## OWASP Top 10 — Checklist de revisión

| Vulnerabilidad | Control implementado |
|----------------|---------------------|
| A01: Broken Access Control | RBAC + validación de permisos en cada servicio |
| A02: Cryptographic Failures | TLS 1.2+, bcrypt para passwords, secrets en vault |
| A03: Injection | Parámetros preparados en SQL, validación de schema |
| A04: Insecure Design | Threat modeling en el diseño, Security review |
| A05: Security Misconfiguration | IaC para configuración, revisión de defaults |
| A06: Vulnerable Components | Dependabot / Snyk para actualizaciones automáticas |
| A07: Authentication Failures | JWT con rotación, brute-force protection |
| A08: Software Integrity Failures | Verificar checksums de dependencias, SBOM |
| A09: Logging Failures | Logs sin PII, centralizados, con alertas |
| A10: SSRF | Whitelist de URLs externas, no seguir redirects automáticamente |

---

## Auditoría y logs de seguridad

### Eventos que SIEMPRE se registran

```typescript
// Eventos de seguridad — guardar en log separado, con retención > 1 año
const SECURITY_EVENTS = [
  'auth.login.success',
  'auth.login.failure',
  'auth.login.brute_force_detected',
  'auth.password.changed',
  'auth.token.revoked',
  'auth.unauthorized_access_attempt',
  'data.pii.accessed',
  'admin.role.changed',
  'admin.user.deleted',
];
```

**Campos obligatorios en logs de seguridad:**
- `userId` (o `ANONYMOUS` si no autenticado)
- `sourceIp`
- `action`
- `resource`
- `result` (SUCCESS / FAILURE)
- `timestamp`

---

## Proceso de vulnerabilidades

### Qué hacer si encuentras una vulnerabilidad

1. **No la comitas al repo público** ni la discutas en canales abiertos
2. Notifica inmediatamente al Tech Lead por canal privado
3. Crea un issue privado o en repositorio restringido
4. Se asigna severidad (CVSS score o clasificación interna)
5. Se remedia en el sprint actual si es crítica, en el siguiente si es alta

### SLAs de remediación

| Severidad | Tiempo de remediación |
|-----------|----------------------|
| Crítica (CVSS 9-10) | 24 horas |
| Alta (CVSS 7-8.9) | 1 semana |
| Media (CVSS 4-6.9) | 1 mes |
| Baja (CVSS < 4) | Próxima revisión de seguridad |

---

## Correlaciones

- Requisitos no funcionales de seguridad → `04-requirements/non-functional.md`
- ADR sobre autenticación → `05-architecture/decisions/`
- Observabilidad de eventos de seguridad → `13-operations/observability.md`
- RBAC implementado en → `09-microservices/services/XX-auth-service/`
