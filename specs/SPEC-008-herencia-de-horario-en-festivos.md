# SPEC-008 — Herencia de horario en festivos

> RF: RF-FES-002
> Criticidad: Must
> Wave: 2 — Festivos y excepciones

## Descripción

Define la lógica de resolución automática del horario cuando un festivo de apertura se configura con la opción "Horario habitual del período" (SPEC-007). El sistema determina qué horario aplicar a partir del día de la semana en que cae el festivo y las franjas definidas en el período correspondiente, evitando que el usuario tenga que introducir manualmente un horario que ya existe. Esta spec complementa SPEC-007 detallando los casos en que la herencia no es directa.

## Criterios de aceptación

### Resolución del horario heredado

- GIVEN un festivo de apertura con "Horario habitual del período" WHEN el sistema resuelve el horario THEN busca el período cuyo rango de fechas contiene la fecha del festivo y obtiene las franjas del día de la semana correspondiente (ej: si el festivo cae en miércoles, hereda las franjas del miércoles de ese período).
- GIVEN un festivo cae en un día con horario partido (2 franjas en SPEC-003) WHEN el sistema resuelve el horario THEN hereda ambas franjas (ej: "10:00 - 14:00 / 17:00 - 21:00").
- GIVEN un festivo cae en un día con una sola franja WHEN el sistema resuelve el horario THEN hereda esa única franja.
- GIVEN existen múltiples períodos y el festivo cae dentro del rango de uno de ellos WHEN el sistema resuelve el horario THEN usa exclusivamente las franjas del período que contiene la fecha (no de otro período).

### Visualización del horario heredado

- GIVEN un festivo de apertura hereda el horario del período WHEN se muestra en la lista de festivos (SPEC-007) THEN la columna de horario muestra el horario resuelto seguido de "(habitual)" en texto muted (ej: "10:00 - 22:00 (habitual)").
- GIVEN un festivo de apertura tiene horario especial definido manualmente WHEN se muestra en la lista de festivos THEN la columna de horario muestra solo el horario sin indicador adicional (ej: "10:00 - 18:00").
- GIVEN un festivo de apertura hereda el horario WHEN el usuario abre el Dialog de edición THEN se muestra la opción "Horario habitual del período" seleccionada, con el texto informativo mostrando el horario resuelto actual.

### Previsualización en el Dialog de creación

- GIVEN el usuario está creando un festivo de apertura con "Horario habitual del período" WHEN selecciona o cambia la fecha THEN el texto informativo se actualiza dinámicamente mostrando el horario que se heredará para esa fecha concreta (ej: al cambiar de lunes a sábado, el horario previsualizado cambia si el sábado tiene horario diferente).
- GIVEN el usuario está creando un festivo de apertura con "Horario habitual del período" WHEN la fecha seleccionada cae en un día con horario partido THEN el texto informativo muestra ambas franjas (ej: "Se aplicará el horario del período 'Horario anual': 10:00 - 14:00 / 17:00 - 21:00").

### Conflicto: día cerrado en el período

- GIVEN un festivo de apertura con "Horario habitual del período" WHEN el día de la semana de la fecha del festivo está marcado como "Cerrado" en el período correspondiente THEN se muestra advertencia inline en el Dialog: "El [día de la semana] está marcado como cerrado en el período '[nombre]'. Define un horario especial para este festivo".
- GIVEN el conflicto de día cerrado se detecta WHEN el usuario intenta guardar con "Horario habitual del período" THEN el guardado se bloquea. El botón "Crear" permanece desactivado con Tooltip: "No hay horario que heredar. El [día] está cerrado en el período".
- GIVEN el conflicto de día cerrado se detecta WHEN el usuario cambia a "Horario especial" y define franjas THEN el festivo se puede guardar correctamente.

### Conflicto: período sin franjas definidas

- GIVEN un festivo de apertura con "Horario habitual del período" WHEN el período correspondiente no tiene ninguna franja definida (estado incompleto) THEN se muestra advertencia inline: "El período '[nombre]' no tiene horario definido aún. Define un horario especial o configura primero el horario del período".
- GIVEN el período no tiene franjas WHEN el usuario intenta guardar THEN el guardado se bloquea (mismo comportamiento que el conflicto de día cerrado).

### Conflicto: fecha fuera de cualquier período

- GIVEN un festivo de apertura con "Horario habitual del período" WHEN la fecha no cae dentro de ningún período definido THEN se muestra advertencia inline: "La fecha seleccionada no está cubierta por ningún período. Define un horario especial".
- GIVEN la fecha no tiene período WHEN el usuario intenta guardar con "Horario habitual" THEN el guardado se bloquea. El usuario debe cambiar a "Horario especial".

