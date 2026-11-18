# Tareas — Menú, raciones y agotados

> Ejecutar después de `001`. Las tareas `[P]` pueden ir en paralelo dentro de su fase. No se implementan costos, IGV, pagos ni tiempos.

## Fase 0 — Setup

- [ ] **T001** Crear `menu.api.ts` con secciones, platos, menú del día, publicación, raciones y agotados. Criterio: JWT y errores normalizados.
- [ ] **T002** [P] Crear `menuAdminStore` en `client/src/stores/menuAdminStore.ts`. Criterio: día, platos, secciones y acciones.
- [ ] **T003** [P] Crear `menuPublicStore` en `client/src/stores/menuPublicStore.ts`. Criterio: menú publicado y evento de disponibilidad.
- [ ] **T004** [P] Crear `dishState.ts` en `client/src/utils/`. Criterio: misma derivación que el servidor.
- [ ] **T005** Crear tipos de menú en `shared/types/menu.ts`. Criterio: `Dish`, `MenuDay`, `MenuItem`, `DishState`.

## Fase 1 — Maestro de platos (bloqueante)

- [ ] **T006** Implementar `GET /api/menu-sections` y `GET /api/dishes` en el cliente. Criterio: secciones y platos cargados.
- [ ] **T007** [P] Crear `DishesPage` en `client/src/pages/encargado/DishesPage.tsx`. Criterio: lista y filtros por sección.
- [ ] **T008** Implementar alta de plato con sección, nombre, descripción, precio, tipo, fijo/rotativo. Criterio: validación.
- [ ] **T009** Implementar `VariationEditor` para variaciones sin costo. Criterio: agregar/editar/eliminar.
- [ ] **T010** Implementar edición y baja lógica de platos. Criterio: historial conservado.
- [ ] **T011** Crear pruebas de maestro de platos. Criterio: validaciones y variaciones.

## Fase 2 — US-001: Menú del día (P1)

- [ ] **T012** [US1] Crear `MenuDayPage` en `client/src/pages/encargado/MenuDayPage.tsx`. Criterio: selector de fecha y estado.
- [ ] **T013** [US1] Implementar `POST /api/menu/days` y `PUT /api/menu/days/:id`. Criterio: borrador editable.
- [ ] **T014** [US1] Implementar `MenuSectionTabs` y `DishPicker`. Criterio: agregar platos por sección.
- [ ] **T015** [US1] Implementar activación/desactivación de platos del día. Criterio: reflejado en el borrador.
- [ ] **T016** [US1] Implementar `PublishMenuButton` con validación de ítems activos. Criterio: `422` si vacío.
- [ ] **T017** [US1] Implementar publicación y refresco del menú del mozo. Criterio: visible en `/api/menu/today`.
- [ ] **T018** [US1] Crear pruebas de creación y publicación. Criterio: borrador, publicación y error vacío.

## Fase 3 — US-002: Raciones y variaciones (P1)

- [ ] **T019** [US2] Implementar configuración de `hasRationsLimit`, `totalRations`, `remainingRations`. Criterio: validación ≥ 0.
- [ ] **T020** [US2] Implementar `RationInput` y `RationBadge`. Criterio: muestra restantes y límite.
- [ ] **T021** [US2] Asociar variaciones del plato al menú del día. Criterio: solo las permitidas.
- [ ] **T022** [US2] Validar que las variaciones no alteren el total. Criterio: precio de línea = precio × cantidad.
- [ ] **T023** [US2] Crear pruebas de raciones y variaciones. Criterio: límites, sin costo y estado.

## Fase 4 — US-003: Menú publicado (P1)

