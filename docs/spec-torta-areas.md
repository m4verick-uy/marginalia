# Spec — Torta de distribución por área

**Feature:** Torta de distribución por área
**Entrega:** Entrega 1
**Stack afectado:** JS, CSS, HTML (sin cambios de Firestore)

## Cambios en modelo de datos

Ninguno — se calcula 100% en memoria a partir de `books` ya cargado.

## Cambios en UI

- `.view-switch` pasa de 2 a 3 botones: Lista / Tablero / Distribución
- Nuevo contenedor `#distribution` (oculto por defecto) dentro de `main`,
  con:
  - `#chart-wrap`: círculo `#pie-chart` (conic-gradient) + `.pie-hole`
    superpuesto con el total de libros
  - `#pie-legend`: lista de `.pie-legend-row`, cada una reutilizando
    `.badge.area-pN` + `.area-dot` (mismos que ya existen) + porcentaje
  - `#distribution-empty`: mensaje cuando no hay libros
- `.filters` se oculta completo (no solo `#status-filters`) cuando
  `viewMode === 'distribution'`

## Cambios en lógica JS

- `renderDistribution()`: calcula `total = books.length`, para cada área de
  `getAreaOptions()` cuenta sus libros, arma los stops de
  `conic-gradient(var(--pN) start% end%, ...)` acumulando porcentajes SIN
  redondear, aplica como `style.background` de `#pie-chart`
- Reutiliza `areaColorIndex(area)` para el índice de color — misma fuente
  de verdad que la lista y el board, no se duplica la lógica de mapeo
- `render()` extendido: si `viewMode === 'distribution'`, oculta
  `.filters`, `.book-list`, `#board`, `#board-dots`, `#empty-state`, muestra
  `#distribution` y llama `renderDistribution()`

## Archivos a modificar

- `web/index.html` (único archivo)

## Riesgos técnicos

- Ya cubiertos en research-torta-areas.md (repetición de color con >10
  áreas, redondeo de porcentajes) — riesgos aceptados, sin mitigación
  adicional más allá de calcular los cortes sin redondear

## Criterios de done

- [ ] Historias 1–2 cumplidas
- [ ] Colores del gráfico coinciden exactamente con los dots de área en
      Lista y Tablero (misma función `areaColorIndex`)
- [ ] Filtros ocultos en la vista Distribución
- [ ] Mobile: el gráfico y la leyenda se acomodan en columna en pantallas chicas
