# Spec — priority: valor "sin_especificar"

**Agente:** Spec Writer
**Fecha:** 2026-09-18
**Feature:** cuarto valor `sin_especificar` en el eje `priority`
**Entrega:** Entrega 1
**Stack afectado:** JS (lógica + estado), HTML (una `<option>`). Sin CSS. Sin
Firestore Rules. Sin API externa. Sin dependencias nuevas.

---

## Cambios en modelo de datos

**Colección:** `users/{uid}/books/{bookId}` — **campo `priority`**

```
priority: "curiosidad" | "interesado" | "must_have" | "sin_especificar"
```

- El campo sigue siendo un string libre a nivel Firestore (no hay validación de
  esquema en el backend; no hay backend). La lista controlada se garantiza en
  el cliente: al escribir, por el `<select>`; al leer, por `normalizePriority()`.
- **`status` no cambia.** Ningún otro campo del modelo se toca.
- **Sin migración.** Los documentos existentes no se reescriben. La
  normalización es en memoria y no dispara ninguna escritura.
- Los dos ejes siguen siendo ortogonales: `status` es progreso, `priority` es
  intención. El default por estado vive **solo en el formulario de alta**.

### Semántica de `sin_especificar` (contrato para Fase 2)

`sin_especificar` es **ausencia de intención**, no un nivel bajo de deseo.
No es comparable con los otros tres valores: no ordena por debajo de
`curiosidad`, está fuera de la escala. Las metas de conocimiento de Fase 2
(que cruzan `priority` + `area` + `status`) deben tratarlo como **no
clasificado** y **nunca** sumarlo al progreso de un objetivo.

---

## Cambios en lógica JS

### Constante nueva (módulo)

```js
const PRIORITIES = ['curiosidad', 'interesado', 'must_have', 'sin_especificar'];
```

Ubicación: junto a las demás constantes de módulo, antes de las funciones que
la usan. Única fuente de verdad del enum en el código.

### Función nueva — `normalizePriority(value)`

Pura, sin efectos secundarios.

- Devuelve `value` si pertenece a `PRIORITIES`.
- Devuelve `'sin_especificar'` en cualquier otro caso: `undefined`, `null`,
  `''`, valor desconocido, o un tipo que no sea string.

### Función nueva — `defaultPriorityForStatus(status)`

Pura. Es el contrato del default inteligente, aislado para que la Fase 2 y el
Test Verifier puedan razonar sobre él sin leer el formulario.

- `'leido'` o `'abandonado'` → `'sin_especificar'`
- cualquier otro status (`'pendiente'`, `'leyendo'`) → `'curiosidad'`

### Estado nuevo (módulo)

```js
let formPriorityTouched = false;
```

Sigue el patrón de `formRatingValue`: estado efímero del formulario abierto.
`true` cuando el usuario cambió el selector de prioridad a mano en el
formulario actual. Mientras sea `false` y se trate de un alta nueva, el
default se recalcula al cambiar el estado.

### Funciones modificadas

**`startListeners(uid)` — mapeo del snapshot (línea ~934)**
El `.map()` pasa a normalizar la prioridad al construir el array `books`:

```js
books = snap.docs.map(d => {
  const data = d.data();
  return { id: d.id, ...data, priority: normalizePriority(data.priority) };
});
```

Es el único punto de entrada de datos a la app, así que toda vista y toda
feature futura recibe un valor válido. **No escribe a Firestore.**

**`openBookForm(book)`**
- El default del selector pasa de `'curiosidad'` literal a
  `defaultPriorityForStatus('pendiente')` — mismo resultado hoy, pero deja de
  haber un valor mágico duplicado.
- Para un libro existente usa `normalizePriority(book.priority)` (defensa en
  profundidad: aunque el array ya viene normalizado, el selector nunca debe
  quedar en `selectedIndex = -1`).
- Resetea `formPriorityTouched = false`.

**`closeBookForm()`**
- Resetea `formPriorityTouched = false`, junto a `editingBookId` y
  `formRatingValue`.

