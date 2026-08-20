# 10 — DevOps

> **¿Qué es esto?** Cómo el código pasa de la computadora del desarrollador a producción,
> y cómo se mantiene en cada ambiente. CI/CD, ambientes y configuración local.

## Por qué existe esta sección

Sin DevOps documentado:
- Cada developer tiene su propio proceso de deploy (y falla diferente)
- "Funciona en mi máquina" es la respuesta más costosa en software
- Los releases son eventos de alto riesgo en lugar de rutina

---

## Qué hay aquí y cómo llenarlo

### `local-setup.md` ⭐ (Lo más urgente)
Cómo levantar TODO el sistema en una computadora nueva desde cero.
**Llena:** paso a paso, sin omitir nada, asumiendo que quien lee nunca ha tocado el proyecto.
Incluye: prerequisitos de software, variables de entorno, comandos exactos, cómo verificar que funciona.

**Formato:**
```markdown
## Prerequisitos
- [ ] Docker Desktop >= 24.0
- [ ] [Lenguaje/Runtime] >= [versión]
- [ ] [Herramienta] >= [versión]

## Pasos
1. Clonar el repositorio principal: `git clone [url]`
2. Copiar variables de entorno: `cp .env.example .env`
3. Levantar infraestructura: `docker-compose up -d`
4. Esperar a que los servicios estén listos: `./scripts/wait-for-services.sh`
5. Verificar: `curl http://localhost:8080/health`

## Puertos locales
| Servicio | Puerto |
|---------|--------|
| API Gateway | 8080 |
| [Servicio 1] | 8001 |
| RabbitMQ UI | 15672 |
| Adminer (BD) | 8090 |

## Problemas comunes
- **Puerto en uso:** `lsof -i :8080` para ver qué lo usa
- **BD no conecta:** verificar que Docker esté corriendo
```

### `environments.md` ⭐
Descripción de todos los ambientes del proyecto.
**Llena:** local, desarrollo, staging, producción. Para cada uno: propósito, URL, quién tiene acceso,
cómo se despliega, qué datos tiene.

**Formato:**
```markdown
| Ambiente | URL | Propósito | Acceso | BD | Deploy |
|---------|-----|-----------|--------|-----|--------|
| local | localhost | Desarrollo individual | Todos | Datos de prueba | Manual |
| dev | dev.api.domain.com | Integración continua | Equipo | Datos de prueba | Automático (push a dev) |
| staging | staging.api.domain.com | QA / validación | Equipo + PO | Datos anonimizados | Manual (aprobación) |
| prod | api.domain.com | Producción | Ops team | Datos reales | Manual (aprobación doble) |
```

### `ci-cd.md` ⭐
Descripción del pipeline de integración y despliegue continuo.
**Llena:** qué herramienta (GitHub Actions, GitLab CI, Jenkins), qué pasos ejecuta,
cuándo se dispara cada pipeline, qué verifica antes de permitir el merge.

**Formato:**
```markdown
## Pipeline de PR (se ejecuta en cada Pull Request)
1. Lint y formato de código
2. Pruebas unitarias
3. Pruebas de integración
4. Análisis de cobertura (mínimo [X]%)
5. Análisis de seguridad (SAST)

## Pipeline de merge a `dev`
1. Todo lo anterior
2. Build de imagen Docker
3. Deploy a ambiente dev
4. Smoke tests en dev

## Pipeline de release a producción
1. Aprobación manual de [rol]
2. Deploy a staging
3. Suite de pruebas de aceptación
4. Aprobación manual de [rol]
5. Deploy a producción con blue-green / canary
```

---

## Correlaciones con otras secciones

| Esta sección se alimenta de... | Y alimenta a... |
|-------------------------------|-----------------|
| `05-architecture/deployment.md` → cómo se despliega | Implementación del pipeline |
| `11-quality/testing-strategy.md` → qué pruebas correr | Pasos del pipeline |
| `09-microservices/` → cada servicio a desplegar | Qué imágenes construye el pipeline |
| `13-operations/` → monitoreo post-deploy | Lo que el pipeline verifica |

---

## Preguntas que esta sección debe responder

- ¿Cómo levanto el sistema localmente en 30 minutos?
- ¿Qué pasa automáticamente cuando hago push?
- ¿Cómo llega el código a producción?
- ¿Cuántos ambientes hay y para qué sirve cada uno?
