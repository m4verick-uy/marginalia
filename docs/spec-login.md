# Spec — Login Google + multiusuario

**Feature:** Login Google + multiusuario
**Entrega:** Entrega 1
**Stack afectado:** JS, CSS, HTML, Firebase Auth, Firestore (Security Rules)

## Cambios en modelo de datos

Ninguno nuevo — Historia 3 depende de que `users/{uid}/books/{bookId}` (ya
definido en CLAUDE.md) quede protegido por Security Rules desde el día uno,
aunque el CRUD de libros todavía no exista. No se escribe ningún doc de
Firestore en esta feature (no hay seed de libros ni de áreas).

## Cambios en UI

- `#login`: pantalla de login — título "marginaLia", subtítulo, botón
  "Continuar con Google" con el ícono oficial de Google (mismo SVG que
  ReMynder, es el logo de Google, no un asset de marca de ReMynder)
- `#app`: header mínimo — nombre del producto, avatar (`user.photoURL`),
  nombre del usuario (`user.displayName` o `user.email`, pasado por
  `escHtml()`), botón de logout, botón de tema (dark/light, ya existe el
  toggle base heredado del patrón de ReMynder). El resto de `#app` (lista de
  libros, board, etc.) queda vacío — es la próxima feature.
- Variables CSS nuevas: ninguna fuera de las ya sembradas en el skeleton
  (`--bg`, `--surface`, `--border`, `--text`, `--text-dim`, `--accent`)

## Cambios en lógica JS

- `firebaseConfig`: objeto con placeholders (`"REEMPLAZAR_..."`) — proyecto
  Firebase propio de marginaLia, a completar por el Ingeniero Jefe
- `initializeApp`, `getAuth`, `getFirestore`: bootstrap estándar del SDK
- `login()`: `GoogleAuthProvider` + `signInWithPopup`, fallback a
  `signInWithRedirect` en `auth/popup-blocked`, manejo de
  `auth/popup-closed-by-user` / `auth/cancelled-popup-request` sin bloquear
  reintento, mensaje de error visible para cualquier otro código
- `logout()`: `signOut(auth)`
- `getRedirectResult(auth)` al cargar el módulo, para resolver el flujo de
  redirect si ocurrió
- `onAuthStateChanged(auth, user => ...)`: única fuente de verdad — toggle
  entre `#login` y `#app`; en logout limpia cualquier estado de sesión (no
  hay listeners de libros todavía en esta feature, pero la función queda
  preparada para que la próxima feature enganche ahí sus `onSnapshot`)
- `escHtml(str)`: sanitizador XSS, mismo patrón que ReMynder — se aplica a
  `user.displayName` antes de `innerHTML`
- Estado nuevo en variables de módulo: `app`, `auth`, `db` (instancias del
  SDK, no estado de negocio)

## Archivos a modificar

- `web/index.html`: agrega el bootstrap de Firebase, HTML de `#login` y
  header de `#app`, CSS de login/header, wiring de eventos

## Archivos a crear

Ninguno — se mantiene el archivo único `web/index.html` (sin build, sin npm).

## Riesgos técnicos

- **firebaseConfig con placeholders:** el código no será funcional end-to-end
  hasta que el Ingeniero Jefe cree el proyecto Firebase real y pegue los
  valores. Mitigación: dejar los placeholders inconfundibles
  (`"REEMPLAZAR_API_KEY"`, etc.) y documentar exactamente qué reemplazar.
- **Security Rules no versionadas:** igual que ReMynder, se configuran en la
  consola de Firebase, no en el repo. Mitigación: dejar la regla exacta
  documentada en CLAUDE.md (ya está) y en este spec, para copiar/pegar en
  la consola.
- **Auth Google requiere dominio autorizado:** Firebase exige agregar el
  dominio de Vercel (preview y producción) a "Authorized domains" en la
  consola de Auth, o el login falla en producción aunque funcione en local.

## Checklist para crear el proyecto Firebase (a cargo del Ingeniero Jefe)

1. https://console.firebase.google.com → Crear proyecto → nombre sugerido
   `marginalia-<algo único>` (Firebase exige IDs únicos globalmente)
2. Authentication → Sign-in method → habilitar **Google**
3. Firestore Database → Crear base de datos (modo producción)
4. Firestore → Rules → pegar:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /users/{userId}/books/{bookId} {
         allow read, write: if request.auth != null && request.auth.uid == userId;
       }
     }
   }
   ```
5. Configuración del proyecto (ícono ⚙️) → General → "Tus apps" → agregar
   app Web → copiar el objeto `firebaseConfig` completo
6. Authentication → Settings → Authorized domains → agregar el dominio de
   Vercel una vez que exista el deploy (preview y/o producción)
7. Pasarme el `firebaseConfig` para reemplazar los placeholders en
   `web/index.html`

## Criterios de done

- [ ] `web/index.html` tiene el bootstrap de Firebase con `firebaseConfig`
      en placeholders claramente identificables
- [ ] Pantalla de login funcional (popup + fallback redirect + manejo de
      errores) según Historia 1
- [ ] Logout funcional según Historia 2
- [ ] `onAuthStateChanged` es la única fuente de verdad para login vs. app
- [ ] `escHtml()` aplicado a todo dato de usuario en `innerHTML`
- [ ] Documentado en este spec el checklist de Firestore Security Rules
      (Historia 3) para aplicar manualmente en consola
