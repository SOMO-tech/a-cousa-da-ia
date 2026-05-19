# SPEC-014 — Edición directa desde calendario

> RF: RF-UXI-004
> Criticidad: Should
> Wave: 3 — Visualización, copia interanual y validación

## Descripción

Permite al usuario hacer clic en un día del calendario anual (SPEC-013) para acceder directamente a la edición de su configuración, sin tener que navegar a la sección correspondiente (festivos, cambios puntuales, cierres temporales) y buscar el registro. En M7, la vista de calendario era solo informativa y la edición requería navegación indirecta. Esta spec convierte el calendario en un punto de entrada dual: visualización + acción.

## Criterios de aceptación

### Click en un día con estado existente

- GIVEN un día del calendario tiene un festivo asociado WHEN el usuario hace clic en la celda THEN se abre el Dialog de edición de ese festivo (SPEC-007), con todos los campos pre-cargados.
- GIVEN un día del calendario tiene un cambio puntual asociado WHEN el usuario hace clic en la celda THEN se abre el Dialog de edición de ese cambio puntual (SPEC-010), con todos los campos pre-cargados.
- GIVEN un día del calendario está dentro de un cierre temporal WHEN el usuario hace clic en la celda THEN se abre el Dialog de edición de ese cierre temporal (SPEC-011), con todos los campos pre-cargados.
- GIVEN un día del calendario tiene horario normal (sin festivo, ni cambio puntual, ni cierre temporal) WHEN el usuario hace clic en la celda THEN se abre un Popover contextual con la información del día y opciones de acción rápida.

### Popover de día normal

- GIVEN el usuario hace clic en un día con horario normal WHEN se abre el Popover THEN muestra: fecha completa, horario del período para ese día de la semana (ej: "10:00 - 22:00"), y dos botones de acción: "Registrar festivo" y "Añadir cambio puntual".
- GIVEN el Popover está abierto WHEN el usuario hace clic en "Registrar festivo" THEN se abre el Dialog de creación de festivo (SPEC-007) con la fecha pre-seleccionada al día clicado.
- GIVEN el Popover está abierto WHEN el usuario hace clic en "Añadir cambio puntual" THEN se abre el Dialog de creación de cambio puntual (SPEC-010) con la fecha pre-seleccionada al día clicado.
- GIVEN el Popover está abierto WHEN el usuario hace clic fuera del Popover THEN el Popover se cierra.

### Popover de día dentro de cierre temporal (alternativa)

- GIVEN un día está dentro de un cierre temporal WHEN el usuario hace clic en la celda THEN se abre un Popover con: información del cierre temporal (motivo, rango de fechas), y botón "Editar cierre temporal" que abre el Dialog de edición del cierre (SPEC-011).
- GIVEN un día dentro de cierre temporal también es festivo WHEN el Popover se muestra THEN incluye una nota informativa: "Este día también es festivo: [nombre]. El cierre temporal tiene prioridad".

### Indicador de interactividad

- GIVEN la celda de un día tiene un estado (festivo, cambio, cierre) WHEN el usuario pasa el cursor sobre la celda THEN el cursor cambia a pointer y la celda muestra un efecto hover sutil (borde o fondo ligeramente más oscuro).
- GIVEN la celda de un día tiene horario normal WHEN el usuario pasa el cursor THEN el cursor cambia a pointer y la celda muestra el mismo efecto hover.

### Feedback tras acción desde calendario

- GIVEN el usuario edita un registro abriendo el Dialog desde el calendario WHEN guarda cambios y cierra el Dialog THEN la celda del calendario se actualiza inmediatamente para reflejar el nuevo estado (el dot de color puede cambiar si el tipo cambió).
- GIVEN el usuario crea un festivo o cambio puntual desde el Popover del calendario WHEN guarda el nuevo registro y cierra el Dialog THEN la celda del día se actualiza añadiendo el dot del nuevo estado. Las estadísticas del calendario también se actualizan.
- GIVEN el usuario elimina un registro desde el Dialog abierto vía calendario WHEN confirma la eliminación THEN la celda vuelve a su estado base (horario normal o sin cobertura). Las estadísticas se actualizan.

### Edge cases

