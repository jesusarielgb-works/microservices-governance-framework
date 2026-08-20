# Convenciones de Git

> **Lee este documento antes de hacer tu primer commit en el proyecto.**

## Estrategia de ramas

```
main        ← Producción. Solo merges de release. Siempre estable.
  └── dev   ← Integración continua. Merges de features.
        └── feat/[descripcion]   ← Una rama por feature/HU
        └── fix/[descripcion]    ← Una rama por bugfix
        └── chore/[descripcion]  ← Cambios de infra, docs, dependencias
        └── hotfix/[descripcion] ← Correcciones urgentes a main
```

**Reglas:**
- Nadie hace commit directamente a `main` ni a `dev`
- Toda tarea = una rama + un Pull Request
- Una rama = una tarea (no mezclar features diferentes)
- Las ramas se eliminan después del merge

---

## Formato de nombre de rama

```
[tipo]/[descripcion-en-kebab-case]

Ejemplos:
feat/login-oauth2
fix/calculo-horario-solapamiento
chore/actualizar-dependencias-spring
hotfix/token-expiracion-nula
```

---

## Formato de commits (Conventional Commits)

```
[tipo]([alcance]): [descripcion en minúsculas, imperativo, sin punto final]

[cuerpo opcional - explicar el POR QUÉ, no el qué]

[footer opcional - referencias a issues/HUs]
```

**Tipos:**
| Tipo | Cuándo usarlo |
|------|--------------|
| `feat` | Nueva funcionalidad |
| `fix` | Corrección de bug |
| `docs` | Solo documentación |
| `style` | Formato, espacios (no lógica) |
| `refactor` | Refactoring sin cambio de comportamiento |
| `test` | Agregar o modificar pruebas |
| `chore` | Herramientas, dependencias, CI |
| `perf` | Mejora de rendimiento |

**Ejemplos:**
```
feat(iam): implementar login con JWT

fix(scheduling): corregir validación de solapamiento de horarios
Closes #42

docs(api): actualizar contrato OpenAPI del servicio de actores

chore(deps): actualizar Spring Boot a 3.2.0
```

---

## Política de Pull Requests

- **Tamaño:** máximo 400 líneas de código (sin contar tests). Si es mayor, dividir.
- **Revisores:** mínimo 1 aprobación antes de mergear
- **Tiempo de revisión:** el revisor tiene máximo 24 horas hábiles
- **Template:** usa el template en `.github/pull_request_template.md`
- **CI verde:** el merge solo procede si todos los checks del pipeline pasan

---

## Política de merge

- Usar **Squash and Merge** para features (mantiene `dev` limpio)
- Usar **Merge Commit** para releases a `main` (preserva el historial)
- **No** usar Rebase & Merge (genera confusión en el historial compartido)

---

## Tags y versiones

Seguimos [SemVer](https://semver.org/): `MAJOR.MINOR.PATCH`

```bash
# Al hacer release a producción
git tag -a v1.2.0 -m "Release v1.2.0: agregar módulo de reportes"
git push origin v1.2.0
```
