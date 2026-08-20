# Matriz de Trazabilidad

> La trazabilidad conecta cada línea de código con su justificación de negocio.
> Permite responder: "¿Por qué existe esta función?" y "¿Cuál HU cubre esta parte del sistema?"
> También identifica: requisitos sin implementar y código sin requisito (posible deuda técnica).

---

## Cómo usar esta matriz

```
Requisito → HU → Caso de Prueba → Implementación → Servicio

Si un requisito no tiene HU: no está planificado
Si una HU no tiene caso de prueba: no tiene criterio de completitud
Si un caso de prueba no tiene implementación: hay deuda técnica de pruebas
Si hay código sin HU: posible gold-plating o bug introducido sin historia
```

---

## Matriz RF → HU → Test → Servicio

| RF ID | Descripción RF | HU(s) | Tests que lo verifican | Servicio | Estado |
|-------|---------------|-------|----------------------|---------|--------|
| RF-001 | [El sistema permite registro de usuarios] | HU-001 | `auth.register.spec.ts` | auth-service | ✅ Done |
| RF-002 | [El sistema permite autenticación] | HU-002 | `auth.login.spec.ts` | auth-service | ✅ Done |
| RF-003 | [El sistema permite crear pedidos] | HU-003, HU-004 | `pedido.create.spec.ts` | pedido-service | 🟡 In progress |
| RF-004 | [El sistema notifica por email] | HU-010 | `notifications.spec.ts` | notification-service | 🔴 Pendiente |
| RF-00X | [descripción] | [HU-00X] | [archivo de test] | [servicio] | [estado] |

---

## Matriz RNF → Validación

| RNF ID | Descripción | Cómo se valida | Herramienta | Estado |
|--------|-------------|---------------|-------------|--------|
| RNF-001 | P95 < 300ms | Test de carga en staging | k6 | ✅ Validado |
| RNF-002 | Disponibilidad 99.9% | SLO monitoring | Grafana | 🟡 Monitoreando |
| RNF-004 | Autenticación JWT | Contract test de seguridad | Postman + OWASP ZAP | 🔴 Pendiente |

---

## Trazabilidad inversa: HU → RF

| HU | Título | RF(s) que implementa | Sprint |
|----|--------|---------------------|--------|
| HU-001 | [Registro de usuario] | RF-001 | Sprint 1 |
| HU-002 | [Login] | RF-002 | Sprint 1 |
| HU-003 | [Crear pedido básico] | RF-003 | Sprint 2 |

---

## Leyenda de estado

| Estado | Significado |
|--------|-------------|
| ✅ Done | Implementado, testeado y en producción |
| 🟡 In progress | En desarrollo en el sprint actual |
| 🔴 Pendiente | En el backlog, no iniciado |
| ⏸ Bloqueado | Tiene bloqueante externo |
| ❌ Cancelado | Se eliminó del scope |

---

## Gaps identificados (requisitos sin cobertura)

> Esta sección se actualiza automáticamente o manualmente al revisar la matriz.
> Un gap es: un RF sin HU, o una HU sin test, o un test sin implementación.

| Tipo de gap | Descripción | Acción requerida | Responsable | Fecha |
|-------------|-------------|-----------------|-------------|-------|
| RF sin HU | RF-00X no tiene HU asociada | Crear HU en el backlog | Product Owner | [fecha] |
| HU sin test | HU-00X no tiene test de aceptación | Agregar test antes del siguiente sprint | QA / Dev | [fecha] |

---

## Cómo mantener esta matriz

1. Cuando se crea una HU: agregar la fila en la sección RF → HU → Test → Servicio
2. Cuando se escribe un test: anotar el archivo en la columna "Tests que lo verifican"
3. Cuando se completa una HU: cambiar el estado a ✅
4. En cada Sprint Planning: revisar los gaps y asignar acciones

---

## Correlaciones

- Historias de Usuario → `04-requirements/user-stories.md`
- Requisitos No Funcionales → `04-requirements/non-functional.md`
- Strategy de testing → `11-quality/testing-strategy.md`
- DoD que determina cuándo una HU es Done → `00-governance/definition-of-done.md`
