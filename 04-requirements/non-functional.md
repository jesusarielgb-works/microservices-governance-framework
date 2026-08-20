# Requisitos No Funcionales (RNF)

> Los RNFs definen las **cualidades del sistema**, no lo que hace sino cómo lo hace.
> La regla de oro: todo RNF debe tener una métrica. "El sistema debe ser rápido" no es un RNF.
> "El P99 de latencia del endpoint /pedidos debe ser < 200ms bajo carga de 500 RPS" sí lo es.

---

## ¿Cómo escribir un RNF medible?

| Malo | Bueno |
|------|-------|
| "El sistema debe ser rápido" | "El P95 de latencia debe ser < 300ms bajo 1000 RPS concurrentes" |
| "El sistema debe ser seguro" | "Todos los endpoints requieren JWT válido; tokens expiran en 1 hora" |
| "El sistema debe escalar" | "El sistema debe soportar hasta 5000 usuarios concurrentes sin degradación" |
| "El sistema debe estar disponible" | "SLO de disponibilidad: 99.9% mensual (máximo 44 min de inactividad/mes)" |

---

## RNF-001: Rendimiento (Performance)

| Atributo | Métrica | Condición de prueba |
|---------|---------|---------------------|
| Latencia P95 — endpoints críticos | < 300ms | Bajo carga de [N] RPS |
| Latencia P99 — endpoints críticos | < 500ms | Bajo carga de [N] RPS |
| Latencia P95 — endpoints no críticos | < 1000ms | Carga normal |
| Throughput mínimo | [N] RPS | Sin degradación |
| Tiempo de inicio del servicio | < 30 segundos | Cold start |

**Endpoints críticos definidos:**
- `POST /[recurso]` — [justificación por qué es crítico]
- `GET /[recurso]/:id` — [justificación]

**Herramientas de prueba de carga:**
- k6, Apache JMeter, Locust, Gatling

**¿Dónde se valida?** CI/CD en el pipeline de staging antes de producción.

---

## RNF-002: Disponibilidad (Availability)

| Ambiente | SLO | Ventana de mantenimiento | Tiempo máx. inactividad/mes |
|---------|-----|--------------------------|---------------------------|
| Producción | 99.9% | Domingos 2am-4am | 44 minutos |
| Staging | 95% | Sin restricción | 36 horas |

**Error Budget mensual en producción:** 44 minutos
**Política de Error Budget:** Si se consume > 50% del error budget en la primera mitad del mes, 
se congela el despliegue de features hasta el siguiente mes y se prioriza estabilidad.

**Health checks:**
- `GET /health` — Liveness: responde 200 si el proceso está vivo
- `GET /health/ready` — Readiness: responde 200 solo si puede procesar tráfico (BD conectada, dependencias OK)

---

## RNF-003: Escalabilidad (Scalability)

| Escenario | Comportamiento esperado |
|-----------|------------------------|
| Crecimiento gradual de carga | Auto-scaling horizontal activado cuando CPU > 70% |
| Pico repentino (Black Friday, etc.) | El sistema escala en < 2 minutos |
| Reducción de carga | Scale-down sin interrupciones al tráfico activo |
| Límite de escalado horizontal | Hasta [N] instancias por servicio |

**Estrategia:** Escalado horizontal stateless — cada instancia no guarda estado en memoria.
El estado va en Redis (sesiones, caché) o PostgreSQL (datos persistentes).

---

## RNF-004: Seguridad (Security)

### Autenticación y Autorización
- Todos los endpoints privados requieren JWT válido en el header `Authorization: Bearer <token>`
- Los tokens JWT expiran en **1 hora**
- Refresh tokens con validez de **7 días**
- RBAC (Role-Based Access Control): roles definidos en `00-governance/security-policy.md`

### Transmisión de datos
- HTTPS obligatorio en producción (TLS 1.2+)
- HTTP solo en desarrollo local

### Datos sensibles
- Contraseñas: hashing con bcrypt (cost factor ≥ 12) o Argon2id
- PII (datos personales): encriptados en reposo
- Secretos/keys: solo en variables de ambiente o vault, **nunca en el código**

### OWASP Top 10
El código debe revisarse contra el OWASP Top 10 en cada release.
Herramientas: SAST (SonarQube/Snyk), dependency scanning, DAST en staging.

### Cumplimiento regulatorio
- [GDPR / Habeas Data / PCI-DSS / etc.] — según aplique al proyecto

---

## RNF-005: Observabilidad (Observability)

| Pilar | Requisito | Herramienta |
|-------|-----------|-------------|
| Logs | Formato JSON estructurado + Correlation ID | Winston / Logback |
| Métricas | RED (Rate, Errors, Duration) por endpoint | Prometheus + Grafana |
| Trazas | Trazas distribuidas end-to-end | OpenTelemetry + Jaeger |
| Alertas | Alerta en < 5 min cuando SLI viola SLO | Alertmanager / PagerDuty |

**Correlation ID:** Cada request externo genera un UUID correlationId propagado en todos los logs y spans de esa transacción.

---

## RNF-006: Mantenibilidad (Maintainability)

| Métrica | Objetivo |
|---------|---------|
| Cobertura de pruebas | ≥ 80% de líneas (≥ 90% en el dominio) |
| Complejidad ciclomática | ≤ 10 por función |
| Deuda técnica | Tiempo de resolución < 1 sprint desde su registro |
| Tiempo de onboarding | Un nuevo dev puede desplegar localmente en < 1 hora siguiendo `10-devops/local-setup.md` |
| Tiempo promedio de build | < 5 minutos en CI |

---

## RNF-007: Portabilidad (Portability)

- Todos los servicios se despliegan como imágenes Docker
- Las imágenes funcionan en cualquier ambiente con Kubernetes 1.28+
- Ningún servicio depende del sistema operativo del host
- Las variables de ambiente son la única fuente de configuración específica del ambiente

---

## RNF-008: Recuperación ante desastres (DR / Recovery)

| Escenario | RTO (Recovery Time Objective) | RPO (Recovery Point Objective) |
|-----------|------------------------------|-------------------------------|
| Fallo de un servicio individual | < 2 minutos (K8s restart) | 0 (stateless) |
| Fallo de base de datos primaria | < 5 minutos (failover a réplica) | < 1 segundo (replicación síncrona) |
| Pérdida de una zona de disponibilidad | < 15 minutos | < 5 minutos |
| Desastre completo de región | < 4 horas (DR en región secundaria) | < 1 hora |

---

## Matriz de prioridad de RNFs

| RNF | Prioridad (P1/P2/P3) | ¿Validado en CI? | Responsable |
|-----|---------------------|-----------------|-------------|
| Rendimiento | P1 | Sí (k6 en staging) | [Tech Lead] |
| Disponibilidad | P1 | Sí (health checks) | [DevOps] |
| Seguridad | P1 | Sí (SAST + OWASP) | [Security] |
| Escalabilidad | P2 | Manual (trimestral) | [DevOps] |
| Observabilidad | P1 | Sí (smoke test en CI) | [Tech Lead] |
| Mantenibilidad | P2 | Sí (cobertura en CI) | [Equipo] |

---

## Correlaciones

- SLOs y SLAs detallados → `13-operations/README.md`
- Pipeline que valida RNFs → `10-devops/README.md`
- Incidentes relacionados con violación de RNFs → `13-operations/incident-management.md`
- Checklist de seguridad → `00-governance/security-policy.md`