### Actualización dinámica del horario heredado

- GIVEN un festivo hereda el horario de un período WHEN el usuario modifica las franjas del período para ese día de la semana THEN el horario del festivo se actualiza automáticamente (no se almacena una copia estática).
- GIVEN un festivo hereda el horario de un período WHEN el usuario elimina el período THEN el festivo queda en estado de conflicto y se muestra el indicador de advertencia (AlertTriangle) con tooltip: "El período asociado ha sido eliminado. Edita el festivo para asignar un horario especial".
- GIVEN un festivo hereda el horario de un período WHEN el usuario divide el período (SPEC-004) y el festivo queda en uno de los dos nuevos períodos THEN la herencia se reasigna al nuevo período que contiene la fecha del festivo.

### Edge cases

- GIVEN un festivo de apertura hereda el horario WHEN se consulta via API (RF-INT-001) THEN el endpoint devuelve el horario resuelto (las franjas concretas), no la referencia al período. Los sistemas downstream reciben datos concretos, no relaciones internas.
- GIVEN se realiza una carga centralizada de festivos (RF-FES-003) para múltiples establecimientos con "Horario habitual" WHEN cada establecimiento tiene períodos con horarios diferentes THEN cada festivo hereda el horario del período de su propio establecimiento.

## UX Design

### Wireframe textual

Esta spec no tiene pantalla nueva. Modifica el comportamiento del Dialog de festivos (SPEC-007) y la lista de festivos:

**En la lista de festivos (SPEC-007):**
- Festivos con horario heredado: columna de horario muestra "10:00 - 22:00 (habitual)" con "(habitual)" en texto muted.
- Festivos en estado de conflicto (período eliminado, día cerrado): icono AlertTriangle (16px, destructive) + tooltip explicativo.

**En el Dialog de creación/edición (SPEC-007):**
- El texto informativo bajo "Horario habitual del período" se actualiza dinámicamente al cambiar la fecha.
- Si hay conflicto: fondo del bloque de horario con borde destructive, mensaje de advertencia visible, botón "Crear" desactivado.

### Componentes shadcn utilizados

Componentes: todos los de SPEC-007. No se requieren componentes adicionales.

### Patrón de interacción

- **Texto informativo dinámico** que cambia al seleccionar otra fecha: el usuario ve inmediatamente qué horario se aplicará sin tener que guardar y comprobar después. (Decisión no cubierta explícitamente por el design system. Se resuelve con texto reactivo debajo del RadioGroup, actualizado on change del campo de fecha.)
- **Bloqueo del guardado en conflicto** en lugar de permitir guardar con advertencia: un festivo de apertura sin horario resolvible generaría el bug de disponibilidad 24/7 (SPEC-006). El bloqueo es la única opción segura.
- **Indicador "(habitual)" en la lista** en lugar de no distinguir: el usuario necesita saber de un vistazo qué festivos tienen horario propio y cuáles dependen del período, especialmente si va a modificar el período.

### Comportamiento responsive

- **Mobile (< md):** El texto informativo dinámico se muestra igual. El indicador "(habitual)" se mantiene. Los mensajes de conflicto se muestran debajo del bloque del festivo.
- **Tablet (md-lg):** Interpolado.
- **Desktop (lg+):** Como descrito.

## Notas técnicas

- La herencia se resuelve en tiempo real, no se almacena una copia de las franjas. El modelo de datos de SPEC-007 ya lo contempla: `inherit_period_schedule = true` significa que el festivo no tiene registros en `holiday_slots`.
- La resolución del horario heredado sigue este algoritmo: (1) Encontrar el período cuyo `start_date <= holiday.date <= end_date`. (2) Si existe, obtener las franjas de `period_slots` donde `period_id = período encontrado` y `day_of_week = day_of_week(holiday.date)`. (3) Si no hay franjas para ese día o no hay período, el festivo está en conflicto.
- El endpoint de API debe resolver la herencia antes de devolver datos: los sistemas downstream nunca reciben `inherit_period_schedule = true`, siempre reciben las franjas concretas.
- Dependencia upstream: SPEC-007 (registro de festivos), SPEC-002 (franjas por día), SPEC-003 (horario partido), SPEC-004 (división de períodos), SPEC-006 (prevención 24/7).
- Dependencia downstream: RF-INT-001 (la API debe resolver la herencia).
