# SPEC-017 — Visualización del horario de cierre en festivos

> RF: RF-FES-005
> Criticidad: Should
> Wave: 3 — Visualización, copia interanual y validación

## Descripción

Garantiza que cuando un festivo es de apertura, la hora de cierre sea claramente visible en todos los puntos de la interfaz donde aparece ese festivo. El punto de dolor reportado en el research es que los usuarios necesitan conocer la hora de cierre para planificar recogidas, turnos de cierre y comunicación a clientes, pero el sistema legacy mostraba los festivos como una marca binaria (abierto/cerrado) sin detalle horario visible. Esta spec no introduce pantallas nuevas: refuerza y estandariza la presentación del horario en la lista de festivos (SPEC-007), el calendario anual (SPEC-013) y los tooltips (SPEC-013/SPEC-014).

## Criterios de aceptación

### Hora de cierre en la lista de festivos

- GIVEN un festivo de apertura con horario habitual (una franja) WHEN se muestra en la lista de festivos (SPEC-007) THEN la columna de horario muestra "Abre [hora inicio] - Cierra [hora fin]" (ej: "Abre 10:00 - Cierra 18:00"). La palabra "Cierra" se muestra en font-medium para enfatizarla.
- GIVEN un festivo de apertura con horario partido (dos franjas) WHEN se muestra en la lista THEN la columna de horario muestra ambas franjas con la hora de cierre de la última franja enfatizada: "10:00 - 14:00 / 17:00 - Cierra 20:00".
- GIVEN un festivo de cierre WHEN se muestra en la lista THEN la columna de horario muestra "Cerrado" (sin hora, ya que no hay apertura).
- GIVEN un festivo de apertura con horario especial diferente al habitual WHEN se muestra en la lista THEN el formato es el mismo ("Abre [inicio] - Cierra [fin]") con el Badge "Horario especial" ya definido en SPEC-009.

### Hora de cierre en el tooltip del calendario

- GIVEN un festivo de apertura WHEN el usuario hace hover sobre la celda del calendario (SPEC-013) THEN el tooltip muestra la hora de cierre de forma explícita: "Festivo de apertura: [nombre] — Cierra a las [hora fin]". La hora de cierre es el dato más prominente del tooltip.
- GIVEN un festivo de apertura con horario partido WHEN el tooltip se muestra THEN indica: "Festivo de apertura: [nombre] — 10:00-14:00 / 17:00-20:00 — Cierra a las 20:00".
- GIVEN un festivo de cierre WHEN el tooltip se muestra THEN indica: "Festivo de cierre: [nombre] — Cerrado todo el día".

### Hora de cierre en el Popover del calendario (SPEC-014)

- GIVEN un festivo de apertura WHEN el usuario hace clic en la celda del calendario y se abre el Popover o Dialog THEN la hora de cierre aparece visible en los datos pre-cargados, con el mismo formato enfatizado.

### Hora de cierre en el Dialog de edición

- GIVEN un festivo de apertura WHEN el usuario abre el Dialog de edición (SPEC-007) THEN el texto informativo de horario resuelto muestra explícitamente la hora de cierre: "Horario: [hora inicio] - [hora fin]. El establecimiento cierra a las [hora fin]".
- GIVEN un festivo de apertura con horario especial WHEN el usuario modifica la hora de fin en el Dialog THEN el texto informativo se actualiza dinámicamente: "El establecimiento cerrará a las [nueva hora fin]" (en lugar de la hora anterior).

### Hora de cierre en comparación con horario habitual

- GIVEN un festivo de apertura con horario especial WHEN el Dialog de edición muestra la comparación con el horario habitual (SPEC-009) THEN la comparación enfatiza la diferencia en hora de cierre: "Horario habitual: cierra a las 22:00. Horario especial: cierra a las 18:00 (4 horas antes)".
- GIVEN la diferencia de cierre es significativa (≥ 2 horas antes o después del habitual) WHEN la comparación se muestra THEN el texto de diferencia usa color warning para llamar la atención.