- GIVEN un día tiene un festivo y el usuario lo elimina desde el Dialog abierto vía calendario WHEN el Dialog se cierra THEN la celda cambia de "festivo" a "horario normal" (o "sin cobertura" si no hay período).
- GIVEN el usuario abre un Dialog desde el calendario y otro usuario modifica el mismo registro WHEN el primer usuario intenta guardar THEN se aplica el manejo de concurrencia de SPEC-012 (error 409).
- GIVEN el usuario está en mobile WHEN toca una celda THEN el Popover/Dialog se comporta igual que en desktop pero ocupa ancho completo.

## UX Design

### Wireframe textual

Esta spec no tiene sección nueva en la página. Modifica las celdas del calendario de SPEC-013:

**Celda de día (modificación sobre SPEC-013):**
- Añade cursor pointer y efecto hover en todas las celdas.
- Click en celda con estado existente → abre Dialog de edición directamente.
- Click en celda con horario normal → abre Popover contextual.

**Popover contextual de día normal:**
- Ancho: 240px (fijo).
- Contenido:
  - Fecha completa (texto small, font-medium).
  - Horario: "[horario del período]" (texto small, muted).
  - Separador.
  - Botón "Registrar festivo" (Button variant ghost, icono CalendarHeart, small, ancho completo).
  - Botón "Añadir cambio puntual" (Button variant ghost, icono CalendarClock, small, ancho completo).

**Popover contextual de día en cierre temporal:**
- Ancho: 280px.
- Contenido:
  - Fecha completa (texto small, font-medium).
  - "Cierre temporal: [motivo]" (texto small).
  - "Del [inicio] al [fin] — [N] días" (texto small, muted).
  - Nota sobre festivo subordinado si aplica (texto xs, muted).
  - Separador.
  - Botón "Editar cierre temporal" (Button variant ghost, icono Pencil, small, ancho completo).

### Componentes shadcn utilizados

Componentes: Popover (ya introducido en SPEC-013 para mobile). No se requieren componentes adicionales.

### Patrón de interacción

- **Click directo en celda con estado → Dialog de edición:** el atajo más rápido posible. El usuario ve un festivo en el calendario, hace clic, y está editándolo. Sin intermediarios. (Decisión de UX: minimizar el número de clics para la acción más frecuente.)
- **Popover para días normales en lugar de Dialog directo:** un día normal no tiene registro que editar. El Popover ofrece las dos acciones posibles sin abrir un formulario vacío. Es un paso intermedio ligero que evita abrir un Dialog innecesario. (Regla: Popover para información contextual breve con 1-2 acciones.)
- **Popover para días en cierre temporal en lugar de Dialog directo:** el cierre temporal abarca un rango. Abrir directamente el Dialog de edición podría confundir al usuario si no recuerda de qué cierre se trata. El Popover contextualiza antes de actuar. (Decisión de UX: dar contexto antes de la acción cuando el registro abarca más de un día.)
- **Actualización reactiva de la celda tras guardado:** el calendario refleja inmediatamente los cambios sin recargar. Esto confirma visualmente que la acción tuvo efecto. (Regla: feedback inmediato tras acción del usuario.)

### Comportamiento responsive

- **Mobile (< md):** El Popover se posiciona debajo de la celda tocada, ocupando ancho disponible. Los Dialogs se abren en ancho completo como en el resto de specs.
- **Tablet (md-lg):** Popover posicionado como en desktop. Dialogs con ancho estándar.
- **Desktop (lg+):** Como descrito.

## Notas técnicas

- La asociación celda → registro se mantiene en memoria (mapa fecha → { tipo, id }). Al hacer clic, se busca en el mapa y se determina qué Dialog abrir y con qué datos.
- No se necesita fetch adicional al hacer clic: los datos del registro ya están cargados (se usan para renderizar el dot de estado). El Dialog de edición recibe los datos directamente desde el state.
- Si el usuario crea un nuevo registro desde el Popover, la fecha se pasa como prop al Dialog. El Dialog de creación la recibe como valor por defecto del campo fecha.
- Tras cerrar un Dialog (guardar o eliminar), se invalida y refresca el estado del calendario. El mapa fecha → registro se actualiza.
- Dependencia upstream: SPEC-013 (calendario anual, vista base), SPEC-007 (Dialog festivos), SPEC-010 (Dialog cambios puntuales), SPEC-011 (Dialog cierres temporales), SPEC-012 (comportamiento de edición estandarizado).
- No hay dependencia downstream directa.
