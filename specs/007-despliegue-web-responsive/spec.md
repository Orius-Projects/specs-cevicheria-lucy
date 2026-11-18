---
feature: 007-despliegue-web-responsive
titulo: Despliegue web responsive
estado: borrador
fase: MVP
prioridad: P1
procesos:
  - Solución propuesta (web responsive)
  - Decisiones acordadas
depende_de:
  - 001-plataforma-base
  - 002-menu-raciones
  - 003-toma-pedidos-mozo
  - 004-cocina-pendientes
  - 005-avisos-tiempo-real
  - 006-cierre-turno
fuentes:
  - docs/sources/DOCUMENTO_PROCESOS_CEVICHERIA_LUCY.pdf
creado: 2026-11-17
actualizado: 2026-11-17
---

# Especificación — Despliegue web responsive

## 1. Contexto y problema

El PDF define la solución como una **aplicación web responsive** para móviles y tablets, no como una app nativa. La demo debe poder ejecutarse en un navegador, con el backend y MongoDB, y validarse en los dispositivos del restaurante. No se necesita APK ni publicación en tiendas.

Esta spec cubre el build, la configuración por entorno, el seed de demo, la verificación responsive y la PWA opcional. No decide la topología de hosting definitiva ni implementa app nativa.

## 2. Objetivos y no objetivos

### Objetivos

- Construir y ejecutar la web responsive en un entorno accesible por navegador.
- Configurar MongoDB, CORS y URLs por variables de entorno.
- Sembrar datos de demostración y usuarios por rol.
- Verificar el flujo completo en móvil y tablet.
- Ofrecer PWA instalable como mejora opcional.
- Documentar el despliegue y el troubleshooting.

### No objetivos

- No implementa app nativa ni APK.
- No decide la topología de hosting (local, LAN o nube).
- No implementa CI/CD, monitoreo avanzado ni alta disponibilidad.
- No implementa pagos, IGV, facturación ni SUNAT.
- No promete funcionamiento offline de datos.
- No implementa dominio, DNS ni certificados reales.

## 3. Actores y roles

| Actor | Rol | Necesidad principal |
|---|---|---|
| Responsable de despliegue | Técnico | Publicar y configurar la demo. |
| Encargado | `ENCARGADO` | Verificar menú, turno y cierre. |
| Mozo / Cocinero | `MOZO` / `COCINERO` | Usar la web en móvil/tablet. |
| Cliente | — | No usa la web. |

## 4. Historias de usuario

### US-001 — Desplegar y acceder a la web (P1)

**Como** responsable de despliegue, **quiero** publicar la web y acceder desde un navegador, **para** validar el flujo con el restaurante.

**Por qué esta prioridad:** sin demo accesible no hay validación.

**Prueba independiente:** desde un entorno limpio, seguir la guía y acceder con los tres roles.

**Escenarios de aceptación:**

- **DADO** un entorno con Node y MongoDB **CUANDO** sigo la guía **ENTONCES** la web queda accesible.
- **DADO** la web desplegada **CUANDO** la abro en tablet **ENTONCES** el flujo funciona.
- **DADO** un error de configuración **CUANDO** reviso logs **ENTONCES** veo la variable faltante.

### US-002 — Configurar el entorno sin cambiar código (P1)

**Como** responsable de despliegue, **quiero** cambiar MongoDB, CORS y URLs por variables, **para** mover la demo sin tocar código.

**Por qué esta prioridad:** el PDF no fija una topología.

**Prueba independiente:** cambiar `MONGODB_URI` y `CORS_ORIGIN` y reiniciar; la web funciona.

**Escenarios de aceptación:**

- **DADO** otra URI de MongoDB **CUANDO** reinicio **ENTONCES** conecta.
- **DADO** otro origen de frontend **CUANDO** ajusto `CORS_ORIGIN` **ENTONCES** REST y Socket.IO funcionan.
- **DADO** un cambio de `VITE_*` **CUANDO** reconstruyo **ENTONCES** usa la nueva URL.

