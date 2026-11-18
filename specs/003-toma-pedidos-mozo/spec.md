---
feature: 003-toma-pedidos-mozo
titulo: Toma de pedidos — Mozo
estado: borrador
fase: MVP
prioridad: P1
procesos:
  - P-02 Tomar y enviar el pedido
  - P-04 Avisar “listo”, recoger y entregar
depende_de:
  - 001-plataforma-base
  - 002-menu-raciones
fuentes:
  - docs/sources/DOCUMENTO_PROCESOS_CEVICHERIA_LUCY.pdf
creado: 2026-11-17
actualizado: 2026-11-17
---

# Especificación — Toma de pedidos (Mozo)

## 1. Contexto y problema

En la situación actual, el mozo anota el pedido en papel, camina a la cocina y dicta la orden. El PDF propone que registre el pedido desde la web, con mesa, platos, cantidades, variaciones y notas, y que lo envíe a cocina sin desplazarse. También debe ver los pedidos listos, recogerlos y marcarlos como entregados.

Esta spec cubre **P-02** y la parte del mozo en **P-04**. No implementa la vista de cocina (`004`), los eventos (`005`) ni el cierre de turno (`006`).

## 2. Objetivos y no objetivos

### Objetivos

- Registrar un pedido con mesa, platos, cantidades, variaciones y notas.
- Mostrar disponibilidad y raciones restantes antes de enviar.
- Revisar el resumen y el total con el cliente.
- Enviar el pedido a cocina una sola vez.
- Ver pedidos activos y listos; todos los mozos pueden ver y entregar.
- Cancelar con motivo según las reglas del PDF.
- Cerrar la mesa cuando no queden pedidos activos.

### No objetivos

- No implementa la cola de cocina.
- No implementa notificaciones (spec `005`).
- No implementa menú/raciones (spec `002`).
- No implementa pagos, IGV, facturación ni costos.
- No implementa edición de un pedido ya enviado.
- No implementa división de cuenta, propinas ni delivery.
- No implementa app nativa/APK.
- No muestra tiempos de cocina ni alertas de demora.

## 3. Actores y roles

| Actor | Rol | Necesidad principal |
|---|---|---|
| Mozo | `MOZO` | Registrar, enviar, seguir y entregar pedidos. |
| Cocinero | `COCINERO` | Recibir el pedido en cocina y marcarlo listo. |
| Encargado | `ENCARGADO` | Supervisar; no opera el flujo diario. |
| Cliente | — | Dicta el pedido; no usa la web. |

## 4. Historias de usuario

### US-001 — Armar un pedido (P1)

**Como** mozo, **quiero** seleccionar una mesa y agregar platos con cantidades, variaciones y notas, **para** dejar el pedido listo antes de enviarlo.

**Por qué esta prioridad:** es el inicio de P-02.

**Prueba independiente:** seleccionar una mesa, agregar dos platos con variaciones y notas, y verificar que el resumen muestra todo.

**Escenarios de aceptación:**

- **DADO** un turno abierto y menú publicado **CUANDO** selecciono una mesa **ENTONCES** puedo agregar platos disponibles.
- **DADO** un plato `AGOTADO` **CUANDO** intento agregarlo **ENTONCES** el botón está deshabilitado.
- **DADO** un plato con 3 raciones **CUANDO** intento agregar 4 **ENTONCES** el sistema limita a 3 o rechaza.
- **DADO** un plato con variaciones **CUANDO** lo agrego **ENTONCES** solo puedo elegir las permitidas.
- **DADO** una nota de ítem **CUANDO** la escribo **ENTONCES** se guarda para cocina.

### US-002 — Revisar y enviar el pedido (P1)

**Como** mozo, **quiero** revisar el resumen con el cliente y enviarlo, **para** que cocina lo reciba completo y en orden.

**Por qué esta prioridad:** es el resultado de P-02.

**Prueba independiente:** enviar un pedido de tres ítems y verificar que aparece en cocina con estado `ENVIADO` y número único.

**Escenarios de aceptación:**

