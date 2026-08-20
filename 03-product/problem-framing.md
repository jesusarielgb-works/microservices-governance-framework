# Problem Framing — Definición del Problema

> **Por qué este documento existe:** Antes de diseñar soluciones, el equipo debe estar
> alineado en el problema que resuelve. Este documento captura ese alineamiento.
> Un problema bien definido ya lleva el 50% de la solución.

---

## 1. El problema en una oración

> Completa esta plantilla:

**[Segmento de usuarios]** que **[contexto de uso]** tiene dificultades con **[dolor/problema]**
porque **[causa raíz]**, lo que resulta en **[impacto cuantificable]**.

**Ejemplo:**
> Los **operadores de inventario de medianas empresas** que **gestionan catálogos de más de 500 productos**
> tienen dificultades con **el control de stock en múltiples bodegas** porque **los sistemas actuales
> no permiten sincronización en tiempo real**, lo que resulta en **15% de pedidos con error de stock
> y 3 horas de trabajo manual de corrección por día**.

---

## 2. Usuarios afectados

| Segmento | Descripción | Tamaño estimado | Prioridad |
|---------|-------------|----------------|-----------|
| [Segmento A] | [Quiénes son, qué hacen] | [N usuarios] | Alta |
| [Segmento B] | [Quiénes son] | [N usuarios] | Media |

### Jobs-to-be-done (JTBD)

> ¿Qué trabajo trata de hacer el usuario cuando "contrata" nuestro producto?

**Cuando** [situación / contexto],
**quiero** [motivación / lo que buscan lograr],
**para** [resultado esperado / beneficio].

---

## 3. Evidencia del problema

> El problema debe ser real. Documenta la evidencia que tienes.

| Tipo de evidencia | Fuente | Fecha | Hallazgo clave |
|-------------------|--------|-------|----------------|
| Entrevistas de usuario | [N] entrevistas con [perfil] | [fecha] | [qué dijeron] |
| Datos de soporte | Tickets de soporte | [período] | [% de tickets sobre este tema] |
| Benchmarking | [Competidores / mercado] | [fecha] | [cómo lo resuelven otros] |
| Observación directa | [Shadowing / field research] | [fecha] | [qué observaron] |

---

## 4. Solución actual del usuario (y sus problemas)

> ¿Cómo resuelve el usuario el problema hoy?

| Solución actual | Limitaciones | Costo/Fricción |
|----------------|-------------|----------------|
| [Excel / proceso manual] | [No escala, errores, lento] | [X horas/día] |
| [Sistema legado] | [No tiene API, sin integración] | [Y errores/semana] |

---

## 5. Hipótesis de solución

> Este es el primer borrador de la dirección de solución. No es un compromiso.

**Creemos que** [describir la solución de alto nivel]
**para** [el segmento de usuarios],
**logrará** [el beneficio esperado].
**Sabremos que tuvimos éxito cuando** [métrica específica].

---

## 6. Métricas de éxito (North Star)

| Métrica | Baseline actual | Objetivo a 6 meses | Cómo medirla |
|---------|----------------|-------------------|--------------|
| [Métrica de negocio 1] | [valor actual] | [valor objetivo] | [instrumento] |
| [Métrica de adopción] | [valor actual] | [valor objetivo] | [instrumento] |

**North Star Metric:** [La única métrica que mejor captura el valor entregado]

---

## 7. Riesgos de la hipótesis

| Riesgo | Probabilidad | Impacto | Experimento para validar |
|--------|-------------|---------|--------------------------|
| [Los usuarios no adoptarán el cambio] | Alta | Alto | [Piloto con N usuarios] |
| [El problema no es tan frecuente como creemos] | Media | Alto | [Análisis de logs de soporte] |

---

## 8. Fuera de alcance (no resolvemos)

> Define explícitamente qué problemas relacionados NO resuelves en esta versión.
> Esto evita scope creep.

- [Problema relacionado pero fuera de alcance: por qué]
- [Funcionalidad que los usuarios piden pero no vamos a hacer ahora: razón]

---

## Correlaciones

- Visión del producto → `03-product/vision.md`
- HUs que implementan esta solución → `04-requirements/user-stories.md`
- KPIs detallados → `13-operations/README.md`
