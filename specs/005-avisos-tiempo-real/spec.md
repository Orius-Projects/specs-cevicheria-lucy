---
feature: 005-avisos-tiempo-real
titulo: Avisos en tiempo real
estado: borrador
fase: MVP
prioridad: P1
procesos:
  - P-04 Avisar “listo”, recoger y entregar
  - P-05 Controlar raciones y marcar platos agotados
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

# Especificación — Avisos en tiempo real

## 1. Contexto y problema

El flujo necesita que el pedido llegue a cocina sin recargar y que todos los mozos reciban un aviso visual y sonoro cuando un pedido está listo. Como no hay mozo asignado por mesa, el aviso debe llegar a todos los mozos y cualquiera puede recogerlo. También deben propagarse los cambios de disponibilidad y raciones para que nadie ofrezca un plato agotado.

Esta spec cubre la mensajería de **P-04** y **P-05**. No define las pantallas (`003`, `004`) ni el modelo de datos (`001`).

## 2. Objetivos y no objetivos

### Objetivos

- Entregar eventos de pedido y disponibilidad en ≤ 2 s en la demo.
- Avisar a todos los mozos cuando un pedido está `LISTO`.
- Notificar a cocina cuando entra un pedido.
- Propagar cambios de disponibilidad/raciones y agotados.
- Recuperar el estado tras una desconexión.
- Evitar notificaciones duplicadas o fuera de orden.
- Aislar conexiones por rol.

### No objetivos

- No implementa las UIs.
- No implementa push del sistema operativo.
- No implementa Redis adapter ni múltiples instancias.
- No implementa replay histórico de eventos.
- No implementa chat.
- No implementa tiempos ni alertas de demora.
- No implementa pagos, IGV ni facturación.

## 3. Actores y roles

| Actor | Rol | Necesidad principal |
|---|---|---|
| Mozo | `MOZO` | Recibir avisos de pedidos listos y cambios de disponibilidad. |
| Cocinero | `COCINERO` | Recibir pedidos nuevos y ver listos. |
| Encargado | `ENCARGADO` | Supervisar eventos y disponibilidad. |
| Sistema | — | Emitir eventos después del commit y aislar salas. |

## 4. Historias de usuario

### US-001 — Aviso de pedido listo para todos los mozos (P1)

**Como** mozo, **quiero** recibir un aviso visual y sonoro cuando un pedido está listo, **para** recogerlo sin preguntar en cocina.

**Por qué esta prioridad:** es el mecanismo central de P-04.

**Prueba independiente:** con dos mozos conectados, cocina marca listo y ambos reciben el aviso.

**Escenarios de aceptación:**

- **DADO** un pedido `ENVIADO` **CUANDO** cocina lo marca `LISTO` **ENTONCES** todos los mozos reciben `order:ready` en ≤ 2 s.
- **DADO** un mozo en otra pantalla **CUANDO** llega el evento **ENTONCES** ve toast, badge y escucha el sonido si está habilitado.
- **DADO** un mozo que no tomó el pedido **CUANDO** llega el evento **ENTONCES** también lo recibe.
- **DADO** el audio bloqueado **CUANDO** llega el evento **ENTONCES** el aviso visual sigue.

### US-002 — Cocina recibe pedidos nuevos (P1)

**Como** cocinero, **quiero** ver los pedidos nuevos sin recargar, **para** empezar a prepararlos.

**Por qué esta prioridad:** P-03 y P-04.

**Prueba independiente:** crear un pedido y verificar que la cocina lo recibe en ≤ 2 s.

**Escenarios de aceptación:**

- **DADO** la cocina conectada **CUANDO** se crea un pedido **ENTONCES** recibe `order:created`.
- **DADO** un pedido creado durante una caída **CUANDO** la cocina reconecta **ENTONCES** lo obtiene por REST.

### US-003 — Cambios de disponibilidad en vivo (P1)

**Como** mozo, **quiero** enterarme cuando un plato se agota o cambian sus raciones, **para** no ofrecerlo por error.

**Por qué esta prioridad:** P-05.

**Prueba independiente:** agotar un plato y verificar que todos los mozos lo ven como `AGOTADO` sin recargar.

**Escenarios de aceptación:**

