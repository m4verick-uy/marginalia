# Research — Login Google + multiusuario

## Resumen del problema

marginaLia necesita autenticación para poder aislar los libros de cada lector
en `users/{uid}/books/`. CLAUDE.md especifica: "Login Google + multiusuario
(transfiere de ReMynder tal cual)" como primer ítem de la Entrega 1.

Se investigó la implementación real de ReMynder (`/home/m4verick/Proyects/reMynder/web/index.html`)
para portar el mecanismo, no reinventarlo.

## A qué entrega pertenece

Entrega 1. No depende de la API externa de libros (Entrega 2) ni de metas de
conocimiento (Fase 2). Es prerequisito de todo lo demás: sin `uid` no hay
`users/{uid}/books/`.

## Cómo lo resuelve ReMynder (patrón a portar)

- Firebase SDK vía CDN gstatic.com, ES modules: `firebase-app.js`,
  `firebase-auth.js`, `firebase-firestore.js` (v10.12.0)
- `firebaseConfig` hardcodeado en el JS (apiKey público por diseño — la
  seguridad vive en las Security Rules, no en ocultar el config)
- Google Sign-In vía `GoogleAuthProvider` + `signInWithPopup`, con fallback
  a `signInWithRedirect` si el popup es bloqueado (`auth/popup-blocked`), y
  manejo explícito de `auth/popup-closed-by-user` / `auth/cancelled-popup-request`
- `getRedirectResult(auth)` se llama una vez al cargar la página para
  resolver el flujo de redirect
- `onAuthStateChanged(auth, user => ...)` es la única fuente de verdad para
  mostrar login vs. app: si `user` existe, oculta `#login-screen` y muestra
  `#app`; si no, la inversa. Además dispara/cancela los listeners de Firestore
  (`startListeners(uid)` / `unsubTasks()`, `unsubCats()`)
- Botón de logout simple: `signOut(auth)`
- Anti-FOUC: ambos root divs (`#login-screen`, `#app`) arrancan con
  `style="display:none"` en el HTML; recién `onAuthStateChanged` decide cuál
  mostrar — evita el flash de contenido incorrecto
- Anti-FOUC de tema: script síncrono en `<head>` que lee `localStorage` y
  setea `data-theme` antes de pintar
- `escHtml()` sanitiza cualquier texto de usuario antes de `innerHTML` (no
  aplica directamente al login, pero sí a `user.displayName` si se muestra)
- No hay un archivo `firestore.rules` en el repo de ReMynder — las Security
  Rules se administran directamente en la consola de Firebase, no versionadas

## Diferencias necesarias para marginaLia

1. **Proyecto Firebase propio.** El `firebaseConfig` de ReMynder apunta al
   proyecto `pendientes-2c0ea`, exclusivo de esa app. marginaLia es "un
   dominio distinto" (CLAUDE.md) con su propia colección `users/{uid}/books/`
   — necesita su propio proyecto Firebase (o al menos su propia app dentro de
   un proyecto), para no mezclar usuarios/datos con ReMynder.
   ⚠️ Esto es una decisión de infraestructura que excede lo que un agente de
   código puede resolver solo: requiere crear el proyecto en la consola de
   Firebase (o Google Cloud) y obtener el `firebaseConfig` real.

2. **Sin concepto de ADMIN_UID.** ReMynder tiene un `ADMIN_UID` hardcodeado
   para el seed de categorías del usuario admin (m4verick). marginaLia no
   tiene un equivalente definido en CLAUDE.md — no hay mención de seed de
   áreas de conocimiento por usuario. Ver pregunta abierta en
   docs/stories-login.md.

3. **Vista post-login distinta.** ReMynder muestra `#app` (tareas). marginaLia
   mostrará la lista/board de libros — pero esa es una feature aparte
   (alta manual + lista con filtros). Para esta feature, el post-login debe
   dejar un contenedor `#app` mínimo (header con nombre/avatar/logout) sin
   construir todavía la UI de libros.

## Patrones y convenciones a respetar

- HTML: root divs con `display:none` al inicio, `onAuthStateChanged` decide
  cuál mostrar (regla explícita de CLAUDE.md, ya sembrada en el skeleton
  actual de `web/index.html`)
- CSS: variables en `:root`, override `html[data-theme="light"]`, fuente de
  sistema única (marginaLia NO usa DM Mono/DM Sans como ReMynder — usa
  `-apple-system, BlinkMacSystemFont, 'SF Pro Text', system-ui, sans-serif`
  para todo, jerarquía por tamaño/peso/color)
- JS: async/await, estado en variables de módulo, escHtml() en todo innerHTML
- Un único token de marca `--accent` (marginaLia todavía no tiene uno
  definido — se puede reutilizar el placeholder `#6d8cff`/`#3a5cff` ya
  sembrado en el skeleton, o el Ingeniero Jefe puede definir uno propio)

## Riesgos o conflictos detectados

- **Bloqueante de infraestructura:** sin un proyecto Firebase propio (o
  credenciales reales) no hay login funcional para probar — el código puede
  quedar implementado y correcto, pero no verificable end-to-end hasta que
  exista un `firebaseConfig` real.
- Sin Security Rules versionadas en el repo (mismo patrón que ReMynder) —
  el Validator no puede confirmar `allow read, write: if uid == userId` leyendo
  código; queda como checklist manual a aplicar en la consola de Firebase.
- Riesgo de reusar el mismo proyecto Firebase de ReMynder "para ir más rápido"
  — rechazar esa opción salvo pedido explícito: mezclaría usuarios y reglas
  de dos productos distintos.

## Recomendaciones para los siguientes agentes

- Story Writer: flaggear como ⚠️ DECISIÓN REQUERIDA la creación/datos del
  proyecto Firebase de marginaLia, y confirmar si existe algún concepto de
  usuario admin/seed de áreas iniciales (equivalente al ADMIN_UID de ReMynder)
- Spec Writer: especificar `firebaseConfig` como placeholder editable, dejando
  explícito qué campos hay que reemplazar y dónde
- Backend Builder: portar el mecanismo de auth (popup + fallback redirect +
  onAuthStateChanged) tal cual, adaptado a los IDs de marginaLia
- Frontend Builder: pantalla de login con la identidad visual de marginaLia
  (no reusar el título "reMynder"), manteniendo el botón de Google idéntico
