# SPEC-001 — Período único con excepciones

> RF: RF-HOR-002
> Criticidad: Must
> Wave: 1 — Modelo de datos core

## Descripción

Permite a la responsable de tienda configurar el horario comercial anual de su establecimiento mediante uno o varios períodos. Un período es un tramo de fechas durante el cual el horario semanal se mantiene constante. El sistema incentiva el uso de un único período anual (1 de enero a 31 de diciembre), de modo que cualquier variación puntual se gestione como excepción o festivo en lugar de obligar a fragmentar el año en múltiples períodos.

## Criterios de aceptación

### Visualización de períodos

- GIVEN el usuario accede a la página de horario comercial de su establecimiento para un año concreto WHEN la página carga THEN se muestra la lista de períodos definidos para ese año, ordenados por fecha de inicio ascendente.
- GIVEN existen períodos definidos WHEN la página carga THEN cada período muestra: nombre, fecha de inicio, fecha de fin, horario resumido (ej: "L-S 10:00-22:00, D 12:00-21:00"), y un indicador visual si es período de cierre.
- GIVEN existen períodos definidos WHEN la página carga THEN se muestra una barra de cobertura anual que indica visualmente qué tramos del año tienen período asignado y cuáles son huecos.
- GIVEN la página está cargando datos WHEN el fetch está en curso THEN se muestran Skeleton placeholders en el área de la lista de períodos.
- GIVEN el fetch de datos falla WHEN la página intenta cargar THEN se muestra un error state con mensaje descriptivo y botón "Reintentar".

### Empty state

- GIVEN el establecimiento no tiene períodos definidos para el año seleccionado WHEN la página carga THEN se muestra un empty state con icono, mensaje "No hay horario comercial configurado para [año]" y botón "Crear horario anual".
- GIVEN el usuario hace clic en "Crear horario anual" desde el empty state WHEN se abre el Dialog de creación THEN los campos fecha inicio y fecha fin vienen pre-rellenados con 1 de enero y 31 de diciembre del año seleccionado.

### Creación de período

- GIVEN el usuario hace clic en "Nuevo período" WHEN se abre el Dialog THEN se muestran los campos: Nombre del período (texto, requerido), Fecha de inicio (date picker, requerido), Fecha de fin (date picker, requerido), y checkbox "Período de cierre".
- GIVEN el Dialog de creación está abierto WHEN el usuario deja el campo Nombre vacío y sale del campo THEN se muestra error inline "El nombre del período es obligatorio".
- GIVEN el Dialog de creación está abierto WHEN el usuario selecciona una fecha de fin anterior a la fecha de inicio THEN se muestra error inline "La fecha de fin debe ser igual o posterior a la fecha de inicio".
- GIVEN el Dialog de creación está abierto WHEN el usuario define un rango de fechas que solapa con un período existente THEN se muestra error inline "Este período solapa con [nombre del período existente] ([fechas])".
- GIVEN el usuario rellena todos los campos correctamente WHEN hace clic en "Crear" THEN el período se guarda, el Dialog se cierra, la lista se actualiza y aparece un Toast "Período creado".
- GIVEN el usuario activa la casilla "Período de cierre" WHEN confirma la creación THEN el período se marca visualmente como cierre (rayado) y no requerirá franjas horarias.
- GIVEN el usuario hace clic en "Cancelar" o pulsa Escape WHEN el Dialog está abierto THEN el Dialog se cierra sin guardar cambios.

### Edición de período

- GIVEN existe un período WHEN el usuario hace clic en la acción "Editar" del período THEN se abre el Dialog con los datos actuales pre-cargados.
- GIVEN el Dialog de edición está abierto WHEN el usuario modifica los campos y hace clic en "Guardar" THEN los cambios se aplican, el Dialog se cierra, la lista se actualiza y aparece un Toast "Período actualizado".
- GIVEN el Dialog de edición está abierto WHEN el usuario modifica el rango de fechas y este solapa con otro período THEN se muestra error inline de solapamiento (mismo comportamiento que en creación).

