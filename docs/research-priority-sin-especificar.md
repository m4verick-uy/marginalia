# Research — priority: valor "sin_especificar"

**Agente:** Researcher
**Fecha:** 2026-09-18
**Feature:** agregar un cuarto valor `sin_especificar` al eje `priority`

---

## 1. Resumen del problema

El eje `priority` (intención) admite hoy tres valores — `curiosidad`,
`interesado`, `must_have` — y los tres expresan algún grado de deseo activo.
Eso obliga al lector a declarar una intención que no existe cuando carga un
libro que ya leyó (o que abandonó): en esos casos la prioridad dejó de ser
accionable y pasó a ser dato histórico. El resultado práctico es ruido en el
eje de intención, que es justamente el que va a alimentar las metas de Fase 2.

El valor neutro `sin_especificar` representa **ausencia de intención**, no un
nivel bajo de deseo. No es "poco interés": es "no aplica / no lo registré".

## 2. A qué entrega pertenece y por qué

**Entrega 1.** Toca el modelo de datos, el formulario de alta/edición y la
lectura defensiva del campo — todo ya construido en la Entrega 1. No requiere
API externa ni portadas, así que no mezcla Entrega 2.

El punto 4 del brief (semántica para las metas de conocimiento) es **solo
documentación**: se deja escrita la regla en CLAUDE.md para que la Fase 2 la
respete, pero no se construye ninguna lógica ni UI de metas en este run.

## 3. Archivos relevantes identificados

| Archivo | Líneas | Rol |
|---|---|---|
| `web/index.html` | 774-780 | `<select id="book-priority">` con las 3 opciones actuales |
| `web/index.html` | 933-936 | `startListeners()` → `onSnapshot` mapea los docs de Firestore al array `books` — **único punto de entrada de datos a la app** |
| `web/index.html` | 1049-1071 | `openBookForm()` — setea los valores del form; default actual `'curiosidad'` |
| `web/index.html` | 1081-1118 | `submitBookForm()` — arma `fields` y llama `addBook`/`updateBook` |
| `web/index.html` | 1003-1015 | `addBook()` / `updateBook()` — escrituras Firestore |
| `CLAUDE.md` | Modelo de datos / Por qué dos ejes | Enum a actualizar |
| `docs/spec-alta-libro.md` | 20 | Documenta `priority` default `'curiosidad'` |

## 4. Hallazgo central: `priority` no se renderiza en ninguna vista

`grep -n "priority" web/index.html` devuelve **3 ocurrencias en JS**, todas
dentro del formulario. Consecuencias directas para los builders:

- **No hay filtro por prioridad**, ni píldora, ni badge, ni dot de color. No
  hay CSS de prioridad que tocar, ni paleta `--pN` involucrada.
- **La lista y el Kanban no muestran `priority`.** Un valor desconocido o
  ausente hoy no rompe nada visualmente; el riesgo real está en el `<select>`,
  que ante un `value` inexistente en sus `<option>` queda en blanco
  (`selectedIndex = -1`) y al guardar escribiría `''`.
- El fallback de lectura tiene **un solo lugar natural**: el `.map()` del
  `onSnapshot` (línea 934). Normalizar ahí garantiza que todo consumidor
  presente y futuro (metas de Fase 2 incluidas) reciba un valor válido, sin
  duplicar la defensa en cada vista.

## 5. Patrones y convenciones a respetar

- Vanilla ES modules, sin build. Estado en variables de módulo (`books`,
  `editingBookId`, `formRatingValue`, `statusFilter`, `areaFilter`).
- `render()` recalcula la UI desde el estado.
- El form es **uno solo** para alta y edición, discriminado por `editingBookId`
  (`null` = alta). Cualquier comportamiento "solo en el alta" se condiciona
  sobre esa variable.
- `closeBookForm()` resetea el estado de formulario (`editingBookId`,
  `formRatingValue`) — cualquier flag nuevo de formulario se resetea ahí.
- Las opciones del `<select>` son literales en el HTML, no generadas por JS.
  Mantener ese patrón: agregar la opción nueva como markup, no por `innerHTML`.
- `escHtml()` antes de todo `innerHTML`. Nota: los `<select>` de status y
  priority son markup estático y sus valores son de una lista controlada —
  no entra dato de usuario, no hay superficie XSS nueva.

## 6. Riesgos y conflictos detectados

