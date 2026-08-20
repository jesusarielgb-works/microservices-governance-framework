# Stack: Go

> This guide is for teams building microservices with **Go 1.21+**.
> Build tool: `go` (native). Common HTTP frameworks: native `net/http`, Gin, Echo, Chi.

---

## Tools and minimum versions

| Tool | Version | Verify with |
|------|---------|------------|
| Go | 1.21+ | `go version` |
| Docker | 24+ | `docker --version` |
| Docker Compose | 2.20+ | `docker compose version` |

---

## Microservice folder structure (Hexagonal)

```
service-name/
├── internal/                        # Internal code — not exportable as a library
│   ├── domain/                      # No external dependencies — only Go stdlib
│   │   ├── entity.go                # Entities and Aggregates (structs with methods)
│   │   ├── value_object.go          # Value Objects (immutable structs)
│   │   ├── event.go                 # Domain Events (simple structs)
│   │   └── port/
│   │       ├── in.go                # Use Case interfaces (primary ports)
│   │       └── out.go               # Repository/service interfaces (secondary ports)
│   │
│   ├── application/                 # Orchestrates the domain
│   │   └── usecase/
│   │       └── create_appointment.go   # implements the port in interface
│   │
│   └── infrastructure/              # Adapters — gin, pgx, kafka-go live here
│       ├── http/                    # Primary adapter: HTTP
│       │   ├── handler/
│       │   │   └── appointment_handler.go
│       │   └── dto/
│       │       └── appointment_dto.go
│       ├── postgres/                # Secondary adapter: PostgreSQL
│       │   └── appointment_repository.go   # implements port/out.AppointmentRepository
│       ├── kafka/                   # Secondary adapter: Kafka (if applicable)
│       │   └── event_publisher.go
│       └── config/
│           └── wire.go              # Manual dependency wiring
│
├── cmd/
│   └── server/
│       └── main.go                  # Entry point: configures and starts the server
│
├── migrations/                      # SQL migration scripts (with golang-migrate)
│   ├── 000001_create_appointments.up.sql
│   └── 000001_create_appointments.down.sql
│
├── go.mod
├── go.sum
└── Makefile                         # Frequently used commands
```

**Dependency rule:** The `internal/domain` package does not import anything from `internal/infrastructure`. The dependency always points inward.

---

## Main dependencies (go.mod)

```go
module github.com/company/service-name

go 1.21

require (
    // HTTP (choose one)
    github.com/gin-gonic/gin v1.9.x
    // github.com/labstack/echo/v4 v4.x
    // github.com/go-chi/chi/v5 v5.x

    // Validation in HTTP adapters
    github.com/go-playground/validator/v10 v10.x

    // Persistence (choose the driver for the project's engine)
    github.com/jackc/pgx/v5 v5.x            // PostgreSQL
    // go.mongodb.org/mongo-driver v1.x      // MongoDB

    // Migrations
    github.com/golang-migrate/migrate/v4 v4.x

    // Observability
    go.opentelemetry.io/otel v1.x
    go.uber.org/zap v1.x                    // Structured logger
)
```

---

## Example: Domain interfaces (ports)

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

// Primary port — implemented by the Use Case
type AppointmentCreator interface {
    Create(ctx context.Context, cmd CreateAppointmentCommand) (string, error)
}
```

```go
// internal/domain/port/out.go
package port

import (
    "context"
    "github.com/company/service-name/internal/domain"
)

// Secondary port — implemented by the infrastructure adapter
type AppointmentRepository interface {
    Save(ctx context.Context, a domain.Appointment) error
    FindByID(ctx context.Context, id string) (*domain.Appointment, error)
}
```

## Example: Use Case (Application layer)

```go
// internal/application/usecase/create_appointment.go
package usecase

import (
    "context"
    "github.com/company/service-name/internal/domain"
    "github.com/company/service-name/internal/domain/port"
)

type createAppointmentUseCase struct {
    repo port.AppointmentRepository
}

// Constructor — receives interfaces, not concrete implementations
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

## Domain unit test (native Go testing)

```go
// internal/domain/appointment_test.go
package domain_test

import (
    "testing"
    "time"
    "github.com/company/service-name/internal/domain"
)

func TestAppointment_RejectsPastDate(t *testing.T) {
    yesterday := time.Now().AddDate(0, 0, -1)
    _, err := domain.NewAppointment("p1", "d1", yesterday)
    if err == nil {
        t.Fatal("expected error for past date")
    }
}
```

## Use Case test with fake repository

```go
// internal/application/usecase/create_appointment_test.go
package usecase_test

import (
    "context"
    "testing"
    "time"
    "github.com/company/service-name/internal/application/usecase"
    "github.com/company/service-name/internal/domain/port"
    "github.com/company/service-name/internal/testutil"
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
        t.Fatalf("unexpected error: %v", err)
    }
    if _, err := repo.FindByID(context.Background(), id); err != nil {
        t.Fatalf("appointment not found in repository: %v", err)
    }
}
```

---

## Makefile with frequent commands

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

## Naming conventions (Go)

| Artifact | Convention | Example |
|----------|-----------|---------|
| Interfaces | `PascalCase` (no I prefix) | `AppointmentRepository` |
| Structs | `PascalCase` | `CreateAppointmentCommand` |
| Files | `snake_case.go` | `appointment_repository.go` |
| Variables and functions | `camelCase` | `scheduledAt`, `findByID` |
| Constants | `PascalCase` (if exported) or `camelCase` (private) | `MaxAppointmentsPerDay` |
| Packages | `lowercase` without hyphens | `usecase`, `persistence` |

---

## Correlations with scaffold documents

- Hexagonal concepts → `05-architecture/hexagonal-architecture.md`
- TDD and test doubles → `11-quality/tdd-guide.md`
- Pattern guide (concepts) → `05-architecture/pattern-guide.md`
- Local setup → `10-devops/local-setup.md`