- **DADO** un plato con raciones **CUANDO** se agota **ENTONCES** todos los mozos reciben `menu:availability-changed`.
- **DADO** una reposición **CUANDO** se registra **ENTONCES** el plato vuelve a disponible en las vistas.
- **DADO** un ajuste del encargado **CUANDO** se registra **ENTONCES** se emite el cambio.

### US-004 — Recuperar estado tras desconexión (P1)

**Como** usuario, **quiero** que la app se resincronice al reconectar, **para** no operar con datos viejos.

**Por qué esta prioridad:** la red puede fallar.

**Prueba independiente:** desconectar 60 s, crear/entregar pedidos y verificar que al reconectar el estado coincide.

**Escenarios de aceptación:**

- **DADO** un cliente desconectado **CUANDO** reconecta **ENTONCES** obtiene el estado activo por REST.
- **DADO** un pedido creado durante la caída **CUANDO** reconecta **ENTONCES** aparece.
- **DADO** un pedido entregado durante la caída **CUANDO** reconecta **ENTONCES** no aparece en listos.

### US-005 — Evitar notificaciones duplicadas (P1)

**Como** usuario, **quiero** que un evento repetido no genere dos avisos, **para** no confundirme.

**Por qué esta prioridad:** la entrega es at-least-once.

**Prueba independiente:** enviar el mismo evento dos veces y verificar un solo toast.

**Escenarios de aceptación:**

- **DADO** un evento con `eventId` repetido **CUANDO** se recibe **ENTONCES** se ignora.
- **DADO** un evento con `version` menor **CUANDO** se recibe **ENTONCES** se ignora.
- **DADO** dos pestañas del mismo mozo **CUANDO** llega el evento **ENTONCES** cada pestaña lo muestra una vez.

### US-006 — Aislar conexiones por rol (P1)

**Como** responsable del sistema, **quiero** que cada cliente reciba solo sus eventos, **para** evitar fugas.

**Por qué esta prioridad:** seguridad y orden.

**Prueba independiente:** conectar mozo y cocina y verificar que cada uno recibe solo sus salas.

**Escenarios de aceptación:**

- **DADO** un handshake sin token **CUANDO** intenta conectar **ENTONCES** se rechaza.
- **DADO** un mozo **CUANDO** se conecta **ENTONCES** entra a `role:MOZO` y `user:<id>`.
- **DADO** una cocina **CUANDO** se conecta **ENTONCES** entra a `role:COCINERO`.
- **DADO** un evento de cocina **CUANDO** se emite **ENTONCES** no llega a un cliente sin rol.

## 5. Requisitos funcionales (EARS)

