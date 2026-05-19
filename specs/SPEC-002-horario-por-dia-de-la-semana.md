# SPEC-002 — Horario por día de la semana

> RF: RF-HOR-004
> Criticidad: Must
> Wave: 1 — Modelo de datos core

## Descripción

Permite definir las franjas horarias de apertura para cada día de la semana dentro de un período. Es el mecanismo core que establece cuándo el establecimiento está abierto. El sistema muestra los 7 días (lunes a domingo) y permite asignar a cada uno una franja de apertura (hora inicio y hora fin). Si un día no tiene franja, se considera cerrado. La funcionalidad de copiar el horario de un día a otros días reduce la repetición en establecimientos con horario uniforme.

## Criterios de aceptación

### Visualización del horario semanal

- GIVEN el usuario tiene un período creado (SPEC-001) WHEN hace clic en la Card del período THEN la Card se expande mostrando el editor de horario semanal con los 7 días de la semana.
- GIVEN el período tiene franjas horarias definidas WHEN la Card se expande THEN cada día muestra su franja horaria (hora inicio - hora fin) o "Cerrado" si no tiene franja asignada.
- GIVEN el período es de tipo cierre WHEN el usuario intenta expandir la Card THEN no se muestra el editor de horario semanal (un período de cierre no tiene franjas).
- GIVEN el período no tiene ninguna franja definida WHEN la Card se expande THEN todos los días aparecen como "Sin horario" con un mensaje indicando "Define el horario de apertura para cada día de la semana" y el primer día (lunes) en modo edición automáticamente.

### Definición de franja horaria

- GIVEN la Card de período está expandida WHEN el usuario hace clic en la fila de un día THEN la fila entra en modo edición mostrando campos de hora inicio y hora fin.
- GIVEN una fila de día está en modo edición WHEN el usuario introduce hora inicio y hora fin válidas y hace clic fuera de la fila o pulsa Enter THEN la franja se asigna al día y la fila vuelve a modo lectura mostrando el horario.
- GIVEN una fila de día está en modo edición WHEN el usuario deja hora inicio vacía y hora fin con valor (o viceversa) y sale del campo THEN se muestra error inline "Ambos campos son obligatorios para definir una franja".
- GIVEN una fila de día está en modo edición WHEN el usuario introduce una hora fin igual o anterior a la hora inicio THEN se muestra error inline "La hora de cierre debe ser posterior a la hora de apertura".
- GIVEN una fila de día tiene una franja asignada WHEN el usuario hace clic en el botón de eliminar franja (icono X) THEN la franja se elimina y el día pasa a mostrar "Cerrado".

### Copiar horario a otros días

- GIVEN un día tiene una franja horaria definida WHEN el usuario hace clic en el botón "Copiar a otros días" de esa fila THEN se muestra un popover con checkboxes para los 6 días restantes, todos deseleccionados por defecto.
- GIVEN el popover de copia está visible WHEN el usuario selecciona uno o más días y hace clic en "Aplicar" THEN la franja horaria se copia a los días seleccionados, el popover se cierra y los días actualizados reflejan el nuevo horario.
- GIVEN el popover de copia está visible WHEN el usuario selecciona días que ya tienen franja definida THEN se muestra un aviso inline en el popover: "Los días con horario existente serán sobreescritos".
- GIVEN el popover de copia está visible WHEN el usuario hace clic fuera del popover o en "Cancelar" THEN el popover se cierra sin cambios.

### Guardado

- GIVEN el usuario ha realizado cambios en las franjas horarias WHEN hay cambios sin guardar THEN se muestra un indicador visual "Cambios sin guardar" y se activa el botón "Guardar horario" en la zona inferior de la Card expandida.
- GIVEN hay cambios sin guardar WHEN el usuario hace clic en "Guardar horario" THEN los cambios se persisten, el botón se desactiva, el indicador desaparece y se muestra un Toast "Horario guardado".
- GIVEN hay cambios sin guardar WHEN el usuario intenta colapsar la Card THEN se muestra un AlertDialog: "Tienes cambios sin guardar. ¿Descartar cambios?" con botones "Seguir editando" / "Descartar".
- GIVEN no hay cambios pendientes WHEN el usuario colapsa la Card THEN la Card se colapsa directamente mostrando el resumen actualizado del horario.