- **DADO** un borrador con mesa y al menos un ítem **CUANDO** presiono “Enviar” **ENTONCES** se crea el pedido y se descuentan raciones.
- **DADO** un borrador sin mesa o sin ítems **CUANDO** intento enviar **ENTONCES** el botón está deshabilitado.
- **DADO** un doble tap **CUANDO** se procesa **ENTONCES** se crea un solo pedido.
- **DADO** un pedido enviado **CUANDO** intento editarlo **ENTONCES** no está permitido.
- **DADO** un pedido enviado **CUANDO** se muestra el resumen **ENTONCES** el total es la suma de platos, sin IGV.

### US-003 — Ver pedidos activos y listos (P1)

**Como** mozo, **quiero** ver todos los pedidos activos y los listos, **para** recoger cualquiera que esté esperando.

**Por qué esta prioridad:** no hay mozo asignado por mesa; todos deben poder atender.

**Prueba independiente:** con dos mozos conectados, verificar que ambos ven la misma lista de listos y que cualquiera puede entregar.

**Escenarios de aceptación:**

- **DADO** un pedido `ENVIADO` **CUANDO** abro la lista **ENTONCES** aparece con mesa, ítems y quién lo tomó.
- **DADO** un pedido `LISTO` **CUANDO** abro la lista **ENTONCES** aparece como “Listo para recoger”.
- **DADO** un pedido `ENTREGADO` o `CANCELADO` **CUANDO** abro la lista activa **ENTONCES** no aparece.
- **DADO** dos mozos **CUANDO** uno marca entregado **ENTONCES** el otro ve el cambio.

### US-004 — Marcar entregado (P1)

**Como** mozo, **quiero** marcar un pedido listo como entregado, **para** cerrar el ciclo en la mesa.

**Por qué esta prioridad:** P-04 y RN-09.

**Prueba independiente:** marcar un pedido `LISTO` como entregado y verificar que sale de listos y queda en el historial.

**Escenarios de aceptación:**

- **DADO** un pedido `LISTO` **CUANDO** cualquier mozo presiona “Entregado” **ENTONCES** el estado cambia y se registra quién entregó.
- **DADO** un pedido `ENVIADO` **CUANDO** intento entregarlo **ENTONCES** la acción no está disponible.
- **DADO** un pedido ya entregado **CUANDO** intento entregarlo **ENTONCES** recibo `409`.

### US-005 — Cancelar un pedido (P2)

**Como** mozo, **quiero** cancelar un pedido con un motivo, **para** corregir errores o atender al cliente que se retira.

**Por qué esta prioridad:** RN-07; es una excepción.

**Prueba independiente:** cancelar un pedido `ENVIADO` con motivo y verificar que las raciones vuelven; cancelar uno `LISTO` y verificar que no vuelven.

**Escenarios de aceptación:**

- **DADO** un borrador **CUANDO** lo descarto **ENTONCES** no queda registro en cocina.
- **DADO** un pedido `ENVIADO` **CUANDO** cancelo con motivo **ENTONCES** se reponen raciones y se audita.
- **DADO** un pedido `LISTO` **CUANDO** cancelo con motivo **ENTONCES** se advierte coordinar con cocina y no se reponen raciones.
- **DADO** un pedido `ENTREGADO` **CUANDO** intento cancelar **ENTONCES** recibo `409`.
- **DADO** una cancelación sin motivo **CUANDO** intento confirmar **ENTONCES** se rechaza.

### US-006 — Cerrar la mesa (P1)

**Como** mozo, **quiero** cerrar una mesa cuando ya no tiene pedidos activos, **para** liberarla para nuevos clientes.

**Por qué esta prioridad:** P-06 y el estado de mesa.

**Prueba independiente:** entregar todos los pedidos de una mesa y cerrarla; verificar que vuelve a `LIBRE`.

**Escenarios de aceptación:**

- **DADO** una mesa sin pedidos activos **CUANDO** el mozo la cierra **ENTONCES** pasa a `LIBRE` y se registra quién la cerró.
- **DADO** una mesa con pedidos `ENVIADO` o `LISTO` **CUANDO** intento cerrarla **ENTONCES** recibo `409`.
- **DADO** una mesa cerrada **CUANDO** se agrega un nuevo pedido **ENTONCES** se abre una nueva sesión.

### US-007 — Conservar el borrador (P2)

**Como** mozo, **quiero** que mi borrador se conserve al recargar, **para** no perder el pedido en curso.

**Por qué esta prioridad:** mejora la resiliencia, no es indispensable para la demo.

