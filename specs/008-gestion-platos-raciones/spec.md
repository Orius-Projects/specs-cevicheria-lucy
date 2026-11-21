# Spec 008 — Gestión de platos y raciones (Encargado)

**Feature:** `008-gestion-platos-raciones`  
**Prioridad:** P1  
**Estado:** implementada (demo)  
**Fecha:** 2026-11-21  
**Actores:** ENCARGADO (principal), MOZO y COCINERO (consumidores del menú publicado)  
**Depende de:** 001, 002, 006, 007  
**Design system:** [`docs/design-system.md`](../../docs/design-system.md)

---

## 1. Problema

El encargado tenía que alternar entre tres pantallas para operar el día:

1. `Menú del Día` (`/encargado/menu`) para crear el borrador y publicar.
2. `Raciones y Agotados` (`/encargado/raciones`) para reponer, ajustar o agotar platos.
3. `Turnos` (`/encargado`) para abrir o cerrar el turno.

Ese recorrido agrega fricción en el momento de mayor presión del servicio y duplica conceptos: `MenuDayPage` y `RationAdjustmentsPage` muestran los mismos platos con acciones parecidas. Además, el catálogo semilla no representaba el menú real del restaurante (faltaban entradas, marinos y criollos; “A la carta” estaba activa).

La primera pantalla del encargado debía responder una sola pregunta operativa: **¿qué platos hay hoy, cuántas raciones quedan y qué está agotado?**

---

## 2. Objetivos

- Convertir `/encargado` en la pantalla principal **“Platos y raciones”** y usarla como primera vista tras el login del ENCARGADO.
- Reducir la navegación principal a tres apartados: **Platos y raciones**, **Turno** y **Cierre de turno**.
- Permitir ver y actualizar el menú del día sin salir de la pantalla:
  - ver platos agrupados por categoría;
  - filtrar por categoría y buscar por nombre;
  - ajustar raciones con `-` / `+`;
  - marcar agotado / reactivar;
  - editar el ítem del borrador en un panel lateral/bottom sheet.
- Sembrar el catálogo real del restaurante y un menú del día idempotente con estados variados para la demo.
- Mantener intactas las reglas de pedidos, cocina, mozo y cierre de turno.
- Conservar usuarios, mesas, turnos, pedidos y base de datos existentes.

### No objetivos

- No se implementa pago, facturación, SUNAT ni control de inventario de almacén.
- No se agrega un endpoint de “despublicar”; la regla de agregar/quitar platos solo en borrador se mantiene.
- No se cambian los contratos REST existentes de pedidos, cocina o cierre.
- No se elimina `DishesPage` ni sus endpoints: pasa a ser catálogo avanzado.

---

## 3. Catálogo semilla

### 3.1 Secciones

| Sección | Nombre | Activa | Uso en el menú del día |
|---|---|---|---|
| `CRIOLLO` | Criollo | Sí | Fondos fijos, premium y rotativos |
| `ENTRADA` | Entrada | Sí | Entradas |
| `MARINO` | Marino | Sí | Platos marinos |
| `BEBIDAS` | Bebidas | Sí | Bebidas |
| `A_LA_CARTA` | A la carta | **No** | Se conserva el registro, pero no recibe ítems |

`A_LA_CARTA` queda con `active: false`; los ítems de esa sección se retiran **solo del menú del día actual**. No se elimina ningún documento histórico.

### 3.2 Platos

#### CRIOLLO — fijos (incluyen arroz + papa frita)

| Plato | Precio provisional | Tipo |
|---|---:|---|
| Chicharrón de pollo | S/ 22.00 | Fijo |
| Milanesa de pollo | S/ 22.00 | Fijo |
| Filete de pollo | S/ 23.00 | Fijo |

#### CRIOLLO — fijos premium (con papa frita)

| Plato | Precio provisional | Tipo |
|---|---:|---|
| Bisteck con papa frita | S/ 28.00 | Fijo |
| Chuleta de cerdo con papa frita | S/ 28.00 | Fijo |
| Trucha frita con papa frita | S/ 30.00 | Fijo |
| Chaufa de cecina | S/ 26.00 | Fijo |

#### CRIOLLO — rotativos de ejemplo

| Plato | Precio provisional | Tipo |
|---|---:|---|
| Ají de gallina | S/ 24.00 | Rotativo |
| Arroz con pollo | S/ 24.00 | Rotativo |
| Seco de res | S/ 26.00 | Rotativo |
| Tallarín saltado | S/ 25.00 | Rotativo |