### Resumen en Card colapsada

- GIVEN un período tiene franjas definidas para todos los días WHEN la Card está colapsada THEN muestra un resumen compacto agrupando días con el mismo horario (ej: "L-S 10:00-22:00, D 12:00-21:00").
- GIVEN un período tiene días sin franja (cerrados) WHEN la Card está colapsada THEN el resumen incluye los días cerrados (ej: "L-S 10:00-22:00, D Cerrado").
- GIVEN un período no tiene ninguna franja definida WHEN la Card está colapsada THEN muestra "Sin franjas definidas" en texto muted cursiva con un indicador visual de estado incompleto.

### Edge cases

- GIVEN el usuario está editando una fila WHEN hace clic en otra fila para editarla THEN la primera fila guarda su estado actual (si válido) o revierte (si inválido) y la nueva fila entra en edición.
- GIVEN el usuario introduce "00:00" como hora inicio y "00:00" como hora fin WHEN sale del campo THEN se muestra error inline "La hora de cierre debe ser posterior a la hora de apertura" (00:00-00:00 no es un rango válido).
- GIVEN el usuario introduce "00:00" como hora inicio y "23:59" como hora fin WHEN sale del campo THEN se acepta como válido (apertura 24h menos 1 minuto).

## UX Design

### Wireframe textual

**Card de período expandida (dentro de la página de Horario Comercial, Layout 1)**

La Card del período (definida en SPEC-001) al hacer clic se expande verticalmente para mostrar el editor de horario semanal:

Zona de resumen del período (siempre visible):
- Nombre del período, rango de fechas, Badge "Cierre" si aplica, DropdownMenu de acciones (heredado de SPEC-001).
- Icono chevron (ChevronDown/ChevronUp) indicando que la Card es expandible.

Zona expandida (editor de horario semanal):
- Grid de 7 filas, una por día:
  - Columna 1: Nombre del día (Label, font-medium, ancho fijo). "Lunes", "Martes", ..., "Domingo".
  - Columna 2: Hora inicio (Input type time, ancho 100px) — solo visible en modo edición. En modo lectura muestra el valor como texto.
  - Columna 3: Separador "—" (texto muted).
  - Columna 4: Hora fin (Input type time, ancho 100px) — solo visible en modo edición. En modo lectura muestra el valor como texto.
  - Columna 5: Estado/valor. En modo lectura: texto del horario ("10:00 — 22:00") o "Cerrado" (Badge, variant secondary). En modo edición: campos de input.
  - Columna 6: Acciones por fila. Botón "Copiar a otros días" (icono Copy, variant ghost, solo visible si el día tiene franja). Botón eliminar franja (icono X, variant ghost, solo visible si el día tiene franja).
- Fila en modo edición: fondo accent (highlight sutil), campos de input visibles, botón confirmar (icono Check) y cancelar (icono X) inline.

Zona inferior de la Card expandida:
- Indicador "Cambios sin guardar" (texto muted + punto naranja) a la izquierda.
- Botón "Guardar horario" (Button, variant default) a la derecha. Desactivado si no hay cambios (con Tooltip "No hay cambios pendientes").

**Popover de "Copiar a otros días"**

- Aparece anclado al botón de copiar del día origen.
- Título: "Copiar horario de [Día]" (texto small, muted).
- Lista de 6 checkboxes (los 6 días restantes) con label del nombre del día.
- Si algún día seleccionado tiene franja existente: texto de aviso "Los días con horario existente serán sobreescritos" (texto destructive, small).
- Botones: "Cancelar" (Button variant ghost, small) y "Aplicar" (Button variant default, small).

### Componentes shadcn utilizados

Componentes: Card, Button, Input, Badge, Toast, AlertDialog, Tooltip, Skeleton