### US-003 — Sembrar datos de demostración (P1)

**Como** responsable de la demo, **quiero** cargar usuarios, menú, mesas y turno, **para** demostrar sin capturar datos.

**Por qué esta prioridad:** la demo debe ser reproducible.

**Prueba independiente:** ejecutar el seed dos veces y verificar que no duplica.

**Escenarios de aceptación:**

- **DADO** una base vacía **CUANDO** ejecuto el seed **ENTONCES** quedan usuarios, mesas, secciones, platos, menú y turno.
- **DADO** una base sembrada **CUANDO** repito **ENTONCES** no duplica.
- **DADO** un seed fallido **CUANDO** repito **ENTONCES** converge.

### US-004 — Instalar la PWA (P2)

**Como** usuario, **quiero** instalar la web en mi tablet, **para** abrirla como una app sin tienda.

**Por qué esta prioridad:** mejora opcional; el PDF solo exige web responsive.

**Prueba independiente:** instalar desde Chrome Android sobre HTTPS/localhost.

**Escenarios de aceptación:**

- **DADO** la web en HTTPS/localhost **CUANDO** abro en Chrome **ENTONCES** aparece “Instalar”.
- **DADO** la PWA instalada **CUANDO** abro **ENTONCES** funciona el login y el flujo.
- **DADO** un navegador sin soporte **CUANDO** abro **ENTONCES** la web sigue funcionando.

### US-005 — Verificar responsive (P1)

**Como** responsable de la demo, **quiero** probar la web en móvil y tablet, **para** asegurar que el flujo es usable.

**Por qué esta prioridad:** el PDF exige web responsive.

**Prueba independiente:** recorrer el flujo completo en 360 px y 1024 px.

**Escenarios de aceptación:**

- **DADO** un móvil de 360 px **CUANDO** recorro el flujo **ENTONCES** no hay scroll horizontal ni controles cortados.
- **DADO** una tablet **CUANDO** abro cocina **ENTONCES** las columnas se ven correctamente.
- **DADO** un objetivo táctil **CUANDO** lo uso **ENTONCES** mide al menos 44 px.

## 5. Requisitos funcionales (EARS)

