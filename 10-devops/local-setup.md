# Setup Local del Proyecto

> Este documento garantiza que cualquier desarrollador pueda tener el ambiente local
> funcional en menos de 1 hora. Si tarda más, el documento está incompleto.
> **Regla:** Si tienes que adivinar algo o buscar en otro lado, agrega lo que falta aquí.

---

## Prerequisitos

### Herramientas universales (todos los stacks)

| Herramienta | Versión mínima | Instalar desde | Verificar con |
|-------------|---------------|----------------|--------------|
| Docker | 24.0+ | docker.com | `docker --version` |
| Docker Compose | 2.20+ | Incluido con Docker Desktop | `docker compose version` |
| Git | 2.40+ | git-scm.com | `git --version` |

### Herramienta del lenguaje del proyecto

> Añade aquí la herramienta específica del stack elegido y elimina esta instrucción.
> Ver guía de tu stack en `_stacks/` para las versiones recomendadas.

| Herramienta | Versión mínima | Instalar desde | Verificar con |
|-------------|---------------|----------------|--------------|
| [Lenguaje / Runtime] | [versión LTS] | [URL oficial] | `[comando --version]` |
| [Gestor de paquetes] | [versión] | [incluido / URL] | `[comando --version]` |
| [Herramienta adicional] | [versión] | [URL] | [comando] |

**Referencia de herramientas por stack:**
- Node.js + TypeScript → [`_stacks/node-typescript.md`](../_stacks/node-typescript.md)
- Java + Spring Boot → [`_stacks/java-spring.md`](../_stacks/java-spring.md)
- Python + FastAPI → [`_stacks/python-fastapi.md`](../_stacks/python-fastapi.md)
- Go → [`_stacks/go.md`](../_stacks/go.md)

**Recomendados (no obligatorios):**
- Editor del equipo: [VS Code / IntelliJ IDEA / PyCharm / GoLand]
- Postman o Insomnia para probar endpoints manualmente

---

## Puertos usados en local

Verifica que estos puertos estén libres antes de iniciar:

| Puerto | Servicio | Descripción |
|--------|---------|-------------|
| 8080 | api-gateway | Punto de entrada principal |
| 3001 | auth-service | Servicio de autenticación |
| 300X | [nombre-servicio] | [descripción] |
| 5432 | PostgreSQL | Base de datos relacional |
| 6379 | Redis | Caché y message queue |
| 5672 | RabbitMQ | Message broker (AMQP) |
| 15672 | RabbitMQ Management | Interfaz web de RabbitMQ |
| 9092 | Kafka | Message broker (si aplica) |

**Verificar puertos libres:**
```bash
# Windows
netstat -ano | findstr ":[PUERTO]"

# macOS / Linux
lsof -i :[PUERTO]
```

---

## Instalación paso a paso

### Paso 1: Clonar el repositorio

```bash
git clone https://github.com/[org]/[repo].git
cd [repo]
```

### Paso 2: Variables de ambiente

```bash
# Copia las variables de ambiente de ejemplo
cp .env.example .env

# Edita .env si necesitas cambiar algún valor
# (para local, los defaults del .env.example deberían funcionar)
```

**Variables obligatorias de configurar en .env:**

| Variable | Descripción | Valor para local |
|----------|-------------|-----------------|
| `DATABASE_URL` | Conexión a PostgreSQL | `postgresql://dev:dev@localhost:5432/dev` |
| `REDIS_URL` | Conexión a Redis | `redis://localhost:6379` |
| `JWT_SECRET` | Clave para firmar JWT | `[generar con: openssl rand -base64 32]` |
| `[VARIABLE]` | [descripción] | [valor de ejemplo] |

> **NUNCA** comitas el archivo `.env`. Está en `.gitignore` por razón.

### Paso 3: Levantar la infraestructura

```bash
# Levantar bases de datos y brokers en Docker
docker compose up -d

# Verificar que estén healthy
docker compose ps
```

