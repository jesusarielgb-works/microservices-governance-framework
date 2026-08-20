# [Nombre del Servicio]

> Reemplaza este README con la documentación del servicio específico.
> Elimina las instrucciones entre `[corchetes]` cuando estén completas.

---

## Responsabilidad

> [Una oración: qué hace este servicio. Qué dato es su dueño autoritativo.]

---

## Ubicación en la arquitectura

| Campo | Valor |
|-------|-------|
| Número en catálogo | [NN] |
| Puerto local | [80XX] |
| Repositorio | [URL] |
| Motor de BD | [PostgreSQL / MongoDB / Redis / —] |
| Comunica con | [lista de otros servicios que consume] |
| Es consumido por | [lista de servicios/frontends que lo llaman] |

---

## Responsabilidades (qué SÍ hace este servicio)

- [Responsabilidad 1]
- [Responsabilidad 2]

## Fuera de su alcance (qué NO hace)

- [Qué delegó a otro servicio y cuál]

---

## Cómo correrlo localmente

```bash
# Desde la raíz del proyecto
docker compose up -d [nombre-servicio]

# Verificar
curl http://localhost:[puerto]/health
```

---

## Documentos relacionados

- [data-model.md](./data-model.md) — Esquema de BD
- [events.md](./events.md) — Eventos que publica y consume
- [decisions.md](./decisions.md) — Decisiones de diseño internas
- [runbook.md](./runbook.md) — Operación en producción
- [Contrato API](../../../07-api/contracts/openapi/[nombre-servicio].yaml)
