# 15 — Control del Proyecto

> **¿Qué es esto?** La administración del proyecto: riesgos, dependencias, preguntas
> sin responder y deuda técnica. La diferencia entre un proyecto que se entrega y uno que se "va alargando".

---

## Qué hay aquí y cómo llenarlo

### `risks.md` ⭐
Registro de riesgos del proyecto.
**Llena:** desde el inicio del proyecto. Actualizar en cada sprint.

**Formato:**
```markdown
| ID | Riesgo | Probabilidad | Impacto | Severidad | Estrategia | Responsable | Estado |
|----|--------|-------------|---------|-----------|-----------|-------------|--------|
| R-001 | [Descripción] | Alta/Media/Baja | Alto/Medio/Bajo | Alta | Mitigar/Aceptar/Transferir/Evitar | [Nombre] | Abierto |
```

**Severidad = Probabilidad × Impacto:**
- Alta × Alto = 🔴 Crítico
- Alta × Bajo o Baja × Alto = 🟡 Moderado
- Baja × Bajo = 🟢 Bajo

**Estrategias:**
- **Mitigar:** reducir la probabilidad o el impacto
- **Aceptar:** registrar y monitorear, no actuar
- **Transferir:** mover el riesgo (seguros, contratos)
- **Evitar:** cambiar el plan para eliminar el riesgo

### `dependencies.md`
Dependencias externas del proyecto.
**Llena:** servicios de terceros, equipos externos, decisiones pendientes de stakeholders.

**Formato:**
```markdown
| Dependencia | Tipo | Necesaria para | Responsable externo | Fecha esperada | Estado |
|------------|------|---------------|---------------------|----------------|--------|
| [API de tercero] | Externa | [Feature X] | [Empresa/Equipo] | [Fecha] | Esperando |
```

### `open-questions.md` ⭐
Preguntas sin responder que bloquean o podrían bloquear el proyecto.
**Llena:** cada vez que el equipo se topa con algo que no sabe y que necesita una decisión.
**Criticidad:** cuando una pregunta tiene respuesta, pásala a un ADR (si es arquitectónica) o ciérrala aquí.

**Formato:**
```markdown
| # | Pregunta | Contexto | Necesaria para | Quien responde | Fecha límite | Estado |
|---|---------|---------|---------------|---------------|-------------|--------|
| Q-001 | ¿Usamos JWT o sessions? | Múltiples servicios necesitan auth | Sprint 2 | Líder técnico | [Fecha] | 🔴 Sin responder |
```

### `technical-backlog.md`
Deuda técnica y mejoras técnicas identificadas.
**Llena:** cuando el equipo identifica algo que "funciona pero no como debería".
Separar claramente de las HUs de producto.

**Formato:**
```markdown
| ID | Descripción | Impacto si no se resuelve | Esfuerzo estimado | Prioridad | Sprint |
|----|-------------|--------------------------|-------------------|-----------|--------|
| TD-001 | Refactorizar módulo X | Mayor dificultad de mantenimiento | 3 SP | Media | Sprint 5 |
```

---

## Correlaciones con otras secciones

| Se alimenta de... | Por qué |
|------------------|---------| 
| `05-architecture/` — decisiones difíciles | Riesgos técnicos |
| `04-requirements/` — RNFs exigentes | Riesgos de rendimiento |
| Incidentes en `13-operations/` | Deuda técnica post-incidente |
| `03-product/product-backlog.md` | Coordinación de prioridades técnicas vs negocio |

---

## Preguntas que esta sección debe responder

- ¿Qué puede salir mal y qué tan preparados estamos?
- ¿De qué cosas externas dependemos que no controlamos?
- ¿Qué decisiones están bloqueadas esperando información?
- ¿Qué código necesita mejorar antes de que se vuelva un problema mayor?
