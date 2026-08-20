# Modelos de Datos por Servicio

> **Qué llenar aquí:** El esquema de datos de cada microservicio.
> Cada servicio tiene su propia sección. Recuerda: **cada servicio tiene su propia base de datos**.
> Los cambios de esquema siempre se hacen con migraciones versionadas, nunca modificando tablas en caliente.

> **Nota de motor de BD:** Este documento es agnóstico de tecnología. Los ejemplos muestran SQL
> estándar compatible con la mayoría de motores relacionales. Para bases de datos documentales
> (MongoDB) o de clave-valor (Redis), adapta los diagramas y esquemas al formato correspondiente.
> La elección del motor se documenta en cada sección de servicio — el scaffold no asume cuál usar.

---

## Principios de modelado de datos

### 1. Database per Service (obligatorio)
Ningún servicio accede directamente a la base de datos de otro.
La comunicación entre servicios es siempre por API o eventos.

```
✓ Servicio A → BD A (PostgreSQL)
✓ Servicio B → BD B (MongoDB)
✗ Servicio A → JOIN con tablas del Servicio B
```

### 2. Campos de auditoría estándar
Todas las tablas incluyen:

```sql
id          UUID        PRIMARY KEY  DEFAULT gen_random_uuid(),
created_at  TIMESTAMPTZ NOT NULL     DEFAULT NOW(),
updated_at  TIMESTAMPTZ NOT NULL     DEFAULT NOW(),
deleted_at  TIMESTAMPTZ              -- NULL = activo (soft delete)
```

### 3. Soft delete por defecto
No borres registros con DELETE físico. Usa `deleted_at IS NOT NULL` para marcar como eliminado.
Esto facilita la auditoría y la recuperación.

### 4. Naming conventions

```sql
-- Tablas:      snake_case, plural              → pedidos, items_pedido, usuarios
-- Columnas:    snake_case, descriptivo         → precio_unitario, fecha_entrega
-- FKs:         [tabla_referenciada]_id         → cliente_id, producto_id
-- Índices:     idx_[tabla]_[columna(s)]        → idx_pedidos_cliente_id
-- Timestamps:  siempre con zona horaria (TIMESTAMPTZ, no TIMESTAMP)
```

---

## Servicio: [nombre-servicio]

**Motor de BD:** PostgreSQL 15 / MongoDB 7 / Redis 7 — [justificación de la elección]

**Justificación del motor:**
- [Por qué este motor para este servicio. Ej: "PostgreSQL por soporte ACID en transacciones financieras"]
- [Qué funcionalidades del motor se usan: JSONB, full-text search, geo, etc.]

### Tabla: [nombre_tabla]

**Propósito:** [Qué registra esta tabla]

```sql
CREATE TABLE [nombre_tabla] (
  id              UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
  
  -- Campos del negocio
  [campo1]        [TIPO]      NOT NULL,
  [campo2]        [TIPO],
  [campo3]        [TIPO]      NOT NULL DEFAULT [valor],
  
  -- Relaciones
  [referencia]_id UUID        REFERENCES [tabla_referenciada](id) ON DELETE RESTRICT,
  
  -- Campos de estado
  estado          VARCHAR(50) NOT NULL DEFAULT 'ACTIVO'
                  CHECK (estado IN ('ACTIVO', 'INACTIVO', 'CANCELADO')),
  
  -- Auditoría (en todas las tablas)
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at      TIMESTAMPTZ,
  created_by      UUID,
  updated_by      UUID
);

-- Índices
CREATE INDEX idx_[tabla]_[campo] ON [nombre_tabla] ([campo]);
CREATE INDEX idx_[tabla]_deleted ON [nombre_tabla] (deleted_at) WHERE deleted_at IS NULL;
-- Para búsquedas frecuentes:
CREATE INDEX idx_[tabla]_[campo_busqueda] ON [nombre_tabla] ([campo_busqueda]);
```

**Diccionario de datos:**

| Columna | Tipo | Descripción | Ejemplo |
|---------|------|-------------|---------|
| id | UUID | Identificador único autogenerado | `550e8400-...` |
| [campo1] | [TIPO] | [Descripción de negocio] | [Ejemplo] |
| estado | VARCHAR(50) | Estado del ciclo de vida | `ACTIVO` |
| created_at | TIMESTAMPTZ | Cuándo se creó el registro | `2024-01-15T10:30:00Z` |
| deleted_at | TIMESTAMPTZ | NULL = activo; con valor = eliminado | `NULL` |

