---
feature: 006-cierre-turno
titulo: Decisiones técnicas del cierre de turno
estado: borrador
creado: 2026-11-17
actualizado: 2026-11-17
fuentes:
  - docs/sources/DOCUMENTO_PROCESOS_CEVICHERIA_LUCY.pdf
---

# Research — Cierre de turno

## 1. Alcance del PDF

P-06 pide:
- Verificar pedidos abiertos.
- Revisar y cerrar mesas abiertas.
- Consultar platos vendidos, raciones restantes, agotados y cancelaciones.
- Registrar observaciones.
- Cerrar el turno y guardar el resumen.
- Cerrar manualmente pedidos entregados sin marcar.
- Ajustar raciones con motivo.
- Registrar lo atendido si falla el sistema.

No incluye cobro ni facturación.

## 2. Snapshot vs. recálculo

**Decisión:** guardar `shiftSummaries` como snapshot inmutable al cerrar.

**Motivo:**
- El resumen debe ser reproducible y no cambiar si se editan pedidos o menú después.
- P-06 dice que el resumen queda guardado.

**Alternativa descartada:** recalcular el resumen cada vez desde pedidos; puede cambiar por ajustes posteriores.

## 3. Cierre con pendientes

**Decisión:** bloquear el cierre si hay pedidos `ENVIADO`/`LISTO` o mesas abiertas.

**Motivo:** P-06 pide verificar que no queden pendientes. El encargado debe entregar/cancelar/cerrar manualmente antes.

**Pendiente:** Q-006-04 (forzar cierre). Se propone no permitirlo en el MVP.

## 4. Cierre manual de pedidos

**Decisión:** el encargado puede cerrar un pedido `ENVIADO`/`LISTO` como `ENTREGADO` con `manualClose=true` y motivo.

**Motivo:** P-06 lo pide para pedidos entregados sin marcar.

**Regla:** es una excepción administrativa; no habilita el flujo normal `ENVIADO → ENTREGADO`.

## 5. Registro por falla

**Decisión:** `manualRecords` con `source: PAPER`.

- No envía a cocina.
- No descuenta raciones.
- Se incluye en el resumen.
- El encargado puede ajustar raciones por separado.

**Motivo:** RN-12; el sistema falló y se usó papel.

**Pendiente:** Q-006-02 (afectar raciones). Se propone solo resumen.

## 6. Ajuste de raciones al cierre

**Decisión:** reutilizar `rationAdjustments` de `002`.

- El encargado fija el conteo físico con motivo.
- El resumen usa el valor ajustado.
- Se audita antes/después.

## 7. Montos

**Decisión:** incluir cantidades y montos informativos.

**Motivo:** el PDF pide “revisar lo vendido”; los pedidos tienen precios. No es cierre de caja ni facturación.

**Etiqueta:** “Informativo; no incluye cobro ni impuestos”.

**Pendiente:** Q-006-01.

## 8. Cierre de mesas

**Decisión:** mozo o encargado pueden cerrar una mesa sin pedidos activos.

**Motivo:** el mozo libera la mesa al terminar; el encargado cierra leftovers en P-06.

**Alternativa descartada:** solo encargado; retrasa la operación.

## 9. Exportación

**Decisión:** no exportar a Excel/PDF en el MVP.

**Motivo:** no está en el PDF. Q-006-03.

## 10. Atomicidad

- `POST /api/shifts/:id/close` valida pendientes, genera snapshot y marca `CERRADO`.
- Si falla la escritura del resumen, el turno no se cierra.
- Si falla la emisión de `shift:closed`, el cierre persiste y se registra el fallo.

## 11. Riesgos

| Riesgo | Mitigación |
|---|---|
| Cierre con pendientes | `409` y listado. |
| Resumen desactualizado | Snapshot. |
| Registro manual duplicado | Advertencia. |
| Confusión con caja | Etiqueta informativa. |
| Ajuste negativo | Validación. |
| Falla de emisión | Log; el cierre persiste. |

## 12. Preguntas abiertas

- Q-006-01: montos.
- Q-006-02: registro por falla y raciones.
- Q-006-03: exportación.
- Q-006-04: forzar cierre.
- Q-006-05: quién cierra mesas.
- Q-006-06: identificación del turno.
