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
