# Stack: Java + Spring Boot

> Esta guía es para equipos que construyen microservicios con **Java** y **Spring Boot 3.x**.
> Build tools: Maven o Gradle.

---

## Herramientas y versiones mínimas

| Herramienta | Versión | Verificar con |
|-------------|---------|--------------|
| JDK | 21 LTS | `java --version` |
| Maven | 3.9+ | `mvn --version` |
| Gradle | 8.x | `gradle --version` |
| Docker | 24+ | `docker --version` |
| Docker Compose | 2.20+ | `docker compose version` |

---

## Estructura de carpetas del microservicio (Hexagonal)

```
src/
├── main/
│   └── java/com/empresa/servicio/
│       ├── domain/                          # Sin dependencias de Spring — POJO puro
│       │   ├── model/                       # Entidades y Value Objects
│       │   │   ├── Appointment.java         # Aggregate root
│       │   │   └── AppointmentStatus.java   # Enum de estados
│       │   ├── event/                       # Domain Events
│       │   │   └── AppointmentCreated.java
│       │   ├── port/
│       │   │   ├── in/                      # Ports primarios (interfaces de casos de uso)
│       │   │   │   └── CreateAppointmentUseCase.java
│       │   │   └── out/                     # Ports secundarios
│       │   │       ├── AppointmentRepository.java
│       │   │       └── EventPublisher.java
│       │   └── service/                     # Domain Services
│       │
│       ├── application/                     # Orquesta — usa Spring para DI pero no frameworks web
│       │   └── usecase/
│       │       └── CreateAppointmentService.java   # implements CreateAppointmentUseCase
│       │
│       └── infrastructure/                  # Adapters — aquí vive Spring Web, JPA, Kafka
│           ├── web/                         # Adapter primario: REST
│           │   ├── AppointmentController.java
│           │   └── dto/
│           │       ├── CreateAppointmentRequest.java
│           │       └── AppointmentResponse.java
│           ├── persistence/                 # Adapter secundario: JPA/JDBC
│           │   ├── JpaAppointmentRepository.java   # implements AppointmentRepository
│           │   └── entity/
│           │       └── AppointmentJpaEntity.java   # @Entity — separada del domain model
│           ├── messaging/                   # Adapter secundario: Kafka/RabbitMQ
│           │   └── KafkaEventPublisher.java        # implements EventPublisher
│           └── config/                      # Spring @Configuration — wiring de dependencias
│               └── AppConfig.java
│
└── test/
    └── java/com/empresa/servicio/
        ├── domain/                          # Tests de dominio — sin Spring
        ├── application/                     # Tests de Use Case — sin Spring (Fake repos)
        └── infrastructure/                  # Tests de integración — con Spring + Testcontainers
            └── web/
            └── persistence/
```

**Regla de dependencias:** El paquete `domain` no importa nada de `org.springframework.*` ni de `jakarta.persistence.*`. Solo POJOs.

---

## Dependencias principales (pom.xml)

```xml
<dependencies>
  <!-- ─── Web (adapter primario) ─────────────────────────────────── -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>

  <!-- ─── Validación de requests ──────────────────────────────────── -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
  </dependency>

  <!-- ─── Persistencia (adapter secundario) ─────────────────────── -->
  <!-- Elegir uno: JPA, JDBC, o acceso nativo al driver -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>

  <!-- ─── Motor de BD (elegir el que corresponda al proyecto) ─────── -->
  <!-- PostgreSQL -->
  <dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
  </dependency>

  <!-- ─── Migraciones de BD ────────────────────────────────────────── -->
  <dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
  </dependency>

  <!-- ─── Observabilidad ──────────────────────────────────────────── -->
  <dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
  </dependency>

  <!-- ─── Testing ─────────────────────────────────────────────────── -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
    <!-- Incluye JUnit 5, Mockito, AssertJ, Testcontainers -->
  </dependency>
  <dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
```

---

## Ejemplo: Port de entrada (Use Case Interface)

```java
// domain/port/in/CreateAppointmentUseCase.java
package com.empresa.servicio.domain.port.in;

import java.time.LocalDateTime;

public interface CreateAppointmentUseCase {
    record CreateAppointmentCommand(
        String patientId,
        String doctorId,
        LocalDateTime scheduledAt
    ) {}

    String execute(CreateAppointmentCommand command);
}
```

