---
feature: 006-cierre-turno
titulo: Cierre de turno
estado: borrador
fase: MVP
prioridad: P1
procesos:
  - P-06 Cerrar el turno y revisar lo vendido
depende_de:
  - 001-plataforma-base
  - 002-menu-raciones
  - 003-toma-pedidos-mozo
  - 004-cocina-pendientes
fuentes:
  - docs/sources/DOCUMENTO_PROCESOS_CEVICHERIA_LUCY.pdf
creado: 2026-11-17
actualizado: 2026-11-17
---

# Especificación — Cierre de turno

## 1. Contexto y problema

Al final del turno, el encargado necesita verificar que no queden pedidos ni mesas abiertas, revisar lo vendido, las raciones restantes, los agotados y las cancelaciones, registrar observaciones y cerrar el turno. También debe poder cerrar manualmente un pedido entregado sin marcar y registrar lo atendido si el sistema falló.

Esta spec cubre **P-06**. No implementa la operación diaria (`003`, `004`), el menú (`002`) ni los avisos (`005`). No incluye cobro ni facturación.

## 2. Objetivos y no objetivos

### Objetivos

- Verificar pedidos y mesas abiertas antes del cierre.
- Mostrar el resumen de platos vendidos, raciones restantes, agotados y cancelaciones.
- Registrar observaciones del turno.
- Cerrar el turno y guardar el resumen.
- Cerrar manualmente pedidos entregados sin marcar.
- Registrar lo atendido durante una falla del sistema.
- Ajustar raciones con motivo.
- Auditar todas las acciones de cierre.

### No objetivos

- No implementa cobro, pagos, IGV ni facturación.
- No implementa cierre de caja.
- No implementa contabilidad.
- No implementa exportación a Excel/PDF (salvo que se solicite).
- No implementa app nativa/APK.
- No muestra tiempos de cocina.

## 3. Actores y roles

| Actor | Rol | Necesidad principal |
|---|---|---|
| Encargado | `ENCARGADO` | Revisar, ajustar, cerrar turno y guardar resumen. |
| Mozo | `MOZO` | Cerrar mesas terminadas; consultar su estado. |
| Cocinero | `COCINERO` | Sin acciones de cierre. |

## 4. Historias de usuario

### US-001 — Ver pedidos y mesas abiertas (P1)

**Como** encargado, **quiero** ver los pedidos `ENVIADO`/`LISTO` y las mesas abiertas antes de cerrar, **para** no dejar operaciones incompletas.

**Por qué esta prioridad:** es el primer paso de P-06.

**Prueba independiente:** con un pedido listo y una mesa abierta, abrir el cierre y verificar que aparecen.

**Escenarios de aceptación:**

- **DADO** un turno con pedidos abiertos **CUANDO** abro el cierre **ENTONCES** veo la lista y sus estados.
- **DADO** una mesa con sesión abierta **CUANDO** abro el cierre **ENTONCES** aparece como pendiente.
- **DADO** un turno sin pendientes **CUANDO** abro el cierre **ENTONCES** veo “Todo cerrado”.

### US-002 — Consultar el resumen del turno (P1)

**Como** encargado, **quiero** ver platos vendidos, raciones restantes, agotados y cancelaciones, **para** decidir cuánto preparar mañana.

**Por qué esta prioridad:** es el resultado de P-06.

**Prueba independiente:** con pedidos y cancelaciones, verificar que el resumen coincide con los datos.

**Escenarios de aceptación:**

- **DADO** pedidos entregados **CUANDO** abro el resumen **ENTONCES** veo cantidades y montos informativos por plato.
- **DADO** un plato con límite **CUANDO** abro el resumen **ENTONCES** veo raciones restantes.
- **DADO** un plato agotado **CUANDO** abro el resumen **ENTONCES** veo el motivo.
- **DADO** cancelaciones **CUANDO** abro el resumen **ENTONCES** veo pedido y motivo.

### US-003 — Cerrar mesas abiertas (P1)

**Como** encargado, **quiero** cerrar las mesas que quedaron abiertas, **para** dejar el turno ordenado.

**Por qué esta prioridad:** P-06 lo pide explícitamente.

**Prueba independiente:** cerrar una mesa sin pedidos activos y verificar que queda libre; intentar cerrar una con pedidos y recibir error.

**Escenarios de aceptación:**

