# 11 — Calidad

> **¿Qué es esto?** Cómo el equipo garantiza que el sistema funciona correctamente
> y sigue funcionando mientras evoluciona. Pruebas, revisión de código y métricas.

## La pirámide de pruebas

```
          /\
         /  \
        / E2E \      ← Pocos, lentos, caros (pruebas de extremo a extremo)
       /--------\
      / Integra-  \   ← Algunos, moderados (prueban la interacción entre partes)
     /  ción       \
    /--------------\
   / Unitarias      \  ← Muchos, rápidos, baratos (prueban una función/clase)
  /------------------\
```

En microservicios, agrega una capa: **Pruebas de contrato** (contract testing).
Verifican que el consumidor y el proveedor de una API están de acuerdo en el contrato.

---

## Qué hay aquí y cómo llenarlo

### `testing-strategy.md` ⭐
La estrategia completa de pruebas del proyecto.
**Llena:** qué tipos de pruebas se hacen, con qué herramienta, cobertura mínima esperada,
cuáles van en el pipeline CI/CD, cuáles son manuales.

**Formato:**
```markdown
## Tipos de pruebas

### Unitarias
- **Herramienta:** [Jest / JUnit / pytest / etc.]
- **Cobertura mínima:** [80%]
- **Qué prueban:** funciones, clases, lógica de negocio en aislamiento
- **Qué NO prueban:** BD, red, servicios externos

### Integración
- **Herramienta:** [Testcontainers + JUnit / pytest]
- **Qué prueban:** servicio + BD real (en contenedor), sin servicios externos
- **Cuándo corren:** en cada PR

### Contrato (Contract Testing)
- **Herramienta:** [Pact / Spring Cloud Contract]
- **Qué prueban:** que el productor cumple lo que el consumidor espera
- **Cuándo corren:** en cada PR del productor y del consumidor

### E2E (Extremo a Extremo)
- **Herramienta:** [Playwright / Cypress / k6]
- **Qué prueban:** flujos completos del usuario con todo el sistema levantado
- **Cuándo corren:** antes de cada deploy a staging

## Cobertura mínima por servicio
| Servicio | Unitarias | Integración | Contrato |
|---------|-----------|-------------|---------|
| iam | 85% | Obligatorio | Obligatorio |
| [resto] | 80% | Obligatorio | Recomendado |
```

### `code-review.md`
Guía de revisión de código del equipo.
**Llena:** qué verificar en un PR (más allá de que "funciona"), lista de checks obligatorios,
cómo dar feedback constructivo, tiempo máximo para revisar un PR.

---

## Correlaciones con otras secciones

| Esta sección se alimenta de... | Y alimenta a... |
|-------------------------------|-----------------|
| `04-requirements/traceability-matrix.md` → qué probar | Casos de prueba que cubren cada RF |
| `07-api/contracts/` → contratos API | Pruebas de contrato |
| `00-governance/definition-of-done.md` → qué debe tener todo PR | Checklist de code review |
| `10-devops/ci-cd.md` | Qué pruebas van en qué paso del pipeline |

---

## Preguntas que esta sección debe responder

- ¿Qué tipos de pruebas escribe el equipo?
- ¿Cuánta cobertura de código se requiere?
- ¿Qué verifica un revisor en un Pull Request?
- ¿Cómo sabemos que un cambio no rompió nada?
