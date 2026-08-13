# CLAUDE.md — marginaLia

Este archivo es el contexto compartido de todos los agentes que trabajan en este proyecto.
Léelo completo antes de hacer cualquier tarea. No asumas nada que no esté acá.

---

## Qué es marginaLia

Bitácora de lectura personal. Registra qué lees, qué querés leer, y cómo se distribuye
tu lectura entre áreas de conocimiento. El nombre es doble juego: **marginalia** (las notas
que un lector escribe en los márgenes de un libro — glosas, comentarios, valoraciones) y el
guiño fonético en español *margina-LÍA* ≈ "leía". La `L` capitalizada marca ese doble sentido,
igual que la `M` de reMynder.

Es el segundo producto del ecosistema de software personal fundado en Uruguay, hermano de
**ReMynder**. Comparte su filosofía (calidad Apple-like, diseño cuidado) y su stack, pero es
un dominio distinto: no es gestión de tareas, es gestión de conocimiento leído.

**Empresa:** Diamantina
**Estado:** proyecto nuevo (fase 1, MVP)

---

## Visión del producto

- Bitácora **personal**, no red social: sin compartir, sin feed, sin seguir a nadie
- Foco en lo que *uno* quiere leer y ya leyó, con mínima fricción
- Elegante, sencilla, sin distraer: el Excel actual tiene límites (no filtra bien, no visualiza)
  y marginaLia los resuelve manteniendo la simplicidad
- El diferencial frente a Goodreads: **integración con áreas de conocimiento** y visualización
  de cobertura por área — no es un catálogo social, es un mapa de lo que sabés y querés saber
- Filosofía de producto heredada de ReMynder: **calidad sobre velocidad**, referente Apple

---

## Stack (heredado de ReMynder — no migrar sin pedido explícito)

| Capa | Tecnología |
|---|---|
| Frontend | HTML + CSS + JavaScript vanilla (sin framework, sin build) |
| Auth | Firebase Authentication — Google OAuth 2.0 |
| Base de datos | Cloud Firestore (NoSQL, tiempo real) |
| Hosting | Vercel (deploy estático, integración Git) |
| Fuentes | Pila de sistema, sin Google Fonts (`-apple-system, BlinkMacSystemFont, 'SF Pro Text', system-ui, sans-serif`) |
| Firebase SDK | CDN gstatic.com (módulos ES) |
| API externa | Google Books API u OpenLibrary (fase 1 – entrega 2, para autocompletado y portadas) |

**No hay backend propio. No hay npm. No hay proceso de build.**
Un único `web/index.html` que el browser ejecuta directamente, igual que ReMynder.

---

## Modelo de datos (Firestore)

Decisión de arquitectura central: el estado de un libro tiene **dos ejes independientes**,
no uno. Esto viene de cómo el usuario piensa la lectura y no debe colapsarse en un solo campo.

```
users/
  {uid}/
    books/
      {bookId}/
        title:      string    — título del libro
        authors:    string[]   — autores (soporta coautoría: ["José Edelstein","Andrés Gomberoff"])
        area:       string    — área de conocimiento (dimensión analítica central)
        subarea:    string    — subárea (opcional)
        status:     string    — EJE DE PROGRESO: "pendiente" | "leyendo" | "leido" | "abandonado"
        priority:   string    — EJE DE INTENCIÓN: "curiosidad" | "interesado" | "must_have"
        rating:     number    — valoración 0–5 (0 = sin valorar)
        notes:      string    — marginalia: notas del lector
        cover:      string    — URL de portada (de la API o manual; vacío = placeholder)
        isbn:       string    — ISBN (de la API, editable)
        pages:      number    — número de páginas (de la API, editable)
        startedAt:  number    — timestamp Unix, inicio de lectura (opcional)
        finishedAt: number    — timestamp Unix, fin de lectura (opcional)
        createdAt:  number     — timestamp Unix (Date.now())
```

### Por qué dos ejes

