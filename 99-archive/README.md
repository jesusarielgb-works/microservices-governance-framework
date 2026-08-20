# 99 — Archivo

> **¿Qué es esto?** Documentos que ya no están activos pero que se conservan por contexto histórico.
> Nunca se borra — se archiva.

## Por qué conservar documentos obsoletos

- Los ADRs rechazados explican por qué NO se tomó cierta decisión (evita repetir discusiones)
- Los documentos deprecados muestran la evolución del sistema
- El historial de decisiones ayuda a entender el presente

## Estructura

```
99-archive/
├── deprecated/         ← Documentos que reemplazaron a otros (mover aquí con nota)
└── old-decisions/      ← Propuestas y decisiones que no prosperaron
```

## Cómo archivar un documento

1. Agrega al inicio del archivo:
   ```markdown
   > ⚠️ ARCHIVADO — [Fecha]
   > Reemplazado por: [link al nuevo documento]
   > Razón: [breve explicación]
   ```
2. Mueve el archivo a la carpeta correspondiente
3. Actualiza cualquier link que apuntara al archivo original
