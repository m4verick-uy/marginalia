# Documentación arquitectural — marginaLia

Este archivo se actualiza con cada cambio arquitectural, a cargo del Validator
al cerrar cada feature. Ver CLAUDE.md para la visión de producto y el modelo
de datos completo.

## Estado actual

Fase 1, MVP, Entrega 1. Features implementadas: Login Google + multiusuario,
Alta manual de libro + lista con filtros, Board Kanban sobre status, Torta
de distribución por área.

## Nota de alcance: portadas (`cover`)

El campo `cover` (URL de portada) existe en el form de alta/edición desde la
Entrega 1 (CLAUDE.md pide "todos los campos del modelo, todos editables"),
pero **no se renderiza en ningún lado todavía** — no hay `<img>` conectado a
ese campo. Es intencional, no un bug: CLAUDE.md escopea explícitamente la
"grilla tipo estantería" con portadas a la Entrega 2, junto con el
autocompletado de Google Books/OpenLibrary (que va a traer la URL de portada
automáticamente). Decisión confirmada con el Ingeniero Jefe el 2026-08-14:
no adelantar esto a la Entrega 1, y no agregar Firebase Storage / subida de
archivos — `cover` sigue siendo una URL, tal como especifica el modelo de
datos.

## Proyecto Firebase

marginaLia usa un **proyecto Firebase propio**, separado del de ReMynder —
decisión tomada el 2026-08-13 (ver docs/stories-login.md, sección Decisiones).
Auth Google + Firestore, sin infraestructura compartida entre productos.

Proyecto real: `marginalia-68e0a`. `firebaseConfig` en `web/index.html` tiene
los valores reales (2026-08-13). Google habilitado en Authentication,
Firestore Database creada, Security Rules de `users/{userId}/books/{bookId}`
aplicadas — **login verificado funcionando el 2026-08-14** tanto en
producción como en preview.

Authorized domains en Firebase:
- `marginalia-uy.vercel.app` (producción)
- `marginalia-git-develop-m4vericks-projects.vercel.app` (preview de `develop`
  — dominio estable por rama, no cambia en cada deploy)

Nota: cada deployment individual de Vercel tiene además su propia URL
efímera (`marginalia-<hash>-m4vericks-projects.vercel.app`) que no está
autorizada — el login solo funciona en los dos dominios estables de arriba.

## Infraestructura de hosting y Git

- Repo: `github.com/m4verick-uy/marginalia` (default branch: `produccion`)
- Ramas: `develop` (trabajo activo) y `produccion` (deploy a producción),
  ambas conectadas a Vercel — push a `produccion` despliega a producción,
  push a `develop` genera preview
- Proyecto Vercel: `m4vericks-projects/marginalia`
- URL de producción: **https://marginalia-uy.vercel.app** (alias manual;
  `marginalia.vercel.app` a secas ya estaba tomado por otro proyecto global)
- Vercel Deployment Protection (SSO) estaba habilitada por defecto en el
  proyecto nuevo y bloqueaba el acceso público — se desactivó
  (`vercel project protection disable marginalia --sso`) para que el login
  sea accesible por cualquier usuario, no solo por cuentas del team de Vercel

## Historial de cambios arquitecturales

