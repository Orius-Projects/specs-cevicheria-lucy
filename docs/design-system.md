# Design System — Cevichería Lucy

**Estado:** vigente  
**Fecha:** 2026-11-21  
**Fuente de verdad de color:** `app-cevicheria-lucy/client/tailwind.config.js`  
**Alcance:** cliente web responsive (React + Vite + Tailwind CSS).

---

## 1. Propósito, alcance y principios

Este documento define las decisiones visuales y de interacción que deben seguir todos los cambios de UI del sistema. Su objetivo es que la demo, las pantallas operativas y los futuros módulos se vean y se comporten como un solo producto, sin depender de la memoria de cada desarrollador.

Principios:

1. **Mobile-first real.** Se diseña y valida primero a 360 px; luego 390, 768, 1024 y 1280 px. Ninguna pantalla debe producir scroll horizontal de página.
2. **Celeste como identidad, no como decoración.** La familia `brand` identifica acciones y foco. Los estados usan colores semánticos (éxito, advertencia, peligro, información).
3. **Contraste antes que estética.** Ningún texto se apoya en un color que no cumpla WCAG AA. El celeste brillante se usa como superficie con texto oscuro, nunca con texto blanco.
4. **Componentes, no parches.** Un bug de tabs se corrige en el componente compartido, no con un `overflow-hidden` en la página.
5. **Áreas táctiles generosas.** Todo control interactivo mide al menos 44 × 44 px y deja separación suficiente entre acciones.

---

## 2. Paleta oficial

### 2.1 Anclas

| Nombre | Hex | Token |
|---|---|---|
| Celeste pálido | `#b6ffff` | `brand-100` |
| Celeste claro | `#7cdaf9` | `brand-200` |
| Celeste primario | `#0cb7f2` | `brand-500` |

### 2.2 Escala `brand` 50–950

| Token | Hex | Rol principal | Derivación |
|---|---|---|---|
| `brand-50` | `#eefcff` | Fondo sutil, hover de superficies | `brand-100` aclarado ~75 % hacia blanco |
| `brand-100` | `#b6ffff` | Fondo de badges y chips, highlight | **Ancla** celeste pálido |
| `brand-200` | `#7cdaf9` | Bordes y acentos decorativos | **Ancla** celeste claro |
| `brand-300` | `#4ed0f7` | Acentos, anillos suaves | Interpolado 200 → 500 (1/3), ajustado a mano |
| `brand-400` | `#25c5f5` | Hover de botones primarios | Interpolado 200 → 500 (2/3), ajustado a mano |
| `brand-500` | `#0cb7f2` | **Superficie primaria** de acciones | **Ancla** celeste primario |
| `brand-600` | `#0891c4` | Iconos, bordes activos y UI no textual | 500 oscurecido ~10 puntos de luminosidad HSL |
| `brand-700` | `#077093` | Texto de marca sobre blanco; fondos con texto blanco | 500 oscurecido ~20 puntos |
| `brand-800` | `#0a5874` | Texto de marca en hover / énfasis | 500 oscurecido ~25 puntos |
| `brand-900` | `#0b475d` | Texto de alto énfasis sobre superficies claras | 500 oscurecido ~30 puntos |
| `brand-950` | `#052e3d` | Texto sobre celeste y contraste máximo | 500 oscurecido ~37 puntos |

Los pasos 300 y 400 se interpolaron en sRGB entre `#7cdaf9` y `#0cb7f2` y se ajustaron manualmente para mantener la armonía visual. Los pasos 600–950 conservan el mismo tono aproximado (195–196°) y reducen la luminosidad HSL en pasos de 5–10 puntos. No se usan hex sueltos en componentes: siempre se referencia el token (`bg-brand-500`, `text-brand-950`, etc.). Si en el futuro se migra a variables CSS, el mapeo debe vivir en `client/src/styles/index.css` y mantener exactamente estos valores.

### 2.3 Colores semánticos

