---
feature: 002-menu-raciones
titulo: Decisiones técnicas de menú, raciones y agotados
estado: borrador
creado: 2026-11-17
actualizado: 2026-11-17
fuentes:
  - docs/sources/DOCUMENTO_PROCESOS_CEVICHERIA_LUCY.pdf
---

# Research — Menú, raciones y agotados

## 1. Decisiones del PDF

- P-01: el encargado carga/actualiza el menú del día, mantiene entradas fijas, selecciona fondos rotativos, activa/desactiva platos, define límites y variaciones, revisa precios y publica.
- P-05: cada unidad confirmada descuenta una ración; con ≤ 5 se muestra “Últimas raciones”; a 0 se bloquea; el mozo repone tras confirmar con cocina; el encargado marca agotado por insumo o ajusta con motivo.
- RN-01 a RN-05, RN-08, RN-11.
- Sin costos, impuestos, inventario de almacén ni tiempos de cocina.

## 2. Modelo menú del día + disponibilidad

**Decisión:** separar `dishes` (maestro), `menuDays` (publicación) y `menuItems` (disponibilidad/raciones).

**Motivo:**
- El precio y las variaciones pertenecen al plato.
- Las raciones y el agotado cambian todos los días.
- El pedido conserva snapshots de nombre y precio.

**Alternativa descartada:** un solo `products` con `available` y `stock`. No permite historial de menús ni separar precio de disponibilidad.

## 3. Secciones y tipos de plato

**Decisión:** secciones configurables (`CRIOLLO`, `MARINO`, `A_LA_CARTA`, `BEBIDAS`) y `course` (`ENTRADA`, `FONDO`, `PLATO`, `BEBIDA`, `OTRO`).

- `isFixed`: entradas fijas del menú criollo.
- `isRotating`: fondos rotativos.
- El encargado selecciona qué platos se ofrecen cada día.

**Motivo:** el PDF distingue entradas fijas, fondos rotativos, carta marina y platos a la carta.

## 4. Variaciones

**Decisión:** lista plana de variaciones por plato, sin costo, con `id`, `name`, `sortOrder`, `active`.

**Motivo:** el PDF y el mockup muestran selección múltiple (“sin arroz”, “arroz por papa”, “papas extra”). No se especifican grupos excluyentes.

**Pendiente:** Q-002-06 (más de una variación del mismo grupo). Se propone permitir múltiples mientras el encargado no defina grupos.

## 5. Raciones y umbral

**Decisión:** `hasRationsLimit`, `totalRations`, `remainingRations`, `lowStockThreshold` default 5.

- Platos sin límite: `hasRationsLimit=false`, siempre `DISPONIBLE` si activos.
- Platos con límite: estado derivado.
- `manuallySoldOut` por insumo prevalece sobre las raciones.

**Motivo:** PDF p. 13 y RN-04.

**Pendiente:** Q-002-04 (umbral fijo o configurable). Se propone configurable, default 5.

## 6. Descuento de raciones

**Decisión:** al enviar el pedido, `findOneAndUpdate` con `remainingRations: { $gte: quantity }` y `$inc: -quantity`.

- Secuencial por ítem con compensación si uno falla.
- `manuallySoldOut` y `active` se validan en el filtro.
- No se descuenta al agregar al borrador.

**Motivo:** evitar sobreventa y no retener stock por borradores abandonados.

## 7. Reposición y ajustes

**Decisión:**
- `REPLENISH`: mozo o encargado; incrementa raciones; motivo obligatorio.
- `ADJUSTMENT`: encargado; fija el conteo físico; motivo obligatorio.
- `CANCELLATION_REVERSAL`: automático al cancelar en `ENVIADO`; registra pedido.
- No se repone al cancelar en `LISTO`.
- Una reposición no limpia `manuallySoldOut`.

**Motivo:** PDF P-05, RN-11 y decisión de negocio confirmada.

## 8. Publicación del menú

**Decisión:** `menuDays.status` `BORRADOR`/`PUBLICADO`; solo un publicado por fecha.

- El mozo solo ve el publicado.
- El encargado edita el borrador.
- Si se edita un publicado, se bloquea la edición; se permiten ajustes de disponibilidad/raciones (Q-002-03).

**Motivo:** P-01 “Guardar y publicar”; evitar cambios silenciosos durante el servicio.

## 9. Precios y totales

**Decisión:** `priceCents` por plato; `totalCents = Σ price × quantity`; variaciones sin costo; sin IGV.

**Motivo:** PDF excluye pagos, facturación y costos.

## 10. Raciones por día o por turno

**Decisión propuesta:** por día.

- `menuDays.date` define el día.
- Si el restaurante separa almuerzo/cena con raciones distintas, el encargado ajusta entre turnos.
- Q-002-01 debe confirmarse.

## 11. UI

- **Encargado:** editor de menú del día, maestro de platos, variaciones, raciones, publicar, marcar agotado, ajustar, historial.
- **Mozo:** tabs por sección, badges `Disponible`/`Últimas raciones`/`Agotado`, raciones restantes, botón “Agregar” deshabilitado si agotado, toast de agotado.
- **Cocinero:** consulta de menú/disponibilidad; no acciones de stock.

Los mockups son referenciales; no se copia el diseño.

## 12. Riesgos

| Riesgo | Mitigación |
|---|---|
| Sobreventa por concurrencia | `$gte` atómico y pruebas. |
| Raciones mal contadas | Ajuste con motivo y auditoría. |
| Menú publicado editado | Despublicar/versionar. |
| Reposición excesiva | Motivo y auditoría; Q-002-02. |
| Umbral distinto a 5 | Configurable. |
| Variaciones ambiguas | Lista plana; Q-002-06. |
| Raciones por turno | Q-002-01. |

## 13. Preguntas abiertas

- Q-002-01: raciones por día o turno.
- Q-002-02: reponer por encima del total.
- Q-002-03: editar publicado.
- Q-002-04: umbral fijo/configurable.
- Q-002-05: rotación automática de fondos.
- Q-002-06: variaciones excluyentes.