**Prueba independiente:** agregar ítems, recargar y verificar que el borrador sigue.

**Escenarios de aceptación:**

- **DADO** un borrador con ítems **CUANDO** recargo **ENTONCES** se restaura.
- **DADO** un pedido enviado **CUANDO** recargo **ENTONCES** el borrador se limpia.

## 5. Requisitos funcionales (EARS)

| ID | Requisito | Fuente | Prioridad |
|---|---|---|---|
| FR-001 | CUANDO un usuario con rol `MOZO` abre la toma de pedidos, EL SISTEMA DEBE mostrar mesas y menú publicado. | P-02 | P1 |
| FR-002 | CUANDO el mozo selecciona una mesa, EL SISTEMA DEBE mostrar su estado y permitir la nueva ronda si ya está ocupada. | P-02 | P1 |
| FR-003 | CUANDO el mozo agrega un plato, EL SISTEMA DEBE permitir solo `DISPONIBLE` o `ULTIMAS_RACIONES` y limitar la cantidad a las raciones restantes. | P-02, P-05 | P1 |
| FR-004 | CUANDO el mozo elige variaciones, EL SISTEMA DEBE permitir solo las definidas para el plato y no alterar el precio. | P-02, RN-08 | P1 |
| FR-005 | CUANDO el mozo agrega notas, EL SISTEMA DEBE aceptar hasta 140 caracteres por ítem y 280 generales. | P-02 | P1 |
| FR-006 | CUANDO el mozo revisa el borrador, EL SISTEMA DEBE mostrar mesa, ítems, variaciones, notas y total en soles. | P-02 | P1 |
| FR-007 | CUANDO el mozo confirma un pedido válido, EL SISTEMA DEBE enviarlo una sola vez con `clientRequestId` y deshabilitar el botón mientras procesa. | P-02, RN-03 | P1 |
| FR-008 | SI no hay turno abierto, menú publicado, mesa activa o al menos un ítem, ENTONCES EL SISTEMA DEBE impedir el envío y explicar el motivo. | P-02, P-05 | P1 |
| FR-009 | CUANDO el pedido se crea, EL SISTEMA DEBE descontar raciones, asignar número y estado `ENVIADO`, y limpiar el borrador. | P-02, P-05 | P1 |
| FR-010 | CUANDO un pedido ya fue enviado, EL SISTEMA DEBE impedir su edición; el mozo debe cancelarlo y crear otro. | P-02 | P1 |
| FR-011 | CUANDO el mozo abre pedidos activos, EL SISTEMA DEBE mostrar todos los pedidos `ENVIADO`/`LISTO` de todos los mozos, con quién los tomó. | P-04 | P1 |
| FR-012 | CUANDO un pedido está `LISTO`, EL SISTEMA DEBE permitir a cualquier mozo marcarlo `ENTREGADO` y registrar quién lo entregó. | P-04, RN-09 | P1 |
| FR-013 | SI el pedido no está `LISTO`, ENTONCES EL SISTEMA DEBE impedir marcarlo entregado. | P-04, RN-09 | P1 |
| FR-014 | CUANDO el mozo cancela un pedido `ENVIADO`, EL SISTEMA DEBE exigir motivo, reponer raciones y auditar. | P-05, RN-07 | P1 |
| FR-015 | CUANDO el mozo cancela un pedido `LISTO`, EL SISTEMA DEBE mostrar advertencia, exigir motivo, no reponer raciones y auditar. | P-05, RN-07 | P1 |
| FR-016 | SI el pedido está `ENTREGADO` o `CANCELADO`, ENTONCES EL SISTEMA DEBE impedir la cancelación. | P-05 | P1 |
| FR-017 | CUANDO el mozo cierra una mesa sin pedidos activos, EL SISTEMA DEBE cerrarla y registrar `closedBy`. | P-06 | P1 |
| FR-018 | SI la mesa tiene pedidos `ENVIADO` o `LISTO`, ENTONCES EL SISTEMA DEBE impedir cerrarla. | P-06 | P1 |
| FR-019 | CUANDO el mozo consulta un pedido, EL SISTEMA DEBE mostrar estados, actores y horas de envío/entrega, sin tiempos transcurridos. | P-04, RN-10 | P1 |
| FR-020 | CUANDO una solicitud falla por red o timeout, EL SISTEMA DEBE conservar el borrador y permitir reintentar con el mismo `clientRequestId`. | P-02 | P1 |
| FR-021 | CUANDO el servidor confirma un pedido idempotente, EL SISTEMA DEBE tratarlo como éxito y no duplicarlo. | P-02 | P1 |
| FR-022 | CUANDO el mozo usa un celular o tablet, EL SISTEMA DEBE presentar diseño responsive con objetivos táctiles ≥ 44 px. | P-02 | P1 |
| FR-023 | CUANDO el mozo recarga la página, EL SISTEMA DEBE conservar el borrador hasta enviarlo o descartarlo. | P-02 | P2 |
| FR-024 | CUANDO se pierde la conexión, EL SISTEMA DEBE mostrar el estado y resincronizar pedidos al reconectar. | P-04 | P1 |
| FR-025 | EL SISTEMA NO DEBE mostrar pagos, IGV, propinas ni divisiones de cuenta. | p. 15 | P1 |
| FR-026 | EL SISTEMA NO DEBE mostrar cronómetros ni alertas de demora. | P-03, RN-10 | P1 |

