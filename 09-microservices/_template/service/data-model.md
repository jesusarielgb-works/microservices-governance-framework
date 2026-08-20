# Modelo de Datos — [Nombre del Servicio]

> Este servicio es el **dueño autoritativo** de los datos descritos aquí.
> Ningún otro servicio debe leer directamente estas tablas/colecciones.

---

## Motor de base de datos

**Motor:** [PostgreSQL / MongoDB / Redis / etc.]
**Justificación:** [por qué este motor es el adecuado para este servicio]

---

## Esquema

### Tabla/Colección: [nombre_tabla]

**Propósito:** [qué representa esta tabla]

| Campo | Tipo | Nullable | Descripción | Restricciones |
|-------|------|----------|-------------|---------------|
| id | UUID | No | Identificador único | PK, generado automáticamente |
| [campo] | [VARCHAR(255) / INT / BOOL / etc.] | [Sí/No] | [descripción] | [FK/Unique/Check/etc.] |
| created_at | TIMESTAMP | No | Fecha de creación | Default: NOW() |
| updated_at | TIMESTAMP | No | Última modificación | Actualizar en cada UPDATE |
| deleted_at | TIMESTAMP | Sí | Soft delete | NULL = activo |

**Relaciones:**
- `[campo_id]` → FK a `[tabla].[campo]` en [este servicio / servicio X via evento]

### Índices

| Nombre | Campos | Tipo | Justificación |
|--------|--------|------|---------------|
| idx_[tabla]_[campo] | [campo] | BTREE | Búsquedas frecuentes por [campo] |

---

## Decisiones de modelado

> Documenta aquí las decisiones que no son obvias — desnormalizaciones, campos que parecen redundantes, etc.

- **[Decisión]:** [por qué se hizo así]

---

## Migración de esquema

**Herramienta:** [Flyway / Liquibase / Alembic / knex]
**Ubicación de scripts:** `src/migrations/`
**Convención de nombres:** `V[NNN]__[descripcion].sql`

**Política de rollback:**
- [¿Se soporta rollback de migraciones? ¿Cómo?]
