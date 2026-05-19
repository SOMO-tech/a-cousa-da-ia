# SPEC-010 — Cambio puntual de horario

> RF: RF-EXC-001
> Criticidad: Must
> Wave: 2 — Festivos y excepciones

## Descripción

Permite registrar un día concreto con un horario diferente al habitual del período, sin necesidad de crear un período nuevo ni de que ese día sea festivo. Cubre casuísticas como eventos especiales, inventarios, jornadas de puertas abiertas o cualquier motivo puntual que altere el horario de un día concreto. El sistema legacy obligaba a crear períodos adicionales para cada variación, generando fragmentación innecesaria. Esta funcionalidad permite mantener un único período anual y gestionar las variaciones como modificaciones puntuales.

## Criterios de aceptación

### Visualización de cambios puntuales

- GIVEN el usuario está en la página de horario comercial de un establecimiento WHEN la página carga THEN se muestra una sección "Cambios puntuales de horario" debajo de la sección de festivos, con la lista de cambios definidos para ese año ordenados por fecha ascendente.
- GIVEN existen cambios puntuales definidos WHEN la página carga THEN cada cambio muestra: fecha, motivo, horario (franjas definidas o "Cerrado") y un indicador visual del día de la semana.
- GIVEN no existen cambios puntuales para el año seleccionado WHEN la página carga THEN se muestra un empty state con icono CalendarClock, mensaje "No hay cambios puntuales de horario para [año]" y botón "Añadir cambio puntual".
- GIVEN la sección está cargando datos WHEN el fetch está en curso THEN se muestran Skeleton placeholders.
- GIVEN el fetch falla WHEN la página intenta cargar THEN se muestra un error state con mensaje descriptivo y botón "Reintentar".

### Creación de cambio puntual

- GIVEN el usuario hace clic en "Añadir cambio puntual" WHEN se abre el Dialog THEN se muestran los campos: Fecha (date picker, requerido), Motivo (texto, requerido, placeholder "Ej: Inventario, Evento especial"), Tipo de cambio (RadioGroup: "Horario diferente" por defecto, "Cierre puntual").
- GIVEN el tipo es "Horario diferente" WHEN el usuario ve el Dialog THEN aparecen campos de hora inicio y hora fin para definir la franja (máximo 2 franjas, mismo patrón que SPEC-003). Los campos aparecen pre-rellenados con el horario habitual del período para el día de la semana de la fecha seleccionada (mismo comportamiento que SPEC-009 al cambiar a "Horario especial").
- GIVEN el tipo es "Cierre puntual" WHEN el usuario ve el Dialog THEN no se muestran campos de horario. Se muestra un texto informativo: "El establecimiento permanecerá cerrado este día".
- GIVEN el usuario rellena todos los campos correctamente WHEN hace clic en "Crear" THEN el cambio se guarda, el Dialog se cierra, la lista se actualiza y aparece un Toast "Cambio puntual registrado".
- GIVEN el usuario hace clic en "Cancelar" o pulsa Escape WHEN el Dialog está abierto THEN el Dialog se cierra sin guardar cambios.

### Validación

- GIVEN el Dialog de creación está abierto WHEN el usuario deja el campo Motivo vacío y sale del campo THEN se muestra error inline "El motivo es obligatorio".
- GIVEN el Dialog de creación está abierto WHEN el usuario selecciona una fecha que ya tiene un festivo registrado THEN se muestra error inline "Esta fecha ya tiene un festivo registrado: [nombre]. Edita el festivo si necesitas cambiar su horario".
- GIVEN el Dialog de creación está abierto WHEN el usuario selecciona una fecha que ya tiene otro cambio puntual THEN se muestra error inline "Ya existe un cambio puntual para el [fecha]: [motivo]".
- GIVEN el tipo es "Horario diferente" WHEN el usuario introduce una hora fin igual o anterior a la hora inicio THEN se muestra error inline "La hora de cierre debe ser posterior a la hora de apertura".
- GIVEN el tipo es "Horario diferente" y hay 2 franjas WHEN las franjas se solapan THEN se muestra error inline de solapamiento (misma validación que SPEC-003).
- GIVEN el tipo es "Horario diferente" WHEN el usuario elimina todas las franjas sin cambiar a "Cierre puntual" THEN se aplica la validación de SPEC-006: el botón "Crear" permanece desactivado.