#### ENTRADA

| Plato | Precio provisional |
|---|---:|
| Caldo de pollo | S/ 12.00 |
| Ceviche | S/ 28.00 |
| Papa rellena | S/ 10.00 |
| Causa de atún | S/ 18.00 |
| Alita broster | S/ 15.00 |
| Chilcano | S/ 15.00 |
| Combinado | S/ 20.00 |
| Ensalada con palta | S/ 14.00 |
| Tequeños | S/ 12.00 |

#### MARINO

| Plato | Precio provisional |
|---|---:|
| Arroz con mariscos | S/ 30.00 |
| Sudado de pescado | S/ 28.00 |
| Chicharrón de pescado | S/ 26.00 |
| Chaufa de marisco | S/ 32.00 |

#### BEBIDAS

Se conservan las bebidas existentes:

| Plato | Precio |
|---|---:|
| Chicha morada 1L | S/ 12.00 |
| Limonada frozen 1L | S/ 14.00 |

### 3.3 Menú del día de hoy (seed idempotente)

El seed publica un menú con **26 ítems**: 11 criollos (7 fijos + 4 rotativos), 9 entradas, 4 marinos y 2 bebidas. Los estados son variados para la demo:

- Varios platos con límite de raciones.
- **Ají de gallina** en últimas raciones: `4 / 15`.
- **Arroz con mariscos** agotado por insumo, con motivo registrado.
- Platos sin límite (por ejemplo, Seco de res, Caldo de pollo, Chicha morada).
- Todos los platos incluyen precios provisionales editables desde el panel de edición.

El seed:

- Hace `upsert` por nombre de plato y por `(menuDayId, dishId)`.
- Desactiva (no borra) los platos del seed anterior que ya no pertenecen al menú real.
- Elimina únicamente los ítems de hoy del catálogo anterior y de `A_LA_CARTA`.
- No toca pedidos, mesas, usuarios ni menús históricos.
- Crea el turno `Almuerzo` solo si no existe; si el encargado lo cerró, lo respeta.

---

## 4. Nueva arquitectura de información y navegación

### 4.1 Navegación principal del ENCARGADO

| Orden | Etiqueta | Ruta | Pantalla |
|---|---|---|---|
| 1 | Platos y raciones | `/encargado` | `DishManagementPage` |
| 2 | Turno | `/encargado/turnos` | `AdminPage` (turnos) |
| 3 | Cierre de turno | `/encargado/cierre` | `ShiftClosePage` |

`Platos Maestros` deja de estar en la navegación principal. La ruta `/encargado/platos` se conserva como **catálogo avanzado** y se accede desde el enlace secundario **“Catálogo maestro”** dentro de la pantalla principal.

### 4.2 Rutas antiguas

| Ruta antigua | Comportamiento |
|---|---|
| `/encargado/menu` | Redirige a `/encargado` (`Navigate replace`) |
| `/encargado/raciones` | Redirige a `/encargado` (`Navigate replace`) |

No se rompen enlaces guardados ni accesos directos.

### 4.3 Cabecera de la pantalla principal

Debe mostrar, en una sola tarjeta compacta:

- fecha del menú (con etiqueta “Hoy (Lima)” cuando corresponde);
- estado del menú (`BORRADOR` / `PUBLICADO` / `SIN MENÚ`);
- estado del turno con enlace a `/encargado/turnos`;
- acciones **Guardar** y **Publicar** solo cuando el menú está en borrador;
- enlace secundario **Catálogo maestro**;
- botón **Actualizar** para recargar menú y turno.

Si no hay turno abierto, se muestra además un banner con enlace para abrirlo.

---

## 5. Flujos

### 5.1 Ver el menú del día

1. El encargado inicia sesión y cae en `/encargado`.
2. La pantalla carga platos del día y turno actual.
3. Las categorías se muestran como chips con conteo: **Todos, Criollo, Entrada, Marino, Bebidas**.
4. Los platos se agrupan por categoría y se ordenan por el orden del catálogo maestro.
5. Cada tarjeta muestra nombre, categoría, descripción breve, precio, badge de disponibilidad y raciones (`restantes / total` o `Sin límite`).
6. En móvil las tarjetas se apilan; en desktop se usa un grid de 2–3 columnas.

### 5.2 Ajustar raciones y disponibilidad

