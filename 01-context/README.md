# 01 — Contexto del Proyecto

> **¿Qué es esto?** El "por qué" del sistema. Cualquier persona nueva debe poder leer esta
> carpeta y entender qué problema resuelve el proyecto, qué incluye y qué NO incluye.

## Por qué existe esta sección

Antes de diseñar cualquier cosa, el equipo necesita acordar:
- ¿Qué problema estamos resolviendo?
- ¿Para quién?
- ¿Qué está dentro y qué está fuera del alcance?
- ¿Qué significa cada término que usamos?

Sin esto, cada miembro trabaja con supuestos distintos y el proyecto se fragmenta.

---

## Qué hay aquí y cómo llenarlo

### `overview.md` ⭐
Descripción ejecutiva del sistema en máximo 1 página.
**Llena:** nombre del sistema, problema que resuelve, usuarios principales, tecnologías clave, 
estado actual (en construcción / en producción / legacy).

**Formato sugerido:**
```markdown
## ¿Qué es [Nombre del Sistema]?
[2-3 oraciones: qué es y para qué sirve]

## Problema que resuelve
[El dolor del usuario antes de este sistema]

## Usuarios principales
- [Rol 1]: [qué hace en el sistema]
- [Rol 2]: [qué hace en el sistema]

## Stack tecnológico
- Backend: [lenguaje/framework]
- Base de datos: [motor]
- Infraestructura: [Docker/K8s/Cloud]
```

### `scope.md` ⭐
Límites del sistema: qué hace y qué NO hace.
**Llena:** lista explícita de lo que está DENTRO y FUERA del alcance del MVP y de versiones futuras.
Esto previene el "scope creep" (el sistema que crece sin control).

**Formato:**
```markdown
## En alcance (MVP)
- [Funcionalidad 1]
- [Funcionalidad 2]

## Fuera de alcance (MVP)
- [Lo que deliberadamente NO hacemos]

## Candidatos para versiones futuras
- [Lo que podría venir después]
```

### `glossary.md` ⭐
Diccionario del dominio del proyecto.
**Llena:** todos los términos técnicos y de negocio que se usan en el proyecto, con su definición exacta.
Si dos personas definen "cliente" de forma diferente, el sistema tendrá bugs.

**Formato:**
```markdown
| Término | Definición | Sinónimos | Notas |
|---------|-----------|-----------|-------|
| [Término] | [Definición precisa en contexto de este sistema] | [si existen] | [si aplica] |
```

### `_template-project-profile.md`
Ficha técnica del proyecto para registros internos.
**Llena:** cuando el proyecto se formaliza (nombre oficial, líder técnico, fechas, stakeholders).

### `_template-scope-declaration.md`
Template formal de declaración de alcance para presentaciones o entregas.

---

## Correlaciones con otras secciones

| Si cambias esto... | Revisar también... |
|--------------------|--------------------|
| El problema descrito en `overview.md` | La visión de producto en `03-product/vision.md` |
| El alcance en `scope.md` | Los requisitos en `04-requirements/`, el PRD en `03-product/` |
| Un término del `glossary.md` | Todo documento donde ese término aparezca |

---

## Orden de llenado recomendado

1. `overview.md` — 30 minutos con el equipo completo
2. `scope.md` — 1 hora de discusión (lo más valioso que pueden hacer al inicio)
3. `glossary.md` — crece durante todo el proyecto, empieza con 10 términos clave

---

## Preguntas que esta sección debe responder

- ¿Para qué existe este sistema?
- ¿Quiénes son los usuarios?
- ¿Qué NO hace el sistema?
- ¿Qué significa [término X] en este proyecto?