| Rol | Tokens habituales | Uso |
|---|---|---|
| Éxito | `emerald-50/100/700/800` | Turno abierto, pedido listo, confirmaciones |
| Advertencia | `amber-50/100/800/900`, `amber-700` para botón sólido | Últimas raciones, observaciones, cierre manual |
| Peligro | `rose-50/100/600/700` | Agotado, cancelaciones, cierre de turno |
| Información | `brand-50/100/700/800` | Estado “Enviado”, ayudas, enlaces |
| Neutro | `slate-50/100/200/300/500/600/700/900` | Superficies, texto y bordes |

Los botones sólidos de estado usan `emerald-700`, `amber-700` o `rose-600/700` con texto blanco, porque esos tonos sí alcanzan 4.5:1. No se usa `emerald-600` ni `amber-600` con texto blanco en tamaños normales.

---

## 3. Roles de color y proporción 60/30/10

- **60 % — Superficies neutras:** `bg-slate-50` de página y `bg-white` de tarjetas. Es la base visual del sistema.
- **30 % — Celeste de marca:** `brand-50/100` como fondos de apoyo, `brand-200/300` como bordes/acentos, `brand-500` en la acción principal y `brand-700` en texto/enlaces. El celeste no debe inundar la pantalla: se reserva para jerarquía y acción.
- **10 % — Estados y énfasis:** verde, ámbar y rosa para estados del pedido, disponibilidad y alertas.

Reglas prácticas:

- **Superficie primaria:** `bg-brand-500` + `text-brand-950`.
- **Hover primario:** `hover:bg-brand-400`.
- **Secundario:** `bg-brand-50` + `text-brand-700` + `border-brand-200`; hover `bg-brand-100`.
- **Texto de marca:** `text-brand-700` mínimo sobre blanco; `text-brand-800` para hover/énfasis.
- **Bordes de marca:** `border-brand-200` (suave), `border-brand-600` (selección/estado).
- **Focus ring:** `ring-brand-700` con `ring-offset-2` en fondos claros.
- **Prohibido:** `text-white` sobre `brand-50`, `brand-100`, `brand-200`, `brand-300`, `brand-400`, `brand-500` o `brand-600` en texto normal.

---

## 4. Contraste WCAG AA

Ratios calculados con la fórmula de luminancia relativa de WCAG 2.x. Se considera texto normal el que mide menos de 24 px, o menos de 18.66 px en negrita.

| Primer plano | Fondo | Ratio | ¿AA texto normal? | Uso permitido |
|---|---|---|---|---|
| Blanco | `#0cb7f2` (500) | **2.32** | ❌ No | Solo texto grande/decorativo; preferir texto oscuro |
| Blanco | `#7cdaf9` (200) | **1.58** | ❌ No | Nunca |
| Blanco | `#b6ffff` (100) | **1.12** | ❌ No | Nunca |
| `#052e3d` (950) | `#0cb7f2` (500) | **6.19** | ✅ Sí | Botón primario |
| `#052e3d` (950) | `#25c5f5` (400) | **7.09** | ✅ Sí | Hover primario |
| `#052e3d` (950) | `#7cdaf9` (200) | **9.05** | ✅ Sí | Chips/acentos con texto |
| `#052e3d` (950) | `#b6ffff` (100) | **12.79** | ✅ Sí | Badges pálidos |
| `#077093` (700) | Blanco | **5.60** | ✅ Sí | Enlaces y texto de marca |
| `#077093` (700) | `#eefcff` (50) | **5.34** | ✅ Sí | Badge informativo |
| `#0a5874` (800) | `#b6ffff` (100) | **7.04** | ✅ Sí | Badge informativo fuerte |
| `#0891c4` (600) | Blanco | **3.59** | ⚠️ Solo texto grande o UI no textual | Iconos, bordes, indicadores |
| Blanco | `#077093` (700) | **5.60** | ✅ Sí | Fondo de marca con texto blanco |
| Blanco | `#0a5874` (800) | **7.89** | ✅ Sí | Fondo de marca oscuro |
| Blanco | `#047857` (emerald-700) | **5.48** | ✅ Sí | Botón de éxito |
| Blanco | `#b45309` (amber-700) | **5.02** | ✅ Sí | Botón de advertencia |
| Blanco | `#e11d48` (rose-600) | **4.70** | ✅ Sí | Botón de peligro |

