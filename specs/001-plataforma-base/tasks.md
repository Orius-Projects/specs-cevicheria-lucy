# Tareas — Plataforma base v2

> Ejecutar antes de `002`–`007`. Las tareas `[P]` pueden ir en paralelo dentro de su fase. No se implementan pagos, IGV, SUNAT ni tiempos de cocina.

## Fase 0 — Setup

- [ ] **T001** Crear monorepo con npm workspaces y carpetas `client/`, `server/`, `shared/`. Criterio: `npm install` resuelve los paquetes.
- [ ] **T002** [P] Configurar TypeScript, ESLint y Prettier. Criterio: `npm run typecheck` y `npm run lint` sin errores.
- [ ] **T003** [P] Crear `.env.example` y validación Zod de variables. Criterio: el servidor falla claro si falta `MONGODB_URI` o `JWT_SECRET`.
- [ ] **T004** Configurar dependencias del backend en `server/package.json`. Criterio: express, mongoose, socket.io, cors, jsonwebtoken, bcryptjs, zod, helmet, morgan.
- [ ] **T005** [P] Configurar dependencias del frontend en `client/package.json`. Criterio: react, react-router-dom, zustand, axios, socket.io-client, tailwindcss, vite, vite-plugin-pwa.
- [ ] **T006** [P] Crear tipos y constantes compartidas en `shared/types/` y `shared/constants/`. Criterio: roles, estados de pedido y plato.
- [ ] **T007** Configurar Vite + Tailwind + PWA opcional. Criterio: `npm run dev --workspace client` levanta la app.
- [ ] **T008** Configurar scripts raíz `dev`, `build`, `test`, `lint`, `typecheck`, `seed`. Criterio: documentados y funcionales.
- [ ] **T009** [P] Crear estructura `client/src`, `server/src` y `shared/`. Criterio: coincide con `plan.md`.
- [ ] **T010** [P] Documentar setup en `quickstart.md`. Criterio: un tercero puede levantar el proyecto.

## Fase 1 — Base compartida (bloqueante)

- [ ] **T011** Implementar conexión MongoDB en `server/src/config/db.ts`. Criterio: conecta, loguea y cierra; error claro.
- [ ] **T012** Implementar logger y `requestId` en `server/src/config/logger.ts`. Criterio: cada request tiene ID.
- [ ] **T013** Implementar middleware de errores en `server/src/middlewares/error.ts`. Criterio: formato uniforme.
- [ ] **T014** Implementar validación Zod en `server/src/middlewares/validate.ts`. Criterio: `400/422` con detalle.
- [ ] **T015** Implementar auth JWT en `server/src/middlewares/auth.ts`. Criterio: token válido adjunta usuario; inválido `401`.
- [ ] **T016** Implementar roles en `server/src/middlewares/roles.ts`. Criterio: `ENCARGADO`, `MOZO`, `COCINERO`; sin permiso `403`.
- [ ] **T017** Crear modelo `User` en `server/src/models/user.model.ts`. Criterio: `passwordHash` nunca se serializa.
- [ ] **T018** [P] Crear modelos `Table`, `TableSession`, `Shift`. Criterio: campos e índices de `data-model.md`.
- [ ] **T019** [P] Crear modelos `MenuSection`, `Dish`, `MenuDay`, `MenuItem`, `RationAdjustment`. Criterio: variaciones, estados y ajustes.
- [ ] **T020** [P] Crear modelos `Order`, `Counter`, `Settings`. Criterio: ítems embebidos, `version`, sin campos de pago/IGV.

## Fase 2 — US-001: Autenticación por rol (P1)

- [ ] **T021** [US1] Implementar `POST /api/auth/login`. Criterio: token y perfil con rol.
- [ ] **T022** [US1] Manejar credenciales inválidas e inactivas. Criterio: `401` genérico.
- [ ] **T023** [US1] Implementar firma/verificación JWT en `server/src/utils/jwt.ts`. Criterio: expiración 8 h.
- [ ] **T024** [US1] Implementar `GET /api/auth/me`. Criterio: sin `passwordHash`.
- [ ] **T025** [P] [US1] Implementar bcrypt en `server/src/utils/password.ts`. Criterio: coste ≥ 10.
- [ ] **T026** [US1] Implementar `POST /api/auth/logout`. Criterio: `204`.
- [ ] **T027** [US1] Crear pruebas de auth en `server/tests/auth.test.ts`. Criterio: válido, inválido, inactivo, expirado.
- [ ] **T028** [US1] Crear login y guards por rol en `client/src/pages/LoginPage.tsx` y `client/src/routes/`. Criterio: cada rol aterriza en su vista.

