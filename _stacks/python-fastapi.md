# Stack: Python + FastAPI

> Esta guía es para equipos que construyen microservicios con **Python 3.11+** y **FastAPI**.
> Gestión de paquetes: Poetry o pip + venv.

---

## Herramientas y versiones mínimas

| Herramienta | Versión | Verificar con |
|-------------|---------|--------------|
| Python | 3.11+ | `python --version` |
| Poetry | 1.7+ | `poetry --version` |
| Docker | 24+ | `docker --version` |
| Docker Compose | 2.20+ | `docker compose version` |

---

## Estructura de carpetas del microservicio (Hexagonal)

```
nombre_servicio/
├── domain/                          # Sin dependencias externas — solo stdlib de Python
│   ├── __init__.py
│   ├── entities/
│   │   ├── __init__.py
│   │   └── appointment.py           # Dataclass o clase con invariantes
│   ├── value_objects/
│   │   └── appointment_status.py    # Enum
│   ├── events/
│   │   └── appointment_created.py   # Dataclass simple
│   └── ports/
│       ├── __init__.py
│       ├── in_/                     # Ports primarios (ABCs / Protocols)
│       │   └── create_appointment_use_case.py
│       └── out/                     # Ports secundarios
│           ├── appointment_repository.py
│           └── event_publisher.py
│
├── application/                     # Orquesta el dominio
│   └── use_cases/
│       └── create_appointment.py    # implements CreateAppointmentUseCase
│
├── infrastructure/                  # Adapters — aquí viven FastAPI, SQLAlchemy, etc.
│   ├── web/                         # Adapter primario: HTTP
│   │   ├── routers/
│   │   │   └── appointment_router.py
│   │   └── schemas/                 # Pydantic models para request/response
│   │       └── appointment_schema.py
│   ├── persistence/                 # Adapter secundario: SQLAlchemy / MongoDB
│   │   ├── sqlalchemy_appointment_repository.py
│   │   └── models/
│   │       └── appointment_orm.py   # ORM model (separado del domain entity)
│   ├── messaging/                   # Adapter secundario: Kafka / RabbitMQ
│   │   └── kafka_event_publisher.py
│   └── config/                      # DI container y configuración
│       ├── dependencies.py          # FastAPI Depends() — wiring
│       └── settings.py              # Pydantic Settings (lee variables de ambiente)
│
├── tests/
│   ├── unit/
│   │   ├── domain/                  # Tests de entidades sin frameworks
│   │   └── application/             # Tests de use cases con repositorios en memoria
│   └── integration/                 # Tests con BD real (pytest + Testcontainers)
│
├── main.py                          # Punto de entrada — instancia FastAPI y registra routers
├── pyproject.toml                   # Dependencias (Poetry)
└── alembic/                         # Migraciones de BD
    └── versions/
```

**Regla de dependencias:** El módulo `domain/` no importa nada de `fastapi`, `sqlalchemy`, ni librerías externas. Solo `dataclasses`, `enum`, `abc`, `datetime`, `uuid` de stdlib.

---

## Dependencias principales (pyproject.toml)

```toml
[tool.poetry.dependencies]
python = "^3.11"

# Web (adapter primario)
fastapi = "^0.110"
uvicorn = {extras = ["standard"], version = "^0.28"}

# Validación en adapters
pydantic = "^2.x"
pydantic-settings = "^2.x"   # Para leer variables de ambiente tipadas

# Persistencia (adapter secundario) — elegir según el motor del proyecto
sqlalchemy = {extras = ["asyncio"], version = "^2.x"}
asyncpg = "^0.29"            # Driver async para PostgreSQL
# motor = "^3.x"             # Si usas MongoDB

# Migraciones
alembic = "^1.13"

# Observabilidad
opentelemetry-instrumentation-fastapi = "^0.44"
structlog = "^24.x"          # Logging estructurado

[tool.poetry.group.dev.dependencies]
pytest = "^8.x"
pytest-asyncio = "^0.23"
httpx = "^0.27"              # Cliente HTTP para tests de integración con FastAPI
testcontainers = {extras = ["postgresql"], version = "^4.x"}
```

---

## Ejemplo: Port de entrada (Protocol de Python)

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
        """Retorna el ID de la cita creada."""
        ...
```

## Ejemplo: Use Case (Application layer)

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

## Ejemplo: Router FastAPI (Adapter primario)

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

## Test unitario de dominio (pytest — sin frameworks)

```python
# tests/unit/domain/test_appointment.py
import pytest
from datetime import datetime, timedelta
from domain.entities.appointment import Appointment

def test_rejects_past_scheduled_dates():
    yesterday = datetime.now() - timedelta(days=1)
    with pytest.raises(ValueError, match="La fecha no puede ser en el pasado"):
        Appointment.create(patient_id="p1", doctor_id="d1", scheduled_at=yesterday)
```

## Test de Use Case con repositorio en memoria

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

## Comandos del proyecto

```bash
# Instalar dependencias
poetry install

# Ejecutar localmente
uvicorn main:app --reload --port 8081

# Tests
pytest                           # todos los tests
pytest tests/unit/               # solo unitarios
pytest tests/integration/        # solo integración (requiere Docker)
pytest --cov=. --cov-report=html # con cobertura

# Migraciones
alembic upgrade head             # aplicar todas las migraciones
alembic revision --autogenerate -m "nombre_migracion"  # generar migración

# Lint y formato
ruff check .                     # linter
ruff format .                    # formateo (similar a black)
```

---

## Convenciones de nombres (Python)

| Artefacto | Convención | Ejemplo |
|-----------|-----------|---------|
| Clases | `PascalCase` | `AppointmentRepository` |
| Archivos de módulo | `snake_case.py` | `appointment_repository.py` |
| Funciones y variables | `snake_case` | `scheduled_at`, `find_by_id` |
| Constantes | `SCREAMING_SNAKE_CASE` | `MAX_APPOINTMENTS_PER_DAY` |
| Enums | `PascalCase` con valores `SCREAMING_SNAKE_CASE` | `AppointmentStatus.CONFIRMED` |
| Carpetas de paquete | `snake_case/` con `__init__.py` | `use_cases/` |

---

## Correlaciones con documentos del scaffold

- Conceptos de hexagonal → `05-architecture/hexagonal-architecture.md`
- TDD y test doubles → `11-quality/tdd-guide.md`
- Guía de patrones (conceptos) → `05-architecture/pattern-guide.md`
- Setup local → `10-devops/local-setup.md`
