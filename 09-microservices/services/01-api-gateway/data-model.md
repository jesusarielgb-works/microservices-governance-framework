# Modelo de Datos — API Gateway

> El API Gateway es **stateless**: no tiene base de datos propia.
> Su estado de configuración (rutas, rate limits, allowlists) se gestiona
> mediante variables de ambiente y archivos de configuración versionados en Git,
> no en tablas de base de datos.

---

## Motor de base de datos

**Motor:** No aplica — sin base de datos persistente.

**Justificación:** Un API Gateway que requiere una BD propia introduce un punto de falla
adicional en la capa que todo el tráfico atraviesa. El estado de routing se gestiona
via configuración (archivos YAML o variables de ambiente), y el estado de sesión
(token blacklist, rate limits) se delega a Redis compartido.

---

## Estado en memoria / Redis

El API Gateway sí mantiene estado efímero:

| Dato | Almacenamiento | TTL | Propósito |
|------|---------------|-----|-----------|
| Rate limit counters | Redis | 60 segundos | Conteo de peticiones por IP |
| Token blacklist check | Redis (read-only) | Según expiración del token | Verificar tokens revocados |

El auth-service es el **dueño** del token blacklist — el gateway solo lo consulta.

---

## Archivos de configuración de rutas

En lugar de tablas, las rutas se definen en:

```
config/
├── routes.yaml          # Mapa de rutas: path → servicio + puerto
├── rate-limits.yaml     # Límites por ruta y por rol
└── cors.yaml            # Configuración CORS por origen
```

Estos archivos están versionados en Git y se recargo en el startup del servicio.
Un cambio de rutas requiere un redeploy (intencional: los cambios de routing son cambios de infraestructura).

---

## Correlaciones

- Configuración de ambientes → `10-devops/environments.md`
- Runbook del gateway → `runbook.md`
- Catálogo de rutas disponibles → `09-microservices/service-catalog.md`
