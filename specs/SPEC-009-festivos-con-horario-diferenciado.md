# SPEC-009 — Festivos con horario diferenciado

> RF: RF-FES-004
> Criticidad: Must
> Wave: 2 — Festivos y excepciones

## Descripción

Garantiza que un festivo de apertura pueda tener un horario completamente diferente al habitual del período (ej: Nochebuena cierre a las 18:00 en lugar de las 22:00, o apertura tardía a las 12:00 en día de Reyes). SPEC-007 ya define la mecánica de creación con la opción "Horario especial". Esta spec complementa con los criterios de visualización diferenciada, la comparación con el horario habitual, y los edge cases de edición y consistencia.

## Criterios de aceptación

### Visualización diferenciada en la lista

- GIVEN un festivo de apertura tiene horario especial (diferente al del período) WHEN se muestra en la lista de festivos (SPEC-007) THEN la columna de horario muestra las franjas definidas sin indicador "(habitual)" (ej: "10:00 - 18:00"), distinguiéndolo visualmente de los festivos con horario heredado.
- GIVEN un festivo de apertura tiene horario especial WHEN se muestra en la lista THEN se añade un Badge "Horario especial" (variant outline, small) junto al Badge de tipo "Apertura", para que el usuario identifique rápidamente los festivos que no siguen el patrón habitual.
- GIVEN un festivo de apertura tiene horario especial con horario partido (2 franjas) WHEN se muestra en la lista THEN la columna de horario muestra ambas franjas separadas por " / " (ej: "10:00 - 14:00 / 17:00 - 20:00").

### Comparación con horario habitual

- GIVEN un festivo de apertura tiene horario especial WHEN el usuario abre el Dialog de edición THEN se muestra un texto comparativo debajo de las franjas: "Horario habitual del [día de la semana]: [horario del período]" (texto muted, small), para que el usuario pueda comparar lo que ha definido con lo que tendría por defecto.
- GIVEN un festivo de apertura tiene horario especial WHEN el horario especial coincide exactamente con el horario habitual del período THEN se muestra una sugerencia inline (texto muted): "Este horario coincide con el habitual. ¿Quieres usar el horario del período en su lugar?" con un enlace "Usar horario habitual" que cambia a inherit_period_schedule = true.

### Cambio entre horario especial y habitual

- GIVEN un festivo de apertura tiene horario especial WHEN el usuario cambia a "Horario habitual del período" en el Dialog de edición THEN las franjas especiales se descartan y se muestra el horario heredado. Se pide confirmación si las franjas especiales eran diferentes al habitual: "Se descartará el horario especial (10:00 - 18:00) y se usará el horario habitual (10:00 - 22:00). ¿Continuar?".
- GIVEN un festivo de apertura tiene horario habitual heredado WHEN el usuario cambia a "Horario especial" en el Dialog de edición THEN los campos de franjas aparecen pre-rellenados con el horario heredado actual, para que el usuario solo modifique lo que necesita en lugar de introducir todo desde cero.

### Validación específica

- GIVEN un festivo de apertura con horario especial WHEN el usuario define franjas THEN se aplican las mismas validaciones que SPEC-002 (hora fin > hora inicio) y SPEC-003 (no solapamiento entre franjas, máximo 2 franjas).
- GIVEN un festivo de apertura con horario especial WHEN el usuario elimina todas las franjas sin cambiar a "Cierre" THEN se aplica la validación de SPEC-006: el botón "Guardar" permanece desactivado.

### Edge cases

- GIVEN un festivo de apertura con horario especial WHEN el usuario modifica el horario del período para ese día de la semana THEN el festivo mantiene su horario especial sin cambios (no se ve afectado por cambios en el período, a diferencia del horario heredado de SPEC-008).
- GIVEN un festivo de apertura con horario especial WHEN el usuario elimina el período que contenía la fecha del festivo THEN el festivo mantiene su horario especial sin cambios (tiene franjas propias, no depende del período).
- GIVEN se realiza una carga centralizada de festivos (RF-FES-003) con horario especial WHEN se aplica a múltiples establecimientos THEN todos reciben exactamente las mismas franjas especiales definidas en la carga (no se resuelve por período de cada establecimiento).

## UX Design

### Wireframe textual

Esta spec no tiene pantalla nueva. Modifica la lista de festivos y el Dialog de SPEC-007:

**En la lista de festivos (SPEC-007):**
- Festivos con horario especial: Badge "Apertura" (variant default) + Badge "Horario especial" (variant outline, small). Columna de horario con las franjas definidas (sin "(habitual)").
- Festivos con horario heredado: Badge "Apertura" (variant default) sin Badge adicional. Columna de horario con indicador "(habitual)".
- La distinción visual entre ambos permite al usuario escanear rápidamente qué festivos requieren atención especial.

**En el Dialog de edición (SPEC-007):**
- Bloque de comparación: debajo de los campos de franjas, texto muted small con el horario habitual del período como referencia.
- Sugerencia de simplificación: si el horario especial coincide con el habitual, enlace "Usar horario habitual" (Button variant link, small).

### Componentes shadcn utilizados

Componentes: Badge, Button (todos ya presentes). No se requieren componentes adicionales.

### Patrón de interacción

- **Badge "Horario especial"** como indicador visual en la lista: el usuario distingue de un vistazo qué festivos tienen configuración personalizada. Esto es importante cuando hay 10-20 festivos y solo 2-3 tienen horario diferente. (Regla: Badge para estados y categorías. Variant outline para información secundaria no crítica.)
- **Comparación con horario habitual en el Dialog:** reduce errores al editar. El usuario ve "normalmente cierras a las 22:00, aquí has puesto las 18:00" y puede validar que es correcto. (Decisión no cubierta por el design system. Se resuelve con texto informativo debajo del formulario.)
- **Pre-rellenado al cambiar a "Horario especial":** el usuario no empieza de cero. Si quiere cerrar 2 horas antes, solo cambia la hora de fin. (Decisión de UX: minimizar entrada de datos.)
- **Confirmación al descartar horario especial:** evita pérdida accidental de una configuración manual. (Regla: AlertDialog o confirmación antes de perder datos del usuario.)

### Comportamiento responsive

- **Mobile (< md):** Los Badges se apilan debajo del nombre del festivo. La comparación con horario habitual se muestra igual.
- **Tablet (md-lg):** Interpolado.
- **Desktop (lg+):** Como descrito.

## Notas técnicas

- Modelo de datos: cuando `inherit_period_schedule = false`, el festivo tiene registros propios en `holiday_slots`. Estructura: `holiday_id`, `start_time` (HH:MM), `end_time` (HH:MM), `slot_order` (1 o 2, para horario partido).
- La independencia del período es total: modificar o eliminar el período no afecta a las franjas de un festivo con horario especial.
- La sugerencia de simplificación (cuando el horario especial coincide con el habitual) se evalúa client-side comparando las franjas del festivo con las del período para el día de la semana correspondiente.
- Dependencia upstream: SPEC-007 (registro de festivos), SPEC-008 (herencia de horario), SPEC-003 (horario partido).