## 6. Entidades y estados

- `Order`, `OrderItem`, `Table`, `TableSession`, `MenuItem` (ver [`001/data-model.md`](../001-plataforma-base/data-model.md)).
- Estados visibles: `BORRADOR` (UI), `ENVIADO`, `LISTO`, `ENTREGADO`, `CANCELADO`.
- No existe `EN_PREPARACION`.
- El mozo puede: crear borrador, enviar, ver activos/listos, entregar, cancelar y cerrar mesa.

## 7. Casos borde y errores

| ID | Caso | Comportamiento esperado |
|---|---|---|
| EC-001 | No hay turno abierto. | Bloqueo y mensaje. |
| EC-002 | No hay menú publicado. | Bloqueo y mensaje. |
| EC-003 | Plato agotado al agregar. | Botón deshabilitado y toast. |
| EC-004 | Plato agotado entre selección y envío. | `409 DISH_UNAVAILABLE`; refrescar. |
| EC-005 | Raciones insuficientes. | `409 RATION_UNAVAILABLE`; ajustar cantidad. |
| EC-006 | Doble tap en enviar. | Un solo pedido. |
| EC-007 | Timeout después de crear. | Reintento idempotente. |
| EC-008 | Mesa ocupada. | Permitir nueva ronda en la misma sesión. |
| EC-009 | Mesa por cerrar. | Abrir nueva sesión (Q-001-04). |
| EC-010 | Cancelación y “Listo” simultáneos. | Una transición gana; la otra `409`. |
| EC-011 | Token expirado. | `401`; conservar borrador y login. |
| EC-012 | Pedido ajeno. | Todos los mozos pueden verlo/entregarlo; no hay bloqueo por dueño. |
| EC-013 | Carrito vacío. | Botón deshabilitado. |
| EC-014 | Variación no permitida. | `422`. |
| EC-015 | Precio cambió después del borrador. | El servidor usa el precio vigente y lo snapshotea. |
| EC-016 | Pedido ya entregado. | `409`. |
| EC-017 | Múltiples pestañas. | Estado consistente por REST/Socket; no duplicar envíos. |

## 8. Requisitos no funcionales

| ID | Requisito | Métrica / criterio |
|---|---|---|
| NFR-001 | Registrar un pedido debe ser rápido. | 90% de pedidos de 3 ítems en ≤ 60 s. |
| NFR-002 | La API debe responder rápido. | p95 ≤ 500 ms en local. |
| NFR-003 | La UI debe ser móvil. | Desde 360 px; objetivos ≥ 44 px. |
| NFR-004 | La app debe ser instalable como PWA. | Opcional; manifest y service worker. |
| NFR-005 | No debe mostrar tiempos de cocina. | Sin cronómetros ni alertas. |
| NFR-006 | Debe degradar sin conexión. | Banner y reintento; sin offline completo. |
| NFR-007 | Debe ser accesible. | Labels, foco, contraste y `aria-live` para avisos. |
| NFR-008 | No debe exponer datos sensibles. | Token local; logout limpia estado. |

## 9. Criterios de éxito medibles

