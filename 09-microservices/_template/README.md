# [Nombre del Microservicio]

> **Copia esta carpeta** para documentar un nuevo microservicio.
> Renombra la carpeta a `NN-nombre-del-servicio` (ej: `03-academic-service`).
> Llena cada archivo — elimina estas instrucciones cuando el documento esté completo.

---

## Responsabilidad

> [Una sola oración: qué hace este servicio y de qué datos es dueño]

**Ejemplo:** "Gestiona la autenticación, sesiones y permisos de todos los usuarios del sistema."

---

## Ubicación en la arquitectura

| Campo | Valor |
|-------|-------|
| Número en el catálogo | [01, 02, 03...] |
| Puerto local | [8001, 8002...] |
| Repositorio | [URL del repo] |
| Equipo dueño | [nombre del equipo] |
| Motor de BD | [PostgreSQL / MongoDB / Redis / ninguna] |
| Estado | [En desarrollo / En producción / Deprecado] |

---

## Cómo correrlo localmente

```bash
# Prerequisitos
# - Docker Desktop instalado
# - [otras dependencias]

# Clonar el repo
git clone [url]

# Variables de entorno
cp .env.example .env
# Editar .env con valores locales

# Levantar con Docker
docker-compose up -d

# Verificar que corre
curl http://localhost:[puerto]/health
```

---

## Endpoints principales

> [Lista rápida de los endpoints más importantes. El contrato completo está en `07-api/contracts/openapi/`]

| Método | Path | Descripción |
|--------|------|-------------|
| POST | /auth/login | Autenticar usuario |
| POST | /auth/refresh | Renovar token |
| GET | /users/{id} | Obtener usuario |

---

## Documentos de este servicio

- [data-model.md](./data-model.md) — Tablas y esquema de BD
- [events.md](./events.md) — Eventos que publica y consume
- [decisions.md](./decisions.md) — Decisiones de diseño específicas
- [runbook.md](./runbook.md) — Cómo operar en producción
