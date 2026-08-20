# Navigation Map

> Defines the screen structure of the system, how screens connect to each other, and what routes
> exist. It is the reference when frontend and backend discuss what endpoints exist
> or how to reach a feature.

---

## Frontend route structure

> **Instruction:** Fill this tree with your application's real routes.
> Use the `[method] /route` format for API endpoints where applicable.

```
/                           → Home / landing page
├── /auth
│   ├── /login              → Authentication form
│   ├── /register           → New user registration
│   └── /forgot-password    → Password recovery
│
├── /dashboard              → Main panel (authenticated)
│   ├── /overview           → Summary and key metrics
│   └── /notifications      → Notification center
│
├── /[resource-a]           → [Resource A] list
│   ├── /new                → Creation form
│   └── /:id
│       ├── /               → Resource detail
│       └── /edit           → Edit form
│
├── /[resource-b]           → [Resource B] list
│   └── /:id                → Detail
│
├── /admin                  → Administration panel (role: ADMIN)
│   ├── /users              → User management
│   └── /settings           → System configuration
│
└── /profile                → Authenticated user's profile
```

---

## Screen map

| Screen | Route | Component | Minimum role | Backend service |
|--------|-------|-----------|--------------|----------------|
| Home | `/` | `HomePage` | Public | — |
| Login | `/auth/login` | `LoginPage` | Public | auth-service |
| Register | `/auth/register` | `RegisterPage` | Public | auth-service |
| Dashboard | `/dashboard` | `DashboardPage` | USER | [service] |
| [Resource A] list | `/[resource-a]` | `[ResourceA]ListPage` | USER | [service] |
| [Resource A] detail | `/[resource-a]/:id` | `[ResourceA]DetailPage` | USER | [service] |
| Create [Resource A] | `/[resource-a]/new` | `[ResourceA]FormPage` | USER | [service] |
| Admin panel | `/admin` | `AdminDashboard` | ADMIN | auth-service |

---

## Main user flows

### Flow 1 — [Name of main flow]

```
[Start screen]
    │
    ▼ [User action]
[Screen 2]
    │
    ├── [Successful case] ──► [OK result screen]
    │
    └── [Error case] ────► [Error screen / feedback]
```

**Related HUs:** HU-[service]-001, HU-[service]-002

### Flow 2 — Authentication

```
Landing (/)
    │
    ▼ Click "Sign in"
Login (/auth/login)
    │
    ├── Valid credentials ──► Dashboard (/dashboard)
    │
    └── Invalid credentials ► Login with error message (max. 5 attempts)
```

**Related HUs:** HU-AUTH-001, HU-AUTH-002

---

## Navigation rules

| Rule | Description |
|------|-------------|
| Authentication | Routes under `/dashboard`, `/[resource]`, `/admin` redirect to `/auth/login` if no session |
| Authorization | Routes under `/admin` redirect to `/dashboard` if the user does not have ADMIN role |
| 404 | Undefined routes show the 404 screen with a link to dashboard |
| Confirmation | Destructive actions (delete, cancel) show a confirmation dialog before executing |

---

## Correlations

- Design system (visual components) → `12-ux-ui/design-system.md`
- Wireframes → `12-ux-ui/wireframes/` (if applicable)
- Frontend API contracts → `07-api/contracts/openapi/`
- Roles and permissions → `00-governance/security-policy.md`
