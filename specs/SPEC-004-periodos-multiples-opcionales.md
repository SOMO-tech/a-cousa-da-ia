# SPEC-004 — Períodos múltiples opcionales

> RF: RF-HOR-003
> Criticidad: Must
> Wave: 1 — Modelo de datos core

## Descripción

Facilita la gestión de horarios estacionales permitiendo dividir un período existente en dos (ej: verano/invierno) y fusionar períodos adyacentes con el mismo horario. SPEC-001 ya soporta la creación manual de múltiples períodos. Esta spec añade acciones de conveniencia que simplifican los dos flujos más habituales: partir un año en dos temporadas y consolidar períodos redundantes.

## Criterios de aceptación

### Dividir período

- GIVEN existe un período que abarca más de 1 día WHEN el usuario hace clic en "Dividir período" en el DropdownMenu de la Card THEN se abre un Dialog con el título "Dividir período", mostrando el rango actual del período y un campo de fecha para indicar el punto de corte.
- GIVEN el Dialog de división está abierto WHEN el usuario selecciona una fecha de corte válida (dentro del rango del período, excluyendo el primer y último día) THEN el sistema muestra una previsualización: "Se crearán dos períodos: [nombre] (1) del [inicio] al [fecha corte] y [nombre] (2) del [fecha corte + 1 día] al [fin]".
- GIVEN el usuario confirma la división WHEN hace clic en "Dividir" THEN el período original se reemplaza por dos períodos nuevos, ambos heredan las franjas horarias del período original, la lista se actualiza y aparece un Toast "Período dividido en dos".
- GIVEN el Dialog de división está abierto WHEN el usuario selecciona una fecha fuera del rango del período THEN se muestra error inline "La fecha debe estar entre [inicio + 1] y [fin - 1]".
- GIVEN el Dialog de división está abierto WHEN el usuario selecciona el primer o el último día del período THEN se muestra error inline "No se puede dividir en el primer o último día del período. Selecciona una fecha intermedia".
- GIVEN el período tiene excepciones o festivos definidos WHEN el usuario confirma la división THEN las excepciones y festivos se asignan automáticamente al período cuyo rango de fechas las contiene.
- GIVEN el período es de tipo cierre WHEN el usuario hace clic en "Dividir período" THEN la acción está disponible (un cierre también puede dividirse en dos tramos).

### Fusionar períodos

- GIVEN existen dos períodos adyacentes (el fin del primero + 1 día = inicio del segundo) con las mismas franjas horarias y el mismo tipo (ambos normales o ambos de cierre) WHEN la página carga THEN se muestra un indicador visual sutil en la Card del primer período: "Este período puede fusionarse con [nombre del siguiente]" (texto muted, small).
- GIVEN el indicador de fusión es visible WHEN el usuario hace clic en "Fusionar con siguiente" en el DropdownMenu de la Card THEN se abre un AlertDialog: "Fusionar períodos", descripción "Se fusionarán [nombre 1] ([fechas]) y [nombre 2] ([fechas]) en un único período del [inicio 1] al [fin 2]. Las franjas horarias se mantienen.", botones "Cancelar" / "Fusionar".
- GIVEN el usuario confirma la fusión WHEN hace clic en "Fusionar" THEN los dos períodos se reemplazan por uno solo que abarca desde el inicio del primero hasta el fin del segundo, hereda las franjas horarias (que son idénticas), las excepciones y festivos de ambos períodos se conservan, la lista se actualiza y aparece un Toast "Períodos fusionados".
- GIVEN dos períodos adyacentes tienen franjas horarias diferentes WHEN la página carga THEN no se muestra el indicador de fusión ni la acción "Fusionar con siguiente".
- GIVEN dos períodos no son adyacentes (hay un hueco entre ellos) WHEN la página carga THEN no se muestra el indicador de fusión.

### Nombres de períodos tras división

- GIVEN el usuario divide un período llamado "Horario anual" WHEN confirma la división THEN los dos períodos resultantes se nombran "Horario anual (1)" y "Horario anual (2)".
- GIVEN el usuario divide un período WHEN los períodos se crean THEN el usuario puede renombrarlos inmediatamente mediante la acción "Editar" de cada Card (SPEC-001).