### Reglas derivadas de la auditoría

1. **El primario es celeste brillante con texto oscuro.** `bg-brand-500` + `text-brand-950`; hover `bg-brand-400`. Esto cumple 6.19:1 y conserva la identidad celeste.
2. **El blanco solo se permite desde `brand-700`.** Si se necesita un fondo de marca con texto blanco, usar `bg-brand-700` (5.60:1) u `bg-brand-800` (7.89:1).
3. **`brand-600` no lleva texto normal.** Se reserva para iconos, bordes e indicadores no textuales (3.59:1 supera el 3:1 exigido a componentes de UI).
4. **Los badges no usan blanco.** Fórmula estándar: `bg-brand-100` + `text-brand-700`/`800` + `border-brand-200`.
5. **Estados:** `emerald-700`, `amber-700` y `rose-600` son los mínimos para botones sólidos con texto blanco.
6. Se auditaron y corrigieron todos los `bg-brand-* text-white` existentes en `client/src`. No deben reintroducirse.

---

## 5. Tipografía, espaciado, radios, sombras y breakpoints

### Tipografía

- Familia: `Inter, system-ui, Avenir, Helvetica, Arial, sans-serif` (definida en `index.css`).
- Escala: `text-xs` 12 px · `text-sm` 14 px · `text-base` 16 px · `text-lg` 18 px · `text-xl` 20 px · `text-2xl` 24 px · `text-3xl` 30 px.
- Pesos: 400 cuerpo, 500/600 controles, 700/800/900 títulos y totales.
- En móvil, el cuerpo mínimo es 14 px. Los textos de 10–12 px se reservan para badges y metadatos cortos.
- No usar párrafos justificados ni tracking negativo por debajo de `text-lg`.

### Espaciado

- Base de 4 px (`1` = 4 px).
- Padding de página: 16 px móvil (`p-4`), 24 px tablet (`sm:p-6`), 32 px desktop (`lg:p-8`).
- Padding de tarjeta: 16–20 px móvil, 20–24 px desktop.
- Separación entre secciones: `space-y-4` a `space-y-6`.
- Separación entre acciones: mínimo `gap-2` (8 px).

### Radios

- Inputs y botones compactos: `rounded-lg` (10 px).
- Botones principales y tarjetas: `rounded-xl` / `rounded-2xl` (12–16 px).
- Modales y contenedores destacados: `rounded-2xl` / `rounded-3xl`.
- Pills y badges: `rounded-full`.

### Sombras

- Tarjetas: `shadow-sm`.
- Elementos elevados o hover: `shadow-md`.
- Modales: `shadow-xl` / `shadow-2xl`.
- No acumular sombras dentro de una misma tarjeta.

### Breakpoints de validación

| Ancho | Escenario | Qué revisar |
|---|---|---|
| **360 × 640** | Móvil mínimo | Sin scroll horizontal; tabs con scroll interno; acciones a ancho completo |
| **390 × 844** | Móvil común (iPhone) | Igual que 360, con más aire |
| **768** | Tablet | Sidebar/admin, grids de 2 columnas |
| **1024** | Desktop chico | Sidebar fija, tablas completas |
| **1280** | Desktop amplio | Contenedores `max-w-*` centrados |

Tailwind mantiene sus breakpoints (`sm` 640, `md` 768, `lg` 1024, `xl` 1280). Los anchos 360 y 390 son de **auditoría**, no nuevos breakpoints.

### Touch y safe area

- Todo control interactivo: `min-h-[44px]`, y `min-w-[44px]` si es solo icono.
- Separación táctil mínima: 8 px.
- Botones de formularios y acciones principales: `w-full` en móvil, `sm:w-auto` desde tablet.
- Elementos fijos inferiores pueden usar `safe-bottom` (`env(safe-area-inset-bottom)`).

---

