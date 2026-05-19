# SPEC-011 — Cierre temporal por obras o emergencia

> RF: RF-EXC-002
> Criticidad: Must
> Wave: 2 — Festivos y excepciones

## Descripción

Permite registrar un rango de fechas durante el cual el establecimiento permanece cerrado por un motivo extraordinario (obras, reforma, emergencia, daños por agua, etc.). A diferencia del "Cierre puntual" de SPEC-010 que cubre un solo día, el cierre temporal abarca múltiples días consecutivos. Y a diferencia de un "Período de cierre" de SPEC-001 que forma parte de la estructura estacional, el cierre temporal es una interrupción no planificada que no debería afectar a la configuración habitual del establecimiento.

## Criterios de aceptación

### Visualización de cierres temporales

- GIVEN el usuario está en la página de horario comercial de un establecimiento WHEN la página carga THEN se muestra una sección "Cierres temporales" debajo de la sección de cambios puntuales, con la lista de cierres temporales definidos para ese año ordenados por fecha de inicio ascendente.
- GIVEN existen cierres temporales definidos WHEN la página carga THEN cada cierre muestra: rango de fechas (inicio - fin), duración en días, motivo y estado (Badge "Activo" si el rango incluye la fecha actual, "Programado" si es futuro, "Finalizado" si es pasado).
- GIVEN no existen cierres temporales para el año seleccionado WHEN la página carga THEN se muestra un empty state con icono Construction, mensaje "No hay cierres temporales para [año]" y botón "Registrar cierre temporal".
- GIVEN la sección está cargando datos WHEN el fetch está en curso THEN se muestran Skeleton placeholders.
- GIVEN el fetch falla WHEN la página intenta cargar THEN se muestra un error state con mensaje descriptivo y botón "Reintentar".

### Creación de cierre temporal

- GIVEN el usuario hace clic en "Registrar cierre temporal" WHEN se abre el Dialog THEN se muestran los campos: Fecha de inicio (date picker, requerido), Fecha de fin (date picker, requerido), Motivo (texto, requerido, placeholder "Ej: Obras de reforma, Daños por inundación").
- GIVEN el Dialog está abierto WHEN el usuario selecciona fecha de inicio y fecha de fin válidas THEN se muestra un texto informativo: "El establecimiento estará cerrado durante [N] días (del [fecha inicio] al [fecha fin])".
- GIVEN el usuario rellena todos los campos correctamente WHEN hace clic en "Registrar" THEN el cierre se guarda, el Dialog se cierra, la lista se actualiza y aparece un Toast "Cierre temporal registrado".
- GIVEN el usuario hace clic en "Cancelar" o pulsa Escape WHEN el Dialog está abierto THEN el Dialog se cierra sin guardar cambios.

### Validación

- GIVEN el Dialog está abierto WHEN el usuario deja el campo Motivo vacío y sale del campo THEN se muestra error inline "El motivo del cierre es obligatorio".
- GIVEN el Dialog está abierto WHEN el usuario selecciona una fecha de fin anterior a la fecha de inicio THEN se muestra error inline "La fecha de fin debe ser igual o posterior a la fecha de inicio".
- GIVEN el Dialog está abierto WHEN el rango de fechas solapa con otro cierre temporal existente THEN se muestra error inline "Este cierre solapa con '[motivo del existente]' ([fechas]). Modifica las fechas o edita el cierre existente".
- GIVEN el Dialog está abierto WHEN el rango de fechas incluye días que tienen festivos registrados THEN se muestra advertencia inline (no bloqueante): "El rango incluye [N] festivo(s): [nombres]. El cierre temporal tendrá prioridad sobre los festivos durante estas fechas".
- GIVEN el Dialog está abierto WHEN el rango de fechas incluye días que tienen cambios puntuales registrados THEN se muestra advertencia inline (no bloqueante): "El rango incluye [N] cambio(s) puntual(es). El cierre temporal tendrá prioridad durante estas fechas".
- GIVEN el Dialog está abierto WHEN el usuario selecciona fecha inicio = fecha fin THEN se acepta (cierre de 1 día), con sugerencia: "Para un cierre de un solo día, también puedes usar un 'Cierre puntual' desde la sección de cambios puntuales".