### Edge cases

- GIVEN un período tiene solo 2 días WHEN el usuario hace clic en "Dividir período" THEN la única fecha de corte posible es el primer día, resultando en un período de 1 día + otro de 1 día.
- GIVEN un período tiene solo 1 día WHEN el usuario abre el DropdownMenu THEN la acción "Dividir período" no aparece (no se puede dividir un período de 1 día).
- GIVEN el usuario divide un período y luego quiere deshacer WHEN los dos períodos resultantes tienen las mismas franjas THEN puede usar "Fusionar con siguiente" para volver al estado original.

## UX Design

### Wireframe textual

**DropdownMenu de la Card de período (extensión de SPEC-001)**

El DropdownMenu existente (icono MoreHorizontal) se extiende con nuevas opciones:
- Editar (heredado de SPEC-001)
- Dividir período (nuevo, icono Scissors)
- Fusionar con siguiente (nuevo, icono Merge, solo visible si hay período adyacente fusionable)
- Separador
- Eliminar (heredado de SPEC-001, variant destructive)

**Dialog de división de período**

- Título: "Dividir período".
- Texto informativo: "El período [nombre] ([fecha inicio] — [fecha fin]) se dividirá en dos. Ambos heredarán el horario actual." (texto muted).
- Campo: "Último día del primer período" (Input type date, requerido). El segundo período comenzará automáticamente el día siguiente.
- Previsualización (debajo del campo, texto muted): "Resultado: [nombre] (1) del [inicio] al [fecha corte] y [nombre] (2) del [fecha corte + 1] al [fin]". Se actualiza dinámicamente al cambiar la fecha.
- Botones: "Cancelar" (Button variant outline) / "Dividir" (Button variant default).

**Indicador de fusión en Card**

- Texto pequeño debajo de las fechas del período: "Fusionable con [nombre del siguiente]" (texto muted, font-size small, icono Merge 16px inline).
- No es un botón, es informativo. La acción se ejecuta desde el DropdownMenu.

### Componentes shadcn utilizados

Componentes: Dialog, Button, Input, AlertDialog, Toast, DropdownMenu (todos ya presentes en specs anteriores). No se requieren componentes adicionales.

### Patrón de interacción

- **Dialog para dividir:** 1 campo (fecha de corte) + previsualización. Interacción de menos de 10 segundos. (Regla: Dialog para 1-4 campos y <10 seg.)
- **AlertDialog para fusionar:** acción que elimina un período y modifica otro. Aunque no es destructiva (no se pierde información), es irreversible sin deshacer manual. Foco en "Cancelar". (Regla: AlertDialog antes de acción irreversible.)
- **Previsualización dinámica en Dialog de división:** el usuario ve el resultado antes de confirmar. Reduce errores y genera confianza. (Decisión no cubierta explícitamente por el design system. Se resuelve con texto dinámico debajo del campo, sin componente adicional.)
- **Indicador pasivo de fusión:** no es un botón para evitar acciones accidentales. La acción se ejecuta deliberadamente desde el menú. (Regla: acciones destructivas o de modificación significativa nunca como inline button.)

### Comportamiento responsive

- **Mobile (< md):** Dialog ocupa ancho completo. El indicador de fusión se muestra igual (texto small). El DropdownMenu funciona igual.
- **Tablet (md-lg):** Interpolado.
- **Desktop (lg+):** Layout como descrito en el wireframe.

## Notas técnicas

- La división es una operación atómica: elimina el período original y crea dos nuevos en una sola transacción. Si falla, ningún cambio se aplica.
- Al dividir, las excepciones y festivos existentes se reasignan al período cuyo rango contiene la fecha de la excepción/festivo. Si una excepción cae exactamente en la fecha de corte, se asigna al primer período (el que termina ese día).
- La detección de "períodos fusionables" se ejecuta client-side al cargar la página: se comparan los períodos adyacentes por igualdad de franjas (mismo número de franjas, mismos días, mismos horarios) y mismo tipo (normal/cierre).
- La fusión conserva el nombre del primer período. Las excepciones y festivos de ambos períodos se mantienen intactos.
- Dependencia upstream: SPEC-001 (gestión de períodos), SPEC-002 (franjas horarias).
