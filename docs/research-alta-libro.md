# Research — Alta manual de libro + lista con filtros

## Resumen del problema

Es la primera feature de contenido real de marginaLia: dar de alta libros a
mano con el modelo de datos completo definido en CLAUDE.md, y verlos en una
lista filtrable por `status` y `area`. Sin esto, ninguna otra pieza de la
Entrega 1 (Kanban, torta de áreas, vista de temas) tiene sobre qué pararse.

## A qué entrega pertenece

Entrega 1. No usa la API externa (Google Books/OpenLibrary es Entrega 2) —
`cover`, `isbn`, `pages` quedan en el modelo pero vacíos/editables a mano.

## Estado actual del código

`web/index.html` tiene: login, header con avatar/nombre/logout/tema. El
`<main>` de `#app` está vacío, con un comentario marcando que acá engancha
esta feature. Ya existen: `db` (Firestore instance), `auth`, `escHtml()`,
convención de `onAuthStateChanged` como fuente de verdad de sesión.

No hay todavía ningún listener de Firestore activo, ni colección `books`
creada, ni gestión de áreas.

## Modelo de datos (de CLAUDE.md, sin cambios)

```
users/{uid}/books/{bookId}
  title, authors[], area, subarea, status, priority, rating, notes,
  cover, isbn, pages, startedAt, finishedAt, createdAt
```

`status`: pendiente | leyendo | leido | abandonado (eje de progreso, Kanban)
`priority`: curiosidad | interesado | must_have (eje de intención, metas Fase 2)
Ambos ejes son independientes — no colapsar en un campo derivado.

## Dependencia con "gestión de áreas de conocimiento"

CLAUDE.md lista "Gestión de áreas de conocimiento (lista dinámica, validada)"
como un ítem separado de "Alta manual de libro" dentro de la Entrega 1. Pero
el alta de libro necesita elegir un área — hay una dependencia circular de
alcance:

- Si el alta de libro exige elegir de una lista de áreas ya creada, no se
  puede probar la feature hasta tener la de gestión de áreas
- Si el alta de libro permite escribir el área libremente sin ningún control,
  se viola la regla explícita de CLAUDE.md: "Evitar que se fragmenten
  ('Física' vs 'física' vs 'Cosmología' suelta) — validar contra la lista
  existente"

**Recomendación:** esta feature incluye una gestión de áreas *mínima e
implícita*, embebida en el form de alta/edición de libro: un input de área
con autocompletado contra las áreas ya usadas por el usuario (normalizado,
case-insensitive) que permite tanto elegir una existente como escribir una
nueva. No incluye una pantalla dedicada de administración de áreas (renombrar
un área ya creada globalmente, fusionar duplicados, elegir color) — eso queda
para una feature futura de "gestión de áreas" más completa, análoga a la
pantalla de Settings de categorías que tiene ReMynder. Se documenta esto
explícitamente para que Story Writer decida si lo confirma o lo escala como
pregunta abierta.

## Patrones a reutilizar de ReMynder

- Selector de categoría inline dentro del input (patrón Recordatorios iOS):
  `.cat-select-btn` + `.cat-menu` desplegable, con dot de color + nombre +
  chevron. Se puede portar como selector de **área** (sin el sistema de color
  fijo por índice de ReMynder — CLAUDE.md pide paleta `--p0`–`--pN` mapeada
  por índice, mismo mecanismo, pero aplicada a áreas en vez de categorías)
- `onSnapshot` con `query(..., orderBy('createdAt'))` para lista reactiva
- Delegación de eventos en listas (no listener por ítem) — CLAUDE.md lo pide
  explícitamente para marginaLia también
- Filtros como píldoras (`.nav-btn`) con estado `.active`, mono-color con el
  acento — CLAUDE.md: "el color de área vive solo en dots/indicadores, no en
  píldoras de filtro"
- Formulario de edición inline sobre la propia tarjeta (`.task-item-form`),
  reutilizable para editar libros sin modal aparte

## Riesgos o conflictos detectados

- **Fragmentación de áreas**: sin normalización (trim + comparación
  case-insensitive) al crear una nueva área, "Física" y "física" quedarían
  como dos áreas distintas — viola CLAUDE.md explícitamente
- **Autores como array**: el input debe soportar múltiples autores
  (coautoría), no un string único — CLAUDE.md lo marca como decisión de
  arquitectura central, no negociable
- **Alcance de "todos los campos editables"**: CLAUDE.md pide que el alta
  tenga "todos los campos del modelo, todos editables". Esto incluye
  `cover`, `isbn`, `pages`, `startedAt`, `finishedAt` — aunque en Entrega 1
  no hay autocompletado de API, el formulario debería permitir cargarlos a
  mano si el usuario quiere (ej. ISBN de la tapa del libro). Confirmar
  alcance con Story Writer: ¿el formulario inicial expone estos campos
  opcionales, o quedan ocultos hasta la Entrega 2 para no sobrecargar el
  formulario del MVP?
- **rating y notes**: CLAUDE.md los lista como ítem propio de la Entrega 1
  ("Valoración (rating) y notas (marginalia) por libro") — deben estar en el
  form de alta/edición, no diferidos

## Recomendaciones para los siguientes agentes

- Story Writer: definir el alcance exacto de campos visibles en el form
  inicial (mínimos vs. completos) y resolver la pregunta de gestión de áreas
  mínima vs. completa antes de escribir criterios de aceptación
- Spec Writer: portar el patrón de selector inline de ReMynder para área,
  definir estructura de `.book-item` / `.book-card` y su form de edición
- Backend Builder: `addBook`, `updateBook`, `deleteBook`, listener de
  `users/{uid}/books` con `onSnapshot` + `orderBy('createdAt')`; derivar la
  lista de áreas existentes del propio listener de libros (no hace falta una
  colección `areas` separada todavía, ya que no hay gestión completa en esta
  feature — evaluar si conviene igual para evitar migrar después)
- Frontend Builder: filtros por status y area como píldoras; lista con
  tarjeta de libro (título, autores, área, status, rating); estado vacío
