---
feature: 002-menu-raciones
titulo: Guía de inicio — Menú, raciones y agotados
estado: borrador
creado: 2026-11-17
actualizado: 2026-11-17
---

# Quickstart — Menú, raciones y agotados

## 1. Precondiciones

- `001` implementado: auth, mesas, turnos, menú y pedidos.
- Seed ejecutado: `npm run seed --workspace server`.
- Turno abierto por el encargado.

## 2. Credenciales

| Rol | Usuario | Contraseña |
|---|---|---|
| Encargado | `encargado1` | `encargado123` |
| Mozo | `mozo1` | `mozo123` |
| Cocinero | `cocinero1` | `cocina123` |

## 3. Flujo del encargado

1. Iniciar sesión como `encargado1`.
2. Abrir `/encargado/menu`.
3. Crear el menú del día (fecha actual).
4. Agregar platos por sección: Criollo, Marino, A la carta, Bebidas.
5. Configurar:
   - Plato sin límite.
   - Plato limitado a 30 raciones.
   - Variaciones: “Sin arroz”, “Arroz por papa”, “Papas extra”.
6. Publicar.
7. Verificar que el mozo ve el menú en `/mozo`.

## 4. Flujo del mozo

1. Iniciar sesión como `mozo1`.
2. Abrir el menú del día.
3. Verificar badges:
   - `Disponible`.
   - `Últimas raciones` (≤ 5).
   - `Agotado` (0 o insumo faltante).
4. Agregar un plato con variación y nota.
5. Enviar el pedido y verificar que las raciones bajan.
6. Si el plato llega a 0, verificar que se bloquea y aparece el toast.

## 5. Reposición y ajustes

### Reponer (mozo/encargado)

```bash
curl -s -X POST http://localhost:3000/api/menu-items/ID/rations/replenish \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"quantity":10,"reason":"Sobraron porciones; confirmado con cocina"}'
```

### Ajustar conteo (encargado)

```bash
curl -s -X POST http://localhost:3000/api/menu-items/ID/rations/adjust \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN_ENC" \
  -d '{"remainingRations":12,"reason":"Conteo físico: quedaban 12"}'
```

### Marcar agotado por insumo

```bash
curl -s -X POST http://localhost:3000/api/menu-items/ID/sold-out \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN_ENC" \
  -d '{"reason":"Falta de pescado"}'
```

### Reactivar

```bash
curl -s -X POST http://localhost:3000/api/menu-items/ID/reactivate \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN_ENC" \
  -d '{"reason":"Llegó insumo"}'
```

## 6. Verificación de estados

| Raciones restantes | Estado |
|---|---|
| Sin límite | `DISPONIBLE` |
| > 5 | `DISPONIBLE` |
| 1–5 | `ULTIMAS_RACIONES` |
| 0 | `AGOTADO` |
| Cualquiera + agotado por insumo | `AGOTADO` |
| Inactivo | `INACTIVO` |

## 7. Pruebas manuales

| Caso | Resultado esperado |
|---|---|
| Publicar menú vacío | `422`. |
| Publicar con ítems | Visible para el mozo. |
| Vender 2 de un plato limitado | Raciones bajan en 2. |
| Dos mozos piden la última ración | Uno gana; el otro `409`. |
| Cancelar en Enviado | Raciones vuelven. |
| Cancelar en Listo | Raciones no vuelven. |
| Reponer sin motivo | `422`. |
| Ajustar a negativo | `422`. |
| Agotado por insumo con raciones | Sigue bloqueado. |
| Reactivar agotado por insumo | Se recalcula por raciones. |

## 8. Pruebas automatizadas

```bash
npm test --workspace server
npm test --workspace client
```

Casos mínimos:
- `dishState` y umbral 5.
- Publicación de menú.
- Descuento y sobreventa.
- Reposición y ajuste auditados.
- Cancelación Enviado/Listo.
- Variaciones sin costo.

## 9. Solución de problemas

| Problema | Causa | Solución |
|---|---|---|
| El mozo no ve el menú | No publicado | Publicar menú del día. |
| `RATION_UNAVAILABLE` | Sin raciones | Reponer o elegir otro plato. |
| `DISH_UNAVAILABLE` | Agotado/inactivo | Reactivar o cambiar plato. |
| No se puede publicar | Sin ítems activos | Activar al menos uno. |
| Raciones no bajan | No se envió el pedido | Confirmar envío. |
| Ajuste no aparece | Rol sin permiso | Usar encargado. |

## 10. Notas

- No hay costos de variaciones.
- No hay IGV ni pagos.
- No hay inventario de almacén.
- Los mockups son referenciales.
