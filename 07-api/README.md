# 07 — Contratos de API

> **¿Qué es esto?** La especificación formal de cómo se comunican los microservicios entre sí
> y con el exterior. El contrato es la promesa que un servicio hace a sus consumidores.

## Por qué los contratos son críticos en microservicios

En un monolito, cambiar una función es fácil: el compilador te dice qué rompes.
En microservicios, un cambio en una API puede romper silenciosamente otros servicios en producción.

**Contract-first development:** diseña la API antes de implementarla. Así el equipo
puede trabajar en paralelo (frontend, backend, otros servicios) con un contrato acordado.

---

## Qué hay aquí y cómo llenarlo

### `guidelines.md` ⭐
Estándares REST del proyecto. **Todo el equipo debe seguir esto antes de diseñar un endpoint.**

**Temas clave a definir:**
```markdown
## Versionado
- URL: /api/v1/recursos (versión en la URL)
- Header: Accept: application/vnd.api+json;version=1

## Naming de endpoints
- Plural para colecciones: GET /usuarios
- Sustantivos, no verbos: GET /usuarios/{id}/pedidos (no: GET /obtenerPedidosDeUsuario)
- snake_case o kebab-case para URLs

## Paginación
- ?page=1&limit=20 (offset-based)
- ?cursor=xyz (cursor-based para grandes volúmenes)

## Respuestas estándar
| Código | Cuándo usarlo |
|--------|---------------|
| 200 | Éxito con body |
| 201 | Creación exitosa |
| 204 | Éxito sin body |
| 400 | Error del cliente (validación) |
| 401 | No autenticado |
| 403 | No autorizado |
| 404 | Recurso no encontrado |
| 422 | Entidad no procesable |
| 500 | Error del servidor |

## Formato de errores
{
  "error": "VALIDATION_ERROR",
  "message": "El campo email es requerido",
  "details": [{"field": "email", "message": "requerido"}]
}
```

### `authentication.md` ⭐
Estrategia de autenticación y autorización del sistema.
**Llena:** qué mecanismo (JWT, OAuth2, API Key), flujo de autenticación, políticas de expiración,
manejo de refresh tokens, RBAC (roles y permisos).

### `contracts/openapi/` ⭐⭐
Un archivo `.yaml` por microservicio con el contrato OpenAPI 3.0.

**Nombre del archivo:** `nombre-del-servicio.yaml`

**Estructura mínima de cada contrato:**
```yaml
openapi: 3.0.3
info:
  title: [Nombre Servicio] API
  version: 1.0.0
  description: [Qué hace este servicio]

servers:
  - url: http://localhost:8080/api/v1
    description: Local

paths:
  /recursos:
    get:
      summary: Listar recursos
      tags: [Recursos]
      parameters:
        - name: page
          in: query
          schema:
            type: integer
      responses:
        '200':
          description: Lista de recursos
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ResourceList'

components:
  schemas:
    Resource:
      type: object
      required: [id, nombre]
      properties:
        id:
          type: string
          format: uuid
        nombre:
          type: string
  
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - bearerAuth: []
```

### `_shared.yaml`
Schemas y componentes reutilizables entre todos los contratos (paginación, errores, etc.)

---

## Herramientas recomendadas

- **Swagger Editor** — Editor online para OpenAPI
- **Redocly** — Genera documentación HTML desde el YAML (ya configurado en `redocly.yaml`)
- **Postman / Insomnia** — Para probar los contratos
- **OpenAPI Generator** — Genera código cliente/servidor desde el YAML

---

## Correlaciones con otras secciones

| Esta sección se alimenta de... | Y alimenta a... |
|-------------------------------|-----------------|
| `04-requirements/functional.md` → qué debe hacer | Endpoints que implementan cada RF |
| `06-data/models.md` → qué datos existen | Schemas en los contratos |
| `02-domain/domain-events.md` | Endpoints de publicación de eventos |
| Contratos aquí | `09-microservices/[servicio]/` que los implementa |
| Contratos aquí | `11-quality/testing-strategy.md` → pruebas de contrato |

---

## Preguntas que esta sección debe responder

- ¿Cómo se llama cada endpoint y qué hace?
- ¿Qué datos recibe y qué devuelve?
- ¿Cómo se autentica el cliente?
- ¿Qué errores puede devolver el servicio?
- ¿Cómo versiono la API sin romper a los consumidores?
