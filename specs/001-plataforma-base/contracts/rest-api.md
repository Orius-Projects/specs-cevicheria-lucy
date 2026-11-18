---
feature: 001-plataforma-base
titulo: Contrato REST base v2
version: 2.0.0
estado: borrador
creado: 2026-11-17
actualizado: 2026-11-17
fuentes:
  - docs/sources/DOCUMENTO_PROCESOS_CEVICHERIA_LUCY.pdf
---

# Contrato REST base — Cevichería Lucy v2

> Contrato canónico para `001`. `002`, `003`, `004`, `005` y `006` lo consumen sin redefinir endpoints ni formatos. No incluye pagos, IGV ni facturación.

## 1. Convenciones

- Base URL: `/api`.
- JSON UTF-8.
- Autenticación: `Authorization: Bearer <token>`.
- Fechas: ISO 8601 UTC.
- Dinero: enteros en céntimos (`2500` = `S/ 25.00`). **Sin IGV.**
- IDs: ObjectId como string.
- Paginación: `?page=1&limit=20`.
- Errores uniformes (§3).
- El servidor es la autoridad del estado y de las raciones.
- No se devuelven campos de pago, cliente tributario, IGV ni facturación.

## 2. Roles y permisos

| Recurso | `ENCARGADO` | `MOZO` | `COCINERO` |
|---|---|---|---|
| Login / me | ✅ | ✅ | ✅ |
| Mesas y estado | ✅ | ✅ | ✅ |
| Abrir/cerrar turno | ✅ | ❌ | ❌ |
| Ver resumen de turno | ✅ | ❌ | ❌ |
| Menú publicado | ✅ | ✅ | ✅ lectura |
| Crear/editar menú, platos y precios | ✅ | ❌ | ❌ |
| Definir raciones/límites | ✅ | ❌ | ❌ |
| Reponer raciones | ✅ | ✅ | ❌ |
| Ajustar conteo con motivo | ✅ | ❌ | ❌ |
| Marcar agotado por insumo | ✅ | ❌ | ❌ |
| Crear/enviar pedido | ❌ | ✅ | ❌ |
| Ver pedidos activos/listos | ✅ todos | ✅ todos | ✅ cola |
| Marcar listo | ❌ | ❌ | ✅ |
| Marcar entregado | ❌ | ✅ cualquiera | ❌ |
| Cancelar pedido | ❌ | ✅ con motivo | ❌ |
| Cerrar mesa | ✅ | ✅ | ❌ |
| Cierre manual de pedido | ✅ P2 | ❌ | ❌ |
| Registro por falla | ✅ P2 | ❌ | ❌ |

## 3. Formato de errores

```json
{
  "error": {
    "code": "RATION_UNAVAILABLE",
    "message": "No quedan raciones suficientes de Chicharrón de pollo.",
    "details": [{ "field": "items[0].quantity", "message": "Solicitadas 2, disponibles 1." }],
    "requestId": "req_01J..."
  }
}
```

| HTTP | `code` | Uso |
|---|---|---|
| 400 | `BAD_REQUEST` | JSON o parámetro inválido. |
| 401 | `UNAUTHORIZED` / `TOKEN_EXPIRED` | Token ausente, inválido o vencido. |
| 403 | `FORBIDDEN` | Rol sin permiso. |
| 404 | `NOT_FOUND` | Recurso inexistente. |
| 409 | `CONFLICT` | Regla de estado, versión o disponibilidad. |
| 422 | `VALIDATION_ERROR` | Datos incompletos o inválidos. |
| 503 | `SERVICE_UNAVAILABLE` | MongoDB no disponible. |
| 500 | `INTERNAL_ERROR` | Error inesperado. |

### Códigos de dominio

