# Stack: Java + Spring Boot

> This guide is for teams building microservices with **Java** and **Spring Boot 3.x**.
> Build tools: Maven or Gradle.

---

## Tools and minimum versions

| Tool | Version | Verify with |
|------|---------|------------|
| JDK | 21 LTS | `java --version` |
| Maven | 3.9+ | `mvn --version` |
| Gradle | 8.x | `gradle --version` |
| Docker | 24+ | `docker --version` |
| Docker Compose | 2.20+ | `docker compose version` |

---

## Microservice folder structure (Hexagonal)

```
src/
├── main/
│   └── java/com/company/service/
│       ├── domain/                          # No Spring dependencies — pure POJO
│       │   ├── model/                       # Entities and Value Objects
│       │   │   ├── Appointment.java         # Aggregate root
│       │   │   └── AppointmentStatus.java   # Status enum
│       │   ├── event/                       # Domain Events
│       │   │   └── AppointmentCreated.java
│       │   ├── port/
│       │   │   ├── in/                      # Primary ports (use case interfaces)
│       │   │   │   └── CreateAppointmentUseCase.java
│       │   │   └── out/                     # Secondary ports
│       │   │       ├── AppointmentRepository.java
│       │   │       └── EventPublisher.java
│       │   └── service/                     # Domain Services
│       │
│       ├── application/                     # Orchestrates — uses Spring for DI but not web frameworks
│       │   └── usecase/
│       │       └── CreateAppointmentService.java   # implements CreateAppointmentUseCase
│       │
│       └── infrastructure/                  # Adapters — Spring Web, JPA, Kafka live here
│           ├── web/                         # Primary adapter: REST
│           │   ├── AppointmentController.java
│           │   └── dto/
│           │       ├── CreateAppointmentRequest.java
│           │       └── AppointmentResponse.java
│           ├── persistence/                 # Secondary adapter: JPA/JDBC
│           │   ├── JpaAppointmentRepository.java   # implements AppointmentRepository
│           │   └── entity/
│           │       └── AppointmentJpaEntity.java   # @Entity — separate from domain model
│           ├── messaging/                   # Secondary adapter: Kafka/RabbitMQ
│           │   └── KafkaEventPublisher.java        # implements EventPublisher
│           └── config/                      # Spring @Configuration — dependency wiring
│               └── AppConfig.java
│
└── test/
    └── java/com/company/service/
        ├── domain/                          # Domain tests — no Spring
        ├── application/                     # Use Case tests — no Spring (Fake repos)
        └── infrastructure/                  # Integration tests — with Spring + Testcontainers
            └── web/
            └── persistence/
```

**Dependency rule:** The `domain` package does not import anything from `org.springframework.*` or `jakarta.persistence.*`. POJOs only.

---

## Main dependencies (pom.xml)

```xml
<dependencies>
  <!-- ─── Web (primary adapter) ─────────────────────────────────── -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>

  <!-- ─── Request validation ──────────────────────────────────── -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
  </dependency>

  <!-- ─── Persistence (secondary adapter) ─────────────────────── -->
  <!-- Choose one: JPA, JDBC, or native driver access -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>

  <!-- ─── DB engine (choose the one matching the project) ─────── -->
  <!-- PostgreSQL -->
  <dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
  </dependency>

  <!-- ─── DB migrations ────────────────────────────────────────── -->
  <dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
  </dependency>

  <!-- ─── Observability ──────────────────────────────────────────── -->
  <dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
  </dependency>

  <!-- ─── Testing ─────────────────────────────────────────────────── -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
    <!-- Includes JUnit 5, Mockito, AssertJ, Testcontainers -->
  </dependency>
  <dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
```

---

## Example: Input port (Use Case Interface)

```java
// domain/port/in/CreateAppointmentUseCase.java
package com.company.service.domain.port.in;

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

## Example: Output port (Repository Interface)

```java
// domain/port/out/AppointmentRepository.java
package com.company.service.domain.port.out;

import com.company.service.domain.model.Appointment;
import java.util.Optional;

public interface AppointmentRepository {
    void save(Appointment appointment);
    Optional<Appointment> findById(String id);
}
```

## Example: Use Case (Application layer)

```java
// application/usecase/CreateAppointmentService.java
package com.company.service.application.usecase;

import com.company.service.domain.model.Appointment;
import com.company.service.domain.port.in.CreateAppointmentUseCase;
import com.company.service.domain.port.out.AppointmentRepository;
import org.springframework.stereotype.Service;

@Service  // Spring manages the lifecycle; the interface belongs to the domain
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

## Domain unit test (JUnit 5 — no Spring)

```java
// test/domain/model/AppointmentTest.java
class AppointmentTest {

    @Test
    void should_reject_past_scheduled_dates() {
        LocalDateTime yesterday = LocalDateTime.now().minusDays(1);

        assertThatThrownBy(() ->
            Appointment.create("p1", "d1", yesterday)
        ).isInstanceOf(IllegalArgumentException.class)
         .hasMessage("The date cannot be in the past");
    }
}
```

## Use Case test with Fake (no Spring, no DB)

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

## Project commands

```bash
# Compile
mvn compile                  # or: gradle build

# Tests
mvn test                     # or: gradle test
mvn test -pl application     # Only tests for one layer

# Integration tests (requires Docker for Testcontainers)
mvn verify

# Build JAR for production
mvn package -DskipTests      # generates target/service-name.jar

# Run locally
java -jar target/service-name.jar

# With Spring profiles
java -jar -Dspring.profiles.active=local target/service-name.jar
```

---

## Naming conventions (Java)

| Artifact | Convention | Example |
|----------|-----------|---------|
| Interfaces | `PascalCase` (no I prefix) | `AppointmentRepository` |
| Classes | `PascalCase` | `CreateAppointmentService` |
| Packages | `lowercase.dot.separated` | `com.company.service.domain.model` |
| Variables and methods | `camelCase` | `scheduledAt`, `findById` |
| Constants | `SCREAMING_SNAKE_CASE` | `MAX_APPOINTMENTS_PER_DAY` |
| Enums | `PascalCase` with `SCREAMING_SNAKE_CASE` values | `AppointmentStatus.CONFIRMED` |

---

## Correlations with scaffold documents

- Hexagonal concepts → `05-architecture/hexagonal-architecture.md`
- TDD and test doubles → `11-quality/tdd-guide.md`
- Pattern guide (concepts) → `05-architecture/pattern-guide.md`
- Local setup → `10-devops/local-setup.md`
