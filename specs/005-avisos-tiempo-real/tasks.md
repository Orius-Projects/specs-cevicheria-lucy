# Tareas — Avisos en tiempo real

> Ejecutar después de `001` y de los contratos de `003`/`004`. Las tareas `[P]` pueden ir en paralelo.

## Fase 0 — Setup

- [ ] **T001** Adjuntar Socket.IO al HTTP server en `server/src/server.ts`. Criterio: mismo puerto que Express.
- [ ] **T002** [P] Configurar CORS de Socket.IO. Criterio: mismo `CORS_ORIGIN`.
- [ ] **T003** [P] Crear envelope en `server/src/realtime/envelope.ts`. Criterio: `eventId`, `type`, `version`, `schemaVersion`, `payload`.
- [ ] **T004** [P] Crear salas en `server/src/realtime/rooms.ts`. Criterio: `role:*` y `user:<id>`.
- [ ] **T005** Instalar `socket.io` y `socket.io-client`. Criterio: build.

## Fase 1 — Autenticación y salas (bloqueante)

- [ ] **T006** Implementar handshake JWT en `server/src/realtime/socket.ts`. Criterio: sin token `UNAUTHORIZED`.
- [ ] **T007** Rechazar usuarios inactivos. Criterio: `FORBIDDEN`.
- [ ] **T008** Unir a `role:<rol>` y `user:<id>`. Criterio: salas correctas.
- [ ] **T009** Exponer estado de conexión al cliente. Criterio: hook compartido.
- [ ] **T010** Crear pruebas de handshake y salas. Criterio: 0 fugas.

## Fase 2 — Emisión de pedidos

- [ ] **T011** Implementar `emitOrderCreated`. Criterio: cocina, mozos y encargado.
- [ ] **T012** [P] Implementar `emitOrderReady`. Criterio: **todos** los mozos.
- [ ] **T013** [P] Implementar `emitOrderDelivered` y `emitOrderCancelled`. Criterio: salas correctas.
- [ ] **T014** [P] Implementar `emitOrderManualClosed` P2. Criterio: encargado.
- [ ] **T015** Integrar emisión después del commit en `order.service.ts`. Criterio: no emitir antes.
- [ ] **T016** Integrar emisión en `order-transition.service.ts`. Criterio: un evento por transición.
- [ ] **T017** Registrar fallos de emisión sin revertir. Criterio: `EMIT_FAILED`.
- [ ] **T018** Crear pruebas de emisión. Criterio: eventos y salas.

## Fase 3 — Disponibilidad

- [ ] **T019** Implementar `emitMenuAvailabilityChanged`. Criterio: `menuItemId`, `state`, `remainingRations`, `changeType`.
- [ ] **T020** Emitir al descontar raciones. Criterio: después del commit.
- [ ] **T021** Emitir al reponer/ajustar. Criterio: actor y motivo.
- [ ] **T022** Emitir al marcar agotado/reactivar. Criterio: estado.
- [ ] **T023** Crear pruebas de eventos de menú. Criterio: 100% de cambios.

## Fase 4 — Cliente

- [ ] **T024** Crear singleton de Socket.IO en `client/src/api/socket.ts`. Criterio: token y reconexión.
- [ ] **T025** Implementar `useRealtimeConnection`. Criterio: `connected/disconnected/error`.
- [ ] **T026** Implementar dedupe por `eventId`. Criterio: `Set`.
- [ ] **T027** Implementar comparación de `version`. Criterio: ignorar viejos.
- [ ] **T028** Implementar `useOrderEvents` para mozos. Criterio: created/ready/delivered/cancelled/availability.
- [ ] **T029** Implementar `useKitchenEvents` para cocina. Criterio: created/ready/delivered/cancelled.
- [ ] **T030** Cerrar socket al logout/cambiar usuario. Criterio: sin zombis.
- [ ] **T031** Crear pruebas de dedupe y orden. Criterio: 1 000 eventos sin duplicados.

## Fase 5 — Notificaciones

- [ ] **T032** Implementar `notificationsStore` con toast y badge. Criterio: dedupe.
- [ ] **T033** Integrar `order:ready` a **todos** los mozos. Criterio: dos mozos.
- [ ] **T034** Implementar sonido con fallback visual. Criterio: autoplay bloqueado.
- [ ] **T035** Anunciar con `aria-live`. Criterio: accesible.
- [ ] **T036** Crear pruebas de notificación. Criterio: una por evento.

## Fase 6 — Reconexión y resync

- [ ] **T037** Resync de cocina con `/api/kitchen/board`. Criterio: cola completa.
- [ ] **T038** Resync de mozo con `/api/orders?scope=active` y `/api/menu/today`. Criterio: pedidos y menú.
- [ ] **T039** Resync al volver del background. Criterio: estado correcto.
- [ ] **T040** Mostrar banner de conexión. Criterio: visible.
- [ ] **T041** Manejar `UNAUTHORIZED`/`TOKEN_EXPIRED` con login. Criterio: redirección.
- [ ] **T042** Crear prueba de 60 s desconectado. Criterio: resync ≤ 5 s.

## Fase 7 — Pruebas y pulido

- [ ] **T043** [P] Probar 20 clientes y 100 eventos. Criterio: p95 ≤ 2 s.
- [ ] **T044** [P] Revisar logs con `eventId`, tipo y pedido. Criterio: diagnóstico.
- [ ] **T045** Verificar que no existan eventos de tiempo, pago o IGV. Criterio: búsqueda sin resultados.
- [ ] **T046** Probar servidor reiniciado y reconexión. Criterio: sin intervención.
- [ ] **T047** Actualizar `contracts/events.md` si cambió algo. Criterio: contrato sincronizado.
- [ ] **T048** Actualizar `quickstart.md` y estado de la spec. Criterio: documentación sincronizada.
- [ ] **T049** Ejecutar análisis de consistencia. Criterio: sin hallazgos críticos.

## Dependencias

```text
001 ─┐
003 ─┼─► Fase 0 → Fase 1 → Fase 2 → Fase 3 → Fase 4 → Fase 5 → Fase 6 → Fase 7
004 ─┘
```

## Definition of Done

- [ ] Handshake y salas.
- [ ] Eventos de pedido y menú.
- [ ] Todos los mozos reciben “Listo”.
- [ ] Dedupe y orden.
- [ ] Resync.
- [ ] Sin push externo, tiempos ni pagos.
- [ ] 20 clientes y p95 ≤ 2 s.
