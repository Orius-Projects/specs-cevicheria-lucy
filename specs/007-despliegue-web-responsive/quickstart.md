---
feature: 007-despliegue-web-responsive
titulo: Guía de despliegue web responsive
estado: borrador
creado: 2026-11-17
actualizado: 2026-11-17
---

# Quickstart — Despliegue web responsive

> No se decide topología de hosting. Se documenta una demo web ejecutable en navegador. No hay APK.

## 1. Requisitos

- Node.js 20 LTS o superior.
- npm 10 o superior.
- MongoDB 6+ local o Atlas.
- Navegador moderno.

## 2. Variables de entorno

```dotenv
NODE_ENV=production
PORT=3000
MONGODB_URI=mongodb://127.0.0.1:27017/cevicheria_lucy
JWT_SECRET=un-secreto-largo-y-unico
JWT_EXPIRES_IN=8h
CORS_ORIGIN=http://localhost:3000

VITE_API_URL=http://localhost:3000/api
VITE_SOCKET_URL=http://localhost:3000
```

- `JWT_SECRET` obligatorio.
- `CORS_ORIGIN` puede tener varios orígenes separados por coma.
- `VITE_*` requiere rebuild.

## 3. Build y arranque

```bash
npm install
npm run build --workspace client
npm run build --workspace server
npm run seed --workspace server
NODE_ENV=production npm run start --workspace server
```

> Windows PowerShell: `$env:NODE_ENV='production'; npm run start --workspace server`.

Abrir `http://localhost:3000`.

## 4. Credenciales demo

| Rol | Usuario | Contraseña |
|---|---|---|
| Encargado | `encargado1` | `encargado123` |
| Mozo | `mozo1` | `mozo123` |
| Mozo | `mozo2` | `mozo223` |
| Cocinero | `cocinero1` | `cocina123` |

## 5. Acceso desde tablet en la misma red

1. Obtener la IP local del servidor:

```bash
# Windows
ipconfig
# Linux/macOS
ifconfig
```

2. Ajustar `CORS_ORIGIN`, `VITE_API_URL` y `VITE_SOCKET_URL` con la IP, por ejemplo `http://192.168.1.50:3000`.
3. Reconstruir el frontend.
4. Abrir `http://192.168.1.50:3000` desde la tablet.
5. Verificar que la red permita la comunicación.

> Sin HTTPS, la PWA no se instala; la web funciona.

## 6. MongoDB Atlas

1. Crear clúster.
2. Crear usuario.
3. Permitir la IP del servidor.
4. Copiar `mongodb+srv://...` a `MONGODB_URI`.

## 7. PWA (opcional)

- Requiere HTTPS o localhost.
- Abrir en Chrome Android y elegir “Instalar aplicación”.
- El service worker cachea el shell; no los datos.
- Si no se instala, la web sigue funcionando.

## 8. Backup y restore

```bash
mongodump --uri="$MONGODB_URI" --out=./backups/20261117-1830
mongorestore --uri="$MONGODB_URI" ./backups/20261117-1830
```

> Windows PowerShell: usar una carpeta manual, por ejemplo `--out=.\backups\20261117-1830`.

Probar la restauración al menos una vez.

## 9. Prueba de humo

| Paso | Resultado |
|---|---|
| `/api/health` | `200`, `db: connected`. |
| Login 3 roles | Correcto. |
| Menú publicado | Visible. |
| Pedido | Aparece en cocina. |
| Listo | Aviso a todos los mozos. |
| Entregado | Sale de listos. |
| Cierre de turno | Resumen guardado. |
| Seed doble | Sin duplicados. |
| Backup/restore | Correcto. |
| 360 px | Sin scroll horizontal. |
| 1024×768 | Cocina en dos columnas. |

## 10. Solución de problemas

| Problema | Causa | Solución |
|---|---|---|
| No arranca | Falta `.env` | Revisar error y `.env.example`. |
| CORS | Origen distinto | Ajustar `CORS_ORIGIN`. |
| Socket.IO | URL/WebSocket | Revisar `VITE_SOCKET_URL`; probar polling. |
| API localhost | `VITE_*` sin rebuild | Reconstruir. |
| PWA no instala | Sin HTTPS | Localhost o túnel. |
| MongoDB | URI/IP | Revisar Atlas. |
| Página en blanco | Build ausente | Revisar `client/dist`. |

## 11. Alcance

- Web responsive, no app nativa.
- Sin APK.
- Sin pagos, IGV, facturación.
- Sin offline de datos.
- Sin exportación de resumen.
- La topología de hosting se decide después.
