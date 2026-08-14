# Test Report — Alta manual de libro + lista con filtros

**Feature:** Alta manual de libro + lista con filtros
**Estado general:** ⚠️ Aprobado con observaciones

## Criterios de aceptación

**Historia 1 — dar de alta un libro**
- [x] Punto de entrada visible (`+ Agregar libro`): ok
- [x] Form pide todos los campos del modelo, título/área obligatorios, resto opcional: ok
- [x] Guardado se refleja sin recargar (`onSnapshot`): ok
- [x] Título vacío → error inline, no guarda: ok
- [x] Área vacía → error inline, no guarda: ok
- [x] Área existente (con distinta capitalización/espacios) se reutiliza vía `normalizeArea()`: ok

**Historia 2 — editar libro**
- [x] Acción de editar por libro (`.edit-book-btn`): ok
- [x] Form pre-carga todos los valores, incluidos ambos ejes por separado: ok
- [x] Guardado actualiza sin recargar: ok
- [x] Mismas validaciones que alta: ok (comparten `submitBookForm`)
- [x] status y priority son campos independientes en el form, no se pisan entre sí: ok

**Historia 3 — borrar libro**
- [x] Acción de eliminar (`.del-book-btn`): ok
- [x] `confirm()` antes de borrar: ok
- [x] Cancelar la confirmación no borra nada (comportamiento nativo de `confirm()`): ok

**Historia 4 — ver lista**
- [x] Card muestra título, autores, área (badge con dot de color), status, rating: ok
- [x] Estado vacío general ("todavía no cargaste ningún libro"): ok
- [x] Orden estable por `createdAt` vía `orderBy`: ok

**Historia 5 — filtro por status**
- [x] 4 estados + "Todas": ok
- [x] No recarga, re-renderiza en memoria: ok
- [x] Filtro activo con clase `.active`: ok

**Historia 6 — filtro por área**
- [x] Píldoras de área generadas dinámicamente desde `getAreaOptions()`: ok
- [x] Combinable con filtro de status (`getFilteredBooks()` aplica ambos): ok
- [x] Área sin libros deja de listarse como filtro (se deriva de `books`, no hay colección separada): ok

## Regresiones detectadas

Ninguna en login/logout/tema — no se tocó esa lógica, solo se agregó código nuevo.

## Edge cases verificados

- Libro sin autores → `authorsHtml` se omite, no rompe el layout: ok
- Libro sin rating (`rating: 0`) → no se renderiza la línea de estrellas: ok
- Filtro con 0 resultados → mensaje específico distinto del estado vacío general: ok
- Título con solo espacios → `.trim()` lo deja vacío, dispara el error: ok
- Autores vacío → válido, se guarda como array vacío: ok

## Seguridad

- [x] `escHtml()` aplicado a título, autores, área (badge y filtros), y en el
      `<datalist>` de autocompletado — todo dato de usuario que llega a innerHTML
- [x] `notes`, `isbn`, `cover`, `pages`, fechas: no se renderizan en ningún
      `innerHTML` todavía (solo se leen/escriben como `.value` de inputs, que
      no ejecuta HTML) — sin superficie XSS aunque no pasen por `escHtml()`
- [x] Firestore Security Rules no tocadas
- [x] Todas las operaciones (`addBook`, `updateBook`, `deleteBook`,
      `startListeners`) usan `auth.currentUser.uid` — un usuario no puede
      tocar libros de otro

## Problemas encontrados (corregidos en esta verificación)

- **Bug de timezone (menor):** `timestampToDateInputValue()` usaba
  `toISOString().slice(0,10)`, que convierte a UTC. Para usuarios en husos
  horarios con offset positivo (Europa, Asia), una fecha de inicio/fin de
  lectura guardada a medianoche local podía mostrarse un día antes al reabrir
  el form de edición. Corregido: ahora arma el string `YYYY-MM-DD` con
  `getFullYear()/getMonth()/getDate()` (hora local), simétrico con
  `dateToTimestamp()` que también construye la fecha en hora local.
- **CSS faltante (menor):** `.book-card-main` se usaba en el template de
  `renderBookCard()` pero no tenía regla CSS — sin `flex:1; min-width:0`, un
  título largo podía forzar el ancho de la tarjeta o encimarse con los
  botones de acción en mobile. Corregido, con `overflow-wrap: break-word`
  agregado a `.book-title` también.

## Recomendaciones

- Verificar en navegador con datos reales (crear 2-3 libros, editar, borrar,
  combinar filtros) antes de dar la feature por cerrada en la práctica
- La normalización de área es solo case-insensitive — no resuelve sinónimos
  reales; ya está documentado como riesgo aceptado en spec-alta-libro.md
