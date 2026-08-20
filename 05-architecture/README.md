# 05 — Arquitectura

> **¿Qué es esto?** Las decisiones de diseño del sistema: cómo está organizado, por qué,
> qué alternativas se evaluaron, y cómo se despliega. Las ADRs son el tesoro de esta sección.

## Por qué existe esta sección

La arquitectura de un sistema es el conjunto de decisiones que son difíciles de cambiar después.
Documentarlas tiene tres beneficios:
1. **Nuevos integrantes** entienden el sistema sin preguntar todo desde cero
2. **El equipo** no repite discusiones ya resueltas
3. **Años después**, todos recuerdan por qué se tomó cada decisión

---

## Qué hay aquí y cómo llenarlo

### `overview.md` ⭐ (Empezar aquí)
Vista de alto nivel del sistema completo.
**Llena:** diagrama C4 Nivel 1 (System) y Nivel 2 (Container), lista de microservicios con
responsabilidad de cada uno, cómo se comunican (sync/async), tecnologías por capa.

**Formato recomendado:**
```markdown
## Diagrama de arquitectura
[Diagrama ASCII, Mermaid, o referencia a imagen en assets/]

## Microservicios
| Servicio | Responsabilidad | Tecnología | BD |
|---------|-----------------|------------|-----|
| [nombre] | [qué hace] | [stack] | [motor] |

## Patrones de comunicación
- Sync: [qué usa REST entre qué servicios]
- Async: [qué usa eventos/mensajes entre qué servicios]
- Gateway: [cómo llegan las peticiones del exterior]
```

### `deployment.md` ⭐
Cómo se despliega el sistema en cada ambiente.
**Llena:** diagrama de infraestructura, qué va en Docker/K8s, configuración de red, requisitos de hardware.

### `cross-cutting.md`
Concerns que aplican a todos los microservicios.
**Llena:** logging estándar, tracing distribuido, configuración centralizada, feature flags,
manejo de errores, retry policies.

### `pattern-guide.md`
Catálogo de patrones de diseño usados en el proyecto.
**Llena:** por cada patrón: nombre, cuándo usarlo, cuándo NO usarlo, ejemplo concreto del proyecto.

### `security-threat-model.md`
Análisis de amenazas de seguridad del sistema.
**Llena:** usando metodología STRIDE: Spoofing, Tampering, Repudiation, Information Disclosure,
Denial of Service, Elevation of Privilege. Para cada amenaza: mitigación implementada.

### `decisions/` ⭐⭐ — Architecture Decision Records (ADRs)

#### ¿Qué es un ADR?
Un registro de UNA decisión arquitectónica importante: qué se decidió, por qué, qué alternativas
se evaluaron y cuáles son las consecuencias. Son **documentos cortos** (1-2 páginas).

**Cuándo crear un ADR:**
- Al elegir un message broker (RabbitMQ vs Kafka vs Redis Streams)
- Al decidir la estrategia de base de datos (una por servicio vs compartida)
- Al elegir patrón de comunicación (REST vs gRPC vs eventos)
- Al elegir librería de autenticación
- Cualquier decisión que si cambia, requiere refactoring significativo

**Cuándo NO crear un ADR:**
- Decisiones operativas del día a día
- Cosas que se pueden cambiar fácilmente sin impacto sistémico

**Usa `decisions/_template-adr.md`**

**Ejemplo de ADRs típicos:**
```
ADR-001-message-broker.md       → Por qué RabbitMQ y no Kafka
ADR-002-auth-strategy.md        → Por qué JWT y no sessions
ADR-003-database-per-service.md → Por qué BD separada por servicio
ADR-004-api-gateway.md          → Por qué Kong y no NGINX custom
```

---

## Correlaciones con otras secciones

| Esta sección se alimenta de... | Y alimenta a... |
|-------------------------------|-----------------|
| `02-domain/domain-map.md` → bounded contexts | `09-microservices/` → un servicio por contexto |
| `04-requirements/non-functional.md` → RNFs | Decisiones sobre tecnología y escala |
| ADRs elegidos aquí | `09-microservices/` implementan los patrones decididos |
| `deployment.md` | `10-devops/environments.md` |

---

## Los 5 errores de arquitectura más comunes

1. **Microservicios demasiado pequeños** — Si un "servicio" no puede existir de forma independiente, no es un microservicio.
2. **Base de datos compartida** — Destruye la independencia de los servicios. Cada servicio, su propia BD.
3. **Comunicación solo sincrónica** — Para operaciones no urgentes, los eventos asíncronos escalan mejor.
4. **Sin API Gateway** — Exponer microservicios directamente al frontend genera acoplamiento.
5. **Sin decisiones documentadas** — En 6 meses nadie recuerda por qué se eligió X.

---

## Preguntas que esta sección debe responder

- ¿Cómo está organizado el sistema en grandes bloques?
- ¿Por qué se eligió cada tecnología clave?
- ¿Qué alternativas se evaluaron y por qué se descartaron?
- ¿Cómo se despliega el sistema?
- ¿Qué patrones aplica el equipo y cómo?
