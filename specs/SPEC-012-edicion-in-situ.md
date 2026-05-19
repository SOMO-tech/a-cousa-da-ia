# SPEC-012 — Edición in situ de excepciones

> RF: RF-EXC-003
> Criticidad: Must
> Wave: 2 — Festivos y excepciones

## Descripción

Garantiza que los festivos (SPEC-007), cambios puntuales de horario (SPEC-010) y cierres temporales (SPEC-011) puedan modificarse directamente sin necesidad de eliminar y recrear el registro. En el sistema legacy, cualquier corrección obligaba a borrar el registro y volver a crearlo desde cero, con riesgo de perder datos o introducir errores. Esta spec formaliza el principio de edición directa ya implementado en las specs anteriores y cubre los edge cases de edición que no se han detallado: cambios de tipo, cambios de fecha, edición de registros pasados y protección contra pérdida de datos durante la edición.

## Criterios de aceptación

### Edición directa de cualquier registro

- GIVEN existe un festivo, cambio puntual o cierre temporal WHEN el usuario hace clic en "Editar" en el DropdownMenu del registro THEN se abre el Dialog de edición con todos los campos pre-cargados con los valores actuales.
- GIVEN el Dialog de edición está abierto WHEN el usuario modifica cualquier campo y hace clic en "Guardar" THEN los cambios se aplican sin eliminar y recrear el registro. El ID del registro se mantiene.
- GIVEN el Dialog de edición está abierto WHEN el usuario no ha modificado ningún campo THEN el botón "Guardar" permanece desactivado con Tooltip "No hay cambios pendientes".

### Cambio de fecha en registros existentes

- GIVEN el usuario edita un festivo WHEN cambia la fecha a una que ya tiene otro festivo THEN se muestra error inline "Ya existe un festivo para el [nueva fecha]: [nombre]" (misma validación que en creación).
- GIVEN el usuario edita un cambio puntual WHEN cambia la fecha a una que tiene un festivo o otro cambio puntual THEN se muestra el error de conflicto correspondiente.
- GIVEN el usuario edita un cierre temporal WHEN cambia las fechas y el nuevo rango solapa con otro cierre temporal THEN se muestra error de solapamiento.
- GIVEN el usuario edita un festivo o cambio puntual WHEN cambia la fecha THEN las franjas horarias asociadas se mantienen (no se pierden al cambiar de fecha).

### Cambio de tipo en registros existentes

- GIVEN el usuario edita un festivo de cierre WHEN cambia el tipo a "Apertura" THEN aparecen los campos de horario (RadioGroup "Horario habitual" / "Horario especial"), pre-seleccionado "Horario habitual del período".
- GIVEN el usuario edita un festivo de apertura con horario especial WHEN cambia el tipo a "Cierre" THEN se muestra confirmación inline: "Se descartarán las franjas horarias definidas (10:00 - 18:00). ¿Continuar?" con opciones "Sí, cambiar a cierre" / "Cancelar cambio".
- GIVEN el usuario edita un cambio puntual de tipo "Horario diferente" WHEN cambia a "Cierre puntual" THEN se muestra la misma confirmación de pérdida de franjas.
- GIVEN el usuario edita un cambio puntual de tipo "Cierre puntual" WHEN cambia a "Horario diferente" THEN los campos de franjas aparecen pre-rellenados con el horario habitual del período (mismo comportamiento que SPEC-010).

### Edición de registros pasados

- GIVEN existe un festivo o cambio puntual con fecha pasada WHEN el usuario hace clic en "Editar" THEN el Dialog se abre normalmente. Se muestra advertencia informativa (no bloqueante) en la parte superior del Dialog: "Este registro corresponde a una fecha pasada ([fecha]). Los cambios se aplicarán retroactivamente".
- GIVEN existe un cierre temporal con estado "Finalizado" WHEN el usuario hace clic en "Editar" THEN el Dialog se abre normalmente con la misma advertencia de fecha pasada.
- GIVEN el usuario edita un registro pasado WHEN hace clic en "Guardar" THEN los cambios se guardan sin restricciones adicionales. El Toast incluye "(retroactivo)": "Festivo actualizado (retroactivo)".

### Protección contra pérdida de datos

- GIVEN el Dialog de edición está abierto y el usuario ha modificado campos WHEN hace clic fuera del Dialog, pulsa Escape o hace clic en "Cancelar" THEN se muestra confirmación: "Tienes cambios sin guardar. ¿Descartar cambios?" con botones "Seguir editando" / "Descartar". Foco en "Seguir editando".
- GIVEN el Dialog de edición está abierto y el usuario NO ha modificado campos WHEN hace clic fuera, pulsa Escape o "Cancelar" THEN el Dialog se cierra directamente sin confirmación.

### Feedback tras edición

