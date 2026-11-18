---
feature: 004-cocina-pendientes
titulo: Guía de inicio — Cocina (Pendientes y listos)
estado: borrador
creado: 2026-11-17
actualizado: 2026-11-17
---

# Quickstart — Cocina (Pendientes y listos)

## 1. Precondiciones

- `001` y `003` implementados.
- Turno abierto y menú publicado.
- Contrato de eventos de `005` disponible.
- Pantalla de tablet/escritorio (se recomienda 1024×768 o superior).

## 2. Flujo de demo

1. Iniciar sesión como `cocinero1`.
2. Abrir `/cocina`.
3. Ver la columna **Pendientes** con los pedidos `ENVIADO` en orden FIFO.
4. En otra pestaña, crear un pedido como `mozo1`.
5. Verificar que aparece sin recargar.
6. Revisar mesa, ítems, variaciones y notas.
7. Presionar **“Marcar listo”**.
8. Verificar que se mueve a **Listos** y que el mozo recibe el aviso.
9. Como mozo, marcar “Entregado”.
10. Verificar que desaparece de **Listos**.

## 3. Pruebas manuales

| Caso | Resultado esperado |
|---|---|
| Tres pedidos en orden | FIFO correcto. |
| Pedido con variaciones | Se ven resaltadas. |
| Pedido con notas | Se ven completas. |
| Doble tap en “Listo” | Una transición. |
| Pedido cancelado | Desaparece con aviso. |
| Pedido entregado | Desaparece de listos. |
| 30 pedidos | Scroll fluido. |
| Socket caído | Banner y resync. |
| Servidor reiniciado | Cola reconstruida. |
| Token expirado | Login y resync. |

## 4. Curl de referencia

```bash
COCINA=$(curl -s -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"cocinero1","password":"cocina123"}' | jq -r .token)

# Cola
curl -s http://localhost:3000/api/kitchen/board \
  -H "Authorization: Bearer $COCINA"

# Marcar listo
curl -s -X PATCH http://localhost:3000/api/orders/ID/ready \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $COCINA" \
  -d '{"version":0}'
```

## 5. Pruebas automatizadas

```bash
npm test --workspace client
npm run e2e --workspace client
```

Casos mínimos:
- Inserción FIFO.
- Detalle con variaciones/notas.
- “Listo” y doble tap.
- Columnas pendientes/listos.
- Eventos created/delivered/cancelled.
- Resync.

## 6. Solución de problemas

| Problema | Causa | Solución |
|---|---|---|
| No aparece el pedido | Socket caído | Resync/recargar. |
| No se puede marcar listo | Estado incorrecto | Verificar `ENVIADO`. |
| `409` | Versión desactualizada | Refrescar cola. |
| No desaparece de listos | No llegó el evento | Resync. |
| Pantalla cortada | Resolución baja | Usar 1024×768+. |
| Token expirado | Sesión vencida | Login. |

## 7. Notas

- La cocina solo tiene “Marcar listo”.
- No marca “En preparación”.
- No modifica stock, raciones ni disponibilidad.
- No muestra tiempos ni alertas de demora.
- No hay pagos, IGV ni facturación.
- Los mockups son referenciales.
