# Tareas — Toma de pedidos (Mozo)

> Ejecutar después de `001`, `002` y el contrato de `005`. Las tareas `[P]` pueden ir en paralelo.

## Fase 0 — Setup

- [ ] **T001** Crear rutas `/mozo`, `/mozo/pedido/nuevo`, `/mozo/pedidos`, `/mozo/pedidos/:id` con guard `MOZO`. Criterio: acceso correcto.
- [ ] **T002** [P] Crear `orders.api.ts` con crear, listar, detalle, entregar y cancelar. Criterio: JWT y errores normalizados.
- [ ] **T003** [P] Crear `tables.api.ts` con listar y cerrar sesión. Criterio: estados de mesa.
- [ ] **T004** [P] Crear tipos de pedido y borrador en `shared/types/order.ts`. Criterio: estados y DTOs.
- [ ] **T005** [P] Crear utilidades de dinero y fecha en `client/src/utils/`. Criterio: `S/` y `America/Lima`.

## Fase 1 — Base compartida

- [ ] **T006** Implementar `orderDraftStore` en `client/src/stores/orderDraftStore.ts`. Criterio: mesa, ítems, notas, `clientRequestId`.
- [ ] **T007** Implementar `ordersStore` en `client/src/stores/ordersStore.ts`. Criterio: activos, listos, detalle, loading/error.
- [ ] **T008** Implementar `tablesStore` en `client/src/stores/tablesStore.ts`. Criterio: mesas y cierre.
- [ ] **T009** Implementar `useOrderEvents` en `client/src/hooks/useOrderEvents.ts`. Criterio: `order:created`, `order:ready`, `order:delivered`, `order:cancelled`, resync.
- [ ] **T010** Implementar `TableSelector` en `client/src/components/mozo/TableSelector.tsx`. Criterio: estados `LIBRE`/`OCUPADA`/`POR_CERRAR`.
- [ ] **T011** [P] Implementar `QuantityStepper` y `OrderCart`. Criterio: cantidades y eliminación.
- [ ] **T012** [P] Implementar `VariationSheet` y `ItemNotesModal`. Criterio: solo variaciones permitidas; notas 140.
- [ ] **T013** Implementar `OrderSummary` con total sin IGV. Criterio: suma exacta.
- [ ] **T014** Implementar `OrderStatusBadge` y `ReadyOrderCard`. Criterio: estados y sin tiempos.

## Fase 2 — US-001: Armar pedido (P1)

- [ ] **T015** [US1] Crear `NewOrderPage` en `client/src/pages/mozo/NewOrderPage.tsx`. Criterio: mesa + menú + carrito.
- [ ] **T016** [US1] Integrar `MenuView` de `002` con disponibilidad y raciones. Criterio: agotados deshabilitados.
- [ ] **T017** [US1] Validar que la cantidad no supere las raciones restantes. Criterio: límite en UI y servidor.
- [ ] **T018** [US1] Asociar variaciones al ítem y validar pertenencia. Criterio: `422` si inválida.
- [ ] **T019** [US1] Guardar notas por ítem y nota general. Criterio: límites 140/280.
- [ ] **T020** [US1] Mostrar resumen con mesa, ítems, variaciones, notas y total. Criterio: total correcto.
- [ ] **T021** [US1] Crear pruebas de armado y validaciones. Criterio: agotado, raciones, variaciones.

## Fase 3 — US-002: Enviar pedido (P1)

- [ ] **T022** [US2] Validar turno abierto, menú publicado, mesa e ítems. Criterio: mensajes claros.
- [ ] **T023** [US2] Implementar envío con `clientRequestId` y bloqueo de doble tap. Criterio: un solo pedido.
- [ ] **T024** [US2] Manejar `201` y `200` idempotente. Criterio: no duplicar en lista.
- [ ] **T025** [US2] Mostrar confirmación con número y estado `ENVIADO`. Criterio: limpiar borrador.
- [ ] **T026** [US2] Manejar `DISH_UNAVAILABLE`/`RATION_UNAVAILABLE`. Criterio: refrescar menú/carrito.
- [ ] **T027** [US2] Impedir edición después de enviar. Criterio: solo detalle.
- [ ] **T028** [US2] Crear pruebas de envío, idempotencia y errores. Criterio: 0 duplicados.

## Fase 4 — US-003: Activos y listos (P1)

