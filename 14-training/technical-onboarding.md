# Onboarding Técnico

> Bienvenido al equipo. Este documento es tu guía de los primeros días.
> Objetivo: puedes hacer tu primer commit en 3 días.
> Si algo en este documento no está claro o está desactualizado, corrígelo tú mismo — ese es tu primer aporte.

---

## Día 1 — Setup y contexto

### Mañana: Accesos y ambiente

- [ ] Recibes acceso al repositorio (confirma con el Tech Lead)
- [ ] Recibes acceso a Slack / Teams (canal del equipo: `#[nombre-canal]`)
- [ ] Recibes acceso a Jira / Linear / GitHub Projects
- [ ] Configura tu entorno local siguiendo `10-devops/local-setup.md`
- [ ] Verifica que `curl http://localhost:8080/health` responde `{"status": "ok"}`

### Tarde: Leer documentación core

Lee en este orden — se construye sobre sí mismo:

1. `00-sdd-guide.md` — Cómo trabaja el equipo (30 min)
2. `01-context/overview.md` — Qué construimos (20 min)
3. `02-domain/domain-map.md` — El dominio del negocio (30 min)
4. `05-architecture/overview.md` — Cómo está construido (30 min)
5. `00-governance/git-conventions.md` — Cómo manejamos el código (20 min)

### Reuniones del Día 1

- [ ] Meet & greet con el equipo
- [ ] 1:1 con el Tech Lead (30 min) — contexto del proyecto y tus responsabilidades
- [ ] Demo del producto (si hay una grabación, mírala antes)

---

## Día 2 — Entender el dominio

### Leer documentación de dominio y requisitos

- [ ] `02-domain/entities-and-rules.md` — Las entidades y reglas de negocio
- [ ] `02-domain/domain-events.md` — Los eventos del sistema
- [ ] `04-requirements/user-stories.md` — Las HUs del sprint actual
- [ ] `01-context/glossary.md` — Los términos del proyecto

### Explorar el código

- [ ] Clona el repositorio del servicio con el que trabajarás
- [ ] Lee el `09-microservices/services/[tu-servicio]/README.md`
- [ ] Sigue la estructura de carpetas — debe coincidir con `05-architecture/hexagonal-architecture.md`
- [ ] Ejecuta los tests: `npm run test:unit` — todos deben estar en verde
- [ ] Ejecuta el servicio en local y prueba los endpoints principales

### Reunión del Día 2

- [ ] Sesión de domain walkthrough con el Tech Lead o un senior (1 hora)
  - Pide que te expliquen el flujo de negocio principal
  - Toma notas de términos que no conozcas — agrégalos al glosario

---

## Día 3 — Primera contribución

### Tu primera tarea

El Tech Lead te asignará una tarea pequeña etiquetada `good-first-issue`:
- Debe ser una corrección de bug pequeña o una mejora de documentación
- El objetivo es aprender el flujo de trabajo, no la complejidad de la tarea

### Flujo de trabajo TDD para tu primera tarea

1. Lee el ticket y los criterios de aceptación
2. Escribe el test que verifica el criterio (`🔴 RED`)
3. Implementa el código mínimo para que el test pase (`🟢 GREEN`)
4. Refactoriza si es necesario (`♻️ REFACTOR`)
5. Crea el PR siguiendo las convenciones de `00-governance/git-conventions.md`

### Checklist antes de abrir el PR

- [ ] `npm run test:unit` pasa (verde)
- [ ] `npm run lint` pasa (0 errores)
- [ ] El título del commit sigue `tipo(scope): descripción`
- [ ] La branch se llama `feat/HU-XXX-descripcion` o `fix/BUG-XXX-descripcion`

---

## Semana 1 — Profundizar

| Día | Actividad |
|-----|-----------|
| 4 | Revisión de código (participa en el code review del equipo — observa primero) |
| 5 | Participa en el Daily Standup con algo concreto que reportar |
| 5 | Lee `05-architecture/pattern-guide.md` — los patrones que usamos |
| 5 | Lee `11-quality/tdd-guide.md` y `11-quality/testing-strategy.md` completos |

---

## Semana 2 — Independencia guiada

- [ ] Completa tu primera HU de forma independiente
- [ ] Participa activamente en un code review
- [ ] Lee `07-api/README.md` y entiende un contrato OpenAPI de un servicio que usas
- [ ] Participa en la Retrospectiva del sprint

---

## Arquitectura: Los 5 conceptos que debes entender primero

Antes de escribir código, entiende estos 5 conceptos de la arquitectura del proyecto:

### 1. Los Bounded Contexts
Cada servicio corresponde a un Bounded Context del dominio.
Ver: `02-domain/domain-map.md`

### 2. La Arquitectura Hexagonal
El dominio es el centro. Nada del framework entra al dominio.
Ver: `05-architecture/hexagonal-architecture.md`

### 3. Los Puertos y Adaptadores
Las interfaces viven en el dominio. Las implementaciones en infraestructura.
Ver: `05-architecture/hexagonal-architecture.md`

### 4. El flujo de un request
```
HTTP Request
  → [Controller] (adapter in)
  → [UseCase] (application)
  → [Aggregate] (domain — aquí vive la lógica)
  → [Repository] (port → adapter out)
  → Base de datos
```

### 5. Los eventos de dominio
Los servicios se comunican por eventos, no por llamadas directas (cuando es posible).
Ver: `02-domain/domain-events.md`

---

## Preguntas frecuentes (FAQ)

**¿Por qué no uso `console.log` en los tests?**
Porque ensucia la salida. Usa `logger.debug()` del logger configurado.

**¿Puedo hacer un commit directamente a `main`?**
No. Todo va por PR con al menos 1 aprobación.

**¿Puedo cambiar el esquema de base de datos directamente?**
No. Todo cambio va como migración versionada. Ver `06-data/models.md`.

**¿Cómo sé si mi cambio en la API rompe a los consumidores?**
Corre los contract tests: `npm run test:contract`. Si pasan, el contrato se mantiene.

**¿Dónde pido ayuda si estoy bloqueado?**
1. Busca en esta documentación primero
2. Pregunta en `#[canal-del-equipo]`
3. No esperes más de 1 hora antes de pedir ayuda — el tiempo del equipo es valioso

**¿Qué hago si encuentro documentación incorrecta o desactualizada?**
Corrígela y abre un PR. La documentación es código.

---

## Recursos adicionales

| Recurso | URL / Ubicación | Para qué |
|---------|----------------|---------|
| Guía de Git | `00-governance/git-conventions.md` | Convenciones de commits y branches |
| ADRs del proyecto | `05-architecture/decisions/` | Entender las decisiones tomadas y por qué |
| Runbook del servicio | `09-microservices/services/[tu-servicio]/runbook.md` | Operar y troubleshootear |
| [Documentación externa] | [URL] | [para qué] |

---

## Correlaciones

- Setup técnico → `10-devops/local-setup.md`
- Proceso TDD → `11-quality/tdd-guide.md`
- Convenciones de código → `00-governance/git-conventions.md`
- Tu servicio → `09-microservices/services/[nombre]/`