- [ ] **T024** [US3] Implementar `MenuView` reutilizable en `client/src/pages/shared/MenuView.tsx`. Criterio: tabs por sección y badges.
- [ ] **T025** [US3] Implementar `AvailabilityBadge` con `DISPONIBLE`, `ULTIMAS_RACIONES`, `AGOTADO`. Criterio: colores y textos.
- [ ] **T026** [US3] Mostrar raciones restantes cuando hay límite. Criterio: “Quedan N”.
- [ ] **T027** [US3] Deshabilitar “Agregar” para agotados/inactivos. Criterio: no se puede seleccionar.
- [ ] **T028** [US3] Mostrar toast cuando un plato se agota. Criterio: mensaje “Ofrece una alternativa”.
- [ ] **T029** [US3] Crear pruebas de badges y bloqueo. Criterio: 100% de estados.

## Fase 5 — US-004: Descuento y disponibilidad (P1)

- [ ] **T030** [US4] Integrar decremento de raciones al enviar pedido (API de `001`). Criterio: una ración por unidad.
- [ ] **T031** [US4] Actualizar el menú tras enviar un pedido. Criterio: restantes y estado refrescados.
- [ ] **T032** [US4] Implementar evento `menu:availability-changed` en el store público. Criterio: actualización sin recargar.
- [ ] **T033** [US4] Probar concurrencia de última ración. Criterio: 0 sobreventa.
- [ ] **T034** [US4] Probar cancelación en Enviado/Listo. Criterio: repone/no repone.

## Fase 6 — US-005: Reposición (P2)

- [ ] **T035** [US5] Implementar `ReplenishRationsModal` con motivo obligatorio. Criterio: no confirma sin motivo.
- [ ] **T036** [US5] Integrar `POST /api/menu-items/:id/rations/replenish`. Criterio: incrementa y audita.
- [ ] **T037** [US5] Recalcular estado tras reponer. Criterio: `DISPONIBLE` o `ULTIMAS_RACIONES`.
- [ ] **T038** [US5] Impedir reactivar un plato con `manuallySoldOut`. Criterio: sigue `AGOTADO`.
- [ ] **T039** [US5] Crear pruebas de reposición. Criterio: motivo, incremento y auditoría.

## Fase 7 — US-006: Agotado y ajustes (P2)

- [ ] **T040** [US6] Implementar `SoldOutModal` para agotado por insumo. Criterio: motivo obligatorio.
- [ ] **T041** [US6] Integrar `POST /api/menu-items/:id/sold-out` y `/reactivate`. Criterio: bloqueo/reactivación.
- [ ] **T042** [US6] Implementar `AdjustRationsModal` para conteo físico. Criterio: valor ≥ 0 y motivo.
- [ ] **T043** [US6] Implementar `AdjustmentHistory`. Criterio: actor, motivo, antes/después.
- [ ] **T044** [US6] Crear pruebas de agotado y ajustes. Criterio: 100% auditado.

## Fase 8 — Pruebas y pulido

- [ ] **T045** [P] Revisar que no existan campos de costo, IGV o pago. Criterio: búsqueda sin resultados.
- [ ] **T046** [P] Revisar responsive en 360/768/1024 px. Criterio: sin cortes.
- [ ] **T047** Medir carga de menú con 50 platos. Criterio: p95 ≤ 500 ms.
- [ ] **T048** Crear E2E encargado publica → mozo ve → se agota → bloqueo. Criterio: pasa.
- [ ] **T049** Actualizar `quickstart.md` y estado de la spec. Criterio: documentación sincronizada.
- [ ] **T050** Ejecutar análisis de consistencia. Criterio: sin hallazgos críticos.

## Dependencias

```text
001 ──► Fase 0 → Fase 1 → Fase 2 → Fase 3 → Fase 4 → Fase 5 → Fase 6 → Fase 7 → Fase 8
                                                      │
                                                      └── habilita 003
```

## Definition of Done

- [ ] Menú del día publicado.
- [ ] Raciones y estados correctos.
- [ ] 0 sobreventa.
- [ ] Ajustes auditados.
- [ ] Cancelación aplica regla.
- [ ] Sin costos/IGV/pagos.
- [ ] Pruebas y consistencia.
