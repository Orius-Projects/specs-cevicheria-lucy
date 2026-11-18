---
feature: 005-avisos-tiempo-real
titulo: Decisiones técnicas de avisos en tiempo real
estado: borrador
creado: 2026-11-17
actualizado: 2026-11-17
fuentes:
  - docs/sources/DOCUMENTO_PROCESOS_CEVICHERIA_LUCY.pdf
---

# Research — Avisos en tiempo real

## 1. Socket.IO vs. WebSocket puro vs. SSE

| Opción | Ventajas | Desventajas | Decisión |
|---|---|---|---|
| **Socket.IO** | Salas, reconexión, acks, fallback | Protocolo propio | **Elegido** |
| WebSocket puro | Estándar, ligero | Salas/reconexión manuales | Descartado |
| SSE | Simple servidor→cliente | Unidireccional | Descartado |

**Motivo:** el stack lo incluye y resuelve salas/reconexión, justo lo que necesitan el KDS y el mozo.

## 2. Entrega at-least-once + deduplicación

**Decisión:** at-least-once con `eventId` y `version`; resync por REST.

**Motivo:**
- Exactly-once requiere coordinación distribuida.
- La fuente de verdad es MongoDB.
- El evento es una notificación, no el estado.

**Implementación:**
- `eventId` UUID por emisión.
- Cliente con `Set` de IDs.
- `version` evita eventos viejos.
- Resync al reconectar.

## 3. Sin replay

**Decisión:** no hay replay histórico; snapshot por REST.

**Motivo:**
- Volumen bajo de pedidos activos.
- Replay exigiría log persistente y retención.
- El snapshot es simple y consistente.

## 4. Sin push del sistema operativo

**Decisión:** notificaciones in-app + sonido; sin push externo.

**Motivo:**
- El PDF pide aviso visual y sonoro al mozo, no push.
- Web Push requiere service worker, permiso y servicio externo.
- El flujo crítico funciona en la demo sin dependencias externas.

**Futuro:** Q-005-01; no en MVP.

## 5. Todos los mozos reciben “Listo”

**Decisión:** `order:ready` se emite a `role:MOZO` (todos), cocina y encargado.

**Motivo:** no hay mozo asignado por mesa; cualquiera puede recoger y entregar. Se registra quién entrega.

**Implicación:** no se usa `user:<id>` para filtrar el aviso principal; se mantiene la sala de usuario para eventos futuros.

## 6. Eventos de disponibilidad

**Decisión:** `menu:availability-changed` con `menuItemId`, `state`, `remainingRations`, `changeType` y actor.

**Motivo:** P-05; todos los mozos deben ver el agotado/raciones sin recargar.

**Emisión:** después de descuento, reposición, ajuste, agotado y reactivación.

## 7. Salas y seguridad

- `role:MOZO`, `role:COCINERO`, `role:ENCARGADO`, `user:<id>`.
- El servidor asigna salas; no el cliente.
- JWT en handshake; usuario activo.
- CORS con el mismo `CORS_ORIGIN`.
- Sin datos sensibles en payload.

## 8. Sin alertas de tiempo

**Decisión:** no se emiten eventos de demora, cronómetro ni tiempo transcurrido.

**Motivo:** P-03, RN-10 y decisiones del PDF.

**Implicación:** `occurredAt` y `enviadoEn` se usan para orden/auditoría, no para mostrar demoras.

## 9. Escalado

**Decisión:** una instancia en la demo; Redis adapter en producción si se requiere.

**Motivo:** un restaurante tiene decenas de clientes.

## 10. Multi-pestaña

**Decisión:** cada pestaña tiene su conexión; el servidor emite a la sala; cada pestaña deduplica.

**Implicación:** no se comparte estado entre pestañas en el MVP.

## 11. Pruebas

- Handshake sin token.
- Salas por rol.
- `order:created` a cocina/mozos.
- `order:ready` a todos los mozos.
- `menu:availability-changed`.
- Deduplicación y orden.
- Resync tras 60 s.
- 20 clientes.

## 12. Riesgos

| Riesgo | Mitigación |
|---|---|
| Pérdida de eventos | Resync REST. |
| Duplicados | `eventId`. |
| Fuera de orden | `version`. |
| Fuga de roles | Salas y pruebas. |
| Audio bloqueado | Fallback visual. |
| Escalado | Redis adapter futuro. |
| Token expirado | Login. |

## 13. Preguntas abiertas

- Q-005-01: push futuro.
- Q-005-02: Redis adapter.
- Q-005-03: replay.
- Q-005-04: retención de logs.
- Q-005-05: sonidos distintos.
