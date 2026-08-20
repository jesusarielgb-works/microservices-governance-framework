# Runbook — [Nombre del Servicio]

> Un runbook es para alguien en guardia a las 3am. Cada sección debe ser ejecutable
> sin necesidad de contexto adicional. Si un procedimiento no tiene comandos concretos
> o no dice quién lo ejecuta, no está completo.

**Servicio:** [nombre]
**Repositorio:** [URL]
**Versión:** 1.0
**Última actualización:** YYYY-MM-DD

---

## 1. Información rápida del servicio

| Campo | Valor |
|-------|-------|
| Puerto local | [8001] |
| URL producción | [https://api.domain.com/servicio] |
| URL staging | [https://staging.api.domain.com/servicio] |
| Dashboard | [URL Grafana] |
| Canal de alertas | [#alerts en Slack / PagerDuty] |
| Escalamiento | [nombre y contacto del responsable] |
| RTO objetivo | [30 min] |
| RPO objetivo | [5 min] |

---

## 2. Verificar que el servicio está sano

```bash
# Health check
curl https://api.domain.com/servicio/health

# Respuesta esperada:
# {"status": "ok", "version": "1.2.3", "db": "connected"}
```

---

## 3. Alertas frecuentes y qué hacer

### Alerta: Alta tasa de errores 5xx

**Síntoma:** Error rate > 2% durante 5 minutos.

```bash
# 1. Ver logs recientes
kubectl logs -n [namespace] -l app=[service-name] --tail=100 | grep ERROR

# 2. Ver estado de los pods
kubectl get pods -n [namespace] -l app=[service-name]

# 3. Ver eventos recientes del deployment
kubectl describe deployment [service-name] -n [namespace]
```

**Árbol de decisión:**
- ¿Hay crash loops? → Ver sección 3.3
- ¿Errores de BD? → Ver sección 4.1
- ¿Errores de servicio externo? → Ver sección 4.2
- ¿Deploy reciente? → Evaluar rollback (sección 5.3)

---

### Alerta: Latencia alta (p95 > SLO)

```bash
# Ver uso de recursos de los pods
kubectl top pods -n [namespace]

# Ver queries lentos en la BD
kubectl exec -n [namespace] [postgres-pod] -- psql -U [user] -d [db] \
  -c "SELECT pid, now()-query_start AS duration, query
      FROM pg_stat_activity
      WHERE state != 'idle' AND now()-query_start > interval '5 seconds'
      ORDER BY duration DESC LIMIT 5;"
```

---

### Alerta: Pod en CrashLoopBackOff

```bash
# Ver razón del crash
kubectl describe pod [pod-name] -n [namespace]
kubectl logs [pod-name] -n [namespace] --previous

# Restart manual
kubectl rollout restart deployment/[service-name] -n [namespace]
```

---

## 4. Procedimientos por componente

### 4.1 Base de datos

```bash
# Verificar salud
kubectl exec -n [namespace] [postgres-pod] -- pg_isready -U [user]

# Ver conexiones activas
kubectl exec -n [namespace] [postgres-pod] -- psql -U [user] -d [db] \
  -c "SELECT state, count(*) FROM pg_stat_activity GROUP BY state;"
```

### 4.2 Dependencias externas

**Si [servicio externo X] falla:**
- El circuit breaker se activa después de [N] fallos
- El servicio responde con [comportamiento degradado]
- Estado del servicio externo: [URL de status page]

---

## 5. Operaciones de mantenimiento

### 5.1 Escalar el servicio

```bash
kubectl scale deployment/[service-name] -n [namespace] --replicas=[N]
kubectl get pods -n [namespace] -l app=[service-name]  # verificar
```

### 5.2 Deploy de emergencia (hotfix)

```bash
# Solo si la imagen ya está probada en staging
kubectl set image deployment/[service-name] \
  [container]=[image]:[hotfix-tag] -n [namespace]
kubectl rollout status deployment/[service-name] -n [namespace]
```

### 5.3 Rollback

```bash
kubectl rollout history deployment/[service-name] -n [namespace]
kubectl rollout undo deployment/[service-name] -n [namespace]
```

---

## 6. Comunicación durante incidente

| Evento | Canal | Mensaje tipo |
|--------|-------|-------------|
| P0 detectado | #incidents | `[P0 INICIO] [servicio] degradado desde [HH:MM]. Investigando.` |
| Update cada 15 min | #incidents | `[P0 UPDATE] Causa probable: [...]. Acción en curso: [...]. ETA: [...]` |
| Resolución | #incidents + stakeholders | `[P0 RESUELTO] Duración: [X min]. Causa raíz: [...]. Post-mortem: [fecha]` |

---

## 7. Checklist post-incidente

- [ ] Servicio estable con métricas en objetivo SLO
- [ ] Error budget actualizado en dashboard
- [ ] Incidente registrado en `13-operations/incident-management.md`
- [ ] Stakeholders notificados de resolución
- [ ] Ticket de mejora creado en `15-project-control/technical-backlog.md`
- [ ] Post-mortem programado (si fue P0 o P1)
