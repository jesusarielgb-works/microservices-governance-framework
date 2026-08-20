# Guía SDD — Software Design Documentation

> Este documento explica el **enfoque y la metodología** que rige todo este repositorio.
> Léelo primero si eres nuevo en el proyecto o en la metodología.

---

## ¿Qué es SDD?

**Software Design Documentation** es un enfoque de desarrollo donde la documentación de diseño
**precede y guía** la implementación. No es documentar lo que ya se construyó — es diseñar en
papel antes de escribir código.

```
Tradicional:   Código  →  Documentación (si se hace)
SDD:           Documentación  →  Código  →  Documentación actualizada
```

### Los 3 principios de SDD

1. **Design before code:** Un documento de diseño revisado y aprobado es el prerequisito para
   empezar a implementar. Si no está documentado, no existe aún.

2. **Living documentation:** La documentación se actualiza con cada cambio. Un documento
   desactualizado es técnicamente incorrecto — es código con bugs, pero en prosa.

3. **Traceability:** Cada línea de código tiene un requisito que la justifica. Cada requisito
   tiene un caso de prueba. Cada caso de prueba tiene un resultado.

---

## Flujo de trabajo SDD

```
┌─────────────────────────────────────────────────────────────────────┐
│  FASE 1: DESCUBRIMIENTO                                             │
│  01-context → 02-domain → 03-product                               │
│  Entiende el problema antes de proponer soluciones.                 │
│  Entregables: overview, problem-framing, domain-map, vision         │
└─────────────────────┬───────────────────────────────────────────────┘
                      │ ← Validación con stakeholders
┌─────────────────────▼───────────────────────────────────────────────┐
│  FASE 2: DEFINICIÓN                                                 │
│  04-requirements → 05-architecture → 06-data → 07-api              │
│  Define QUÉ y CÓMO antes de implementar.                           │
│  Entregables: HUs con ACs, ADRs, data models, contratos OpenAPI    │
└─────────────────────┬───────────────────────────────────────────────┘
                      │ ← Revisión de arquitectura (Architecture Review Board)
┌─────────────────────▼───────────────────────────────────────────────┐
│  FASE 3: DISEÑO DETALLADO                                           │
│  08-uml → 09-microservices → 12-ux-ui                              │
│  Diseña en detalle cada componente del sistema.                     │
│  Entregables: diagramas, runbooks, wireframes, design system        │
└─────────────────────┬───────────────────────────────────────────────┘
                      │ ← Kickoff de sprint (Sprint Planning)
┌─────────────────────▼───────────────────────────────────────────────┐
│  FASE 4: IMPLEMENTACIÓN (TDD)                                       │
│  Código guiado por los documentos de diseño y pruebas primero.     │
│  Entregables: código + pruebas + documentación actualizada         │
└─────────────────────┬───────────────────────────────────────────────┘
                      │ ← Code review + QA
┌─────────────────────▼───────────────────────────────────────────────┐
│  FASE 5: OPERACIÓN                                                  │
│  10-devops → 11-quality → 13-operations → 14-training              │
│  Despliega, monitorea y capacita.                                   │
│  Entregables: CI/CD, runbooks operativos, manuales                 │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Orden de llenado recomendado para un proyecto nuevo

### Semana 1 — Contexto y Dominio
1. `01-context/overview.md` — ¿Qué construimos?
2. `01-context/scope.md` — ¿Qué NO construimos?
3. `02-domain/domain-map.md` — ¿Cuáles son los bounded contexts?
4. `02-domain/entities-and-rules.md` — ¿Cuáles son las entidades y reglas?
5. `02-domain/domain-events.md` — ¿Qué eventos ocurren?
6. `01-context/glossary.md` — Primer borrador con 15-20 términos

### Semana 2 — Producto y Requisitos
7. `03-product/problem-framing.md` — Valida el problema
8. `03-product/vision.md` — Define la estrella del norte
9. `04-requirements/user-stories.md` — Primeras 10-15 HUs del MVP
10. `04-requirements/non-functional.md` — RNFs con métricas

### Semana 2-3 — Arquitectura
11. `05-architecture/overview.md` — Diagrama C4 y lista de servicios
12. `05-architecture/decisions/records/ADR-001-*.md` — Primera decisión arquitectónica
13. `06-data/models.md` — Esquema inicial de datos por servicio
14. `07-api/contracts/openapi/` — Contratos API (contract-first)

### Semana 3-4 — Diseño detallado
15. `09-microservices/service-catalog.md` — Catálogo completo de servicios
16. `09-microservices/services/01-[servicio]/` — README + data-model + events de cada servicio
17. `08-uml/diagrams/source/` — Diagrama de secuencia para flujos críticos
18. `12-ux-ui/navigation-map.md` + wireframes de pantallas principales

### Sprint 1 en adelante — Implementación con TDD
19. `10-devops/local-setup.md` — Primero antes de codificar
20. `11-quality/testing-strategy.md` — Antes del primer sprint
21. Implementación siguiendo el flujo TDD (ver `11-quality/tdd-guide.md`)
22. `13-operations/observability.md` — Antes del primer deploy

---

## Artefactos de revisión (gates)

| Gate | Cuándo | Qué se revisa | Quién aprueba |
|------|--------|---------------|---------------|
| **Domain Review** | Después de `02-domain/` | ¿Capturamos bien el dominio? | Domain Expert + Tech Lead |
| **Architecture Review** | Después de ADRs y `05-architecture/` | ¿La arquitectura cumple los RNFs? | Tech Lead + Equipo |
| **API Review** | Antes de implementar cada servicio | ¿El contrato es correcto y consistente? | Consumidores de la API |
| **Sprint Demo** | Al final de cada sprint | ¿El software cumple los ACs? | Product Owner |
| **Go/No-Go** | Antes de producción | ¿Cumple el DoD, DoR, RNFs? | Tech Lead + PO |

---

## La regla de los documentos vivos

```
Si el código cambió pero el documento no → El documento está ROTO
Si el documento dice X pero el código hace Y → El documento es MENTIRA
```

**Responsabilidad:** quien hace el PR que cambia el comportamiento del sistema
es responsable de actualizar la documentación correspondiente.
