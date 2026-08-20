# Guía de contribución

> Bienvenido al equipo. Esta guía explica cómo hacer tu primera contribución
> y las reglas que aplican a todos los cambios en este repositorio.

---

## Antes de empezar

1. Lee `00-governance/README.md` — las reglas del equipo
2. Lee `00-governance/git-conventions.md` — cómo trabajar con Git
3. Verifica que tienes el ambiente local configurado: `10-devops/local-setup.md`
4. Entiende el dominio del sistema: `02-domain/domain-map.md`

---

## Flujo de trabajo

```
1. Elige una HU del sprint (estado: Ready)
2. Crea tu rama desde develop
3. Implementa con TDD (Red → Green → Refactor)
4. Actualiza la documentación afectada
5. Abre un Pull Request usando el template
6. El PR es revisado y mergeado por el Tech Lead
```

### Nomenclatura de ramas

```
feat/HU-[servicio]-[número]-descripcion-corta
fix/HU-[servicio]-[número]-descripcion-corta
chore/descripcion-corta
docs/descripcion-corta
```

Ejemplos:
```
feat/HU-AUTH-001-login-con-jwt
fix/HU-SCHEDULING-003-solapamiento-horarios
docs/adr-002-motor-de-bd
```

---

## Proceso de Pull Request

1. **Abre el PR** contra la rama `develop` (nunca contra `main` directamente)
2. **Llena el template** del PR completo (`/.github/pull_request_template.md`)
3. **Asigna revisores:** mínimo 1 (preferiblemente el Tech Lead o el dueño del servicio)
4. **El CI debe estar verde** — no pedir review con pipeline fallido
5. **No hagas force-push** a una rama cuyo PR ya tiene comentarios — crea commits nuevos

### Qué bloquea el merge

- Pipeline CI fallido (lint, tests, build)
- Menos de 1 aprobación
- Documentación no actualizada (API, modelo de datos, dominio)
- Cobertura de tests por debajo del mínimo (ver `11-quality/tdd-guide.md`)

---

## Estilo de código

- Seguir la configuración de ESLint/Prettier del repo (no modificar sin ADR)
- Conventional Commits obligatorio: `tipo(scope): descripción`
- Un commit = una unidad lógica de cambio. No commitear "wip" o "arreglando cosas"

### Tipos de commit

| Tipo | Cuándo usarlo |
|------|---------------|
| `feat` | Nueva funcionalidad |
| `fix` | Corrección de bug |
| `refactor` | Mejora de código sin cambio de comportamiento |
| `test` | Agregar o mejorar tests |
| `docs` | Documentación únicamente |
| `chore` | Build, CI, dependencias, config |
| `perf` | Mejora de rendimiento |

---

## TDD — Test-Driven Development

**Regla del equipo:** Las HUs se implementan en modo TDD. No hay excepciones.

```
1. Escribe el test que describe el comportamiento (RED — falla)
2. Escribe el código mínimo para pasar el test (GREEN — pasa)
3. Refactoriza sin romper los tests (REFACTOR)
```

Ver guía completa: `11-quality/tdd-guide.md`

---

## Documentación

Si tu cambio:
- Agrega/modifica un endpoint → actualiza el contrato OpenAPI
- Cambia el modelo de datos → actualiza `data-model.md` del servicio
- Toma una decisión de arquitectura → escribe un ADR
- Agrega un nuevo microservicio → actualiza `service-catalog.md`

---

## ¿Preguntas?

- Revisa la documentación en este repo — probablemente ya está respondida
- Pregunta en el daily o en el canal del equipo
- Si crees que algo falta en la documentación, documéntalo tú mismo (ese es el espíritu del proyecto)
