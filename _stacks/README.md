# Stack-Specific Implementation Guides

> Este cascarón de documentación es **agnóstico de tecnología**: los conceptos de DDD,
> hexagonal, TDD y patrones funcionan igual en Java, Python, Go o Node.js.
>
> Esta carpeta contiene las guías de implementación **específicas por stack**: estructura
> de carpetas real, comandos concretos, librerías recomendadas y ejemplos de código.
> Los documentos principales te dicen el **qué y el porqué**; tu guía de stack te dice el **cómo concreto**.

---

## Guías disponibles

| Stack | Archivo | Cuándo usarlo |
|-------|---------|---------------|
| Node.js + TypeScript | [node-typescript.md](./node-typescript.md) | Backend en Express/Fastify/NestJS con TypeScript |
| Java + Spring Boot | [java-spring.md](./java-spring.md) | Backend en Spring Boot 3.x con Maven o Gradle |
| Python + FastAPI | [python-fastapi.md](./python-fastapi.md) | Backend en FastAPI/Django con Python 3.10+ |
| Go | [go.md](./go.md) | Backend en Go con net/http o Gin/Echo |

---

## Cómo usar estas guías

1. Elige el stack de tu proyecto
2. Lee la guía de tu stack — te dará la estructura de carpetas exacta para ese lenguaje
3. Usa esa estructura cuando llenes `09-microservices/services/NN-tu-servicio/`
4. Los ejemplos de código en los documentos principales (hexagonal, TDD, patrones) son
   conceptuales; tradúcelos a tu stack usando las convenciones de esta guía

---

## ¿No ves tu stack?

Si tu proyecto usa una tecnología no listada (Ruby on Rails, .NET, Kotlin, Rust, etc.),
usa `node-typescript.md` como referencia estructural y adapta los conceptos a las
convenciones de tu lenguaje. La lógica de capas hexagonales es la misma en todos.

---

## Qué contiene cada guía de stack

- **Estructura de carpetas** del microservicio en ese lenguaje
- **Dependencias** principales por capa (dominio, aplicación, infraestructura)
- **Comandos** de desarrollo, test y build
- **Ejemplo de código** de un Port, un Adapter y un Use Case en ese lenguaje
- **Convenciones de nombres** del lenguaje (camelCase, snake_case, PascalCase)
- **Setup del test runner** y ejemplos de test unitario y de integración