## 6. Patrones de componentes

### 6.1 Botones

| Variante | Clases base | Ejemplo |
|---|---|---|
| Primario | `bg-brand-500 hover:bg-brand-400 text-brand-950 font-semibold rounded-lg min-h-[44px]` | Enviar, guardar, publicar |
| Secundario | `bg-brand-50 hover:bg-brand-100 text-brand-700 border border-brand-200 rounded-lg min-h-[44px]` | Reponer, volver, acciones de apoyo |
| Neutro | `bg-white hover:bg-slate-50 text-slate-700 border border-slate-200 rounded-lg min-h-[44px]` | Cancelar, cerrar panel |
| Éxito | `bg-emerald-700 hover:bg-emerald-800 text-white rounded-lg min-h-[44px]` | Marcar listo, entregar |
| Advertencia | `bg-amber-700 hover:bg-amber-800 text-white rounded-lg min-h-[44px]` | Cierre manual |
| Peligro | `bg-rose-600 hover:bg-rose-700 text-white rounded-lg min-h-[44px]` | Cancelar comanda, cerrar turno |

Reglas: `disabled:opacity-50 disabled:cursor-not-allowed`, foco visible con `focus-visible:ring-2 focus-visible:ring-brand-700 focus-visible:ring-offset-2`, icono + texto con `gap-1.5/2`, sin texto blanco sobre celeste claro.

### 6.2 Tabs y chips

Componente canónico: `client/src/components/menu/MenuSectionTabs.tsx`.

- Regla: si el tab bar tiene **5 opciones fijas o menos**, hacer wrap en móvil (dos filas como máximo); no usar scroll horizontal. Si tiene muchas opciones, usar scroll interno.
- Contenedor (pocas opciones): `w-full min-w-0 max-w-full flex flex-wrap gap-2`.
- Botón: `shrink-0 whitespace-nowrap min-h-[44px] px-4 rounded-xl border text-sm font-semibold`.
- Activo: `bg-brand-500 text-brand-950 border-brand-500 shadow-sm`.
- Inactivo: `bg-white text-slate-700 border-slate-200 hover:bg-slate-50`.
- Estado accesible: `aria-pressed={isActive}`.
- Desde `sm`: el componente ocupa `sm:flex-1` y mantiene una sola fila si cabe; si no, envuelve.
- Scroll horizontal (solo tab bars extensas): `flex-nowrap overflow-x-auto scrollbar-none snap-x` + botones `shrink-0 snap-start`; añadir affordance de degradado lateral con `pointer-events-none`.
- El padre debe ser `flex flex-col sm:flex-row min-w-0`; el botón de acción hermano debe ser `shrink-0`.

### 6.3 Cards

`bg-white rounded-2xl border border-slate-200 shadow-sm p-4 sm:p-6`. Cabecera con título `text-base font-bold text-slate-900` y metadatos `text-xs text-slate-500`. Las cards de estado pueden teñirse con `bg-emerald-50/20`, `bg-rose-50/20` o `bg-brand-50/20`, manteniendo el borde semántico.

### 6.4 Formularios

- Label: `block text-sm font-medium text-slate-700` (o `text-xs font-semibold uppercase tracking-wider text-slate-500` en filtros).
- Input/select/textarea: `w-full px-3 py-2 border border-slate-300 rounded-lg min-h-[44px] bg-white`.
- Foco: `focus:outline-none focus:ring-1 focus:ring-brand-600` (o `focus:ring-2` en modales).
- Error: texto `text-rose-700`, borde `border-rose-300`, fondo `bg-rose-50`.
- En móvil, los campos y el botón de envío ocupan todo el ancho.

### 6.5 Badges

- Informativo: `bg-brand-100 text-brand-800 border border-brand-200`.
- Éxito: `bg-emerald-50 text-emerald-800 border border-emerald-300`.
- Advertencia: `bg-amber-50 text-amber-800 border border-amber-300`.
- Peligro: `bg-rose-50 text-rose-700 border border-rose-300`.
- Neutro: `bg-slate-100 text-slate-700 border border-slate-200`.
- Forma: `inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-semibold`.
- Nunca usar `text-white` en badges de fondo claro.

