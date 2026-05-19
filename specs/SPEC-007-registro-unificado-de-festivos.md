# SPEC-007 — Registro unificado de festivos

> RF: RF-FES-001
> Criticidad: Must
> Wave: 2 — Festivos y excepciones

## Descripción

Permite registrar los días festivos de un establecimiento en un único flujo, indicando si el establecimiento abre o cierra ese día y, en caso de apertura, con qué horario. Esta funcionalidad reemplaza el proceso dual del sistema legacy donde los festivos se registraban primero en M3 (apertura/cierre) y luego en M7 (horario), eliminando la duplicación de trabajo y el riesgo de inconsistencia entre ambos módulos.

## Criterios de aceptación

### Visualización de festivos

- GIVEN el usuario está en la página de horario comercial de un establecimiento para un año concreto WHEN la página carga THEN se muestra una sección "Festivos" debajo de la sección de períodos, con la lista de festivos definidos para ese año ordenados por fecha ascendente.
- GIVEN existen festivos definidos WHEN la página carga THEN cada festivo muestra: fecha, nombre, tipo (Badge "Cierre" o "Apertura") y, si es de apertura, el horario resumido (ej: "10:00 - 18:00").
- GIVEN no existen festivos definidos para el año seleccionado WHEN la página carga THEN se muestra un empty state con icono CalendarOff, mensaje "No hay festivos registrados para [año]" y botón "Añadir festivo".
- GIVEN la sección de festivos está cargando datos WHEN el fetch está en curso THEN se muestran Skeleton placeholders en el área de la lista de festivos.
- GIVEN el fetch de festivos falla WHEN la página intenta cargar THEN se muestra un error state con mensaje descriptivo y botón "Reintentar".

### Creación de festivo

- GIVEN el usuario hace clic en "Añadir festivo" WHEN se abre el Dialog THEN se muestran los campos: Fecha (date picker, requerido), Nombre del festivo (texto, requerido, placeholder "Ej: Navidad"), Tipo (RadioGroup con opciones "Cierre" y "Apertura", por defecto "Cierre").
- GIVEN el tipo seleccionado es "Cierre" WHEN el usuario ve el Dialog THEN no se muestran campos de horario (el establecimiento permanece cerrado todo el día).
- GIVEN el tipo seleccionado es "Apertura" WHEN el usuario ve el Dialog THEN aparece una sección adicional con dos opciones (RadioGroup): "Horario habitual del período" (por defecto) y "Horario especial".
- GIVEN el tipo es "Apertura" y la opción es "Horario habitual del período" WHEN el usuario ve el Dialog THEN se muestra un texto informativo (muted) con el horario que se heredará del período correspondiente a esa fecha (ej: "Se aplicará el horario del período 'Horario anual': 10:00 - 22:00"). Si la fecha no cae dentro de ningún período, se muestra advertencia: "La fecha seleccionada no está cubierta por ningún período. Define un horario especial".
- GIVEN el tipo es "Apertura" y la opción es "Horario especial" WHEN el usuario ve el Dialog THEN aparecen campos de hora inicio y hora fin para definir la franja (máximo 2 franjas, reutilizando el patrón de SPEC-003).
- GIVEN el usuario rellena todos los campos correctamente WHEN hace clic en "Crear" THEN el festivo se guarda, el Dialog se cierra, la lista se actualiza y aparece un Toast "Festivo registrado".
- GIVEN el usuario hace clic en "Cancelar" o pulsa Escape WHEN el Dialog está abierto THEN el Dialog se cierra sin guardar cambios.

### Validación

- GIVEN el Dialog de creación está abierto WHEN el usuario deja el campo Nombre vacío y sale del campo THEN se muestra error inline "El nombre del festivo es obligatorio".
- GIVEN el Dialog de creación está abierto WHEN el usuario selecciona una fecha que ya tiene un festivo registrado THEN se muestra error inline "Ya existe un festivo para el [fecha]: [nombre del festivo existente]".
- GIVEN el tipo es "Apertura" con "Horario especial" WHEN el usuario introduce una hora fin igual o anterior a la hora inicio THEN se muestra error inline "La hora de cierre debe ser posterior a la hora de apertura" (misma validación que SPEC-002).
- GIVEN el tipo es "Apertura" con "Horario especial" y 2 franjas WHEN las franjas se solapan THEN se muestra error inline de solapamiento (misma validación que SPEC-003).
- GIVEN el tipo es "Apertura" con "Horario especial" WHEN el usuario no define ninguna franja THEN se aplica la validación de SPEC-006: el botón "Crear" permanece desactivado con Tooltip "Un festivo de apertura requiere horario".

