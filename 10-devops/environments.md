# Ambientes de Despliegue

> Define los ambientes del sistema, sus propósitos, acceso y configuraciones clave.
> **Regla:** Si necesitas acceder a producción directamente, algo está mal en el proceso.
> La mayoría de los problemas se deben detectar en staging.

---

## Ambientes del proyecto

| Ambiente | Propósito | Actualización | Acceso | Datos |
|---------|-----------|--------------|--------|-------|
| **local** | Desarrollo en máquina del dev | Manual | Dev individual | Sintéticos / seed |
| **dev** | Integración continua | Automático (cada PR merge a `develop`) | Todo el equipo | Sintéticos |
| **staging** | Pre-producción, QA, demos | Automático (cada merge a `main`) | Equipo + PO | Anonimizados de prod |
| **production** | Usuarios reales | Manual (aprobación requerida) | Solo Tech Lead + DevOps | Reales |

---

## Variables de ambiente por ambiente

> Las variables específicas van en el sistema de secretos del ambiente (Vault, AWS Secrets, K8s Secrets).
> Los valores de local van en `.env.example` (sin datos reales).

### Convención de nombres

```
APP_[SERVICIO]_[VARIABLE]

Ejemplos:
  APP_AUTH_JWT_SECRET
  APP_PEDIDOS_DATABASE_URL
  APP_NOTIFICATIONS_SMTP_HOST
```

### Variables por ambiente (valores de ejemplo — los reales en el vault)

| Variable | Local | Dev | Staging | Production |
|----------|-------|-----|---------|------------|
| `DATABASE_URL` | `postgresql://dev:dev@localhost/dev_db` | Vault | Vault | Vault |
| `REDIS_URL` | `redis://localhost:6379` | Vault | Vault | Vault |
| `JWT_SECRET` | `dev-secret-only-local` | Vault | Vault | Vault |
| `LOG_LEVEL` | `debug` | `info` | `info` | `warn` |
| `NODE_ENV` | `development` | `development` | `staging` | `production` |
| `CORS_ORIGIN` | `http://localhost:3000` | `https://dev.[dominio]` | `https://staging.[dominio]` | `https://[dominio]` |

---

## CI/CD Pipeline

```
Desarrollador
  │
  │ git push feature/HU-XXX
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│  PIPELINE DE PR (corre en cada push a una rama)         │
│  ─────────────────────────────────────────────          │
│  1. Lint + Format check      (< 2 min)                  │
│  2. Unit Tests               (< 5 min)                  │
│  3. Build                    (< 3 min)                  │
│  ─────────────────────────────────────────────          │
│  Si falla: bloquea el merge del PR                      │
└─────────────────────────────────────────────────────────┘
  │
  │ PR aprovado y mergeado a `develop`
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│  PIPELINE DE DEVELOP (deploy a dev)                     │
│  ─────────────────────────────────────────────          │
│  1. Lint + Build + Unit Tests   (< 10 min)             │
│  2. Integration Tests           (< 15 min)              │
│  3. Build Docker image          (< 5 min)               │
│  4. Deploy a dev                (< 5 min)               │
│  5. Smoke tests en dev          (< 3 min)               │
└─────────────────────────────────────────────────────────┘
  │
  │ Release: merge develop → main
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│  PIPELINE DE MAIN (deploy a staging)                    │
│  ─────────────────────────────────────────────          │
│  1. Todos los tests             (< 20 min)              │
│  2. Contract Tests              (< 5 min)               │
│  3. Build + push imagen con tag (< 5 min)               │
│  4. Deploy a staging            (< 5 min)               │
│  5. Smoke tests en staging      (< 5 min)               │
│  6. Performance tests (k6)      (< 15 min)              │
│  7. Notificar PO (listo para QA)                        │
└─────────────────────────────────────────────────────────┘
  │
  │ Aprobación manual del Tech Lead
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│  DEPLOY A PRODUCCIÓN                                    │
│  ─────────────────────────────────────────────          │
│  1. Canary deploy (10% del tráfico)                     │
│  2. Monitorear métricas 15 min                          │
│  3. Si OK: 100% del tráfico                             │
│  4. Si KO: rollback automático                          │
└─────────────────────────────────────────────────────────┘
```

---

## Estrategia de deploy a producción

### Opciones disponibles

| Estrategia | Descripción | Riesgo | Rollback |
|-----------|-------------|--------|---------|
| **Canary** | 10% → 50% → 100% con validación en cada paso | Bajo | Automático |
| **Blue-Green** | Cambio instantáneo entre dos versiones | Medio | Inmediato |
| **Rolling** | Reemplaza instancias una por una | Bajo | Lento |
| **Recreate** | Baja todo y sube la nueva versión | Alto | Manual | 

**Estrategia adoptada:** [Canary / Blue-Green / Rolling]

### Checklist de deploy a producción

- [ ] El PR del release fue revisado y aprobado
- [ ] Los tests de staging pasaron (incluyendo performance)
- [ ] El PO aprobó las HUs incluidas en este release
- [ ] El Tech Lead aprobó el deploy
- [ ] Hay alguien disponible para monitorear los primeros 30 min post-deploy
- [ ] Si hay cambios de base de datos: la migración se ejecutó antes del deploy del código
- [ ] El runbook de rollback está disponible: `[link al runbook]`

### Rollback

```bash
# Rollback de Kubernetes a la versión anterior
kubectl rollout undo deployment/[nombre-servicio] -n [namespace]

# Verificar el rollback
kubectl rollout status deployment/[nombre-servicio] -n [namespace]

# Si hay migración de BD incompatible (EMERGENCIA — consultar primero con Tech Lead)
npm run db:migrate:revert
```

---

## Registro de deploys

| Fecha | Servicio | Versión | Ambiente | Desplegado por | Issues |
|-------|---------|---------|---------|----------------|--------|
| [fecha] | [servicio] | v[X.Y.Z] | production | [nombre] | Ninguno |

---

## Correlaciones

- Setup local → `10-devops/local-setup.md`
- Observabilidad post-deploy → `13-operations/observability.md`
- Runbook de cada servicio → `09-microservices/services/XX/runbook.md`
- Política de branches → `00-governance/git-conventions.md`
