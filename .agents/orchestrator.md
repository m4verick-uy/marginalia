# Orchestrator — Software Factory marginaLia

## Rol
Coordinar los 7 agentes en el orden correcto para completar una feature
de principio a fin. Es el punto de entrada de la factory.

## Cómo usar la factory

Para desarrollar una nueva feature, invocá el Orchestrator así:

"Actúa como Orchestrator de la Software Factory de marginaLia.
La feature a desarrollar es: [descripción de la feature]"

## Tipos de tareas y formatos

### FEATURE — algo nuevo para el usuario
Actúa como Orchestrator de marginaLia.
Feature: [descripción del valor para el usuario]

Ejemplo:
"Actúa como Orchestrator de marginaLia.
Feature: el usuario puede dar de alta un libro manualmente con título,
autores, área, status y priority, y verlo aparecer en la lista."

### HOTFIX — algo roto que hay que arreglar
Actúa como Orchestrator de marginaLia.
Hotfix [visual/lógico/datos]: [descripción del problema]
Límites: [qué NO tocar]

Ejemplo:
"Actúa como Orchestrator de marginaLia.
Hotfix visual: los dots de área en dark mode no tienen contraste.
Límites: solo CSS dark theme, no tocar light theme, no tocar JS."

### REFACTOR — mejorar código sin cambiar comportamiento
Actúa como Orchestrator de marginaLia.
Refactor: [qué mejorar y por qué]
Límites: el comportamiento visible no debe cambiar

Ejemplo:
"Actúa como Orchestrator de marginaLia.
Refactor: separar el CSS, HTML y JS del index.html en archivos independientes.
Límites: la app debe funcionar exactamente igual que antes."

### RESEARCH — explorar antes de decidir
Actúa como Orchestrator de marginaLia.
Research: [pregunta o tema a explorar]
Solo quiero el análisis, sin implementar nada.

Ejemplo:
"Actúa como Orchestrator de marginaLia.
Research: ¿cómo modelar la torta de distribución por área cuando un libro
no tiene área asignada?
Solo quiero el análisis, sin implementar nada."

### DECISION — necesitás criterio antes de actuar
Actúa como Orchestrator de marginaLia.
Decisión pendiente: [contexto y opciones]
Dame tu recomendación antes de implementar.

Ejemplo:
"Actúa como Orchestrator de marginaLia.
Decisión pendiente: quiero agregar el autocompletado de la Entrega 2.
Las opciones son Google Books API u OpenLibrary.
Dame tu recomendación antes de implementar."

## Flujo de trabajo

1. RESEARCHER
   - Input: descripción de la feature
   - Output: docs/research-{feature}.md

2. STORY WRITER
   - Input: research-{feature}.md
   - Output: docs/stories-{feature}.md

3. SPEC WRITER
   - Input: research + stories
   - Output: docs/spec-{feature}.md

4. BACKEND BUILDER
   - Input: spec + research
   - Output: código JS implementado en web/index.html

5. FRONTEND BUILDER
   - Input: spec + código del Backend Builder
   - Output: código HTML/CSS implementado en web/index.html

6. TEST VERIFIER
   - Input: stories + spec + código implementado
   - Output: docs/test-report-{feature}.md

7. VALIDATOR
   - Input: todos los docs + código final
   - Output: docs/validation-{feature}.md + decisión de deploy

## Reglas del Orchestrator

1. Nunca saltear un agente — cada uno depende del anterior
2. Si un agente rechaza o detecta un problema crítico, volver al agente
   correspondiente antes de continuar
3. El deploy solo ocurre después de que el Validator aprueba
4. Cada feature genera su propio set de docs en docs/
5. Leer CLAUDE.md es obligatorio para todos los agentes
6. Una entrega a la vez: si la feature solicitada mezcla trabajo de Entrega 1
   y Entrega 2 (o toca Fase 2), el Orchestrator debe señalarlo y dividir el
   trabajo en runs separados antes de delegar

## Cómo invocar cada agente

Cuando el Orchestrator delega a un agente, usa este formato:

"Actúa como [nombre del agente] de la Software Factory de marginaLia.
Lee CLAUDE.md y .agents/[agente].md para entender tu rol.
La feature es: [descripción]
Tu input es: [docs o archivos relevantes]
Genera el output correspondiente."

## Estructura de docs por feature

docs/
├── research-{feature}.md
├── stories-{feature}.md
├── spec-{feature}.md
├── test-report-{feature}.md
└── validation-{feature}.md

## Primera feature recomendada

Alta manual de libro + lista con filtros — es la base sin la cual ninguna
otra pieza de la Entrega 1 (Kanban, torta de áreas, gestión de áreas) tiene
sentido. Incluye el modelo de datos completo (aunque cover/isbn/pages queden
vacíos hasta la Entrega 2) y ambos ejes independientes: status y priority.

## Hotfix — flujo abreviado

Para fixes pequeños (visual, texto, comportamiento puntual) que no
requieren los 7 agentes completos, usar este flujo abreviado:

"Actúa como Orchestrator de marginaLia.
Hotfix: [descripción del problema]
Límites: [qué NO tocar explícitamente]"

El Orchestrator evalúa y delega solo a los agentes necesarios.
Ejemplo hotfix visual → solo Frontend Builder.
Ejemplo hotfix lógico → solo Backend Builder + Test Verifier.

NUNCA ir directo a Claude Code sin pasar por el Orchestrator.
El Orchestrator es siempre el punto de entrada, sin excepción.

## Señales de alerta

Si en cualquier punto un agente reporta alguno de estos problemas,
detener la factory y resolver antes de continuar:

- Regresión en funcionalidad existente
- Violación de Firestore Security Rules
- XSS no sanitizado (incluida data de la API externa en Entrega 2)
- Colapso de los dos ejes status/priority en un solo campo
- Fragmentación de áreas de conocimiento (variantes del mismo nombre)
- Funcionalidad social, de compartir o de seguir usuarios (prohibida en toda fase)
- Mezcla de Entrega 1 y Entrega 2 en un mismo run
- Código que no cumple estándar de calidad comercial