**Decisiones de modelado:**
1. [Por qué el campo X es NOT NULL y no nullable]
2. [Por qué se usa soft delete en vez de hard delete]
3. [Por qué el campo de estado tiene ese conjunto de valores]

---

### Tabla: [tabla_relacionada]

```sql
CREATE TABLE [tabla_relacionada] (
  id              UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
  [principal]_id  UUID        NOT NULL REFERENCES [tabla_principal](id) ON DELETE CASCADE,
  
  -- Campos
  [campo]         [TIPO]      NOT NULL,
  
  -- Auditoría
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at      TIMESTAMPTZ
);

CREATE INDEX idx_[tabla_rel]_[principal]_id ON [tabla_relacionada] ([principal]_id);
```

---

## Estrategia de migraciones

**Herramienta:** [Flyway / Liquibase / Prisma Migrate / TypeORM Migrations]

**Convención de nombre de archivos:**

```
V{número_versión}__{descripcion_snake_case}.sql

Ejemplos:
  V001__create_pedidos_table.sql
  V002__add_estado_to_pedidos.sql
  V003__create_index_pedidos_cliente_id.sql
```

**Reglas de migraciones:**

```
✓ Las migraciones son SIEMPRE hacia adelante (forward-only)
✓ Una migración por cambio lógico
✓ Los datos de seed van en migraciones separadas con prefijo S: S001__seed_...
✗ Nunca modifiques una migración ya ejecutada en cualquier ambiente
✗ Nunca hagas DROP COLUMN / DROP TABLE en una migración si hay código en producción que la usa
    (proceso: 1-deprecar en código → 2-migración de limpieza en el siguiente release)
```

**Cambios de esquema compatibles (no rompen):**

```sql
-- Agregar columna nullable → siempre seguro
ALTER TABLE pedidos ADD COLUMN notas TEXT;

-- Agregar columna NOT NULL con DEFAULT → seguro si el DEFAULT es válido
ALTER TABLE pedidos ADD COLUMN prioridad VARCHAR(20) NOT NULL DEFAULT 'NORMAL';

-- Crear nuevo índice → seguro (en producción usar CONCURRENTLY)
CREATE INDEX CONCURRENTLY idx_pedidos_fecha ON pedidos (created_at);
```

**Cambios incompatibles (requieren migración en 2 fases):**

```sql
-- Renombrar columna → 2 fases:
-- Fase 1 (release N): Agregar nueva columna, copiar datos, actualizar código para usar ambas
ALTER TABLE pedidos ADD COLUMN fecha_entrega TIMESTAMPTZ;
UPDATE pedidos SET fecha_entrega = delivery_date;

-- Fase 2 (release N+1): Eliminar columna antigua (código ya no la usa)
ALTER TABLE pedidos DROP COLUMN delivery_date;
```

---

## Guía de selección de motor de BD

| Motor | Usar cuando... | Evitar cuando... |
|-------|---------------|-----------------|
| **PostgreSQL** | Transacciones ACID, relaciones complejas, JSONB, full-text | Documentos muy anidados, grafos |
| **MongoDB** | Documentos flexibles, catálogos de productos, catálogos | Transacciones complejas entre colecciones |
| **Redis** | Caché, sesiones, colas ligeras, contadores | Fuente de verdad, datos críticos |
| **Elasticsearch** | Búsqueda full-text, análisis, logs | Fuente de verdad (es un índice, no una BD) |
| **InfluxDB / TimescaleDB** | Series de tiempo, métricas, IoT | Datos de negocio transaccionales |

---

## Diagrama de relaciones (por servicio)

```
Reemplaza con un diagrama ER del servicio usando notación de tu herramienta preferida.

Ejemplo Mermaid:
\```mermaid
erDiagram
    PEDIDOS ||--o{ ITEMS_PEDIDO : contiene
    PEDIDOS {
        uuid id PK
        uuid cliente_id FK
        varchar estado
        decimal total
        timestamptz created_at
    }
    ITEMS_PEDIDO {
        uuid id PK
        uuid pedido_id FK
        uuid producto_id FK
        integer cantidad
        decimal precio_unitario
    }
\```
```

---

## Correlaciones

- Entidades del dominio que mapean a estas tablas → `02-domain/entities-and-rules.md`
- Saga y Outbox pattern para consistencia distribuida → `05-architecture/pattern-guide.md`
- Datos de cada servicio en detalle → `09-microservices/services/XX/data-model.md`
- Cómo se accede a los datos vía API → `07-api/contracts/openapi/`
