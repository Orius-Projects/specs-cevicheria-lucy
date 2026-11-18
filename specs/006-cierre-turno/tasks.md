# Tareas — Cierre de turno

> Ejecutar después de `001`–`004`. Las tareas `[P]` pueden ir en paralelo. No se implementan pagos, IGV, exportación ni tiempos.

## Fase 0 — Setup

- [ ] **T001** Crear ruta `/encargado/cierre` con guard `ENCARGADO`. Criterio: acceso correcto.
- [ ] **T002** [P] Crear `shiftClose.api.ts` con turno, pendientes, resumen, cierre y registros. Criterio: JWT y errores.
- [ ] **T003** [P] Crear `shiftCloseStore` en `client/src/stores/shiftCloseStore.ts`. Criterio: turno, pedidos, mesas y resumen.
- [ ] **T004** [P] Crear tipos de resumen en `shared/types/shift.ts`. Criterio: `ShiftSummary`, `ManualRecord`.
- [ ] **T005** Crear `ShiftClosePage` en `client/src/pages/encargado/ShiftClosePage.tsx`. Criterio: layout de cierre.

## Fase 1 — US-001: Pendientes (P1)

- [ ] **T006** [US1] Cargar turno actual y pedidos `ENVIADO`/`LISTO`. Criterio: lista visible.
- [ ] **T007** [US1] Cargar mesas con sesión abierta. Criterio: lista visible.
- [ ] **T008** [US1] Mostrar estado vacío “Todo cerrado”. Criterio: sin errores.
- [ ] **T009** [P] [US1] Crear `OpenOrdersList`. Criterio: mesa, número y estado.
- [ ] **T010** [P] [US1] Crear `OpenTablesList`. Criterio: mesa y apertura.
- [ ] **T011** [US1] Crear pruebas de pendientes. Criterio: pedidos y mesas.

## Fase 2 — US-002: Resumen (P1)

- [ ] **T012** [US2] Implementar `GET /api/shifts/:id/summary`. Criterio: snapshot o cálculo previo.
- [ ] **T013** [US2] Crear `SummaryCards` con totales informativos. Criterio: sin IGV.
- [ ] **T014** [US2] Crear `SoldDishesTable` agrupada por plato. Criterio: cantidades y montos.
- [ ] **T015** [US2] Crear `RemainingRationsTable`. Criterio: platos con límite.
- [ ] **T016** [US2] Crear `SoldOutList` con motivo. Criterio: agotados.
- [ ] **T017** [US2] Crear `CancellationsTable` con motivo y actor. Criterio: cancelaciones.
- [ ] **T018** [US2] Crear pruebas de resumen. Criterio: cuadre exacto.

## Fase 3 — US-003: Cerrar mesas (P1)

- [ ] **T019** [US3] Implementar cierre de mesa desde el cierre. Criterio: solo sin pedidos activos.
- [ ] **T020** [US3] Manejar `TABLE_HAS_ACTIVE_ORDERS`. Criterio: lista de pendientes.
- [ ] **T021** [US3] Actualizar estado de mesa. Criterio: `LIBRE`.
- [ ] **T022** [US3] Crear pruebas de cierre de mesa. Criterio: válido/inválido.

## Fase 4 — US-004: Cerrar turno (P1)

- [ ] **T023** [US4] Implementar `ObservationForm`. Criterio: texto opcional.
- [ ] **T024** [US4] Implementar `CloseShiftButton` con validación de pendientes. Criterio: `409` si hay.
- [ ] **T025** [US4] Implementar `POST /api/shifts/:id/close`. Criterio: snapshot y `CERRADO`.
- [ ] **T026** [US4] Guardar `shiftSummaries` inmutable. Criterio: no recalcular.
- [ ] **T027** [US4] Bloquear nuevos pedidos tras el cierre. Criterio: `SHIFT_NOT_OPEN`.
- [ ] **T028** [US4] Emitir `shift:closed` (spec `005`). Criterio: evento.
- [ ] **T029** [US4] Crear pruebas de cierre. Criterio: snapshot, bloqueo y doble cierre.

## Fase 5 — US-005: Cierre manual (P2)

- [ ] **T030** [US5] Implementar `ManualCloseModal` con motivo. Criterio: no confirma sin motivo.
- [ ] **T031** [US5] Integrar `POST /api/orders/:id/manual-close`. Criterio: `manualClose=true`.
- [ ] **T032** [US5] Auditar actor/motivo. Criterio: historial.
- [ ] **T033** [US5] Crear pruebas de cierre manual. Criterio: solo `ENVIADO`/`LISTO`.

## Fase 6 — US-006: Registro por falla (P2)

- [ ] **T034** [US6] Implementar `ManualRecordForm` con mesa, ítems y nota. Criterio: validación.
- [ ] **T035** [US6] Integrar `POST /api/shifts/:id/manual-records`. Criterio: `source: PAPER`.
- [ ] **T036** [US6] Incluir registros en el resumen. Criterio: visible.
- [ ] **T037** [US6] Crear pruebas de registro manual. Criterio: sin cocina ni stock.

## Fase 7 — US-007: Ajuste de raciones (P2)

- [ ] **T038** [US7] Implementar `AdjustRationsModal` con motivo. Criterio: entero ≥ 0.
- [ ] **T039** [US7] Integrar ajuste de `002`. Criterio: auditado.
- [ ] **T040** [US7] Reflejar el ajuste en el resumen. Criterio: raciones restantes.
- [ ] **T041** [US7] Crear pruebas de ajuste. Criterio: antes/después y motivo.

## Fase 8 — Pulido y pruebas

- [ ] **T042** [P] Verificar que no existan pagos/IGV/facturación. Criterio: búsqueda sin resultados.
- [ ] **T043** [P] Verificar que no existan tiempos/cronómetros. Criterio: búsqueda sin resultados.
- [ ] **T044** [P] Revisar responsive tablet/escritorio. Criterio: sin cortes.
- [ ] **T045** Medir cierre con 50 pedidos. Criterio: p95 ≤ 800 ms.
- [ ] **T046** Crear E2E operar → cerrar mesas → cerrar turno. Criterio: pasa.
- [ ] **T047** Actualizar `quickstart.md` y estado de la spec. Criterio: documentación sincronizada.
- [ ] **T048** Ejecutar análisis de consistencia. Criterio: sin hallazgos críticos.

## Dependencias

```text
001 ─┐
002 ─┼─► Fase 0 → Fase 1 → Fase 2 → Fase 3 → Fase 4 → Fase 5 → Fase 6 → Fase 7 → Fase 8
003 ─┤
004 ─┘
```

## Definition of Done

- [ ] Pendientes visibles.
- [ ] Resumen exacto.
- [ ] Mesas cerradas.
- [ ] Turno cerrado con snapshot.
- [ ] Cierre manual y registro por falla P2.
- [ ] Ajustes auditados.
- [ ] Sin pagos/IGV/tiempos.
