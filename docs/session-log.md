## Sesión 2026-09-18
**Entorno:** Debian 12, Linux topgun, x86_64
**Entrega:** Entrega 1
**Duración aproximada:** una sesión

### Qué se hizo
- Feature: **cuarto valor `sin_especificar` en el eje `priority`** — corrida
  completa de la Software Factory (Researcher → Story Writer → Spec Writer →
  Backend Builder → Frontend Builder → Test Verifier → Validator), aprobada
  sin observaciones
- Normalización defensiva de `priority` en lectura (`normalizePriority()`,
  aplicada en el `.map()` del `onSnapshot`) — en memoria, sin migración batch
- Default inteligente y reactivo en el alta según status
  (`defaultPriorityForStatus()`), con guardas para no acoplar los dos ejes
  (`editingBookId`, `formPriorityTouched`) — verificado que `moveBook()` y la
  edición de libros existentes nunca tocan `priority`
- `CLAUDE.md` y `docs/documentacion.md` actualizados con el enum nuevo y la
  semántica de "no clasificado" para las metas de Fase 2
- Merge `develop` → `main` (fast-forward) y deploy a producción disparado por
  push; preview de `develop` verificado funcionando por el Ingeniero Jefe
  antes del merge

### Decisiones tomadas
- Se detectó y resolvió una contradicción en el brief original: el default
  para `pendiente`/`leyendo` se mantiene en `'curiosidad'` (el implementado
  hoy), no `'interesado'` como decía el pedido — evita un cambio de
  comportamiento no solicitado en el alta
- El default por status es reactivo solo mientras el usuario no toque la
  prioridad a mano, y solo en alta nueva — nunca al editar ni al mover un
  libro en el Kanban
- Sin migración de los documentos existentes en Firestore; conservan su
  `priority` hasta que el usuario los edite manualmente
- Trabajo hecho en `develop` (no directo en `main`), siguiendo el flujo
  documentado de preview → validación manual → merge a producción

### Pendiente para próxima sesión
- [ ] Confirmar en el dashboard de Vercel que el deploy a producción de este
      commit (`4132cd7`) terminó en verde
- [ ] Re-aliasear `marginalia-uy.vercel.app` manualmente (requerido tras cada
      deploy a producción, ver entrada 2026-08-14 más abajo)
- [ ] Gestión completa de áreas de conocimiento (pantalla de administración)
- [ ] Vista de temas/áreas
- [ ] Entrega 2: autocompletado Google Books/OpenLibrary, portadas

### Estado del repo
- Branch: develop
- Último commit: 4132cd7 feat: cuarto valor "sin_especificar" en el eje priority
- Deploy: sí — push a `main` disparó producción; preview de `develop`
  verificado funcionando por el Ingeniero Jefe antes del merge

---

## Sesión 2026-08-14
**Entorno:** Debian 12, Linux topgun, x86_64
**Entrega:** Entrega 1
**Duración aproximada:** — (continuación directa de la sesión de scaffold del 2026-08-13)

### Qué se hizo
- Firebase: creado el proyecto real `marginalia-68e0a` (Auth Google, Firestore,
  Security Rules aplicadas), `firebaseConfig` real cargado en `web/index.html`
- GitHub: repo `m4verick-uy/marginalia` creado y conectado a Vercel (ramas
  `develop`/`produccion`, `produccion` como production branch)
- Vercel: deployment protection (SSO) desactivada, alias de producción
  `marginalia-uy.vercel.app`, dominio de preview de `develop` autorizado en
  Firebase Authorized domains
- Feature: **Login Google + multiusuario** — verificado funcionando en
  producción y preview (se encontró y corrigió un bug real de especificidad
  CSS detectado recién en producción)
- Feature: **Alta manual de libro + lista con filtros** — CRUD completo,
  gestión de áreas mínima e implícita (autocompletado + normalización)
- Feature: **Board Kanban sobre status** — 4 columnas, drag-and-drop, select
  de status como fallback sin drag
- Feature: **Torta de distribución por área** — gráfico en CSS puro
  (conic-gradient), sin agregar dependencias
- Hotfixes: colores de los dots de área (paleta `--p0`–`--p9`) ajustados a
  pedido del Ingeniero Jefe, tras un par de idas y vueltas por ambigüedad en
  el pedido inicial (documentado en los commits correspondientes)

### Decisiones tomadas
- Proyecto Firebase propio para marginaLia, sin compartir infraestructura
  con ReMynder
- Gestión de áreas mínima e implícita en la Entrega 1 (sin pantalla de
  administración completa todavía — queda para una feature futura)
- Portadas (`cover`) quedan sin renderizar hasta la Entrega 2; no se
  adelanta la UI ni se agrega Firebase Storage
- Kanban: 4 estados no lineales (abandonado es una bifurcación) resueltos
  con un `<select>` de status por tarjeta en vez de botones prev/next
  lineales como en ReMynder
- Torta: sin librería de gráficos — CSS puro (`conic-gradient`)

### Pendiente para próxima sesión
- [ ] Gestión completa de áreas de conocimiento (pantalla de administración)
- [ ] Vista de temas/áreas
- [ ] Revisar si CLAUDE.md pide algo más específico para valoración/notas
      más allá del form actual
- [ ] Entrega 2: autocompletado Google Books/OpenLibrary, portadas

### Estado del repo
- Branch: develop
- Último commit: e7beb1c feat: torta de distribución por área
- Deploy: sí — producción (marginalia-uy.vercel.app) y preview (develop),
  ambos verificados funcionando por el Ingeniero Jefe

---

## Sesión 2026-08-13
**Entorno:** Debian 12, Linux topgun, x86_64
**Entrega:** Entrega 1
**Duración aproximada:** —

### Qué se hizo
- Creados los 9 agentes de la Software Factory en `.agents/`, adaptados desde ReMynder
- Inicializado el repositorio Git con ramas `produccion` y `develop`
- Creada la estructura base: `web/index.html` (skeleton con anti-FOUC de tema),
  `web/assets/`, `docs/documentacion.md`, `docs/session-log.md`, `.gitignore`, `vercel.json`

### Decisiones tomadas
- Ramas del repo: `produccion` (default) y `develop` (trabajo activo), igual que ReMynder
- `web/index.html` queda como skeleton vacío — el login y toda funcionalidad se
  implementan feature por feature vía el Orchestrator, no en este scaffold inicial

### Pendiente para próxima sesión
- [ ] Primer run del Orchestrator: alta manual de libro + lista con filtros
- [ ] Login Google + multiusuario (transferir de ReMynder)
- [ ] Gestión de áreas de conocimiento

### Estado del repo
- Branch: develop
- Último commit: (ver git log)
- Deploy: no
