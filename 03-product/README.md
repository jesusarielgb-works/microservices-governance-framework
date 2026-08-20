# 03 — Definición de Producto

> **¿Qué es esto?** La respuesta a "¿qué vamos a construir?". No es "cómo" — eso viene en
> arquitectura. Aquí se define el problema validado, la visión del producto y el plan de construcción.

## Por qué existe esta sección

Sin una definición clara de producto:
- El equipo construye features que nadie pidió
- El alcance crece sin control (scope creep)
- No hay forma de saber si el proyecto fue exitoso

Esta sección es el contrato entre el equipo y los stakeholders sobre **qué se va a construir y por qué**.

---

## Qué hay aquí y cómo llenarlo

### `problem-framing.md` ⭐ (Empezar aquí)
Articula el problema antes de proponer soluciones.
**Llena:** quién tiene el problema, qué dolor exactamente, evidencia del problema, cómo lo resuelven hoy.

**Formato:**
```markdown
## El problema
**¿Quién lo tiene?** [Perfil del usuario afectado]
**¿Qué problema tienen?** [Descripción del dolor, específica]
**¿Cuándo ocurre?** [Situación que desencadena el problema]
**¿Cuál es el impacto?** [Consecuencia concreta: tiempo, dinero, frustración]
**¿Cómo lo resuelven hoy?** [Workaround actual y por qué es insuficiente]

## Por qué vale la pena resolverlo
[Justificación del valor de construir este sistema]
```

### `discovery-brief.md`
Hallazgos de la investigación con usuarios.
**Llena:** entrevistas realizadas, insights encontrados, supuestos validados e invalidados.

### `vision.md` ⭐
La estrella del norte del producto en 1-2 oraciones.
**Llena:** formato "Para [usuario], que [necesidad], [nombre del sistema] es [tipo de producto]
que [beneficio clave]. A diferencia de [alternativa], nuestro producto [diferenciador]."

### `roadmap.md`
Plan de entregas en el tiempo.
**Llena:** hitos por trimestre/sprint, qué features entran en cada fase.

**Formato:**
```markdown
## Fase 1 — MVP (Sprint 1-3)
- [Feature crítica 1]
- [Feature crítica 2]

## Fase 2 — Iteración (Sprint 4-6)
- [Mejoras basadas en feedback]
```

### `product-backlog.md` ⭐
Lista priorizada de todo lo que debe construirse.
**Llena:** usando el template `_template-backlog.md`. Ordena por valor al usuario.

### `_template-prd.md`
Product Requirements Document completo.
**Usa cuando:** necesitas formalizar los requisitos para un stakeholder externo o entrega académica.

### `_template-discovery-brief.md`
Template para documentar investigación con usuarios.

### `_template-problem-framing.md`
Template estructurado para enmarcar el problema.

### `_template-backlog.md`
Template para las historias de usuario del backlog inicial.

---

## Formato de Historia de Usuario

```markdown
## HU-[SERVICIO]-[NNN]: [Título]
**Como** [rol del usuario]
**Quiero** [acción que quiere realizar]
**Para** [beneficio que obtiene]

### Criterios de aceptación
- [ ] AC1: Dado [contexto], cuando [acción], entonces [resultado esperado]
- [ ] AC2: ...

### Notas técnicas
[Restricciones o consideraciones de implementación]

**Estimación:** [SP]  **Prioridad:** [Alta/Media/Baja]
```

---

## Correlaciones con otras secciones

| Esta sección alimenta... | Por qué |
|--------------------------|---------|
| `04-requirements/` | Las HUs del backlog se formalizan como requisitos |
| `02-domain/` | El problem framing revela entidades del dominio |
| `15-project-control/technical-backlog.md` | Deuda técnica identificada durante la definición |

---

## Preguntas que esta sección debe responder

- ¿Qué problema exactamente resolvemos?
- ¿Cómo luce el éxito del producto?
- ¿Qué construimos primero y por qué?
- ¿Qué NO construimos en este ciclo?
