# Gobernanza de Proyectos de Microservicios

> Plantilla genérica · Versión 1.0 · CORHUILA — Sistemas Distribuidos / Diseño de Software

Este repositorio es un **cascarón de documentación** para cualquier proyecto de microservicios.
No contiene información de ningún proyecto específico. Su propósito es **enseñar** cómo documentar
un sistema de software de manera profesional, mostrando qué va en cada sección, por qué importa
y cómo cada documento se relaciona con los demás.

---

## Cómo usar este cascarón

1. **Haz un fork** (o copia la estructura) para tu proyecto.
2. **Llena los archivos** comenzando por las secciones marcadas como prioritarias (⭐).
3. **Lee cada `README.md`** antes de llenar los documentos de esa sección — explican qué se espera.
4. **No borres las instrucciones** hasta terminar de llenar un documento.
5. Cuando un documento esté completo y revisado, elimina el bloque `> [!NOTE] INSTRUCCIONES`.

---

## Mapa de secciones y correlaciones

```
┌─────────────────────────────────────────────────────────────────┐
│                    FLUJO DE CONSTRUCCIÓN                        │
│                                                                 │
│  01-context ──► 02-domain ──► 03-product ──► 04-requirements   │
│      │               │              │              │            │
│      └───────────────┴──────────────┴──────────────▼           │
│                                               05-architecture   │
│                                                    │            │
│                         ┌──────────────────────────┤            │
│                         ▼                          ▼            │
│                      06-data                   07-api           │
│                         │                          │            │
│                         └──────────────┬───────────┘            │
│                                        ▼                        │
│                                 09-microservices                │
│                                        │                        │
│                    ┌───────────────────┼──────────────┐         │
│                    ▼                   ▼              ▼         │
│                08-uml             10-devops       12-ux-ui      │
│                                        │                        │
│                    ┌───────────────────┤                        │
│                    ▼                   ▼                        │
│               11-quality          13-operations                 │
│                                        │                        │
│                    ┌───────────────────┤                        │
│                    ▼                   ▼                        │
│               14-training         15-project-control            │
└─────────────────────────────────────────────────────────────────┘

Gobernanza (00-governance) aplica a TODO el proyecto
```

---

## Índice de secciones

| # | Carpeta | Propósito | Prioridad |
|---|---------|-----------|-----------|
| 00 | [00-governance](./00-governance/README.md) | Reglas del equipo: Git, naming, Definition of Done/Ready, seguridad | ⭐ Primero |
| 01 | [01-context](./01-context/README.md) | Por qué existe el sistema: visión, alcance, glosario | ⭐ Primero |
| 02 | [02-domain](./02-domain/README.md) | El problema de negocio: entidades, reglas y eventos del dominio | ⭐ Primero |
| 03 | [03-product](./03-product/README.md) | Qué se va a construir: PRD, visión de producto, backlog inicial | ⭐ Primero |
| 04 | [04-requirements](./04-requirements/README.md) | Qué debe hacer el sistema: HUs funcionales y no-funcionales | ⭐ Primero |
| 05 | [05-architecture](./05-architecture/README.md) | Cómo está organizado el sistema: ADRs, deployment, patrones | 🔵 Diseño |
| 06 | [06-data](./06-data/README.md) | Cómo se almacenan los datos: modelos, diccionario, migración | 🔵 Diseño |
| 07 | [07-api](./07-api/README.md) | Contratos entre servicios: OpenAPI, autenticación, guías REST | 🔵 Diseño |
| 08 | [08-uml](./08-uml/README.md) | Diagramas: clases, secuencia, componentes, ER | 🔵 Diseño |
| 09 | [09-microservices](./09-microservices/README.md) | Cada microservicio documentado individualmente | 🟢 Impl. |
| 10 | [10-devops](./10-devops/README.md) | CI/CD, ambientes, setup local, release process | 🟢 Impl. |
| 11 | [11-quality](./11-quality/README.md) | Estrategia de pruebas, revisión de código, métricas | 🟢 Impl. |
| 12 | [12-ux-ui](./12-ux-ui/README.md) | Diseño de interfaz: design system, flujos, wireframes | 🔵 Diseño |
| 13 | [13-operations](./13-operations/README.md) | Operación en producción: observabilidad, incidentes, SLA/SLO | 🟠 Ops |
| 14 | [14-training](./14-training/README.md) | Manuales de usuario, admin y onboarding técnico | 🟠 Ops |
| 15 | [15-project-control](./15-project-control/README.md) | Riesgos, dependencias, preguntas abiertas, backlog técnico | 🔵 Diseño |
| 99 | [99-archive](./99-archive/README.md) | Decisiones obsoletas y documentos deprecados | — |

