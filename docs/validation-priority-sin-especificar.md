# Validation — priority: valor "sin_especificar"

**Agente:** Validator
**Feature:** cuarto valor `sin_especificar` en el eje `priority`
**Fecha:** 2026-09-18
**Decisión final:** ✅ Aprobado para deploy

---

## Revisión de documentación

- [x] `docs/research-priority-sin-especificar.md` existe y es completo
- [x] `docs/stories-priority-sin-especificar.md` existe y es completo (H1–H4, sin preguntas abiertas)
- [x] `docs/spec-priority-sin-especificar.md` existe y es completo
- [x] `docs/test-report-priority-sin-especificar.md` existe, estado ✅ Aprobado
- [x] `docs/documentacion.md` actualizada — el cambio **sí es arquitectural** (toca el modelo de datos) y tiene su entrada en el historial
- [x] `CLAUDE.md` actualizado: enum en "Modelo de datos", más dos subsecciones nuevas en "Por qué dos ejes" (semántica de `sin_especificar` y tabla de default por status)
- [x] `docs/spec-alta-libro.md` y `docs/research-alta-libro.md` quedaron consistentes con el enum nuevo — no hay documentación contradictoria en el repo

## Revisión de estándares (CLAUDE.md)

- [x] **Código legible** — dos funciones puras de una línea con nombre explícito; los tres comentarios agregados explican el *por qué* (ausencia de intención, no reescritura, ejes desacoplados), no el *qué*
- [x] **Sin regresiones** — Test Verifier revisó las 8 superficies funcionales existentes; `node --check` pasa; el diff no toca `moveBook()`, `submitBookForm()`, `getFilteredBooks()` ni ninguna función de render
- [x] **Seguridad** — sin `innerHTML` nuevo, sin dato de usuario en el markup agregado, Security Rules sin tocar, sin secrets nuevos
- [x] **Diseño consistente** — cero líneas de CSS en el diff. La opción nueva usa el `<select>` ya estilado, con dark/light y mobile ya resueltos
- [x] **Mobile-first** — el layout no cambia; una `<option>` más en un control nativo
- [x] **Sin dependencias nuevas**
- [x] **No mezcla entregas** — todo Entrega 1. El punto de Fase 2 quedó como documentación, sin una línea de lógica de metas
- [x] **Compatible con el roadmap** — `PRIORITIES` y `defaultPriorityForStatus()` son el contrato que la Fase 2 va a consumir; `cover`/`isbn`/`pages` intactos para la Entrega 2
- [x] **Sin funcionalidad social**

### Señales de alerta del Orchestrator — revisadas una por una

| Señal | Veredicto |
|---|---|
| Regresión en funcionalidad existente | No |
| Violación de Security Rules | No — no se tocaron |
| XSS no sanitizado | No — no hay `innerHTML` nuevo |
| **Colapso de los ejes status/priority** | **No** — es el riesgo central de esta feature y está contenido por tres guardas verificadas: `editingBookId === null`, `formPriorityTouched`, y `moveBook()` sin modificar |
| Fragmentación de áreas | No aplica |
| Funcionalidad social | No |
| Mezcla Entrega 1 / Entrega 2 | No |

## Revisión de calidad de código

- [x] Responsabilidad única: `normalizePriority()` valida, `defaultPriorityForStatus()` sugiere, los listeners orquestan. Nada hace dos cosas
- [x] Nombres consistentes con el código existente (`normalizeArea` → `normalizePriority`; `formRatingValue` → `formPriorityTouched`; `STATUSES` → `PRIORITIES`)
- [x] Sin `console.log` nuevos (verificado sobre el diff)
- [x] Sin código comentado
- [x] Se eliminó el literal mágico `'curiosidad'` duplicado en `openBookForm()` — ahora hay una sola fuente del default

## Feedback para el equipo

1. **El brief traía una contradicción y se resolvió antes de escribir código.** Pedía "mantener el default actual (`interesado`)" cuando el default implementado era `'curiosidad'`. Detectarlo en el Researcher y no en el Builder evitó un cambio de comportamiento no deseado en el alta de libros pendientes.
2. **El hallazgo de que `priority` no se renderiza en ninguna vista** redujo el alcance real a tres puntos de código y bajó el riesgo de regresión a prácticamente cero. Vale tenerlo presente: cuando la Fase 2 empiece a mostrar prioridad, esa superficie crece.
3. **La normalización quedó en el único punto de entrada de datos.** Toda feature futura hereda la defensa sin repetirla. Mantener esa disciplina: normalizar en el `onSnapshot`, no en cada vista.
4. **Pendiente de verificación humana antes del merge a producción:** la prueba en preview descrita en el test report. La factory verificó lógica y sintaxis, no el navegador real.
5. Recordatorio de infraestructura: según `docs/documentacion.md`, el alias `marginalia-uy.vercel.app` requiere re-alias manual tras cada deploy a producción.

## Decisión

**Aprobado.** El código cumple los estándares de CLAUDE.md, no introduce
regresiones, no relaja seguridad, no acopla los dos ejes y deja la semántica de
Fase 2 documentada sin adelantar implementación. Listo para merge a `develop`,
verificación en preview y, con el visto bueno del Ingeniero Jefe, promoción a
producción.