### Edición de cierre temporal

- GIVEN existe un cierre temporal WHEN el usuario hace clic en la acción "Editar" THEN se abre el Dialog con los datos actuales pre-cargados.
- GIVEN el Dialog de edición está abierto WHEN el usuario modifica las fechas o el motivo y hace clic en "Guardar" THEN los cambios se aplican, el Dialog se cierra, la lista se actualiza y aparece un Toast "Cierre temporal actualizado".
- GIVEN el cierre temporal está activo (incluye la fecha actual) WHEN el usuario acorta la fecha de fin a una fecha pasada THEN el cierre se marca como "Finalizado" inmediatamente.
- GIVEN el cierre temporal está activo WHEN el usuario extiende la fecha de fin THEN se actualiza la duración y el cierre sigue activo.

### Finalización anticipada

- GIVEN un cierre temporal está activo o programado WHEN el usuario hace clic en "Finalizar ahora" en el DropdownMenu THEN se muestra un AlertDialog: "Finalizar cierre temporal", descripción "Se finalizará el cierre '[motivo]'. La fecha de fin se ajustará a hoy ([fecha actual]). El establecimiento volverá a su horario habitual a partir de mañana.", botones "Cancelar" / "Finalizar".
- GIVEN el usuario confirma la finalización WHEN hace clic en "Finalizar" THEN la fecha de fin del cierre se actualiza a la fecha actual, el estado cambia a "Finalizado" y aparece un Toast "Cierre temporal finalizado. El horario habitual se restablece a partir de mañana".

### Eliminación de cierre temporal

- GIVEN existe un cierre temporal WHEN el usuario hace clic en la acción "Eliminar" THEN se muestra un AlertDialog: "Eliminar cierre temporal", descripción "Se eliminará el cierre '[motivo]' ([fecha inicio] - [fecha fin]). Los días afectados volverán al horario habitual del período. Esta acción no se puede deshacer.", botones "Cancelar" / "Eliminar".
- GIVEN el AlertDialog está abierto WHEN el usuario hace clic en "Eliminar" THEN el cierre se elimina, la lista se actualiza y aparece un Toast "Cierre temporal eliminado".
- GIVEN el AlertDialog está abierto WHEN el usuario hace clic en "Cancelar" THEN el AlertDialog se cierra sin eliminar.

### Prioridad de resolución

- GIVEN existe un cierre temporal que abarca un rango de fechas WHEN el sistema resuelve el horario de cualquier día dentro del rango THEN el cierre temporal tiene prioridad máxima. Todos los días del rango se consideran cerrados.
- GIVEN existe un cierre temporal y un festivo de apertura para el mismo día WHEN el sistema resuelve el horario THEN el cierre temporal gana: el establecimiento está cerrado ese día (el cierre temporal es una emergencia/obra que invalida cualquier apertura planificada).
- GIVEN existe un cierre temporal y un cambio puntual para el mismo día WHEN el sistema resuelve el horario THEN el cierre temporal gana.

### Edge cases

- GIVEN un cierre temporal abarca fechas en dos años diferentes (ej: diciembre a enero) WHEN el usuario está en la vista del año actual THEN se muestra la parte del cierre que cae en el año seleccionado, con indicador de que continúa en el año siguiente (o viene del anterior).
- GIVEN un cierre temporal está programado para el futuro WHEN el usuario lo elimina THEN se elimina sin consecuencias (no hay datos afectados aún).
- GIVEN un cierre temporal de 1 solo día WHEN el usuario lo crea THEN se acepta pero se sugiere usar "Cierre puntual" de SPEC-010.

## UX Design

### Wireframe textual

**Sección de Cierres temporales (dentro de la página de Horario Comercial, Layout 1)**

La sección se ubica debajo de la sección de cambios puntuales:

Zona de título de sección:
- Heading "Cierres temporales" (h3, font-medium).
- Contador "(N cierres)" (texto muted, inline).
- Botón "Registrar cierre temporal" (Button, variant default, icono Plus, small), alineado a la derecha.

