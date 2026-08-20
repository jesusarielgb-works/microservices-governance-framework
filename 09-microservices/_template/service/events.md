# Eventos — [Nombre del Servicio]

> Los eventos son la forma en que los microservicios se comunican de manera asíncrona.
> Documenta aquí todos los eventos que este servicio **publica** y **consume**.

---

## Eventos que PUBLICA este servicio

> Estos son los eventos que este servicio emite cuando algo importante ocurre.
> Otros servicios pueden suscribirse a ellos.

| Evento | Topic/Exchange | Cuándo se emite | Payload (campos clave) |
|--------|---------------|-----------------|----------------------|
| [NombreEnPasado] | [nombre.del.topic] | [condición que lo dispara] | `{campo1, campo2}` |

### Esquema de evento: [NombreEvento]

```json
{
  "eventId": "uuid",
  "eventType": "[NombreEvento]",
  "timestamp": "2024-01-01T12:00:00Z",
  "version": "1.0",
  "source": "[nombre-servicio]",
  "payload": {
    "[campo1]": "[tipo y descripción]",
    "[campo2]": "[tipo y descripción]"
  }
}
```

---

## Eventos que CONSUME este servicio

> Estos son los eventos de otros servicios a los que este servicio reacciona.

| Evento | Publicado por | Topic/Exchange | Qué hace este servicio al recibirlo |
|--------|--------------|----------------|-------------------------------------|
| [NombreEvento] | [servicio origen] | [topic] | [acción que toma] |

---

## Garantías de entrega

| Garantía | Valor | Implicación |
|----------|-------|-------------|
| Al menos una vez (at-least-once) | [Sí/No] | Los consumidores deben ser idempotentes |
| Como máximo una vez (at-most-once) | [Sí/No] | Puede perderse algún evento |
| Exactamente una vez (exactly-once) | [Sí/No] | Más costoso, más confiable |

---

## Manejo de errores

- **Dead Letter Queue:** [nombre de la DLQ donde van los eventos fallidos]
- **Reintentos:** [N reintentos con backoff exponencial]
- **Alertas:** [cuándo se alerta al equipo de un evento en DLQ]