| Código | HTTP | Significado |
|---|---|---|
| `SHIFT_NOT_OPEN` | 409 | No hay turno abierto. |
| `SHIFT_ALREADY_OPEN` | 409 | Ya existe un turno abierto. |
| `MENU_NOT_PUBLISHED` | 409 | No hay menú publicado para hoy. |
| `DISH_UNAVAILABLE` | 409 | Plato inactivo, agotado o no ofrecido. |
| `RATION_UNAVAILABLE` | 409 | Raciones insuficientes. |
| `TABLE_INACTIVE` | 409 | Mesa inactiva. |
| `TABLE_HAS_ACTIVE_ORDERS` | 409 | No se puede cerrar la mesa. |
| `INVALID_TRANSITION` | 409 | Transición no permitida. |
| `VERSION_CONFLICT` | 409 | El pedido cambió desde la lectura. |
| `CANCELLATION_NOT_ALLOWED` | 409 | Estado no cancelable. |
| `INVALID_REASON` | 422 | Falta el motivo obligatorio. |
| `OPEN_ORDERS_REMAIN` | 409 | Hay pedidos sin cerrar. |
| `OPEN_TABLES_REMAIN` | 409 | Hay mesas abiertas. |
| `DUPLICATE_USERNAME` | 409 | `username` existente. |

## 4. Salud

### `GET /api/health`

Público.

```json
{
  "status": "ok",
  "db": "connected",
  "uptimeSeconds": 1234,
  "timestamp": "2026-11-17T18:30:00.000Z"
}
```

## 5. Autenticación

### `POST /api/auth/login`

```json
{ "username": "mozo1", "password": "mozo123" }
```

**200:**

```json
{
  "token": "eyJhbGciOi...",
  "user": {
    "id": "66f0c0...",
    "username": "mozo1",
    "displayName": "Mozo 1",
    "role": "MOZO"
  }
}
```

**401** si credenciales inválidas o usuario inactivo.

### `GET /api/auth/me`

**200:** perfil sin `passwordHash`.

### `POST /api/auth/logout`

**204.** El cliente descarta el token.

## 6. Mesas y sesiones

### `GET /api/tables`

**200:**

```json
{
  "data": [
    {
      "id": "66f2...",
      "code": "M01",
      "zone": "Salón",
      "seats": 4,
      "state": "OCUPADA",
      "openSessionId": "66f5..."
    }
  ]
}
```

`state`: `LIBRE`, `OCUPADA` o `POR_CERRAR`.

### `POST /api/table-sessions/:id/close`

Roles: `MOZO`, `ENCARGADO`.

**Request:** `{ "notes": "Mesa atendida sin novedades" }`

**200:** sesión cerrada.

**409 `TABLE_HAS_ACTIVE_ORDERS`** si quedan pedidos `ENVIADO` o `LISTO`.

## 7. Turnos

### `GET /api/shifts/current`

**200:**

```json
{
  "id": "66f6...",
  "date": "2026-11-17",
  "label": "Almuerzo",
  "status": "ABIERTO",
  "openedAt": "2026-11-17T12:00:00.000Z"
}
```

**404** si no hay turno abierto.

### `POST /api/shifts` — `ENCARGADO`

**Request:** `{ "date": "2026-11-17", "label": "Almuerzo" }`

**201:** turno abierto. **409 `SHIFT_ALREADY_OPEN`** si ya existe uno.

### `POST /api/shifts/:id/close` — `ENCARGADO`

**Request:** `{ "observations": "Faltó pescado; se agotó el ceviche." }`

**200:** resumen generado.

**409 `OPEN_ORDERS_REMAIN`** o `OPEN_TABLES_REMAIN` si hay pendientes.

### `GET /api/shifts/:id/summary` — `ENCARGADO`

**200:**

```json
{
  "shiftId": "66f6...",
  "date": "2026-11-17",
  "label": "Almuerzo",
  "soldDishes": [
    { "dishId": "66f1...", "dishName": "Chicharrón de pollo", "quantity": 12, "amountCents": 21600 }
  ],
  "remainingRations": [
    { "dishId": "66f1...", "dishName": "Chicharrón de pollo", "remainingRations": 18 }
  ],
  "soldOutDishes": [
    { "dishId": "66f3...", "dishName": "Tallarín saltado", "reason": "Falta de insumo" }
  ],
  "cancellations": [
    { "orderNumber": 12, "reason": "Cliente se retiró", "cancelledBy": "Mozo 1" }
  ],
  "totalAmountCents": 125000,
  "observations": "Faltó pescado; se agotó el ceviche.",
  "closedAt": "2026-11-17T16:00:00.000Z"
}
```

