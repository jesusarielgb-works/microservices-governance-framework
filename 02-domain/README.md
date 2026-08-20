# 02 — Dominio del Problema

> **¿Qué es esto?** El modelo mental del negocio. No es tecnología — es entender el problema
> que el sistema resuelve antes de escribir código. Esta sección viene de Domain-Driven Design (DDD).

## Por qué existe esta sección

Los errores más costosos en software no son bugs — son malentendidos del dominio.
Cuando los desarrolladores no entienden profundamente el negocio:
- Crean abstracciones incorrectas que hay que reescribir
- Los nombres en el código no coinciden con los del negocio → confusión permanente
- Los límites entre microservicios quedan mal trazados

Esta sección captura el conocimiento del dominio **antes** de diseñar la arquitectura.

---

## Conceptos clave que debes conocer

**Entidad:** Objeto del dominio con identidad única (ej: un `Estudiante` identificado por su código).

**Value Object:** Objeto sin identidad propia, se define por sus atributos (ej: `Dirección`, `Precio`).

**Agregado (Aggregate):** Grupo de entidades que se tratan como una unidad. Solo el raíz del agregado
puede ser referenciado desde afuera.

**Evento de dominio:** Algo que ocurrió en el negocio y que otros partes del sistema deben saber
(ej: `EstudianteMatriculado`, `PagoAprobado`). Son hechos, van en pasado.

**Contexto delimitado (Bounded Context):** Área del sistema donde un modelo particular aplica.
Cada microservicio generalmente corresponde a un bounded context.

---

## Qué hay aquí y cómo llenarlo

### `domain-map.md` ⭐
Mapa de todos los bounded contexts y cómo se relacionan.
**Llena:** dibuja los contextos como rectángulos y las relaciones entre ellos
(upstream/downstream, shared kernel, anti-corruption layer).

**Formato:**
```markdown
## Bounded Contexts

### [Nombre del Contexto]
**Responsabilidad:** [qué maneja este contexto]
**Entidades principales:** [lista]
**Equipo dueño:** [equipo]

## Mapa de relaciones
[Diagrama ASCII o descripción de cómo se relacionan los contextos]

| Contexto A | Relación | Contexto B | Descripción |
|------------|----------|------------|-------------|
| [A] | downstream-of | [B] | [A] consume eventos de [B] |
```

### `entities-and-rules.md` ⭐
Catálogo de entidades, value objects y reglas de negocio.
**Llena:** para cada entidad: nombre, atributos, invariantes (reglas que SIEMPRE deben cumplirse),
comportamientos.

**Formato:**
```markdown
## Entidad: [NombreEntidad]
**Pertenece a:** [Bounded Context]
**Identificador:** [campo que la hace única]

### Atributos
| Atributo | Tipo | Descripción | Restricciones |
|----------|------|-------------|---------------|

### Reglas de negocio (invariantes)
- [ ] [Una regla que siempre debe cumplirse]

### Comportamientos (métodos de dominio)
- `[nombre()]`: [qué hace]
```

### `domain-events.md` ⭐
Lista de todos los eventos que ocurren en el dominio.
**Llena:** nombre del evento (pasado), qué lo dispara, qué datos lleva, quién lo consume.

**Formato:**
```markdown
| Evento | Disparado por | Datos | Consumidores | Bounded Context |
|--------|--------------|-------|--------------|-----------------|
| [NombreEnPasado] | [acción que lo causa] | [campos del evento] | [qué escucha esto] | [contexto] |
```

---

## Correlaciones con otras secciones

| Esta sección alimenta... | Por qué |
|--------------------------|---------|
| `05-architecture/overview.md` | Los bounded contexts → microservicios |
| `06-data/models.md` | Las entidades → tablas/colecciones de datos |
| `07-api/contracts/` | Los eventos de dominio → eventos en APIs asíncronas |
| `09-microservices/event-catalog.md` | Todos los eventos de dominio se registran ahí |
| `04-requirements/user-stories.md` | Las reglas de negocio → criterios de aceptación |

---

## Herramienta recomendada: Event Storming

**Event Storming** es un taller de descubrimiento del dominio con post-its:
1. 🟠 Naranja: Eventos de dominio (pasado)
2. 🔵 Azul: Comandos (lo que dispara el evento)
3. 🟡 Amarillo: Actores (quién ejecuta el comando)
4. 🟣 Morado: Políticas (reacciones automáticas)
5. 🟦 Celeste: Sistemas externos

Hacer un Event Storming con el equipo antes de llenar esta sección ahorra semanas de rediseño.

---

## Preguntas que esta sección debe responder

- ¿Cuáles son las entidades principales del negocio?
- ¿Qué reglas NUNCA se pueden violar en el sistema?
- ¿Qué eventos importantes ocurren en el dominio?
- ¿Dónde están los límites naturales del sistema (para definir microservicios)?
