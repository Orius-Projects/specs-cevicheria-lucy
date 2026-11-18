---
feature: 005-avisos-tiempo-real
titulo: Contrato de eventos Socket.IO v2
version: 2.0.0
estado: borrador
creado: 2026-11-17
actualizado: 2026-11-17
fuentes:
  - docs/sources/DOCUMENTO_PROCESOS_CEVICHERIA_LUCY.pdf
---

# Contrato de eventos Socket.IO — Cevichería Lucy v2

> Contrato canónico para `003`, `004` y `005`. No hay eventos de tiempo, pagos ni IGV.

## 1. Conexión

```ts
import { io } from 'socket.io-client';

const socket = io(import.meta.env.VITE_SOCKET_URL, {
  auth: { token },
  transports: ['websocket', 'polling'],
  reconnection: true,
  reconnectionAttempts: Infinity,
  reconnectionDelay: 500,
  reconnectionDelayMax: 5000
});
```

- JWT en `handshake.auth.token`.
- Sin token válido: `connect_error` con `UNAUTHORIZED`, `TOKEN_EXPIRED` o `FORBIDDEN`.
- El servidor valida usuario activo.

## 2. Salas

| Sala | Miembros | Recibe |
|---|---|---|
| `role:MOZO` | Todos los mozos | `order:created`, `order:ready`, `order:delivered`, `order:cancelled`, `menu:availability-changed`, `shift:closed` |
| `role:COCINERO` | Cocineros | `order:created`, `order:ready`, `order:delivered`, `order:cancelled`, `order:manual-closed`, `shift:closed` |
| `role:ENCARGADO` | Encargados | Todos los eventos |
| `user:<id>` | Pestañas del usuario | Eventos dirigidos (opcional) |

**Regla:** el servidor asigna las salas; el cliente no puede suscribirse a salas ajenas. Todos los mozos reciben `order:ready`, sin filtro por quién tomó el pedido.

## 3. Envelope

```json
{
  "eventId": "0f5b0c2e-9a4a-4d0e-8f4f-1a2b3c4d5e6f",
  "type": "order:ready",
  "occurredAt": "2026-11-17T12:30:00.000Z",
  "version": 2,
  "schemaVersion": 2,
  "orderId": "66f4c0...",
  "payload": {
    "order": { },
    "previousStatus": "ENVIADO",
    "changedBy": {
      "id": "66f9...",
      "name": "Cocinero 1",
      "role": "COCINERO"
    }
  }
}
```

| Campo | Descripción |
|---|---|
| `eventId` | UUID v4; deduplicación. |
| `type` | Nombre del evento. |
| `occurredAt` | Fecha del servidor. |
| `version` | `order.version` o versión del menú. |
| `schemaVersion` | `2` en esta entrega. |
| `orderId` | Pedido, si aplica. |
| `payload` | Datos del evento. |

## 4. Snapshot del pedido

Incluye los campos del contrato REST excepto `statusHistory` y `clientRequestId`:

```json
{
  "id": "66f4c0...",
  "orderNumber": 42,
  "table": { "id": "66f2...", "code": "M01", "zone": "Salón" },
  "tableSessionId": "66f5...",
  "takenBy": { "id": "66f0...", "name": "Mozo 1" },
  "status": "LISTO",
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
  "totalCents": 3600,
  "currency": "PEN",
  "enviadoEn": "2026-11-17T12:20:00.000Z",
  "listoEn": "2026-11-17T12:30:00.000Z",
  "entregadoEn": null,
  "canceladoEn": null,
  "cancelReason": null,
  "version": 2
}
```

**Sin** pagos, IGV, cliente tributario ni facturación.

## 5. Eventos

### 5.1 `order:created`

**Cuándo:** después de crear un pedido `ENVIADO`.
**Salas:** `role:COCINERO`, `role:MOZO`, `role:ENCARGADO`.

```json
{
  "eventId": "uuid",
  "type": "order:created",
  "occurredAt": "2026-11-17T12:20:00.000Z",
  "version": 0,
  "schemaVersion": 2,
  "orderId": "66f4c0...",
  "payload": {
    "order": { "status": "ENVIADO", "orderNumber": 42 },
    "previousStatus": null,
    "changedBy": { "id": "66f0...", "name": "Mozo 1", "role": "MOZO" }
  }
}
```

### 5.2 `order:ready`

**Cuándo:** después de `ENVIADO → LISTO`.
**Salas:** `role:MOZO`, `role:COCINERO`, `role:ENCARGADO`.

```json
{
  "eventId": "uuid",
  "type": "order:ready",
  "occurredAt": "2026-11-17T12:30:00.000Z",
  "version": 1,
  "schemaVersion": 2,
  "orderId": "66f4c0...",
  "payload": {
    "order": { "status": "LISTO", "listoEn": "2026-11-17T12:30:00.000Z" },
    "previousStatus": "ENVIADO",
    "changedBy": { "id": "66f9...", "name": "Cocinero 1", "role": "COCINERO" }
  }
}
```

**Uso en el mozo:** toast, badge y sonido si está habilitado. Todos los mozos lo reciben.

### 5.3 `order:delivered`

**Cuándo:** después de `LISTO → ENTREGADO`.
**Salas:** `role:COCINERO`, `role:MOZO`, `role:ENCARGADO`.

```json
{
  "eventId": "uuid",
  "type": "order:delivered",
  "occurredAt": "2026-11-17T12:35:00.000Z",
  "version": 2,
  "schemaVersion": 2,
  "orderId": "66f4c0...",
  "payload": {
    "order": { "status": "ENTREGADO", "entregadoEn": "2026-11-17T12:35:00.000Z" },
    "previousStatus": "LISTO",
    "changedBy": { "id": "66f0...", "name": "Mozo 1", "role": "MOZO" }
  }
}
```

