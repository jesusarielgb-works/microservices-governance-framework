# 12 — UX/UI

> **¿Qué es esto?** El diseño de la experiencia de usuario: cómo se ve, cómo se navega
> y cómo se comporta el sistema desde la perspectiva del usuario final.

## Por qué el diseño va antes del código

Cambiar un wireframe toma 5 minutos. Cambiar el código toma horas.
Cambiar el código en producción con usuarios reales puede costar días y mala reputación.

**Diseñar primero → implementar después.**

---

## Qué hay aquí y cómo llenarlo

### `navigation-map.md` ⭐ (Empezar aquí)
El mapa de todas las pantallas/páginas y cómo se conectan.
**Llena:** árbol de navegación, desde qué pantalla se llega a cuál, qué rol puede acceder a qué.

**Formato:**
```markdown
## Mapa de navegación

### Área pública (sin autenticación)
- / (inicio)
  - /login
  - /registro
  - /recuperar-contraseña

### Área privada — Rol: [Rol 1]
- /dashboard
  - /[modulo-1]
    - /[modulo-1]/lista
    - /[modulo-1]/{id}/detalle
  - /perfil

### Área privada — Rol: [Rol 2]
[...]

## Matriz de acceso
| Pantalla | [Rol 1] | [Rol 2] | [Admin] |
|---------|---------|---------|---------|
| /dashboard | ✅ | ✅ | ✅ |
| /admin | ❌ | ❌ | ✅ |
```

### `wireframes.md`
Diseños de baja fidelidad de las pantallas principales.
**Llena:** wireframes en ASCII, Figma, o Balsamiq. Enfócate en estructura, no en colores.

### `design-system.md`
El sistema de diseño del proyecto: tokens, componentes, patrones.
**Llena:** paleta de colores, tipografía, espaciado, componentes base (botones, formularios, tablas).

**Formato:**
```markdown
## Tokens de diseño

### Colores
| Token | Valor | Uso |
|-------|-------|-----|
| --color-primary | #1976D2 | Botones principales, links |
| --color-error | #D32F2F | Mensajes de error |
| --color-success | #388E3C | Confirmaciones |

### Tipografía
| Nivel | Tamaño | Peso | Uso |
|-------|--------|------|-----|
| H1 | 32px | 700 | Títulos de página |
| Body | 16px | 400 | Texto general |

## Componentes
### Botón primario
[descripción, variantes, cuándo usarlo]

### Tabla de datos
[columnas, paginación, búsqueda, acciones inline]
```

---

## Correlaciones con otras secciones

| Esta sección se alimenta de... | Y alimenta a... |
|-------------------------------|-----------------|
| `04-requirements/user-stories.md` → qué flujos existen | Pantallas que implementan cada HU |
| `02-domain/entities-and-rules.md` → qué datos mostrar | Campos en wireframes |
| `09-microservices/` → qué APIs consume el frontend | Qué datos llegan a cada pantalla |

---

## Preguntas que esta sección debe responder

- ¿Cuántas pantallas tiene el sistema?
- ¿Cómo navega cada tipo de usuario?
- ¿Qué componentes visuales se repiten?
- ¿Cuál es el lenguaje visual del sistema?