### 6.6 Modales

- Backdrop: `fixed inset-0 bg-slate-900/60 z-50 flex items-center justify-center p-4`.
- Panel: `bg-white rounded-2xl w-full max-w-md/lg p-6 shadow-xl border border-slate-200`.
- **Obligatorio:** `max-h-[90dvh] overflow-y-auto` en el panel para que el scroll ocurra dentro del modal.
- En móvil, los selectores largos pueden presentarse como bottom sheet (`items-end sm:items-center`, `rounded-t-3xl sm:rounded-2xl`).
- Título `text-lg font-bold text-slate-900`; acciones al pie con `flex flex-col-reverse sm:flex-row sm:justify-end gap-2`.
- Cerrar con botón explícito de 44 px y `aria-label`.

### 6.7 Tablas

- Siempre dentro de `div.overflow-x-auto.rounded-xl.border.border-slate-200.bg-white`.
- `table.w-full.text-left.text-sm`; cabecera `text-xs uppercase tracking-wider text-slate-500`.
- En móvil, la tabla puede desplazarse horizontalmente dentro de su contenedor, nunca la página.
- Para tablas muy anchas, añadir `min-w-[640px]` a la tabla.

### 6.8 Navegación móvil y jerarquía del Encargado

- Sidebar admin: drawer fijo de 288 px (`w-72`), backdrop `bg-slate-900/50`, cierre explícito de 44 px; en `md+` pasa a columna estática.
- Header móvil: hamburguesa de 44 px, título truncado (`truncate`) y contenedor `min-w-0`.
- Headers de operación (mozo/cocina): `flex flex-wrap justify-between items-center gap-2`; acciones con `shrink-0`.
- Tab bars de sección (cierre de turno, zonas de mesas): scroll horizontal con `scrollbar-none snap-x` y botones `shrink-0 snap-start`.
- **Jerarquía del Encargado (spec 008):** la navegación principal tiene solo tres apartados:
  1. **Platos y raciones** → `/encargado` (pantalla inicial tras login).
  2. **Turno** → `/encargado/turnos`.
  3. **Cierre de turno** → `/encargado/cierre`.
- **Catálogo maestro** (`/encargado/platos`) es un enlace secundario dentro de “Platos y raciones”; no ocupa un ítem de la navegación principal.
- Las rutas antiguas `/encargado/menu` y `/encargado/raciones` redirigen a `/encargado`.

### 6.9 Tarjeta de plato, stepper de raciones y panel de edición

Patrón canónico: `client/src/components/menu/DishRationCard.tsx`, `DishEditSheet.tsx`, `CategoryTabs.tsx` y `ShiftBanner.tsx`. Base de la pantalla: `pages/encargado/DishManagementPage.tsx`.

**Tarjeta de plato (`DishRationCard`)**

- Contenedor: `bg-white rounded-2xl border border-slate-200 shadow-sm p-4 sm:p-5`, con `min-w-0` y `break-words` en el nombre. Nunca truncar el nombre del plato.
- Contenido clicable: un `button` que ocupa el ancho (`w-full text-left`) y abre el panel de edición. Los controles de acción viven fuera de ese botón para evitar botones anidados.
- Cabecera: badge de categoría (`bg-slate-100`, texto 11 px, mayúsculas), nombre (`text-base font-bold`), precio (`text-brand-700 font-black`).
- Descripción breve: `text-xs text-slate-500 line-clamp-2`.
- Estados: `AvailabilityBadge` (Disponible, Últimas raciones, Agotado, Inactivo) y `RationBadge` (`restantes / total` o “Sin límite”).
- En estado publicado, ajustar raciones o disponibilidad no requiere confirmar la tarjeta; se guarda al instante.

**Stepper de raciones**

