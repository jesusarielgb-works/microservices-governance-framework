# Ejemplo: Diagrama C4 Nivel Contenedor

> Este archivo muestra cómo documentar el sistema con un diagrama C4 Level 2 (Container).
> El diagrama usa Mermaid, que renderiza nativamente en GitHub, GitLab y la mayoría de
> editores modernos (VS Code con extensión, Obsidian, etc.).
>
> **Instrucción:** Copia esta estructura, reemplaza los servicios de ejemplo (api-gateway,
> auth-service) por los servicios reales de tu proyecto, y muévelo a `diagrams/source/c4-container.md`.

---

## Sistema de ejemplo: plataforma de gestión de [X]

```mermaid
C4Container
    title Diagrama de Contenedores — [Nombre del Sistema]

    Person(usuario, "Usuario / Cliente", "Accede al sistema desde el navegador o app móvil")
    Person(admin, "Administrador", "Gestiona configuración y usuarios del sistema")

    System_Boundary(sistema, "[Nombre del Sistema]") {

        Container(frontend, "Frontend Web", "React / Vue / Angular", "Interfaz de usuario. SPA servida como archivos estáticos.")

        Container(gateway, "API Gateway", "[Tu framework]", "Punto único de entrada. Autentica tokens JWT, enruta al servicio correcto, aplica rate limiting.")

        Container(auth, "Auth Service", "[Tu stack]", "Gestiona autenticación (login/registro/refresh). Dueño de la entidad User y Role.")

        Container(servicio_a, "[Servicio A]", "[Tu stack]", "[Responsabilidad principal del servicio A. Dueño de [Entidad A].]")

        Container(servicio_b, "[Servicio B]", "[Tu stack]", "[Responsabilidad principal del servicio B. Dueño de [Entidad B].]")

        ContainerDb(db_auth, "BD Auth", "PostgreSQL", "Usuarios, roles, refresh tokens.")
        ContainerDb(db_a, "BD [Servicio A]", "[Motor elegido]", "Datos del dominio de [Servicio A].")
        ContainerDb(db_b, "BD [Servicio B]", "[Motor elegido]", "Datos del dominio de [Servicio B].")
        ContainerDb(redis, "Redis", "Redis", "Token blacklist, rate limiting, caché compartida.")
        ContainerDb(broker, "Message Broker", "Kafka / RabbitMQ", "Eventos de dominio entre servicios.")
    }

    System_Ext(ext_email, "Servicio de Email", "Sendgrid / SES / SMTP")
    System_Ext(ext_pago, "Pasarela de pago", "[Si aplica al proyecto]")

    %% Relaciones Usuario → Sistema
    Rel(usuario, frontend, "Usa", "HTTPS")
    Rel(admin, frontend, "Administra", "HTTPS")
    Rel(frontend, gateway, "API calls", "HTTPS / REST")

    %% Gateway → Servicios
    Rel(gateway, auth, "Verifica tokens / enruta", "HTTP interno")
    Rel(gateway, servicio_a, "Enruta peticiones", "HTTP interno")
    Rel(gateway, servicio_b, "Enruta peticiones", "HTTP interno")

    %% Servicios → BD (cada servicio tiene su propia BD)
    Rel(auth, db_auth, "Lee / escribe", "SQL")
    Rel(auth, redis, "Token blacklist", "Redis protocol")
    Rel(gateway, redis, "Rate limiting", "Redis protocol")
    Rel(servicio_a, db_a, "Lee / escribe", "SQL / NoSQL")
    Rel(servicio_b, db_b, "Lee / escribe", "SQL / NoSQL")

    %% Comunicación asíncrona
    Rel(auth, broker, "Publica eventos (user.registered)", "Async")
    Rel(servicio_a, broker, "Publica / consume eventos", "Async")
    Rel(servicio_b, broker, "Consume eventos de [Servicio A]", "Async")

    %% Sistemas externos
    Rel(servicio_b, ext_email, "Envía notificaciones", "HTTPS / SMTP")
    Rel(servicio_a, ext_pago, "Procesa pagos", "HTTPS")
```

---

## Cómo usar este diagrama

1. **Renderizar en GitHub:** El diagrama se renderiza automáticamente al hacer push. No se necesita ninguna herramienta adicional.

2. **Renderizar en VS Code:** Instalar la extensión "Markdown Preview Mermaid Support" o usar la extensión oficial de Mermaid.

3. **Exportar como imagen:** Usar la [CLI de Mermaid](https://github.com/mermaid-js/mermaid-cli):
```bash
mmdc -i c4-container-example.md -o ../exports/c4-container.svg
```

4. **Actualizar el diagrama:** El diagrama vive en Git junto al código. Cuando cambies la arquitectura (nuevo servicio, nuevo motor de BD), actualiza el diagrama en el mismo PR.

---

## Convenciones para este proyecto

| Elemento | Color / Estilo | Cuándo usarlo |
|----------|---------------|---------------|
| `Container` | Azul | Microservicio o aplicación deployable |
| `ContainerDb` | Cilindro azul | Base de datos, caché, broker |
| `Person` | Icono persona | Actor humano |
| `System_Ext` | Gris | Sistema de terceros que no controlas |

---

## Correlaciones

- Catálogo de servicios → `09-microservices/service-catalog.md`
- Detalles de cada servicio → `09-microservices/services/`
- Índice de todos los diagramas → `08-uml/diagram-index.md`