### Edición de cambio puntual

- GIVEN existe un cambio puntual WHEN el usuario hace clic en la acción "Editar" THEN se abre el Dialog con los datos actuales pre-cargados.
- GIVEN el Dialog de edición está abierto WHEN el usuario modifica los campos y hace clic en "Guardar" THEN los cambios se aplican, el Dialog se cierra, la lista se actualiza y aparece un Toast "Cambio puntual actualizado".
- GIVEN el usuario cambia el tipo de "Horario diferente" a "Cierre puntual" WHEN hace clic en "Guardar" THEN las franjas horarias asociadas se eliminan.
- GIVEN el usuario cambia el tipo de "Cierre puntual" a "Horario diferente" WHEN el Dialog actualiza la vista THEN los campos de franjas aparecen pre-rellenados con el horario habitual del período para ese día de la semana.

### Eliminación de cambio puntual

- GIVEN existe un cambio puntual WHEN el usuario hace clic en la acción "Eliminar" THEN se muestra un AlertDialog: "Eliminar cambio puntual", descripción "Se eliminará el cambio puntual del [fecha] ([motivo]). El día volverá al horario habitual del período. Esta acción no se puede deshacer.", botones "Cancelar" / "Eliminar".
- GIVEN el AlertDialog está abierto WHEN el usuario hace clic en "Eliminar" THEN el cambio se elimina, la lista se actualiza y aparece un Toast "Cambio puntual eliminado. El día vuelve al horario habitual".
- GIVEN el AlertDialog está abierto WHEN el usuario hace clic en "Cancelar" o pulsa Escape THEN el AlertDialog se cierra sin eliminar.

### Comparación con horario habitual

- GIVEN el tipo es "Horario diferente" WHEN el usuario ve los campos de franjas THEN se muestra un texto comparativo debajo: "Horario habitual del [día de la semana]: [horario del período]" (texto muted, small).
- GIVEN el tipo es "Horario diferente" WHEN el horario introducido coincide exactamente con el habitual del período THEN se muestra sugerencia inline: "Este horario coincide con el habitual. ¿Seguro que necesitas un cambio puntual?" (texto muted).

### Prioridad sobre el horario del período

- GIVEN existe un cambio puntual para una fecha WHEN el sistema resuelve el horario de ese día THEN el cambio puntual tiene prioridad sobre el horario del período. El día muestra el horario del cambio puntual, no el del período.
- GIVEN existe un cambio puntual de tipo "Cierre puntual" para una fecha WHEN el sistema resuelve el horario THEN el día se considera cerrado, independientemente del horario del período.

### Edge cases

- GIVEN existe un cambio puntual para una fecha WHEN esa fecha también es festivo THEN el festivo tiene prioridad (no puede haber ambos para la misma fecha, la validación lo impide en creación).
- GIVEN existe un cambio puntual WHEN el usuario elimina o modifica el período que contiene esa fecha THEN el cambio puntual se mantiene intacto (tiene franjas propias o es cierre puntual, no depende del período).
- GIVEN la fecha del cambio puntual no cae dentro de ningún período WHEN el usuario crea el cambio THEN se muestra advertencia inline (no bloqueante): "Esta fecha no está cubierta por ningún período".
- GIVEN el usuario selecciona una fecha pasada WHEN crea un cambio puntual THEN se permite (puede ser necesario corregir datos retroactivamente), con advertencia: "Esta fecha ya ha pasado".

## UX Design

### Wireframe textual

**Sección de Cambios puntuales (dentro de la página de Horario Comercial, Layout 1)**

La sección se ubica debajo de la sección de festivos:

Zona de título de sección:
- Heading "Cambios puntuales de horario" (h3, font-medium).
- Contador "(N cambios)" (texto muted, inline).
- Botón "Añadir cambio puntual" (Button, variant default, icono Plus, small), alineado a la derecha.