| ID | Requisito | Fuente | Prioridad |
|---|---|---|---|
| FR-001 | CUANDO se ejecuta el build del frontend, EL SISTEMA DEBE generar assets estáticos servibles por navegador. | p. 5 | P1 |
| FR-002 | CUANDO se inicia el backend, EL SISTEMA DEBE exponer REST y Socket.IO en el host/puerto configurado. | p. 5 | P1 |
| FR-003 | CUANDO el entorno cambia, EL SISTEMA DEBE leer `MONGODB_URI`, `JWT_SECRET`, `CORS_ORIGIN`, `VITE_API_URL` y `VITE_SOCKET_URL` de variables. | p. 5 | P1 |
| FR-004 | CUANDO un navegador llama a la API, EL SISTEMA DEBE permitir solo los orígenes definidos en `CORS_ORIGIN`. | p. 5 | P1 |
| FR-005 | CUANDO se consulta `/api/health`, EL SISTEMA DEBE reportar servicio y MongoDB. | p. 5 | P1 |
| FR-006 | CUANDO se ejecuta el seed, EL SISTEMA DEBE cargar usuarios, mesas, menú y turno de forma idempotente. | p. 15 | P1 |
| FR-007 | CUANDO se sirve la web, EL SISTEMA DEBE permitir su uso en Chrome, Edge, Firefox y Safari recientes. | p. 5 | P1 |
| FR-008 | CUANDO la web se abre en móvil o tablet, EL SISTEMA DEBE ser responsive desde 360 px. | p. 5 | P1 |
| FR-009 | CUANDO se usa la web, EL SISTEMA DEBE ofrecer objetivos táctiles ≥ 44 px. | p. 5 | P1 |
| FR-010 | CUANDO se configura la PWA, EL SISTEMA DEBE ofrecer manifest, iconos y service worker. | p. 5 | P2 |
| FR-011 | CUANDO se sirve la PWA, EL SISTEMA DEBE usar HTTPS o localhost. | p. 5 | P2 |
| FR-012 | CUANDO se documenta el despliegue, EL SISTEMA DEBE incluir build, variables, seed y troubleshooting. | p. 5 | P1 |
| FR-013 | CUANDO se cambia de entorno, EL SISTEMA NO DEBE requerir cambios de código. | p. 5 | P1 |
| FR-014 | CUANDO se versiona el proyecto, EL SISTEMA NO DEBE incluir secretos ni `.env` reales. | p. 5 | P1 |
| FR-015 | CUANDO se sirve el frontend desde otro origen, EL SISTEMA DEBE usar `VITE_API_URL` y `VITE_SOCKET_URL`. | p. 5 | P1 |
| FR-016 | CUANDO se despliega el backend, EL SISTEMA DEBE permitir Socket.IO desde el origen del frontend. | p. 5 | P1 |
| FR-017 | CUANDO se aloja MongoDB fuera del servidor, EL SISTEMA DEBE usar una URI con credenciales por entorno. | p. 5 | P1 |
| FR-018 | CUANDO se despliega la web, EL SISTEMA NO DEBE requerir app nativa, APK ni tienda. | p. 15 | P1 |
| FR-019 | CUANDO se cierra un turno, EL SISTEMA DEBE seguir disponible para consultar el resumen, pero no para nuevos pedidos. | p. 10 | P1 |
| FR-020 | CUANDO se usa la web, EL SISTEMA NO DEBE prometer sincronización offline de datos. | p. 5 | P1 |
| FR-021 | CUANDO se registran eventos, EL SISTEMA DEBE escribir logs con nivel, timestamp y contexto. | p. 5 | P2 |
| FR-022 | CUANDO se hace backup, EL SISTEMA DEBE poder restaurar MongoDB con `mongodump`/`mongorestore`. | p. 5 | P2 |

## 6. Entidades y estados

No introduce entidades de negocio nuevas. Depende de `001`–`006`.

Configuración de entorno:

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

## 7. Casos borde y errores

| ID | Caso | Comportamiento esperado |
|---|---|---|
| EC-001 | Falta `MONGODB_URI`. | Error claro al arrancar. |
| EC-002 | Falta `JWT_SECRET`. | Error claro. |
| EC-003 | `CORS_ORIGIN` no coincide. | Error de CORS; guía. |
| EC-004 | MongoDB Atlas no permite IP. | Error; agregar IP. |
| EC-005 | Hosting sin WebSocket. | Fallback polling; documentar. |
| EC-006 | Sin HTTPS. | PWA no instalable; web funciona. |
| EC-007 | Backup con MongoDB caído. | Error; no generar archivo vacío. |
| EC-008 | Seed doble. | Sin duplicados. |
| EC-009 | Servidor reiniciado. | Web vuelve; datos persisten. |
| EC-010 | `VITE_*` desactualizado. | Rebuild. |
| EC-011 | Pantalla de 360 px. | Sin scroll horizontal. |
| EC-012 | Navegador sin PWA. | Web normal. |

## 8. Requisitos no funcionales

| ID | Requisito | Métrica / criterio |
|---|---|---|
| NFR-001 | Despliegue reproducible. | ≤ 30 min. |
| NFR-002 | Seed rápido. | ≤ 60 s. |
| NFR-003 | Responsive. | 360 px a 1280 px. |
| NFR-004 | Navegadores modernos. | Chrome, Edge, Firefox, Safari. |
| NFR-005 | Secretos fuera del repo. | `.env` ignorado. |
| NFR-006 | Sin offline de datos. | Solo shell PWA. |
| NFR-007 | Backup verificable. | Restauración probada. |
| NFR-008 | Observabilidad mínima. | Health y logs. |
| NFR-009 | Sin nativo/APK. | 0 dependencias. |
| NFR-010 | Sin pagos/IGV. | 0 campos. |

