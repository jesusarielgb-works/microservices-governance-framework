# Stack-Specific Implementation Guides

> This documentation scaffold is **technology-agnostic**: the concepts of DDD,
> hexagonal architecture, TDD, and patterns work the same in Java, Python, Go, or Node.js.
>
> This folder contains **stack-specific implementation guides**: real folder structure,
> concrete commands, recommended libraries, and code examples.
> The main documents tell you the **what and why**; your stack guide tells you the **concrete how**.

---

## Available guides

| Stack | File | When to use it |
|-------|------|----------------|
| Node.js + TypeScript | [node-typescript.md](./node-typescript.md) | Backend in Express/Fastify/NestJS with TypeScript |
| Java + Spring Boot | [java-spring.md](./java-spring.md) | Backend in Spring Boot 3.x with Maven or Gradle |
| Python + FastAPI | [python-fastapi.md](./python-fastapi.md) | Backend in FastAPI/Django with Python 3.10+ |
| Go | [go.md](./go.md) | Backend in Go with net/http or Gin/Echo |

---

## How to use these guides

1. Choose your project's stack
2. Read your stack guide — it will give you the exact folder structure for that language
3. Use that structure when filling in `09-microservices/services/NN-your-service/`
4. The code examples in the main documents (hexagonal, TDD, patterns) are
   conceptual; translate them to your stack using the conventions in this guide

---

## Why all examples use the same domain

All 4 stack guides use the same example domain: **medical appointments system**
(entity `Appointment`, `Patient`, `Doctor`). This is intentional: it allows comparing
how the same logic is implemented in Java, Python, Go, and Node.js without the difference
in domain confusing the comparison.

**Your team must adapt the concepts to your specific domain.** If you are building an
inventory, billing, or logistics system, the patterns are identical — only the entity
names and their business invariants change.

---

## Don't see your stack?

If your project uses an unlisted technology (Ruby on Rails, .NET, Kotlin, Rust, etc.),
use `node-typescript.md` as a structural reference and adapt the concepts to your
language's conventions. The hexagonal layer logic is the same across all of them.

---

## What each stack guide contains

- **Folder structure** of the microservice in that language
- **Dependencies** per layer (domain, application, infrastructure)
- **Commands** for development, testing, and build
- **Code example** of a Port, an Adapter, and a Use Case in that language
- **Naming conventions** of the language (camelCase, snake_case, PascalCase)
- **Test runner setup** and unit/integration test examples