Lista de cambios puntuales:
- Misma estructura que la lista de festivos (SPEC-007), formato de fila compacta:
  - Columna 1: Fecha (formato locale, font-medium) + día de la semana en texto muted small debajo (ej: "15/03/2026" y "domingo").
  - Columna 2: Motivo (texto).
  - Columna 3: Badge de tipo ("Horario especial" variant outline, o "Cierre puntual" variant secondary).
  - Columna 4: Horario si es de tipo "Horario diferente" (ej: "10:00 - 18:00"), vacío si es cierre puntual.
  - Columna 5: Acciones. DropdownMenu (icono MoreHorizontal) con "Editar" y "Eliminar" (variant destructive, con separador).

**Dialog de creación/edición de cambio puntual**

- Título: "Añadir cambio puntual" o "Editar cambio puntual".
- Campos (stack vertical):
  - Fecha (Input type date, requerido).
  - Motivo (Input, placeholder "Ej: Inventario, Evento especial", requerido).
  - Tipo de cambio (RadioGroup vertical):
    - "Horario diferente — El establecimiento abre con horario distinto al habitual" (por defecto).
    - "Cierre puntual — El establecimiento permanece cerrado".
  - [Visible solo si tipo = Horario diferente] Franjas horarias: campos de hora inicio y hora fin, pre-rellenados con horario habitual. Botón "Añadir franja" (máximo 2). Texto comparativo con horario habitual debajo.
- Botones en footer: "Cancelar" (Button variant outline) / "Crear" o "Guardar" (Button variant default).

### Componentes shadcn utilizados

Componentes: Button, Input, Badge, Toast, AlertDialog, DropdownMenu, Skeleton, Dialog, Form, RadioGroup (todos ya presentes en specs anteriores). No se requieren componentes adicionales.

### Patrón de interacción

- **Lista compacta** para los cambios puntuales: mismo patrón que festivos (SPEC-007). Un establecimiento tiene típicamente 5-10 cambios puntuales por año. (Regla: List para items simples.)
- **Dialog para creación/edición:** 3-5 campos según tipo. Misma justificación que SPEC-007. (Regla: Dialog para 1-4 campos. En el caso máximo con franjas, está en el límite, pero el caso habitual es rápido.)
- **Pre-rellenado de franjas con horario habitual:** el usuario modifica solo lo que cambia, no introduce todo desde cero. (Decisión de UX: minimizar entrada de datos, coherente con SPEC-009.)
- **RadioGroup para tipo de cambio:** 2 opciones visibles simultáneamente con descripción. (Regla: RadioGroup para 2-4 opciones.)
- **Texto comparativo con horario habitual:** mismo patrón que SPEC-009, el usuario valida su modificación contra la referencia.
- **Mensaje descriptivo en el AlertDialog de eliminación:** incluye "el día volverá al horario habitual" para que el usuario entienda la consecuencia.

### Comportamiento responsive

- **Mobile (< md):** Lista en layout vertical (cada cambio es un bloque). Dialog ocupa ancho completo. El día de la semana se muestra inline con la fecha.
- **Tablet (md-lg):** Interpolado.
- **Desktop (lg+):** Layout completo como descrito.

## Notas técnicas

- Modelo de datos: tabla `schedule_overrides` con `id`, `establishment_id`, `date`, `reason` (texto), `type` (enum: 'modified', 'closed'). Si type = 'modified', las franjas se almacenan en `override_slots` con estructura: `override_id`, `start_time`, `end_time`, `slot_order`.
- Prioridad de resolución del horario de un día: (1) Festivo → (2) Cambio puntual → (3) Horario del período. El primero que aplica gana.
- La restricción de unicidad es por `(establishment_id, date)` en la tabla `schedule_overrides`. Además, no puede existir un festivo y un cambio puntual para la misma fecha (validación cross-table).
- Los cambios puntuales son independientes del período: tienen franjas propias y no se ven afectados por modificaciones del período.
- Dependencia upstream: SPEC-002 (patrón de franjas), SPEC-003 (horario partido), SPEC-006 (prevención 24/7), SPEC-007 (festivos, para validación de conflicto de fecha).
- Dependencia downstream: RF-EXC-003 (edición in situ, que esta spec ya cubre con el Dialog de edición).