El comando debe mostrar todos los contenedores en estado `healthy` o `running`.

### Paso 4: Instalar dependencias

```bash
# Desde la raíz del repo (si es un monorepo)
npm install

# O por servicio (si son repositorios separados)
cd services/auth-service && npm install
cd ../[otro-servicio] && npm install
```

### Paso 5: Ejecutar migraciones

```bash
# Aplica las migraciones de base de datos
npm run db:migrate

# Opcionalmente: cargar datos de prueba (seed)
npm run db:seed
```

### Paso 6: Iniciar los servicios

```bash
# Opción A: Todos los servicios a la vez (monorepo)
npm run dev

# Opción B: Por servicio en terminales separadas
npm run dev --workspace=auth-service
npm run dev --workspace=[nombre-servicio]
```

### Paso 7: Verificar que funciona

```bash
# Health check del API Gateway
curl http://localhost:8080/health

# Respuesta esperada:
# {"status": "ok", "timestamp": "2024-01-15T10:30:00Z"}
```

Abre el navegador en `http://localhost:15672` para la interfaz de RabbitMQ (si usas RabbitMQ).

---

## Flujo de trabajo diario

```bash
# Al empezar el día
git pull origin develop
npm install  # si cambiaron dependencias
npm run db:migrate  # si hay migraciones nuevas
npm run dev

# Al terminar
git add [archivos específicos]
git commit -m "feat(scope): descripción"
```

---

## Problemas comunes

### "Port already in use"

```bash
# Encuentra qué proceso usa el puerto
lsof -i :5432    # macOS/Linux
netstat -ano | findstr :5432  # Windows

# Mata el proceso (reemplaza PID)
kill -9 [PID]    # macOS/Linux
taskkill /PID [PID] /F  # Windows
```

### "Docker containers not starting"

```bash
# Ver los logs del contenedor que falla
docker compose logs [nombre-servicio]

# Reiniciar desde cero (borra volúmenes)
docker compose down -v
docker compose up -d
```

### "Database connection refused"

```bash
# Verificar que PostgreSQL esté corriendo
docker compose ps | grep postgres

# Ver logs de PostgreSQL
docker compose logs postgres
```

### "Migration failed"

```bash
# Ver el estado de las migraciones
npm run db:migrate:status

# Revertir la última migración
npm run db:migrate:revert

# Ejecutar desde una versión específica
npm run db:migrate -- --from V005
```

### "JWT_SECRET not set" o errores de autenticación

```bash
# Generar un JWT_SECRET válido
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
# Copia el resultado en .env
```

---

## Comandos de utilidad

```bash
# Tests
npm test                    # Todos los tests
npm run test:unit           # Solo unitarios
npm run test:integration    # Solo integración
npm run test:coverage       # Con reporte de cobertura

# Base de datos
npm run db:migrate          # Aplicar migraciones pendientes
npm run db:seed             # Cargar datos de prueba
npm run db:reset            # Revertir todo y volver a empezar (¡BORRA DATOS!)

# Calidad de código
npm run lint                # Verificar ESLint
npm run lint:fix            # Corregir automáticamente
npm run format              # Aplicar Prettier

# Build
npm run build               # Compilar TypeScript
npm run build:docker        # Construir imagen Docker local
```

---

## URLs útiles en local

| Servicio | URL | Credentials (solo local) |
|---------|-----|--------------------------|
| API Gateway | http://localhost:8080 | |
| RabbitMQ UI | http://localhost:15672 | guest / guest |
| [Herramienta] | http://localhost:[puerto] | [user / pass] |
| Swagger UI | http://localhost:3001/api-docs | |

---

## Correlaciones

- Variables de ambiente de cada ambiente → `10-devops/environments.md`
- Pipeline de CI/CD → `10-devops/README.md`
- Troubleshooting en producción → `09-microservices/services/XX/runbook.md`