- Solo aparece si el plato tiene límite de raciones.
- Botones `-` y `+` de **44 × 44 px** (`min-h-[44px] min-w-[44px]`), `rounded-xl`, borde neutro y foco visible.
- El valor central usa `text-sm font-black` y `aria-live="polite"`.
- `-` se deshabilita en 0; `+` se deshabilita al alcanzar el total preparado.
- Para platos sin límite se muestra “Sin límite” y se invita a definir el límite desde el panel de edición.

**Panel de edición (`DishEditSheet`)**

- Móvil: bottom sheet (`items-end`, `rounded-t-3xl`, `max-h-[92dvh]`, scroll interno). Desktop: modal centrado (`sm:items-center`, `sm:rounded-2xl`).
- Cabecera con categoría, badge de estado, nombre completo y botón de cierre de 44 px.
- En **borrador** permite editar: activo para venta, precio, límite, total, restantes y motivo de ajuste. Acciones: “Registrar ajuste ahora” (auditoría con `adjust`), “Quitar del menú” y “Guardar cambios” (`updateDay`).
- En **publicado** muestra los valores en modo lectura y permite ajustar el conteo con motivo, marcar agotado o reactivar. Avisa que agregar/quitar platos exige un borrador.
- “Historial de ajustes” abre `AdjustmentHistory`.
- Toda acción destructiva se confirma y todo error se muestra con `role="alert"`.

**Chips de categoría (`CategoryTabs`)**

- Cinco opciones fijas: Todos, Criollo, Entrada, Marino, Bebidas.
- Wrap obligatorio (`flex flex-wrap gap-2`), nunca scroll horizontal; a 360 px se aceptan dos filas.
- Botón: `min-h-[44px]`, `rounded-xl`, `text-xs sm:text-sm`, conteo en pill.
- Activo: `bg-brand-500 text-brand-950 border-brand-500`; inactivo: `bg-white text-slate-700 border-slate-200`.
- `aria-pressed` en cada botón y `aria-label` en el grupo.

**Cabecera de la pantalla**

- Una tarjeta compacta con fecha, estado del menú, estado del turno con enlace a `/encargado/turnos` y acciones “Guardar” / “Publicar” solo en borrador.
- Si no hay turno abierto: `ShiftBanner` ámbar con enlace “Abrir turno”.
- En móvil los bloques se apilan y las acciones ocupan el ancho disponible; no se usa scroll horizontal de página.

---

## 7. Reglas responsive

1. Ancho mínimo soportado: **360 px**. El layout debe funcionar sin reglas de ancho fijo que fuercen scroll.
2. Sin scroll horizontal de página. Todo desborde se contiene en un wrapper con `overflow-x-auto` (tabs, tablas) o se resuelve con `min-w-0` + `flex-wrap`.
3. En flex rows, el hijo que contiene texto o tabs lleva `min-w-0`; el botón de acción lleva `shrink-0`.
4. Nunca usar anchos fijos mayores al viewport. Preferir `w-full sm:w-auto`, `max-w-full`, `flex-1`.
5. Tablas: wrapper con scroll propio; no reducir la tipografía por debajo de 12 px.
6. Modales: `max-h-[90dvh]` + scroll interno; el backdrop no debe crecer con el contenido.
7. Tab bars: si son pocas opciones fijas (5 o menos), envolver en dos filas con `flex-wrap`; si son muchas, usar `overflow-x-auto` + `scrollbar-none` + `snap-x` con affordance. Nunca cortar el texto de una opción.
8. Botones de formulario y acciones primarias: `w-full sm:w-auto`; en móvil, apilados con `gap-2`.
9. Imágenes y gráficos: `max-w-full h-auto`.
10. Validar visualmente a 360 × 640 y 390 × 844 antes de cerrar cualquier cambio.

---

## 8. Accesibilidad