### Eliminación de período

- GIVEN existe un período WHEN el usuario hace clic en la acción "Eliminar" del período THEN se muestra un AlertDialog con título "Eliminar período", descripción "Se eliminará el período [nombre] ([fecha inicio] - [fecha fin]) y todas sus franjas horarias y excepciones asociadas. Esta acción no se puede deshacer." y botones "Cancelar" / "Eliminar".
- GIVEN el AlertDialog de eliminación está abierto WHEN el usuario hace clic en "Eliminar" THEN el período se elimina, el AlertDialog se cierra, la lista se actualiza y aparece un Toast "Período eliminado".
- GIVEN el AlertDialog de eliminación está abierto WHEN el usuario hace clic en "Cancelar" o pulsa Escape THEN el AlertDialog se cierra sin eliminar.

### Validación de cobertura anual

- GIVEN los períodos definidos no cubren los 365 días del año WHEN la página carga o un período se crea/edita/elimina THEN se muestra un banner de advertencia (Alert, variant default) indicando "El horario no cubre todo el año. Faltan [N] días sin período asignado." con los rangos de fechas sin cobertura.
- GIVEN los períodos cubren todo el año sin huecos WHEN la página carga THEN no se muestra ningún banner de advertencia.
- GIVEN hay huecos en la cobertura WHEN el usuario intenta navegar fuera de la página THEN puede hacerlo sin bloqueo (la advertencia es informativa, no bloqueante).

### Selección de año

- GIVEN el usuario está en la página de horario comercial WHEN hace clic en el selector de año THEN puede navegar entre años disponibles (año actual y siguiente como mínimo).
- GIVEN el usuario selecciona un año diferente WHEN el selector cambia THEN la lista de períodos se actualiza para mostrar los del año seleccionado.

## UX Design

### Wireframe textual

**Página de horario comercial del establecimiento — Layout 1 (Estándar)**

Zona superior:
- Título de página: "Horario comercial" con el nombre del establecimiento a la derecha como texto secundario (muted).
- Selector de año: Select con los años disponibles, alineado a la derecha del título.
- Botón "Nuevo período" (Button, variant default, icono Plus), a la derecha.

Zona de cobertura (debajo del título):
- Barra de cobertura anual: representación visual horizontal de los 12 meses. Los tramos con período asignado se muestran con fondo sólido (color primary/muted). Los huecos se muestran vacíos o con patrón rayado. Los períodos de cierre se muestran con rayado diagonal.
- Si hay huecos: Alert (variant default) debajo de la barra, indicando los rangos sin cobertura.

Zona principal (lista de períodos):
- Cada período se muestra como una Card con:
  - Nombre del período (título, font-medium).
  - Rango de fechas (texto secundario, muted).
  - Resumen de horario semanal (ej: "L-S 10:00-22:00, D Cerrado") o "Sin franjas definidas" si aún no se han configurado (texto muted, cursiva).
  - Badge "Cierre" (variant destructive) si es período de cierre.
  - Acciones: DropdownMenu (icono MoreHorizontal) con opciones "Editar" y "Eliminar" (esta última con estilo destructive y separador).
- Las cards se apilan verticalmente, ordenadas por fecha de inicio.

Empty state (si no hay períodos):
- Centrado en el área principal. Icono Calendar (Lucide, 24px). Mensaje "No hay horario comercial configurado para [año]". Botón "Crear horario anual" (Button, variant default).

**Dialog de creación/edición de período**

- Título: "Nuevo período" o "Editar período".
- Campos (stack vertical):
  - Nombre del período (Input, placeholder "Ej: Horario anual").
  - Fecha de inicio (Input type date).
  - Fecha de fin (Input type date).
  - Período de cierre (checkbox con label "Este período es de cierre (el establecimiento permanece cerrado)").