**Listeners nuevos (zona de wiring, junto a los demás `addEventListener`)**
- `#book-priority` → `change`: marca `formPriorityTouched = true`.
- `#book-status` → `change`: si `editingBookId === null` (alta nueva) **y**
  `formPriorityTouched === false`, setea el valor del selector de prioridad a
  `defaultPriorityForStatus(nuevoStatus)`. En cualquier otro caso no hace nada.

**`submitBookForm()`**
- Sin cambios de comportamiento; sigue leyendo `document.getElementById('book-priority').value`.
  El valor siempre pertenece al enum porque proviene del `<select>`.

### Funciones que NO se tocan (invariantes a preservar)

- `moveBook(id, status)` — el Kanban cambia **solo** `status`. Jamás `priority`.
- `addBook()` / `updateBook()` — sin cambios.
- `getFilteredBooks()`, `render()`, `renderBoard()` — no leen `priority`.

---

## Cambios en UI

**`web/index.html`, `<select id="book-priority">` (~línea 775):** una opción
nueva al final.

```html
<option value="sin_especificar">Sin especificar</option>
```

- Al final, después de Must-have: `sin_especificar` no es un escalón de la
  escala de deseo.
- **Sin CSS nuevo.** Es el mismo control `<select>`, ya estilado, con soporte
  dark/light y mobile ya resuelto. No se introducen variables CSS, ni colores,
  ni clases.
- No aplica la regla de jerarquía visual (`>` en acciones primarias): no se
  agrega ningún botón ni control anidado.
- No aplica sanitización: markup estático, sin dato de usuario.

---

## Archivos a modificar

| Archivo | Qué cambia | Por qué |
|---|---|---|
| `web/index.html` | `<option>` nueva; `PRIORITIES`; `normalizePriority()`; `defaultPriorityForStatus()`; `formPriorityTouched`; mapeo del snapshot; `openBookForm()`; `closeBookForm()`; 2 listeners | Implementa H1, H2 y H3 |
| `CLAUDE.md` | Enum en "Modelo de datos"; regla de default y semántica Fase 2 en "Por qué dos ejes" | H4 — contexto compartido de los agentes |
| `docs/spec-alta-libro.md` | Enum y default de `priority` | Consistencia con la feature previa |
| `docs/research-alta-libro.md` | Enum de `priority` | Consistencia |
| `docs/documentacion.md` | Registro del cambio de modelo | Estándar 7 de CLAUDE.md |

**Archivos a crear:** ninguno además de los docs de la factory.

---

## Riesgos técnicos y mitigación

| Riesgo | Mitigación |
|---|---|
| Colapsar los dos ejes derivando `priority` de `status` | El default vive exclusivamente en el listener del form de alta, condicionado a `editingBookId === null`. `moveBook()` y la edición quedan intactos. Verificación explícita en el test report |
| Migración accidental de datos | La normalización ocurre en el `.map()` de lectura, en memoria. No hay `updateDoc` en ese camino |
| El `<select>` queda en blanco ante un valor desconocido | `normalizePriority()` en la lectura **y** en `openBookForm()` |
| El flag queda sucio entre formularios | Se resetea en `openBookForm()` y en `closeBookForm()` |
| Valor mágico `'curiosidad'` duplicado en el código | Un único `defaultPriorityForStatus()` |
| Relajar Security Rules | No se tocan: mismo documento, mismo owner, mismo campo |

---

## Criterios de done

- [ ] `PRIORITIES` es la única lista del enum en el código
- [ ] `normalizePriority()` y `defaultPriorityForStatus()` son funciones puras y de responsabilidad única
- [ ] El array `books` siempre tiene una `priority` válida
- [ ] Ninguna ruta de código nueva escribe a Firestore
- [ ] El `<select>` tiene las 4 opciones, "Sin especificar" al final
- [ ] El default reactivo respeta la elección manual del usuario
- [ ] Editar un libro existente y cambiarle el status no altera su `priority`
- [ ] `moveBook()` no aparece modificada en el diff
- [ ] Sin `console.log`, sin CSS nuevo, sin dependencias nuevas
- [ ] Firestore Security Rules sin cambios
- [ ] `CLAUDE.md` refleja el enum, el default por estado y la semántica de Fase 2
