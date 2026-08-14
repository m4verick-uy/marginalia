# Stories — Login Google + multiusuario

Basado en docs/research-login.md.

---

**Historia 1:** Como lector, quiero iniciar sesión con mi cuenta de Google
para que mis libros queden guardados de forma privada y asociados solo a mí.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Al entrar a marginaLia sin sesión activa, veo una pantalla de login con
      el nombre del producto y un botón "Continuar con Google"
- [ ] Al tocar el botón, se abre el flujo estándar de Google Sign-In
- [ ] Si el login es exitoso, veo la app (header con mi nombre/avatar) y ya
      no veo la pantalla de login
- [ ] Si cierro el popup de Google sin completar el login, vuelvo a ver el
      botón de login sin error bloqueante
- [ ] Si el navegador bloquea el popup, el login continúa por redirect en
      vez de fallar en silencio

**Edge cases:**
- Popup bloqueado por el navegador → fallback a redirect
- Usuario cierra el popup a mitad de camino → puede reintentar sin recargar
- Refresco de página estando logueado → sigo logueado (no vuelvo a ver login)
- Error de red o de Google durante el login → mensaje de error visible, no
  pantalla en blanco

---

**Historia 2:** Como lector, quiero cerrar sesión para dejar de ver mis
libros en un dispositivo que no es mío.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Estando logueado, veo un botón/enlace de logout visible en el header
- [ ] Al tocarlo, se cierra la sesión y vuelvo a ver la pantalla de login
- [ ] Después de logout, no queda ningún dato del usuario anterior visible
      en pantalla

**Edge cases:**
- Logout mientras hay una operación de datos en curso (no aplica todavía —
  esta feature no incluye CRUD de libros)

---

**Historia 3:** Como lector, quiero que mis libros sean privados y que
ningún otro usuario de marginaLia pueda verlos ni modificarlos.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Los datos de cada usuario viven exclusivamente bajo `users/{uid}/`
- [ ] Las Firestore Security Rules restringen lectura y escritura al propio
      `uid` (`request.auth.uid == userId`), tal como especifica CLAUDE.md
- [ ] Nadie puede acceder a `users/{uid}/books/` sin haber iniciado sesión

**Edge cases:**
- Ninguno adicional — esta historia es puramente de seguridad/infraestructura

---

## Decisiones

**1. Proyecto Firebase para marginaLia — RESUELTO.**
El Ingeniero Jefe decidió: proyecto Firebase nuevo y separado del de ReMynder
(`pendientes-2c0ea`). marginaLia tiene su propio proyecto, con Auth Google
habilitado y Firestore propio — infraestructura completamente aislada, cuentas
Google independientes por proyecto (no hay sesión compartida con ReMynder).
El Backend Builder implementa el código con `firebaseConfig` como placeholder
explícito, listo para pegar los valores reales una vez creado el proyecto.

**2. Seed inicial de áreas de conocimiento por usuario nuevo — PENDIENTE (no bloquea esta feature).**
ReMynder tiene un `ADMIN_UID` hardcodeado que siembra categorías fijas para
el usuario admin, y 2 categorías de ejemplo para el resto. CLAUDE.md no
especifica si un usuario nuevo de marginaLia arranca con áreas de ejemplo
predefinidas (ej. "Física", "Historia") o con la lista de áreas
completamente vacía hasta que el usuario cree la primera.
Esta decisión pertenece a la feature de "gestión de áreas de conocimiento",
no a esta feature de login — se documenta acá porque research-login.md la
detectó, pero no bloquea la implementación de login en sí (no hay seed de
libros/áreas involucrado en Historia 1-3).
