# Definition of Ready (DoR)

> Una Historia de Usuario está **lista para entrar al sprint** cuando cumple TODOS estos criterios.
> Si no los cumple, se devuelve al backlog para ser refinada.
>
> **Por qué importa:** Comprometer una HU que no está lista es la causa principal de que las HUs
> no se terminen en el sprint. El equipo pierde tiempo descubriendo ambigüedades durante la implementación.

---

## Criterios del Definition of Ready

### Claridad y comprensión
- [ ] La HU tiene título, descripción y criterios de aceptación escritos
- [ ] Todos en el equipo entienden qué se pide (nadie tiene preguntas sin respuesta)
- [ ] El rol de usuario está claramente definido (no es genérico "como usuario")
- [ ] El beneficio esperado está justificado ("para que..." describe un valor real)

### Criterios de Aceptación
- [ ] Los ACs están escritos en formato Gherkin o equivalente verificable
- [ ] Hay al menos un AC por cada flujo principal
- [ ] Hay ACs para los casos de error más probables
- [ ] Los ACs son testables (se puede escribir una prueba automática o manual)

### Dependencias y alcance
- [ ] Las dependencias externas están identificadas y no bloquean el trabajo
- [ ] Las dependencias de otras HUs están completadas o en progreso avanzado
- [ ] El equipo acordó qué microservicio(s) implementan esta HU
- [ ] El impacto en el modelo de datos está identificado (si aplica)
- [ ] El contrato API a crear/modificar está esbozado (si aplica)

### Estimación
- [ ] La HU está estimada (tiene story points)
- [ ] Si la estimación es ≥ 8 SP, se dividió en sub-tareas menores

### Acceso e información
- [ ] El equipo tiene acceso a los mockups o diseños necesarios (si aplica)
- [ ] Las credenciales o accesos de terceros están disponibles (si aplica)

---

## Relación con el DoD

El DoR define cuándo una HU entra al sprint.
El DoD define cuándo una HU está terminada.

```
Backlog → [DoR cumplido] → Sprint → [DoD cumplido] → Done
```

Ver: `00-governance/definition-of-done.md`

---

## ¿Quién verifica el DoR?

El **Product Owner** es responsable de mantener el backlog refinado.
El **Tech Lead** verifica los criterios técnicos (dependencias, microservicio responsable, API).
El **equipo completo** tiene derecho a rechazar una HU que no cumpla el DoR.

---

## Correlaciones

- Definition of Done → `00-governance/definition-of-done.md`
- Template de HU → `04-requirements/_template-hu.md`
- Ceremonias ágiles → `00-governance/agile-conventions.md`
