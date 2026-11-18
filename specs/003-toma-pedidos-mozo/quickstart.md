---
feature: 003-toma-pedidos-mozo
titulo: Guía de inicio — Toma de pedidos (Mozo)
estado: borrador
creado: 2026-11-17
actualizado: 2026-11-17
---

# Quickstart — Toma de pedidos (Mozo)

## 1. Precondiciones

- `001` y `002` implementados.
- Seed ejecutado.
- Turno abierto y menú publicado.
- Contrato de eventos de `005` disponible.

## 2. Flujo de demo

1. Iniciar sesión como `mozo1`.
2. Abrir “Nuevo pedido”.
3. Seleccionar mesa `M01`.
4. Agregar 2 chicharrones de pollo con “Arroz por papa” y “Sin ají”.
5. Agregar 1 ceviche mixto.
6. Revisar el total (suma de platos, sin IGV).
7. Enviar.
8. Verificar número de pedido y estado `ENVIADO`.
9. En otra pestaña, cocina marca “Listo”.
10. Verificar que el pedido aparece en “Listos”.
11. Marcar “Entregado”.
12. Cerrar la mesa.

## 3. Probar con dos mozos

1. Abrir una segunda sesión con `mozo2`.
2. Verificar que ambos ven los mismos pedidos activos y listos.
3. Cualquiera de los dos puede entregar.
4. Verificar que se registra quién entregó.

## 4. Probar cancelación

### Enviado

1. Crear un pedido y no marcarlo listo.
2. Cancelar con motivo.
3. Verificar que las raciones vuelven.
4. Verificar el ajuste `CANCELLATION_REVERSAL`.

### Listo

1. Crear un pedido y marcarlo listo desde cocina.
2. Cancelar con motivo.
3. Verificar la advertencia “Coordina con cocina”.
4. Verificar que las raciones no vuelven.

## 5. Pruebas manuales

| Caso | Resultado esperado |
|---|---|
| Sin turno | No se puede enviar. |
| Sin menú publicado | No se puede enviar. |
| Plato agotado | Botón deshabilitado. |
| Raciones insuficientes | Se limita o `409`. |
| Doble tap | Un solo pedido. |
| Variación inválida | `422`. |
| Entrega de `ENVIADO` | Acción no disponible. |
| Dos mozos entregan | Uno gana; el otro `409`. |
| Cancelar sin motivo | Rechazado. |
| Cerrar mesa con pedidos | `409`. |
| Recargar con borrador | Se conserva (P2). |

## 6. Curl de referencia

```bash
TOKEN=$(curl -s -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"mozo1","password":"mozo123"}' | jq -r .token)

# Menú
curl -s http://localhost:3000/api/menu/today -H "Authorization: Bearer $TOKEN"

# Pedido
curl -s -X POST http://localhost:3000/api/orders \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"tableId":"...","items":[{"dishId":"...","quantity":2,"variationIds":["v2"],"notes":"Sin ají"}],"clientRequestId":"uuid"}'

# Entregar
curl -s -X PATCH http://localhost:3000/api/orders/ID/deliver \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"version":1}'

# Cancelar
curl -s -X PATCH http://localhost:3000/api/orders/ID/cancel \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"reason":"Cliente se retiró","version":0}'

# Cerrar mesa
curl -s -X POST http://localhost:3000/api/table-sessions/ID/close \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"notes":"Mesa atendida"}'
```

## 7. Pruebas automatizadas

```bash
npm test --workspace client
npm run e2e --workspace client
```

Casos mínimos:
- Borrador y totales.
- Límite de raciones.
- Envío idempotente.
- Dos mozos.
- Entrega y cancelación.
- Cierre de mesa.

## 8. Solución de problemas

| Problema | Causa | Solución |
|---|---|---|
| `SHIFT_NOT_OPEN` | Turno cerrado | Abrir turno. |
| `MENU_NOT_PUBLISHED` | Menú no publicado | Publicar. |
| `DISH_UNAVAILABLE` | Plato agotado | Elegir otro. |
| `RATION_UNAVAILABLE` | Sin raciones | Ajustar cantidad. |
| `TABLE_HAS_ACTIVE_ORDERS` | Pedidos abiertos | Entregar/cancelar. |
| No aparece el listo | Socket caído | Resync/recargar. |
| No se puede entregar | Estado incorrecto | Verificar `LISTO`. |

## 9. Notas

- No hay mozo asignado por mesa.
- Todos los mozos ven todos los pedidos.
- No se muestran tiempos de cocina.
- No hay pagos, IGV, propinas ni división de cuenta.
- Los mockups son referenciales.
