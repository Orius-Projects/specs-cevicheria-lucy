---
titulo: Constitución del Proyecto — Cevichería Lucy
version: 0.2.0
estado: borrador
ratificada: pendiente
creado: 2026-11-17
actualizado: 2026-11-17
fuente: docs/sources/DOCUMENTO_PROCESOS_CEVICHERIA_LUCY.pdf
---

# Constitución del Proyecto — Cevichería Lucy

## Preámbulo

Esta constitución define los principios no negociables del sistema de pedidos y cocina de la Cevichería Lucy. Todos los artefactos SDD (`spec.md`, `plan.md`, `tasks.md`, contratos y código) deben cumplirla.

La fuente de verdad es el documento `docs/sources/DOCUMENTO_PROCESOS_CEVICHERIA_LUCY.pdf` (15 páginas, versión 1.0, noviembre 2026). Los mockups y los documentos previos son material externo/histórico: los mockups son solo referenciales y no imponen el diseño ni el flujo, y los documentos previos no se usan como fuente.

**Alcance:** aplicación web responsive para móviles y tablets. No es una app nativa. No incluye pagos, facturación electrónica, SUNAT, costos de variaciones, delivery ni inventario de almacén.

---

## Principio 1 — Spec-first y trazabilidad total

**Regla:** ningún comportamiento se implementa sin una especificación aprobada. Cada requisito debe rastrearse a una sección del PDF: procesos `P-01` a `P-06`, reglas `RN-01` a `RN-12`, estados o decisiones acordadas.

**Implicaciones:**
- `spec.md` describe el **qué** y el **porqué**; no contiene stack ni decisiones de implementación.
- Los IDs (`US-###`, `FR-###`, `NFR-###`, `SC-###`, `EC-###`) son estables y no se renumeran.
- Toda ambigüedad se registra como `[NECESITA ACLARACIÓN]`; no se inventa una regla de negocio.
- Los cambios de proceso se hacen primero en la spec y luego se propagan a plan, tareas y código.
- Los mockups no son requisitos; si un mockup contradice el PDF, prevalece el PDF.

**Verificación:** checklist de requisitos aprobado y análisis de consistencia sin hallazgos críticos.

---

## Principio 2 — Un solo registro, tres vistas

**Regla:** el pedido es la fuente única de verdad para el mozo, la cocina y el encargado. Las tres vistas deben mostrar el mismo estado, los mismos ítems, las mismas variaciones y las mismas notas.

**Implicaciones:**
- El servidor es la autoridad del estado; el cliente solo lo refleja.
- No hay mozo asignado por mesa. Cada pedido registra quién lo tomó, quién lo preparó y quién lo entregó.
- Todos los mozos pueden ver los pedidos activos y listos, y cualquiera puede entregar.
- Las acciones de cada rol se limitan por permisos, no por asignación de mesa.
- Una corrección o cancelación debe mostrar el estado resultante y quién la ejecutó.

**Verificación:** pruebas de consistencia entre vistas y auditoría de responsables.

---

## Principio 3 — FIFO y una sola acción de cocina

**Regla:** la cocina atiende por orden de llegada. La vista de cocina tiene un único botón de acción: **“Marcar listo”**.

**Implicaciones:**
- La cola de pendientes se ordena por `enviadoEn` ascendente.
- No existe el estado “En preparación” ni un botón “Iniciar”.
- No hay prioridad manual, arrastrar y soltar, ni reordenamiento.
- No se muestran tiempos transcurridos, cronómetros ni alertas de demora.
- La cocina no modifica stock, disponibilidad ni raciones; si falta un insumo, avisa al encargado.
- Se puede mostrar la hora actual como referencia, pero no un temporizador por pedido.

**Verificación:** pruebas de orden FIFO y revisión de que la vista de cocina no tenga acciones distintas a “Marcar listo”.

---

## Principio 4 — Control de raciones y agotados confiable

**Regla:** cada unidad confirmada descuenta una ración de los platos con límite. El estado del plato cambia automáticamente y todo ajuste queda auditado.

**Implicaciones:**
- Estados del plato: `INACTIVO`, `DISPONIBLE`, `ULTIMAS_RACIONES`, `AGOTADO`.
- Con 5 o menos raciones se muestra “Últimas raciones”; al llegar a 0, “Agotado”.
- Un plato agotado no se puede agregar a un pedido nuevo.
- El descuento ocurre al **enviar** el pedido, no al agregarlo al borrador.
- El mozo repone raciones tras confirmar con cocina; el encargado marca agotado por falta de insumo o ajusta el conteo con motivo.
- El conteo físico prevalece; toda diferencia se registra en `rationAdjustments`.
- Cancelar un pedido en `ENVIADO` repone las raciones automáticamente; cancelar en `LISTO` no las repone.

**Verificación:** pruebas de concurrencia sin sobreventa, auditoría de ajustes y reglas de cancelación.

---

## Principio 5 — Trazabilidad de responsables y cierre de turno

**Regla:** cada pedido debe saber quién lo tomó, quién lo marcó listo, quién lo entregó y quién lo canceló. El turno se cierra con un resumen verificable.