### Edición de festivo

- GIVEN existe un festivo WHEN el usuario hace clic en la acción "Editar" del festivo THEN se abre el Dialog con los datos actuales pre-cargados.
- GIVEN el Dialog de edición está abierto WHEN el usuario modifica los campos y hace clic en "Guardar" THEN los cambios se aplican, el Dialog se cierra, la lista se actualiza y aparece un Toast "Festivo actualizado".
- GIVEN el usuario cambia el tipo de "Apertura" a "Cierre" WHEN hace clic en "Guardar" THEN las franjas horarias asociadas se eliminan (un festivo de cierre no tiene horario).

### Eliminación de festivo

- GIVEN existe un festivo WHEN el usuario hace clic en la acción "Eliminar" del festivo THEN se muestra un AlertDialog: "Eliminar festivo", descripción "Se eliminará el festivo [nombre] ([fecha]). Esta acción no se puede deshacer.", botones "Cancelar" / "Eliminar".
- GIVEN el AlertDialog de eliminación está abierto WHEN el usuario hace clic en "Eliminar" THEN el festivo se elimina, el AlertDialog se cierra, la lista se actualiza y aparece un Toast "Festivo eliminado".
- GIVEN el AlertDialog de eliminación está abierto WHEN el usuario hace clic en "Cancelar" o pulsa Escape THEN el AlertDialog se cierra sin eliminar.

### Edge cases

- GIVEN el usuario selecciona una fecha de festivo WHEN la fecha no cae dentro de ningún período definido THEN se muestra advertencia inline (no bloqueante): "Esta fecha no está cubierta por ningún período. El festivo se registrará igualmente".
- GIVEN el usuario crea un festivo de apertura con "Horario habitual" WHEN posteriormente modifica el horario del período asociado THEN el festivo refleja automáticamente el nuevo horario (no almacena una copia, hereda en tiempo real).
- GIVEN el usuario divide un período (SPEC-004) WHEN el período contenía festivos THEN los festivos se reasignan al período cuyo rango contiene la fecha del festivo (ya cubierto en SPEC-004).

## UX Design

### Wireframe textual

**Sección de Festivos (dentro de la página de Horario Comercial, Layout 1)**

La sección se ubica debajo de la sección de períodos, dentro de la misma página:

Zona de título de sección:
- Heading "Festivos" (h3, font-medium).
- Contador "(N festivos)" (texto muted, inline).
- Botón "Añadir festivo" (Button, variant default, icono Plus, small), alineado a la derecha.

Lista de festivos:
- Cada festivo se muestra como una fila dentro de una lista compacta (no Cards individuales, los festivos son datos simples y pueden ser 10-20 por año):
  - Columna 1: Fecha (formato locale, font-medium, ancho fijo).
  - Columna 2: Nombre del festivo (texto).
  - Columna 3: Badge de tipo ("Cierre" variant secondary, "Apertura" variant default).
  - Columna 4: Horario resumido si es de apertura (ej: "10:00 - 18:00" o "Horario habitual"), vacío si es cierre.
  - Columna 5: Acciones. DropdownMenu (icono MoreHorizontal) con opciones "Editar" y "Eliminar" (variant destructive, con separador).

Empty state (si no hay festivos):
- Centrado en el área de la sección. Icono CalendarOff (Lucide, 24px). Mensaje "No hay festivos registrados para [año]". Botón "Añadir festivo" (Button, variant default).

**Dialog de creación/edición de festivo**

- Título: "Añadir festivo" o "Editar festivo".
- Campos (stack vertical):
  - Fecha (Input type date, requerido).
  - Nombre del festivo (Input, placeholder "Ej: Navidad", requerido).
  - Tipo de festivo (RadioGroup vertical):
    - "Cierre — El establecimiento permanece cerrado" (por defecto).
    - "Apertura — El establecimiento abre este día".
  - [Visible solo si tipo = Apertura] Horario de apertura (RadioGroup vertical):
    - "Horario habitual del período" (por defecto). Debajo: texto informativo muted con el horario heredado.
    - "Horario especial". Debajo: campos de hora inicio y hora fin (mismo patrón que SPEC-002), con botón "Añadir franja" (mismo patrón que SPEC-003, máximo 2 franjas).
