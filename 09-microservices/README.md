# 09 — Microservicios

> **¿Qué es esto?** La documentación individual de cada microservicio del sistema.
> Esta es la sección más viva del repositorio — se actualiza con cada cambio significativo.

## Estructura de cada microservicio

Cada servicio tiene su propia carpeta en `services/` con esta estructura:

```
services/
└── 01-nombre-del-servicio/
    ├── README.md              ← Descripción, responsabilidad, cómo correrlo
    ├── data-model.md          ← Modelo de datos propio del servicio
    ├── events.md              ← Eventos que publica y consume
    ├── decisions.md           ← Decisiones de diseño específicas del servicio
    ├── runbook.md             ← Cómo operar el servicio en producción
    └── components/            ← Si el servicio tiene sub-componentes
        └── [componente]/
            ├── README.md
            └── contract.md
```

**Usa `_template/service/` como punto de partida para cada nuevo servicio.**

---

## Documentos transversales (aplican a TODOS los servicios)

### `service-catalog.md` ⭐ (Empezar aquí)
Registro de todos los microservicios del sistema.
**Llena:** cuando defines los microservicios. Actualiza cuando agregas/eliminas/renombras.

**Formato:**
```markdown
| # | Servicio | Responsabilidad | Puerto local | Repo | Estado |
|---|---------|-----------------|-------------|------|--------|
| 01 | iam | Autenticación y autorización | 8001 | [repo] | ✅ En producción |
| 02 | reference-data | Datos maestros del sistema | 8002 | [repo] | 🚧 En desarrollo |
```

### `service-boundary-rules.md` ⭐
Las reglas que definen dónde termina cada servicio.
**Llena:** criterios para decidir si algo pertenece a servicio A o B. Estas reglas previenen
que los servicios se solapen o que responsabilidades queden en el limbo.

**Incluye:**
- ¿Cómo decidimos los límites? (por dominio, por equipo, por ciclo de vida de datos)
- ¿Qué hacer cuando algo "podría ir en cualquiera de los dos"?
- Regla de la "base de datos": si dos cosas comparten BD, son el mismo servicio o hay un error

### `communication-patterns.md` ⭐
Cómo se comunican los servicios entre sí.
**Llena:** qué usa REST (síncrono), qué usa eventos (asíncrono), por qué, ejemplos concretos.

**Patrones a documentar:**
- Request/Response (REST/gRPC): cuándo usarlo, cuándo no
- Eventos/Mensajes (RabbitMQ/Kafka): cuándo usarlo, cómo se garantiza la entrega
- API Gateway: qué pasa ahí, qué no pasa
- Service Discovery: cómo se encuentran los servicios

### `event-catalog.md` ⭐
Catálogo de todos los eventos del sistema.
**Llena:** nombre, payload, quién publica, quién consume, en qué topic/exchange.

**Formato:**
```markdown
| Evento | Payload (campos clave) | Publicado por | Consumido por | Topic/Exchange |
|--------|----------------------|--------------|---------------|----------------|
| UsuarioCreado | userId, email, rol | iam | actors, audit | users.events |
```

### `data-ownership-matrix.md` ⭐
Quién es el dueño autoritativo de cada dato.
**Llena:** para cada entidad de negocio, cuál servicio tiene la "versión oficial".

**Formato:**
```markdown
| Entidad | Servicio dueño | Otros que tienen copia | Cómo se sincroniza |
|---------|---------------|----------------------|-------------------|
| Usuario | iam | actors (datos básicos) | evento UsuarioCreado |
```

### `dependency-map.md`
Mapa de dependencias entre servicios.
**Llena:** grafo (puede ser ASCII) que muestra qué servicio depende de cuál.
Alerta roja: si el grafo tiene ciclos → hay un problema de diseño.

### `storage-and-documents.md`
Cómo el sistema maneja almacenamiento de archivos.
**Llena:** qué servicio maneja archivos, dónde se almacenan (S3, MinIO, local), política de nombres.

### `service-readiness-checklist.md`
Checklist que todo servicio debe cumplir antes de ir a producción.

---

## Correlaciones con otras secciones

| Esta sección se alimenta de... | Y alimenta a... |
|-------------------------------|-----------------|
| `02-domain/domain-map.md` → bounded contexts | Uno por contexto |
| `05-architecture/overview.md` → servicios definidos | README de cada servicio |
| `07-api/contracts/openapi/` → contrato del servicio | `services/[servicio]/` lo implementa |
| `06-data/models.md` → modelo de datos | `services/[servicio]/data-model.md` |
| `services/[servicio]/runbook.md` | `13-operations/` los consolida |

---

## Numeración de servicios

Numera los servicios para indicar dependencias implícitas:
los servicios con número más bajo tienden a ser más fundamentales.

Convención sugerida:
- `01` = IAM / Seguridad (todos dependen de este)
- `02` = Datos de referencia / maestros
- `03-0N` = Servicios de dominio
- `0N+1` = Servicios de soporte (documentos, notificaciones, audit)
