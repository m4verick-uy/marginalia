# Stories — Board Kanban sobre status

Basado en docs/research-kanban.md.

---

**Historia 1:** Como lector, quiero cambiar entre vista de Lista y vista de
Tablero para elegir cómo prefiero ver mi bitácora en cada momento.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Hay un selector de 2 vistas (Lista / Tablero) visible en el header
- [ ] La vista elegida persiste entre sesiones (recargar la página mantiene
      la última vista usada)
- [ ] Cambiar de vista no pierde el filtro de área activo

**Edge cases:**
- Primera vez que se abre la app (sin preferencia guardada) → arranca en Lista

---

**Historia 2:** Como lector, quiero ver mis libros organizados en columnas
por estado de lectura para tener una vista rápida de mi progreso.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] 4 columnas: Pendiente, Leyendo, Leído, Abandonado
- [ ] Cada tarjeta muestra al menos título y área
- [ ] Una columna sin libros sigue visible (no desaparece)
- [ ] El filtro de área (de la feature anterior) también funciona en Tablero
- [ ] El filtro de status no se muestra en la vista Tablero (el status ya es
      el eje de las columnas — mostrarlo sería redundante)

**Edge cases:**
- Ningún libro coincide con el área filtrada → las 4 columnas quedan vacías
  con su mensaje de "soltá acá" o equivalente, no un error

---

**Historia 3:** Como lector, quiero arrastrar un libro de una columna a otra
para actualizar su estado de lectura rápidamente.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Arrastrar una tarjeta a otra columna actualiza su `status` en Firestore
- [ ] El cambio se refleja sin recargar la página
- [ ] Soltar en la misma columna no genera una escritura innecesaria
- [ ] Arrastrar funciona entre cualquier par de columnas (no solo adyacentes) —
      ej. de Pendiente directo a Abandonado

**Edge cases:**
- Soltar fuera de cualquier columna → no pasa nada, la tarjeta vuelve a su lugar

---

**Historia 4:** Como lector que usa el celular (donde arrastrar es incómodo),
quiero cambiar el estado de un libro sin necesidad de drag-and-drop.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Cada tarjeta del board tiene un control (select) para elegir
      directamente cualquiera de los 4 estados
- [ ] Cambiar el valor actualiza el libro igual que el drag-and-drop
- [ ] En mobile, las columnas se navegan con scroll horizontal tipo carrusel,
      con indicadores (dots) de a cuál columna se está mirando

**Edge cases:**
- Elegir en el select el mismo status que ya tiene el libro → no genera
  escritura innecesaria a Firestore