---

## Regla de oro de la documentación

> **Un documento que nadie lee es un documento que no existe.**
>
> Antes de crear un documento pregúntate: ¿quién lo va a leer? ¿cuándo? ¿qué decisión
> les ayuda a tomar? Si no puedes responder esas tres preguntas, no lo crees todavía.

---

## Correlaciones clave que debes conocer

| Si cambias... | Debes revisar también... |
|---------------|--------------------------|
| El alcance (01-context) | PRD (03), requirements (04), architecture overview (05) |
| Una entidad del dominio (02) | Modelos de datos (06), API contracts (07), diagramas UML (08) |
| Un requisito funcional (04) | Criterios de aceptación, casos de prueba (11), HUs (03) |
| La arquitectura (05) | ADRs (05/decisions), cada microservicio afectado (09) |
| Un modelo de datos (06) | API contract del servicio dueño (07, 09), diagramas ER (08) |
| Un contrato API (07) | Microservicio dueño (09), clientes del contrato (09) |
| Un microservicio (09) | dependency-map (09), event-catalog (09), data-ownership-matrix (09) |
| El pipeline CI/CD (10) | Release checklist (10), ambientes (10) |

---

## Convenciones de nombres

- **Archivos de contenido:** `kebab-case.md`
- **Templates:** `_template-nombre.md` (prefijo `_` para que aparezcan primero)
- **ADRs:** `ADR-NNN-titulo-corto.md` (numerados secuencialmente)
- **Contratos OpenAPI:** `nombre-servicio.yaml`

---

## Guías metodológicas incluidas

Este cascarón incluye documentación completa sobre las siguientes metodologías:

| Metodología | Documento principal | Sección |
|-------------|---------------------|---------|
| **SDD** (Software Design Documentation) | [`00-sdd-guide.md`](./00-sdd-guide.md) | Raíz |
| **DDD** (Domain-Driven Design) | [`02-domain/domain-map.md`](./02-domain/domain-map.md) | 02-domain |
| DDD — Entidades, VOs, Aggregates | [`02-domain/entities-and-rules.md`](./02-domain/entities-and-rules.md) | 02-domain |
| DDD — Eventos de dominio | [`02-domain/domain-events.md`](./02-domain/domain-events.md) | 02-domain |
| **Hexagonal Architecture** | [`05-architecture/hexagonal-architecture.md`](./05-architecture/hexagonal-architecture.md) | 05-architecture |
| **Patrones** (GoF + Microservicios) | [`05-architecture/pattern-guide.md`](./05-architecture/pattern-guide.md) | 05-architecture |
| **TDD** (Test-Driven Development) | [`11-quality/tdd-guide.md`](./11-quality/tdd-guide.md) | 11-quality |
| TDD — Estrategia y pirámide | [`11-quality/testing-strategy.md`](./11-quality/testing-strategy.md) | 11-quality |

---

## Recursos de referencia

- [adr.github.io](https://adr.github.io/) — Más sobre Architecture Decision Records
- [12factor.net](https://12factor.net/) — Buenas prácticas para microservicios
- [OpenAPI Specification](https://swagger.io/specification/) — Estándar para contratos REST
- [C4 Model](https://c4model.com/) — Diagramas de arquitectura (System, Container, Component, Code)
- [Domain-Driven Design Reference](https://www.domainlanguage.com/ddd/reference/) — Eric Evans
- [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/) — Alistair Cockburn
- [Test Driven Development](https://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530) — Kent Beck
