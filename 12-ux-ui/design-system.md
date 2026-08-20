# Design System

> The design system is the shared visual language between design and development.
> It prevents inconsistencies, accelerates design, and reduces rework.
> **Rule:** Before creating a new component, check here if it already exists.

---

## Design tokens

Tokens are the design system's variables. Changing a token changes the entire system.

### Colors

```css
/* Base palette */
--color-primary-50:  #[hex];   /* Lightest */
--color-primary-100: #[hex];
--color-primary-500: #[hex];   /* Default */
--color-primary-900: #[hex];   /* Darkest */

--color-secondary-500: #[hex];
--color-neutral-50:  #[hex];
--color-neutral-900: #[hex];

/* Semantic colors */
--color-success:  #[hex];      /* Green — success, confirmed */
--color-warning:  #[hex];      /* Yellow — caution, pending */
--color-error:    #[hex];      /* Red — error, cancelled */
--color-info:     #[hex];      /* Blue — neutral information */

/* Text */
--color-text-primary:   #[hex];
--color-text-secondary: #[hex];
--color-text-disabled:  #[hex];

/* Backgrounds */
--color-bg-page:    #[hex];
--color-bg-card:    #[hex];
--color-bg-overlay: rgba([r],[g],[b], 0.5);
```

### Typography

```css
/* Families */
--font-family-sans:  '[Font name], sans-serif';
--font-family-mono:  '[Mono font name], monospace';

/* Sizes (modular scale 1.25) */
--font-size-xs:   0.75rem;   /* 12px */
--font-size-sm:   0.875rem;  /* 14px */
--font-size-base: 1rem;      /* 16px */
--font-size-lg:   1.25rem;   /* 20px */
--font-size-xl:   1.563rem;  /* 25px */
--font-size-2xl:  1.953rem;  /* 31px */
--font-size-3xl:  2.441rem;  /* 39px */

/* Weights */
--font-weight-regular: 400;
--font-weight-medium:  500;
--font-weight-bold:    700;

/* Line height */
--line-height-tight:  1.2;
--line-height-normal: 1.5;
--line-height-loose:  1.8;
```

### Spacing

```css
/* 4px system */
--space-1:  0.25rem;   /* 4px */
--space-2:  0.5rem;    /* 8px */
--space-3:  0.75rem;   /* 12px */
--space-4:  1rem;      /* 16px */
--space-6:  1.5rem;    /* 24px */
--space-8:  2rem;      /* 32px */
--space-12: 3rem;      /* 48px */
--space-16: 4rem;      /* 64px */
```

### Borders and shadows

```css
/* Border radius */
--radius-sm: 4px;
--radius-md: 8px;
--radius-lg: 16px;
--radius-full: 9999px;  /* Pill */

/* Shadows */
--shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
--shadow-md: 0 4px 6px rgba(0,0,0,0.1);
--shadow-lg: 0 10px 15px rgba(0,0,0,0.15);
```

---

## Components

### Buttons

| Variant | Use | Disabled state |
|---------|-----|----------------|
| Primary | Main action on the page | `opacity: 0.5; cursor: not-allowed` |
| Secondary | Secondary actions | same |
| Danger | Destructive actions (delete) | same |
| Ghost | Tertiary actions, links | same |

**Usage rules:**
- Only one Primary action per view
- Danger only with modal confirmation ("Are you sure?")
- Buttons have a loading state for async operations

### Forms

| Component | When to use |
|-----------|-------------|
| Input text | Single-line free text |
| Textarea | Multi-line free text |
| Select | Fixed list of options (< 15 items) |
| Combobox | List with search (> 15 items or dynamic loading) |
| Checkbox | Independent binary option |
| Radio | Select one option from a few (2-5) |
| Toggle | Enable/disable a feature |
| DatePicker | Date selection |

**Error messages in forms:**
- The message appears below the field, in red
- The field border turns red
- The message says how to fix the error, not just that there is an error

```
✓ "The email must have the format user@domain.com"
✗ "Invalid email"
```

### Feedback

| Component | When | Duration |
|-----------|------|---------|
| Toast/Snackbar | Action confirmations | 4 seconds |
| Inline alert | Form errors | Until corrected |
| Modal | Destructive confirmations, irreversible actions | Until the user decides |
| Loading spinner | Operations > 200ms | Until finished |
| Skeleton | Loading list content / cards | Until loaded |

### Data table

| Aspect | Behavior |
|--------|---------|
| Pagination | Maximum 20 rows per page (user-configurable) |
| Sorting | Click on column, toggle asc/desc |
| Filters | Side panel or filter row above the table |
| Selection | Checkbox in the first column |
| Actions | Final column with actions menu (edit, delete, etc.) |
| Empty state | Illustration + message + primary action CTA |

---

## UX patterns

### Principles

1. **Confirm before destroying:** Any action that permanently deletes or modifies data requires a confirmation modal.

2. **Immediate feedback:** Every action must have a visual response in < 100ms (even if it is just the loading state).

3. **Prevent rather than correct:** Validate in real time in the form, not only on submit.

4. **Empty state as a feature:** The screen without data is the new user's first impression — guide them to the first action.

### Error handling

| Scenario | What to show |
|----------|-------------|
| Network error | Toast "No connection. Retrying..." with automatic retry |
| 401 error | Redirect to login with message "Your session expired" |
| 403 error | Screen "You do not have permission to view this" with link to support |
| 404 error | 404 screen with back navigation |
| 500 error | Error toast + "Retry" button |
| Timeout | Toast "This is taking longer than normal" with cancel option |

---

## Accessibility guide (minimums)

| Aspect | Minimum required |
|--------|-----------------|
| Text contrast | WCAG AA (4.5:1 for normal text, 3:1 for large text) |
| Keyboard navigation | All interactive elements accessible with Tab |
| Form labels | All fields with associated label (`for` / `aria-label`) |
| Images | Descriptive alt text on all non-decorative images |
| Visible focus | Visible focus indicator on all interactive elements |

---

## Correlations

- Navigation map → `12-ux-ui/navigation-map.md`
- Wireframes → `12-ux-ui/wireframes.md`
- UX non-functional requirements → `04-requirements/non-functional.md`
