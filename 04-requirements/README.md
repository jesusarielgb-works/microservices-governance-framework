# 04 — Requisitos

> **¿Qué es esto?** La especificación formal de qué debe hacer el sistema.
> Funcionales: qué hace. No-funcionales: con qué calidad lo hace.

## Por qué existe esta sección

Los requisitos son el contrato entre el equipo y el cliente/stakeholder.
Sin ellos:
- No hay forma de verificar si el sistema está completo
- Los cambios de alcance no tienen baseline de comparación
- Las pruebas no tienen criterio de éxito

---

## Tipos de requisitos

### Funcionales (RF)
Describen **qué hace** el sistema: funciones, comportamientos, transformaciones de datos.
*Ejemplo: "El sistema debe permitir al usuario recuperar su contraseña mediante correo electrónico."*

### No funcionales (RNF)
Describen **cómo lo hace**: calidad, rendimiento, disponibilidad, seguridad.
*Ejemplo: "El sistema debe responder en menos de 200ms para el 95% de las solicitudes."*

Los RNF suelen ser más difíciles de cumplir que los RF y se ignoran con más frecuencia. **Son igual de importantes.**

---

## Qué hay aquí y cómo llenarlo

### `functional.md` ⭐
Lista de todos los requisitos funcionales del sistema.
**Llena:** numerados, con el módulo/servicio al que pertenecen, fuente (HU que los origina), prioridad.

**Formato:**
```markdown
| ID | Módulo | Descripción | Fuente (HU) | Prioridad |
|----|--------|-------------|-------------|-----------|
| RF-001 | [Servicio] | El sistema debe [hacer algo] | HU-XXX-001 | Alta |
```

### `non-functional.md` ⭐
Requisitos de calidad, rendimiento y restricciones técnicas.
**Llena:** por categoría (rendimiento, disponibilidad, seguridad, escalabilidad, etc.)

**Formato:**
```markdown
## Rendimiento
| ID | Requisito | Métrica | Cómo verificar |
|----|-----------|---------|----------------|
| RNF-001 | Tiempo de respuesta | p95 < 200ms | Load test con K6 |

## Disponibilidad
| ID | Requisito | Métrica | Cómo verificar |
|----|-----------|---------|----------------|
| RNF-010 | Uptime | 99.9% mensual | Monitoreo en producción |

## Seguridad
| ID | Requisito | Descripción |
|----|-----------|-------------|
| RNF-020 | Autenticación | JWT con expiración de 1 hora |
```

### `user-stories.md`
Historias de usuario formalizadas (vienen del backlog de `03-product/`).
**Llena:** con formato Como/Quiero/Para + criterios de aceptación verificables.

### `traceability-matrix.md` ⭐
Tabla que conecta: HU → Requisito → Caso de prueba.
**Llena:** cuando tienes requisitos y pruebas definidas. Permite verificar cobertura.

**Formato:**
```markdown
| HU | RF/RNF | Descripción | Caso de prueba | Estado |
|----|--------|-------------|----------------|--------|
| HU-IAM-001 | RF-001 | Login con email | TC-001 | ✅ |
```

### `_template-hu.md`
Template para una Historia de Usuario completa con criterios de aceptación.

### `_template-nfr.md`
Template para especificar requisitos no funcionales con sus métricas de verificación.

---

## Correlaciones con otras secciones

| Esta sección alimenta... | Por qué |
|--------------------------|---------|
| `05-architecture/` | Los RNF de rendimiento/disponibilidad guían decisiones de arquitectura |
| `11-quality/testing-strategy.md` | Cada RF debe tener al menos un caso de prueba |
| `09-microservices/` | Los RF se agrupan por servicio responsable |
| `07-api/` | Los RF de integración → endpoints en los contratos API |
| `15-project-control/risks.md` | RNF muy exigentes suelen generar riesgos técnicos |

---

## Errores comunes a evitar

❌ **"El sistema debe ser rápido"** → No medible. Mejor: "p95 < 200ms"

❌ **"El sistema debe ser seguro"** → No verificable. Mejor: "Autenticación con JWT, tokens expiran en 1h"

❌ Escribir requisitos que describen la solución en lugar del problema.

✅ Un buen requisito es: **específico, medible, alcanzable, relevante y verificable**.

---

## Preguntas que esta sección debe responder

- ¿Qué debe hacer el sistema para cada tipo de usuario?
- ¿Con qué velocidad, disponibilidad y seguridad?
- ¿Qué requisito origina cada caso de prueba?
- ¿Están todos los requisitos cubiertos por pruebas?
