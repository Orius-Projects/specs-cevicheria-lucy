# Cevichería Lucy — Especificaciones SDD v2

Repositorio de especificaciones del sistema web responsive de pedidos y cocina de la Cevichería Lucy, organizado con **Spec-Driven Development (SDD)**.

Esta versión se reconstruyó a partir de:

> **`docs/sources/DOCUMENTO_PROCESOS_CEVICHERIA_LUCY.pdf`** — Documento de procesos, versión 1.0, noviembre 2026 (15 páginas).

Los mockups y los documentos `00/01/02` son material externo/histórico y **no forman parte de este repositorio**; los mockups son solo referenciales y no imponen el diseño ni el flujo.

---

## 1. Alcance

- Aplicación **web responsive** para móviles y tablets.
- Toma de pedidos por el mozo.
- Cola de cocina FIFO con un único botón: **“Marcar listo”**.
- Aviso visual y sonoro al mozo cuando un pedido está listo.
- Control de raciones y estados del plato (`Disponible`, `Últimas raciones`, `Agotado`).
- Cierre de turno con resumen de lo vendido, raciones, agotados y cancelaciones.
- **No** incluye pagos, facturación, SUNAT, IGV, costos de variaciones, delivery, inventario de almacén ni app nativa/APK.

---

## 2. SDD y estructura

| Pregunta | Artefacto | Ubicación |
|---|---|---|
| ¿Qué reglas son no negociables? | `constitution.md` | `.specify/memory/` |
| ¿Qué construir y por qué? | `spec.md` | `specs/NNN-feature/` |
| ¿Cómo construirlo? | `plan.md`, `data-model.md`, `contracts/` | `specs/NNN-feature/` |
| ¿En qué orden? | `tasks.md` | `specs/NNN-feature/` |

```text
Constitución
    ↓
spec.md  →  clarify  →  plan.md  →  tasks.md  →  analyze  →  implement  →  converge
```

```text
specs-cevicheria-lucy/
├── README.md                           # este documento
├── .gitignore
├── .specify/
│   ├── memory/constitution.md
│   └── templates/
├── docs/
│   ├── design-system.md                # paleta, componentes y reglas responsive
│   └── sources/
│       └── DOCUMENTO_PROCESOS_CEVICHERIA_LUCY.pdf
└── specs/
    ├── README.md
    ├── 001-plataforma-base/
    ├── 002-menu-raciones/
    ├── 003-toma-pedidos-mozo/
    ├── 004-cocina-pendientes/
    ├── 005-avisos-tiempo-real/
    ├── 006-cierre-turno/
    ├── 007-despliegue-web-responsive/
    └── 008-gestion-platos-raciones/
```

### 2.1 Diseño de interfaz

Las decisiones de paleta celeste, componentes, comportamiento responsive y accesibilidad están en **[docs/design-system.md](docs/design-system.md)**. Es la referencia obligatoria para cualquier cambio de UI y complementa los criterios de la spec 007.

---

## 3. Specs

| # | Spec | Procesos | Prioridad | Estado |
|---|---|---|---|---|
| 001 | [Plataforma base](specs/001-plataforma-base/spec.md) | Actores, estados, flujo | P1 | Borrador |
| 002 | [Menú, raciones y agotados](specs/002-menu-raciones/spec.md) | P-01, P-05 | P1 | Borrador |
| 003 | [Toma de pedidos — Mozo](specs/003-toma-pedidos-mozo/spec.md) | P-02, P-04 | P1 | Borrador |
| 004 | [Cocina — Pendientes y listos](specs/004-cocina-pendientes/spec.md) | P-03, P-04 | P1 | Borrador |
| 005 | [Avisos en tiempo real](specs/005-avisos-tiempo-real/spec.md) | P-04, P-05 | P1 | Borrador |
| 006 | [Cierre de turno](specs/006-cierre-turno/spec.md) | P-06 | P1 | Borrador |
| 007 | [Despliegue web responsive](specs/007-despliegue-web-responsive/spec.md) | Web | P1 | Borrador |
| 008 | [Gestión de platos y raciones](specs/008-gestion-platos-raciones/spec.md) | P-01, P-05 | P1 | Implementada (demo) |

---

## 4. Stack