Lista de cierres temporales:
- Formato de fila compacta:
  - Columna 1: Rango de fechas (font-medium). Formato: "[fecha inicio] — [fecha fin]". Debajo en texto muted small: "[N] días".
  - Columna 2: Motivo (texto).
  - Columna 3: Badge de estado:
    - "Activo" (variant destructive) si la fecha actual está dentro del rango.
    - "Programado" (variant default) si fecha inicio > hoy.
    - "Finalizado" (variant secondary) si fecha fin < hoy.
  - Columna 4: Acciones. DropdownMenu (icono MoreHorizontal) con:
    - "Editar"
    - "Finalizar ahora" (solo visible si estado = Activo o Programado)
    - Separador
    - "Eliminar" (variant destructive)

**Dialog de creación/edición de cierre temporal**

- Título: "Registrar cierre temporal" o "Editar cierre temporal".
- Campos (stack vertical):
  - Fecha de inicio (Input type date, requerido).
  - Fecha de fin (Input type date, requerido).
  - Motivo (Input, placeholder "Ej: Obras de reforma, Daños por inundación", requerido).
- Texto informativo dinámico: "El establecimiento estará cerrado durante [N] días".
- Advertencias inline si hay festivos o cambios puntuales en el rango (no bloqueantes).
- Botones en footer: "Cancelar" (Button variant outline) / "Registrar" o "Guardar" (Button variant default).

### Componentes shadcn utilizados

Componentes: Button, Input, Badge, Toast, AlertDialog, DropdownMenu, Skeleton, Dialog, Form (todos ya presentes en specs anteriores). No se requieren componentes adicionales.

### Patrón de interacción

- **Lista compacta** para cierres temporales: un establecimiento tiene típicamente 0-2 cierres temporales por año. Son eventos excepcionales. (Regla: List para items simples.)
- **Dialog para creación/edición:** 3 campos. Interacción rápida. (Regla: Dialog para 1-4 campos y <10 seg.)
- **Badge de estado con semántica de color:** destructive para "Activo" (atención inmediata), default para "Programado" (futuro), secondary para "Finalizado" (histórico). (Regla: Badge para estados. Variant destructive solo para estados críticos.)
- **"Finalizar ahora" como acción rápida:** evita que el usuario tenga que editar las fechas manualmente para cerrar un cierre que terminó antes de lo previsto. Es un acceso directo frecuente en emergencias que se resuelven antes del plazo estimado. (Decisión no cubierta por el design system. Se resuelve como acción en DropdownMenu con AlertDialog de confirmación.)
- **Advertencias no bloqueantes** para festivos/cambios puntuales en el rango: el cierre temporal tiene prioridad por naturaleza (es una emergencia). Informar pero no bloquear.

### Comportamiento responsive

- **Mobile (< md):** Lista en layout vertical. Cada cierre es un bloque con rango de fechas arriba, motivo y Badge debajo. Dialog ocupa ancho completo.
- **Tablet (md-lg):** Interpolado.
- **Desktop (lg+):** Layout completo como descrito.

## Notas técnicas

- Modelo de datos: tabla `temporary_closures` con `id`, `establishment_id`, `start_date`, `end_date`, `reason` (texto). No tiene franjas horarias (siempre es cierre completo).
- Prioridad de resolución actualizada del horario de un día: (1) Cierre temporal → (2) Festivo → (3) Cambio puntual → (4) Horario del período. El cierre temporal tiene la prioridad más alta porque representa una situación física (obras, daños) que invalida cualquier planificación.
- La restricción de no solapamiento entre cierres temporales se valida client-side y server-side: no puede haber dos cierres temporales con rangos que se solapen para el mismo establecimiento.
- Un cierre temporal que cruza el límite de año (diciembre-enero) se almacena como un solo registro. La vista por año lo muestra parcialmente según el año seleccionado.
- La acción "Finalizar ahora" es un UPDATE de `end_date = CURRENT_DATE`, no una eliminación.
- Dependencia upstream: SPEC-007 (festivos), SPEC-010 (cambios puntuales), para las advertencias de solapamiento.
- Dependencia downstream: RF-ADM-001 (panel de monitorización mostrará cierres activos).