> `totalAmountCents` es informativo; no es un cierre de caja ni una factura.

## 8. Menú

### `GET /api/menu/today`

Roles: todos. Devuelve solo el menú `PUBLICADO` de la fecha actual.

```json
{
  "menuDayId": "66f7...",
  "date": "2026-11-17",
  "sections": [
    {
      "key": "CRIOLLO",
      "name": "Criollo",
      "items": [
        {
          "id": "66f8...",
          "dishId": "66f1...",
          "dishName": "Chicharrón de pollo",
          "description": "Arroz, papa frita y ensalada",
          "priceCents": 1800,
          "course": "FONDO",
          "isFixed": false,
          "isRotating": true,
          "state": "ULTIMAS_RACIONES",
          "active": true,
          "hasRationsLimit": true,
          "totalRations": 30,
          "remainingRations": 5,
          "lowStockThreshold": 5,
          "manuallySoldOut": false,
          "version": 0,
          "variations": [
            { "id": "v1", "name": "Sin arroz" },
            { "id": "v2", "name": "Arroz por papa" },
            { "id": "v3", "name": "Papas extra" }
          ]
        }
      ]
    }
  ]
}
```

`state`: `INACTIVO`, `DISPONIBLE`, `ULTIMAS_RACIONES`, `AGOTADO`.

### `GET /api/menu/days/:date` — `ENCARGADO`

Devuelve el menú del día en borrador o publicado, con todos los ítems.

### `POST /api/menu/days` — `ENCARGADO`

**Request:** `{ "date": "2026-11-17", "notes": "Menú del lunes" }`

**201:** menú en `BORRADOR`.

### `PUT /api/menu/days/:id` — `ENCARGADO`

**Request:**

```json
{
  "notes": "Menú del lunes",
  "items": [
    {
      "dishId": "66f1...",
      "active": true,
      "hasRationsLimit": true,
      "totalRations": 30,
      "remainingRations": 30
    }
  ]
}
```

**200:** menú actualizado. Solo se permite editar en `BORRADOR`.

### `POST /api/menu/days/:id/publish` — `ENCARGADO`

**200:** menú `PUBLICADO`; visible en `/api/menu/today`.

**422** si no tiene ítems activos.

### `GET /api/menu-sections`

Roles: todos. Devuelve secciones activas.

### `GET /api/dishes` — `ENCARGADO`

Lista los platos maestros, incluidas variaciones y precios.

### `POST /api/dishes` — `ENCARGADO`

```json
{
  "sectionId": "66f0...",
  "name": "Chicharrón de pollo",
  "description": "Arroz, papa frita y ensalada",
  "priceCents": 1800,
  "course": "FONDO",
  "isFixed": false,
  "isRotating": true,
  "variations": [
    { "name": "Sin arroz" },
    { "name": "Arroz por papa" },
    { "name": "Papas extra" }
  ]
}
```

**201:** plato creado.

### `PATCH /api/dishes/:id` — `ENCARGADO`

Campos parciales: `name`, `description`, `priceCents`, `course`, `isFixed`, `isRotating`, `variations`, `active`.

### `DELETE /api/dishes/:id` — `ENCARGADO`

Desactiva el plato (`active: false`) sin borrar historial. **204.**

### `POST /api/menu-items/:id/rations/replenish`

Roles: `MOZO`, `ENCARGADO`.

```json
{ "quantity": 10, "reason": "Sobraron porciones; confirmado con cocina" }
```

**200:** raciones incrementadas y ajuste auditado.