| Capa | Tecnología |
|---|---|
| Base de datos | MongoDB + Mongoose |
| Backend | Node.js + Express + Socket.IO + CORS |
| Frontend | React + Vite + Tailwind CSS + React Router + Zustand + Axios |
| Tiempo real | Socket.IO Client / Server |
| PWA | `vite-plugin-pwa` (opcional) |
| Lenguaje | TypeScript |
| Autenticación | JWT + bcrypt |
| Pruebas | Vitest, Supertest, React Testing Library, Playwright |

---

## 5. Actores y estados

### Roles

| Rol | Responsabilidad |
|---|---|
| **Mozo** | Registra, envía, entrega y cancela pedidos; repone raciones tras confirmar con cocina; puede cerrar mesas. |
| **Cocinero** | Ve la cola FIFO, revisa el detalle y marca “Listo”. No modifica stock ni disponibilidad. |
| **Encargado** | Configura menú, precios, variaciones y raciones; marca agotado; ajusta stock con motivo; cierra turno. |

### Pedido

`BORRADOR → ENVIADO → LISTO → ENTREGADO`

Cancelaciones: desde `BORRADOR`, `ENVIADO` o `LISTO`, con motivo. En `LISTO` se advierte coordinar con cocina. No existe `EN_PREPARACION`.

### Plato

`INACTIVO → DISPONIBLE → ULTIMAS_RACIONES → AGOTADO`

Con 5 o menos raciones se muestra “Últimas raciones”; al llegar a 0, “Agotado”. La reposición la hace el mozo tras confirmar con cocina; el encargado marca agotado por falta de insumo o ajusta con motivo.

---

## 6. Glosario

| Término | Definición |
|---|---|
| **Pedido / comanda** | Conjunto de platos y bebidas de una mesa. |
| **Mozo** | Toma y entrega el pedido. |
| **Cocinero** | Prepara y marca “Listo”. |
| **Encargado** | Configura el menú, raciones y variaciones; supervisa y cierra el turno. |
| **Menú criollo** | Menú del día con entradas fijas y fondos rotativos. |
| **Menú marino** | Platos de pescados y mariscos estables. |
| **A la carta** | Platos individuales fuera del menú del día. |
| **Entrada** | Plato que se sirve antes del fondo; suele ser fijo. |
| **Fondo** | Plato principal del menú criollo; rota según el día. |
| **Ración** | Unidad de venta de un plato. |
| **Variación** | Cambio permitido en un plato; no tiene costo. |
| **Agotado** | Plato sin raciones o sin insumo; no se puede pedir. |
| **Últimas raciones** | Quedan entre 1 y 5 unidades de un plato con límite. |
| **FIFO** | Primero en llegar, primero en atenderse. |
| **Turno** | Servicio configurable (por ejemplo, almuerzo o cena) que el encargado abre y cierra. |

---

## 7. Convenciones

- Contenido en español; nombres de archivo SDD en inglés.
- Carpetas `NNN-kebab-case`.
- IDs estables: `US-###`, `FR-###`, `NFR-###`, `SC-###`, `EC-###`, `T###`.
- Dinero en céntimos enteros; sin IGV.
- Fechas ISO 8601 UTC; visualización `America/Lima`.
- No se citan los documentos históricos `00/01/02`.

---

## 8. Cómo continuar

1. Leer `.specify/memory/constitution.md`.
2. Revisar `specs/README.md` para estado, dependencias y trazabilidad.
3. Validar `specs/001-plataforma-base/spec.md`, `specs/001-plataforma-base/data-model.md` y `specs/001-plataforma-base/contracts/rest-api.md`.
4. Responder las preguntas abiertas de cada spec.
5. Implementar por fases: `001` → `002` → `003`/`004` → `005` → `006` → `007` → `008`.
6. Revisar la spec `008` para la experiencia del encargado, el catálogo semilla y el flujo de raciones.

### Próximo paso recomendado

Validar el modelo de datos y las reglas de raciones de `001` y `002` antes de iniciar la UI del mozo y la cocina.

---

## 9. Notas

- El repositorio contiene solo artefactos SDD; el código de implementación vive fuera.
- No se ha instalado la CLI de Spec Kit; la estructura se mantiene manualmente.
- Los nombres, precios, mesas y platos de los mockups son ilustrativos.
- El logo y el diseño final se validarán con el restaurante.
