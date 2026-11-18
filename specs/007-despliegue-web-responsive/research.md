---
feature: 007-despliegue-web-responsive
titulo: Decisiones de despliegue web responsive
estado: borrador
creado: 2026-11-17
actualizado: 2026-11-17
fuentes:
  - docs/sources/DOCUMENTO_PROCESOS_CEVICHERIA_LUCY.pdf
---

# Research — Despliegue web responsive

## 1. Decisión del PDF

- La solución es una **web responsive** para móviles y tablets.
- **No es una app nativa.**
- No se menciona APK ni tienda.
- No se define topología de hosting.
- No se menciona offline-first.
- No hay pagos, facturación, IGV ni SUNAT.

## 2. PWA vs. app nativa

| Opción | Ventajas | Limitaciones | Decisión |
|---|---|---|---|
| **Web responsive** | Sin instalación, acceso por URL, misma base para todos. | Requiere navegador y conectividad. | **Requisito** |
| **PWA** | Instalable, shell cacheado, experiencia tipo app. | HTTPS/localhost, service worker, sin offline de datos. | **Opcional** |
| **APK/nativa** | Acceso a hardware, notificaciones nativas. | Fuera del PDF; más mantenimiento. | **No se usa** |

## 3. Hosting

El PDF no fija topología. Opciones viables:

| Opción | Ventajas | Limitaciones |
|---|---|---|
| PC/servidor local | Control, sin nube. | Mantenimiento, IP local, HTTPS. |
| VPS | Control y URL pública. | Administración. |
| PaaS (Render/Railway/Fly.io) | Despliegue simple, HTTPS. | Costo, límites, WebSocket. |
| Frontend estático + API aparte | CDN y separación. | CORS y dos despliegues. |

**Decisión:** no elegir en esta spec; documentar y configurar por entorno.

## 4. MongoDB

- Local para demo en PC.
- Atlas para acceso remoto.
- URI por `MONGODB_URI`.
- Backup con `mongodump`/`mongorestore`.

## 5. Configuración por entorno

- Backend: `MONGODB_URI`, `JWT_SECRET`, `CORS_ORIGIN`, `PORT`.
- Frontend: `VITE_API_URL`, `VITE_SOCKET_URL` (build time).
- Secretos fuera del repo.
- `.env.example` sin valores reales.

## 6. CORS y Socket.IO

- `CORS_ORIGIN` para REST y Socket.IO.
- Si frontend/backend comparten origen, un solo valor.
- Si se separan, agregar el origen del frontend.
- Verificar preflight y handshake.

## 7. Responsive

- Móvil: 360 px+.
- Tablet: 768–1024 px.
- Cocina: 1024×768+.
- Objetivos táctiles ≥ 44 px.
- Sin scroll horizontal.
- Probar Chrome/Edge/Firefox/Safari.

## 8. PWA

- `vite-plugin-pwa` con manifest, iconos y service worker.
- HTTPS o localhost.
- Cachea shell; no cachea datos.
- No promete offline.
- Si no se instala, la web sigue funcionando.

## 9. Datos de demo

- Seed idempotente.
- Usuarios: encargado, mozos, cocinero.
- Mesas, secciones, platos, menú, turno abierto.
- Reset de demo opcional.

## 10. Backup

- `mongodump` y `mongorestore`.
- Probar restauración.
- No almacenar solo en el mismo servidor.

## 11. Riesgos

| Riesgo | Mitigación |
|---|---|
| CORS mal configurado | Guía y smoke test. |
| WebSocket bloqueado | Polling/documentar. |
| `VITE_*` desactualizado | Rebuild. |
| Pérdida de datos | Backup. |
| PWA sin HTTPS | Túnel/certificado. |
| Confusión con APK | Alcance explícito. |
| Hosting no definido | Q-007-01. |

## 12. Preguntas abiertas

- Q-007-01: hosting.
- Q-007-02: dominio/HTTPS.
- Q-007-03: Mongo local/Atlas.
- Q-007-04: PWA.
- Q-007-05: backup.
- Q-007-06: responsable.