### `POST /api/menu-items/:id/rations/adjust` — `ENCARGADO`

```json
{ "remainingRations": 12, "reason": "Conteo físico: quedaban 12" }
```

**200:** valor fijado y auditado.

### `POST /api/menu-items/:id/sold-out` — `ENCARGADO`

```json
{ "reason": "Falta de pescado" }
```

**200:** `manuallySoldOut: true`, estado `AGOTADO`.

### `POST /api/menu-items/:id/reactivate` — `ENCARGADO`

```json
{ "reason": "Llegó insumo" }
```

**200:** `manuallySoldOut: false`; el estado se recalcula por raciones.

### `GET /api/menu-items/:id/rations/adjustments` — `ENCARGADO`

**200:** historial de ajustes con actor, motivo, antes y después.

## 9. Pedidos

### `POST /api/orders` — `MOZO`

**Request:**

```json
{
  "tableId": "66f2...",
  "items": [
    {
      "dishId": "66f1...",
      "quantity": 2,
      "variationIds": ["v2"],
      "notes": "Sin ají"
    }
  ],
  "notes": "Mesa con niños",
  "clientRequestId": "3f1c2a9e-7b1d-4c2f-9a1e-2b6d8f0c4a11"
}
```

**Validaciones:**
- Turno abierto.
- Mesa activa.
- Menú publicado.
- Al menos un ítem; cantidad entera ≥ 1.
- Plato `DISPONIBLE` o `ULTIMAS_RACIONES`; nunca `AGOTADO`/`INACTIVO`.
- Raciones suficientes.
- Variaciones pertenecientes al plato.
- Nota de ítem ≤ 140 caracteres; nota general ≤ 280.

**201:**

```json
{
  "id": "66f4...",
  "orderNumber": 42,
  "table": { "id": "66f2...", "code": "M01", "zone": "Salón" },
  "tableSessionId": "66f5...",
  "takenBy": { "id": "66f0...", "name": "Mozo 1" },
  "status": "ENVIADO",
  "items": [
    {
      "dishId": "66f1...",
      "dishName": "Chicharrón de pollo",
      "sectionName": "Criollo",
      "unitPriceCents": 1800,
      "quantity": 2,
      "variations": [{ "id": "v2", "name": "Arroz por papa" }],
      "notes": "Sin ají",
      "lineTotalCents": 3600
    }
  ],
  "notes": "Mesa con niños",
  "totalCents": 3600,
  "currency": "PEN",
  "statusHistory": [
    {
      "status": "ENVIADO",
      "previousStatus": null,
      "at": "2026-11-17T12:30:00.000Z",
      "byUserId": "66f0...",
      "byName": "Mozo 1",
      "byRole": "MOZO",
      "version": 0
    }
  ],
  "enviadoEn": "2026-11-17T12:30:00.000Z",
  "version": 0
}
```

**Idempotencia:** si `clientRequestId` ya existe, devuelve `200` con el pedido existente.

**Errores:** `SHIFT_NOT_OPEN`, `MENU_NOT_PUBLISHED`, `DISH_UNAVAILABLE`, `RATION_UNAVAILABLE`, `TABLE_INACTIVE`, `VERSION_CONFLICT`.

### `GET /api/orders`

Roles: todos los autenticados. Todos los mozos ven todos los pedidos.

| Parámetro | Valores | Descripción |
|---|---|---|
| `scope` | `active` \| `ready` \| `all` | `active` = `ENVIADO`/`LISTO`; `ready` = `LISTO`; `all` = historial. |
| `status` | estado o lista | Filtro. |
| `tableId` | ObjectId | Filtro. |
| `shiftId` | ObjectId | Filtro. |
| `takenByUserId` | ObjectId | Solo `ENCARGADO`. |
| `page`, `limit` | number | Paginación. |

**200:** `{ data: [pedidos], meta: { page, limit, total, hasNext } }`.

### `GET /api/orders/:id`

Roles: todos los autenticados. **200:** pedido completo.

