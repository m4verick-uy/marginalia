# Spec — Alta manual de libro + lista con filtros

**Feature:** Alta manual de libro + lista con filtros
**Entrega:** Entrega 1
**Stack afectado:** JS, CSS, HTML, Firestore

## Cambios en modelo de datos

Sin cambios de esquema — se empieza a escribir/leer `users/{uid}/books/{bookId}`
tal como está definido en CLAUDE.md. No hay colección `areas` separada: la
lista de áreas se deriva en memoria de `books` (`[...new Set(books.map(b => b.area))]`).

Campos y sus defaults al crear:
- `title` (string, requerido)
- `authors` (string[], default `[]`)
- `area` (string, requerido, normalizado: trim + comparación case-insensitive
  contra áreas existentes al guardar)
- `subarea` (string, default `''`)
- `status` (string, default `'pendiente'`)
- `priority` (string, default `'curiosidad'`)
- `rating` (number, default `0`)
- `notes` (string, default `''`)
- `cover` (string, default `''`)
- `isbn` (string, default `''`)
- `pages` (number, default `null`)
- `startedAt` (number|null, default `null`)
- `finishedAt` (number|null, default `null`)
- `createdAt` (number, `Date.now()` al crear, inmutable)

## Cambios en UI

- `#app main`: reemplaza el placeholder. Estructura:
  - `.add-row`: botón "+ Agregar libro" que abre el form de alta
  - `.book-form`: form de alta/edición (mismo componente para ambos casos),
    inline arriba de la lista cuando está abierto — no modal
  - `.filters`: píldoras de status (Todas/Pendiente/Leyendo/Leído/Abandonado)
    + píldoras de área (dinámicas, generadas de `books`)
  - `.book-list`: contenedor de `.book-card`
  - `.empty`: estado vacío (reutiliza patrón de ReMynder)
- `.book-card`: título, autores (join con ", "), badge de área (dot + label,
  color por índice de área vía `--p0`–`--p9` igual que ReMynder), badge de
  status, rating (si > 0), acciones (editar, borrar)
- Selector de área dentro del form: input + datalist nativo de HTML
  (`<input list="area-options">` + `<datalist id="area-options">`) en vez de
  portar el dropdown custom de ReMynder — más simple, accesible por teclado
  out-of-the-box, y alcanza para "autocompletar + crear si no existe" sin
  reimplementar el patrón completo de `.cat-menu`. Si en una feature futura
  de gestión de áreas hace falta más control visual (colores, edición
  in-place), se reevalúa.
- Selector de autores: input de texto único donde el usuario separa por
  coma (`"José Edelstein, Andrés Gomberoff"`), parseado a array al guardar
  (trim de cada elemento, descarta vacíos). Alternativa de chips
  individuales se difiere — no la pide ninguna historia y agrega complejidad
  de UI no requerida todavía.
- `status` y `priority`: `<select>` nativos con las opciones fijas del modelo
- `rating`: control de 5 estrellas clicleables (0 = sin valorar), no input
  numérico libre
- Variables CSS nuevas: ninguna fuera de la paleta `--p0`–`--p9` (agregar al
  `:root`, siguiendo el patrón de ReMynder, recalibrada por tema)

## Cambios en lógica JS

- Estado nuevo en variables de módulo: `books` (array, espejo del listener),
  `unsubBooks`, `statusFilter` (default `'all'`), `areaFilter` (default
  `null`), `editingBookId` (default `null`)
- `startListeners(uid)`: engancha `onSnapshot(query(collection(db, 'users',
  uid, 'books'), orderBy('createdAt')), ...)`, se llama desde
  `onAuthStateChanged` en el branch de usuario logueado (ya existe el punto
  de enganche, comentado en el código actual)
- `addBook(data)`: valida título/área no vacíos, normaliza área (busca
  case-insensitive en `books` existente, reusa el string ya usado si hay
  match), `addDoc` con defaults
- `updateBook(id, data)`: misma validación/normalización, `updateDoc`
- `deleteBook(id)`: `confirm()` + `deleteDoc`, mismo patrón que
  `deleteCategory` de ReMynder
- `getAreaOptions()`: deriva la lista de áreas únicas de `books`, ordenada
  alfabéticamente, para poblar el `<datalist>` y las píldoras de filtro
- `normalizeArea(input)`: trim + busca coincidencia case-insensitive en
  `getAreaOptions()`, devuelve el string existente si matchea o el input
  trimeado si es nueva
- `render()` extendido: recalcula `.book-list` a partir de `books` +
  `statusFilter` + `areaFilter`, igual que el patrón de `render()` de
  ReMynder (recalcula todo desde el estado, no mutaciones incrementales del DOM)
- `escHtml()` aplicado a título, autores, área, subárea, notes, isbn — todo
  dato de usuario que va a `innerHTML`

## Archivos a modificar

- `web/index.html`: todo en el único archivo (sin build, sin npm)

## Archivos a crear

Ninguno.

## Riesgos técnicos

- **Normalización de área con solo case-insensitive**: no resuelve
  sinónimos reales ("Física" vs "Cosmología" que mencionaba CLAUDE.md como
  ejemplo) — eso requiere que el usuario elija conscientemente del
  autocompletado. Mitigación: el datalist muestra las áreas existentes
  primero, reduciendo la tentación de escribir una nueva a mano.
- **rating con 5 estrellas por click**: requiere manejo de evento por
  estrella individual — usar delegación de eventos en el contenedor
  `.rating-input`, no un listener por estrella.
- **Áreas derivadas de `books` en vez de colección propia**: si en el futuro
  se necesita reordenar/colorear áreas manualmente sin depender de tener al
  menos un libro, habrá que migrar a una colección `areas` — riesgo
  documentado y aceptado para esta feature (research-alta-libro.md ya lo
  señala como decisión de alcance).

## Criterios de done

- [ ] Historia 1–6 cumplidas según sus criterios de aceptación
- [ ] `escHtml()` aplicado a todo dato de usuario en `innerHTML`
- [ ] Ningún filtro ni alta/edición requiere recargar la página
- [ ] Mobile-first: form y lista usables en pantalla chica
- [ ] Paleta de área (`--p0`–`--p9`) agregada a `:root` y al bloque
      `html[data-theme="light"]`, recalibrada para contraste