### 6.1 Discrepancia en el brief — RESUELTA por el Ingeniero Jefe
El brief pedía "mantener el default actual (`interesado`)", pero el default
implementado es `'curiosidad'` (`web/index.html:1057`, y así lo documenta
`docs/spec-alta-libro.md:20`). **Decisión tomada:** se mantiene `'curiosidad'`
para `pendiente`/`leyendo`. No se cambia el default vigente; el único default
nuevo es el de `leido`/`abandonado`.

### 6.2 El form abre siempre en `status = 'pendiente'`
Si el default de prioridad se calculara una sola vez al abrir el formulario,
`sin_especificar` no se aplicaría nunca solo — el usuario elige "Leído"
*después* de que el form ya abrió. **Decisión tomada:** el default es
**reactivo hasta que el usuario toque la prioridad**. Al cambiar el status en
un alta nueva se recalcula la prioridad; en cuanto el usuario elige una
prioridad a mano, la app deja de recalcular por el resto de ese formulario.
Al **editar** un libro existente nunca se recalcula.

### 6.3 Riesgo de colapsar los dos ejes — señal de alerta del Orchestrator
Derivar `priority` de `status` es exactamente el tipo de acoplamiento que
CLAUDE.md prohíbe. La mitigación es precisa: el status **sugiere un valor
inicial en el formulario de alta y nada más**. En particular:
- `moveBook()` (drag & drop del Kanban y `<select>` de status en la tarjeta)
  **NO debe tocar `priority`**. Mover un libro a "Leído" desde el board no
  cambia su intención.
- Editar un libro existente y cambiarle el status **NO debe tocar `priority`**.
- Los dos campos siguen siendo independientes en Firestore y en el modelo.

### 6.4 Migración: prohibida en batch
Hay libros en producción sin el valor nuevo. El brief es explícito: nada de
reescritura masiva. La normalización en lectura debe ser **solo en memoria** —
no dispara `updateDoc`. Un documento existente solo cambia si el usuario lo
edita y guarda a mano, que es el comportamiento normal del form.

Efecto secundario aceptado y esperado: si un libro tuviera `priority` vacía o
corrupta y el usuario lo edita y guarda, se persiste `sin_especificar`. Eso es
una edición manual del usuario, no una migración.

### 6.5 `form.reset()` y el flag nuevo
`closeBookForm()` llama a `form.reset()`, que devuelve el `<select>` a su
primer `<option>`. `openBookForm()` setea todos los valores explícitamente, así
que no hay conflicto — pero el flag de "el usuario ya tocó la prioridad" debe
resetearse en `closeBookForm()` junto a `editingBookId` y `formRatingValue`,
o el segundo alta de la sesión arrancaría con el flag sucio.

### 6.6 Orden de las opciones en el `<select>`
`sin_especificar` no es un nivel de la escala curiosidad → interesado →
must_have; es ortogonal a ella. Ponerlo en el medio de la escala la rompe
visualmente. Va al final, después de `must_have`, separado conceptualmente.

## 7. Recomendaciones para los siguientes agentes

**Story Writer**
- Cuatro historias: normalización en lectura, opción en el selector, default
  inteligente reactivo, documentación de la semántica.
- Edge cases obligatorios a cubrir: libro viejo sin el campo; libro con valor
  basura; alta donde el usuario elige prioridad *antes* de cambiar el status;
  alta donde la cambia *después*; edición de un libro existente; cambio de
  status desde el Kanban.

**Spec Writer**
- Definir una constante de módulo con el enum y un helper puro
  `normalizePriority(value)`; que el mapeo del snapshot lo use.
- Definir `defaultPriorityForStatus(status)` como función pura separada —
  es la que la Fase 2 va a querer leer, y mantiene "una función, una cosa".
- Nombrar el flag de formulario siguiendo el patrón existente
  (`formRatingValue` → sugerido `formPriorityTouched`).

**Backend Builder**
- Todo el cambio de lógica vive en JS. No tocar Security Rules (no cambian:
  el campo es del mismo documento, mismo owner).
- Cero `console.log`. Nada de `updateDoc` disparado por la normalización.

**Frontend Builder**
- Un `<option value="sin_especificar">Sin especificar</option>` al final del
  `<select>`. No hay CSS nuevo: el `<select>` ya está estilado y es el mismo
  control. Mobile-first se mantiene por construcción (no cambia el layout).

**Test Verifier**
- Verificar explícitamente que el Kanban no altera `priority`, y que un libro
  existente sin tocar conserva su valor en Firestore.
