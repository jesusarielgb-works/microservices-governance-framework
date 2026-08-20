# Design System

> El design system es el lenguaje visual compartido entre diseño y desarrollo.
> Previene inconsistencias, acelera el diseño y reduce el re-trabajo.
> **Regla:** Antes de crear un componente nuevo, busca aquí si ya existe.

---

## Tokens de diseño

Los tokens son las variables del design system. Un cambio de token cambia el sistema entero.

### Colores

```css
/* Paleta base */
--color-primary-50:  #[hex];   /* El más claro */
--color-primary-100: #[hex];
--color-primary-500: #[hex];   /* El default */
--color-primary-900: #[hex];   /* El más oscuro */

--color-secondary-500: #[hex];
--color-neutral-50:  #[hex];
--color-neutral-900: #[hex];

/* Colores semánticos */
--color-success:  #[hex];      /* Verde — éxito, confirmado */
--color-warning:  #[hex];      /* Amarillo — precaución, pendiente */
--color-error:    #[hex];      /* Rojo — error, cancelado */
--color-info:     #[hex];      /* Azul — información neutral */

/* Texto */
--color-text-primary:   #[hex];
--color-text-secondary: #[hex];
--color-text-disabled:  #[hex];

/* Fondos */
--color-bg-page:    #[hex];
--color-bg-card:    #[hex];
--color-bg-overlay: rgba([r],[g],[b], 0.5);
```

### Tipografía

```css
/* Familias */
--font-family-sans:  '[Nombre de fuente], sans-serif';
--font-family-mono:  '[Nombre de fuente mono], monospace';

/* Tamaños (escala modular 1.25) */
--font-size-xs:   0.75rem;   /* 12px */
--font-size-sm:   0.875rem;  /* 14px */
--font-size-base: 1rem;      /* 16px */
--font-size-lg:   1.25rem;   /* 20px */
--font-size-xl:   1.563rem;  /* 25px */
--font-size-2xl:  1.953rem;  /* 31px */
--font-size-3xl:  2.441rem;  /* 39px */

/* Pesos */
--font-weight-regular: 400;
--font-weight-medium:  500;
--font-weight-bold:    700;

/* Interlineado */
--line-height-tight:  1.2;
--line-height-normal: 1.5;
--line-height-loose:  1.8;
```

### Espaciado

```css
/* Sistema de 4px */
--space-1:  0.25rem;   /* 4px */
--space-2:  0.5rem;    /* 8px */
--space-3:  0.75rem;   /* 12px */
--space-4:  1rem;      /* 16px */
--space-6:  1.5rem;    /* 24px */
--space-8:  2rem;      /* 32px */
--space-12: 3rem;      /* 48px */
--space-16: 4rem;      /* 64px */
```

### Bordes y sombras

```css
/* Border radius */
--radius-sm: 4px;
--radius-md: 8px;
--radius-lg: 16px;
--radius-full: 9999px;  /* Píldora */

/* Sombras */
--shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
--shadow-md: 0 4px 6px rgba(0,0,0,0.1);
--shadow-lg: 0 10px 15px rgba(0,0,0,0.15);
```

---

## Componentes

### Botones

| Variante | Uso | Estado desactivado |
|---------|-----|-------------------|
| Primary | Acción principal de la página | `opacity: 0.5; cursor: not-allowed` |
| Secondary | Acciones secundarias | idem |
| Danger | Acciones destructivas (eliminar) | idem |
| Ghost | Acciones terciarias, links | idem |

**Reglas de uso:**
- Una sola acción Primary por vista
- Danger solo con confirmación modal ("¿Estás seguro?")
- Los botones tienen estado loading para operaciones async

### Formularios

| Componente | Cuando usar |
|-----------|-------------|
| Input text | Texto libre de una línea |
| Textarea | Texto libre de múltiples líneas |
| Select | Lista fija de opciones (< 15 items) |
| Combobox | Lista con búsqueda (> 15 items o carga dinámica) |
| Checkbox | Opción binaria independiente |
| Radio | Selección de una opción entre pocas (2-5) |
| Toggle | Activar/desactivar una funcionalidad |
| DatePicker | Selección de fecha |

**Mensajes de error en formularios:**
- El mensaje aparece debajo del campo, en rojo
- El borde del campo se pone rojo
- El mensaje dice cómo corregir el error, no solo que hay error

```
✓ "El email debe tener formato usuario@dominio.com"
✗ "Email inválido"
```

### Retroalimentación

| Componente | Cuándo | Duración |
|-----------|--------|---------|
| Toast/Snackbar | Confirmaciones de acciones | 4 segundos |
| Alert inline | Errores de formulario | Hasta que se corrija |
| Modal | Confirmaciones destructivas, acciones irreversibles | Hasta que el usuario decida |
| Loading spinner | Operaciones > 200ms | Hasta que termine |
| Skeleton | Carga de contenido de lista / tarjetas | Hasta que carguen |

### Tabla de datos

| Aspecto | Comportamiento |
|---------|---------------|
| Paginación | Máximo 20 filas por página (configurable por usuario) |
| Ordenamiento | Click en columna, toggle asc/desc |
| Filtros | Panel lateral o fila de filtros arriba de la tabla |
| Selección | Checkbox en la primera columna |
| Acciones | Columna final con menú de acciones (editar, eliminar, etc.) |
| Estado vacío | Ilustración + mensaje + CTA de acción principal |

---

## Patrones de UX

### Principios

1. **Confirmación antes de destruir:** Cualquier acción que borre o modifique datos permanentemente requiere un modal de confirmación.

2. **Feedback inmediato:** Toda acción debe tener respuesta visual en < 100ms (aunque sea solo el estado loading).

3. **Prevenir antes que corregir:** Validar en tiempo real en el formulario, no solo al submit.

4. **Estado vacío como feature:** La pantalla sin datos es la primera impresión del usuario nuevo — guíalo a la primera acción.

### Manejo de errores

| Escenario | Qué mostrar |
|----------|-------------|
| Error de red | Toast "Sin conexión. Reintentando..." con retry automático |
| Error 401 | Redirigir al login con mensaje "Tu sesión expiró" |
| Error 403 | Pantalla "No tienes permisos para ver esto" con link a soporte |
| Error 404 | Pantalla de 404 con navegación de regreso |
| Error 500 | Toast de error + botón "Reintentar" |
| Timeout | Toast "Esto está tardando más de lo normal" con opción cancelar |

---

## Guía de accesibilidad (mínimos)

| Aspecto | Mínimo requerido |
|---------|-----------------|
| Contraste de texto | WCAG AA (4.5:1 para texto normal, 3:1 para texto grande) |
| Navegación con teclado | Todos los elementos interactivos accesibles con Tab |
| Labels de formularios | Todos los campos con label asociado (`for` / `aria-label`) |
| Imágenes | Alt text descriptivo en todas las imágenes no decorativas |
| Focus visible | Indicador de focus visible en todos los elementos interactivos |

---

## Correlaciones

- Mapa de navegación → `12-ux-ui/navigation-map.md`
- Wireframes → `12-ux-ui/wireframes.md`
- Requisitos no funcionales de UX → `04-requirements/non-functional.md`
