# Definition of Done (DoD)

> Una Historia de Usuario está **TERMINADA** cuando cumple TODOS los criterios de esta lista.
> Si falta uno solo, la HU NO está done — regresa al In Progress.

## Checklist obligatorio

### Código
- [ ] El código implementa todos los criterios de aceptación de la HU
- [ ] El código fue revisado y aprobado por al menos 1 miembro del equipo (PR review)
- [ ] El código sigue los estándares del proyecto (linting y formato pasan en CI)
- [ ] No hay deuda técnica introducida sin registrar en `15-project-control/technical-backlog.md`

### Pruebas
- [ ] Pruebas unitarias escritas para la lógica de negocio nueva
- [ ] Cobertura de pruebas no disminuye respecto al baseline del proyecto
- [ ] Todos los tests pasan localmente y en CI
- [ ] Criterios de aceptación verificados (manual o automatizado)

### Integración
- [ ] Los cambios no rompen otros servicios (pruebas de integración pasan)
- [ ] Si hay cambios en la API: contrato OpenAPI actualizado en `07-api/contracts/`
- [ ] Si hay cambios en el modelo de datos: `data-model.md` del servicio actualizado
- [ ] Si hay eventos nuevos/modificados: `event-catalog.md` actualizado

### Despliegue
- [ ] El código está mergeable a `dev` (sin conflictos)
- [ ] CI/CD verde en la rama
- [ ] Desplegado en ambiente de staging
- [ ] Smoke test básico en staging pasando

### Documentación
- [ ] `README.md` del servicio actualizado si cambió la interfaz pública
- [ ] Si fue una decisión técnica significativa: ADR creado o actualizado

---

## Excepciones permitidas

Las siguientes excepciones deben ser acordadas explícitamente por el Tech Lead:
- Pruebas E2E omitidas por limitación de ambiente (documentar el riesgo)
- Documentación diferida por entrega urgente (crear ticket de deuda técnica)

---

## ¿Qué NO es un criterio de Done?

- "El código está en mi máquina" — debe estar en el repositorio
- "Funciona en mi ambiente local" — debe funcionar en staging
- "El PM/PO lo aprobó" — eso es la Definition of Done del producto, no del código
