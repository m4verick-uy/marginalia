# Research — Board Kanban sobre status

## Resumen del problema

CLAUDE.md pide, para la Entrega 1: "Board Kanban sobre status (transfiere del
`.board` de ReMynder casi literal)". Es una segunda forma de ver los mismos
libros que ya lista `alta-libro`, organizada por columnas de `status` en vez
de una lista lineal.

## A qué entrega pertenece

Entrega 1, sin dependencias de API externa.

## Cómo lo resuelve ReMynder (patrón a portar)

Estudiado en detalle en `/home/m4verick/Proyects/reMynder/web/index.html`:

- `viewMode` (`'list' | 'board' | 'focus'`) en `localStorage`, con un
  `.view-switch` de botones en el header que llama `setViewMode()`
- `renderBoardColumnsHtml(visible)`: agrupa las tareas visibles por status en
  un objeto `{todo:[], doing:[], done:[]}` y arma el HTML de columnas +
  tarjetas en un solo string
- Cada `.board-card` tiene `draggable="true"` y botones de fallback
  (`move-prev`/`move-next`) para mover sin drag — importante para mobile y
  accesibilidad
- `initBoardDnd(board)`: un único listener por tipo de evento en el
  contenedor del board (delegación), usando `AbortController` para poder
  desregistrar todo en el próximo render sin fugas de memoria
  (`dragstart/dragend/dragover/dragleave/drop` + `click` delegado con
  `data-action`)
- Columna vacía: sigue mostrándose (con mensaje "soltá acá"), nunca desaparece
  — necesaria como zona de drop aunque no tenga tarjetas
- Mobile: columnas en carrusel horizontal con `scroll-snap`, dots debajo
  sincronizados por scroll (`initMobileDots`)
- El filtro de categoría (`catFilter`) sí se aplica dentro del board; el
  filtro de estado (pendiente/completada) en ReMynder **no se aplica en board
  ni en focus** — queda como pill clickeable pero sin efecto visible en esas
  vistas, porque el status ya es el eje de las columnas. Es una inconsistencia
  menor de UX en el original (el pill sigue reaccionando al click aunque no
  cambie nada).

## Diferencia clave para marginaLia: 4 estados, no 3

ReMynder tiene 3 columnas lineales (`todo → doing → done`), con
`PREV_STATUS`/`NEXT_STATUS` como mapeos de un paso. marginaLia tiene 4
estados (`pendiente | leyendo | leido | abandonado`), y **no son
estrictamente lineales**: `abandonado` es una bifurcación que puede alcanzarse
desde `pendiente` o `leyendo`, no un paso siguiente de `leido`. Forzar
botones `move-prev`/`move-next` de un solo paso no representa bien ese grafo.

**Decisión de diseño (no es una decisión de producto, es una adaptación de
interacción):** en vez de botones prev/next lineales, cada tarjeta del board
tiene un `<select>` de status embebido como fallback sin drag — permite saltar
a cualquiera de los 4 estados directamente, sin forzar un orden. El
drag-and-drop tampoco tiene restricción de adyacencia en el original (el
`drop` handler acepta cualquier columna destino), así que esto no reduce
funcionalidad respecto a ReMynder, solo cambia el fallback sin mouse.

## Segunda diferencia: sí aplicar el filtro de status como corresponde

En vez de reproducir la inconsistencia de ReMynder (pill de status visible
pero sin efecto en board), en marginaLia el filtro de `status` se **oculta**
cuando la vista activa es Tablero (el status ya es el eje de las columnas —
mostrarlo sería redundante y confuso). El filtro de área se mantiene visible
y funcional en ambas vistas, igual que en ReMynder.

## Vista: agregar selector Lista / Tablero

Hoy marginaLia solo tiene la vista de lista (feature anterior). Esta feature
agrega un selector de 2 vistas (no 3 — no hay "Foco" en el alcance de
marginaLia) en el header, mismo patrón visual que `.view-switch` de ReMynder.

## Patrones a reutilizar tal cual

- Persistencia de `viewMode` en `localStorage`
- Delegación de eventos con `AbortController` para drag-and-drop
- Columna vacía siempre visible como zona de drop
- Carrusel mobile con `scroll-snap` + dots

## Riesgos o conflictos detectados

- Reutilizar `renderBookCard` tal cual no alcanza — la tarjeta de board
  necesita ser más compacta (sin repetir el badge de status, ya que es
  redundante con la columna) y agregar el `<select>` de cambio rápido +
  drag handle implícito
- Mobile: con 4 columnas en vez de 3, el carrusel y los dots deben soportar
  4 elementos — no es un cambio estructural, solo de cantidad
- `moveBook(id, newStatus)` es una operación nueva (no existía `updateBook`
  llamado desde el board) pero reutiliza `updateBook` ya implementado

## Recomendaciones para los siguientes agentes

- Story Writer: las historias deben cubrir drag-and-drop, el fallback sin
  drag (select de status), filtro de área en board, y el cambio de vista
- Spec Writer: definir `COL_ORDER` = `['pendiente','leyendo','leido','abandonado']`,
  reutilizar `updateBook(id, { status })` para mover
- Backend Builder: `moveBook(id, status)` como wrapper fino sobre `updateBook`
- Frontend Builder: portar drag-and-drop con `AbortController`, columnas +
  carrusel mobile, `.view-switch` de 2 opciones
