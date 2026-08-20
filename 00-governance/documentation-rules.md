# Reglas de Documentación

> Estas reglas determinan cómo se escribe, organiza y mantiene la documentación de este proyecto.
> Una documentación que no sigue estas reglas puede ser rechazada en code review.

---

## Principio fundamental

> **"La documentación es código. Si no está actualizada, está rota."**

Cada HU que modifica comportamiento del sistema DEBE incluir la actualización de los documentos
afectados. El DoD lo exige.

---

## Idioma

| Artefacto | Idioma |
|-----------|--------|
| Código fuente (variables, funciones, clases) | [Inglés / Español] |
| Comentarios en código | [Inglés / Español] |
| Commits | [Inglés / Español] (Conventional Commits) |
| Nombres de ramas | [Inglés / Español] |
| Documentación en Markdown | [Español] |
| Contratos OpenAPI (descriptions) | [Español] |
| Mensajes de error devueltos al frontend | [Español] |
| Logs internos del sistema | [Inglés] |

> **Regla:** Una vez elegido el idioma para cada categoría, es vinculante para todo el proyecto.
> Mezclar idiomas en la misma categoría es motivo de rechazo en PR.

---

## Estructura de archivos

```
Cada sección tiene su README.md que explica el propósito de la carpeta.
Los documentos de contenido usan kebab-case.md (ej: domain-map.md, risk-register.md).
Los templates tienen prefijo _ para aparecer primero (ej: _template-hu.md, _template-adr.md).
Los ADRs se numeran secuencialmente: ADR-001-titulo-corto.md.
```

---

## Qué documentar y qué NO

### SÍ documentar

| Qué | Dónde |
|-----|-------|
| Decisiones de arquitectura no obvias | `05-architecture/decisions/records/ADR-NNN.md` |
| Reglas de negocio e invariantes del dominio | `02-domain/entities-and-rules.md` |
| Contratos API de cada servicio | `07-api/contracts/openapi/[servicio].yaml` |
| Cambios al modelo de datos | `06-data/models.md` |
| Procedimientos de operación | `13-operations/` |
| Riesgos identificados | `15-project-control/risks.md` |

### NO documentar

- Lo que el código ya dice claramente (no repetir en comentarios lo que se lee en el código)
- Decisiones temporales o experimentos que se van a revertir
- Detalle de implementación de librerías externas (esas tienen su propia documentación)
- El historial de cambios (eso es git log)

---

## Dueños de cada sección

| Sección | Dueño | Frecuencia de revisión |
|---------|-------|------------------------|
| `00-governance/` | Tech Lead | Inicio de cada sprint |
| `02-domain/` | Tech Lead + PO | Cuando cambia el dominio |
| `04-requirements/` | Product Owner | Cada sprint |
| `05-architecture/` | Tech Lead | Cada decisión de diseño |
| `07-api/contracts/` | Desarrollador dueño del servicio | Cada cambio de API |
| `09-microservices/` | Desarrollador dueño del servicio | Cada release |
| `13-operations/` | DevOps / On-call | Después de cada incidente |
| `15-project-control/` | Tech Lead | Revisión semanal |

---

## Formato de los documentos

### Encabezados
- `# H1` — solo uno por archivo, es el título
- `## H2` — secciones principales
- `### H3` — subsecciones
- No usar H4 o más profundo; si lo necesitas, el documento tiene demasiada jerarquía

### Tablas
Usar tablas para comparaciones, registros y matrices. No usar tablas para listas simples.

### Código
Siempre usar bloques de código con el lenguaje especificado:
````
```typescript
const x = 1;
```
````

### Instrucciones de plantilla
Los bloques `> [!NOTE] INSTRUCCIONES` indican que el documento es una plantilla sin llenar.
Eliminarlos cuando el documento esté completo.

---

## Proceso de actualización

1. El desarrollador identifica qué documentos afecta su cambio
2. Actualiza los documentos junto con el código (mismo PR)
3. El reviewer verifica que la documentación esté actualizada
4. Si el PR cierra una HU que tenía impacto en API → el contrato OpenAPI debe estar actualizado

---

## Correlaciones

- Git conventions → `00-governance/git-conventions.md`
- Estándar de documentación por microservicio → `00-governance/microservices-documentation.md`
- Definition of Done (doc como parte del DoD) → `00-governance/definition-of-done.md`