### 2026-08-14 — Torta de distribución por área
- Tercera vista en `.view-switch` (Lista/Tablero/**Distribución**)
- Gráfico de torta en **CSS puro** (`conic-gradient` + círculo superpuesto
  para efecto donut, total de libros en el centro) — evaluado explícitamente
  no agregar ninguna librería de gráficos, según la regla de CLAUDE.md de
  sin dependencias innecesarias
- Reutiliza `areaColorIndex()` (ya existente) como única fuente de verdad de
  color por área — el color de un área es el mismo en Lista, Tablero y
  Distribución
- Leyenda reutiliza el mismo patrón visual `.badge.area-pN` + `.area-dot`
  que ya usan las tarjetas de libro
- `.filters` se oculta completo en esta vista (a diferencia de Tablero, que
  solo oculta el filtro de status) — la torta muestra siempre el 100% de
  los libros, sin filtrar
- Docs: docs/research-torta-areas.md, docs/stories-torta-areas.md,
  docs/spec-torta-areas.md, docs/test-report-torta-areas.md,
  docs/validation-torta-areas.md
- Pendiente: verificación manual en navegador (confirmar que los colores
  coinciden entre vistas) — no se hizo en esta sesión

### 2026-08-14 — Board Kanban sobre status
- Segunda vista de los libros (además de la Lista): selector `.view-switch`
  en el header, `viewMode` persistido en `localStorage`
- 4 columnas por `status` (`pendiente|leyendo|leido|abandonado`), portado del
  `.board` de ReMynder: drag-and-drop nativo con delegación de eventos vía
  `AbortController`, columna vacía siempre visible como zona de drop,
  carrusel mobile con `scroll-snap` + dots
- Adaptación respecto a ReMynder (3 estados lineales → 4 estados con
  bifurcación en "abandonado"): en vez de botones prev/next de un solo paso,
  cada tarjeta tiene un `<select>` que permite saltar a cualquiera de los 4
  estados — fallback completo sin drag, no una versión reducida
- El filtro de status se oculta en la vista Tablero (es redundante con las
  columnas); el filtro de área funciona igual en ambas vistas
- `moveBook(id, status)` como wrapper sobre `updateBook`, sin escritura si el
  status no cambia
- Docs: docs/research-kanban.md, docs/stories-kanban.md, docs/spec-kanban.md,
  docs/test-report-kanban.md, docs/validation-kanban.md
- Pendiente: verificación manual en navegador (drag desktop, select mobile,
  carrusel) — no se hizo en esta sesión

### 2026-08-14 — Alta manual de libro + lista con filtros
- CRUD completo de `users/{uid}/books/`: `addBook`, `updateBook`, `deleteBook`,
  listener reactivo con `onSnapshot(query(..., orderBy('createdAt')))`
- Form único de alta/edición con todos los campos del modelo (incluidos
  `cover`, `isbn`, `pages`, `startedAt`, `finishedAt` aunque Entrega 1 no
  tenga autocompletado de API — se cargan a mano si el usuario quiere)
- **Gestión de áreas mínima e implícita**: sin colección `areas` separada:
  se derivan de `books` en memoria (`getAreaOptions()`). El form tiene
  autocompletado (`<datalist>`) + normalización case-insensitive
  (`normalizeArea()`) para evitar fragmentación ("Física" vs "física"). No
  hay pantalla de administración de áreas todavía — decisión de alcance
  documentada en docs/research-alta-libro.md, queda pendiente para una
  feature futura si hace falta renombrar/fusionar áreas ya creadas
  independientemente de tener libros cargados
- Paleta de área `--p0`–`--p9` agregada a `:root` (dark + light), color de
  área solo en dots/badges, nunca en píldoras de filtro (regla de CLAUDE.md)
- Filtros combinables por `status` y `area`, en memoria (sin re-consultar Firestore)
- Docs: docs/research-alta-libro.md, docs/stories-alta-libro.md,
  docs/spec-alta-libro.md, docs/test-report-alta-libro.md,
  docs/validation-alta-libro.md
- 2 bugs menores encontrados y corregidos en la propia revisión (ver
  test-report-alta-libro.md): timezone en fechas de inicio/fin de lectura,
  CSS faltante para `.book-card-main`
- Pendiente: verificación manual en navegador (no se hizo en esta sesión)

### 2026-08-13 — Login Google + multiusuario
- Bootstrap de Firebase SDK (v10.12.0, CDN gstatic) en `web/index.html`:
  `initializeApp`, `getAuth`, `getFirestore`
- Auth: `GoogleAuthProvider` + `signInWithPopup` con fallback a
  `signInWithRedirect`, `onAuthStateChanged` como única fuente de verdad para
  mostrar `#login` vs `#app`
- `escHtml()` sembrado como utilidad compartida para sanitizar innerHTML en
  las próximas features
- Docs: docs/research-login.md, docs/stories-login.md, docs/spec-login.md,
  docs/test-report-login.md, docs/validation-login.md
- Pendiente de infraestructura (no bloquea el código, sí el uso real): crear
  el proyecto Firebase, pegar el `firebaseConfig` real, aplicar Security
  Rules en consola, autorizar dominio de Vercel
