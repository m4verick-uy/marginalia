# Research — Torta de distribución por área

## Resumen del problema

CLAUDE.md marca la torta de distribución por área como parte de la Entrega 1
y como uno de los diferenciales de producto explícitos frente a Goodreads:
"no es un catálogo social, es un mapa de lo que sabés y querés saber". Es un
gráfico de % de libros por área, para ver cobertura actual de lectura.

## A qué entrega pertenece

Entrega 1, sin dependencias externas.

## Estado actual del código

No hay precedente en ReMynder (no tiene gráficos). Ya existen y se
reutilizan tal cual:
- `getAreaOptions()`: áreas únicas derivadas de `books`
- `areaColorIndex(area)`: mapeo área → índice 0–9, usado por los dots de
  área en la lista y el board — la torta debe usar la MISMA función para que
  el color de un área sea consistente en las tres vistas
- Paleta `--p0`–`--p9` ya definida en `:root`

## Decisión técnica: sin librería de gráficos

CLAUDE.md prohíbe dependencias innecesarias y no hay build/npm. Se construye
con **CSS puro**: un círculo con `background: conic-gradient(...)` calculado
en JS a partir de los porcentajes reales, más un círculo interno superpuesto
(mismo color que el fondo) para el efecto donut, con el total de libros en
el centro (patrón visual tipo Apple Health / Activity rings, consistente con
la aspiración "calidad Apple-like" de CLAUDE.md). Cero dependencias nuevas.

## Dónde vive en la UI

Se agrega como una tercera opción al selector de vistas ya existente
(Lista | Tablero | **Distribución**), mismo patrón que el `.view-switch` de
2 opciones agregado en la feature de Kanban — ahora pasa a 3, análogo a las
3 vistas de ReMynder (Lista/Tablero/Foco), aunque acá la tercera no es una
variante de la lista de tareas sino un análisis agregado.

## Alcance de filtros en esta vista

La torta muestra la distribución de **todos** los libros del usuario, sin
aplicar `statusFilter` ni `areaFilter` — filtrar la torta por área no tendría
sentido (mostraría 100% de un solo color). Al entrar a esta vista, se oculta
el `<nav class="filters">` completo (no solo el filtro de status como en
Tablero).

## Riesgos o conflictos detectados

- Con muchas áreas (>10), `areaColorIndex()` empieza a repetir colores del
  módulo 10 — riesgo ya aceptado desde la paleta original (mismo límite que
  ReMynder con sus categorías), no es nuevo de esta feature
- Redondeo: si se muestran porcentajes redondeados en la leyenda pero el
  cálculo del `conic-gradient` usa valores sin redondear, los porcentajes
  visibles pueden no sumar exactamente 100% por redondeo (ej. 33.3+33.3+33.4
  se ve bien, pero 33+33+33=99 se nota). Mitigación: redondear solo para
  mostrar texto, no para calcular los cortes del gráfico.

## Recomendaciones para los siguientes agentes

- Story Writer: cubrir el caso sin libros (torta vacía) y el caso de una
  sola área (100%, un solo color)
- Spec Writer: especificar la fórmula de conic-gradient y la reutilización
  de `areaColorIndex()` / paleta existente
- Frontend Builder: leyenda con el mismo patrón `.badge.area-pN` +
  `.area-dot` ya usado en tarjetas y board, para consistencia visual
