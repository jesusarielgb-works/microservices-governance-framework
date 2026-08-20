# 06 — Datos

> **¿Qué es esto?** Cómo el sistema almacena, estructura y migra los datos.
> En microservicios, la gestión de datos es uno de los retos más complejos.

## Principio fundamental en microservicios

> **Cada microservicio es dueño de sus propios datos.**

Ningún servicio debe acceder directamente a la base de datos de otro. Si necesita datos de otro
servicio, los solicita vía API o los recibe vía evento. Este principio garantiza independencia.

---

## Qué hay aquí y cómo llenarlo

### `models.md` ⭐
Modelos de datos por cada microservicio.
**Llena:** diagrama ER (entidad-relación) o descripción de colecciones/tablas para cada servicio.

**Formato por servicio:**
```markdown
## Servicio: [nombre]
**Motor de BD:** [PostgreSQL / MongoDB / Redis / etc.]
**Justificación:** [por qué este motor para este servicio]

### Tabla/Colección: [nombre]
| Campo | Tipo | Nullable | Descripción | Restricciones |
|-------|------|----------|-------------|---------------|
| id | UUID | No | Identificador único | PK |
| [campo] | [tipo] | [Sí/No] | [descripción] | [FK/Unique/etc.] |

### Índices
| Nombre | Campos | Tipo | Justificación |
|--------|--------|------|---------------|
```

### `data-dictionary.md` ⭐
Significado exacto de cada campo importante en el sistema.
**Llena:** especialmente para campos que pueden ser ambiguos o tienen reglas de negocio.

**Formato:**
```markdown
| Campo | Servicio | Tabla | Tipo | Descripción detallada | Valores posibles |
|-------|---------|-------|------|----------------------|------------------|
| estado | scheduling | horario | ENUM | Estado actual del horario | ACTIVO, CANCELADO, PENDIENTE |
```

### `modeling-conventions.md`
Convenciones de nomenclatura y estilo para bases de datos del proyecto.
**Llena:** naming (snake_case o camelCase), uso de UUIDs vs secuenciales, timestamps estándar,
soft delete vs hard delete, auditoría (created_at, updated_at, created_by).

### `normalization-assessment.md`
Análisis del nivel de normalización y justificación de desnormalizaciones.
**Llena:** para cada desnormalización intencional, explicar por qué (rendimiento, simplificación).

### `migration-strategy.md`
Estrategia para migrar datos entre versiones del esquema.
**Llena:** herramienta de migraciones (Flyway, Liquibase, Alembic), política de rollback,
cómo manejar migraciones con datos en producción.

---

## Correlaciones con otras secciones

| Esta sección se alimenta de... | Y alimenta a... |
|-------------------------------|-----------------|
| `02-domain/entities-and-rules.md` → entidades del dominio | Tablas de la BD |
| `05-architecture/` → decisiones de motor de BD | Elección del motor en `models.md` |
| `models.md` | `07-api/contracts/` → qué datos expone cada servicio |
| `models.md` | `08-uml/` → diagramas ER |
| `models.md` | `09-microservices/[servicio]/data-model.md` |

---

## Decisiones importantes de datos en microservicios

### ¿SQL o NoSQL?
No hay respuesta única. Depende del servicio:
- **SQL** (PostgreSQL, MySQL): datos relacionales, transacciones ACID, esquema fijo
- **Documento** (MongoDB): datos jerárquicos, esquema flexible, alta variabilidad
- **Clave-valor** (Redis): caché, sesiones, datos temporales de alta velocidad
- **Series de tiempo** (InfluxDB, TimescaleDB): métricas, logs de eventos

### ¿Cómo manejar consistencia entre servicios?
Sin base de datos compartida, la consistencia es **eventual**:
- Saga Pattern: cadena de transacciones compensatorias
- Outbox Pattern: garantizar que el evento se publique junto con la transacción

---

## Preguntas que esta sección debe responder

- ¿Qué datos maneja cada microservicio?
- ¿Por qué se eligió ese motor de base de datos para cada servicio?
- ¿Cómo se actualiza el esquema sin romper el sistema?
- ¿Quién es el "dueño" de cada dato en el sistema?
