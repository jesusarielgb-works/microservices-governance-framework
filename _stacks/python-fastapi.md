# Stack: Python + FastAPI

> This guide is for teams building microservices with **Python 3.11+** and **FastAPI**.
> Package management: Poetry or pip + venv.

---

## Tools and minimum versions

| Tool | Version | Verify with |
|------|---------|------------|
| Python | 3.11+ | `python --version` |
| Poetry | 1.7+ | `poetry --version` |
| Docker | 24+ | `docker --version` |
| Docker Compose | 2.20+ | `docker compose version` |

---

## Microservice folder structure (Hexagonal)

```
service_name/
├── domain/                          # No external dependencies — only Python stdlib
│   ├── __init__.py
│   ├── entities/
│   │   ├── __init__.py
│   │   └── appointment.py           # Dataclass or class with invariants
│   ├── value_objects/
│   │   └── appointment_status.py    # Enum
│   ├── events/
│   │   └── appointment_created.py   # Simple dataclass
│   └── ports/
│       ├── __init__.py
│       ├── in_/                     # Primary ports (ABCs / Protocols)
│       │   └── create_appointment_use_case.py
│       └── out/                     # Secondary ports
│           ├── appointment_repository.py
│           └── event_publisher.py
│
├── application/                     # Orchestrates the domain
│   └── use_cases/
│       └── create_appointment.py    # implements CreateAppointmentUseCase
│
├── infrastructure/                  # Adapters — FastAPI, SQLAlchemy, etc. live here
│   ├── web/                         # Primary adapter: HTTP
│   │   ├── routers/
│   │   │   └── appointment_router.py
│   │   └── schemas/                 # Pydantic models for request/response
│   │       └── appointment_schema.py
│   ├── persistence/                 # Secondary adapter: SQLAlchemy / MongoDB
│   │   ├── sqlalchemy_appointment_repository.py
│   │   └── models/
│   │       └── appointment_orm.py   # ORM model (separate from domain entity)
│   ├── messaging/                   # Secondary adapter: Kafka / RabbitMQ
│   │   └── kafka_event_publisher.py
│   └── config/                      # DI container and configuration
│       ├── dependencies.py          # FastAPI Depends() — wiring
│       └── settings.py              # Pydantic Settings (reads environment variables)
│
├── tests/
│   ├── unit/
│   │   ├── domain/                  # Entity tests without frameworks
│   │   └── application/             # Use case tests with in-memory repositories
│   └── integration/                 # Tests with real DB (pytest + Testcontainers)
│
├── main.py                          # Entry point — instantiates FastAPI and registers routers
├── pyproject.toml                   # Dependencies (Poetry)
└── alembic/                         # DB migrations
    └── versions/
```

**Dependency rule:** The `domain/` module does not import anything from `fastapi`, `sqlalchemy`, or external libraries. Only `dataclasses`, `enum`, `abc`, `datetime`, `uuid` from stdlib.

---

## Main dependencies (pyproject.toml)

```toml
[tool.poetry.dependencies]
python = "^3.11"

# Web (primary adapter)
fastapi = "^0.110"
uvicorn = {extras = ["standard"], version = "^0.28"}

# Validation in adapters
pydantic = "^2.x"
pydantic-settings = "^2.x"   # For reading typed environment variables

# Persistence (secondary adapter) — choose based on the project's engine
sqlalchemy = {extras = ["asyncio"], version = "^2.x"}
asyncpg = "^0.29"            # Async driver for PostgreSQL
# motor = "^3.x"             # If you use MongoDB

# Migrations
alembic = "^1.13"

# Observability
opentelemetry-instrumentation-fastapi = "^0.44"
structlog = "^24.x"          # Structured logging

[tool.poetry.group.dev.dependencies]
pytest = "^8.x"
pytest-asyncio = "^0.23"
httpx = "^0.27"              # HTTP client for integration tests with FastAPI
testcontainers = {extras = ["postgresql"], version = "^4.x"}
```

---

## Example: Input port (Python Protocol)

```python
# domain/ports/in_/create_appointment_use_case.py
from abc import ABC, abstractmethod
from dataclasses import dataclass
from datetime import datetime

@dataclass(frozen=True)
class CreateAppointmentCommand:
    patient_id: str
    doctor_id: str
    scheduled_at: datetime

class CreateAppointmentUseCase(ABC):
    @abstractmethod
    def execute(self, command: CreateAppointmentCommand) -> str:
        """Returns the ID of the created appointment."""
        ...
```