**Implicaciones:**
- `orders` registra `takenBy`, `readyBy`, `deliveredBy` y `cancelledBy` con fecha y rol.
- El encargado abre y cierra el turno; el mozo o el encargado pueden cerrar una mesa.
- El cierre de turno verifica pedidos y mesas abiertas, muestra platos vendidos, raciones restantes, agotados y cancelaciones, y guarda observaciones.
- Un pedido entregado sin marcar puede cerrarse manualmente por el encargado con motivo y registro.
- Si falla el sistema, el encargado puede registrar lo atendido en papel al restablecerse (P2).

**Verificación:** resumen de turno que cuadra con los pedidos y auditoría de cierres manuales.

---

## Principio 6 — Web responsive y alcance acotado

**Regla:** el producto es una aplicación web responsive para móviles y tablets. No es una app nativa. El alcance se limita a los procesos de salón y cocina descritos en el PDF.

**Implicaciones:**
- No hay APK ni publicación en tiendas. La PWA es una mejora opcional, no un requisito.
- No se procesan pagos, facturación electrónica, SUNAT, IGV, propinas ni costos de variaciones.
- No hay delivery ni inventario de almacén; solo raciones de platos preparados.
- No se muestran demoras ni alertas de tiempo de cocina.
- La interfaz debe funcionar desde 360 px de ancho, con objetivos táctiles ≥ 44 px.
- El stack se mantiene: MongoDB, Node/Express/Socket.IO, React/Vite/Tailwind.

**Verificación:** pruebas en móvil/tablet y búsqueda de términos prohibidos en las specs.

---

## Restricciones técnicas ratificadas

| Área | Decisión |
|---|---|
| Base de datos | MongoDB + Mongoose |
| Backend | Node.js + Express + Socket.IO + CORS |
| Frontend | React + Vite + Tailwind CSS + React Router + Zustand + Axios |
| Tiempo real | Socket.IO Client / Server |
| PWA | `vite-plugin-pwa` (opcional) |
| Lenguaje | TypeScript en frontend y backend |
| Autenticación | JWT + bcrypt |
| Roles | `ENCARGADO`, `MOZO`, `COCINERO` |
| Dinero | Enteros en céntimos (`priceCents`, `totalCents`); sin IGV |
| Fechas | ISO 8601 UTC; visualización `America/Lima` |
| Moneda | PEN (S/) |
| Alcance | Un local, una cocina, una instancia |
| Prohibido | Pagos, facturación, SUNAT, IGV, costos de variaciones, delivery, inventario de almacén, app nativa/APK, demoras/tiempos de cocina |

---

## Flujo SDD y gates

```text
Constitución
    ↓
spec.md  ──►  clarify  ──►  plan.md  ──►  tasks.md  ──►  analyze  ──►  implement
    │                          │             │
    └── checklist de requisitos ┘             └── constitution check
```

| Gate | Criterio |
|---|---|
| G0 | Constitución vigente y leída |
| G1 | `spec.md` completo, con requisitos testeables y checklist |
| G2 | Preguntas bloqueantes resueltas o aceptadas como supuestos |
| G3 | `plan.md` con Constitution Check y arquitectura clara |
| G4 | `tasks.md` ordenado por dependencias y con rutas |
| G5 | Análisis de consistencia sin hallazgos críticos |
| G6 | Implementación con pruebas y demostración del flujo |
| G7 | Convergencia: la implementación satisface spec, plan y tareas |

**Regla de oro:** no se escribe `plan.md` sin `spec.md`; no se escriben `tasks.md` sin `plan.md`; no se implementa sin `tasks.md` y sin pasar el análisis de consistencia.

---

## Convenciones de documentación

- Contenido en español; nombres de archivo estándar SDD en inglés.
- Carpetas de feature en kebab-case con prefijo numérico: `001-plataforma-base`.
- Frontmatter YAML compatible con Obsidian: `feature`, `titulo`, `estado`, `fase`, `prioridad`, `procesos`, `depende_de`, `fuentes`.
- Cada `spec.md` cita el PDF y sus secciones; no cita documentos previos.
- Los mockups se mencionan solo como referencia visual, nunca como requisito.
- El PDF fuente se conserva en `docs/sources/`.

---

## Gobernanza

- **Versión:** SemVer. Esta versión es `0.2.0`, borrador pendiente de ratificación.
- **Enmiendas:** requieren justificación, impacto en specs existentes y registro en el historial.
- **Excepciones:** cualquier violación justificada se documenta en “Complexity Tracking” del `plan.md` y se aprueba explícitamente.
- **Revisión:** al cierre del MVP y de cada fase posterior.
- **Autoridad:** ante conflicto entre el PDF y esta constitución, se detiene la implementación y se resuelve con el responsable del proyecto.

## Historial de cambios

| Versión | Fecha | Cambio |
|---|---|---|
| 0.1.0 | 2026-11-17 | Versión inicial basada en documentos previos. |
| 0.2.0 | 2026-11-17 | Reescritura basada en el PDF de procesos v1.0: roles, estados, raciones, cierre de turno y alcance web responsive. |
