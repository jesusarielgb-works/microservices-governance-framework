# Stack: Node.js + TypeScript

> This guide is for teams building microservices with **Node.js** and **TypeScript**.
> Typical frameworks: Express, Fastify, NestJS.

---

## Tools and minimum versions

| Tool | Version | Verify with |
|------|---------|------------|
| Node.js | 20 LTS | `node --version` |
| npm | 10+ | `npm --version` |
| TypeScript | 5.x | `npx tsc --version` |
| Docker | 24+ | `docker --version` |
| Docker Compose | 2.20+ | `docker compose version` |

---

## Microservice folder structure (Hexagonal)

```
src/
├── domain/                     # No external dependencies — the heart of the system
│   ├── entities/               # Entities and Aggregates
│   │   └── MyEntity.ts
│   ├── value-objects/          # Immutable Value Objects
│   │   └── MyValueObject.ts
│   ├── events/                 # Domain Events (interfaces or simple classes)
│   │   └── MyEntityCreated.ts
│   ├── ports/                  # Interfaces defined by the domain
│   │   ├── in/                 # Primary ports (use cases)
│   │   │   └── ICreateMyEntityUseCase.ts
│   │   └── out/                # Secondary ports (repositories, external services)
│   │       ├── IMyEntityRepository.ts
│   │       └── IEventPublisher.ts
│   └── services/               # Domain Services (logic that doesn't fit in one entity)
│
├── application/                # Orchestrates the domain — only depends on domain/
│   └── use-cases/
│       └── CreateMyEntityUseCase.ts
│
├── infrastructure/             # Adapters — depends on application/ and domain/
│   ├── http/                   # Primary adapter: HTTP
│   │   ├── controllers/
│   │   │   └── MyEntityController.ts
│   │   └── routes/
│   │       └── myEntity.routes.ts
│   ├── persistence/            # Secondary adapter: Database
│   │   └── MyEntityRepository.ts  # implements IMyEntityRepository
│   ├── messaging/              # Secondary adapter: Message broker
│   │   └── KafkaEventPublisher.ts # implements IEventPublisher
│   └── external/               # Secondary adapter: External APIs
│
└── main.ts                     # Bootstrap: dependency wiring (manual DI or container)
```

**Dependency rule:** `infrastructure → application → domain`. The domain does not import anything from outside.

---

## Dependencies by layer

```json
// package.json — production dependencies
{
  "dependencies": {
    // HTTP infrastructure
    "express": "^4.x",          // or "fastify": "^4.x"
    "@types/express": "^4.x",

    // Data infrastructure (choose one)
    "typeorm": "^0.3.x",        // ORM
    "pg": "^8.x",               // PostgreSQL driver
    // "mongoose": "^8.x",       // If you use MongoDB
    // "ioredis": "^5.x",        // If you use Redis

    // Messaging infrastructure (if applicable)
    // "kafkajs": "^2.x",
    // "amqplib": "^0.10.x",

    // Validation (primary adapter)
    "zod": "^3.x",              // or "class-validator": "^0.14.x"

    // Observability
    "@opentelemetry/sdk-node": "^0.x",
    "pino": "^8.x"              // Structured logger
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
    "supertest": "^6.x",        // HTTP integration tests
    "testcontainers": "^10.x",  // Integration tests with real DB

    // Linting
    "eslint": "^8.x",
    "@typescript-eslint/parser": "^6.x"
  }
}
```

---

## Example: Input port (Use Case Interface)

```typescript
// src/domain/ports/in/ICreateAppointmentUseCase.ts

export interface CreateAppointmentCommand {
  patientId: string;
  doctorId: string;
  scheduledAt: Date;
}

export interface ICreateAppointmentUseCase {
  execute(command: CreateAppointmentCommand): Promise<string>; // returns the ID
}
```

## Example: Output port (Repository Interface)

```typescript
// src/domain/ports/out/IAppointmentRepository.ts

import { Appointment } from '../entities/Appointment';

export interface IAppointmentRepository {
  save(appointment: Appointment): Promise<void>;
  findById(id: string): Promise<Appointment | null>;
  findByPatient(patientId: string): Promise<Appointment[]>;
}
```

## Example: Use Case (Application layer)

```typescript
// src/application/use-cases/CreateAppointmentUseCase.ts

import { ICreateAppointmentUseCase, CreateAppointmentCommand } from '../../domain/ports/in/ICreateAppointmentUseCase';
import { IAppointmentRepository } from '../../domain/ports/out/IAppointmentRepository';
import { Appointment } from '../../domain/entities/Appointment';

export class CreateAppointmentUseCase implements ICreateAppointmentUseCase {
  constructor(private readonly repository: IAppointmentRepository) {}

  async execute(command: CreateAppointmentCommand): Promise<string> {
    const appointment = Appointment.create(command); // invariants in the factory
    await this.repository.save(appointment);
    return appointment.id;
  }
}
```

---

## Domain unit test (Jest)

```typescript
// src/domain/entities/__tests__/Appointment.test.ts

describe('Appointment', () => {
  it('should reject past scheduled dates', () => {
    const yesterday = new Date(Date.now() - 86400000);
    expect(() =>
      Appointment.create({ patientId: 'p1', doctorId: 'd1', scheduledAt: yesterday })
    ).toThrow('The date cannot be in the past');
  });
});
```

## Use Case test with Fake (no real DB)

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

## Project commands

```bash
# Install dependencies
npm install

# Development with hot-reload
npm run dev          # ts-node-dev src/main.ts

# Tests
npm test             # jest
npm run test:watch   # jest --watch
npm run test:cov     # jest --coverage

# Build for production
npm run build        # tsc --project tsconfig.build.json

# Lint
npm run lint         # eslint src/**/*.ts
```

---

## Naming conventions (TypeScript)

| Artifact | Convention | Example |
|----------|-----------|---------|
| Interfaces | `PascalCase` with `I` prefix | `IAppointmentRepository` |
| Classes | `PascalCase` | `CreateAppointmentUseCase` |
| Files | `PascalCase.ts` for classes, `kebab-case.ts` for modules | `Appointment.ts`, `appointment.routes.ts` |
| Variables and functions | `camelCase` | `scheduledAt`, `findById` |
| Global constants | `SCREAMING_SNAKE_CASE` | `MAX_APPOINTMENTS_PER_DAY` |
| Enums | `PascalCase` with `SCREAMING_SNAKE_CASE` values | `AppointmentStatus.CONFIRMED` |

---

## Correlations with scaffold documents

- Hexagonal concepts → `05-architecture/hexagonal-architecture.md`
- TDD and test doubles → `11-quality/tdd-guide.md`
- Pattern guide (concepts) → `05-architecture/pattern-guide.md`
- Local setup → `10-devops/local-setup.md`
