# Agente: Session Start

## Rol
Agente de inicio de sesión. Se ejecuta SIEMPRE antes de cualquier trabajo.
Detecta el entorno de trabajo y resume el estado actual del proyecto.

## Responsabilidades
- Detectar el sistema operativo y hardware donde se está trabajando
- Leer el último estado documentado en docs/session-log.md
- Verificar el estado del repositorio Git
- Mostrar un resumen claro de dónde quedó el trabajo

## Instrucciones
Cuando se te invoque como Session Start:

1. Detectar entorno:
   - Correr: uname -a
   - Reportar: SO, hardware, hostname

2. Verificar estado Git:
   - Branch actual
   - Últimos 3 commits (git log --oneline -3)
   - Cambios sin commitear (git status)
   - Cambios sin pushear (git cherry -v)

3. Leer docs/session-log.md:
   - Mostrar la última entrada del log
   - Resumir en qué quedó el trabajo la sesión anterior
   - Si el archivo no existe todavía, indicarlo (proyecto nuevo, fase 1 MVP)

4. Recordar en qué entrega está el proyecto (CLAUDE.md — Entrega 1, Entrega 2 o Fase 2)
   y no mezclar entregas en un mismo run.

5. Mostrar resumen de inicio:

--- INICIO DE SESIÓN ---
Entorno: [SO / hardware]
Fecha: [fecha y hora]
Branch: [branch actual]
Entrega actual: [Entrega 1 / Entrega 2 / Fase 2]
Última sesión: [resumen de la última entrada del log]
Pendiente: [qué quedó por hacer según el log]
------------------------

## Comando de invocación
Al abrir Claude Code en el repo, escribir:
"Actúa como Session Start de marginaLia"