## Ejemplo: Port de salida (Repository Interface)

```java
// domain/port/out/AppointmentRepository.java
package com.empresa.servicio.domain.port.out;

import com.empresa.servicio.domain.model.Appointment;
import java.util.Optional;

public interface AppointmentRepository {
    void save(Appointment appointment);
    Optional<Appointment> findById(String id);
}
```

## Ejemplo: Use Case (Application layer)

```java
// application/usecase/CreateAppointmentService.java
package com.empresa.servicio.application.usecase;

import com.empresa.servicio.domain.model.Appointment;
import com.empresa.servicio.domain.port.in.CreateAppointmentUseCase;
import com.empresa.servicio.domain.port.out.AppointmentRepository;
import org.springframework.stereotype.Service;

@Service  // Spring gestiona el ciclo de vida; el interface es del dominio
public class CreateAppointmentService implements CreateAppointmentUseCase {

    private final AppointmentRepository repository;

    public CreateAppointmentService(AppointmentRepository repository) {
        this.repository = repository;
    }

    @Override
    public String execute(CreateAppointmentCommand command) {
        Appointment appointment = Appointment.create(
            command.patientId(), command.doctorId(), command.scheduledAt()
        );
        repository.save(appointment);
        return appointment.getId();
    }
}
```

---

## Test unitario de dominio (JUnit 5 — sin Spring)

```java
// test/domain/model/AppointmentTest.java
class AppointmentTest {

    @Test
    void should_reject_past_scheduled_dates() {
        LocalDateTime yesterday = LocalDateTime.now().minusDays(1);

        assertThatThrownBy(() ->
            Appointment.create("p1", "d1", yesterday)
        ).isInstanceOf(IllegalArgumentException.class)
         .hasMessage("La fecha no puede ser en el pasado");
    }
}
```

## Test de Use Case con Fake (sin Spring, sin BD)

```java
// test/application/usecase/CreateAppointmentServiceTest.java
class CreateAppointmentServiceTest {

    private InMemoryAppointmentRepository repository;
    private CreateAppointmentUseCase useCase;

    @BeforeEach
    void setUp() {
        repository = new InMemoryAppointmentRepository();
        useCase = new CreateAppointmentService(repository);
    }

    @Test
    void should_create_and_persist_appointment() {
        var command = new CreateAppointmentCommand(
            "p1", "d1", LocalDateTime.now().plusDays(1)
        );

        String id = useCase.execute(command);

        assertThat(repository.findById(id)).isPresent();
    }
}
```

---

## Comandos del proyecto

```bash
# Compilar
mvn compile                  # o: gradle build

# Tests
mvn test                     # o: gradle test
mvn test -pl application     # Solo tests de una capa

# Tests de integración (requiere Docker para Testcontainers)
mvn verify

# Build JAR para producción
mvn package -DskipTests      # genera target/nombre-servicio.jar

# Ejecutar localmente
java -jar target/nombre-servicio.jar

# Con Spring profiles
java -jar -Dspring.profiles.active=local target/nombre-servicio.jar
```

---

## Convenciones de nombres (Java)

| Artefacto | Convención | Ejemplo |
|-----------|-----------|---------|
| Interfaces | `PascalCase` (sin prefijo I) | `AppointmentRepository` |
| Clases | `PascalCase` | `CreateAppointmentService` |
| Paquetes | `lowercase.separado.por.puntos` | `com.empresa.servicio.domain.model` |
| Variables y métodos | `camelCase` | `scheduledAt`, `findById` |
| Constantes | `SCREAMING_SNAKE_CASE` | `MAX_APPOINTMENTS_PER_DAY` |
| Enums | `PascalCase` con valores `SCREAMING_SNAKE_CASE` | `AppointmentStatus.CONFIRMED` |

---

## Correlaciones con documentos del scaffold

- Conceptos de hexagonal → `05-architecture/hexagonal-architecture.md`
- TDD y test doubles → `11-quality/tdd-guide.md`
- Guía de patrones (conceptos) → `05-architecture/pattern-guide.md`
- Setup local → `10-devops/local-setup.md`