- **DADO** una mesa sin pedidos activos **CUANDO** la cierro **ENTONCES** queda `LIBRE`.
- **DADO** una mesa con pedidos `ENVIADO`/`LISTO` **CUANDO** intento cerrarla **ENTONCES** recibo `409`.
- **DADO** una mesa cerrada **CUANDO** se consulta **ENTONCES** no aparece abierta.

### US-004 — Cerrar el turno con observaciones (P1)

**Como** encargado, **quiero** registrar observaciones y cerrar el turno, **para** guardar el resumen y bloquear nuevas operaciones.

**Por qué esta prioridad:** cierra P-06.

**Prueba independiente:** cerrar un turno sin pendientes y verificar que el resumen queda guardado y no se pueden crear pedidos.

**Escenarios de aceptación:**

- **DADO** un turno sin pedidos/mesas abiertas **CUANDO** lo cierro con observaciones **ENTONCES** se guarda el resumen.
- **DADO** un turno cerrado **CUANDO** intento crear un pedido **ENTONCES** recibo `SHIFT_NOT_OPEN`.
- **DADO** un turno ya cerrado **CUANDO** intento cerrarlo **ENTONCES** recibo `409`.

### US-005 — Cerrar manualmente un pedido entregado sin marcar (P2)

**Como** encargado, **quiero** cerrar un pedido que fue entregado pero no marcado, **para** que el resumen no pierda la venta.

**Por qué esta prioridad:** excepción de P-06.

**Prueba independiente:** cerrar manualmente un pedido `LISTO` con motivo y verificar que queda `ENTREGADO` con `manualClose`.

**Escenarios de aceptación:**

- **DADO** un pedido `LISTO` **CUANDO** el encargado lo cierra con motivo **ENTONCES** pasa a `ENTREGADO` y se audita.
- **DADO** un pedido `ENVIADO` **CUANDO** el encargado lo cierra manualmente **ENTONCES** se registra la excepción con motivo.
- **DADO** un pedido `ENTREGADO`/`CANCELADO` **CUANDO** intento cerrarlo **ENTONCES** recibo `409`.
- **DADO** un cierre manual sin motivo **CUANDO** intento confirmar **ENTONCES** se rechaza.

### US-006 — Registrar lo atendido por falla (P2)

**Como** encargado, **quiero** registrar lo atendido en papel durante una falla, **para** incluirlo en el resumen.

**Por qué esta prioridad:** RN-12; excepción.

**Prueba independiente:** registrar un pedido manual y verificar que aparece en el resumen sin pasar por cocina.

**Escenarios de aceptación:**

- **DADO** una falla del sistema **CUANDO** registro mesa, platos y montos **ENTONCES** queda un registro `PAPER`.
- **DADO** un registro manual **CUANDO** abro el resumen **ENTONCES** se incluye.
- **DADO** un registro manual **CUANDO** se consulta **ENTONCES** muestra quién y cuándo lo registró.
- **DADO** un registro sin mesa o ítems **CUANDO** intento guardar **ENTONCES** se rechaza.

### US-007 — Ajustar raciones al cierre (P2)

**Como** encargado, **quiero** ajustar el conteo de raciones con motivo, **para** que el resumen refleje el conteo físico.

**Por qué esta prioridad:** excepción de P-06.

**Prueba independiente:** ajustar un plato a 12 raciones y verificar que el resumen y el menú lo reflejan.

**Escenarios de aceptación:**

- **DADO** una diferencia de conteo **CUANDO** ajusto a 12 **ENTONCES** se registra antes/después y motivo.
- **DADO** un ajuste negativo **CUANDO** intento **ENTONCES** se rechaza.
- **DADO** un ajuste **CUANDO** se consulta **ENTONCES** aparece en el historial.

## 5. Requisitos funcionales (EARS)

