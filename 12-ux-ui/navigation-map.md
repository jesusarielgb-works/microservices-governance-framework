# Mapa de Navegación

> Define la estructura de pantallas del sistema, cómo se conectan entre sí y qué rutas
> existen. Es la referencia cuando frontend y backend discuten qué endpoints existen
> o cómo llegar a una funcionalidad.

---

## Estructura de rutas del frontend

> **Instrucción:** Llena este árbol con las rutas reales de tu aplicación.
> Usa el formato `[método] /ruta` para endpoints de API cuando corresponda.

```
/                           → Página de inicio / landing
├── /auth
│   ├── /login              → Formulario de autenticación
│   ├── /register           → Registro de nuevo usuario
│   └── /forgot-password    → Recuperación de contraseña
│
├── /dashboard              → Panel principal (autenticado)
│   ├── /overview           → Resumen y métricas clave
│   └── /notifications      → Centro de notificaciones
│
├── /[recurso-a]            → Lista de [Recurso A]
│   ├── /new                → Formulario de creación
│   └── /:id
│       ├── /               → Detalle del recurso
│       └── /edit           → Formulario de edición
│
├── /[recurso-b]            → Lista de [Recurso B]
│   └── /:id                → Detalle
│
├── /admin                  → Panel de administración (rol: ADMIN)
│   ├── /users              → Gestión de usuarios
│   └── /settings           → Configuración del sistema
│
└── /profile                → Perfil del usuario autenticado
```

---

## Mapa de pantallas

| Pantalla | Ruta | Componente | Rol mínimo | Servicio backend |
|----------|------|------------|------------|------------------|
| Inicio | `/` | `HomePage` | Público | — |
| Login | `/auth/login` | `LoginPage` | Público | auth-service |
| Registro | `/auth/register` | `RegisterPage` | Público | auth-service |
| Dashboard | `/dashboard` | `DashboardPage` | USER | [servicio] |
| Lista [Recurso A] | `/[recurso-a]` | `[RecursoA]ListPage` | USER | [servicio] |
| Detalle [Recurso A] | `/[recurso-a]/:id` | `[RecursoA]DetailPage` | USER | [servicio] |
| Crear [Recurso A] | `/[recurso-a]/new` | `[RecursoA]FormPage` | USER | [servicio] |
| Panel Admin | `/admin` | `AdminDashboard` | ADMIN | auth-service |

---

## Flujos de usuario principales

### Flujo 1 — [Nombre del flujo principal]

```
[Pantalla inicio]
    │
    ▼ [Acción del usuario]
[Pantalla 2]
    │
    ├── [Caso exitoso] ──► [Pantalla resultado OK]
    │
    └── [Caso error] ────► [Pantalla error / feedback]
```

**HUs relacionadas:** HU-[servicio]-001, HU-[servicio]-002

### Flujo 2 — Autenticación

```
Landing (/)
    │
    ▼ Click "Iniciar sesión"
Login (/auth/login)
    │
    ├── Credenciales válidas ──► Dashboard (/dashboard)
    │
    └── Credenciales inválidas ► Login con mensaje de error (máx. 5 intentos)
```

**HUs relacionadas:** HU-AUTH-001, HU-AUTH-002

---

## Reglas de navegación

| Regla | Descripción |
|-------|-------------|
| Autenticación | Rutas bajo `/dashboard`, `/[recurso]`, `/admin` redirigen a `/auth/login` si no hay sesión |
| Autorización | Rutas bajo `/admin` redirigen a `/dashboard` si el usuario no tiene rol ADMIN |
| 404 | Rutas no definidas muestran la pantalla de 404 con link al dashboard |
| Confirmación | Acciones destructivas (borrar, cancelar) muestran diálogo de confirmación antes de ejecutar |

---

## Correlaciones

- Design system (componentes visuales) → `12-ux-ui/design-system.md`
- Wireframes → `12-ux-ui/wireframes/` (si aplica)
- Contratos API del frontend → `07-api/contracts/openapi/`
- Roles y permisos → `00-governance/security-policy.md`
