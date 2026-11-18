---
feature: 001-plataforma-base
titulo: Decisiones técnicas de la plataforma base v2
estado: borrador
creado: 2026-11-17
actualizado: 2026-11-17
fuentes:
  - docs/sources/DOCUMENTO_PROCESOS_CEVICHERIA_LUCY.pdf
---

# Research — Plataforma base v2

> Registro de decisiones técnicas, alternativas y riesgos. Aplica a `002`, `003`, `004`, `005` y `006`.

## 1. Fuente y alcance

Este research parte únicamente del PDF `docs/sources/DOCUMENTO_PROCESOS_CEVICHERIA_LUCY.pdf` v1.0. Los mockups y documentos previos son externos/históricos y no se usan como fuente.

Decisiones del PDF que condicionan el modelo:
- Web responsive, no app nativa.
- Roles Mozo, Cocinero, Encargado.
- Sin mozo asignado por mesa.
- Cocina solo marca “Listo”; no hay “En preparación” ni stock.
- Raciones por plato, “Últimas raciones” con ≤ 5 y “Agotado” en 0.
- Sin tiempos ni alertas de demora.
- Sin pagos, facturación, IGV, costos, delivery ni inventario de almacén.

## 2. MongoDB como base de datos

**Decisión:** MongoDB 6+ con Mongoose 8.

**Motivo:** stack acordado; flexibilidad para menú, variaciones y snapshots.

**Implicaciones:**
- No hay joins; se usan referencias y snapshots.
- Los ítems del pedido se embeben.
- Las raciones viven en `menuItems` separados para descuento atómico por plato.
- Las transacciones multi-documento requieren replica set; se evitan en el flujo crítico.

**Alternativas descartadas:** PostgreSQL (mejor para transacciones, pero no es el stack acordado), SQLite (no encaja con el stack).

## 3. Roles y autenticación

**Decisión:** JWT con `JWT_SECRET`, expiración de 8 h; bcrypt coste ≥ 10; roles `ENCARGADO`, `MOZO`, `COCINERO`.

**Motivo:** simplicidad para demo web y trazabilidad por rol.

**Trade-offs:**
- Sin revocación inmediata; logout del lado del cliente.
- Sin CRUD de usuarios en la demo; seed de usuarios.
- El rol `ENCARGADO` es el perfil de configuración y supervisión.

## 4. Sin mozo asignado por mesa

**Decisión:** `tables` no tiene `waiterId`; `orders` registra `takenBy`, `readyBy`, `deliveredBy` y `cancelledBy`.

**Motivo:** el PDF dice “se usan mesas, sin mozo asignado por mesa” y exige trazabilidad de responsables.

**Implicaciones:**
- Todos los mozos ven los pedidos activos y listos.
- Cualquier mozo puede entregar.
- El aviso de “Listo” se envía a todos los mozos (spec `005`).
- La tabla de pedidos no se filtra por mozo.

## 5. Mesas y sesiones de mesa

**Decisión:** `tableSessions` agrupa las rondas de una mesa por turno.

**Motivo:**
- Una mesa puede recibir varios pedidos.
- P-06 pide cerrar mesas abiertas.
- El estado de la mesa se deriva: `LIBRE`, `OCUPADA`, `POR_CERRAR`.

**Alternativa descartada:** derivar el estado solo de pedidos activos sin sesión; no permite cerrar una mesa con historial ni agrupar rondas.

**Regla:** no se cierra una mesa con pedidos `ENVIADO` o `LISTO`.

## 6. Turnos

**Decisión:** `shifts` con `date`, `label`, `status`, `openedBy`, `closedBy`.

**Motivo:** P-06 cierra el turno y el resumen se guarda. El mockup menciona “Turno almuerzo”, pero es referencial; la etiqueta es configurable.

**Regla:** solo un turno abierto a la vez; sin turno abierto no se envían pedidos.

**Pendiente:** Q-001-01 (apertura manual vs automática).

## 7. Menú del día y disponibilidad

**Decisión:** `dishes` (maestro) + `menuDays` (publicación) + `menuItems` (disponibilidad/raciones).

**Motivo:**
- P-01: el encargado carga/actualiza el menú del día, activa/desactiva platos, define raciones y variaciones, y publica.
- P-05: las raciones y el estado del plato cambian durante el turno.
- Separar maestro y disponibilidad permite cambiar precios/raciones sin perder historial.

