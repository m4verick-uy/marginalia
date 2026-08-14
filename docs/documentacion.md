# Documentación arquitectural — marginaLia

Este archivo se actualiza con cada cambio arquitectural, a cargo del Validator
al cerrar cada feature. Ver CLAUDE.md para la visión de producto y el modelo
de datos completo.

## Estado actual

Fase 1, MVP, Entrega 1. Features implementadas: Login Google + multiusuario,
Alta manual de libro + lista con filtros.

## Nota de alcance: portadas (`cover`)

El campo `cover` (URL de portada) existe en el form de alta/edición desde la
Entrega 1 (CLAUDE.md pide "todos los campos del modelo, todos editables"),
pero **no se renderiza en ningún lado todavía** — no hay `<img>` conectado a
ese campo. Es intencional, no un bug: CLAUDE.md escopea explícitamente la
"grilla tipo estantería" con portadas a la Entrega 2, junto con el
autocompletado de Google Books/OpenLibrary (que va a traer la URL de portada
automáticamente). Decisión confirmada con el Ingeniero Jefe el 2026-08-14:
no adelantar esto a la Entrega 1, y no agregar Firebase Storage / subida de
archivos — `cover` sigue siendo una URL, tal como especifica el modelo de
datos.

## Proyecto Firebase

marginaLia usa un **proyecto Firebase propio**, separado del de ReMynder —
decisión tomada el 2026-08-13 (ver docs/stories-login.md, sección Decisiones).
Auth Google + Firestore, sin infraestructura compartida entre productos.

Proyecto real: `marginalia-68e0a`. `firebaseConfig` en `web/index.html` tiene
los valores reales (2026-08-13). Google habilitado en Authentication,
Firestore Database creada, Security Rules de `users/{userId}/books/{bookId}`
aplicadas — **login verificado funcionando el 2026-08-14** tanto en
producción como en preview.

Authorized domains en Firebase:
- `marginalia-uy.vercel.app` (producción)
- `marginalia-git-develop-m4vericks-projects.vercel.app` (preview de `develop`
  — dominio estable por rama, no cambia en cada deploy)

Nota: cada deployment individual de Vercel tiene además su propia URL
efímera (`marginalia-<hash>-m4vericks-projects.vercel.app`) que no está
autorizada — el login solo funciona en los dos dominios estables de arriba.

## Infraestructura de hosting y Git

- Repo: `github.com/m4verick-uy/marginalia` (default branch: `produccion`)
- Ramas: `develop` (trabajo activo) y `produccion` (deploy a producción),
  ambas conectadas a Vercel — push a `produccion` despliega a producción,
  push a `develop` genera preview
- Proyecto Vercel: `m4vericks-projects/marginalia`
- URL de producción: **https://marginalia-uy.vercel.app** (alias manual;
  `marginalia.vercel.app` a secas ya estaba tomado por otro proyecto global)
- Vercel Deployment Protection (SSO) estaba habilitada por defecto en el
  proyecto nuevo y bloqueaba el acceso público — se desactivó
  (`vercel project protection disable marginalia --sso`) para que el login
  sea accesible por cualquier usuario, no solo por cuentas del team de Vercel

## Historial de cambios arquitecturales

### 2026-08-14 — Alta manual de libro + lista con filtros
- CRUD completo de `users/{uid}/books/`: `addBook`, `updateBook`, `deleteBook`,
  listener reactivo con `onSnapshot(query(..., orderBy('createdAt')))`
- Form único de alta/edición con todos los campos del modelo (incluidos
  `cover`, `isbn`, `pages`, `startedAt`, `finishedAt` aunque Entrega 1 no
  tenga autocompletado de API — se cargan a mano si el usuario quiere)
- **Gestión de áreas mínima e implícita**: sin colección `areas` separada:
  se derivan de `books` en memoria (`getAreaOptions()`). El form tiene
  autocompletado (`<datalist>`) + normalización case-insensitive
  (`normalizeArea()`) para evitar fragmentación ("Física" vs "física"). No
  hay pantalla de administración de áreas todavía — decisión de alcance
  documentada en docs/research-alta-libro.md, queda pendiente para una
  feature futura si hace falta renombrar/fusionar áreas ya creadas
  independientemente de tener libros cargados
- Paleta de área `--p0`–`--p9` agregada a `:root` (dark + light), color de
  área solo en dots/badges, nunca en píldoras de filtro (regla de CLAUDE.md)
- Filtros combinables por `status` y `area`, en memoria (sin re-consultar Firestore)
- Docs: docs/research-alta-libro.md, docs/stories-alta-libro.md,
  docs/spec-alta-libro.md, docs/test-report-alta-libro.md,
  docs/validation-alta-libro.md
- 2 bugs menores encontrados y corregidos en la propia revisión (ver
  test-report-alta-libro.md): timezone en fechas de inicio/fin de lectura,
  CSS faltante para `.book-card-main`
- Pendiente: verificación manual en navegador (no se hizo en esta sesión)

### 2026-08-13 — Login Google + multiusuario
- Bootstrap de Firebase SDK (v10.12.0, CDN gstatic) en `web/index.html`:
  `initializeApp`, `getAuth`, `getFirestore`
- Auth: `GoogleAuthProvider` + `signInWithPopup` con fallback a
  `signInWithRedirect`, `onAuthStateChanged` como única fuente de verdad para
  mostrar `#login` vs `#app`
- `escHtml()` sembrado como utilidad compartida para sanitizar innerHTML en
  las próximas features
- Docs: docs/research-login.md, docs/stories-login.md, docs/spec-login.md,
  docs/test-report-login.md, docs/validation-login.md
- Pendiente de infraestructura (no bloquea el código, sí el uso real): crear
  el proyecto Firebase, pegar el `firebaseConfig` real, aplicar Security
  Rules en consola, autorizar dominio de Vercel
