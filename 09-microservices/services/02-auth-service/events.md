# Eventos — Auth Service

> El Auth Service publica eventos de dominio cuando ocurren cambios significativos
> en la identidad del usuario. Los eventos son la forma en que otros servicios
> reaccionan sin consultar directamente a auth-service.

---

## Eventos publicados

### `user.registered`

**Cuándo se emite:** Al completarse el registro exitoso de un nuevo usuario.
**Tópico / Exchange:** `auth.events` (o `user.registered` según el broker del proyecto)
**Consumidores típicos:** notification-service (enviar email de bienvenida), analytics-service

```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440001",
  "eventType": "user.registered",
  "aggregateId": "550e8400-e29b-41d4-a716-446655440000",
  "occurredAt": "2024-01-15T10:30:00.000Z",
  "version": 1,
  "payload": {
    "userId": "550e8400-e29b-41d4-a716-446655440000",
    "email": "usuario@example.com",
    "roles": ["USER"]
  },
  "metadata": {
    "correlationId": "req-abc-123",
    "causationId": "cmd-register-456"
  }
}
```

---

### `user.login_failed`

**Cuándo se emite:** Cuando un intento de login falla por credenciales inválidas.
**Tópico / Exchange:** `auth.events`
**Consumidores típicos:** security-monitoring-service (detección de ataques de fuerza bruta)

```json
{
  "eventId": "...",
  "eventType": "user.login_failed",
  "aggregateId": "email-hash-del-usuario",
  "occurredAt": "2024-01-15T10:31:00.000Z",
  "version": 1,
  "payload": {
    "email": "usuario@example.com",
    "failedAttempts": 3,
    "ipAddress": "192.168.1.100",
    "userAgent": "Mozilla/5.0 ..."
  }
}
```

**Nota de privacidad:** El email se incluye para correlacionar intentos, pero el servicio
consumidor no debe loguearlo en texto plano. Considerar hashear con HMAC antes de publicar.

---

### `user.password_changed`

**Cuándo se emite:** Al cambiarse exitosamente la contraseña.
**Tópico / Exchange:** `auth.events`
**Consumidores típicos:** notification-service (alertar al usuario del cambio)

```json
{
  "eventType": "user.password_changed",
  "aggregateId": "[userId]",
  "payload": {
    "userId": "[userId]",
    "changedAt": "2024-01-15T10:35:00.000Z"
  }
}
```

---

### `user.account_locked`

**Cuándo se emite:** Al bloquearse una cuenta por demasiados intentos fallidos.
**Tópico / Exchange:** `auth.events`
**Consumidores típicos:** notification-service (alertar al usuario), security-monitoring

---

## Eventos consumidos

**Ninguno actualmente.** El Auth Service no reacciona a eventos de otros servicios.

Si en el futuro necesita datos de negocio (ej: suspender cuenta por impago), deberá
suscribirse al evento correspondiente — registrar esa decisión en `decisions.md`.

---

## Garantías de entrega

**At-least-once:** Los eventos se publican después de que la transacción de BD confirma (Outbox Pattern recomendado — ver `05-architecture/pattern-guide.md`).

**Idempotencia:** Los consumidores deben ser idempotentes — pueden recibir el mismo evento más de una vez. El `eventId` es la clave de idempotencia.

---

## Correlaciones

- Patrón Outbox → `05-architecture/pattern-guide.md#outbox-pattern`
- Formato estándar de eventos → `02-domain/domain-events.md`
- Datos del usuario (modelo) → `data-model.md`