- **`status`** responde "¿en qué punto de lectura está?" → alimenta el **board Kanban**.
- **`priority`** responde "¿cuánto lo quiero?" → alimenta las **metas de conocimiento** (fase 2).
- Son ortogonales: un libro puede ser `must_have` + `pendiente` (lo quiero sí o sí, no empecé),
  o `curiosidad` + `leido` (lo leí por curiosidad). Un solo campo perdería ese cruce, que es
  justo lo que el usuario quiere poder filtrar ("mis must-have que no empecé").

### Por qué `area` es de primera clase

`area` no es un tag decorativo: es la dimensión sobre la que se calcula la **torta de cobertura
por áreas** (entrega 1) y las **metas por tema** (fase 2). Debe estar bien modelada desde el MVP
para no migrar datos después. `authors` es array desde el día uno por la misma razón: la coautoría
es real (Edelstein + Gomberoff) y meterla como string único obliga a migrar luego.

### Áreas de conocimiento

Lista controlada de áreas (Física, Biología, Neurociencia, Historia, Filosofía, Literatura,
Economía, etc.), editable por el usuario. Mismo principio que las categorías de ReMynder, pero
aquí **dinámicas desde el inicio** (no hardcodeadas) porque son el eje analítico. Evitar que se
fragmenten ("Física" vs "física" vs "Cosmología" suelta) — validar contra la lista existente.

---

## Alcance — MVP partido en dos entregas

El MVP es deliberadamente incremental. **Toda tarea entra por el Orchestrator, una entrega a la vez.**

### Entrega 1 — Base sólida (sin dependencias externas)

- Login Google + multiusuario (transfiere de ReMynder tal cual)
- Alta manual de libro con todos los campos del modelo, todos editables
- Gestión de áreas de conocimiento (lista dinámica, validada)
- Lista de libros con filtro por `status` y por `area`
- Board Kanban sobre `status` (transfiere del `.board` de ReMynder casi literal)
- Vista de temas/áreas
- Torta de distribución por área (% de libros por área) — permite ver cobertura actual
- Valoración (rating) y notas (marginalia) por libro

### Entrega 2 — Enriquecimiento (con API externa)

- Autocompletado en el alta desde Google Books / OpenLibrary: escribir título → trae
  autores, portada, ISBN, páginas
- **Todo lo autocompletado es editable** — la API puede fallar, no encontrar el libro, o
  traer datos incorrectos. El usuario siempre puede corregir cada campo manualmente.
- Manejo de estados de error: API caída, libro no encontrado, portada rota (placeholder)
- Portadas de libros → grilla tipo estantería (cambia el diseño: entran imágenes)

### Fase 2 — Fuera del MVP (no construir todavía)

- Metas de conocimiento: definir objetivos por área/cantidad ("leer N libros de Física")
  y seguir su progreso — se calcula sobre `priority` + `area` + `status`
- Estadísticas temporales: libros/año, páginas/año
- Rachas de lectura

**Sin redes sociales, sin compartir, sin seguir usuarios — en ninguna fase por ahora.**

---

## Convenciones de código (heredadas de ReMynder)

### JavaScript
- Vanilla ES modules, sin transpilación
- async/await para todas las operaciones Firestore
- Estado en variables de módulo
- `render()` recalcula la UI desde el estado
- Delegación de eventos en listas (no listeners por ítem)
- Sanitización XSS con `escHtml()` antes de todo `innerHTML` — **incluye títulos, autores y
  notas de libros, y cualquier dato que venga de la API externa**

### CSS

**Regla de jerarquía visual (no negociable, heredada de ReMynder):** un control secundario no
debe ser descendiente de un elemento con tratamiento visual de acción primaria. Usar `>` (hijo
directo) en selectores de botones de acción primaria salvo que el selector deba aplicar a
descendientes intencionalmente. Un selector como `.add-row button` (sin `>`) captura todo
`<button>` anidado y gana la cascada por especificidad. Esto costó rondas de debug en ReMynder.

