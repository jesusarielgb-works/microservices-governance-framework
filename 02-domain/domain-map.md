# Mapa de Dominio — Bounded Contexts

> **Qué llenar aquí:** El mapa de dominio es el artefacto central del DDD (Domain-Driven Design).
> Define los límites del sistema y cómo se relacionan entre sí.
> Construirlo primero con el equipo y los expertos del dominio en una sesión de Event Storming.

---

## 1. Visión general del dominio

> Un párrafo de contexto sobre el negocio y qué problema resuelve el sistema.
> Escríbelo sin términos técnicos — debe poder leerlo un experto del negocio.

```
[Describe el dominio del negocio aquí. Ej: "El sistema gestiona el ciclo completo de
reservas de [X], desde la solicitud del cliente hasta la confirmación y facturación."]
```

---

## 2. Bounded Contexts identificados

Un **Bounded Context** es el límite explícito dentro del cual un modelo de dominio particular
tiene significado consistente. Cada bounded context tiene su propio lenguaje ubicuo (Ubiquitous Language).

> **Señal de que tienes un buen bounded context:**
> - Tiene un equipo responsable claro
> - Tiene su propia base de datos
> - Puede desplegarse independientemente
> - El mismo término en dos contextos diferentes puede significar cosas distintas

### Bounded Context: [Nombre — ej: Gestión de Usuarios]

| Campo | Valor |
|-------|-------|
| **Nombre** | [NombreDelContexto] |
| **Responsabilidad** | [Qué captura este contexto en una oración] |
| **Equipo dueño** | [Equipo o persona responsable] |
| **Microservicio(s)** | [service-name, service2-name] |
| **Base de datos** | [PostgreSQL / MongoDB / Redis / etc.] |
| **Lenguaje ubicuo** | [Términos clave del negocio en este contexto] |

**Términos propios del contexto (Ubiquitous Language):**

| Término | Significado en ESTE contexto | ¿Diferente en otro contexto? |
|---------|------------------------------|------------------------------|
| [Usuario] | [Persona con cuenta activa] | [Sí — en Facturación es "Cliente"] |
| [Cuenta] | [Conjunto de credenciales] | [No] |

---

### Bounded Context: [Nombre — ej: Gestión de Pedidos]

| Campo | Valor |
|-------|-------|
| **Nombre** | [NombreDelContexto] |
| **Responsabilidad** | |
| **Equipo dueño** | |
| **Microservicio(s)** | |
| **Base de datos** | |
| **Lenguaje ubicuo** | |

---

## 3. Mapa de contextos (Context Map)

El Context Map muestra las relaciones entre bounded contexts. Las relaciones definen
cómo se comunican los contextos y quién tiene el "poder" en la integración.

```
┌─────────────────────┐        ┌──────────────────────┐
│  [Contexto A]       │        │  [Contexto B]        │
│                     │──────▶│                      │
│  Dominio:           │  D→C  │  Dominio:            │
│  [responsabilidad]  │       │  [responsabilidad]   │
└─────────────────────┘       └──────────────────────┘
                                        │
                               D→C      │
                                        ▼
                              ┌──────────────────────┐
                              │  [Contexto C]        │
                              │                      │
                              │  [responsabilidad]   │
                              └──────────────────────┘
```

### Tipos de relaciones entre contextos

| Tipo | Símbolo | Descripción | Ejemplo |
|------|---------|-------------|---------|
| **Upstream → Downstream** | `U → D` | U provee, D consume. D depende de U. | Auth → Orders |
| **Shared Kernel** | `SK` | Dos equipos comparten parte del modelo | Shared User ID |
| **Customer/Supplier** | `C/S` | Supplier (U) negocia con Customer (D) | Inventory → Sales |
| **Conformist** | `CONF` | D adopta el modelo de U sin negociar | Legacy integration |
| **Anti-Corruption Layer** | `ACL` | D traduce el modelo de U para protegerse | Gateway → External API |
| **Open Host Service** | `OHS` | U publica un protocolo publicado | Event Bus, REST API |
| **Published Language** | `PL` | Lenguaje compartido explícito | OpenAPI spec, events |

### Tabla de relaciones

| Contexto A | Relación | Contexto B | Canal de comunicación | Contrato |
|-----------|----------|------------|----------------------|---------|
| [Contexto A] | U → D | [Contexto B] | REST / Evento | OpenAPI / AsyncAPI |
| [Contexto B] | ACL | [Contexto C] | Adaptador | Interfaz interna |

---

## 4. Core Domain, Supporting, Generic

El DDD clasifica los subdominios por su valor estratégico:

| Tipo | Descripción | Inversión | Ejemplo |
|------|-------------|-----------|---------|
| **Core Domain** | Donde está la ventaja competitiva del negocio. Lo que nos diferencia. | MÁXIMA — construir, no comprar | Algoritmo de matching |
| **Supporting Subdomain** | Necesario para el core pero no diferenciador. Puede tercerizarse. | MEDIA | Gestión de órdenes |
| **Generic Subdomain** | Commodity. Existe solución off-the-shelf. | MÍNIMA — comprar/usar OSS | Autenticación, emails |

### Clasificación de los bounded contexts de este proyecto

| Bounded Context | Tipo | Justificación |
|----------------|------|---------------|
| [Contexto A] | Core | [por qué es el corazón del negocio] |
| [Contexto B] | Supporting | [por qué apoya sin ser diferenciador] |
| [Contexto C] | Generic | [solución estándar disponible] |

---

## 5. Decisiones de modelado

### ¿Cómo se tomaron estas decisiones?

> Documenta la sesión de Event Storming o el proceso que usaste para llegar al mapa.
> Si cambiaste el mapa, explica por qué.

- **Sesión de Event Storming:** [fecha], [participantes]
- **Herramienta usada:** [Miro / Lucidchart / pizarra física]
- **Iteraciones del mapa:** v1 (fecha), v2 (fecha)

### Decisiones clave y alternativas descartadas

| Decisión | Alternativa descartada | Razón |
|----------|----------------------|-------|
| [Separar Contexto A y B] | [Tenerlos en uno solo] | [La lógica de negocio es diferente y evolucionan a ritmos distintos] |

---

## 6. Cómo actualizar este mapa

1. Antes de agregar un nuevo microservicio, verifica si pertenece a un bounded context existente.
2. Si el lenguaje ubicuo de un contexto está cambiando, revisa si el contexto debe dividirse.
3. Haz una sesión de Event Storming cada vez que el dominio cambie significativamente.
4. El mapa de contextos DEBE estar sincronizado con el diagrama C4 nivel sistema (`05-architecture/overview.md`).

> **Correlación importante:** Los bounded contexts en este documento →
> Microservicios en `09-microservices/service-catalog.md` →
> Diagramas C4 en `08-uml/` →
> ADRs de separación de servicios en `05-architecture/decisions/`
