# Stack: Go

> Esta guía es para equipos que construyen microservicios con **Go 1.21+**.
> Build tool: `go` (nativo). Frameworks HTTP comunes: `net/http` nativo, Gin, Echo, Chi.

---

## Herramientas y versiones mínimas

| Herramienta | Versión | Verificar con |
|-------------|---------|--------------|
| Go | 1.21+ | `go version` |
| Docker | 24+ | `docker --version` |
| Docker Compose | 2.20+ | `docker compose version` |

---

## Estructura de carpetas del microservicio (Hexagonal)

```
nombre-servicio/
├── internal/                        # Código interno — no exportable como librería
│   ├── domain/                      # Sin dependencias externas — solo stdlib de Go
│   │   ├── entity.go                # Entidades y Aggregates (structs con métodos)
│   │   ├── value_object.go          # Value Objects (structs inmutables)
│   │   ├── event.go                 # Domain Events (structs simples)
│   │   └── port/
│   │       ├── in.go                # Interfaces de Use Cases (ports primarios)
│   │       └── out.go               # Interfaces de repositorios/servicios (ports secundarios)
│   │
│   ├── application/                 # Orquesta el dominio
│   │   └── usecase/
│   │       └── create_appointment.go   # implements el interface del port in
│   │
│   └── infrastructure/              # Adapters — aquí viven gin, pgx, kafka-go
│       ├── http/                    # Adapter primario: HTTP
│       │   ├── handler/
│       │   │   └── appointment_handler.go
│       │   └── dto/
│       │       └── appointment_dto.go
│       ├── postgres/                # Adapter secundario: PostgreSQL
│       │   └── appointment_repository.go   # implements port/out.AppointmentRepository
│       ├── kafka/                   # Adapter secundario: Kafka (si aplica)
│       │   └── event_publisher.go
│       └── config/
│           └── wire.go              # Wiring manual de dependencias
│
├── cmd/
│   └── server/
│       └── main.go                  # Punto de entrada: configura e inicia el servidor
│
├── migrations/                      # Scripts SQL de migración (con golang-migrate)
│   ├── 000001_create_appointments.up.sql
│   └── 000001_create_appointments.down.sql
│
├── go.mod
├── go.sum
└── Makefile                         # Comandos de uso frecuente
```

**Regla de dependencias:** El paquete `internal/domain` no importa nada de `internal/infrastructure`. La dependencia es siempre hacia adentro.

---

## Dependencias principales (go.mod)

```go
module github.com/empresa/nombre-servicio

go 1.21

require (
    // HTTP (elegir uno)
    github.com/gin-gonic/gin v1.9.x
    // github.com/labstack/echo/v4 v4.x
    // github.com/go-chi/chi/v5 v5.x

    // Validación en adapters HTTP
    github.com/go-playground/validator/v10 v10.x

    // Persistencia (elegir el driver del motor del proyecto)
    github.com/jackc/pgx/v5 v5.x            // PostgreSQL
    // go.mongodb.org/mongo-driver v1.x      // MongoDB

    // Migraciones
    github.com/golang-migrate/migrate/v4 v4.x

    // Observabilidad
    go.opentelemetry.io/otel v1.x
    go.uber.org/zap v1.x                    // Logger estructurado
)
```

---

## Ejemplo: Interfaces del dominio (ports)

```go
// internal/domain/port/in.go
package port

import (
    "context"
    "time"
)

type CreateAppointmentCommand struct {
    PatientID   string
    DoctorID    string
    ScheduledAt time.Time
}

// Port primario — lo implementa el Use Case
type AppointmentCreator interface {
    Create(ctx context.Context, cmd CreateAppointmentCommand) (string, error)
}
```

```go
// internal/domain/port/out.go
package port

import (
    "context"
    "github.com/empresa/nombre-servicio/internal/domain"
)

// Port secundario — lo implementa el adapter de infraestructura
type AppointmentRepository interface {
    Save(ctx context.Context, a domain.Appointment) error
    FindByID(ctx context.Context, id string) (*domain.Appointment, error)
}
```