1. En una tarjeta con límite, `-` resta una ración y `+` suma una ración.
2. Cada cambio usa los endpoints existentes:
   - `POST /menu-items/:id/rations/adjust` para fijar el conteo (resta);
   - `POST /menu-items/:id/rations/replenish` para reponer (suma);
   - `POST /menu-items/:id/sold-out` para agotar;
   - `POST /menu-items/:id/reactivate` para reactivar.
3. Los cambios se guardan de inmediato y actualizan la tarjeta sin recargar la página.
4. El botón **Agotado** abre el modal de motivo; el botón **Reactivar** se muestra cuando el plato está agotado manualmente.
5. El panel de edición permite ajustar el conteo con motivo y acceder al historial de ajustes.

### 5.3 Editar un plato en borrador

1. Al tocar el contenido de la tarjeta se abre `DishEditSheet` (bottom sheet en móvil, modal en desktop).
2. En borrador se pueden editar: activo para venta, precio, límite de raciones, total y restantes.
3. **Guardar cambios** usa `PUT /menu/days/:id` (`updateDay`) con todos los ítems del borrador.
4. **Registrar ajuste ahora** usa `POST /menu-items/:id/rations/adjust` y deja auditoría con el motivo.
5. **Quitar del menú** solo está disponible en borrador.
6. **Historial de ajustes** abre la auditoría existente de raciones.

### 5.4 Agregar platos

1. El botón **Agregar platos** solo aparece en borrador.
2. Abre el catálogo maestro filtrado por la categoría activa.
3. Se muestran solo platos activos que no estén ya en el menú.
4. Al seleccionar uno se agrega al arreglo local y se persiste de inmediato con `updateDay`.
5. Si el menú está publicado, el botón no se muestra y aparece un aviso claro sobre la regla de borrador.

### 5.5 Publicar

1. **Guardar** persiste el borrador actual.
2. **Publicar** guarda primero y luego llama a `POST /menu/days/:id/publish`.
3. Si no hay ítems activos, se mantiene la validación actual (422) y se muestra el error.
4. Tras publicar, las raciones y la disponibilidad siguen siendo editables en caliente.

### 5.6 Sin menú o sin turno

- Si no hay menú para la fecha, se muestra una tarjeta con CTA **Crear menú del día**.
- Si no hay turno abierto, se muestra `ShiftBanner` con enlace a **Turno**.
- Ambas situaciones son recuperables sin salir de `/encargado`.

---

## 6. Reglas de negocio y consistencia

- Los cambios de raciones y disponibilidad se registran con actor, motivo y fecha mediante los endpoints existentes.
- En borrador, editar campos del ítem usa `updateDay`; el servicio no debe reactivar por accidente un plato marcado como agotado.
- En publicado no se permite agregar/quitar platos desde esta pantalla.
- El menú del mozo se alimenta de `GET /menu/today`; si el menú está publicado, refleja las categorías y platos nuevos.
- El seed es idempotente y no borra la base ni pedidos.
- `A_LA_CARTA` no aparece en el menú del día ni en los chips principales.

---

## 7. Criterios de aceptación

- **AC-008-01:** Al iniciar sesión como ENCARGADO, la primera pantalla es `/encargado` y muestra “Platos y raciones”.
- **AC-008-02:** La navegación principal tiene exactamente 3 ítems: Platos y raciones, Turno y Cierre de turno.
- **AC-008-03:** `/encargado/turnos` renderiza el control de turnos y `/encargado/cierre` sigue funcionando.
- **AC-008-04:** `/encargado/menu` y `/encargado/raciones` redirigen a `/encargado`.
- **AC-008-05:** El catálogo semilla contiene las 4 secciones activas y `A_LA_CARTA` inactiva, con los 26 platos definidos.
- **AC-008-06:** El menú de hoy contiene 11 criollos, 9 entradas, 4 marinos y 2 bebidas; incluye al menos un plato con límite, uno en últimas raciones y uno agotado.
- **AC-008-07:** Los chips muestran Todos, Criollo, Entrada, Marino y Bebidas con conteo; a 360 px se ven todos en máximo dos filas.
- **AC-008-08:** El buscador filtra por nombre y descripción sin recargar.
- **AC-008-09:** El stepper `-` / `+` ajusta raciones con botones de al menos 44 px y persiste de inmediato.
- **AC-008-10:** “Agotado” y “Reactivar” actualizan el estado y se reflejan en la tarjeta y en el menú del mozo.
- **AC-008-11:** En borrador se puede editar activo, precio, límite, total y restantes; “Guardar cambios” persiste con `updateDay`.
- **AC-008-12:** En borrador se pueden agregar platos desde el catálogo filtrado por categoría y solo se muestran activos no incluidos.
- **AC-008-13:** En publicado se pueden ajustar raciones y disponibilidad, pero no agregar/quitar platos; se muestra el aviso.
- **AC-008-14:** El panel de edición incluye acceso al historial de ajustes.
- **AC-008-15:** No hay overflow horizontal a 360 × 640 ni 390 × 844; las tarjetas se apilan y los nombres no se truncan.
- **AC-008-16:** `npm run typecheck`, `npm run lint` y `npm test` quedan en verde.
- **AC-008-17:** La demo en `5173/3000` sigue viva y `/mozo/menu` refleja las nuevas secciones y platos.

