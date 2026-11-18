# Tareas — Título del feature

> **Formato:** cada tarea tiene ID secuencial `T###`, marca `[P]` si puede ejecutarse en paralelo, etiqueta `[US#]` si pertenece a una historia y la ruta de archivo afectada. Ejecutar por fases.

## Fase 0 — Setup

- [ ] T001 Crear la estructura base en `client/`, `server/` y `shared/`.
- [ ] T002 [P] Configurar TypeScript, ESLint y Prettier.
- [ ] T003 [P] Configurar variables de entorno en `.env.example`.

## Fase 1 — Base compartida (bloqueante)

- [ ] T004 Configurar conexión a MongoDB en `server/src/config/db.ts`.
- [ ] T005 Implementar middleware de errores en `server/src/middlewares/error.ts`.
- [ ] T006 Implementar autenticación JWT en `server/src/middlewares/auth.ts`.
- [ ] T007 Crear modelos Mongoose en `server/src/models/`.
- [ ] T008 Crear tipos compartidos en `shared/types/`.

## Fase 2 — US-001 (P1)

- [ ] T009 [US1] Implementar endpoint en `server/src/routes/`.
- [ ] T010 [US1] Implementar servicio en `server/src/services/`.
- [ ] T011 [P] [US1] Crear store de Zustand en `client/src/stores/`.
- [ ] T012 [US1] Crear página en `client/src/pages/`.
- [ ] T013 [US1] Probar el flujo en `client/src/__tests__/`.

## Fase 3 — US-002 (P2)

- [ ] T014 [US2] ...

## Fase N — Pruebas y pulido

- [ ] T015 [P] Pruebas unitarias de reglas de dominio.
- [ ] T016 [P] Pruebas de integración REST.
- [ ] T017 Prueba E2E del flujo completo.
- [ ] T018 Actualizar `quickstart.md` y documentación.
- [ ] T019 Ejecutar análisis de consistencia y corregir hallazgos.

## Dependencias

```text
Fase 0 → Fase 1 → Fase 2 → Fase 3 → Fase N
```

## Definition of Done

- [ ] Todas las tareas de la fase completadas.
- [ ] Pruebas pasando.
- [ ] Sin hallazgos críticos de consistencia.
- [ ] Documentación actualizada.