| ID | Requisito | Fuente | Prioridad |
|---|---|---|---|
| FR-001 | CUANDO un cliente abre Socket.IO, EL SISTEMA DEBE autenticarlo con JWT antes de aceptar eventos. | P-04 | P1 |
| FR-002 | CUANDO un usuario se autentica, EL SISTEMA DEBE unirlo a `role:<rol>` y `user:<id>`. | P-04 | P1 |
| FR-003 | CUANDO se crea un pedido, EL SISTEMA DEBE emitir `order:created` a cocina, mozos y encargado después del commit. | P-03, P-04 | P1 |
| FR-004 | CUANDO un pedido se marca listo, EL SISTEMA DEBE emitir `order:ready` a todos los mozos, cocina y encargado. | P-04 | P1 |
| FR-005 | CUANDO un pedido se entrega, EL SISTEMA DEBE emitir `order:delivered` a cocina, mozos y encargado. | P-04 | P1 |
| FR-006 | CUANDO un pedido se cancela, EL SISTEMA DEBE emitir `order:cancelled` a cocina, mozos y encargado. | P-04, RN-07 | P1 |
| FR-007 | CUANDO cambia la disponibilidad o las raciones, EL SISTEMA DEBE emitir `menu:availability-changed` a mozos, encargado y cocina (lectura). | P-05 | P1 |
| FR-008 | CADA evento DEBE incluir `eventId`, `type`, `occurredAt`, `version`, `schemaVersion` y `payload`. | P-04 | P1 |
| FR-009 | CUANDO un cliente recibe un evento, EL SISTEMA DEBE permitir deduplicarlo por `eventId`. | P-04 | P1 |
| FR-010 | SI un evento tiene `version` menor o igual a la conocida, ENTONCES EL SISTEMA DEBE ignorarlo. | P-04 | P1 |
| FR-011 | CUANDO un cliente reconecta, EL SISTEMA DEBE resincronizar el estado activo por REST. | P-04 | P1 |
| FR-012 | SI el handshake no tiene token válido o el usuario está inactivo, ENTONCES EL SISTEMA DEBE rechazar la conexión. | P-04 | P1 |
| FR-013 | CUANDO se emite un evento, EL SISTEMA DEBE registrarlo en logs con `eventId`, tipo y pedido. | P-04 | P2 |
| FR-014 | CUANDO el canal se desconecta o reconecta, EL SISTEMA DEBE exponer el estado a la UI. | P-04 | P2 |
| FR-015 | CUANDO un mozo recibe `order:ready`, EL SISTEMA DEBE mostrar toast, badge y sonido si está habilitado. | P-04 | P1 |
| FR-016 | CUANDO el audio está bloqueado, EL SISTEMA DEBE mantener el aviso visual. | P-04 | P1 |
| FR-017 | CUANDO se cierra el turno, EL SISTEMA DEBE emitir `shift:closed` para bloquear nuevas operaciones. | P-06 | P2 |
| FR-018 | CUANDO un pedido se cierra manualmente, EL SISTEMA DEBE emitir `order:manual-closed` (P2). | P-06 | P2 |
| FR-019 | EL SISTEMA NO DEBE emitir ni mostrar alertas basadas en tiempo o demora. | P-03, RN-10 | P1 |
| FR-020 | EL SISTEMA NO DEBE depender de push externo para el flujo crítico. | P-04 | P1 |
| FR-021 | CUANDO un cliente no recibe un evento, EL SISTEMA DEBE recuperar el estado por REST en la siguiente conexión. | P-04 | P1 |
| FR-022 | CUANDO se emite un evento, EL SISTEMA DEBE hacerlo después de persistir la transición. | P-04 | P1 |
| FR-023 | CUANDO un pedido cambia varias veces rápido, EL SISTEMA DEBE preservar el orden por `version` y `occurredAt`. | P-04 | P1 |
| FR-024 | EL SISTEMA NO DEBE emitir eventos de pago, IGV o facturación. | p. 15 | P1 |

## 6. Entidades y estados

- `Order`, `MenuItem`, `Shift` (ver [`001/data-model.md`](../001-plataforma-base/data-model.md)).
- Salas: `role:MOZO`, `role:COCINERO`, `role:ENCARGADO`, `user:<id>`.
- Eventos: `order:created`, `order:ready`, `order:delivered`, `order:cancelled`, `order:manual-closed`, `menu:availability-changed`, `shift:closed`.
- Contrato: [`contracts/events.md`](contracts/events.md).

## 7. Casos borde y errores

| ID | Caso | Comportamiento esperado |
|---|---|---|
| EC-001 | Evento duplicado. | Ignorar por `eventId`. |
| EC-002 | Evento fuera de orden. | Ignorar por `version`. |
| EC-003 | Reconexión masiva. | Backoff y resync controlado. |
| EC-004 | Servidor reiniciado. | Clientes reconectan y resync. |
| EC-005 | Token expirado. | Rechazo y login. |
| EC-006 | Varias pestañas. | Cada una recibe una vez. |
| EC-007 | Pestaña en segundo plano. | Al volver, resync. |
| EC-008 | Cliente lento. | No bloquea el servidor; resync. |
| EC-009 | 20 clientes. | Sin degradación. |
| EC-010 | Emisión antes del commit. | No debe ocurrir. |
| EC-011 | Falla de emisión. | Log; resync. |
| EC-012 | Audio bloqueado. | Aviso visual. |
| EC-013 | Turno cerrado. | Evento `shift:closed`; UI bloquea. |

## 8. Requisitos no funcionales