- [ ] **T029** [US3] Crear `WaiterDashboardPage` y `ActiveOrdersPage`. Criterio: todos los pedidos activos.
- [ ] **T030** [US3] Mostrar mesa, ítems, estado y quién tomó. Criterio: sin filtro por mozo.
- [ ] **T031** [US3] Implementar panel de listos con “Listo para recoger”. Criterio: visible para todos.
- [ ] **T032** [US3] Actualizar por Socket.IO sin recargar. Criterio: `order:ready`/`delivered`.
- [ ] **T033** [US3] Implementar resync al reconectar. Criterio: activos y listos correctos.
- [ ] **T034** [US3] Crear pruebas con dos mozos. Criterio: ambos ven lo mismo.

## Fase 5 — US-004: Entregar (P1)

- [ ] **T035** [US4] Implementar botón “Entregado” solo en `LISTO`. Criterio: visible para cualquier mozo.
- [ ] **T036** [US4] Integrar `PATCH /api/orders/:id/deliver` con `version`. Criterio: registra `deliveredBy`.
- [ ] **T037** [US4] Manejar `409` y refrescar. Criterio: sin estado local inconsistente.
- [ ] **T038** [US4] Crear pruebas de entrega. Criterio: solo `LISTO`; cualquier mozo.

## Fase 6 — US-005: Cancelar (P2)

- [ ] **T039** [US5] Implementar `CancelOrderModal` con motivo obligatorio. Criterio: no confirma sin texto.
- [ ] **T040** [US5] Implementar advertencia para `LISTO` (“Coordina con cocina”). Criterio: visible antes de confirmar.
- [ ] **T041** [US5] Integrar `PATCH /api/orders/:id/cancel` con `version`. Criterio: regla de raciones.
- [ ] **T042** [US5] Mostrar cancelados en historial con motivo. Criterio: trazabilidad.
- [ ] **T043** [US5] Crear pruebas de cancelación. Criterio: Enviado repone; Listo no; terminal rechazado.

## Fase 7 — US-006: Cerrar mesa (P1)

- [ ] **T044** [US6] Implementar `CloseTableButton` solo sin pedidos activos. Criterio: `409` si hay.
- [ ] **T045** [US6] Integrar `POST /api/table-sessions/:id/close`. Criterio: registra `closedBy`.
- [ ] **T046** [US6] Actualizar estado de mesa a `LIBRE`. Criterio: refresco.
- [ ] **T047** [US6] Crear pruebas de cierre y reapertura. Criterio: nueva sesión al volver a pedir.

## Fase 8 — US-007: Borrador persistente (P2)

- [ ] **T048** [US7] Persistir borrador con `zustand/persist`. Criterio: recarga conserva.
- [ ] **T049** [US7] Limpiar borrador al enviar o descartar. Criterio: no reaparece.
- [ ] **T050** [US7] Crear pruebas de persistencia. Criterio: restauración y limpieza.

## Fase 9 — Pulido y pruebas

- [ ] **T051** [P] Revisar responsive 360/768/1024 px. Criterio: sin cortes.
- [ ] **T052** [P] Revisar accesibilidad: labels, foco, `aria-live`. Criterio: checklist AA básico.
- [ ] **T053** Verificar que no se muestren tiempos ni cronómetros. Criterio: búsqueda sin resultados.
- [ ] **T054** Verificar que no existan pagos, IGV ni propinas. Criterio: búsqueda sin resultados.
- [ ] **T055** Medir tiempo de pedido de 3 ítems. Criterio: ≤ 60 s.
- [ ] **T056** Crear E2E mozo → cocina → listo → entregado. Criterio: pasa.
- [ ] **T057** Actualizar `quickstart.md` y estado de la spec. Criterio: documentación sincronizada.
- [ ] **T058** Ejecutar análisis de consistencia. Criterio: sin hallazgos críticos.

## Dependencias

```text
001 ─┐
002 ─┼─► Fase 0 → Fase 1 → Fase 2 → Fase 3 → Fase 4 → Fase 5 → Fase 6 → Fase 7 → Fase 8 → Fase 9
005 ─┘
```

## Definition of Done

- [ ] Pedido creado y enviado.
- [ ] Raciones exactas y sin duplicados.
- [ ] Todos los mozos ven/entregan.
- [ ] Cancelación aplica regla.
- [ ] Cierre de mesa.
- [ ] Sin tiempos, pagos ni IGV.
- [ ] Pruebas y consistencia.