| ID | Requisito | Fuente | Prioridad |
|---|---|---|---|
| FR-001 | CUANDO el encargado abre el cierre, EL SISTEMA DEBE mostrar el turno actual y sus pedidos abiertos. | P-06 | P1 |
| FR-002 | CUANDO se consultan pedidos abiertos, EL SISTEMA DEBE incluir `ENVIADO` y `LISTO` con mesa y estado. | P-06 | P1 |
| FR-003 | CUANDO se consultan mesas, EL SISTEMA DEBE mostrar las que tienen sesión abierta. | P-06 | P1 |
| FR-004 | CUANDO el encargado cierra una mesa sin pedidos activos, EL SISTEMA DEBE cerrarla y registrar `closedBy`. | P-06 | P1 |
| FR-005 | SI una mesa tiene pedidos `ENVIADO`/`LISTO`, ENTONCES EL SISTEMA DEBE impedir cerrarla con `409`. | P-06 | P1 |
| FR-006 | CUANDO se genera el resumen, EL SISTEMA DEBE incluir platos vendidos, cantidades y montos informativos. | P-06 | P1 |
| FR-007 | CUANDO se genera el resumen, EL SISTEMA DEBE incluir raciones restantes de platos con límite. | P-06 | P1 |
| FR-008 | CUANDO se genera el resumen, EL SISTEMA DEBE incluir platos agotados y su motivo. | P-06 | P1 |
| FR-009 | CUANDO se genera el resumen, EL SISTEMA DEBE incluir cancelaciones y motivos. | P-06 | P1 |
| FR-010 | CUANDO el encargado registra observaciones, EL SISTEMA DEBE guardarlas en el resumen. | P-06 | P1 |
| FR-011 | CUANDO el encargado cierra el turno sin pendientes, EL SISTEMA DEBE guardar `shiftSummaries` y marcarlo `CERRADO`. | P-06 | P1 |
| FR-012 | SI hay pedidos o mesas abiertas, ENTONCES EL SISTEMA DEBE impedir el cierre con `409` y listar pendientes. | P-06 | P1 |
| FR-013 | CUANDO el turno está cerrado, EL SISTEMA DEBE bloquear nuevos pedidos con `SHIFT_NOT_OPEN`. | P-06 | P1 |
| FR-014 | CUANDO el encargado cierra manualmente un pedido, EL SISTEMA DEBE exigir motivo, marcar `manualClose` y auditar. | P-06 | P2 |
| FR-015 | CUANDO el encargado registra lo atendido por falla, EL SISTEMA DEBE guardar un registro `PAPER` con mesa, ítems y montos. | P-06, RN-12 | P2 |
| FR-016 | CUANDO se registra un pedido manual, EL SISTEMA DEBE incluirlo en el resumen sin enviarlo a cocina. | P-06, RN-12 | P2 |
| FR-017 | CUANDO el encargado ajusta raciones, EL SISTEMA DEBE exigir motivo y auditar antes/después. | P-06 | P2 |
| FR-018 | CUANDO se cierra el turno, EL SISTEMA DEBE guardar un snapshot inmutable del resumen. | P-06 | P1 |
| FR-019 | CUANDO se consulta un turno cerrado, EL SISTEMA DEBE devolver el resumen guardado, no recalcularlo. | P-06 | P1 |
| FR-020 | EL SISTEMA NO DEBE incluir cobro, pagos, IGV ni facturación. | p. 15 | P1 |
| FR-021 | CUANDO el encargado cierra el turno, EL SISTEMA DEBE emitir `shift:closed` (spec `005`). | P-06 | P2 |
| FR-022 | CUANDO se cierra un turno, EL SISTEMA DEBE registrar actor y fecha. | P-06 | P1 |
| FR-023 | CUANDO hay un error de red, EL SISTEMA DEBE conservar el estado y permitir reintentar. | P-06 | P1 |
| FR-024 | EL SISTEMA NO DEBE mostrar tiempos de cocina ni alertas de demora. | P-03, RN-10 | P1 |

## 6. Entidades y estados

- `Shift`, `TableSession`, `Order`, `MenuItem`, `ShiftSummary`, `ManualRecord` (ver [`001/data-model.md`](../001-plataforma-base/data-model.md)).
- Estados del turno: `ABIERTO`, `CERRADO`.
- El resumen se guarda en `shiftSummaries` como snapshot.

## 7. Casos borde y errores

| ID | Caso | Comportamiento esperado |
|---|---|---|
| EC-001 | Pedidos `ENVIADO`/`LISTO` al cerrar. | `409`; listar. |
| EC-002 | Mesas abiertas al cerrar. | `409`; listar. |
| EC-003 | Turno ya cerrado. | `409`. |
| EC-004 | Turno sin ventas. | Resumen vacío sin error. |
| EC-005 | Cierre manual sin motivo. | `422`. |
| EC-006 | Registro manual sin ítems. | `422`. |
| EC-007 | Ajuste negativo. | `422`. |
| EC-008 | Doble cierre. | `409`. |
| EC-009 | Falla durante el cierre. | No se marca cerrado; reintento. |
| EC-010 | Token expirado. | `401`. |
| EC-011 | Resumen con montos. | Etiquetado informativo, no contable. |
| EC-012 | Registro manual duplicado. | Advertir; permitir con nota. |

## 8. Requisitos no funcionales

