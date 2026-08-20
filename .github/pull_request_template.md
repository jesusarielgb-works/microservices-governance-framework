## ¿Qué hace este PR?

> Describe brevemente el cambio. Una o dos oraciones que un revisor pueda leer en 10 segundos.

---

## Tipo de cambio

- [ ] Nueva funcionalidad (feat)
- [ ] Corrección de bug (fix)
- [ ] Refactor sin cambio de comportamiento (refactor)
- [ ] Documentación (docs)
- [ ] Infraestructura / CI / configuración (chore)
- [ ] Pruebas (test)

---

## HU(s) relacionada(s)

Cierra: #[número de issue o HU]

---

## Cambios incluidos

> Lista los archivos/módulos relevantes que cambiaron y por qué. Enfócate en el "por qué", no en el "qué" — el diff ya muestra el qué.

- `[archivo o módulo]` — [razón del cambio]
- `[archivo o módulo]` — [razón del cambio]

---

## Pruebas realizadas

- [ ] Tests unitarios escritos / actualizados y pasando (`npm test`)
- [ ] Tests de integración pasando (si aplica)
- [ ] Probé manualmente el flujo principal en local
- [ ] Probé los casos de error más probables

### Casos de prueba ejecutados

| Escenario | Resultado esperado | ¿Pasó? |
|-----------|-------------------|--------|
| [flujo principal] | [resultado] | ✅ / ❌ |
| [caso de error] | [resultado] | ✅ / ❌ |

---

## Checklist de documentación

- [ ] Si cambia el comportamiento de un endpoint → el contrato OpenAPI está actualizado
- [ ] Si cambia el modelo de datos → `06-data/models.md` o `services/NN/data-model.md` actualizado
- [ ] Si cambia el dominio (entidades, reglas, eventos) → `02-domain/` actualizado
- [ ] Si se tomó una decisión de arquitectura → ADR escrito en `05-architecture/decisions/records/`
- [ ] Si hay un nuevo servicio → `09-microservices/service-catalog.md` actualizado

---

## Definition of Done

- [ ] Código revisado por al menos 1 persona
- [ ] Pruebas unitarias escritas (cobertura ≥ 80% en los archivos modificados)
- [ ] No hay warnings de linter en los archivos modificados
- [ ] Documentación actualizada (ver checklist arriba)
- [ ] El PR despliega exitosamente en el ambiente de staging (pipeline verde)

Ver DoD completo: [`00-governance/definition-of-done.md`](../00-governance/definition-of-done.md)

---

## Notas para el reviewer

> ¿Hay algo que quieras que el reviewer preste especial atención? ¿Alguna decisión técnica discutible?
> ¿Algún contexto que el diff no muestra?

[Escribe aquí o elimina esta sección si no aplica]