| ID | Criterio | Cómo se mide |
|---|---|---|
| SC-001 | El 90% de pedidos de 3 ítems se registra en ≤ 60 s. | Prueba con 5 mozos. |
| SC-002 | No se crean pedidos duplicados. | 100 envíos con doble tap; 0 duplicados. |
| SC-003 | Las raciones se descuentan exactamente una vez. | 100 pedidos; 0 diferencias. |
| SC-004 | Cancelar en Enviado repone; en Listo no. | 100% de casos. |
| SC-005 | Solo se entregan pedidos `LISTO`. | 100% de estados inválidos bloqueados. |
| SC-006 | Todos los mozos ven todos los listos. | Prueba con dos mozos. |
| SC-007 | No se muestran tiempos ni pagos. | Revisión de UI/API. |

## 10. Supuestos

- El turno lo abre el encargado y debe estar abierto para pedir.
- El mozo no tiene mesas asignadas; todos ven todos los pedidos.
- Cualquier mozo puede entregar y cancelar, registrando su identidad.
- El borrador vive en la UI; persistencia local es P2.
- El total es la suma de platos; sin IGV.
- No se edita un pedido enviado.
- No se registran comensales.
- No se divide la cuenta ni se manejan propinas.

## 11. Dependencias y riesgos

| Tipo | Descripción | Mitigación |
|---|---|---|
| Dependencia | `001` provee auth, mesas, turnos y pedidos. | Contratos congelados. |
| Dependencia | `002` provee menú, disponibilidad y raciones. | Validar antes de enviar. |
| Dependencia | Contrato de eventos de `005` para avisos y resync. | Se integra al final; la UI funciona con REST. |
| Riesgo | Doble envío. | `clientRequestId` y botón bloqueado. |
| Riesgo | Última ración concurrente. | `409` y refresco. |
| Riesgo | Cancelación y listo simultáneos. | `version`. |
| Riesgo | Mesa ocupada. | Nueva ronda en la misma sesión. |
| Riesgo | Autoplay del sonido. | Fallback visual (spec `005`). |

## 12. Fuera de alcance

- Cocina.
- Menú/raciones.
- Cierre de turno.
- Pagos, IGV, facturación.
- División de cuenta, propinas, delivery.
- Edición de pedidos enviados.
- App nativa/APK.
- Tiempos de cocina.

## 13. Preguntas abiertas

| ID | Pregunta | Impacto | Estado |
|---|---|---|---|
| Q-003-01 | ¿Se persiste el borrador entre recargas? | Bajo | `[NECESITA ACLARACIÓN]` (se propone P2). |
| Q-003-02 | ¿Un mozo puede cancelar cualquier pedido o solo el que tomó? | Medio | `[NECESITA ACLARACIÓN]` (se propone cualquiera). |
| Q-003-03 | ¿La mesa se cierra manualmente o al entregar todo? | Medio | `[NECESITA ACLARACIÓN]` (se propone manual). |
| Q-003-04 | ¿Se requiere registrar comensales? | Bajo | `[NECESITA ACLARACIÓN]` (se propone no). |
| Q-003-05 | ¿Se permite agregar una nueva ronda a una mesa ocupada? | Medio | `[NECESITA ACLARACIÓN]` (se propone sí). |
| Q-003-06 | ¿Se require imprimir la comanda? | Bajo | `[NECESITA ACLARACIÓN]` (fuera de alcance). |

## 14. Trazabilidad

| Fuente | Sección | Historias | Requisitos | Criterios |
|---|---|---|---|---|
| PDF p. 8 | P-02 Tomar y enviar el pedido | US-001, US-002 | FR-001–FR-010, FR-020–FR-023 | SC-001–SC-003 |
| PDF p. 9 | P-04 Avisar, recoger y entregar | US-003, US-004 | FR-011–FR-013, FR-019 | SC-005, SC-006 |
| PDF p. 10 | P-06 Cerrar el turno (mesas) | US-006 | FR-017, FR-018 | — |
| PDF p. 11 | RN-07, RN-09, RN-10 | US-004, US-005 | FR-012–FR-016, FR-019, FR-026 | SC-004, SC-005 |
| PDF p. 15 | Decisiones y alcance | — | FR-025, FR-026 | SC-007 |
