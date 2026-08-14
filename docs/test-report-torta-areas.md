# Test Report — Torta de distribución por área

**Feature:** Torta de distribución por área
**Estado general:** ✅ Aprobado

## Criterios de aceptación

**Historia 1 — ver torta de distribución**
- [x] Tercera opción "Distribución" en `.view-switch`: ok
- [x] Colores del gráfico vía `areaColorIndex()`, misma función que usan
      Lista y Board — un área tiene el mismo color en las 3 vistas: ok
- [x] Centro del donut muestra `books.length`: ok
- [x] Leyenda con nombre + % debajo del gráfico, reutilizando
      `.badge.area-pN` + `.area-dot`: ok
- [x] `.filters` completo oculto en esta vista (`display:none`), no solo el
      status como en Tablero: ok

**Historia 2 — porcentajes consistentes**
- [x] `conic-gradient` se calcula acumulando `pct` sin redondear
      (`acc += pct` con el valor exacto); solo el texto de la leyenda usa
      `Math.round(... * 10) / 10`: ok, los cortes del gráfico siempre suman
      exactamente 100% aunque el texto redondeado individualmente no sume
      perfecto a la vista (limitación aceptada, es solo texto)
- [x] Área con 1 libro sobre un total grande → sigue generando un stop de
      ancho proporcional real, no desaparece: ok (no hay ningún `if` que
      filtre áreas por conteo mínimo)

## Regresiones detectadas

Ninguna. Se revisó que:
- Cambiar entre Lista/Tablero sigue funcionando igual que antes (la lógica
  de esas dos vistas no cambió, solo se extendió `render()` con un tercer
  branch)
- El filtro de área en Lista/Tablero no se ve afectado por la nueva vista

## Edge cases verificados

- 0 libros + vista Distribución → mensaje vacío, sin gráfico ni leyenda: ok
- 1 sola área → un único stop `var(--pN) 0% 100%`, círculo de un solo color: ok
- Cambiar de vista repetidas veces → `renderDistribution()` recalcula todo
  desde cero cada vez (no hay estado incremental que pueda desincronizarse): ok

## Seguridad

- [x] `escHtml()` aplicado al nombre de área en la leyenda (innerHTML)
- [x] No hay operaciones de escritura en esta feature — solo lectura de
      `books` ya cargado en memoria, sin nuevas consultas a Firestore

## Problemas encontrados

Ninguno.

## Recomendaciones

- Probar visualmente con 2-3 áreas distintas que los colores coincidan
  entre la Lista y la Distribución (verificación manual, no automatizable
  sin navegador)
