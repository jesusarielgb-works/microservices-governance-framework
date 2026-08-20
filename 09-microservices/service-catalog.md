# Catálogo de Microservicios

> **Qué llenar aquí:** El inventario completo de servicios del sistema.
> Para el detalle de cada servicio, ve a `09-microservices/services/NN-nombre-servicio/`.
> Este catálogo es la vista ejecutiva — una fila por servicio.

---

## Mapa de servicios

```
                          ┌─────────────────┐
  Clientes externos       │   API Gateway   │
  (Web, Móvil, API)  ──▶ │   :8080         │
                          └────────┬────────┘
                                   │ enruta según path
          ┌───────────────┬────────┴───────────┬──────────────────┐
          ▼               ▼                    ▼                  ▼
  ┌──────────────┐ ┌──────────────┐  ┌──────────────────┐ ┌──────────────┐
  │ auth-service │ │[servicio-A]  │  │ [servicio-B]     │ │[servicio-N]  │
  │    :3001     │ │    :3002     │  │    :3003         │ │    :300N     │
  └──────────────┘ └──────┬───────┘  └────────┬─────────┘ └──────────────┘
                           │                   │
                           └──────────┬────────┘
                                      ▼
                            ┌─────────────────┐
                            │  Message Broker  │
                            │  (Kafka/RabbitMQ)│
                            └─────────────────┘
```

---

## Registro de servicios

| # | Servicio | Responsabilidad principal | Puerto | BD | Tipo BD | Estado |
|---|---------|--------------------------|--------|-----|---------|--------|
| 01 | `api-gateway` | Enrutamiento, autenticación, rate limiting | 8080 | Redis | Caché | 🟢 Activo |
| 02 | `auth-service` | Registro, login, tokens JWT, RBAC | 3001 | PostgreSQL | Relacional | 🟢 Activo |
| 03 | `[nombre-servicio]` | [responsabilidad] | 3002 | [BD] | [tipo] | 🟡 En desarrollo |
| 04 | `[nombre-servicio]` | [responsabilidad] | 3003 | [BD] | [tipo] | 🔴 Planificado |

**Estados:** 🟢 Activo en prod | 🟡 En desarrollo | 🔴 Planificado | ⏸ Deprecado

---

## Detalle por servicio

### 01 — `api-gateway`

| Campo | Valor |
|-------|-------|
| **Carpeta** | `09-microservices/services/01-api-gateway/` |
| **Responsabilidad** | Punto único de entrada. Enruta, autentica y aplica rate limiting |
| **Tipo** | Infrastructure service (no de dominio) |
| **Puerto** | 8080 (HTTPS 8443) |
| **Tecnología** | [Kong / NGINX + custom / Spring Cloud Gateway] |
| **BD** | Redis (caché de tokens, rate limiting) |
| **Comunica con** | Todos los servicios internos |
| **Expuesto externamente** | Sí — es el único punto de entrada |

**Endpoints clave:**
- `/*` — Proxy a servicios internos según el path

---

### 02 — `auth-service`

| Campo | Valor |
|-------|-------|
| **Carpeta** | `09-microservices/services/02-auth-service/` |
| **Responsabilidad** | Identidad y acceso: registro, login, refresh token, RBAC |
| **Tipo** | Supporting service |
| **Puerto** | 3001 |
| **Tecnología** | [Node.js + Express / Spring Boot / etc.] |
| **BD** | PostgreSQL |
| **Bounded Context** | `02-domain/domain-map.md` → Contexto IAM |
| **Expuesto externamente** | Solo a través del API Gateway |

**Endpoints clave:**
- `POST /auth/register` — Registro de usuario
- `POST /auth/login` — Autenticación, devuelve JWT
- `POST /auth/refresh` — Renovar token
- `DELETE /auth/logout` — Invalidar refresh token

**Eventos que publica:**
- `UsuarioRegistrado` → Topic: `iam.usuario.registrado`
- `SesionIniciada` → Topic: `iam.sesion.iniciada`

---

### 03 — `[nombre-servicio]`

| Campo | Valor |
|-------|-------|
| **Carpeta** | `09-microservices/services/03-[nombre]/` |
| **Responsabilidad** | [Describe en una oración] |
| **Tipo** | [Core / Supporting / Generic] |
| **Puerto** | 300X |
| **Tecnología** | [Stack] |
| **BD** | [Motor y nombre] |
| **Bounded Context** | [Referencia en domain-map] |
| **Expuesto externamente** | [Sí / Solo interno] |

**Endpoints clave:**
- `[MÉTODO] /[path]` — [descripción]

**Eventos que publica:**
- `[NombreEvento]` → Topic: `[topic.name]`

**Eventos que consume:**
- `[NombreEvento]` de `[servicio origen]`

---

## Matriz de comunicación entre servicios

> Quién llama a quién y por qué canal.

| Servicio origen | Servicio destino | Canal | Tipo | Descripción |
|----------------|-----------------|-------|------|-------------|
| api-gateway | auth-service | HTTP/REST | Síncrono | Validar JWT en cada request |
| [svc-a] | [svc-b] | Evento | Asíncrono | Notificar de [evento] |
| [svc-b] | [svc-c] | HTTP/REST | Síncrono | Consultar [dato] |

---

## Matriz de propiedad de datos

Cada entidad de negocio tiene un solo servicio propietario.
Los demás acceden vía API o eventos, nunca directamente a la BD.

| Entidad / Dato | Servicio propietario (Source of Truth) | Cómo acceden otros servicios |
|---------------|---------------------------------------|------------------------------|
| Usuario / Identidad | auth-service | API REST (`GET /users/:id`) |
| [Entidad A] | [servicio-A] | Evento `[EntidadACreada]` |
| [Entidad B] | [servicio-B] | API REST (`GET /b/:id`) |

---

## Cómo agregar un nuevo servicio

1. Asigna el número siguiente disponible en este catálogo
2. Crea la carpeta: `cp -r 09-microservices/_template/service 09-microservices/services/NN-nuevo-servicio`
3. Llena el README, data-model, events y decisions del nuevo servicio
4. Crea el contrato OpenAPI en `07-api/contracts/openapi/NN-nuevo-servicio.yaml`
5. Agrega el servicio a este catálogo
6. Agrega el ADR de la decisión de crear el servicio en `05-architecture/decisions/`
7. Actualiza el diagrama en `05-architecture/overview.md`

---

## Correlaciones

- Arquitectura y patrones → `05-architecture/`
- Contratos API por servicio → `07-api/contracts/openapi/`
- Eventos del sistema → `02-domain/domain-events.md`
- Template para crear servicios → `09-microservices/_template/service/`
- Detalle completo por servicio → `09-microservices/services/NN-[nombre]/`
