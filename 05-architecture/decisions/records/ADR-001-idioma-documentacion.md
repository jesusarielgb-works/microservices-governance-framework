# ADR-001 — Idioma de la documentación y el código

| Campo | Valor |
|-------|-------|
| **ID** | ADR-001 |
| **Fecha** | [YYYY-MM-DD] |
| **Estado** | Aceptado |
| **Autores** | [Nombre del Tech Lead] |
| **Revisores** | [Nombre del equipo] |

---

## Contexto

El equipo está formado por desarrolladores hispano-hablantes, y el cliente/stakeholder también
opera en español. Sin embargo, las convenciones de la industria del software (Stack Overflow,
documentación de librerías, artículos técnicos) usan inglés. 

Es necesario establecer una regla clara y única desde el inicio para evitar inconsistencias
que dificultan el mantenimiento: código con variables en español y comentarios en inglés,
o contratos API con campos en inglés y documentos en español.

---

## Alternativas evaluadas

### Alternativa A — Todo en inglés
- **Pros:** Estándar de la industria, fácil de contratar desarrolladores externos, librerías y frameworks en inglés, documentación técnica de referencia en inglés
- **Contras:** Barrera cognitiva para stakeholders no técnicos, términos de negocio pueden perder precisión en traducción

### Alternativa B — Todo en español
- **Pros:** Comunicación natural con el cliente, los nombres del dominio de negocio se preservan exactos
- **Contras:** Mezcla incómoda con keywords de lenguajes (if, for, return, etc.), inconsistente con el ecosistema de librerías

### Alternativa C — Dividido por capa (ELEGIDA)
- **Pros:** Cada artefacto usa el idioma más natural para su audiencia
- **Contras:** Requiere disciplina y reglas explícitas; más difícil de explicar a nuevos miembros

---

## Decisión

**Alternativa C:** Dividir por audiencia y propósito.

| Artefacto | Idioma | Razón |
|-----------|--------|-------|
| Variables, funciones, clases en código | Inglés | Consistencia con librerías y frameworks |
| Nombres de tablas y columnas en BD | Inglés | Coherencia con el código que las mapea |
| Commits (Conventional Commits) | Inglés | Estándar establecido, legible en GitHub |
| Nombres de ramas de Git | Inglés | Consistente con los commits |
| Documentación Markdown | Español | Audiencia: desarrolladores y stakeholders hispanohablantes |
| Contratos OpenAPI (descriptions) | Español | Audiencia: desarrolladores del equipo |
| Mensajes de error al usuario final | Español | Audiencia: usuarios del producto |
| Logs internos del sistema | Inglés | Facilita búsqueda en documentación de librerías y alertas |
| ADRs y documentación técnica | Español | Audiencia: equipo hispano |

---

## Consecuencias

**Positivas:**
- Los términos de negocio quedan en español en la documentación (donde importa la precisión semántica)
- El código es consistente con el ecosistema de librerías y el estándar de la industria
- Los nuevos miembros del equipo tienen una regla clara desde el día 1

**Negativas:**
- Existe una frontera de traducción entre el dominio en documentación (español) y el código (inglés) que debe gestionarse con un glosario
- Algunos nombres del dominio pueden ser ambiguos al traducirse (ej: `Appointment` vs `Cita` vs `Turno`)

**Mitigación:**
- Mantener un glosario de términos del dominio con la traducción canónica: `01-context/glossary.md`
- Cuando un término tenga traducción discutible, se documenta en el glosario antes de usarlo en el código

---

## Referencias

- Convenciones de documentación del equipo → `00-governance/documentation-rules.md`
- Glosario de términos del dominio → `01-context/glossary.md`
