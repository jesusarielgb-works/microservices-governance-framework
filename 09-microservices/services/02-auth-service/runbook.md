# Runbook — Auth Service

> Procedimientos de operación para quien esté en guardia.
> Este servicio es crítico — un P0 en auth-service impide que TODOS los usuarios accedan al sistema.

**Servicio:** auth-service
**Puerto local:** 8081
**Versión:** 1.0
**Última actualización:** [YYYY-MM-DD]

---

## 1. Información rápida

| Campo | Valor |
|-------|-------|
| Puerto local | 8081 |
| URL producción | `https://api.[dominio]/api/v1/auth` |
| URL staging | `https://staging.api.[dominio]/api/v1/auth` |
| Dashboard Grafana | [URL del panel de auth-service] |
| Canal de alertas | [#alerts] |
| Escalamiento | [Tech Lead — contacto — URGENTE si auth está caído] |
| RTO objetivo | 5 min (autenticación bloqueada = sistema inoperable) |
| RPO objetivo | 0 (PostgreSQL con réplica en tiempo real) |

---

## 2. Verificar salud

```bash
# Health check
curl https://api.[dominio]/api/v1/auth/health

# Respuesta esperada:
# {"status": "ok", "db": "connected", "redis": "connected"}

# Si hay problemas, verificar las dependencias:

# Verificar PostgreSQL
kubectl exec -n [ns] [postgres-pod] -- pg_isready -U [user]

# Verificar Redis
kubectl exec -n [ns] [redis-pod] -- redis-cli ping
# Respuesta esperada: PONG
```

---

## 3. Alertas frecuentes

### Alta tasa de errores 401 en login

**Síntoma:** Muchos intentos de login fallidos en poco tiempo (posible ataque).

```bash
# Ver IPs con más intentos
kubectl logs -n [ns] -l app=auth-service --tail=500 | \
  grep '"eventType":"user.login_failed"' | jq '.payload.ipAddress' | \
  sort | uniq -c | sort -rn | head -10
```

**Acción:** Si una IP supera [N] intentos en 5 minutos, agregar a denylist del gateway.

### Tokens de actualización no funcionan (usuarios con sesión activa no pueden renovar)

**Causa probable:** Redis caído o la clave JWT fue rotada sin actualizar todos los servicios.

```bash
# Verificar Redis
kubectl exec -n [ns] [redis-pod] -- redis-cli ping

# Verificar que la clave pública en el gateway coincide con la del auth-service
kubectl exec -n [ns] [auth-pod] -- curl localhost:8081/api/v1/auth/jwks
```

### Accounts being locked out en masa

**Síntoma:** Muchos usuarios reportan "cuenta bloqueada" simultáneamente.

**Causa probable:** (A) ataque coordinado, (B) bug de contador que se incrementa incorrectamente.

```bash
# Ver cantidad de cuentas bloqueadas actualmente
kubectl exec -n [ns] [redis-pod] -- redis-cli keys "locked:*" | wc -l

# Desbloquear una cuenta específica (solo emergencias, aprobación Tech Lead)
kubectl exec -n [ns] [redis-pod] -- redis-cli del "locked:usuario@example.com"
```

---

## 4. Rotación de claves JWT

> Solo ejecutar con aprobación del Tech Lead. Tiene impacto en todos los servicios.

```bash
# 1. Generar nuevo par de claves
openssl genrsa -out new-private.pem 2048
openssl rsa -in new-private.pem -pubout -out new-public.pem

# 2. Actualizar el secreto en el vault del ambiente
# (seguir el proceso del vault de tu infraestructura)

# 3. El JWKS endpoint soporta múltiples claves — agregar la nueva sin remover la vieja
# Esto permite que los tokens existentes (firmados con la clave vieja) sigan siendo válidos
# hasta su expiración (máx 1 hora)

# 4. Después de 1 hora: remover la clave vieja del JWKS

# 5. Verificar que el gateway puede usar la nueva clave pública
curl https://api.[dominio]/api/v1/auth/jwks
```

---

## 5. Rollback

```bash
kubectl rollout history deployment/auth-service -n [ns]
kubectl rollout undo deployment/auth-service -n [ns]
kubectl rollout status deployment/auth-service -n [ns]
```

**Si hay migraciones de BD en el deploy que se está revirtiendo:**
Contactar al Tech Lead antes de hacer rollback — puede requerirse una migración de rollback.

---

## 6. Operaciones de mantenimiento

### Limpiar refresh tokens expirados

```bash
# Si el job automático de limpieza falla:
kubectl exec -n [ns] [postgres-pod] -- psql -U [user] -d [db] \
  -c "DELETE FROM refresh_tokens WHERE expires_at < NOW() AND revoked_at IS NOT NULL;"
```

### Revocar todas las sesiones de un usuario (emergencia)

```bash
kubectl exec -n [ns] [postgres-pod] -- psql -U [user] -d [db] \
  -c "UPDATE refresh_tokens SET revoked_at = NOW() WHERE user_id = '[user-uuid]';"
```

---

## 7. Post-incidente

- [ ] Login y refresh funcionando normalmente
- [ ] Zero 5xx en los últimos 5 minutos
- [ ] Cuentas bloqueadas incorrectamente desbloqueadas (si aplica)
- [ ] Incidente registrado en `13-operations/incident-management.md`
- [ ] Si hubo breach de credenciales: escalar a protocolo de seguridad