| ID | Requisito | Métrica / criterio |
|---|---|---|
| NFR-001 | El resumen debe cuadrar. | 100% con pedidos entregados. |
| NFR-002 | Las acciones deben auditarse. | 100% con actor/fecha/motivo. |
| NFR-003 | El cierre debe ser atómico. | Turno y resumen consistentes. |
| NFR-004 | La UI debe ser responsive. | Tablet/escritorio. |
| NFR-005 | No debe haber pagos/IGV. | 0 campos. |
| NFR-006 | El resumen no debe recalcularse. | Snapshot inmutable. |
| NFR-007 | El cierre debe ser rápido. | p95 ≤ 800 ms. |
| NFR-008 | No debe haber tiempos. | 0 cronómetros. |

## 9. Criterios de éxito medibles

| ID | Criterio | Cómo se mide |
|---|---|---|
| SC-001 | 0 pedidos abiertos tras un cierre válido. | Consulta posterior. |
| SC-002 | 0 mesas abiertas tras el cierre. | Consulta posterior. |
| SC-003 | El resumen coincide con pedidos y ajustes. | Comparación; diferencia 0. |
| SC-004 | 100% de cierres manuales auditados. | Revisión. |
| SC-005 | 100% de turnos cerrados tienen snapshot. | Revisión. |
| SC-006 | No existen campos de pago/IGV. | Revisión. |
| SC-007 | No se muestran tiempos. | Revisión. |

## 10. Supuestos

- El turno lo abre el encargado.
- Solo el encargado cierra el turno.
- Mozo o encargado pueden cerrar mesas.
- El resumen incluye montos informativos, sin fines contables.
- El registro por falla no descuenta raciones; el encargado ajusta por separado.
- El cierre manual de pedidos es P2.
- No se exporta a Excel/PDF en el MVP.
- No hay pagos ni facturación.

## 11. Dependencias y riesgos

| Tipo | Descripción | Mitigación |
|---|---|---|
| Dependencia | `001` provee turnos, pedidos y resumen. | Contrato congelado. |
| Dependencia | `002` provee raciones y agotados. | Snapshot al cierre. |
| Dependencia | `003`/`004` proveen pedidos entregados. | Verificación previa. |
| Riesgo | Cierre con pendientes. | `409` y listado. |
| Riesgo | Resumen desactualizado. | Snapshot inmutable. |
| Riesgo | Registro manual duplicado. | Advertencia y auditoría. |
| Riesgo | Confusión con caja. | Etiqueta “informativo”. |

## 12. Fuera de alcance

- Cobro, pagos, IGV, facturación.
- Cierre de caja/contabilidad.
- Exportación a Excel/PDF.
- App nativa/APK.
- Tiempos de cocina.
- Delivery/inventario de almacén.

## 13. Preguntas abiertas

| ID | Pregunta | Impacto | Estado |
|---|---|---|---|
| Q-006-01 | ¿El resumen incluye montos además de cantidades? | Medio | `[NECESITA ACLARACIÓN]` (se propone sí, informativo). |
| Q-006-02 | ¿El registro por falla afecta raciones o solo el resumen? | Medio | `[NECESITA ACLARACIÓN]` (se propone solo resumen). |
| Q-006-03 | ¿Se requiere exportación a Excel/PDF? | Medio | `[NECESITA ACLARACIÓN]` (fuera del MVP). |
| Q-006-04 | ¿Se puede cerrar un turno con pedidos abiertos usando “forzar”? | Alto | `[NECESITA ACLARACIÓN]` (se propone no). |
| Q-006-05 | ¿Quién puede cerrar mesas? | Bajo | `[NECESITA ACLARACIÓN]` (mozo/encargado). |
| Q-006-06 | ¿El turno se identifica por fecha y etiqueta? | Bajo | `[NECESITA ACLARACIÓN]` (se propone sí). |

## 14. Trazabilidad

| Fuente | Sección | Historias | Requisitos | Criterios |
|---|---|---|---|---|
| PDF p. 10 | P-06 Cerrar el turno | US-001–US-004 | FR-001–FR-013, FR-018, FR-019, FR-022 | SC-001–SC-003, SC-005 |
| PDF p. 10 | Excepciones de P-06 | US-005, US-006, US-007 | FR-014–FR-017 | SC-004 |
| PDF p. 11 | RN-10, RN-12 | US-006 | FR-015, FR-016, FR-024 | SC-007 |
| PDF p. 15 | Decisiones y alcance | — | FR-020 | SC-006 |