## Example: Use Case (Application layer)

```python
# application/use_cases/create_appointment.py
from domain.entities.appointment import Appointment
from domain.ports.in_.create_appointment_use_case import (
    CreateAppointmentCommand, CreateAppointmentUseCase
)
from domain.ports.out.appointment_repository import AppointmentRepository

class CreateAppointmentService(CreateAppointmentUseCase):
    def __init__(self, repository: AppointmentRepository) -> None:
        self._repository = repository

    def execute(self, command: CreateAppointmentCommand) -> str:
        appointment = Appointment.create(
            patient_id=command.patient_id,
            doctor_id=command.doctor_id,
            scheduled_at=command.scheduled_at,
        )
        self._repository.save(appointment)
        return str(appointment.id)
```

## Example: FastAPI Router (Primary adapter)

```python
# infrastructure/web/routers/appointment_router.py
from fastapi import APIRouter, Depends, status
from infrastructure.config.dependencies import get_create_appointment_use_case
from infrastructure.web.schemas.appointment_schema import (
    CreateAppointmentRequest, AppointmentResponse
)

router = APIRouter(prefix="/appointments", tags=["Appointments"])

@router.post("/", status_code=status.HTTP_201_CREATED, response_model=AppointmentResponse)
async def create_appointment(
    request: CreateAppointmentRequest,
    use_case = Depends(get_create_appointment_use_case),
):
    appointment_id = use_case.execute(request.to_command())
    return AppointmentResponse(id=appointment_id)
```

---

## Domain unit test (pytest — no frameworks)

```python
# tests/unit/domain/test_appointment.py
import pytest
from datetime import datetime, timedelta
from domain.entities.appointment import Appointment

def test_rejects_past_scheduled_dates():
    yesterday = datetime.now() - timedelta(days=1)
    with pytest.raises(ValueError, match="The date cannot be in the past"):
        Appointment.create(patient_id="p1", doctor_id="d1", scheduled_at=yesterday)
```

## Use Case test with in-memory repository

```python
# tests/unit/application/test_create_appointment.py
from datetime import datetime, timedelta
from application.use_cases.create_appointment import CreateAppointmentService
from domain.ports.in_.create_appointment_use_case import CreateAppointmentCommand
from tests.support.in_memory_appointment_repository import InMemoryAppointmentRepository

def test_creates_and_persists_appointment():
    repo = InMemoryAppointmentRepository()
    use_case = CreateAppointmentService(repo)
    command = CreateAppointmentCommand(
        patient_id="p1",
        doctor_id="d1",
        scheduled_at=datetime.now() + timedelta(days=1),
    )

    appointment_id = use_case.execute(command)

    assert repo.find_by_id(appointment_id) is not None
```

---

## Project commands

```bash
# Install dependencies
poetry install

# Run locally
uvicorn main:app --reload --port 8081

# Tests
pytest                           # all tests
pytest tests/unit/               # unit only
pytest tests/integration/        # integration only (requires Docker)
pytest --cov=. --cov-report=html # with coverage

# Migrations
alembic upgrade head             # apply all migrations
alembic revision --autogenerate -m "migration_name"  # generate migration

# Lint and format
ruff check .                     # linter
ruff format .                    # formatting (similar to black)
```

---

## Naming conventions (Python)

| Artifact | Convention | Example |
|----------|-----------|---------|
| Classes | `PascalCase` | `AppointmentRepository` |
| Module files | `snake_case.py` | `appointment_repository.py` |
| Functions and variables | `snake_case` | `scheduled_at`, `find_by_id` |
| Constants | `SCREAMING_SNAKE_CASE` | `MAX_APPOINTMENTS_PER_DAY` |
| Enums | `PascalCase` with `SCREAMING_SNAKE_CASE` values | `AppointmentStatus.CONFIRMED` |
| Package folders | `snake_case/` with `__init__.py` | `use_cases/` |

---

## Correlations with scaffold documents

- Hexagonal concepts → `05-architecture/hexagonal-architecture.md`
- TDD and test doubles → `11-quality/tdd-guide.md`
- Pattern guide (concepts) → `05-architecture/pattern-guide.md`
- Local setup → `10-devops/local-setup.md`
