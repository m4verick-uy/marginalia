# Documentación arquitectural — marginaLia

Este archivo se actualiza con cada cambio arquitectural, a cargo del Validator
al cerrar cada feature. Ver CLAUDE.md para la visión de producto y el modelo
de datos completo.

## Estado actual

Fase 1, MVP, Entrega 1. Primera feature implementada: Login Google + multiusuario.

## Proyecto Firebase

marginaLia usa un **proyecto Firebase propio**, separado del de ReMynder —
decisión tomada el 2026-08-13 (ver docs/stories-login.md, sección Decisiones).
Auth Google + Firestore, sin infraestructura compartida entre productos.

Proyecto real: `marginalia-68e0a`. `firebaseConfig` en `web/index.html` ya
tiene los valores reales (2026-08-13) — pendiente de confirmar en la consola:
método Google habilitado en Authentication, Firestore Database creada, y las
Security Rules de `users/{userId}/books/{bookId}` pegadas (checklist en
docs/spec-login.md).

## Historial de cambios arquitecturales

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
