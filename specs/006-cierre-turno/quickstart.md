---
feature: 006-cierre-turno
titulo: Guía de inicio — Cierre de turno
estado: borrador
creado: 2026-11-17
actualizado: 2026-11-17
---

# Quickstart — Cierre de turno

## 1. Precondiciones

- `001`–`004` implementados.
- Turno abierto.
- Pedidos en distintos estados para probar el resumen.

## 2. Flujo de demo

1. Iniciar sesión como `encargado1`.
2. Abrir `/encargado/cierre`.
3. Ver pedidos `ENVIADO`/`LISTO` y mesas abiertas.
4. Cerrar una mesa sin pedidos activos.
5. Revisar el resumen:
   - Platos vendidos.
   - Raciones restantes.
   - Agotados.
   - Cancelaciones.
6. Registrar observaciones.
7. Cerrar el turno.
8. Verificar que no se pueden crear pedidos.
9. Consultar el resumen guardado.

## 3. Probar cierre con pendientes

1. Dejar un pedido `LISTO` sin entregar.
2. Intentar cerrar el turno.
3. Verificar `409` y la lista de pendientes.
4. Entregar o cancelar el pedido.
5. Cerrar el turno.

## 4. Probar cierre manual (P2)

1. Dejar un pedido `LISTO` entregado físicamente pero sin marcar.
2. En el cierre, usar “Cierre manual” con motivo.
3. Verificar que pasa a `ENTREGADO` con `manualClose=true`.
4. Verificar que el resumen lo incluye.

## 5. Probar registro por falla (P2)

1. En el cierre, abrir “Registrar lo atendido”.
2. Ingresar mesa, platos y montos.
3. Guardar.
4. Verificar que aparece en el resumen como `PAPER`.
5. Verificar que no se envió a cocina ni descontó raciones.

## 6. Probar ajuste de raciones (P2)

1. En el cierre, ajustar un plato a 12 raciones.
2. Ingresar motivo.
3. Verificar el cambio en el resumen y en el menú.
4. Verificar el historial de ajustes.

## 7. Pruebas manuales

| Caso | Resultado esperado |
|---|---|
| Turno sin pendientes | Cierre correcto. |
| Pedido abierto | `409`. |
| Mesa abierta | `409`. |
| Doble cierre | `409`. |
| Cierre manual sin motivo | `422`. |
| Registro manual vacío | `422`. |
| Ajuste negativo | `422`. |
| Turno cerrado | No se puede pedir. |
| Resumen guardado | No cambia después. |

## 8. Pruebas automatizadas

```bash
npm test --workspace server
npm test --workspace client
npm run e2e --workspace client
```

Casos mínimos:
- Resumen por plato.
- Raciones restantes.
- Agotados.
- Cancelaciones.
- Cierre con pendientes.
- Snapshot inmutable.
- Cierre manual.
- Registro `PAPER`.

## 9. Solución de problemas

| Problema | Causa | Solución |
|---|---|---|
| `OPEN_ORDERS_REMAIN` | Pedidos sin cerrar | Entregar/cancelar/cierre manual. |
| `OPEN_TABLES_REMAIN` | Mesas abiertas | Cerrar mesas. |
| `409` doble cierre | Turno ya cerrado | Consultar resumen. |
| Resumen no cuadra | Pedidos/ajustes posteriores | Revisar snapshot. |
| No se puede pedir | Turno cerrado | Abrir nuevo turno. |
| `401` | Token expirado | Login. |

## 10. Notas

- El resumen es informativo; no es cierre de caja.
- No hay pagos, IGV ni facturación.
- No hay exportación a Excel/PDF en el MVP.
- No se muestran tiempos de cocina.
- Los mockups son referenciales.
