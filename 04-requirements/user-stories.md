# Historias de Usuario — Backlog

> **Qué llenar aquí:** El backlog de Historias de Usuario del producto.
> Cada HU tiene el formato estándar con Criterios de Aceptación en Given/When/Then.
> Las HUs refinadas (Ready) van al sprint. Las no refinadas son epics o ideas.

---

## Estado del backlog

| Corte | Sprint | Total HUs | Refinadas | En progreso | Completadas |
|-------|--------|-----------|-----------|-------------|-------------|
| Corte 1 | Sprint 1-2 | [N] | [N] | [N] | [N] |
| Corte 2 | Sprint 3-4 | [N] | [N] | [N] | [N] |

---

## Épicas

| ID | Épica | Descripción |
|----|-------|-------------|
| EP-001 | [Nombre de la épica] | [Descripción breve del objetivo de la épica] |
| EP-002 | [Nombre] | [Descripción] |

---

## Historias de Usuario

### HU-001 — [Nombre descriptivo] {#HU-001}

**Épica:** EP-00X

> **Como** [rol del usuario]
> **quiero** [acción / funcionalidad]
> **para** [beneficio / valor que obtiene]

**Criterios de Aceptación:**

```gherkin
Escenario 1: [Nombre del escenario — camino feliz]
  Given [contexto inicial]
  When  [acción del usuario]
  Then  [resultado esperado]
  And   [condición adicional si aplica]

Escenario 2: [Nombre del escenario — edge case / error]
  Given [contexto]
  When  [acción]
  Then  [resultado de error, ej: se muestra mensaje de validación]
```

**Definition of Done:**
- [ ] Código revisado y aprobado
- [ ] Tests unitarios escritos
- [ ] Criterios de aceptación verificados (manual o automatizado)
- [ ] API contract actualizado si aplica
- [ ] Desplegado en staging

| Campo | Valor |
|-------|-------|
| Story Points | [1 / 2 / 3 / 5 / 8 / 13] |
| Prioridad | [Must Have / Should Have / Could Have] |
| Sprint objetivo | Sprint [N] |
| Asignada a | [Nombre] |
| Estado | [Backlog / Ready / In Progress / Done] |
| Dependencias | [HU-00X, HU-00Y] |
| Servicio(s) afectado(s) | [nombre-servicio] |

---

### HU-002 — [Nombre descriptivo] {#HU-002}

**Épica:** EP-00X

> **Como** [rol]
> **quiero** [acción]
> **para** [beneficio]

**Criterios de Aceptación:**

```gherkin
Escenario 1: [Camino feliz]
  Given [contexto]
  When  [acción]
  Then  [resultado]

Escenario 2: [Caso de error]
  Given [contexto]
  When  [acción inválida]
  Then  se muestra error "[código de error]" con mensaje "[mensaje]"
```

| Campo | Valor |
|-------|-------|
| Story Points | [N] |
| Prioridad | [Must Have] |
| Sprint objetivo | Sprint [N] |
| Estado | [Backlog] |

---

## Reglas de escritura de HUs

### 1. El rol importa
No escribas "Como usuario" — eso no dice nada. Usa el rol específico:
```
✓ Como administrador del sistema
✓ Como cliente registrado
✓ Como operador de inventario
✗ Como usuario
✗ Como persona
```

### 2. El beneficio justifica el trabajo
El "para" debe describir un beneficio de negocio, no describir la acción de nuevo:
```
✓ para poder gestionar mis pedidos sin llamar a soporte
✗ para poder ver mis pedidos (esto solo describe la funcionalidad)
```

### 3. Los ACs son verificables
Cada AC debe poder ser verificado manualmente o automatizarse como test:
```
✓ Then el sistema muestra un mensaje "Pedido #123 confirmado"
✓ Then el email de confirmación llega en menos de 30 segundos
✗ Then el sistema funciona bien (no verificable)
✗ Then el usuario está satisfecho (no verificable)
```

### 4. Una HU = una unidad de valor
Si la HU tiene 15 ACs, probablemente son 3 HUs.
El equipo debe poder completarla en un sprint (máximo 2 semanas).

---

## Plantilla de HU lista para copiar

```markdown
### HU-00X — [Nombre] {#HU-00X}

**Épica:** EP-00X

> **Como** [rol]
> **quiero** [acción]
> **para** [beneficio]

**Criterios de Aceptación:**

\```gherkin
Escenario 1: [nombre]
  Given [contexto]
  When  [acción]
  Then  [resultado]
\```

| Campo | Valor |
|-------|-------|
| Story Points | |
| Prioridad | |
| Sprint objetivo | |
| Estado | Backlog |
| Dependencias | |
```

---

## Correlaciones

- Template completo con DoD checklist → `04-requirements/_template-hu.md`
- Requisitos no funcionales → `04-requirements/non-functional.md`
- Matriz de trazabilidad → `04-requirements/traceability-matrix.md`
- Contratos API derivados de estas HUs → `07-api/contracts/openapi/`