### Edge cases

- GIVEN un festivo de apertura hereda el horario del período WHEN el período se modifica cambiando la hora de cierre THEN la hora de cierre mostrada en la lista y el calendario se actualiza dinámicamente (herencia en tiempo real, ya cubierta por SPEC-008).
- GIVEN un festivo de apertura con horario especial tiene la misma hora de cierre que el habitual WHEN se muestra la comparación THEN el texto indica "Misma hora de cierre que el habitual (22:00)" sin color warning.

## UX Design

### Wireframe textual

Esta spec no tiene pantalla nueva. Modifica la presentación del horario en componentes existentes:

**En la lista de festivos (SPEC-007), columna de horario:**
- Formato actual: "10:00 - 18:00 (habitual)"
- Formato nuevo: "Abre 10:00 - **Cierra 18:00** (habitual)"
- El "Cierra" en font-medium (no bold completo, sutil pero distinguible).

**En el tooltip del calendario (SPEC-013):**
- Formato actual: "Festivo de apertura: Nochebuena — 10:00 - 18:00"
- Formato nuevo: "Festivo de apertura: Nochebuena — Cierra a las 18:00"
- La hora de cierre como dato principal del tooltip.

**En el Dialog de edición (SPEC-007), texto informativo:**
- Línea adicional debajo de las franjas: "El establecimiento cierra a las [hora fin]" (texto small, font-medium).

**En la comparación (SPEC-009):**
- Formato nuevo: "Horario habitual: cierra a las 22:00. Horario especial: cierra a las 18:00 (4h antes)".
- Diferencia ≥ 2h: color warning.

### Componentes shadcn utilizados

No se requieren componentes adicionales. Se modifica el contenido de componentes existentes.

### Patrón de interacción

- **Enfatizar "Cierra" en lugar de mostrar un rango neutro:** el rango "10:00 - 18:00" es preciso pero no comunica la urgencia de la hora de cierre. "Cierra a las 18:00" responde directamente a la pregunta del usuario: "¿a qué hora cerramos?". (Decisión de UX: optimizar para la pregunta más frecuente del usuario.)
- **Diferencia horaria en la comparación:** "4 horas antes" es más inmediato que comparar mentalmente "22:00 vs 18:00". Reduce la carga cognitiva. (Decisión de UX: hacer explícita la diferencia en lugar de dejar que el usuario la calcule.)
- **Color warning para diferencias significativas (≥ 2h):** una diferencia de 30 minutos es un ajuste menor. Una diferencia de 4 horas es un cambio que afecta a turnos, recogidas y comunicación a clientes. El umbral de 2 horas marca el punto donde la diferencia merece atención especial. (Decisión de UX: umbral basado en impacto operativo.)

### Comportamiento responsive

- **Mobile (< md):** El formato "Cierra [hora]" se mantiene igual. Es más compacto que el rango completo, lo cual beneficia a pantallas pequeñas.
- **Tablet (md-lg):** Interpolado.
- **Desktop (lg+):** Como descrito.

## Notas técnicas

- Esta spec no modifica el modelo de datos ni la API. Es puramente de presentación (formatting del horario en la capa de UI).
- La hora de cierre es siempre el `end_time` de la última franja del festivo (ya sea heredada del período o especial). Si hay horario partido (2 franjas), es el `end_time` de la franja con `slot_order = 2`.
- El cálculo de diferencia horaria (ej: "4h antes") se realiza client-side comparando el `end_time` del festivo con el `end_time` del horario habitual del período para el día de la semana correspondiente.
- Dependencia upstream: SPEC-007 (lista de festivos), SPEC-008 (herencia de horario), SPEC-009 (comparación con horario habitual), SPEC-013 (calendario, tooltip), SPEC-014 (Popover de calendario).
- No hay dependencia downstream.
