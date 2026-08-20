# Convenciones Ágiles del Equipo

> Define cómo trabaja el equipo en sus ciclos de desarrollo. Acuerda y firma con todo el equipo
> antes de iniciar el primer sprint. Actualiza cuando el equipo decida cambiar algo.

---

## Estructura del Sprint

| Campo | Valor |
|-------|-------|
| Duración | [1 semana / 2 semanas / 3 semanas] |
| Inicio del sprint | [Lunes / Martes / Miércoles] |
| Fin del sprint | [Viernes de la semana N] |
| Sprint actual | Sprint [N] — [fecha inicio] al [fecha fin] |
| Capacidad estimada | [N story points por sprint] |

---

## Ceremonias

### Sprint Planning
- **Cuándo:** Primer día del sprint — [hora]
- **Duración:** Máximo [1h por semana de sprint]
- **Quiénes:** Todo el equipo
- **Objetivo:** Seleccionar y comprometer las HUs del sprint, dividir en tareas técnicas
- **Artefacto de salida:** Sprint Backlog actualizado en [herramienta: Jira / Linear / GitHub Issues]

### Daily Stand-up
- **Cuándo:** Todos los días — [hora]
- **Duración:** Máximo 15 minutos
- **Formato:**
  1. ¿Qué hice ayer?
  2. ¿Qué haré hoy?
  3. ¿Hay algo que me bloquea?
- **Regla:** Las discusiones técnicas van después del daily, no durante

### Sprint Review
- **Cuándo:** Último día del sprint — [hora]
- **Duración:** Máximo [30 min]
- **Quiénes:** Equipo + Product Owner (+ stakeholders si aplica)
- **Objetivo:** Mostrar lo que se construyó y recoger feedback

### Sprint Retrospectiva
- **Cuándo:** Último día del sprint — después del review
- **Duración:** Máximo [45 min]
- **Formato:** [Qué salió bien / Qué mejorar / Compromisos de acción]
- **Regla:** Cada retro produce mínimo 1 acción de mejora con dueño y fecha

### Refinamiento (Backlog Grooming)
- **Cuándo:** [Miércoles de la segunda semana / a mitad del sprint]
- **Duración:** Máximo [1h]
- **Objetivo:** Detallar y estimar HUs para el siguiente sprint
- **Criterio de salida de refinamiento:** La HU cumple el Definition of Ready

---

## Estimación

### Escala
| Puntos | Significado |
|--------|-------------|
| 1 | Trivial — se hace en horas |
| 2 | Pequeño — se hace en un día |
| 3 | Mediano — toma 2-3 días |
| 5 | Grande — toma casi un sprint completo |
| 8 | Muy grande — debería dividirse |
| 13 | Épico — DEBE dividirse antes de sprint |

**Técnica:** [Planning Poker / T-shirt sizing]
**Herramienta:** [nombre de la herramienta]

### Regla de estimación
- Si hay desacuerdo en 2+ niveles (ej: alguien dice 3 y otro dice 8), discutir antes de votar de nuevo.
- Si una HU se estima en 8 o 13, debe dividirse en sub-tareas menores.

---

## Herramienta de backlog

**Herramienta:** [Jira / Linear / GitHub Projects / Trello]
**URL del tablero:** [URL]

### Columnas del tablero
| Columna | Significado |
|---------|-------------|
| Backlog | Pendiente de refinar |
| Ready | Lista para entrar al sprint (cumple DoR) |
| In Progress | Alguien está trabajando en ello |
| In Review | En Pull Request / code review |
| Done | Cumple DoD y está cerrada |

---

## Velocidad del equipo

| Sprint | Story Points completados | Notas |
|--------|--------------------------|-------|
| Sprint 1 | — | — |
| Sprint 2 | — | — |
| Sprint 3 | — | — |
| Promedio | — | — |

---

## Correlaciones

- Definition of Ready → `00-governance/definition-of-ready.md`
- Definition of Done → `00-governance/definition-of-done.md`
- Gestión de riesgos → `15-project-control/risks.md`
- Backlog de deuda técnica → `15-project-control/tech-backlog.md`
