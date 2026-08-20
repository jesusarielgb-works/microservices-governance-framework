# Gestión de Incidentes

> Un incidente es cualquier evento no planificado que interrumpe o degrada el servicio.
> La respuesta efectiva a incidentes se entrena antes de que ocurran, no durante.
> Sigue este playbook en orden — en un incidente, el estrés bloquea el pensamiento libre.

---

## Clasificación de severidad

| Nivel | Nombre | Definición | Ejemplo | Tiempo de respuesta |
|-------|--------|------------|---------|---------------------|
| **P0** | Crítico | Sistema completamente caído. Impacto total a todos los usuarios. | API Gateway down, DB inaccesible | **Inmediato** (<5 min) |
| **P1** | Alto | Funcionalidad core degradada. Impacto a > 50% de usuarios. | Login no funciona, pagos fallan | **15 minutos** |
| **P2** | Medio | Funcionalidad secundaria degradada. Workaround disponible. | Exportación de reportes lenta | **1 hora** |
| **P3** | Bajo | Molestia menor. Sin impacto en operación del negocio. | Botón con label incorrecto | **Próximo sprint** |

---

## Roles durante un incidente

| Rol | Responsabilidad | Cuándo se activa |
|-----|----------------|-----------------|
| **Incident Commander (IC)** | Coordina la respuesta. Toma decisiones. Comunica. | P0 y P1 siempre |
| **Tech Lead** | Diagnóstico técnico profundo | P0 y P1 siempre |
| **On-Call Engineer** | Primer respondedor. Ejecuta acciones técnicas | Todos los incidentes |
| **Comms Lead** | Comunica a stakeholders externos | P0 y P1 |

---

## Playbook de respuesta

### Paso 1: DETECTAR (0-5 min)

```
¿Cómo sabes que hay un incidente?
  → Alerta de Prometheus / PagerDuty
  → Reporte de usuario
  → Monitoreo proactivo en el dashboard

Acción inmediata:
  1. Acepta la alerta en PagerDuty (para silenciarla y marcar que alguien respondió)
  2. Ve al canal de Slack #incidents y escribe: "Investigando [descripción breve] — [tu nombre]"
  3. Abre el runbook del servicio afectado: `09-microservices/services/[servicio]/runbook.md`
```

### Paso 2: CLASIFICAR (5-10 min)

```
Determina la severidad:
  → ¿Cuántos usuarios están afectados?
  → ¿Qué funcionalidades están caídas?
  → ¿Hay un workaround disponible?

Si es P0 o P1:
  → Activa al Incident Commander
  → Crea el ticket de incidente en [Jira/Linear]: INC-XXX
  → Abre el War Room (canal de Slack temporal o videollamada)
```

### Paso 3: COMUNICAR (10-15 min)

Actualización inicial en el canal del equipo y a los stakeholders:

```
Template de comunicación inicial:
─────────────────────────────────
🔴 INCIDENTE P[N] — [INC-XXX]
Servicio afectado: [nombre]
Impacto: [Qué no funciona para quién]
Estado: Investigando
Próxima actualización: en 30 minutos
IC: [Nombre del Incident Commander]
─────────────────────────────────
```

### Paso 4: DIAGNOSTICAR (10-30 min)

```
Herramientas de diagnóstico (en orden):
  1. Grafana → ¿Cuándo empezó el problema? ¿Qué servicio tiene alta tasa de errores?
  2. Logs (Kibana/CloudWatch) → ¿Qué dice el servicio afectado?
  3. Jaeger/Zipkin → ¿Dónde se rompe la traza?
  4. kubectl get pods / docker ps → ¿El pod/contenedor está corriendo?
  5. Health check manual → curl http://[servicio]/health

Pregunta clave: ¿Qué cambió en las últimas 2 horas?
  → Último deploy
  → Cambio de configuración
  → Aumento de tráfico
  → Cambio en un sistema externo
```

**Ver árbol de decisión específico:** `09-microservices/services/[servicio]/runbook.md`

### Paso 5: MITIGAR (15-60 min)

```
Opciones de mitigación (de más segura a más riesgosa):
  1. Rollback al último deploy estable (si fue un deploy el que causó el problema)
  2. Desactivar la feature flag que introdujo el problema
  3. Escalar horizontalmente si es un problema de capacidad
  4. Aumentar el timeout / circuit breaker temporalmente
  5. Redirigir tráfico a una región alternativa (si hay)
  6. Activar modo de mantenimiento (último recurso)
```

**Antes de cada cambio durante un incidente:**
- Anuncia en el War Room qué vas a hacer
- Espera confirmación del IC
- Ejecuta el cambio
- Reporta el resultado en 2 minutos

### Paso 6: RESOLVER Y COMUNICAR

```
El incidente está resuelto cuando:
  - Las métricas de error vuelven a niveles normales
  - Health checks responden 200 en todos los servicios
  - El PO / stakeholder confirma que los usuarios pueden operar normal

Comunicación de resolución:
─────────────────────────
✅ RESUELTO — [INC-XXX]
Causa raíz: [descripción breve]
Resolución: [qué se hizo]
Duración total: [X] minutos
Post-mortem: [fecha de la reunión]
─────────────────────────
```

---

## Post-Mortem (Retrospectiva del Incidente)

El post-mortem no es para culpar a nadie — es para entender y prevenir.
Se hace dentro de los 2 días hábiles después de resolver el incidente.

### Estructura del post-mortem

**INC-XXX — [Título del incidente]**

**Datos clave:**
- Fecha y hora inicio: 
- Fecha y hora resolución: 
- Duración total: 
- Severidad: P[N]
- Usuarios afectados: [N]

**Timeline:**

| Hora | Evento | Acción tomada |
|------|--------|---------------|
| HH:MM | [Qué ocurrió] | [Quién hizo qué] |

**Causa raíz:**
> ¿Por qué ocurrió? (5 Why's — llega a la causa real, no al síntoma)

**Contribuidores:**
> Factores que facilitaron el incidente (sin culpar personas)

**Lo que funcionó bien:**
> [Qué parte de la respuesta al incidente fue efectiva]

**Lo que no funcionó:**
> [Qué parte falló o podría mejorar]

**Action items:**

| Acción | Propietario | Fecha límite | Estado |
|--------|-------------|-------------|--------|
| [Acción concreta para prevenir recurrencia] | [Nombre] | [fecha] | Pendiente |

---

## Canales de comunicación

| Canal | Propósito |
|-------|-----------|
| Slack `#incidents` | Canal principal de todos los incidentes |
| Slack `#war-room-[INC-XXX]` | Canal temporal creado para incidentes P0/P1 |
| PagerDuty | Alertas y on-call rotation |
| [Sistema de tickets] | Registro oficial: INC-XXX |
| Status Page | Comunicación pública a usuarios (si aplica) |

---

## Correlaciones

- Alertas de observabilidad → `13-operations/observability.md`
- Runbook de cada servicio → `09-microservices/services/XX/runbook.md`
- SLOs y Error Budget → `13-operations/README.md`
