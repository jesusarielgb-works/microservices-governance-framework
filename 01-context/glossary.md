# Glosario del Proyecto

> **Instrucciones:** Define aquí todos los términos técnicos y de negocio usados en el proyecto.
> Este es el diccionario oficial — si hay ambigüedad, este documento gana.
> Agrega términos durante todo el proyecto, no solo al inicio.

---

## Cómo usar este glosario

1. Antes de usar un término técnico o de negocio en código, docs o conversaciones: búscalo aquí.
2. Si no está: agrégalo con su definición.
3. Si hay desacuerdo sobre la definición: discútela en el equipo y actualiza este documento.

---

## Términos del dominio

| Término | Definición | Notas / Sinónimos |
|---------|------------|------------------|
| [Término A] | [Definición precisa en el contexto de este sistema] | [Sinónimos o usos alternativos a EVITAR] |
| [Término B] | [Definición] | |

---

## Términos técnicos del proyecto

| Término | Definición |
|---------|------------|
| Microservicio | Servicio independiente con responsabilidad única, su propio proceso y su propia base de datos |
| Evento de dominio | Hecho que ocurrió en el negocio y que otros servicios pueden observar. Nombre siempre en pasado. |
| Bounded Context | Límite dentro del cual un modelo de dominio particular tiene significado consistente |
| API Gateway | Punto único de entrada al sistema que enruta solicitudes a los microservicios correspondientes |
| Circuit Breaker | Patrón que detiene las llamadas a un servicio que está fallando, evitando cascadas de fallos |
| Saga | Secuencia de transacciones locales en diferentes servicios con compensaciones en caso de fallo |
| Dead Letter Queue | Cola donde van los mensajes que no pudieron ser procesados después de varios reintentos |
| Idempotencia | Propiedad de una operación de producir el mismo resultado si se ejecuta varias veces |

---

## Acrónimos

| Acrónimo | Significado |
|----------|-------------|
| IAM | Identity and Access Management |
| JWT | JSON Web Token |
| API | Application Programming Interface |
| CRUD | Create, Read, Update, Delete |
| DTO | Data Transfer Object |
| RF | Requisito Funcional |
| RNF | Requisito No Funcional |
| SLO | Service Level Objective |
| SLA | Service Level Agreement |
| ADR | Architecture Decision Record |
| PR | Pull Request |
| DoD | Definition of Done |
| CI/CD | Continuous Integration / Continuous Delivery |
