# Stories — priority: valor "sin_especificar"

**Agente:** Story Writer
**Fecha:** 2026-09-18
**Input:** docs/research-priority-sin-especificar.md
**Entrega:** Entrega 1

---

## Decisiones del Ingeniero Jefe ya tomadas (no son preguntas abiertas)

1. Default de `priority` para `pendiente`/`leyendo`: **`curiosidad`** — el que
   ya está implementado. No se cambia.
2. Default de `priority` para `leido`/`abandonado`: **`sin_especificar`**.
3. El default es **reactivo hasta que el usuario toque la prioridad**, y solo
   en el alta de un libro nuevo.
4. Sin migración batch de los libros existentes.

No quedan preguntas abiertas para el Ingeniero Jefe en esta feature.

---

## H1 — La app nunca se rompe con una prioridad vieja o ausente

**Historia:** Como lector con libros cargados desde antes, quiero que la app
los siga mostrando y editando con normalidad aunque su prioridad sea un valor
que la app ya no conoce, para no perder ni ver corrompidos los datos que
vengo acumulando.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Un libro cuyo documento no tiene el campo `priority` se lista, se abre en
      el formulario y se puede guardar sin errores
- [ ] Un libro con `priority` vacía, `null` o con un valor desconocido se
      comporta igual: la app lo trata como `sin_especificar`
- [ ] Al abrir un libro así en el formulario, el selector de prioridad muestra
      "Sin especificar" — nunca queda en blanco
- [ ] Abrir la app **no modifica ningún documento en Firestore**: un libro que
      el usuario no editó conserva en la base exactamente el valor que tenía
- [ ] Un libro con una prioridad válida (`curiosidad`, `interesado`,
      `must_have`) la conserva intacta, en la app y en la base

**Edge cases:**
- Documento sin el campo `priority` (libro cargado antes de que existiera)
- `priority: ""` guardada por un selector que quedó en blanco
- `priority: "must-have"` u otra variante tipográfica no contemplada
- Usuario sin ningún libro cargado: la app no debe romper al no tener qué normalizar

---

## H2 — Puedo elegir "sin especificar" en el formulario

**Historia:** Como lector, quiero poder marcar explícitamente que un libro no
tiene una intención de lectura asociada, para no tener que inventarle un nivel
de deseo a algo que ya leí o que cargué solo como registro.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] El selector de prioridad ofrece cuatro opciones: Curiosidad, Interesado,
      Must-have y **Sin especificar**
- [ ] La etiqueta visible está en español ("Sin especificar") y el valor
      almacenado es `sin_especificar`
- [ ] "Sin especificar" aparece **al final** de la lista, después de Must-have:
      no es un escalón de la escala de deseo, es su ausencia
- [ ] Elegirla y guardar persiste `sin_especificar` en el documento del libro
- [ ] Reabrir ese libro muestra "Sin especificar" seleccionada
- [ ] El selector se ve y funciona igual que antes en móvil y en desktop, en
      tema oscuro y claro

**Edge cases:**
- Elegir "Sin especificar" en un libro que antes era `must_have` (debe pisarlo)
- Elegirla en un libro `pendiente` (es una elección válida, no se corrige sola)

---

## H3 — La prioridad arranca en el valor que tiene sentido para el estado

**Historia:** Como lector que registra libros que ya terminó, quiero que la
prioridad venga por defecto en "Sin especificar" cuando marco el libro como
leído o abandonado, para no tener que corregir a mano un campo que en ese caso
no significa nada.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] En un alta nueva, cambiar el estado a "Leído" o "Abandonado" pone la
      prioridad en "Sin especificar"
- [ ] En un alta nueva, cambiar el estado a "Pendiente" o "Leyendo" pone la
      prioridad en "Curiosidad" (el default de siempre)
- [ ] Si el usuario elige una prioridad a mano, cambiar el estado **ya no la
      pisa** por el resto de ese formulario
- [ ] Abrir el formulario para un libro nuevo lo deja en Pendiente + Curiosidad,
      igual que hoy
- [ ] **Al editar un libro existente, cambiar su estado nunca toca su prioridad**
- [ ] Cerrar y volver a abrir el formulario arranca limpio: el default vuelve a
      ser reactivo
- [ ] **Mover un libro en el Kanban (drag & drop o el selector de estado de la
      tarjeta) nunca cambia su prioridad**

**Edge cases:**
- Usuario elige "Must-have", después cambia el estado a "Leído" → la prioridad
  se queda en Must-have (decisión deliberada del usuario, se respeta)
- Usuario cambia el estado a "Leído", después a "Pendiente", sin tocar la
  prioridad → sigue el default de cada estado (sin_especificar, luego curiosidad)
- Usuario elige una prioridad, la vuelve a cambiar, y recién ahí cambia el
  estado → la prioridad no se pisa (ya la tocó)
- Cargar un libro, guardarlo, y abrir el formulario de nuevo para otro libro →
  el segundo arranca con el default reactivo intacto

---

## H4 — La semántica queda escrita donde se busca

**Historia:** Como equipo que va a construir las metas de conocimiento en la
Fase 2, quiero que el significado de "sin_especificar" esté documentado en el
contexto compartido, para que nadie lo sume por error a un objetivo de lectura.

**Prioridad:** Media

**Criterios de aceptación:**
- [ ] `CLAUDE.md` lista los cuatro valores de `priority` en el modelo de datos
- [ ] `CLAUDE.md` documenta la regla de default por estado en el alta
- [ ] `CLAUDE.md` deja explícito que `sin_especificar` es **ausencia de
      intención**: no cuenta como `must_have` ni como ningún nivel de deseo, y
      las metas de Fase 2 deben tratarlo como "no clasificado"
- [ ] `CLAUDE.md` deja explícito que el default por estado es una sugerencia del
      formulario de alta y **no acopla los dos ejes**
- [ ] `docs/spec-alta-libro.md` y `docs/research-alta-libro.md` quedan
      consistentes con el enum nuevo
- [ ] **No se construye ninguna UI ni lógica de metas en este run**

**Edge cases:** ninguno (documentación).

---

## Fuera de alcance (explícito)

- UI o lógica de metas de conocimiento (Fase 2)
- Filtro por prioridad, badge o dot de prioridad en la lista o el Kanban
- Cualquier cambio al eje `status`
- Migración o reescritura en batch de documentos existentes