- **Foco visible:** `ring-brand-700` + `ring-offset-2` o el `:focus-visible` global definido en `index.css`.
- **Tabs/chips:** `aria-pressed` en cada botón; `aria-label` en el grupo o tablist.
- **Iconos solos:** `aria-label` y `title`; iconos decorativos con `aria-hidden="true"`.
- **Modales:** `role="dialog"`, `aria-modal="true"`, `aria-labelledby`; cierre con Escape cuando el componente lo soporte.
- **Mensajes dinámicos:** `role="status"` / `aria-live="polite"` para toasts y confirmaciones; `role="alert"` para errores.
- **Movimiento:** respetar `prefers-reduced-motion`; la regla global reduce animaciones y transiciones.
- **Contraste:** cumplir la tabla de la sección 4. No introducir combinaciones nuevas sin calcular el ratio.
- **Touch:** mínimo 44 × 44 px; no depender solo del color para comunicar estado (usar texto/ícono).

---

## 9. Registro de decisión y relación con la spec 007

**Contexto:** el bug reportado en Encargado → Menú del día (`/encargado/menu`) mostraba los tabs “Todos los platos / Criollo / Marino / A la carta” desbordando la tarjeta en móvil. La causa era un flex item sin `min-w-0`/`w-full` cuyo contenido empujaba el ancho de la página. Además, la paleta `brand` anterior era la escala sky genérica (`#0ea5e9`/`#0284c7`) y algunos botones usaban texto blanco sobre celestes claros.

**Decisión:**

1. Adoptar la escala `brand` 50–950 con anclas `#b6ffff`, `#7cdaf9` y `#0cb7f2` como única fuente de verdad en `tailwind.config.js`.
2. Usar `brand-500` + `text-brand-950` para acciones primarias y reservar el blanco para fondos `brand-700` o más oscuros.
3. Corregir los tabs en el componente compartido `MenuSectionTabs`, de modo que `MenuDayPage`, `MenuView` y `NewOrderPage` hereden el comportamiento.
4. Aplicar wrap en tab bars con 5 opciones o menos y scroll horizontal solo en tab bars extensas; auditar tablas/modales/headers en móvil.

**Alternativas descartadas:** ocultar el overflow con `overflow-hidden` en la página (oculta el problema, no lo resuelve); usar `brand-600` con texto blanco (3.59:1, no cumple AA para texto normal); mantener la paleta sky anterior (no responde a la identidad celeste solicitada).

**Consecuencias:** los botones primarios son visualmente celestes con texto oscuro; los enlaces y textos de marca usan `brand-700`; los estados conservan sus colores semánticos. Cualquier nuevo componente debe reutilizar estos tokens y patrones.

**Refinamiento posterior (2026-11-21):** se validó la captura a 360 px y se ajustó el patrón: los 5 tabs fijos de `/encargado/menu` y `/mozo/menu` ahora hacen wrap en dos filas dentro de la tarjeta (sin scroll ni texto cortado), y el header de `AdminLayout` muestra “Cevichería Lucy” completo en móvil, con el sufijo de sección como subtítulo. El scroll horizontal queda reservado para tab bars con muchas opciones.

**Relación con la spec 007:** `specs/007-despliegue-web-responsive/spec.md` establece la aplicación web responsive como criterio de aceptación. Este documento operacionaliza esos requisitos para la capa visual y es la referencia canónica para futuras revisiones de UI.

**Relación con la spec 008 (2026-11-21):** `specs/008-gestion-platos-raciones/spec.md` redefine la pantalla inicial del ENCARGADO como “Platos y raciones”. Este documento fija el patrón visual de tarjeta + stepper + panel de edición y la jerarquía de navegación de tres apartados. La regla mobile-first se mantiene: validar a 360 × 640 y 390 × 844 antes de cerrar cualquier cambio.

**Guía para futuros cambios:**

1. Leer este documento antes de tocar estilos.
2. Usar tokens `brand-*`, nunca hex sueltos.
3. Reutilizar `MenuSectionTabs` para filtros de sección.
4. Verificar contraste si se introduce una combinación nueva.
5. Probar a 360 y 390 px, y confirmar que la página no scrollea horizontalmente.
6. Ejecutar `npm run typecheck`, `npm run lint` y `npm test` antes de cerrar el cambio.

---

## 10. Checklist de verificación