- Botones en footer: "Cancelar" (Button variant outline) a la izquierda, "Crear" / "Guardar" (Button variant default) a la derecha.

### Componentes shadcn utilizados

Componentes: Button, Input, Badge, Toast, AlertDialog, DropdownMenu, Skeleton, Dialog, Form (todos ya presentes en specs anteriores).

Componente adicional necesario: RadioGroup (no incluido en la lista base del scaffold).

### Patrón de interacción

- **Lista compacta en lugar de Cards** para los festivos: un establecimiento puede tener 10-20 festivos por año. Son datos simples (fecha + nombre + tipo + horario). No justifican Cards individuales. La lista permite escanear rápidamente. (Regla: List para items con 1-2 campos principales y acción secundaria.)
- **Dialog para creación/edición:** 3-5 campos según el tipo seleccionado. La aparición condicional de campos mantiene el Dialog ligero para el caso simple (cierre) y completo para el caso complejo (apertura con horario especial). (Regla: Dialog para 1-4 campos y <10 seg. Con 5 campos en el caso máximo, está en el límite. Se mantiene Dialog porque el caso habitual es 3 campos y la interacción sigue siendo rápida.)
- **RadioGroup para tipo de festivo** en lugar de Select: solo hay 2 opciones (Cierre/Apertura), ambas deben ser visibles simultáneamente. La descripción debajo de cada opción ayuda a entender la diferencia. (Regla: RadioGroup para 2-4 opciones que deben verse todas.)
- **Revelación progresiva** del bloque de horario: los campos de horario solo aparecen al seleccionar "Apertura", reduciendo la carga cognitiva en el caso simple. (Decisión no cubierta explícitamente por el design system como "conditional field visibility". Se resuelve con show/hide animado del bloque, por analogía con formularios condicionales estándar.)
- **AlertDialog para eliminación:** acción destructiva e irreversible. Foco en "Cancelar". (Regla: AlertDialog siempre antes de acción irreversible.)
- **Toast para feedback:** tras crear, editar o eliminar. Variant default, 3-5 seg, bottom-right. (Regla: Toast después de acción mutadora exitosa.)
- **DropdownMenu por fila** con acciones: 2 acciones (Editar, Eliminar), la destructiva no debe ser inline. (Regla: DropdownMenu para 2+ acciones.)

### Comportamiento responsive

- **Mobile (< md):** La lista de festivos cambia a layout vertical: cada festivo es un bloque con fecha y nombre arriba, Badge de tipo y horario debajo, DropdownMenu a la derecha. El Dialog ocupa ancho completo.
- **Tablet (md-lg):** Interpolado. Lista en formato compacto con columnas más estrechas.
- **Desktop (lg+):** Layout completo como descrito en el wireframe.

## Notas técnicas

- Modelo de datos: cada festivo es un registro con `establishment_id`, `year`, `date`, `name`, `type` (enum: 'closure', 'opening'), `inherit_period_schedule` (boolean, solo aplica si type = 'opening').
- Si `type = 'opening'` e `inherit_period_schedule = true`, el festivo no almacena franjas propias. El horario se resuelve en tiempo real consultando las franjas del período cuyo rango contiene la fecha del festivo.
- Si `type = 'opening'` e `inherit_period_schedule = false`, el festivo tiene sus propias franjas en una tabla `holiday_slots` con estructura idéntica a `period_slots` (day_of_week se sustituye por `holiday_id`).
- La restricción de unicidad es por `(establishment_id, date)`: no puede haber dos festivos en la misma fecha para el mismo establecimiento.
- Dependencia upstream: SPEC-001 (períodos, para resolver herencia de horario), SPEC-002 y SPEC-003 (patrón de franjas horarias), SPEC-006 (validación de disponibilidad 24/7).
- Dependencia downstream: RF-FES-002 (herencia de horario, parcialmente cubierta aquí con la opción "Horario habitual del período"), RF-FES-004 (horario diferenciado, cubierta aquí con la opción "Horario especial"), RF-FES-003 (carga centralizada por mercado, extiende esta funcionalidad para bulk).
- La sección de festivos comparte página con los períodos (Layout 1). No es una página separada. Esto mantiene toda la configuración de un año en una sola vista.
