# Tareas — Cocina (Pendientes y listos)

> Ejecutar después de `001`, `003` y el contrato de `005`. Las tareas `[P]` pueden ir en paralelo.

## Fase 0 — Setup

- [ ] **T001** Crear ruta `/cocina` con guard `COCINERO`. Criterio: acceso correcto.
- [ ] **T002** [P] Crear `kitchen.api.ts` con `getBoard` y `markReady`. Criterio: JWT y errores.
- [ ] **T003** [P] Crear `kitchenStore` en `client/src/stores/kitchenStore.ts`. Criterio: pending, ready, acciones y resync.
- [ ] **T004** [P] Crear tipos de cola en `shared/types/kitchen.ts`. Criterio: pedidos y columnas.
- [ ] **T005** Crear `useKitchenEvents` en `client/src/hooks/useKitchenEvents.ts`. Criterio: eventos y deduplicación.

## Fase 1 — Layout y componentes base

- [ ] **T006** Crear `KitchenPage` en `client/src/pages/cocina/KitchenPage.tsx`. Criterio: dos columnas y carga inicial.
- [ ] **T007** [P] Crear `KitchenHeader` con contadores y conexión. Criterio: pendientes/listos.
- [ ] **T008** [P] Crear `KitchenEmptyState`. Criterio: mensaje claro.
- [ ] **T009** [P] Crear `KitchenConnectionBanner`. Criterio: visible al desconectar.
- [ ] **T010** Crear `KitchenOrderCard` con mesa, número, ítems, variaciones y notas. Criterio: detalle completo.
- [ ] **T011** Crear `KitchenItems` con scroll y notas legibles. Criterio: 20 ítems.
- [ ] **T012** Estilo oscuro de alto contraste y objetivos ≥ 44 px. Criterio: legible a distancia.

## Fase 2 — US-001: Pendientes FIFO (P1)

- [ ] **T013** [US1] Cargar `/api/kitchen/board` al abrir. Criterio: pendientes y listos.
- [ ] **T014** [US1] Ordenar pendientes por `enviadoEn` ascendente. Criterio: FIFO.
- [ ] **T015** [US1] Excluir `LISTO`, `ENTREGADO` y `CANCELADO` de pendientes. Criterio: solo `ENVIADO`.
- [ ] **T016** [US1] Crear `PendingColumn` con scroll. Criterio: 30 pedidos.
- [ ] **T017** [US1] Crear pruebas de orden FIFO. Criterio: 10 pedidos conocidos.

## Fase 3 — US-002: Detalle (P1)

- [ ] **T018** [US2] Mostrar variaciones resaltadas. Criterio: legibles.
- [ ] **T019** [US2] Mostrar notas completas. Criterio: sin truncar.
- [ ] **T020** [US2] Permitir expandir tarjeta con muchos ítems. Criterio: scroll interno.
- [ ] **T021** [US2] Crear pruebas de detalle. Criterio: variaciones y notas.

## Fase 4 — US-003: Marcar listo (P1)

- [ ] **T022** [US3] Implementar botón “Marcar listo” solo en `ENVIADO`. Criterio: un toque.
- [ ] **T023** [US3] Integrar `PATCH /api/orders/:id/ready` con `version`. Criterio: registra `readyBy`.
- [ ] **T024** [US3] Deshabilitar el botón mientras la acción está en curso. Criterio: sin doble transición.
- [ ] **T025** [US3] Mover el pedido de pendientes a listos al confirmar. Criterio: sin recargar.
- [ ] **T026** [US3] Manejar `409` con aviso y resync. Criterio: sin estado inconsistente.
- [ ] **T027** [US3] Crear pruebas de “Marcar listo” y doble tap. Criterio: una transición.

## Fase 5 — US-004: Listos (P1)

- [ ] **T028** [US4] Crear `ReadyColumn` con pedidos `LISTO`. Criterio: visible.
- [ ] **T029** [US4] Mostrar “Listo para recoger” sin botón de cocina. Criterio: solo informativo.
- [ ] **T030** [US4] Retirar de listos al recibir `order:delivered`. Criterio: sin recargar.
- [ ] **T031** [US4] Retirar de listos al recibir `order:cancelled`. Criterio: aviso.
- [ ] **T032** [US4] Crear pruebas de columnas. Criterio: mover/retirar.

## Fase 6 — US-005: Nuevos pedidos (P1)

- [ ] **T033** [US5] Integrar `order:created` para insertar en pendientes. Criterio: FIFO.
- [ ] **T034** [US5] Resaltar tarjeta nueva por 2 s. Criterio: animación.
- [ ] **T035** [US5] Anunciar nuevos pedidos con `aria-live`. Criterio: accesible.
- [ ] **T036** [US5] Deduplicar por `eventId` y `version`. Criterio: sin duplicados.
- [ ] **T037** [US5] Crear pruebas de inserción y resaltado. Criterio: posición correcta.

## Fase 7 — US-006: Reconexión (P1)

- [ ] **T038** [US6] Resincronizar al reconectar. Criterio: cola completa.
- [ ] **T039** [US6] Mostrar banner de conexión. Criterio: visible.
- [ ] **T040** [US6] Reconstruir cola tras reinicio del servidor. Criterio: sin intervención.
- [ ] **T041** [US6] Crear prueba de 60 s desconectado. Criterio: resync ≤ 5 s.

## Fase 8 — Pulido y pruebas

- [ ] **T042** [P] Medir render de 30 pedidos. Criterio: ≤ 1 s.
- [ ] **T043** [P] Revisar legibilidad 1024×768 y pantallas grandes. Criterio: sin truncar.
- [ ] **T044** [P] Revisar accesibilidad: contraste, foco, `aria-live`. Criterio: AA básico.
- [ ] **T045** Verificar que no existan tiempos/cronómetros. Criterio: búsqueda sin resultados.
- [ ] **T046** Verificar que no existan acciones de stock o “En preparación”. Criterio: búsqueda sin resultados.
- [ ] **T047** Verificar que no existan pagos/IGV. Criterio: búsqueda sin resultados.
- [ ] **T048** Crear E2E mozo → KDS → listo → entrega. Criterio: pasa.
- [ ] **T049** Actualizar `quickstart.md` y estado de la spec. Criterio: documentación sincronizada.
- [ ] **T050** Ejecutar análisis de consistencia. Criterio: sin hallazgos críticos.

## Dependencias

```text
001 ─┐
003 ─┼─► Fase 0 → Fase 1 → Fase 2 → Fase 3 → Fase 4 → Fase 5 → Fase 6 → Fase 7 → Fase 8
005 ─┘
```

## Definition of Done

- [ ] Pendientes FIFO.
- [ ] Detalle completo.
- [ ] “Listo” en un toque.
- [ ] Listos visibles hasta entrega/cancelación.
- [ ] Nuevos pedidos ≤ 2 s.
- [ ] Resync ≤ 5 s.
- [ ] Sin tiempos, stock, “En preparación”, pagos ni IGV.
- [ ] Pruebas y consistencia.
