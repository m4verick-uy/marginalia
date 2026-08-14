# Spec — Board Kanban sobre status

**Feature:** Board Kanban sobre status
**Entrega:** Entrega 1
**Stack afectado:** JS, CSS, HTML (sin cambios de Firestore/Auth)

## Cambios en modelo de datos

Ninguno — usa `status` de `users/{uid}/books/{bookId}`, ya existente.

## Cambios en UI

- `.view-switch` en `.user-info` del header, antes de `.theme-btn`: 2 botones
  (`Lista` / `Tablero`), mismo patrón visual que ReMynder
- `#app main` reorganiza: `.filters` cambia dinámicamente según `viewMode`
  (oculta las píldoras de status cuando `viewMode === 'board'`, siempre
  muestra las de área)
- Nuevo contenedor `#board` (oculto en vista Lista) con 4 `.board-col`
  (`data-col-status="pendiente|leyendo|leido|abandonado"`)
- `.board-card`: título, área (badge con dot), autores si hay, `<select>` de
  status, sin badge de status propio (redundante con la columna)
- `.book-list` (ya existente) se oculta cuando `viewMode === 'board'`
- Mobile (`max-width: 640px`): `.board` con `scroll-snap-type: x mandatory`,
  columnas de ancho `calc(100vw - 3rem)`, `.board-dots` con 4 dots

## Cambios en lógica JS

- Estado nuevo: `viewMode` (`'list' | 'board'`, default desde
  `localStorage.getItem('viewMode') || 'list'`), `dragBookId`,
  `boardDndController`, `dotsController` (ambos `AbortController | null`)
- `COL_ORDER = ['pendiente', 'leyendo', 'leido', 'abandonado']`
- `COL_LABELS`: mapeo de status a label visible (reusa `STATUSES` ya definido)
- `moveBook(id, status)`: wrapper sobre `updateBook(id, { status })`; no
  escribe si el status nuevo es igual al actual
- `setViewMode(mode)`: persiste en `localStorage`, actualiza clase `.active`
  del view-switch, llama `render()`
- `render()` extendido: si `viewMode === 'list'` muestra `.book-list` y
  oculta `#board`; si `'board'` es al revés, y además oculta las píldoras de
  status del `.filters`
- `renderBoard()`: agrupa `getFilteredBooksForBoard()` (aplica solo
  `areaFilter`, no `statusFilter`) por columna, arma HTML, llama
  `initBoardDnd()`
- `initBoardDnd(boardEl)`: delegación con `AbortController`, eventos
  `dragstart/dragend/dragover/dragleave/drop` sobre `.board-card`/`.board-col`,
  + `change` delegado para los `<select>` de status por tarjeta
- `initMobileDots(boardEl)`: idéntico patrón a ReMynder, sincronizado a scroll

## Archivos a modificar

- `web/index.html` (único archivo)

## Riesgos técnicos

- **Fuga de listeners de drag-and-drop**: mitigado reusando el patrón
  `AbortController` de ReMynder — cada `renderBoard()` aborta el anterior
  antes de registrar los nuevos
- **4 columnas en mobile carousel**: ReMynder maneja 3, la lógica de dots es
  genérica (`board.querySelectorAll('.board-col')`), no requiere cambios
  estructurales, solo se agregará un 4º dot en el HTML estático

## Criterios de done

- [ ] Historias 1–4 cumplidas
- [ ] Sin fugas de listeners entre renders sucesivos del board (drag repetido
      no acumula handlers duplicados)
- [ ] Filtro de área funciona igual en Lista y Tablero
- [ ] Mobile: carrusel de 4 columnas navegable con swipe y con los dots
