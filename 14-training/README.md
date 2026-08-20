# 14 — Capacitación

> **¿Qué es esto?** Manuales para los diferentes perfiles que interactúan con el sistema:
> usuarios finales, administradores y nuevos desarrolladores del equipo.

---

## Qué hay aquí y cómo llenarlo

### `user-manual.md`
Manual para el usuario final del sistema.
**Llena:** al final del proyecto, cuando las pantallas están estables.
**Audiencia:** persona NO técnica que usa el sistema en su trabajo diario.
**Incluye:** cómo realizar las tareas principales, screenshots, qué hacer si algo falla.

### `admin-manual.md`
Manual para el administrador del sistema.
**Llena:** configuraciones del sistema, gestión de usuarios y permisos, tareas periódicas.
**Audiencia:** persona técnica que configura y mantiene el sistema (no necesariamente desarrollador).

### `technical-onboarding.md` ⭐
Guía para que un nuevo desarrollador contribuya al proyecto.
**Llena:** desde el primer día del proyecto.
**Audiencia:** desarrollador nuevo con experiencia general pero sin conocimiento de este proyecto.

**Contenido mínimo:**
```markdown
## Día 1 — Setup
1. Lee [00-governance/git-conventions.md] — 30 min
2. Levanta el sistema localmente ([10-devops/local-setup.md]) — 2 horas
3. Lee el overview arquitectónico ([05-architecture/overview.md]) — 1 hora
4. Corre las pruebas y asegúrate de que todo pasa — 30 min

## Semana 1 — Entender el dominio
- Lee [01-context/overview.md] y [02-domain/domain-map.md]
- Revisa el service catalog [09-microservices/service-catalog.md]
- Lee el ADR del servicio en el que vas a trabajar

## Primera contribución
1. Busca una tarea etiquetada como "good first issue" en el backlog
2. Sigue el flujo de Git definido en [00-governance/git-conventions.md]
3. Pide revisión de código a [persona del equipo]

## Preguntas frecuentes
- **¿Cómo agrego un endpoint nuevo?** → Ver [07-api/guidelines.md], luego el servicio correspondiente en [09-microservices/services/]
- **¿Cómo agrego una migración de BD?** → Ver [06-data/migration-strategy.md]
- **¿Cómo corro solo un servicio?** → Ver [10-devops/local-setup.md#servicios-individuales]
```

---

## Correlaciones

- `10-devops/local-setup.md` → primer paso del onboarding técnico
- `00-governance/` → reglas que el nuevo debe conocer
- `09-microservices/` → destino de la mayor parte del trabajo
