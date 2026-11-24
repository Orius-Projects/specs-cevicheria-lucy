# Spec 009 — Experiencia de Salón, Gestión de Mesas y Tema Marino Dual

**Feature:** `009-ux-mesas-temas`  
**Prioridad:** P1  
**Estado:** implementada  
**Fecha:** 2026-11-23  
**Actores:** ENCARGADO (gestión de mesas y menú), MOZO (toma de pedidos aislada y navegación táctil), COCINERO (KDS de alto contraste)  
**Depende de:** 001, 002, 003, 004, 007, 008  
**Design system:** [`docs/design-system.md`](../../docs/design-system.md)

---

## 1. Problema y Diagnóstico Operativo

Durante las pruebas operativas en dispositivos móviles y tablets de salón y cocina se detectaron los siguientes puntos críticos de fricción:

1. **Compartición accidental de pedidos entre mesas:**
   El carrito de compras del mozo almacenaba los ítems en un estado global unificado sin asociar el borrador a la mesa actual. Si un mozo agregaba platos para la Mesa 1, luego cambiaba a la Mesa 2 para atender a otro cliente antes de confirmar, los ítems de la Mesa 1 se mezclaban con los de la Mesa 2, generando comandas erróneas en cocina y confusiones graves.
2. **Fricción ergonómica al tomar pedidos en smartphones:**
   Para revisar los platos agregados y enviar la comanda, el mozo debía hacer scroll vertical prolongado hasta el fondo de la pantalla, perdiendo tiempo y visibilidad de las categorías del menú.
3. **Rigidez en la distribución de mesas:**
   El encargado no contaba con una interfaz para agregar nuevas mesas, editar capacidades o cambiar su ubicación física (Primer Piso, Segundo Piso, Afuera del Local), dependiendo de migraciones manuales en base de datos.
4. **Navegación restringida en el selector de zonas:**
   Las pestañas de zonas del local en móviles no permitían arrastre táctil fluido (drag-to-scroll), dificultando el cambio ágil de ambientes.
5. **Envíos múltiples accidentales a cocina:**
   Al enviar un pedido, la pantalla de orden no se cerraba de inmediato ni deshabilitaba el botón de confirmación, permitiendo que un doble tap enviara 2 o 3 comandas idénticas a cocina.
6. **Fatiga visual y contraste deficiente:**
   La paleta clara original resultaba agotadora en turnos continuos bajo iluminación de salón y cocina. Además, en pantallas oscuras existían textos en tonos carbón (`slate-800`) sobre fondos negros, o textos claros sobre fondos blancos en formularios de login, incumpliendo los criterios WCAG AA.
7. **Inaccesibilidad de ayuda y gestión de perfil:**
   Los botones de campana, ayuda interrogativa (`?`) y perfil de usuario no contaban con modales funcionales ni feedback interactivo.

---

## 2. Objetivos y Alcance

### 2.1 Objetivos

- **Aislamiento estricto de borradores por mesa:** Cada mesa mantiene su propio borrador en `localStorage` con la clave `cevicheria_draft_<tableCode>`, asegurando independencia total al alternar entre mesas.
- **Carrito flotante inferior (Bottom Sheet) para móviles:** Mostrar una barra inferior interactiva con resumen de platos y monto total, expandible mediante deslizamiento vertical para verificar y enviar comandas sin scroll manual.
- **Módulo de gestión de mesas para el Encargado (`/encargado/mesas`):** Permitir crear, editar (código, capacidad, zona), reactivar y deshabilitar mesas del salón en tiempo real.
- **Carrusel táctil de zonas con soporte de arrastre:** Permitir deslizar horizontalmente las zonas del local (`Todo el local`, `Primer Piso`, `Segundo Piso`, `Afuera del local`) con gestos táctiles y ratón sin barras de scroll invasivas.
- **Protección contra envíos duplicados:** Bloqueo instantáneo del botón de envío, asignación de `idempotencyKey` única y redirección inmediata al salón tras confirmación exitosa.
- **Paleta marina dual ("Brisa Marina" y "Mar Profundo"):** Sistema de temas conmutables con turquesas oceánicos (`brand`) y azul marino abisal (`marine`), garantizando contraste WCAG AA en todas las vistas (Login, Mozo, Cocina y Encargado).
- **Centro de ayuda interactivo y menú de perfil:** Componentes modales con guías operativas según rol y menú de usuario con cambio de tema y cierre de sesión.

