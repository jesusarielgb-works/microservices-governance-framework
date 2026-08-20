# 00 — Gobernanza del Proyecto

> **¿Qué es esto?** Las reglas del equipo. Antes de escribir una línea de código o documentación,
> todos los miembros deben leer y acordar lo que está en esta carpeta.

## Por qué existe esta sección

Sin reglas explícitas, cada desarrollador trabaja a su manera:
- Nombres de ramas inconsistentes → merges caóticos
- Commits sin sentido → historial ilegible
- Sin Definition of Done → PR que nunca están "listos"
- Sin política de seguridad → vulnerabilidades ignoradas

Esta sección previene esos problemas **antes de que ocurran**.

---

## Qué hay aquí y cómo llenarlo

### `git-conventions.md` ⭐
Define cómo trabaja el equipo con Git.
**Llena:** estrategia de ramas (Gitflow / trunk-based), naming de ramas (`feat/`, `fix/`, `chore/`),
formato de commits (Conventional Commits recomendado), política de PRs, quién aprueba.

**Ejemplo de commit correcto:**
```
feat(iam): agregar login con OAuth2
fix(scheduling): corregir cálculo de solapamiento de horarios
docs(api): actualizar contrato OpenAPI del servicio de actores
```

### `agile-conventions.md`
Cómo trabaja el equipo en sprints.
**Llena:** duración del sprint, ceremonias (planning, daily, retro, review), herramienta de backlog,
escala de estimación (Fibonacci, T-shirt), velocidad esperada.

### `definition-of-done.md` ⭐
Cuándo una Historia de Usuario está **terminada**.
**Llena:** lista de checks que TODA HU debe cumplir antes de cerrarla.
Ejemplo mínimo: código revisado, pruebas escritas y pasando, documentación actualizada, desplegado en staging.

### `definition-of-ready.md`
Cuándo una HU está **lista para entrar al sprint**.
**Llena:** criterios que debe cumplir antes de planificar (descripción clara, ACs definidos, estimada, sin dependencias bloqueantes).

### `documentation-rules.md`
Reglas de cómo escribir y mantener esta documentación.
**Llena:** idioma del proyecto, quién es dueño de cada sección, frecuencia de revisión, qué NO documentar.

### `security-policy.md`
Reglas de seguridad del equipo.
**Llena:** manejo de secretos (nunca en código, usar vault/env), reporte de vulnerabilidades, política de dependencias, quién tiene acceso a qué ambiente.

### `security-rules.md`
Reglas técnicas de seguridad para el código.
**Llena:** OWASP Top 10 que aplican al proyecto, validaciones obligatorias en inputs, política de autenticación y autorización.

### `microservices-documentation.md`
Estándar de documentación para cada microservicio.
**Llena:** qué archivos debe tener cada servicio (ver `09-microservices/_template/`), quién los actualiza, cuándo.

---

## Correlaciones con otras secciones

- Las reglas de Git de aquí → afectan cómo se trabaja en `09-microservices/` (cada servicio en su propio repo o monorepo)
- El DoD de aquí → es la puerta de salida para tareas en `15-project-control/`
- La política de seguridad → se implementa técnicamente en `05-architecture/security-threat-model.md` y `07-api/authentication.md`

---

## Preguntas que esta sección debe responder

- ¿Cómo nombramos las ramas de Git?
- ¿Qué formato tienen los commits?
- ¿Cuándo podemos decir que una tarea está "lista"?
- ¿Dónde van los secretos y credenciales?
- ¿Quién tiene permiso de hacer merge a `main`?