## 9. Criterios de éxito medibles

| ID | Criterio | Cómo se mide |
|---|---|---|
| SC-001 | Entorno limpio desplegado en ≤ 30 min. | Cronometraje. |
| SC-002 | Seed listo en ≤ 60 s. | Cronometraje. |
| SC-003 | Flujo completo en móvil/tablet. | Prueba manual. |
| SC-004 | Cambiar `MONGODB_URI` sin código. | Prueba con dos bases. |
| SC-005 | PWA instalable en HTTPS/localhost. | Prueba manual. |
| SC-006 | 0 dependencias de app nativa/APK. | Revisión. |
| SC-007 | 0 campos de pago/IGV. | Revisión. |

## 10. Supuestos

- La web se ejecuta en un navegador moderno.
- MongoDB puede ser local o Atlas; la URI se configura.
- El frontend y backend pueden compartir origen o separarse.
- Una instancia de backend en la demo.
- No hay CI/CD obligatorio.
- El dominio y HTTPS se definen al desplegar.
- No hay alta disponibilidad.
- La PWA es opcional.

## 11. Dependencias y riesgos

| Tipo | Descripción | Mitigación |
|---|---|---|
| Dependencia | `001`–`006` funcionales. | Desplegar al final. |
| Dependencia | MongoDB local/Atlas. | Documentar. |
| Riesgo | CORS mal configurado. | Guía y smoke test. |
| Riesgo | WebSocket no soportado. | Polling/documentar. |
| Riesgo | `VITE_*` mal configurado. | Rebuild. |
| Riesgo | Pérdida de datos. | Backup. |
| Riesgo | PWA sin HTTPS. | Túnel/certificado. |
| Riesgo | Confusión con app nativa. | Alcance explícito. |

## 12. Fuera de alcance

- App nativa/APK.
- Decisión de topología de hosting.
- CI/CD, monitoreo avanzado, alta disponibilidad.
- Dominio/DNS/SSL.
- Pagos, IGV, facturación.
- Offline de datos.
- Exportación de resumen.

## 13. Preguntas abiertas

| ID | Pregunta | Impacto | Estado |
|---|---|---|---|
| Q-007-01 | ¿Dónde se desplegará la demo? | Alto | `[NECESITA ACLARACIÓN]`. |
| Q-007-02 | ¿Habrá dominio y HTTPS desde la primera demo? | Medio | `[NECESITA ACLARACIÓN]`. |
| Q-007-03 | ¿MongoDB local o Atlas? | Medio | `[NECESITA ACLARACIÓN]`. |
| Q-007-04 | ¿Se requiere PWA instalable o basta web? | Bajo | `[NECESITA ACLARACIÓN]` (PWA opcional). |
| Q-007-05 | ¿Con qué frecuencia se respaldará? | Bajo | `[NECESITA ACLARACIÓN]`. |
| Q-007-06 | ¿Quién será el responsable de desplegar? | Medio | `[NECESITA ACLARACIÓN]`. |

## 14. Trazabilidad

| Fuente | Sección | Historias | Requisitos | Criterios |
|---|---|---|---|---|
| PDF p. 5 | Solución web responsive | US-001, US-005 | FR-001, FR-002, FR-007–FR-009 | SC-001, SC-003 |
| PDF p. 5 | Configuración y demo | US-002, US-003 | FR-003–FR-006 | SC-002, SC-004 |
| PDF p. 15 | Decisiones: no app nativa | US-004 | FR-010, FR-011, FR-018 | SC-005, SC-006 |
| PDF p. 15 | Sin pagos/IGV | — | FR-020 | SC-007 |
