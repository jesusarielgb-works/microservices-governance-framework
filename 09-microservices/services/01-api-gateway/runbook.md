# Runbook — API Gateway

> Procedimientos de operación para quien esté en guardia.
> Cada sección es ejecutable sin contexto adicional.

**Servicio:** api-gateway
**Puerto local:** 8080
**Versión:** 1.0
**Última actualización:** [YYYY-MM-DD]

---

## 1. Información rápida

| Campo | Valor |
|-------|-------|
| Puerto local | 8080 |
| URL producción | `https://api.[dominio]/` |
| URL staging | `https://staging.api.[dominio]/` |
| Dashboard Grafana | [URL del panel de gateway] |
| Canal de alertas | [#alerts] |
| Escalamiento | [Tech Lead — contacto] |
| RTO objetivo | 5 min (es la puerta de entrada — P0 inmediato) |
| RPO objetivo | N/A (stateless) |

---

## 2. Verificar salud

```bash
# Health check público (no requiere autenticación)
curl https://api.[dominio]/health

# Respuesta esperada:
# {"status": "ok", "timestamp": "2024-01-15T10:30:00Z"}

# Si hay problemas, verificar los upstreams:
curl -H "Authorization: Bearer [token-dev]" https://api.[dominio]/api/v1/auth/health
```

---

## 3. Alertas frecuentes

### Alta tasa de errores 502/503 (upstream no responde)

**Síntoma:** Los servicios detrás del gateway devuelven 502 o 503.

```bash
# Ver a qué servicio corresponden los errores
kubectl logs -n [ns] -l app=api-gateway --tail=200 | grep '"status":50'

# Verificar que los pods del servicio afectado estén running
kubectl get pods -n [ns] -l app=[servicio-afectado]
```

**Árbol de decisión:**
- ¿El servicio afectado tiene pods `Running`? → El problema puede ser una excepción no manejada. Ver los logs del servicio.
- ¿Los pods están en `CrashLoopBackOff`? → El servicio se cayó. Escalar el runbook del servicio afectado.
- ¿Todos los servicios están caídos? → Problema de red o del gateway mismo. Ver sección 3.3.

### Alta tasa de errores 401 (token inválido)

**Síntoma:** Muchos 401. Puede ser ataque o clave JWT rotada.

```bash
# Ver IPs con más 401s
kubectl logs -n [ns] -l app=api-gateway --tail=500 | \
  grep '"status":401' | jq '.ip' | sort | uniq -c | sort -rn | head
```

- ¿Una IP sola genera todos los 401? → Posible ataque. Agregar a denylist.
- ¿Muchas IPs generan 401 después de un deploy? → La clave pública JWT puede haber cambiado. Verificar variables de ambiente del gateway.

### Rate limit disparado globalmente

```bash
# Ver cuántas peticiones bloqueadas por rate limit
kubectl logs -n [ns] -l app=api-gateway --tail=200 | grep '"status":429'
```

Si el rate limit está disparado en condiciones normales, el límite puede ser muy bajo.
Coordinar con Tech Lead antes de modificarlo.

---

## 4. Rollback

El API Gateway es stateless — el rollback es un redeploy de la versión anterior:

```bash
kubectl rollout history deployment/api-gateway -n [ns]
kubectl rollout undo deployment/api-gateway -n [ns]
kubectl rollout status deployment/api-gateway -n [ns]
```

---

## 5. Escalar el gateway

```bash
# Escalar horizontalmente (sin downtime, stateless)
kubectl scale deployment/api-gateway -n [ns] --replicas=3
kubectl get pods -n [ns] -l app=api-gateway
```

---

## 6. Post-incidente

- [ ] Gateway respondiendo con latencia P95 < 50ms
- [ ] Zero 5xx en los últimos 5 minutos
- [ ] Incidente registrado en `13-operations/incident-management.md`
- [ ] Si el incidente causó SLA breach: notificar al Tech Lead