- GIVEN el usuario guarda cambios en un festivo WHEN la operación es exitosa THEN aparece Toast "Festivo actualizado" (3-5 seg, bottom-right).
- GIVEN el usuario guarda cambios en un cambio puntual WHEN la operación es exitosa THEN aparece Toast "Cambio puntual actualizado".
- GIVEN el usuario guarda cambios en un cierre temporal WHEN la operación es exitosa THEN aparece Toast "Cierre temporal actualizado".
- GIVEN la operación de guardado falla WHEN el servidor devuelve error THEN se muestra Toast de error (variant destructive): "Error al guardar. Inténtalo de nuevo" y el Dialog permanece abierto con los datos del usuario intactos.

### Edge cases

- GIVEN el usuario edita un festivo de apertura con "Horario habitual" WHEN el período asociado ha sido eliminado entre la creación del festivo y esta edición THEN el Dialog muestra la advertencia de conflicto de SPEC-008 y obliga a definir horario especial.
- GIVEN el usuario edita un cierre temporal activo WHEN cambia la fecha de inicio a una fecha futura THEN el estado cambia a "Programado" y el establecimiento vuelve inmediatamente al horario habitual.
- GIVEN la operación de guardado falla por conflicto de concurrencia (otro usuario modificó el mismo registro) WHEN el servidor devuelve error 409 THEN se muestra Toast de error: "Este registro ha sido modificado por otro usuario. Recarga la página para ver los cambios actuales". El Dialog se cierra.

## UX Design

### Wireframe textual

Esta spec no tiene pantalla nueva. Refuerza y estandariza el comportamiento de edición de los Dialogs de SPEC-007, SPEC-010 y SPEC-011:

**Elementos comunes a todos los Dialogs de edición:**
- Título: "Editar [tipo]" (Editar festivo / Editar cambio puntual / Editar cierre temporal).
- Todos los campos pre-cargados con valores actuales.
- Botón "Guardar" desactivado si no hay cambios (con Tooltip).
- Advertencia de fecha pasada si aplica (Alert inline, variant default, icono Clock, en la parte superior del Dialog).
- Confirmación de descarte si hay cambios sin guardar.

**Confirmación de cambio de tipo (cuando se pierden datos):**
- Texto inline debajo del RadioGroup de tipo, con fondo accent sutil: "Se descartarán las franjas horarias definidas ([horario actual]). ¿Continuar?" con botones inline "Sí, cambiar" (Button variant ghost, small) / "Cancelar cambio" (Button variant ghost, small).

### Componentes shadcn utilizados

Componentes: Dialog, Button, Toast, Tooltip, Alert (todos ya presentes en specs anteriores). No se requieren componentes adicionales.

### Patrón de interacción

- **Edición en el mismo Dialog que creación:** no hay un componente o flujo separado para editar. Es el mismo Dialog con datos pre-cargados. Esto reduce la superficie de la aplicación y mantiene la consistencia. (Regla: reutilizar componentes cuando la estructura es idéntica.)
- **Botón "Guardar" desactivado sin cambios:** evita guardados innecesarios y da señal visual de que no hay nada pendiente. (Regla: botón disabled con Tooltip explicativo.)
- **Confirmación de descarte con foco en "Seguir editando":** protege contra pérdida accidental de trabajo. (Regla: AlertDialog antes de acción irreversible, foco en la opción segura.)
- **Confirmación inline para cambio de tipo** en lugar de AlertDialog: es una decisión dentro del formulario, no una acción destructiva global. El contexto del RadioGroup es suficiente. (Decisión no cubierta explícitamente por el design system. Se resuelve con confirmación inline por analogía con validaciones contextuales.)
- **Toast de error que mantiene el Dialog abierto:** el usuario no pierde su trabajo si falla el servidor. (Regla: los errores de red no deben provocar pérdida de datos del usuario.)

### Comportamiento responsive

- **Mobile (< md):** Mismos comportamientos. Los Dialogs ocupan ancho completo. Las confirmaciones inline se adaptan al ancho.
- **Tablet (md-lg):** Interpolado.
- **Desktop (lg+):** Como descrito.

## Notas técnicas

- La edición es un UPDATE, no un DELETE + INSERT. El ID del registro se preserva. Esto es importante para el historial de cambios (RF-ADM-003) y para las referencias de otros sistemas.
- El conflicto de concurrencia (error 409) se detecta server-side comparando un campo `updated_at` (timestamp) del registro. El cliente envía el `updated_at` que tenía al abrir el Dialog. Si el servidor tiene un `updated_at` más reciente, rechaza la operación.
- La advertencia de "fecha pasada" no bloquea la edición porque hay casos legítimos de corrección retroactiva (ej: se detecta un error en un festivo pasado que afectó a datos downstream).
- Dependencia upstream: SPEC-007 (Dialog de festivos), SPEC-010 (Dialog de cambios puntuales), SPEC-011 (Dialog de cierres temporales).
- Esta spec no introduce funcionalidad nueva sino que estandariza y completa el comportamiento de edición de las tres specs anteriores. Los criterios aquí definidos aplican retroactivamente a los Dialogs de SPEC-007, SPEC-010 y SPEC-011.
