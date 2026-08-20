# Stack: Node.js + TypeScript

> Esta guía es para equipos que construyen microservicios con **Node.js** y **TypeScript**.
> Frameworks típicos: Express, Fastify, NestJS.

---

## Herramientas y versiones mínimas

| Herramienta | Versión | Verificar con |
|-------------|---------|--------------|
| Node.js | 20 LTS | `node --version` |
| npm | 10+ | `npm --version` |
| TypeScript | 5.x | `npx tsc --version` |
| Docker | 24+ | `docker --version` |
| Docker Compose | 2.20+ | `docker compose version` |

---

## Estructura de carpetas del microservicio (Hexagonal)

```
src/
├── domain/                     # Sin dependencias externas — el corazón del sistema
│   ├── entities/               # Entidades y Aggregates
│   │   └── MyEntity.ts
│   ├── value-objects/          # Value Objects inmutables
│   │   └── MyValueObject.ts
│   ├── events/                 # Domain Events (interfaces o clases simples)
│   │   └── MyEntityCreated.ts
│   ├── ports/                  # Interfaces que el dominio define
│   │   ├── in/                 # Ports primarios (casos de uso)
│   │   │   └── ICreateMyEntityUseCase.ts
│   │   └── out/                # Ports secundarios (repositorios, servicios externos)
│   │       ├── IMyEntityRepository.ts
│   │       └── IEventPublisher.ts
│   └── services/               # Domain Services (lógica que no cabe en una entidad)
│
├── application/                # Orquesta el dominio — solo depende de domain/
│   └── use-cases/
│       └── CreateMyEntityUseCase.ts
│
├── infrastructure/             # Adapters — depende de application/ y domain/
│   ├── http/                   # Adapter primario: HTTP
│   │   ├── controllers/
│   │   │   └── MyEntityController.ts
│   │   └── routes/
│   │       └── myEntity.routes.ts
│   ├── persistence/            # Adapter secundario: Base de datos
│   │   └── MyEntityRepository.ts  # implements IMyEntityRepository
│   ├── messaging/              # Adapter secundario: Broker de mensajes
│   │   └── KafkaEventPublisher.ts # implements IEventPublisher
│   └── external/               # Adapter secundario: APIs externas
│
└── main.ts                     # Bootstrap: wiring de dependencias (DI manual o container)
```

**Regla de dependencias:** `infrastructure → application → domain`. El dominio no importa nada de fuera.

---

## Dependencias por capa

```json
// package.json — dependencias de producción
{
  "dependencies": {
    // Infraestructura HTTP
    "express": "^4.x",          // o "fastify": "^4.x"
    "@types/express": "^4.x",

    // Infraestructura de datos (elegir uno)
    "typeorm": "^0.3.x",        // ORM
    "pg": "^8.x",               // Driver PostgreSQL
    // "mongoose": "^8.x",       // Si usas MongoDB
    // "ioredis": "^5.x",        // Si usas Redis

    // Infraestructura de mensajería (si aplica)
    // "kafkajs": "^2.x",
    // "amqplib": "^0.10.x",

    // Validación (adapter primario)
    "zod": "^3.x",              // o "class-validator": "^0.14.x"

    // Observabilidad
    "@opentelemetry/sdk-node": "^0.x",
    "pino": "^8.x"              // Logger estructurado
  },
  "devDependencies": {
    // TypeScript
    "typescript": "^5.x",
    "ts-node": "^10.x",
    "@types/node": "^20.x",

    // Testing
    "jest": "^29.x",
    "ts-jest": "^29.x",
    "@types/jest": "^29.x",
    "supertest": "^6.x",        // Tests de integración HTTP
    "testcontainers": "^10.x",  // Tests de integración con DB real

    // Linting
    "eslint": "^8.x",
    "@typescript-eslint/parser": "^6.x"
  }
}
```

---

## Ejemplo: Port de entrada (Use Case Interface)

