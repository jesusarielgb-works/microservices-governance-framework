# 12 — UX/UI

> **What is this?** The user experience design: how the system looks, how it is navigated,
> and how it behaves from the end user's perspective.

## Why design comes before code

Changing a wireframe takes 5 minutes. Changing the code takes hours.
Changing the code in production with real users can cost days and reputation.

**Design first → implement later.**

---

## What is here and how to fill it in

### `navigation-map.md` ⭐ (Start here)
The map of all screens/pages and how they connect.
**Fill in:** navigation tree, from which screen you reach which, what role can access what.

**Format:**
```markdown
## Navigation map

### Public area (no authentication)
- / (home)
  - /login
  - /register
  - /recover-password

### Private area — Role: [Role 1]
- /dashboard
  - /[module-1]
    - /[module-1]/list
    - /[module-1]/{id}/detail
  - /profile

### Private area — Role: [Role 2]
[...]

## Access matrix
| Screen | [Role 1] | [Role 2] | [Admin] |
|--------|---------|---------|---------|
| /dashboard | ✅ | ✅ | ✅ |
| /admin | ❌ | ❌ | ✅ |
```

### `wireframes.md`
Low-fidelity designs of the main screens.
**Fill in:** wireframes in ASCII, Figma, or Balsamiq. Focus on structure, not colors.

### `design-system.md`
The project's design system: tokens, components, patterns.
**Fill in:** color palette, typography, spacing, base components (buttons, forms, tables).

**Format:**
```markdown
## Design tokens

### Colors
| Token | Value | Use |
|-------|-------|-----|
| --color-primary | #1976D2 | Primary buttons, links |
| --color-error | #D32F2F | Error messages |
| --color-success | #388E3C | Confirmations |

### Typography
| Level | Size | Weight | Use |
|-------|------|--------|-----|
| H1 | 32px | 700 | Page titles |
| Body | 16px | 400 | General text |

## Components
### Primary button
[description, variants, when to use it]

### Data table
[columns, pagination, search, inline actions]
```

---

## Correlations with other sections

| This section is fed by... | And feeds into... |
|---------------------------|-------------------|
| `04-requirements/user-stories.md` → what flows exist | Screens implementing each HU |
| `02-domain/entities-and-rules.md` → what data to display | Fields in wireframes |
| `09-microservices/` → what APIs the frontend consumes | What data arrives at each screen |

---

## Questions this section must answer

- How many screens does the system have?
- How does each type of user navigate?
- What visual components are repeated?
- What is the system's visual language?