- [ ] La página no tiene scroll horizontal a 360 × 640 ni a 390 × 844.
- [ ] Los tabs con 5 opciones o menos se ven completos (wrap en móvil) sin scroll horizontal; las tab bars extensas hacen scroll interno con affordance.
- [ ] Los botones primarios usan `bg-brand-500` + `text-brand-950`.
- [ ] Ningún `text-white` está sobre `brand-*` claro.
- [ ] Enlaces y textos de marca usan `text-brand-700` o más oscuro.
- [ ] Tablas dentro de `overflow-x-auto`.
- [ ] Modales y bottom sheets con `max-h-[90dvh]` / `max-h-[92dvh]` y scroll interno.
- [ ] Controles interactivos de al menos 44 px.
- [ ] Stepper de raciones (`-` / `+`) usable a 360 px y sin abrir el panel por accidente.
- [ ] Chips de categoría con wrap; cinco categorías visibles en máximo dos filas.
- [ ] Navegación del Encargado con solo tres ítems; catálogo maestro como enlace secundario.
- [ ] Foco visible en teclado.
- [ ] `npm run typecheck`, `npm run lint` y `npm test` en verde.

---

## 11. Verificación de la revisión inicial y refinamiento (2026-11-21)

- `npm run typecheck`: OK en `shared`, `server` y `client`.
- `npm run lint`: 0 errores y 2 warnings preexistentes en `WaiterDashboardPage.tsx` (`user` y `logout` sin uso).
- `npm test`: servidor 15 archivos / 108 tests; cliente 24 archivos / 104 tests; todo en verde.
- Auditoría automatizada con Chrome headless (CDP) a **360 × 640** y **390 × 844** en `/login`, `/encargado`, `/encargado/menu`, `/encargado/platos`, `/encargado/raciones`, `/encargado/cierre`, `/mozo`, `/mozo/pedido/nuevo`, `/mozo/pedidos`, `/mozo/menu` y `/cocina`: **0 overflow horizontal de página** (`scrollWidth == clientWidth`).
- Tabs de `/encargado/menu` a 360 px: 5 opciones visibles en 2 filas, contenedor de 294 px dentro de una tarjeta de 328 px, `flex-wrap: wrap`, `overflow-x: visible` y `scrollWidth == clientWidth`; la página permanece en 360 px. En `/mozo/menu` ocurre lo mismo con las secciones activas de la demo.
- Header de `AdminLayout` a 360 y 390 px: “Cevichería Lucy” completo, sin truncado, y el sufijo de sección como subtítulo (“Administración”, “Cierre de Turno”). `MenuView` aplica el mismo criterio en `/mozo/menu`.

---

## 12. Verificación de la spec 008 (2026-11-21)

- `npm run typecheck`: OK en `shared`, `server` y `client`.
- `npm run lint`: 0 errores; se mantienen los 2 warnings preexistentes de `WaiterDashboardPage.tsx`.
- `npm test`: servidor 15 archivos / 108 tests; cliente 27 archivos / 113 tests; todo en verde.
- Auditoría con Chrome headless (CDP) en `/encargado`:
  - **360 × 640:** `scrollWidth == clientWidth == 360`, sin overflow horizontal; 5 chips en 2 filas; 26 tarjetas; nombres sin truncar; stepper de 44 × 44 px.
  - **390 × 844:** `scrollWidth == clientWidth == 390`, sin overflow horizontal; 5 chips en 2 filas; 26 tarjetas; stepper de 44 × 44 px.
  - Rutas antiguas: `/encargado/menu` y `/encargado/raciones` terminan en `/encargado`; `/encargado/turnos` muestra el control de turnos.
- `/mozo/menu` con las 4 secciones activas (Criollo, Entrada, Marino, Bebidas), sin `A_LA_CARTA` y sin overflow horizontal; el filtro Criollo muestra 11 platos.
- Flujo de borrador verificado en navegador: CTA sin menú → crear borrador → catálogo de 26 platos activos → agregar plato → editar precio a S/ 23.50 y límite 10/10 → guardar → quitar del menú. Los datos de prueba se limpiaron después.
