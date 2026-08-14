# Stories — Alta manual de libro + lista con filtros

Basado en docs/research-alta-libro.md.

## Decisiones de alcance (no requieren pregunta al Ingeniero Jefe)

- **Áreas — gestión mínima e implícita.** El form de alta/edición de libro
  incluye un selector de área con autocompletado contra las áreas ya usadas
  (normalizado, sin duplicar por mayúsculas/espacios) que permite crear una
  nueva escribiendo. No hay pantalla dedicada de administración de áreas
  (renombrar, fusionar, elegir color) en esta feature — queda para una
  feature futura, análoga a la de categorías de ReMynder.
- **Todos los campos del modelo son parte del form**, tal como pide CLAUDE.md
  ("todos los campos del modelo, todos editables"): título, autores, área,
  subárea, status, priority, rating, notes, cover (URL), isbn, pages,
  startedAt, finishedAt. Los que dependen de la API externa (Entrega 2)
  quedan como campos manuales opcionales, no ocultos.

---

**Historia 1:** Como lector, quiero dar de alta un libro a mano con todos sus
datos para empezar a registrar mi bitácora de lectura.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Hay un punto de entrada visible en `#app main` para agregar un libro
      (ej. botón "+ Agregar libro")
- [ ] El form pide: título (obligatorio), autores (uno o más, coautoría
      soportada), área (obligatorio, con autocompletado + creación inline),
      subárea (opcional), status (default "pendiente"), priority (default
      "curiosidad"), rating (default 0/sin valorar), notes, cover (URL),
      isbn, pages, startedAt, finishedAt — todos menos título y área son
      opcionales
- [ ] Al guardar, el libro aparece en la lista sin recargar la página
      (Firestore `onSnapshot`)
- [ ] Si el título está vacío, no se guarda y se muestra un error inline
- [ ] Si el área está vacía, no se guarda y se muestra un error inline
- [ ] Escribir un área que ya existe (aunque con mayúsculas/espacios
      distintos) reutiliza la existente, no crea una duplicada

**Edge cases:**
- Título con solo espacios → tratado como vacío, error
- Autores vacío → válido (libro sin autor cargado todavía, ej. antología)
- Área nueva escrita desde cero → se crea y queda disponible para el
  próximo libro
- Área "Física " (con espacio) cuando ya existe "Física" → se normaliza a
  la existente, no genera "Física " y "Física" como dos áreas
- rating fuera de 0–5 → el control de UI no permite salir de ese rango
  (no es un input de texto libre)

---

**Historia 2:** Como lector, quiero editar cualquier dato de un libro ya
cargado (incluida su valoración y mis notas) para corregir errores o
actualizar mi opinión después de leerlo.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Cada libro en la lista tiene una acción para editarlo
- [ ] El form de edición pre-carga todos los valores actuales, incluidos los
      dos ejes (status y priority) por separado
- [ ] Guardar actualiza el libro en Firestore y se refleja en la lista sin
      recargar
- [ ] Las mismas validaciones de la Historia 1 (título y área obligatorios,
      normalización de área) aplican en edición
- [ ] Cambiar el status de un libro no toca su priority, y viceversa

**Edge cases:**
- Editar y vaciar el título → error, no guarda, no pierde los demás cambios
  tipeados en el form
- Cambiar el área a una que no existía → se crea, igual que en alta
- Cancelar la edición sin guardar → el libro queda como estaba

---

**Historia 3:** Como lector, quiero borrar un libro que cargué por error
para mantener mi bitácora prolija.

**Prioridad:** Media

**Criterios de aceptación:**
- [ ] Cada libro tiene una acción de eliminar
- [ ] Se pide confirmación antes de borrar (acción irreversible)
- [ ] Al confirmar, el libro desaparece de la lista sin recargar

**Edge cases:**
- Cancelar la confirmación → el libro no se borra

---

**Historia 4:** Como lector, quiero ver todos mis libros en una lista para
tener una vista general de mi bitácora.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] La lista muestra, por libro: título, autores, área, status, rating (si
      tiene)
- [ ] Si no hay libros todavía, se muestra un estado vacío con mensaje claro
      (no una lista en blanco sin explicación)
- [ ] El orden es consistente entre recargas (por fecha de creación)

**Edge cases:**
- Un libro sin autores → no rompe el layout, simplemente no muestra autores
- Un libro sin rating → no muestra estrellas vacías confusas, se omite o se
  marca claramente como "sin valorar"

---

**Historia 5:** Como lector, quiero filtrar mi lista por estado de lectura
(pendiente/leyendo/leído/abandonado) para enfocarme en lo que me interesa ver.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Hay un filtro por `status` con las 4 opciones + "todos"
- [ ] Aplicar un filtro no recarga la página, solo re-renderiza la lista ya
      cargada en memoria
- [ ] El filtro activo se distingue visualmente (píldora con el acento)

**Edge cases:**
- Filtrar por un status sin resultados → estado vacío específico ("no hay
  libros en leyendo"), no confundir con "no hay libros todavía"

---

**Historia 6:** Como lector, quiero filtrar mi lista por área de conocimiento
para ver cuánto leí de un tema puntual.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Hay un filtro por `area`, generado dinámicamente a partir de las áreas
      que el usuario ya usó (no una lista fija)
- [ ] Se puede combinar con el filtro de status (ej. "Física" + "leyendo")
- [ ] El filtro de área activo se distingue visualmente igual que el de status

**Edge cases:**
- Un área que ya no tiene ningún libro asociado (todos sus libros se
  borraron) → deja de aparecer como opción de filtro, ya que se deriva de
  los libros existentes, no de una colección de áreas separada
