# Tareas — Despliegue web responsive

> Ejecutar cuando `001`–`006` estén funcionales. Las tareas `[P]` pueden ir en paralelo.

## Fase 0 — Build y configuración

- [ ] **T001** Configurar scripts `build`, `start` y `preview` en los workspaces. Criterio: build sin errores.
- [ ] **T002** [P] Completar `.env.example` con variables. Criterio: sin secretos reales.
- [ ] **T003** Validar variables obligatorias al arrancar. Criterio: error claro.
- [ ] **T004** Documentar que `VITE_*` requiere rebuild. Criterio: nota en `quickstart.md`.
- [ ] **T005** Configurar modo producción en `server/src/app.ts`. Criterio: sirve `client/dist` y fallback SPA.

## Fase 1 — Servicio único y CORS

- [ ] **T006** Servir build estático desde Express. Criterio: `/` devuelve `index.html`.
- [ ] **T007** Excluir `/api` y `/socket.io` del fallback. Criterio: API/WebSocket funcionan.
- [ ] **T008** Configurar CORS REST. Criterio: origen permitido.
- [ ] **T009** Configurar CORS Socket.IO. Criterio: handshake.
- [ ] **T010** Verificar mismo HTTP server. Criterio: eventos.
- [ ] **T011** [P] Agregar `helmet`. Criterio: cabeceras.
- [ ] **T012** Probar preflight. Criterio: `OPTIONS` correcto.

## Fase 2 — Datos y salud

- [ ] **T013** [P] Implementar seed de demo. Criterio: idempotente.
- [ ] **T014** [P] Implementar `/api/health`. Criterio: servicio y MongoDB.
- [ ] **T015** [P] Documentar `mongodump`/`mongorestore`. Criterio: comandos ejecutables.
- [ ] **T016** Probar seed doble. Criterio: sin duplicados.
- [ ] **T017** Probar backup/restore. Criterio: datos recuperados.
- [ ] **T018** Documentar reset de demo. Criterio: comando seguro.

## Fase 3 — PWA

- [ ] **T019** Configurar manifest e iconos. Criterio: instalable.
- [ ] **T020** Configurar service worker. Criterio: shell cacheado.
- [ ] **T021** Verificar HTTPS/localhost. Criterio: instalación.
- [ ] **T022** Documentar limitación sin HTTPS. Criterio: nota.
- [ ] **T023** Probar PWA en Chrome Android. Criterio: standalone.

## Fase 4 — Responsive

- [ ] **T024** [P] Probar 360 px (móvil). Criterio: sin scroll horizontal.
- [ ] **T025** [P] Probar 768 px (tablet). Criterio: layout correcto.
- [ ] **T026** [P] Probar 1024×768 (cocina). Criterio: dos columnas.
- [ ] **T027** [P] Probar 1280 px. Criterio: legible.
- [ ] **T028** Verificar objetivos táctiles ≥ 44 px. Criterio: todos.
- [ ] **T029** Verificar navegadores modernos. Criterio: Chrome/Edge/Firefox/Safari.

## Fase 5 — Documentación y perfiles

- [ ] **T030** Documentar servicio único. Criterio: pasos.
- [ ] **T031** Documentar frontend separado. Criterio: `VITE_*` y CORS.
- [ ] **T032** Documentar MongoDB local/Atlas. Criterio: URI.
- [ ] **T033** Documentar que no hay APK/nativo. Criterio: alcance.
- [ ] **T034** Crear `DEPLOY.md` resumido. Criterio: enlaza quickstart.

## Fase 6 — Pruebas de humo

- [ ] **T035** Build limpio en entorno nuevo. Criterio: ≤ 30 min.
- [ ] **T036** Login 3 roles. Criterio: funciona.
- [ ] **T037** Flujo completo en tablet. Criterio: pedido → cocina → listo → entrega → cierre.
- [ ] **T038** CORS y Socket.IO. Criterio: sin errores.
- [ ] **T039** Seed y reset. Criterio: reproducibles.
- [ ] **T040** Backup/restore. Criterio: correcto.
- [ ] **T041** Health y logs. Criterio: diagnóstico.
- [ ] **T042** Verificar que no existan pagos/IGV/APK. Criterio: búsqueda sin resultados.
- [ ] **T043** Actualizar `specs/README.md` y estado. Criterio: sincronizado.
- [ ] **T044** Ejecutar análisis de consistencia. Criterio: sin hallazgos críticos.

## Dependencias

```text
001-006 ──► Fase 0 → Fase 1 → Fase 2 → Fase 3 → Fase 4 → Fase 5 → Fase 6
```

## Definition of Done

- [ ] Build reproducible.
- [ ] Servicio único funcionando.
- [ ] CORS/Socket correctos.
- [ ] Seed/backup probados.
- [ ] PWA instalable (opcional).
- [ ] Responsive 360–1280.
- [ ] Sin nativo/APK/pagos/IGV.
- [ ] Flujo completo en tablet.
