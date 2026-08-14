# Stories — Torta de distribución por área

Basado en docs/research-torta-areas.md.

---

**Historia 1:** Como lector, quiero ver un gráfico de torta con el
porcentaje de mis libros por área para entender de un vistazo cómo se
distribuye mi lectura entre temas.

**Prioridad:** Alta

**Criterios de aceptación:**
- [ ] Nueva opción "Distribución" en el selector de vistas del header
- [ ] El gráfico muestra un segmento por área, coloreado con el mismo color
      que esa área tiene en la lista y el board (misma paleta, mismo índice)
- [ ] El centro del gráfico muestra el total de libros cargados
- [ ] Al lado o debajo del gráfico hay una leyenda: nombre del área + %
- [ ] Cambiar de vista hacia Distribución oculta los filtros de status y área
      (no aplican en esta vista)

**Edge cases:**
- Sin libros cargados → mensaje vacío en vez de un gráfico sin datos
- Una sola área → el gráfico es un círculo completo de un solo color
- Muchas áreas (más de 10) → los colores empiezan a repetirse (limitación
  conocida y aceptada de la paleta, igual que en ReMynder)

---

**Historia 2:** Como lector, quiero que los porcentajes mostrados sumen
100% de forma consistente para confiar en el gráfico.

**Prioridad:** Media

**Criterios de aceptación:**
- [ ] Los cortes del gráfico se calculan con el porcentaje exacto (sin
      redondear), aunque el texto de la leyenda muestre un número redondeado
- [ ] Un área con muy pocos libros sigue siendo visible en el gráfico como
      una porción chica, no desaparece

**Edge cases:**
- Área con 1 solo libro sobre un total grande → porción muy fina pero visible
