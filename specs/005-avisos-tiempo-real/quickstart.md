---
feature: 005-avisos-tiempo-real
titulo: Guía de inicio — Avisos en tiempo real
estado: borrador
creado: 2026-11-17
actualizado: 2026-11-17
---

# Quickstart — Avisos en tiempo real

## 1. Precondiciones

- `001`–`004` implementados.
- Seed ejecutado.
- Turno abierto y menú publicado.
- Dos pestañas o dos dispositivos para probar mozo y cocina.

## 2. Probar “Listo” para todos los mozos

1. Abrir una pestaña como `mozo1`.
2. Abrir otra pestaña como `mozo2`.
3. Abrir una tercera como `cocinero1` en `/cocina`.
4. Con `mozo1`, crear un pedido.
5. En cocina, presionar “Marcar listo”.
6. Verificar que **ambos mozos** reciben toast, badge y sonido (si está habilitado).
7. Verificar que cualquiera puede marcarlo entregado.

## 3. Probar cambios de disponibilidad

1. Abrir el menú como `mozo1`.
2. Como encargado, marcar un plato `AGOTADO` por insumo.
3. Verificar que el mozo ve el cambio sin recargar.
4. Reponer raciones y verificar el cambio de estado.
5. Verificar que el plato agotado no se puede agregar.

## 4. Probar reconexión

1. Abrir la cocina.
2. Detener el servidor o desconectar la red 30 s.
3. Crear un pedido con otro cliente.
4. Reconectar.
5. Verificar que la cola se resincroniza y muestra el pedido.
6. Verificar que no hay duplicados.

## 5. Probar deduplicación

- Enviar el mismo evento dos veces desde una prueba de integración.
- Verificar que solo aparece un toast y un badge.

## 6. Pruebas automatizadas

```bash
npm test --workspace server
npm test --workspace client
npm run e2e --workspace client
```

Casos mínimos:
- Handshake sin token.
- Salas por rol.
- `order:created` a cocina/mozos.
- `order:ready` a todos los mozos.
- `menu:availability-changed`.
- Dedupe y `version`.
- Resync 60 s.
- 20 clientes.

## 7. Prueba de contrato con Node

```js
import { io } from 'socket.io-client';

const socket = io('http://localhost:3000', {
  auth: { token: process.env.TOKEN }
});

socket.on('connect', () => console.log('conectado', socket.id));
socket.on('order:ready', (event) => console.log('listo', event));
socket.on('menu:availability-changed', (event) => console.log('menú', event));
socket.on('connect_error', (error) => console.error(error.message));
```

## 8. Solución de problemas

| Problema | Causa | Solución |
|---|---|---|
| No conecta | Token inválido | Revisar login y `VITE_SOCKET_URL`. |
| No llega el aviso | Socket caído | Resync/recargar. |
| Llega duplicado | `eventId` no deduplicado | Revisar `Set`. |
| Estado viejo | `version` no comparada | Revisar hook. |
| No suena | Autoplay | Tocar la pantalla; fallback visual. |
| Fuga de roles | Salas mal asignadas | Revisar handshake. |
| CORS | Origen distinto | Ajustar `CORS_ORIGIN`. |

## 9. Notas

- Todos los mozos reciben “Listo”.
- No hay push del sistema operativo.
- No hay replay; se usa resync REST.
- No hay eventos de tiempo ni alertas de demora.
- No hay pagos, IGV ni facturación.
