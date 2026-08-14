# Test Report — Login Google + multiusuario

**Feature:** Login Google + multiusuario
**Estado general:** ⚠️ Aprobado con observaciones

## Criterios de aceptación

**Historia 1 — iniciar sesión con Google**
- [x] Pantalla de login con nombre del producto y botón "Continuar con Google": ok
- [x] Click abre `signInWithPopup`: ok
- [x] Login exitoso oculta `#login`, muestra `#app` (vía `onAuthStateChanged`): ok
- [x] Cierre de popup sin completar → mensaje, sin bloquear reintento (`btn.disabled = false`): ok
- [x] Popup bloqueado → fallback a `signInWithRedirect`: ok

**Historia 2 — cerrar sesión**
- [x] Botón de logout visible en el header: ok
- [x] `signOut(auth)` dispara `onAuthStateChanged` → vuelve a `#login`: ok
- [x] Al volver a login, `user-avatar` y `user-name` se limpian: ok

**Historia 3 — privacidad de datos**
- [x] Modelo de datos apunta a `users/{uid}/books/` (sin cambios, ya definido en CLAUDE.md): ok
- [ ] Security Rules aplicadas en la consola de Firebase: **no verificable desde el código** —
      depende de que el Ingeniero Jefe las pegue en la consola del proyecto real
      (regla documentada en docs/spec-login.md y en CLAUDE.md)
- [x] No hay acceso a `users/{uid}/books/` sin sesión: la app no monta ningún
      listener de Firestore en esta feature (se agrega en la próxima), así que
      no hay superficie de datos expuesta todavía

## Regresiones detectadas

Ninguna — es la primera feature implementada sobre el skeleton, no hay
funcionalidad previa que romper.

## Edge cases verificados

- Popup bloqueado → redirect: ok (código idéntico al patrón probado en ReMynder)
- Cierre de popup / cancelación → mensaje sin bloqueo: ok
- Refresco de página logueado → `onAuthStateChanged` restaura sesión automáticamente: ok
- Error de red/Google genérico → mensaje de error visible con `err.code`: ok
- `getRedirectResult` con `auth/no-auth-event` → silenciado correctamente (no es un error real): ok

## Seguridad

- [x] `escHtml()` está definido y disponible para innerHTML futuro (ninguna
      asignación de esta feature usa `innerHTML`; `user-name` usa `textContent`,
      que ya escapa por sí solo — usar `escHtml()` antes de `textContent`
      hubiera producido doble-escape, ej. "&" → "&amp;" visible en pantalla.
      **Bug encontrado y corregido durante la verificación**: se removió el
      `escHtml()` redundante en la asignación de `user-name`)
- [x] Firestore rules no modificadas en código (no versionadas, según patrón de ReMynder)
- [x] No hay API keys reales expuestas — `firebaseConfig` está en placeholders
      explícitos (`"REEMPLAZAR_..."`)
- [x] Login/logout funcionan según lo esperado
- [x] No hay forma de que un usuario vea datos de otro en esta feature (no hay
      lectura de datos todavía)

## Problemas encontrados

- **Corregido en esta verificación (menor):** doble-escape de `user-name` por
  uso incorrecto de `escHtml()` antes de `textContent`. Ya resuelto.
- **No bloqueante (esperado):** `firebaseConfig` con placeholders — el login
  no es funcional end-to-end hasta que se cree el proyecto Firebase real y se
  reemplacen los valores. Documentado en docs/spec-login.md como riesgo conocido.

## Recomendaciones

- Antes de dar por cerrada la feature en la práctica, pegar el `firebaseConfig`
  real y probar el flujo completo en un navegador (popup + logout + refresh)
- Aplicar las Security Rules en la consola de Firebase antes de escribir la
  primera colección real de libros
- Agregar el dominio de Vercel a "Authorized domains" en Firebase Auth antes
  del primer deploy a producción