## Ejemplo: Use Case (Application layer)

```go
// internal/application/usecase/create_appointment.go
package usecase

import (
    "context"
    "github.com/empresa/nombre-servicio/internal/domain"
    "github.com/empresa/nombre-servicio/internal/domain/port"
)

type createAppointmentUseCase struct {
    repo port.AppointmentRepository
}

// Constructor — recibe interfaces, no implementaciones concretas
func NewCreateAppointment(repo port.AppointmentRepository) port.AppointmentCreator {
    return &createAppointmentUseCase{repo: repo}
}

func (uc *createAppointmentUseCase) Create(ctx context.Context, cmd port.CreateAppointmentCommand) (string, error) {
    appointment, err := domain.NewAppointment(cmd.PatientID, cmd.DoctorID, cmd.ScheduledAt)
    if err != nil {
        return "", err
    }
    if err := uc.repo.Save(ctx, appointment); err != nil {
        return "", err
    }
    return appointment.ID, nil
}
```

---

## Test unitario de dominio (testing nativo de Go)

```go
// internal/domain/appointment_test.go
package domain_test

import (
    "testing"
    "time"
    "github.com/empresa/nombre-servicio/internal/domain"
)

func TestAppointment_RejectsPastDate(t *testing.T) {
    yesterday := time.Now().AddDate(0, 0, -1)
    _, err := domain.NewAppointment("p1", "d1", yesterday)
    if err == nil {
        t.Fatal("se esperaba error para fecha pasada")
    }
}
```

## Test de Use Case con fake repository

```go
// internal/application/usecase/create_appointment_test.go
package usecase_test

import (
    "context"
    "testing"
    "time"
    "github.com/empresa/nombre-servicio/internal/application/usecase"
    "github.com/empresa/nombre-servicio/internal/domain/port"
    "github.com/empresa/nombre-servicio/internal/testutil"
)

func TestCreateAppointment(t *testing.T) {
    repo := testutil.NewInMemoryAppointmentRepository()
    uc := usecase.NewCreateAppointment(repo)

    id, err := uc.Create(context.Background(), port.CreateAppointmentCommand{
        PatientID:   "p1",
        DoctorID:    "d1",
        ScheduledAt: time.Now().AddDate(0, 0, 1),
    })

    if err != nil {
        t.Fatalf("no se esperaba error: %v", err)
    }
    if _, err := repo.FindByID(context.Background(), id); err != nil {
        t.Fatalf("cita no encontrada en repositorio: %v", err)
    }
}
```

---

## Makefile con comandos frecuentes

```makefile
.PHONY: dev test build lint migrate

dev:
	go run ./cmd/server/...

test:
	go test ./...

test-cover:
	go test -coverprofile=coverage.out ./...
	go tool cover -html=coverage.out

test-integration:
	go test -tags=integration ./...

build:
	go build -o bin/server ./cmd/server/...

lint:
	golangci-lint run ./...

migrate-up:
	migrate -path migrations/ -database "$$DATABASE_URL" up

migrate-down:
	migrate -path migrations/ -database "$$DATABASE_URL" down 1
```

---

## Convenciones de nombres (Go)

| Artefacto | Convención | Ejemplo |
|-----------|-----------|---------|
| Interfaces | `PascalCase` (sin prefijo I) | `AppointmentRepository` |
| Structs | `PascalCase` | `CreateAppointmentCommand` |
| Archivos | `snake_case.go` | `appointment_repository.go` |
| Variables y funciones | `camelCase` | `scheduledAt`, `findByID` |
| Constantes | `PascalCase` (si exportadas) o `camelCase` (privadas) | `MaxAppointmentsPerDay` |
| Paquetes | `lowercase` sin guiones | `usecase`, `persistence` |

---

## Correlaciones con documentos del scaffold

- Conceptos de hexagonal → `05-architecture/hexagonal-architecture.md`
- TDD y test doubles → `11-quality/tdd-guide.md`
- Guía de patrones (conceptos) → `05-architecture/pattern-guide.md`
- Setup local → `10-devops/local-setup.md`