Componente adicional necesario: Popover (no incluido en la lista base del scaffold).
Componente adicional necesario: Checkbox (no incluido en la lista base del scaffold).

### Patrón de interacción

- **Card expandible** en lugar de navegación a página de detalle: el horario semanal es un dato simple (7 filas × 2 campos) que no justifica una navegación completa. El usuario necesita ver el contexto de los otros períodos mientras edita. (Regla del design system: Sheet para 5-10 campos cuando el usuario necesita ver la página detrás. En este caso la Card expandible cumple el mismo principio sin superponer contenido.)
- **Edición inline por fila** en lugar de formulario completo: cada día es una unidad atómica de 2 campos. No hay dependencia entre días. Editar inline reduce clics y mantiene el contexto visual. (Decisión no cubierta explícitamente por el design system como patrón de edición inline en grid. Se resuelve con edición fila-a-fila por analogía con la edición de celdas en tablas, dado que el volumen es fijo y pequeño — siempre 7 filas.)
- **Popover para copiar a otros días** en lugar de Dialog: la interacción es ligera (seleccionar checkboxes y confirmar), dura menos de 10 segundos y no necesita bloquear la pantalla. (Regla: Dialog para 1-4 campos. Popover es más ligero aún — interacción contextual anclada al elemento.)
- **Guardado explícito con botón** en lugar de auto-save: el usuario puede modificar varios días antes de confirmar. El guardado explícito permite descartar todos los cambios si se equivoca. (Regla: botones de acción en sticky bottom bar para formularios con scroll — adaptado aquí a la zona inferior de la Card expandida.)
- **AlertDialog al intentar descartar cambios:** protege contra pérdida accidental de trabajo. Foco en "Seguir editando". (Regla: AlertDialog siempre antes de acción irreversible — descartar cambios no guardados.)
- **Toast tras guardar:** variant default, "Horario guardado", 3-5 seg, bottom-right. (Regla: Toast después de acción mutadora exitosa.)
- **Tooltip en botón desactivado:** "No hay cambios pendientes" cuando Guardar está desactivado. (Regla: botón disabled siempre con Tooltip explicativo.)

### Comportamiento responsive

- **Mobile (< md):** La Card expandida ocupa ancho completo. El grid de días cambia a layout vertical: cada día es un bloque con nombre del día como label arriba y los campos de hora debajo en una fila. Los botones de acción por fila (copiar, eliminar) se agrupan a la derecha del bloque. El Popover de copia se reposiciona centrado en viewport si no cabe anclado.
- **Tablet (md-lg):** Interpolado. Grid de días en formato tabla con columnas más estrechas. Sidebar colapsada a iconos.
- **Desktop (lg+):** Layout completo como descrito en el wireframe. Grid horizontal con todas las columnas visibles. La Card expandida aprovecha el ancho disponible.

## Notas técnicas

- Modelo de datos: cada franja es un registro con `period_id`, `day_of_week` (0-6 o ISO: 1=lunes, 7=domingo), `start_time` (HH:MM), `end_time` (HH:MM).
- Un día sin registro = cerrado. No se almacena explícitamente un registro de "cerrado".
- La funcionalidad de "Copiar a otros días" es client-side (duplica los valores localmente antes del guardado). No es una operación de backend separada.
- El resumen compacto de la Card colapsada se genera client-side agrupando días consecutivos con el mismo horario. Algoritmo: iterar L-D, agrupar días adyacentes con mismo start/end, formatear como "L-V 10:00-22:00".
- Dependencia upstream: requiere que exista un período (SPEC-001). El editor solo aparece dentro de una Card de período.
- Dependencia downstream: RF-HOR-005 (horario partido) extenderá este editor para soportar múltiples franjas por día.
- La hora se almacena en formato 24h (HH:MM). No se almacenan segundos.
- No incluir el tiempo de recogida en la hora de fin (según manual legacy: si cierra al público a las 21:00 con 30 min de recogida, la hora fin es 21:00).