### 5.4 `order:cancelled`

**Cuándo:** después de cancelar.
**Salas:** `role:COCINERO`, `role:MOZO`, `role:ENCARGADO`.

```json
{
  "eventId": "uuid",
  "type": "order:cancelled",
  "occurredAt": "2026-11-17T12:36:00.000Z",
  "version": 3,
  "schemaVersion": 2,
  "orderId": "66f4c0...",
  "payload": {
    "order": { "status": "CANCELADO", "cancelReason": "Cliente se retiró" },
    "previousStatus": "ENVIADO",
    "changedBy": { "id": "66f0...", "name": "Mozo 1", "role": "MOZO" }
  }
}
```

### 5.5 `order:manual-closed` (P2)

**Cuándo:** el encargado cierra manualmente un pedido en P-06.
**Salas:** `role:COCINERO`, `role:MOZO`, `role:ENCARGADO`.

Mismo envelope; `payload.order.status = "ENTREGADO"` y `manualClose: true`.

### 5.6 `menu:availability-changed`

**Cuándo:** cambia el estado/raciones de un `menuItem`.
**Salas:** `role:MOZO`, `role:ENCARGADO`, `role:COCINERO` (lectura).

```json
{
  "eventId": "uuid",
  "type": "menu:availability-changed",
  "occurredAt": "2026-11-17T12:40:00.000Z",
  "version": 4,
  "schemaVersion": 2,
  "orderId": null,
  "payload": {
    "menuDayId": "66f7...",
    "menuItemId": "66f8...",
    "dishId": "66f1...",
    "dishName": "Chicharrón de pollo",
    "state": "AGOTADO",
    "remainingRations": 0,
    "manuallySoldOut": false,
    "changeType": "RATION_DECREMENT",
    "changedBy": { "id": "66f0...", "name": "Mozo 1", "role": "MOZO" }
  }
}
```

`changeType`: `RATION_DECREMENT`, `REPLENISH`, `ADJUSTMENT`, `SOLD_OUT`, `REACTIVATE`, `CANCELLATION_REVERSAL`.

### 5.7 `shift:closed`

**Cuándo:** el encargado cierra el turno.
**Salas:** todas.

```json
{
  "eventId": "uuid",
  "type": "shift:closed",
  "occurredAt": "2026-11-17T16:00:00.000Z",
  "version": 1,
  "schemaVersion": 2,
  "orderId": null,
  "payload": {
    "shiftId": "66f6...",
    "date": "2026-11-17",
    "label": "Almuerzo",
    "closedBy": { "id": "66fa...", "name": "Encargado 1", "role": "ENCARGADO" }
  }
}
```

## 6. Reglas de emisión

1. Emitir después de persistir la transición.
2. Si la emisión falla, no revertir la transición; registrar `EMIT_FAILED`.
3. No emitir eventos de tiempo, demora, pago, IGV ni facturación.
4. `order:ready` va a **todos** los mozos, no solo al que tomó el pedido.
5. `menu:availability-changed` se emite después de cada descuento, reposición, ajuste, agotado o reactivación.
6. Un evento puede llegar más de una vez; el cliente deduplica por `eventId`.

## 7. Consumo en el cliente

```ts
const seen = new Set<string>();

socket.on('order:ready', (event) => {
  if (seen.has(event.eventId)) return;
  seen.add(event.eventId);

  const current = store.ordersById[event.orderId];
  if (current && event.version <= current.version) return;

  store.upsert(event.payload.order);
  notifications.push({
    id: event.eventId,
    orderId: event.orderId,
    title: `Mesa ${event.payload.order.table.code} · Listo`,
    body: `${event.payload.order.orderNumber} para recoger`,
    createdAt: event.occurredAt
  });
});
```

- Ignorar `version` menor o igual.
- Al reconectar: `GET /api/kitchen/board` (cocina), `GET /api/orders?scope=active` (mozo), `GET /api/menu/today` (menú).

## 8. Reconexión

| Situación | Acción |
|---|---|
| `disconnect` | Banner “Desconectado”. |
| `connect` | Resync REST y aceptar eventos nuevos. |
| `connect_error` `UNAUTHORIZED`/`TOKEN_EXPIRED` | Login. |
| Error de red | Reintentar con backoff. |
| Servidor reiniciado | Reconectar y resync. |
| Pestaña en segundo plano | Al volver, resync. |

No hay replay de eventos; la fuente de verdad es REST.

## 9. Errores

| Código | Cuándo | Acción |
|---|---|---|
| `UNAUTHORIZED` | Sin token. | Login. |
| `TOKEN_EXPIRED` | Token vencido. | Login. |
| `FORBIDDEN` | Usuario inactivo/rol. | Cerrar sesión. |
| `ROOM_NOT_ALLOWED` | Suscripción manual. | Ignorar. |
| `EMIT_FAILED` | Falla interna. | Log; resync. |

## 10. Versionado

- `schemaVersion: 2`.
- Cambios aditivos no rompen.
- Cambios incompatibles: nueva versión y convivencia.
- `version` es del pedido o del menú, no del contrato.

## 11. Pruebas del contrato

- [ ] Sin token → `UNAUTHORIZED`.
- [ ] Mozo → `role:MOZO` y `user:<id>`.
- [ ] Cocinero → `role:COCINERO`.
- [ ] `order:created` llega a cocina y mozos.
- [ ] `order:ready` llega a **todos** los mozos.
- [ ] Evento duplicado no duplica notificación.
- [ ] `version` menor se ignora.
- [ ] `menu:availability-changed` actualiza el menú.
- [ ] Reconexión tras 60 s resincroniza.
- [ ] 20 clientes mantienen p95 ≤ 2 s.
- [ ] No hay eventos de tiempo/pago.