```typescript
// src/domain/ports/in/ICreateAppointmentUseCase.ts

export interface CreateAppointmentCommand {
  patientId: string;
  doctorId: string;
  scheduledAt: Date;
}

export interface ICreateAppointmentUseCase {
  execute(command: CreateAppointmentCommand): Promise<string>; // retorna el ID
}
```

## Ejemplo: Port de salida (Repository Interface)

```typescript
// src/domain/ports/out/IAppointmentRepository.ts

import { Appointment } from '../entities/Appointment';

export interface IAppointmentRepository {
  save(appointment: Appointment): Promise<void>;
  findById(id: string): Promise<Appointment | null>;
  findByPatient(patientId: string): Promise<Appointment[]>;
}
```

## Ejemplo: Use Case (Application layer)

```typescript
// src/application/use-cases/CreateAppointmentUseCase.ts

import { ICreateAppointmentUseCase, CreateAppointmentCommand } from '../../domain/ports/in/ICreateAppointmentUseCase';
import { IAppointmentRepository } from '../../domain/ports/out/IAppointmentRepository';
import { Appointment } from '../../domain/entities/Appointment';

export class CreateAppointmentUseCase implements ICreateAppointmentUseCase {
  constructor(private readonly repository: IAppointmentRepository) {}

  async execute(command: CreateAppointmentCommand): Promise<string> {
    const appointment = Appointment.create(command); // invariantes en el factory
    await this.repository.save(appointment);
    return appointment.id;
  }
}
```

---

## Test unitario de dominio (Jest)

```typescript
// src/domain/entities/__tests__/Appointment.test.ts

describe('Appointment', () => {
  it('should reject past scheduled dates', () => {
    const yesterday = new Date(Date.now() - 86400000);
    expect(() =>
      Appointment.create({ patientId: 'p1', doctorId: 'd1', scheduledAt: yesterday })
    ).toThrow('La fecha no puede ser en el pasado');
  });
});
```

## Test de Use Case con Fake (sin BD real)

```typescript
// src/application/use-cases/__tests__/CreateAppointmentUseCase.test.ts

import { InMemoryAppointmentRepository } from '../../../test-support/InMemoryAppointmentRepository';

describe('CreateAppointmentUseCase', () => {
  it('should create and persist an appointment', async () => {
    const repo = new InMemoryAppointmentRepository();
    const useCase = new CreateAppointmentUseCase(repo);

    const id = await useCase.execute({
      patientId: 'p1',
      doctorId: 'd1',
      scheduledAt: new Date(Date.now() + 86400000),
    });

    expect(repo.findById(id)).toBeDefined();
  });
});
```

---

## Comandos del proyecto

```bash
# Instalar dependencias
npm install

# Desarrollo con hot-reload
npm run dev          # ts-node-dev src/main.ts

# Tests
npm test             # jest
npm run test:watch   # jest --watch
npm run test:cov     # jest --coverage

# Build para producción
npm run build        # tsc --project tsconfig.build.json

# Lint
npm run lint         # eslint src/**/*.ts
```

---

## Convenciones de nombres (TypeScript)

| Artefacto | Convención | Ejemplo |
|-----------|-----------|---------|
| Interfaces | `PascalCase` con prefijo `I` | `IAppointmentRepository` |
| Clases | `PascalCase` | `CreateAppointmentUseCase` |
| Archivos | `PascalCase.ts` para clases, `kebab-case.ts` para módulos | `Appointment.ts`, `appointment.routes.ts` |
| Variables y funciones | `camelCase` | `scheduledAt`, `findById` |
| Constantes globales | `SCREAMING_SNAKE_CASE` | `MAX_APPOINTMENTS_PER_DAY` |
| Enums | `PascalCase` con valores `SCREAMING_SNAKE_CASE` | `AppointmentStatus.CONFIRMED` |

---

## Correlaciones con documentos del scaffold

- Conceptos de hexagonal → `05-architecture/hexagonal-architecture.md`
- TDD y test doubles → `11-quality/tdd-guide.md`
- Guía de patrones (conceptos) → `05-architecture/pattern-guide.md`
- Setup local → `10-devops/local-setup.md`
