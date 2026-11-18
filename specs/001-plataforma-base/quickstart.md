---
feature: 001-plataforma-base
titulo: Guía de inicio — Plataforma base v2
estado: borrador
creado: 2026-11-17
actualizado: 2026-11-17
---

# Quickstart — Plataforma base v2

## 1. Requisitos

- Node.js 20 LTS o superior.
- npm 10 o superior.
- MongoDB 6+ local o una URI de MongoDB Atlas.
- Navegador moderno.

## 2. Instalación

```bash
npm install
```

## 3. Variables de entorno

Copiar `.env.example` a `.env`:

```dotenv
NODE_ENV=development
PORT=3000
MONGODB_URI=mongodb://127.0.0.1:27017/cevicheria_lucy
JWT_SECRET=un-secreto-largo-y-unico
JWT_EXPIRES_IN=8h
CORS_ORIGIN=http://localhost:5173

VITE_API_URL=http://localhost:3000/api
VITE_SOCKET_URL=http://localhost:3000
```

## 4. Ejecutar

```bash
# Terminal 1
npm run dev --workspace server

# Terminal 2
npm run dev --workspace client
```

- API: `http://localhost:3000/api`
- Cliente: `http://localhost:5173`

Salud:

```bash
curl http://localhost:3000/api/health
```

## 5. Sembrar demo

```bash
npm run seed --workspace server
```

### Credenciales demo

| Rol | Usuario | Contraseña |
|---|---|---|
| Encargado | `encargado1` | `encargado123` |
| Mozo | `mozo1` | `mozo123` |
| Mozo | `mozo2` | `mozo223` |
| Cocinero | `cocinero1` | `cocina123` |

### Datos sembrados

- 8 mesas `M01`–`M08`.
- Secciones Criollo, Marino, A la carta y Bebidas.
- Al menos 10 platos con precios y variaciones.
- Menú del día publicado con un plato limitado a 30 raciones y uno agotado por insumo.
- Turno `Almuerzo` abierto.
- Configuración: Cevichería Lucy, PEN, umbral 5.

> No hay IGV, pagos ni facturación. El total es la suma de platos.

## 6. Probar la API

### Login

```bash
curl -s -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"mozo1","password":"mozo123"}'
```

Guardar token:

```bash
TOKEN=$(curl -s -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"mozo1","password":"mozo123"}' | jq -r .token)
```

> Si no tienes `jq`, copia el token manualmente.

### Menú de hoy

```bash
curl -s http://localhost:3000/api/menu/today \
  -H "Authorization: Bearer $TOKEN"
```

### Mesas

```bash
curl -s http://localhost:3000/api/tables \
  -H "Authorization: Bearer $TOKEN"
```

### Enviar pedido

```bash
curl -s -X POST http://localhost:3000/api/orders \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "tableId": "REEMPLAZAR",
    "items": [
      { "dishId": "REEMPLAZAR", "quantity": 2, "variationIds": ["v2"], "notes": "Sin ají" }
    ],
    "clientRequestId": "3f1c2a9e-7b1d-4c2f-9a1e-2b6d8f0c4a11"
  }'
```

### Cocina: cola y marcar listo

```bash
TOKEN_COCINA=$(curl -s -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"cocinero1","password":"cocina123"}' | jq -r .token)

curl -s http://localhost:3000/api/kitchen/board \
  -H "Authorization: Bearer $TOKEN_COCINA"

curl -s -X PATCH http://localhost:3000/api/orders/ID/ready \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN_COCINA" \
  -d '{"version":0}'
```

### Mozo: entregar

```bash
curl -s -X PATCH http://localhost:3000/api/orders/ID/deliver \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"version":1}'
```

### Cancelar

```bash
curl -s -X PATCH http://localhost:3000/api/orders/ID/cancel \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"reason":"Cliente se retiró","version":0}'
```

Recuerda:
- Cancelar en `ENVIADO` repone raciones.
- Cancelar en `LISTO` no repone y debe coordinarse con cocina.

### Cerrar mesa

```bash
curl -s -X POST http://localhost:3000/api/table-sessions/ID/close \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"notes":"Mesa atendida"}'
```

### Turno

```bash
TOKEN_ENC=$(curl -s -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"encargado1","password":"encargado123"}' | jq -r .token)

curl -s http://localhost:3000/api/shifts/current \
  -H "Authorization: Bearer $TOKEN_ENC"

curl -s -X POST http://localhost:3000/api/shifts/ID/close \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN_ENC" \
  -d '{"observations":"Cierre de prueba"}'
```

## 7. Pruebas

```bash
npm test --workspace server
npm test --workspace client
npm run e2e --workspace client
```

Casos mínimos:
- Login por rol.
- Menú publicado y agotado.
- Raciones: última unidad y concurrencia.
- Cancelación en Enviado/Listo.
- Cierre de mesa y turno.
- Seed doble.

## 8. Solución de problemas

| Problema | Causa | Solución |
|---|---|---|
| `MONGODB_URI is required` | Falta `.env` | Copiar `.env.example`. |
| `SHIFT_NOT_OPEN` | Turno cerrado | Abrir turno con el encargado. |
| `MENU_NOT_PUBLISHED` | Menú no publicado | Publicar menú del día. |
| `RATION_UNAVAILABLE` | Sin raciones | Reponer o elegir otro plato. |
| `TABLE_HAS_ACTIVE_ORDERS` | Pedidos sin entregar | Entregar/cancelar antes de cerrar. |
| `401` | Token expirado | Volver a iniciar sesión. |
| `503` | MongoDB caído | Revisar la URI/servicio. |

## 9. Notas de alcance

- No hay pagos, IGV, facturación ni SUNAT.
- No hay APK ni app nativa.
- No se muestran tiempos de cocina ni alertas de demora.
- Los mockups son referenciales.