**Estructura:**
- `menuSections`: Criollo, Marino, A la carta, Bebidas.
- `dishes`: nombre, precio, sección, tipo, fijo/rotativo, variaciones.
- `menuDays`: fecha y estado `BORRADOR`/`PUBLICADO`.
- `menuItems`: activo, límite, total, restantes, umbral 5, agotado manual.

**Alternativa descartada:** productos/categorías genéricos sin menú del día; no cubre entradas fijas, fondos rotativos ni publicación.

## 8. Descuento atómico de raciones

**Decisión:** por cada ítem con límite, `findOneAndUpdate({ _id, active: true, manuallySoldOut: false, remainingRations: { $gte: quantity } }, { $inc: { remainingRations: -quantity } })`.

**Motivo:** evita sobreventa por plato sin transacciones multi-documento.

**Compensación:** si un ítem falla, se reponen los descuentos ya aplicados. Si la creación del pedido falla, se reponen todos.

**Trade-off:** no es una transacción ACID entre ítems; la compensación cubre el caso normal. Para producción con alto volumen se evaluaría replica set y transacciones.

## 9. Regla de cancelación y raciones

**Decisión confirmada con el negocio:**
- `ENVIADO → CANCELADO`: repone raciones automáticamente y registra `CANCELLATION_REVERSAL`.
- `LISTO → CANCELADO`: no repone; registra motivo.
- `BORRADOR → CANCELADO`: no hubo descuento; no repone.
- Si el plato está `manuallySoldOut`, la reposición no lo reactiva.

**Motivo:** el PDF dice que el mozo coordina con cocina si el pedido ya está listo; un plato preparado no debe volver a stock.

## 10. Borrador en la UI

**Decisión:** el `BORRADOR` vive en el cliente; no se persiste como pedido.

**Motivo:**
- El PDF lo describe como “el mozo está armando el pedido; aún no llega a cocina”.
- Evita números de pedido y pedidos huérfanos.
- Persistencia local opcional en `003` (P2).

## 11. Dinero sin IGV

**Decisión:** precios y total en céntimos; `totalCents = Σ lineTotalCents`. Sin IGV, impuestos ni pagos.

**Motivo:** el PDF excluye facturación, pagos y costos. El “total” del resumen es informativo.

## 12. Sin tiempos de cocina

**Decisión:** guardar `enviadoEn`, `listoEn` y `entregadoEn` para FIFO/auditoría, pero **no** mostrar tiempos transcurridos, cronómetros ni alertas de demora.

**Motivo:** el PDF lo repite en P-03, RN-10 y las decisiones acordadas.

**Implicación:** el KDS puede mostrar la hora actual; no un temporizador por pedido.

## 13. Seed y demo

**Decisión:** seed idempotente con `upsert` por clave natural.

**Datos:** usuarios por rol, mesas, secciones, platos, menú publicado, un plato limitado a 30, uno agotado por insumo, turno abierto y configuración.

## 14. Pruebas

- Unitarias: estados, totales, derivación de estado del plato, reglas de cancelación.
- Integración: auth, menú, pedidos, raciones, turnos.
- Concurrencia: última ración y doble envío.
- E2E: mozo → cocina → listo → entregado.
- Manual: demo con tres roles.

## 15. Riesgos abiertos

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Sin transacciones multi-documento | Inconsistencia parcial | Descuento secuencial + compensación; pruebas. |
| Cancelación y “Listo” simultáneos | Estado inconsistente | `version` + filtro por estado. |
| Menú no publicado | Bloqueo de pedidos | Validación y mensaje. |
| Raciones por día vs turno | Conteo incorrecto | Q-001-03; se propone por día. |
| Mesa por cerrar con nuevo pedido | Ambigüedad | Q-001-04; se propone nueva ronda. |
| Sin CRUD de usuarios | Dependencia de seed | Q-001-05; seed documentado. |
| Resumen con montos | Confusión con facturación | Etiquetar “informativo, sin fines contables”. |

## 16. Preguntas que deben resolverse antes de producción

- Q-001-01: apertura de turno manual vs automática.
- Q-001-02: comensales por mesa.
- Q-001-03: raciones por día o por turno.
- Q-001-04: reapertura de mesa por cerrar.
- Q-001-05: CRUD de usuarios/mesas.
- Q-001-06: número de pedido visible.
- Q-001-07: retención del historial.