- Botones en footer: "Cancelar" (Button variant outline) a la izquierda, "Crear" / "Guardar" (Button variant default) a la derecha.

### Componentes shadcn utilizados

Componentes: Card, Button, Select, Dialog, Input, Badge, Toast, AlertDialog, DropdownMenu, Skeleton, Form

Componente adicional necesario: Alert (no incluido en la lista base del scaffold).
Componente adicional necesario: Checkbox (no incluido en la lista base del scaffold).

### Patrón de interacción

- **Cards en lugar de Table** para los períodos: un establecimiento tiene típicamente 1-5 períodos por año. No hay necesidad de sorting, filtering ni selección múltiple. Las Cards permiten mostrar el resumen de horario de forma legible. (Regla: Table cuando hay 3+ campos comparables y sorting; Cards cuando hay pocos items con información compuesta.)
- **Dialog para creación/edición:** 4 campos, interacción de menos de 10 segundos, el usuario no necesita ver la página detrás durante la edición. (Regla: Dialog para 1-4 campos y <10 seg de interacción.)
- **AlertDialog para eliminación:** acción destructiva e irreversible (elimina período + franjas + excepciones asociadas). Botón "Eliminar" con variant destructive. Foco inicial en "Cancelar". (Regla: AlertDialog siempre antes de acción irreversible.)
- **Toast para feedback:** tras crear, editar o eliminar un período. Variant default, 3-5 seg, auto-dismiss, bottom-right. (Regla: Toast siempre después de acción mutadora exitosa.)
- **Validación inline on blur** para campos del Dialog + validación on submit para reglas de negocio (solapamiento). (Regla: ambas, inline on blur + on submit.)
- **Alert informativo (no bloqueante)** para huecos de cobertura anual. No impide guardar ni navegar. (Decisión no cubierta explícitamente por el design system: se usa Alert variant default como banner informativo persistente en la página, no como error. Se resuelve con Alert sin acción de cierre, visible mientras exista el hueco, por analogía con los empty states informativos del design system.)
- **DropdownMenu por card** en lugar de botones inline: hay 2 acciones por período (Editar, Eliminar), y la acción destructiva no debe ser inline. (Regla: DropdownMenu para 2+ acciones, acciones destructivas nunca como inline button.)

### Comportamiento responsive

- **Mobile (< md):** Sidebar colapsa a drawer. Las Cards de período ocupan el ancho completo. El selector de año y el botón "Nuevo período" se apilan debajo del título. La barra de cobertura anual se simplifica a una versión compacta (solo indicador de % de cobertura + texto de huecos). El Dialog ocupa el ancho completo del viewport.
- **Tablet (md-lg):** Interpolado entre mobile y desktop. Cards a ancho completo. Sidebar colapsada a iconos.
- **Desktop (lg+):** Layout completo como descrito en el wireframe. Sidebar expandida. Cards con ancho máximo contenido por el padding estándar (p-6).

## Notas técnicas

- El modelo de datos debe soportar N períodos por año/establecimiento, pero la UX incentiva que N=1 sea el caso habitual. No hay restricción técnica que fuerce un único período.
- La validación de solapamiento se ejecuta client-side contra los períodos ya cargados (son pocos, siempre <10).
- La barra de cobertura anual se calcula client-side a partir de las fechas de los períodos. No requiere endpoint específico.
- Dependencia downstream: una vez creado un período, RF-HOR-004 (franjas horarias por día) permite definir el horario dentro del período. Un período sin franjas es un estado válido pero incompleto (se señala visualmente).
- Dependencia downstream: RF-EXC-001/002/003 gestionan las variaciones puntuales dentro de un período, evitando la necesidad de fragmentar en múltiples períodos.
- El campo "Período de cierre" elimina la necesidad de definir franjas (el establecimiento está cerrado). Es un flag booleano en el modelo.