### 2.2 No Objetivos

- No se integran pasarelas de pago ni facturación electrónica SUNAT (inviolable según constitución).
- No se introducen estados de pedido intermedios no autorizados (se preservan estrictamente `ENVIADO` y `LISTO`).
- No se agregan temporizadores restrictivos en cocina.

---

## 3. Especificación Técnica de Componentes

### 3.1 Aislamiento de Borradores de Carrito (`cartStore.ts`)

```typescript
// Almacenamiento aislado por código de mesa
interface CartDraft {
  items: CartItem[];
  notes?: string;
  updatedAt: string;
}

// Al cambiar de mesa, el store recupera el borrador específico:
getDraftForTable(tableCode: string): CartDraft;
saveDraftForTable(tableCode: string, draft: CartDraft): void;
clearDraftForTable(tableCode: string): void;
```

### 3.2 Carrito Flotante Inferior (`OrderCart.tsx`)

- **Modo Colapsado (Mobile):** Barra inferior fija sobre el viewport (`fixed bottom-0 left-0 right-0 z-30`) que exhibe la cantidad de raciones seleccionadas, el subtotal en soles y el botón de despliegue.
- **Modo Expandido (Bottom Sheet):** Panel deslizable hacia arriba con animación fluida `transform translate-y-0 transition-transform duration-300`, listado de platos con steppers `+` / `-`, notas por ítem, nota general de comanda y botón de confirmación destacado.

### 3.3 Gestión de Mesas (`TablesManagementPage.tsx`)

- **Ruta:** `/encargado/mesas`
- **Operaciones:**
  - Crear nueva mesa: Código único (ej. `M13`), capacidad en personas (1–20), zona asignada (`Primer Piso`, `Segundo Piso`, `Afuera del Local`).
  - Editar mesa: Modificación de zona o capacidad.
  - Alternar estado: Inactivar mesa temporalmente o marcar disponible.
- **Reglas de seguridad:** No se permite eliminar mesas que cuenten con pedidos abiertos en el turno actual.

### 3.4 Paleta Marina Dual y Alto Contraste

| Token | Hex | Rol en Brisa Marina | Rol en Mar Profundo |
|---|---|---|---|
| `brand-500` | `#06b6d4` | Botón primario y foco de atención | Botón primario y foco de atención |
| `marine-950` | `#030811` | N/A | Fondo principal de página (`body`) |
| `marine-900` | `#06101e` | N/A | Superficie de tarjetas y encabezados |
| `marine-800` | `#0b192c` | N/A | Bordes estructurales y contenedores internos |
| `slate-100` | `#f1f5f9` | N/A | Texto principal de alta legibilidad en modo oscuro |

- **Login de Alto Contraste:** Los campos de texto declaran explícitamente `text-slate-900 dark:text-white bg-white dark:bg-marine-950 border-slate-300 dark:border-marine-700` para garantizar perfecta legibilidad sin importar el esquema del sistema operativo.
- **Cocina (KDS) de Alto Contraste:** Títulos de columnas en `dark:text-slate-100 font-bold`, badges de estado legibles y tarjetas de estado vacío integradas armónicamente al modo oscuro.

---

## 4. Criterios de Aceptación y Verificación

1. **Aislamiento de mesas:** Un mozo puede agregar 2 ceviches a la Mesa 01, cambiar a la Mesa 02 y verificar que el carrito de la Mesa 02 esté vacío, y luego volver a la Mesa 01 encontrando sus 2 ceviches intactos.
2. **Sin duplicados en cocina:** Al presionar "Enviar Pedido", el botón se bloquea inmediatamente y la pantalla navega al salón principal; en el tablero de cocina solo aparece 1 orden con su correspondiente ID.
3. **Persistencia del tema:** El cambio de tema ("Brisa Marina" / "Mar Profundo") persiste entre recargas de página mediante `localStorage` y se sincroniza en todas las vistas.
4. **Legibilidad WCAG AA:** Todos los inputs de formulario y textos de cocina cumplen una relación de contraste superior a 4.5:1 en ambos temas.
5. **Cero regresiones:** Las 108 pruebas del servidor y las 126 pruebas del cliente pasan satisfactoriamente.