### `GET /api/kitchen/board` — `COCINERO`, `ENCARGADO`

**200:**

```json
{
  "pending": [ /* ENVIADO, ordenados por enviadoEn ASC */ ],
  "ready": [ /* LISTO, ordenados por listoEn ASC */ ],
  "serverTime": "2026-11-17T12:42:00.000Z"
}
```

- `pending` es FIFO.
- `ready` permanece visible hasta que un mozo entregue o cancele.
- No incluye tiempos transcurridos ni alertas de demora.

### `PATCH /api/orders/:id/ready` — `COCINERO`

**Request:** `{ "version": 0 }`

**Efecto:** `ENVIADO → LISTO`; set `readyByUserId`, `readyByName`, `listoEn`.

**200:** pedido actualizado.

**409** si el estado no es `ENVIADO` o la versión no coincide.

### `PATCH /api/orders/:id/deliver` — `MOZO`

**Request:** `{ "version": 1 }`

**Efecto:** `LISTO → ENTREGADO`; set `deliveredByUserId`, `deliveredByName`, `entregadoEn`.

**200:** pedido actualizado. Cualquier mozo puede entregar.

**409** si el estado no es `LISTO` o la versión no coincide.

### `PATCH /api/orders/:id/cancel` — `MOZO`

**Request:** `{ "reason": "Cliente se retiró", "version": 0 }`

**Reglas:**
- `ENVIADO`: cancela y **repone raciones**; registra `CANCELLATION_REVERSAL`.
- `LISTO`: cancela y **no repone raciones**; la UI debe advertir “coordina con cocina”.
- `ENTREGADO`/`CANCELADO`: `409 CANCELLATION_NOT_ALLOWED`.
- Sin motivo: `422 INVALID_REASON`.

**200:** pedido cancelado, con `warning` si estaba `LISTO`.

### `POST /api/orders/:id/manual-close` — `ENCARGADO` (P2)

**Request:** `{ "reason": "Entregado sin marcar; cierre de turno" }`

**Efecto:** `ENVIADO` o `LISTO → ENTREGADO`, `manualClose: true`, `manualCloseReason`, `deliveredBy`/`entregadoEn`.

**200:** pedido cerrado. Solo para cierre de turno.

## 10. Configuración

### `GET /api/settings`

Roles: todos.

```json
{
  "restaurantName": "Cevichería Lucy",
  "currency": "PEN",
  "lowStockThreshold": 5
}
```

### `PATCH /api/settings` — `ENCARGADO`

**Request:** `{ "lowStockThreshold": 5 }`

**200:** configuración actualizada. No existen campos de IGV ni facturación.

## 11. Registro por falla (P2)

### `POST /api/shifts/:id/manual-records` — `ENCARGADO`

```json
{
  "tableCode": "M04",
  "items": [
    { "dishName": "Chicharrón de pollo", "quantity": 2, "amountCents": 3600 }
  ],
  "note": "Registrado desde papel por caída del sistema"
}
```

**201:** registro `PAPER`.

- No envía a cocina.
- No descuenta raciones automáticamente.
- Se incluye en el resumen del turno.
- Si se requiere ajustar raciones, el encargado lo hace por separado con motivo.

## 12. Ejemplos de uso

```bash
# Login mozo
curl -s -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"mozo1","password":"mozo123"}'

# Menú de hoy
curl -s http://localhost:3000/api/menu/today \
  -H "Authorization: Bearer $TOKEN"

# Enviar pedido
curl -s -X POST http://localhost:3000/api/orders \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"tableId":"...","items":[{"dishId":"...","quantity":2,"variationIds":["v2"],"notes":"Sin ají"}],"clientRequestId":"uuid"}'

# Cola de cocina
curl -s http://localhost:3000/api/kitchen/board \
  -H "Authorization: Bearer $TOKEN_COCINA"

# Marcar listo
curl -s -X PATCH http://localhost:3000/api/orders/ID/ready \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN_COCINA" \
  -d '{"version":0}'
```