---

## 8. Supuestos

1. **Precios provisionales:** no se cuenta con la lista oficial de precios. Los valores de la sección 3 son coherentes para la demo, se muestran como editables en la UI y **no deben tratarse como definitivos**.
2. **Platos rotativos de ejemplo:** Ají de gallina, Arroz con pollo, Seco de res y Tallarín saltado son ejemplos para la demo; el encargado podrá cambiarlos a diario.
3. **Nombres:** se usan los nombres entregados en el brief, sin normalizar tildes ni marcas.
4. **Bebidas:** se conservan las dos bebidas existentes; no se agregan nuevas.
5. **Regla de publicación:** se mantiene la regla de la spec 002: solo un menú en borrador se puede editar en estructura (agregar/quitar platos).
6. **Turno:** el seed crea el turno Almuerzo solo si no existe; no reabre un turno cerrado.
7. **Catálogo legado:** los platos del seed anterior se desactivan, no se eliminan, para no romper pedidos históricos.

---

## 9. Preguntas abiertas

| ID | Pregunta | Impacto |
|---|---|---|
| Q-008-01 | ¿Cuál es la lista oficial de precios de cada plato? | Alto |
| Q-008-02 | ¿Qué platos rotativos se servirán cada día de la semana? | Medio |
| Q-008-03 | ¿Se permite reponer por encima del total preparado? La UI hoy limita el `+` al total. | Medio |
| Q-008-04 | ¿La “Chuleta de cerdo” y el “Bisteck” incluyen arroz además de papa frita? | Bajo |
| Q-008-05 | ¿Las entradas “Combinado” y “Chilcano” son platos de fondo o entradas para el menú? | Medio |
| Q-008-06 | ¿Se requiere un flujo explícito para volver un menú publicado a borrador? | Medio |

---

## 10. Verificación

- Pruebas unitarias/componentes: `AdminNavigation`, `AdminLayout`, `CategoryTabs`, `DishRationCard`, `ShiftBanner`.
- Pruebas de servidor: seed idempotente con conteos del catálogo real, estados del menú y conservación de secciones.
- Resultado de la corrida de cierre:
  - `npm run typecheck`: OK en `shared`, `server` y `client`.
  - `npm run lint`: 0 errores (2 warnings preexistentes en `WaiterDashboardPage.tsx`).
  - `npm test`: servidor 15 archivos / 108 tests; cliente 27 archivos / 113 tests.
- Pruebas manuales/automatizadas a **360 × 640** y **390 × 844**:
  - `scrollWidth == clientWidth` (sin overflow horizontal).
  - 5 chips visibles en 2 filas.
  - 26 tarjetas, nombres sin truncar y stepper de 44 × 44 px.
  - Rutas antiguas redirigidas y `/encargado/turnos` operativo.
- Flujo de borrador verificado en navegador: crear menú, agregar plato, editar precio/límite y quitar plato.
- Verificación de la demo:
  - API `http://localhost:3000/api/health` en `ok`.
  - Cliente `http://localhost:5173` responde 200.
  - `/mozo/menu` muestra Criollo, Entrada, Marino y Bebidas, sin `A_LA_CARTA`.

---

## 11. Referencias de implementación

- `client/src/pages/encargado/DishManagementPage.tsx`
- `client/src/components/menu/DishRationCard.tsx`
- `client/src/components/menu/DishEditSheet.tsx`
- `client/src/components/menu/CategoryTabs.tsx`
- `client/src/components/menu/ShiftBanner.tsx`
- `client/src/components/menu/AdminNavigation.tsx`
- `client/src/routes/AppRoutes.tsx`
- `client/src/stores/menuAdminStore.ts`
- `server/src/seed/data.ts`
- `server/src/services/menu.service.ts`
