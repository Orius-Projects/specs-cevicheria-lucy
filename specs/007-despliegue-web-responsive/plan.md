---
feature: 007-despliegue-web-responsive
spec: spec.md
estado: borrador
stack: Node.js + Express + Socket.IO / React + Vite PWA / MongoDB
creado: 2026-11-17
actualizado: 2026-11-17
---

# Plan técnico — Despliegue web responsive

## 1. Contexto técnico

| Área | Decisión |
|---|---|
| Backend | Node.js + Express + Socket.IO |
| Frontend | React + Vite; build estático |
| Base de datos | MongoDB local o Atlas |
| PWA | `vite-plugin-pwa` (opcional) |
| Configuración | Variables de entorno |
| Hosting | No decidido; perfiles documentados |
| Nativo | No se usa APK |

## 2. Constitution Check

| Principio | Cumple | Evidencia |
|---|---|---|
| 1. Spec-first y trazabilidad | Sí | Cada FR tiene tarea. |
| 2. Un solo registro, tres vistas | Sí | Misma web para roles. |
| 3. FIFO y una sola acción de cocina | Sí | El despliegue no altera reglas. |
| 4. Control de raciones y agotados | Sí | Se prueba el flujo. |
| 5. Trazabilidad y cierre de turno | Sí | Se prueba el cierre. |
| 6. Web responsive y alcance acotado | Sí | Sin nativo, pagos, IGV ni offline. |

## 3. Arquitectura de despliegue

```mermaid
flowchart LR
    U[Navegador móvil/tablet] -->|HTTP/HTTPS| E[Express + Socket.IO]
    E -->|estático| C[Build React]
    E -->|REST| A[/api]
    E -->|WebSocket| S[Socket.IO]
    E --> M[(MongoDB local o Atlas)]
```

- Servicio único: Express sirve el build y la API.
- Alternativa: frontend estático + backend separado.
- HTTPS recomendado; PWA lo requiere salvo localhost.

## 4. Estructura

```text
cevicheria-lucy/
├── client/
│   ├── dist/
│   └── vite.config.ts
├── server/
│   ├── src/app.ts
│   ├── src/server.ts
│   └── scripts/
│       ├── seed.ts
│       └── backup.sh
├── .env.example
├── package.json
└── DEPLOY.md
```

## 5. Variables

```dotenv
NODE_ENV=production
PORT=3000
MONGODB_URI=mongodb://127.0.0.1:27017/cevicheria_lucy
JWT_SECRET=cambiar-en-produccion
JWT_EXPIRES_IN=8h
CORS_ORIGIN=http://localhost:3000

VITE_API_URL=http://localhost:3000/api
VITE_SOCKET_URL=http://localhost:3000
```

- `JWT_SECRET` obligatorio.
- `CORS_ORIGIN` acepta varios orígenes separados por coma.
- `VITE_*` requiere rebuild.

## 6. Build y ejecución

```bash
npm install
npm run build --workspace client
npm run build --workspace server
npm run seed --workspace server
NODE_ENV=production npm run start --workspace server
```

- Express sirve `client/dist` en producción.
- Fallback SPA para rutas del frontend.
- Socket.IO en el mismo HTTP server.

## 7. CORS

```ts
const origins = env.CORS_ORIGIN.split(',').map((s) => s.trim());
app.use(cors({ origin: origins, credentials: true }));
const io = new Server(httpServer, { cors: { origin: origins, credentials: true } });
```

## 8. MongoDB

- Local: `mongodb://127.0.0.1:27017/cevicheria_lucy`.
- Atlas: `mongodb+srv://...` con IP permitida.
- Índices al iniciar/seed.
- Backup con `mongodump`; restore con `mongorestore`.

## 9. PWA

- `vite-plugin-pwa` con manifest e iconos.
- `registerType: 'autoUpdate'`.
- HTTPS/localhost.
- Cachea shell, no datos.
- Si no se instala, web normal.

## 10. Responsive

- Probar 360, 768, 1024 y 1280 px.
- Cocina en dos columnas.
- Mozo en una columna.
- Objetivos ≥ 44 px.
- Sin scroll horizontal.

## 11. Perfiles

### Servicio único

- Express sirve frontend + API.
- Un solo origen; CORS simple.
- Recomendado para demo.

### Frontend separado

- `client/dist` en hosting estático.
- Backend con `CORS_ORIGIN` y `VITE_*`.
- Requiere WebSocket.

**No se elige topología.**

## 12. Pruebas de despliegue

| Prueba | Criterio |
|---|---|
| Build limpio | Sin errores. |
| Arranque con env | Falla clara si falta variable. |
| Health | `200` y `db: connected`. |
| Login 3 roles | Funciona. |
| Flujo completo | Pedido → cocina → listo → entrega → cierre. |
| CORS/Socket | Sin errores. |
| Seed doble | Sin duplicados. |
| Responsive | 360/768/1024. |
| PWA | Instalable en HTTPS/localhost. |
| Backup/restore | Correcto. |

## 13. Riesgos

| Riesgo | Mitigación |
|---|---|
| CORS | Guía y smoke test. |
| WebSocket | Polling/documentar. |
| `VITE_*` | Rebuild. |
| Datos | Backup. |
| PWA sin HTTPS | Túnel/certificado. |
| Hosting | Q-007-01. |

## 14. Fuera de alcance

- APK/nativo.
- Topología definitiva.
- CI/CD y monitoreo.
- Dominio/SSL.
- Pagos/IGV.
- Offline de datos.

## 15. Checklist del plan

- [x] Variables.
- [x] Build/arranque.
- [x] CORS/Socket.
- [x] Seed/backup.
- [x] PWA/responsive.
- [x] Sin nativo/pagos/IGV.