- Variables CSS en `:root` para theming (dark por defecto, `html[data-theme="light"]` override)
- Un único token de marca `--accent`, recalibrado por tema para contraste AA
- Portadas: manejar aspect ratio de libro (~2:3) con placeholder consistente cuando falta imagen
- Paleta de área análoga a la paleta de categoría de ReMynder (`--p0`–`--pN`, mapeo por índice);
  el color de área vive solo en dots/indicadores, no en píldoras de filtro
- Radios, superficies, bordes y texto derivado del fondo: mismo sistema que ReMynder

### HTML
- Root divs con `display:none` al inicio; `onAuthStateChanged` decide cuál mostrar (anti-FOUC)
- Anti-FOUC de tema: script síncrono en `<head>` aplica `data-theme` antes del render

### Tipografía
- Fuente única de sistema, sin Google Fonts; jerarquía por tamaño/peso/color, no por familia

---

## Reglas de seguridad (Firestore)

```
match /users/{userId}/books/{bookId} {
  allow read, write: if request.auth != null && request.auth.uid == userId;
}
```

Cada usuario solo lee y escribe sus propios libros. Esta regla NO se relaja.
El `apiKey` de Firebase es público por diseño — la seguridad está en las Security Rules.

---

## Estándares de calidad (NO negociables, heredados de ReMynder)

1. **Código legible** — cualquier dev lo entiende sin preguntar
2. **Sin regresiones** — cada cambio preserva el comportamiento existente
3. **Seguridad primero** — sanitización XSS en todo `innerHTML` (incluida data de la API), Rules intactas
4. **Diseño consistente** — usar las variables CSS existentes, no inventar sin justificación
5. **Mobile-first** — funciona perfecto en móvil
6. **Sin dependencias innecesarias** — agregar una lib requiere justificación explícita
7. **Documentar cambios** — actualizar la doc con cada cambio arquitectural

---

## Para los agentes: cómo trabajar en este proyecto

- **Leer este archivo completo** antes de cualquier tarea
- **Preguntar antes de asumir** si algo no está claro aquí
- **Respetar el stack** — no proponer migraciones a frameworks sin pedido explícito
- **Pensar en las fases** — toda solución debe ser compatible con la Entrega 2 y la Fase 2
  (ej.: el modelo de datos ya contempla `cover`, `isbn`, `pages` aunque la Entrega 1 no use API)
- **Priorizar calidad** — producto con aspiración comercial, no prototipo

---

## Flujo de trabajo — Software Factory (idéntico a ReMynder)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  COMANDOS DE SESIÓN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

INICIO (siempre primero):
"Actúa como Session Start de marginaLia"

CIERRE (siempre al final):
"Actúa como Session End de marginaLia"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  TIPOS DE TAREA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FEATURE — algo nuevo para el usuario:
"Actúa como Orchestrator de marginaLia.
Feature: [descripción]"

HOTFIX — algo roto:
"Actúa como Orchestrator de marginaLia.
Hotfix [visual/lógico/datos]: [descripción]
Límites: [qué NO tocar]"

REFACTOR — mejorar sin cambiar comportamiento:
"Actúa como Orchestrator de marginaLia.
Refactor: [qué y por qué]
Límites: el comportamiento visible no debe cambiar"

RESEARCH — explorar antes de decidir:
"Actúa como Orchestrator de marginaLia.
Research: [pregunta]
Solo análisis, sin implementar nada."

DECISION — criterio antes de actuar:
"Actúa como Orchestrator de marginaLia.
Decisión pendiente: [contexto y opciones]
Dame tu recomendación antes de implementar."

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  REGLAS DE ORO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

- Sin Session Start no se trabaja
- Sin Validator aprobado no hay deploy
- Sin Session End no se cierra
- Toda tarea entra SIEMPRE por el Orchestrator
- Nunca directo a Claude Code sin pasar por el Orchestrator
- Una entrega a la vez: no mezclar Entrega 1 y Entrega 2 en un mismo run

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  AGENTES DISPONIBLES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

.agents/session-start.md
.agents/orchestrator.md
.agents/researcher.md
.agents/story-writer.md
.agents/spec-writer.md
.agents/backend-builder.md
.agents/frontend-builder.md
.agents/test-verifier.md
.agents/validator.md
.agents/session-end.md