| ID | Requisito | Métrica / criterio |
|---|---|---|
| NFR-001 | Entrega oportuna. | p95 ≤ 2 s en demo. |
| NFR-002 | Sin pérdida en operación normal. | 0 en 1 000 eventos. |
| NFR-003 | Resync rápido. | ≤ 5 s tras reconectar. |
| NFR-004 | Concurrencia. | 20 clientes. |
| NFR-005 | Seguridad. | JWT, CORS y salas. |
| NFR-006 | No bloquear la API. | Emisión asíncrona. |
| NFR-007 | Orden por pedido. | `version` y `occurredAt`. |
| NFR-008 | Sin fugas. | Sin listeners duplicados. |
| NFR-009 | Diagnóstico. | Logs con `eventId`. |
| NFR-010 | Degradación sin socket. | REST disponible. |
| NFR-011 | Sin alertas de tiempo. | 0 eventos de demora. |

## 9. Criterios de éxito medibles

| ID | Criterio | Cómo se mide |
|---|---|---|
| SC-001 | 100% de pedidos nuevos llega a cocina en ≤ 2 s. | 100 eventos. |
| SC-002 | 100% de avisos `LISTO` llega a todos los mozos. | Prueba con dos mozos. |
| SC-003 | 0 notificaciones duplicadas. | 1 000 eventos. |
| SC-004 | Resync en ≤ 5 s tras 60 s desconectado. | Prueba. |
| SC-005 | 0 fugas entre roles. | Prueba de salas. |
| SC-006 | 20 clientes mantienen p95 ≤ 2 s. | Carga ligera. |
| SC-007 | 0 eventos de tiempo/pago. | Revisión del contrato. |

## 10. Supuestos

- Una sola instancia de Socket.IO.
- Los clientes están en red de baja latencia.
- No se usa Redis adapter.
- No se usa push del sistema operativo.
- La resincronización es por REST.
- Todos los mozos reciben `order:ready`.
- El audio requiere interacción; fallback visual.
- No se emiten alertas de tiempo.

## 11. Dependencias y riesgos

| Tipo | Descripción | Mitigación |
|---|---|---|
| Dependencia | `001` define pedidos/menú. | Contrato congelado. |
| Dependencia | `003` y `004` consumen eventos. | Contrato antes de UI. |
| Riesgo | Eventos perdidos. | Resync REST. |
| Riesgo | Duplicados. | `eventId`. |
| Riesgo | Fuera de orden. | `version`. |
| Riesgo | Fuga entre roles. | Salas y pruebas. |
| Riesgo | Audio bloqueado. | Fallback visual. |
| Riesgo | Escalado. | Redis adapter futuro. |

## 12. Fuera de alcance

- Push del sistema operativo.
- Redis adapter/múltiples instancias.
- Replay histórico.
- Chat/presencia.
- Tiempos/alertas.
- Pagos/IGV/facturación.

## 13. Preguntas abiertas

| ID | Pregunta | Impacto | Estado |
|---|---|---|---|
| Q-005-01 | ¿Se requiere push del sistema operativo en el futuro? | Medio | `[NECESITA ACLARACIÓN]` (fuera del MVP). |
| Q-005-02 | ¿Se usará Redis adapter en producción? | Medio | `[NECESITA ACLARACIÓN]`. |
| Q-005-03 | ¿Se requiere replay de eventos? | Medio | `[NECESITA ACLARACIÓN]` (se propone REST). |
| Q-005-04 | ¿Cuánto se conservan los logs de eventos? | Bajo | `[NECESITA ACLARACIÓN]`. |
| Q-005-05 | ¿Se requiere sonido distinto por tipo de aviso? | Bajo | `[NECESITA ACLARACIÓN]`. |

## 14. Trazabilidad

| Fuente | Sección | Historias | Requisitos | Criterios |
|---|---|---|---|---|
| PDF p. 9 | P-04 Avisar, recoger y entregar | US-001, US-004 | FR-003–FR-006, FR-011, FR-015, FR-016 | SC-001–SC-004 |
| PDF p. 10 | P-05 Control de raciones | US-003 | FR-007 | SC-001 |
| PDF p. 11 | RN-10 (sin tiempos) | — | FR-019 | SC-007 |
| PDF p. 15 | Decisiones y alcance | US-006 | FR-012, FR-020, FR-024 | SC-005, SC-007 |
