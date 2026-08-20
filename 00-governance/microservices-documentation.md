# Estándar de Documentación por Microservicio

> Define exactamente qué documentos debe tener cada microservicio, quién los escribe,
> cuándo se crean y cuándo deben actualizarse. El incumplimiento bloquea el merge.

---

## Estructura obligatoria de cada servicio

Cada microservicio vive en `09-microservices/services/NN-nombre-servicio/` y DEBE tener:

```
09-microservices/services/NN-nombre-servicio/
├── README.md         ⭐ OBLIGATORIO desde Sprint 1
├── data-model.md     ⭐ OBLIGATORIO antes de crear migraciones
├── events.md         ⭐ OBLIGATORIO si el servicio emite/consume eventos
├── decisions.md      🔵 RECOMENDADO — decisiones técnicas internas del servicio
└── runbook.md        🟢 OBLIGATORIO antes del primer deploy a staging
```

Y su contrato OpenAPI en:
```
07-api/contracts/openapi/nombre-servicio.yaml   ⭐ OBLIGATORIO si expone endpoints REST
```

---

## README.md — Ficha técnica del servicio

**Cuándo crearlo:** Al inicio del sprint donde se crea el servicio
**Dueño:** Desarrollador asignado al servicio
**Actualizar cuando:** Cambia la responsabilidad, los puertos, las dependencias entre servicios

Contenido mínimo (usa `_template/service/README.md`):

| Sección | Qué debe decir |
|---------|----------------|
| Responsabilidad | Una oración: qué hace y qué dato es el dueño autoritativo |
| Ubicación en la arquitectura | Puerto, repositorio, motor de BD, con quién comunica |
| Responsabilidades (qué SÍ hace) | Lista de responsabilidades concretas |
| Fuera de su alcance (qué NO hace) | Qué delegó y a quién |
| Cómo correrlo localmente | Comandos exactos, debe funcionar |
| Documentos relacionados | Links a los demás archivos del servicio |

---

## data-model.md — Modelo de datos del servicio

**Cuándo crearlo:** Antes del primer migration script
**Dueño:** Desarrollador asignado al servicio
**Actualizar cuando:** Se crea o modifica una tabla/colección

Contenido mínimo:
- Diagrama ER (Mermaid) de las tablas del servicio
- Descripción de cada tabla con sus columnas, tipos, restricciones y propósito
- Justificación del motor de BD elegido (PostgreSQL, MongoDB, Redis, etc.)
- Estrategia de migración (Flyway, Liquibase, o scripts manuales)

**Regla:** Un campo cuya razón de existir no sea obvia DEBE tener un comentario en el diagrama.

---

## events.md — Catálogo de eventos del servicio

**Cuándo crearlo:** Cuando el servicio publica o consume su primer evento de dominio
**Dueño:** Desarrollador asignado al servicio
**Actualizar cuando:** Se agrega, modifica o elimina un evento

Contenido mínimo:
- Tabla de eventos publicados: nombre, tópico/exchange, cuándo se emite, schema
- Tabla de eventos consumidos: nombre, de qué servicio viene, qué acción desencadena
- Schema del payload (puede referenciar el contrato OpenAPI o Avro)

Ver estructura estándar de eventos: `02-domain/domain-events.md`

---

## decisions.md — Decisiones técnicas del servicio

**Cuándo crearlo:** Cuando el equipo toma una decisión técnica no obvia sobre el servicio
**Dueño:** Quien tomó la decisión
**Actualizar cuando:** Se toma una nueva decisión o se revoca una anterior

Formato recomendado: miniADR (sin el rigor completo del ADR de arquitectura):
```markdown
### Decisión: [nombre corto]
**Fecha:** [fecha]
**Contexto:** [qué problema se estaba resolviendo]
**Decisión:** [qué se decidió]
**Consecuencias:** [trade-offs conocidos]
```

---

## runbook.md — Libro de operaciones del servicio

**Cuándo crearlo:** Antes del primer deploy a staging
**Dueño:** Desarrollador responsable + DevOps
**Actualizar cuando:** Se descubre un nuevo problema operacional o cambia un procedimiento

Contenido mínimo:
- Cómo verificar que el servicio está sano (health check, métricas clave)
- Síntomas conocidos y sus causas: "Si ves X, el problema es Y, la solución es Z"
- Cómo hacer rollback del servicio
- Cómo ejecutar migraciones de BD
- Alertas configuradas y qué hacer cuando se disparan

---

## Contrato OpenAPI

**Cuándo crearlo:** Antes de implementar el primer endpoint del servicio (API-first)
**Dueño:** Desarrollador asignado al servicio
**Actualizar cuando:** Se agrega, modifica o elimina un endpoint

**Regla API-First:** El contrato se escribe ANTES del código. Los tests de contrato validan
que el código cumple el contrato, no al revés.

Usar el template: `07-api/contracts/openapi/_template-service.yaml`

---

## Cómo añadir un nuevo microservicio

1. Copiar `09-microservices/_template/service/` → `09-microservices/services/NN-nombre/`
2. Actualizar `09-microservices/service-catalog.md` con la ficha del nuevo servicio
3. Actualizar `09-microservices/dependency-map.md` (o crearlo si no existe)
4. Copiar `07-api/contracts/openapi/_template-service.yaml` → `07-api/contracts/openapi/nombre-servicio.yaml`
5. Crear un PR con al menos el README.md y el contrato API esbozado

---

## Correlaciones

- Template de servicio → `09-microservices/_template/service/`
- Catálogo de servicios → `09-microservices/service-catalog.md`
- Contratos API → `07-api/contracts/openapi/`
- Reglas de documentación generales → `00-governance/documentation-rules.md`