## Fase 3 — US-002: Mesas y sesiones (P1)

- [ ] **T029** [US2] Implementar `GET /api/tables` con estado derivado. Criterio: `LIBRE`, `OCUPADA`, `POR_CERRAR`.
- [ ] **T030** [US2] Implementar apertura de `tableSession` al primer pedido. Criterio: `openedBy`, `openedAt`, `shiftId`.
- [ ] **T031** [US2] Implementar `POST /api/table-sessions/:id/close`. Criterio: solo sin pedidos activos; `409` si hay.
- [ ] **T032** [US2] Implementar consulta de mesa con pedidos activos. Criterio: el mozo ve rondas y estado.
- [ ] **T033** [P] [US2] Crear store de mesas en `client/src/stores/tablesStore.ts`. Criterio: refresco y estado.
- [ ] **T034** [US2] Crear pruebas de mesas y sesiones. Criterio: estados y cierre.
- [ ] **T035** [US2] Crear selector visual de mesa en `client/src/components/waiter/TableSelector.tsx`. Criterio: mesas libres/ocupadas.

## Fase 4 — US-003: Turnos (P1)

- [ ] **T036** [US3] Implementar `POST /api/shifts` para abrir turno. Criterio: solo `ENCARGADO`; uno abierto.
- [ ] **T037** [US3] Implementar `GET /api/shifts/current` y bloqueo de pedidos sin turno. Criterio: `SHIFT_NOT_OPEN`.
- [ ] **T038** [US3] Implementar cierre de turno básico en `POST /api/shifts/:id/close`. Criterio: `409` si hay pedidos/mesas abiertas.
- [ ] **T039** [US3] Implementar resumen inicial en `shiftSummaries`. Criterio: snapshot guardado (detalle en `006`).
- [ ] **T040** [P] [US3] Crear store de turno en `client/src/stores/shiftStore.ts`. Criterio: turno actual y estado.
- [ ] **T041** [US3] Crear pruebas de turnos. Criterio: apertura única, bloqueo y cierre.

## Fase 5 — Menú y raciones (dominio de 002)

- [ ] **T042** Implementar `GET /api/menu/today` con menú publicado. Criterio: secciones, platos, estados.
- [ ] **T043** Implementar `POST /api/menu/days` y `PUT /api/menu/days/:id`. Criterio: borrador editable.
- [ ] **T044** Implementar `POST /api/menu/days/:id/publish`. Criterio: solo con ítems activos.
- [ ] **T045** Implementar `GET /api/menu-sections` y `GET /api/dishes`. Criterio: secciones y platos.
- [ ] **T046** Implementar CRUD de platos en `POST/PATCH/DELETE /api/dishes`. Criterio: variaciones sin costo; baja lógica.
- [ ] **T047** Implementar derivación de estado del plato. Criterio: `INACTIVO`, `DISPONIBLE`, `ULTIMAS_RACIONES`, `AGOTADO`.
- [ ] **T048** Implementar `POST /api/menu-items/:id/rations/replenish`. Criterio: mozo/encargado; motivo obligatorio.
- [ ] **T049** Implementar `POST /api/menu-items/:id/rations/adjust`. Criterio: encargado; motivo y valores.
- [ ] **T050** Implementar `POST /api/menu-items/:id/sold-out` y `/reactivate`. Criterio: agotado manual y reactivación.
- [ ] **T051** Implementar historial de ajustes en `rationAdjustments`. Criterio: actor, motivo, antes/después.
- [ ] **T052** Crear pruebas de menú, estados y ajustes. Criterio: 0 sobreventa, 100% auditado.

## Fase 6 — Pedidos (dominio de 003/004)

