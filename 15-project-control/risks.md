# Registro de Riesgos

> Los riesgos no mitigados se convierten en problemas. Este registro permite al equipo
> anticiparse, no solo reaccionar.
> Revisa y actualiza en cada retrospectiva o cuando haya un cambio importante en el proyecto.

---

## Matriz de probabilidad × impacto

```
IMPACTO
  │
  │ ALTO  │  [Mitigar]   │  [Evitar]     │
  │       │  Probabilidad│  Probabilidad │
  │       │  Baja        │  Alta         │
  │───────────────────────────────────────
  │ MEDIO │  [Aceptar]   │  [Mitigar]    │
  │       │  con monitoreo│              │
  │───────────────────────────────────────
  │ BAJO  │  [Aceptar]   │  [Aceptar]    │
  │       │              │               │
  └───────────────────────────────────────
             BAJA          ALTA
                     PROBABILIDAD
```

**Estrategias de respuesta:**
- **Evitar:** Cambiar el plan para que el riesgo no pueda materializarse
- **Mitigar:** Reducir la probabilidad o el impacto
- **Transferir:** Pasar el riesgo a otro (seguro, proveedor, contrato)
- **Aceptar:** Reconocer el riesgo y tener un plan de contingencia

---

## Registro de riesgos activos

### R-001 — [Nombre del riesgo]

| Campo | Valor |
|-------|-------|
| **ID** | R-001 |
| **Categoría** | [Técnico / De negocio / De equipo / Externo] |
| **Descripción** | [Qué podría salir mal] |
| **Probabilidad** | [Alta / Media / Baja] |
| **Impacto** | [Alto / Medio / Bajo] |
| **Nivel de riesgo** | [Crítico / Alto / Medio / Bajo] |
| **Estrategia** | [Mitigar / Evitar / Transferir / Aceptar] |
| **Plan de mitigación** | [Qué hacemos para reducir probabilidad o impacto] |
| **Plan de contingencia** | [Qué hacemos si el riesgo se materializa] |
| **Disparador** | [Señal de alerta de que el riesgo está por ocurrir] |
| **Dueño** | [Nombre del responsable de monitorear este riesgo] |
| **Fecha de revisión** | [fecha] |
| **Estado** | [Activo / Mitigado / Ocurrido / Cerrado] |

---

### R-002 — [Nombre del riesgo]

| Campo | Valor |
|-------|-------|
| **ID** | R-002 |
| **Categoría** | [Técnico] |
| **Descripción** | [Ej: La dependencia X puede no estar disponible para la fecha de integración] |
| **Probabilidad** | [Media] |
| **Impacto** | [Alto] |
| **Nivel de riesgo** | [Alto] |
| **Estrategia** | [Mitigar] |
| **Plan de mitigación** | [Iniciar conversación con proveedor 6 semanas antes; diseñar mock/stub por si acaso] |
| **Plan de contingencia** | [Desarrollar con mock, lanzar sin la integración, agregarla en la siguiente iteración] |
| **Disparador** | [No hay respuesta del proveedor en 2 semanas] |
| **Dueño** | [Nombre] |
| **Fecha de revisión** | [fecha] |
| **Estado** | [Activo] |

---

## Riesgos técnicos comunes en microservicios

Estos riesgos aplican a casi todos los proyectos de microservicios. Evalúa cuáles aplican:

| Riesgo | Probabilidad típica | Mitigación estándar |
|--------|--------------------|--------------------|
| Cascada de fallos (un servicio caído tumba todo) | Media | Circuit Breaker, timeout, fallback |
| Inconsistencia de datos entre servicios | Alta | Saga, Outbox, eventos idempotentes |
| Latencia de red en llamadas síncronas | Alta | gRPC, cache, async donde sea posible |
| Mensajes perdidos en el broker | Media | At-least-once + consumidores idempotentes |
| Schema evolution rompe consumidores | Alta | Versionar eventos, cambios compatibles first |
| Acumulación de deuda técnica | Alta | DoD con cobertura mínima, reviews regulares |
| Over-engineering prematuro | Media | Start simple, YAGNI, medir antes de optimizar |
| Exposición de datos sensibles en logs | Media | Logging policy, PII masking, SAST |
| Drift de configuración entre ambientes | Alta | Infrastructure as Code, variables de ambiente |

---

## Riesgos cerrados / Lecciones aprendidas

| ID | Riesgo | Resultado | Lección |
|----|--------|-----------|---------|
| R-00X | [nombre] | [Ocurrió / No ocurrió] | [Qué aprendimos] |

---

## Plantilla para agregar un nuevo riesgo

```markdown
### R-00X — [Nombre]

| Campo | Valor |
|-------|-------|
| **ID** | R-00X |
| **Categoría** | |
| **Descripción** | |
| **Probabilidad** | |
| **Impacto** | |
| **Nivel de riesgo** | |
| **Estrategia** | |
| **Plan de mitigación** | |
| **Plan de contingencia** | |
| **Disparador** | |
| **Dueño** | |
| **Fecha de revisión** | |
| **Estado** | Activo |
```

---

## Correlaciones

- Dependencias externas → `15-project-control/dependencies.md`
- Preguntas abiertas relacionadas → `15-project-control/open-questions.md`
- Decisiones arquitectónicas que mitigaron riesgos → `05-architecture/decisions/`
