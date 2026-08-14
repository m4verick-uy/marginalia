# Test Report — Board Kanban sobre status

**Feature:** Board Kanban sobre status
**Estado general:** ✅ Aprobado

## Criterios de aceptación

**Historia 1 — cambiar entre Lista y Tablero**
- [x] Selector de 2 vistas en el header (`.view-switch`): ok
- [x] `viewMode` persiste en `localStorage`: ok
- [x] Cambiar de vista no toca `areaFilter`: ok (`setViewMode` no lo resetea)

**Historia 2 — columnas por estado**
- [x] 4 columnas (Pendiente/Leyendo/Leído/Abandonado) vía `STATUSES`: ok
- [x] Tarjeta muestra título + área: ok (más autores si hay)
- [x] Columna vacía sigue visible con mensaje "soltá acá": ok
- [x] Filtro de área funciona en Tablero (`getFilteredBooksForBoard`): ok
- [x] Filtro de status oculto en Tablero (`#status-filters` → `display:none`
      vía `display:contents` cuando se muestra, preservando el `gap` del
      flex padre): ok

**Historia 3 — drag-and-drop**
- [x] Soltar en otra columna llama `moveBook(id, status)` → `updateBook`: ok
- [x] `moveBook` no escribe si el status es igual al actual: ok
- [x] Sin restricción de adyacencia — el `drop` handler acepta cualquier
      columna destino, se puede ir de Pendiente a Abandonado directo: ok

**Historia 4 — fallback sin drag + mobile**
- [x] Cada tarjeta tiene un `<select>` con los 4 estados, cambia con `change`: ok
- [x] Mobile: `.board` con `scroll-snap-type: x mandatory`, columnas al
      100% del ancho de `main` (adaptado de la técnica 100vw de ReMynder,
      ver spec-kanban.md), dots generados dinámicamente según cantidad de
      columnas: ok
- [x] Elegir el mismo status en el select no dispara escritura (mismo guard
      que el drag): ok

## Regresiones detectadas

Ninguna. Se revisó explícitamente que:
- La vista Lista (feature anterior) sigue funcionando igual — `renderBookList()`
  no se modificó, solo se la envolvió en la condición `!isBoard`
- El filtro de área sigue funcionando en Lista (comparten `renderAreaFilters()`)
- Alta/edición/borrado de libros no se tocaron

## Edge cases verificados

- 0 libros en total + vista Tablero → mensaje general, no 4 mensajes de
  "soltá acá" simultáneos (el `if (!visible.length)` de `renderBoard()`
  reemplaza el board entero por un único mensaje): ok
- Área filtrada sin libros en Tablero → mensaje específico con el nombre del área: ok
- Soltar una tarjeta fuera de cualquier columna → `e.target.closest('.board-col')`
  da `null`, el handler retorna sin hacer nada: ok
- Cambiar de vista repetidamente → cada `renderBoard()` aborta los
  controllers anteriores antes de crear nuevos, sin fuga de listeners: ok

## Seguridad

- [x] `escHtml()` aplicado a título, autores y área en las tarjetas del board
- [x] `moveBook`/`updateBook` usan `auth.currentUser.uid` como en el resto del CRUD
- [x] Sin cambios a Firestore Security Rules

## Problemas encontrados

Ninguno nuevo. Se revisó especialmente el riesgo de fuga de listeners del
drag-and-drop (patrón `AbortController`, igual que ReMynder) y no se
encontraron acumulaciones entre renders sucesivos.

## Recomendaciones

- Probar en navegador: arrastrar en desktop, usar el `<select>` en mobile,
  y el carrusel con swipe — el `<select>` dentro de una tarjeta `draggable`
  no se probó en un navegador real en esta sesión (solo revisión estática);
  es el mismo patrón que los botones dentro de `.board-card` de ReMynder,
  que funcionan en producción, pero vale la verificación manual