- [ ] **T053** Implementar `POST /api/orders` con validaciones. Criterio: turno, mesa, menú, ítems, disponibilidad.
- [ ] **T054** Implementar descuento secuencial de raciones con compensación. Criterio: 0 sobreventa; restaura si falla.
- [ ] **T055** Implementar numeración atómica con `Counter`. Criterio: 1 000 pedidos sin duplicados.
- [ ] **T056** Implementar snapshots de plato, precio y variaciones. Criterio: historial inmutable.
- [ ] **T057** Implementar cancelación `ENVIADO` con reposición de raciones. Criterio: `CANCELLATION_REVERSAL`.
- [ ] **T058** Implementar cancelación `LISTO` sin reposición. Criterio: motivo y auditoría.
- [ ] **T059** Implementar `PATCH /api/orders/:id/ready`. Criterio: solo `COCINERO`, `ENVIADO → LISTO`.
- [ ] **T060** Implementar `PATCH /api/orders/:id/deliver`. Criterio: cualquier `MOZO`, `LISTO → ENTREGADO`.
- [ ] **T061** Implementar `GET /api/kitchen/board` con pendientes FIFO y listos. Criterio: sin tiempos ni “En preparación”.
- [ ] **T062** Implementar rechazo de `ENVIADO → ENTREGADO`. Criterio: `409`.
- [ ] **T063** Implementar `GET /api/orders` y `GET /api/orders/:id`. Criterio: todos los mozos ven todos; filtros.
- [ ] **T064** Implementar cierre manual de pedido P2. Criterio: solo `ENCARGADO`, con motivo.
- [ ] **T065** Crear pruebas de pedidos y concurrencia. Criterio: transiciones, raciones, cancelación, doble envío.

## Fase 7 — US-004: Seed, salud y documentación (P1)

- [ ] **T066** [US4] Crear datos demo en `server/src/seed/data.ts`. Criterio: usuarios, mesas, secciones, platos, menú, turno.
- [ ] **T067** [US4] Implementar seed idempotente. Criterio: dos ejecuciones sin duplicados.
- [ ] **T068** [US4] Implementar `GET /api/health`. Criterio: servicio y MongoDB.
- [ ] **T069** [P] [US4] Documentar credenciales demo. Criterio: `quickstart.md`.
- [ ] **T070** [US4] Crear prueba de seed doble. Criterio: mismos conteos.

## Fase 8 — US-005: Trazabilidad, seguridad y pulido (P2)

- [ ] **T071** [US5] Completar `statusHistory` con actor, rol y motivo. Criterio: 100%.
- [ ] **T072** [US5] Completar `takenBy`, `readyBy`, `deliveredBy`, `cancelledBy`. Criterio: 100%.
- [ ] **T073** [P] Agregar `helmet`, CORS y rate limit. Criterio: cabeceras y orígenes correctos.
- [ ] **T074** [P] Documentar endpoints en `contracts/rest-api.md`. Criterio: ejemplos ejecutables.
- [ ] **T075** Revisar que no existan campos de pago, IGV o facturación. Criterio: búsqueda sin resultados.
- [ ] **T076** Revisar que no existan tiempos de cocina en API/UI. Criterio: sin cronómetros ni alertas.
- [ ] **T077** Ejecutar pruebas de concurrencia de raciones. Criterio: 200 envíos, 0 sobreventa.
- [ ] **T078** Crear E2E básico de login y pedido. Criterio: flujo pasa.
- [ ] **T079** Actualizar estado de la spec y `specs/README.md`. Criterio: documentos sincronizados.
- [ ] **T080** Ejecutar análisis de consistencia. Criterio: sin hallazgos críticos.

## Dependencias

```text
Fase 0 → Fase 1 → Fase 2 → Fase 3 → Fase 4 → Fase 5 → Fase 6 → Fase 7 → Fase 8
                                      │
                                      ├── habilita 002
                                      └── habilita 003/004
```

- La Fase 5 debe cerrarse antes de `002`.
- La Fase 6 debe cerrarse antes de `003`, `004` y `005`.
- La Fase 8 es P2 y puede ejecutarse al final del MVP.

## Definition of Done

- [ ] Auth y roles funcionando.
- [ ] Mesas, sesiones y turnos operativos.
- [ ] Menú publicado y raciones auditadas.
- [ ] Pedidos con estados y snapshots.
- [ ] 0 sobreventa y 0 duplicados.
- [ ] Sin pagos, IGV, facturación ni tiempos.
- [ ] Seed reproducible.
- [ ] Pruebas y análisis de consistencia.
