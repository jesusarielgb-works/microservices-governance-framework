# Alcance del Sistema

> **Por qué este documento existe:** El alcance previene el scope creep y alinea las expectativas.
> Es igual de importante definir lo que el sistema NO hace como lo que sí hace.
> Revisa este documento al inicio de cada ciclo de planificación.

---

## En alcance (In Scope)

Lo que el sistema **SÍ construye y mantiene**:

### Funcionalidades del MVP

| # | Funcionalidad | Descripción | Servicio responsable |
|---|--------------|-------------|---------------------|
| 1 | [Funcionalidad A] | [Descripción breve] | [nombre-servicio] |
| 2 | [Funcionalidad B] | [Descripción breve] | [nombre-servicio] |
| 3 | [Funcionalidad C] | [Descripción breve] | [nombre-servicio] |

### Integraciones incluidas

| Sistema externo | Tipo de integración | Para qué |
|----------------|--------------------|-|
| [Sistema A] | REST API / Webhook / SDK | [propósito] |
| [Sistema B] | SFTP / Base de datos | [propósito] |

### Ambientes que se construyen

| Ambiente | Propósito |
|---------|-----------|
| Local | Desarrollo en máquina del desarrollador |
| Development (dev) | Integración continua y pruebas de desarrollo |
| Staging | Pre-producción, pruebas de aceptación del PO |
| Production | Ambiente productivo |

---

## Fuera de alcance (Out of Scope)

Lo que el sistema **NO construye** en esta versión y por qué:

| # | Qué está fuera | Razón | ¿Versión futura? |
|---|---------------|-------|-----------------|
| 1 | [Funcionalidad X] | [Fuera del presupuesto / No es MVP / Usa sistema externo] | Sí — H2 2024 |
| 2 | [Integración con Y] | [El proveedor no tiene API pública aún] | Pendiente de proveedor |
| 3 | [Módulo Z] | [Otro equipo lo construye] | N/A |

### Qué hace otro sistema / equipo (y por qué no nosotros)

| Funcionalidad | Quién la construye | Por qué no nosotros |
|--------------|-------------------|--------------------:|
| [Autenticación SSO] | Equipo IAM central | Reusar la implementación existente |
| [Reportes financieros] | Sistema de BI / Equipo Analytics | Fuera del core domain |

---

## Supuestos del alcance

> Estos supuestos se asumen verdaderos. Si cambian, el alcance debe renegociarse.

| # | Supuesto | Consecuencia si es falso |
|---|----------|--------------------------|
| 1 | El sistema externo [X] tiene una API REST disponible | Tendríamos que construir la integración diferente |
| 2 | Los usuarios manejan [idioma / dispositivo / etc.] | El diseño UX cambiaría |
| 3 | El volumen de datos inicial es < [N] registros | La estrategia de base de datos podría cambiar |

---

## Restricciones

| Tipo | Descripción |
|------|-------------|
| **Tiempo** | [El MVP debe estar listo en X semanas / para fecha Y] |
| **Presupuesto** | [N horas de desarrollo / X USD de infraestructura] |
| **Tecnología** | [Debe usar el stack corporativo: Java + PostgreSQL] |
| **Regulatorio** | [Debe cumplir con X regulación / certificación] |
| **Equipo** | [N desarrolladores disponibles] |

---

## Dependencias externas

| Dependencia | Equipo / Proveedor | Fecha requerida | Estado |
|------------|-------------------|----------------|--------|
| API de [Sistema X] | [Nombre del equipo] | [fecha] | 🟢 Disponible |
| Credenciales de [Proveedor Y] | [Contacto] | [fecha] | 🟡 En proceso |
| [Infraestructura Z] | DevOps | [fecha] | 🔴 Pendiente |

---

## Cómo actualizar el alcance

El alcance puede cambiar, pero el cambio tiene un proceso:

1. Documentar el cambio propuesto en este archivo
2. Evaluar el impacto en el cronograma y esfuerzo
3. Obtener aprobación del Product Owner y Tech Lead
4. Actualizar el roadmap en `03-product/vision.md`
5. Crear o actualizar HUs en `04-requirements/user-stories.md`

---

## Correlaciones

- Visión y roadmap → `03-product/vision.md`
- Glosario de términos → `01-context/glossary.md`
- Descripción general del sistema → `01-context/overview.md`
- Riesgos relacionados con el alcance → `15-project-control/risks.md`
