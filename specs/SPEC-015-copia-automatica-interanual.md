# SPEC-015 — Copia automática interanual

> RF: RF-HOR-001
> Criticidad: Must
> Wave: 3 — Visualización, copia interanual y validación

## Descripción

Permite copiar la configuración completa del año anterior como punto de partida para el nuevo año, incluyendo períodos, horarios semanales, festivos recurrentes y cambios puntuales recurrentes. En el sistema legacy, la copia solo trasladaba los períodos y sus franjas, obligando a recrear manualmente todos los festivos y excepciones. Esto multiplicaba el trabajo innecesariamente, ya que la mayoría de establecimientos mantienen horarios estables año tras año y solo necesitan ajustar los festivos de fecha variable (Semana Santa, puentes). Esta funcionalidad es el mayor ahorro de tiempo del nuevo sistema, mencionada en 7 de 15 fuentes del research.

## Criterios de aceptación

### Activación de la copia

- GIVEN el establecimiento no tiene configuración para el año seleccionado WHEN la página carga en el empty state THEN se muestra, además del botón "Crear horario anual" (SPEC-001), un botón destacado "Copiar desde [año anterior]" (Button variant default, icono Copy).
- GIVEN el establecimiento no tiene configuración para el año seleccionado WHEN existe configuración completa en el año anterior THEN el botón "Copiar desde [año anterior]" aparece como acción principal (más prominente que "Crear horario anual").
- GIVEN el establecimiento no tiene configuración para el año seleccionado WHEN no existe configuración en el año anterior THEN el botón "Copiar desde [año anterior]" no se muestra. Solo aparece "Crear horario anual".
- GIVEN el establecimiento ya tiene configuración parcial para el año seleccionado (al menos un período) WHEN la página carga THEN el botón de copia no se muestra (la copia solo está disponible para años vacíos).

### Dialog de previsualización y confirmación

- GIVEN el usuario hace clic en "Copiar desde [año anterior]" WHEN se abre el Dialog THEN muestra un resumen de lo que se va a copiar: número de períodos, número de festivos (desglosados en recurrentes y de fecha fija), número de cambios puntuales, y si hay cierres temporales (estos no se copian).
- GIVEN el Dialog está abierto WHEN el usuario ve el resumen THEN cada categoría muestra un checkbox activado por defecto, permitiendo al usuario desmarcar lo que no quiere copiar. Categorías: "Períodos y horarios semanales" (siempre activado, no desmarcable), "Festivos" (activado por defecto), "Cambios puntuales" (activado por defecto).
- GIVEN el Dialog está abierto WHEN existen cierres temporales en el año anterior THEN se muestra una nota informativa: "Los cierres temporales no se copian (son eventos extraordinarios no recurrentes)".
- GIVEN el Dialog está abierto WHEN el usuario ve la previsualización THEN se muestra un texto informativo: "Las fechas se ajustarán automáticamente al [año destino]. Los festivos de fecha variable (ej: Semana Santa) deberán revisarse manualmente tras la copia".

### Ajuste de fechas

- GIVEN se ejecuta la copia de períodos WHEN los períodos se trasladan al nuevo año THEN las fechas se ajustan sumando la diferencia de años. Un período "01/01/2025 - 31/12/2025" se convierte en "01/01/2026 - 31/12/2026".
- GIVEN se ejecuta la copia de períodos WHEN el año origen era bisiesto y el destino no (o viceversa) THEN el 29 de febrero se ajusta: si el año destino no es bisiesto, un período que terminaba el 29/02 termina el 28/02. Si el año destino es bisiesto y el origen no, no se añade día automáticamente.
- GIVEN se ejecuta la copia de festivos WHEN los festivos se trasladan al nuevo año THEN las fechas fijas se trasladan al mismo día y mes del nuevo año (ej: 25/12/2025 → 25/12/2026). El día de la semana puede cambiar.
- GIVEN se ejecuta la copia de cambios puntuales WHEN los cambios se trasladan THEN las fechas se ajustan al mismo día y mes del nuevo año.

### Ejecución de la copia

- GIVEN el usuario confirma la copia haciendo clic en "Copiar" WHEN la operación se inicia THEN se muestra un indicador de progreso en el Dialog (spinner + texto "Copiando configuración..."). El botón "Copiar" se desactiva durante la operación.
- GIVEN la copia se ejecuta correctamente WHEN finaliza THEN el Dialog se cierra, la página se recarga con la nueva configuración, y aparece un Toast: "Configuración copiada desde [año anterior]. Revisa los festivos de fecha variable".
- GIVEN la copia falla por error del servidor WHEN se recibe el error THEN se muestra Toast de error (variant destructive): "Error al copiar la configuración. Inténtalo de nuevo". El Dialog permanece abierto.

### Estado post-copia

- GIVEN la copia se ha completado WHEN la página muestra la nueva configuración THEN todos los períodos copiados mantienen sus nombres, horarios semanales y estructura de franjas idénticos al año anterior.
- GIVEN la copia incluyó festivos WHEN la página muestra la nueva configuración THEN los festivos copiados aparecen con sus tipos (cierre/apertura), horarios (heredado/especial) y nombres originales, con las fechas ajustadas al nuevo año.
- GIVEN la copia incluyó festivos con horario heredado del período WHEN un festivo copiado cae en un día de la semana diferente al del año anterior THEN la herencia se recalcula: el festivo hereda el horario del nuevo día de la semana en el nuevo período. (Ej: si el 25/12 era jueves en 2025 y es viernes en 2026, hereda el horario del viernes.)
- GIVEN la copia incluyó festivos con horario especial WHEN se copian al nuevo año THEN las franjas especiales se mantienen exactamente iguales (no dependen del día de la semana).
- GIVEN la copia incluyó cambios puntuales WHEN la página muestra la nueva configuración THEN cada cambio copiado aparece con un Badge "Copiado" (variant outline, small) visible durante 30 días tras la copia, para que el usuario identifique qué cambios fueron copiados y cuáles creó manualmente.

### Advertencia de festivos de fecha variable

- GIVEN la copia se ha completado WHEN la página muestra la nueva configuración THEN se muestra un Alert persistente (variant default, icono AlertTriangle) en la parte superior de la sección de festivos: "Configuración copiada desde [año anterior]. Revisa las fechas de los festivos variables (Semana Santa, puentes, etc.)". El Alert incluye un botón "Entendido" que lo descarta permanentemente.

### Validación pre-copia

- GIVEN el usuario intenta copiar desde un año anterior WHEN el año anterior tiene períodos con huecos de cobertura THEN se muestra advertencia en el Dialog: "El año [anterior] tiene [N] días sin cobertura. Se copiarán los mismos huecos". No es bloqueante.
- GIVEN el usuario intenta copiar desde un año anterior WHEN el año anterior tiene festivos en estado de conflicto (SPEC-008) THEN se muestra advertencia: "[N] festivo(s) tienen conflictos de horario. Se copiarán tal cual, revísalos tras la copia". No es bloqueante.

### Edge cases

- GIVEN el usuario copia la configuración WHEN después elimina toda la configuración copiada THEN el empty state vuelve a mostrar el botón "Copiar desde [año anterior]".
- GIVEN el usuario copia la configuración y luego navega a un año aún más nuevo (ej: de 2026 a 2027) WHEN el año 2027 está vacío THEN puede copiar desde 2026 (que contiene la configuración copiada de 2025 más las modificaciones hechas en 2026).
- GIVEN el año anterior tiene un período que cruza el límite de año (ej: 01/11/2025 - 28/02/2026) WHEN se copia THEN solo se copia la parte que cae en el año anterior. El período copiado al nuevo año mantiene sus fechas ajustadas.
- GIVEN la copia incluye un festivo cuya fecha no existe en el año destino (29/02 en año no bisiesto) WHEN se ejecuta la copia THEN ese festivo se omite y se informa al usuario: "1 festivo no se pudo copiar (29 de febrero no existe en [año destino])".

## UX Design

### Wireframe textual

**Empty state modificado (SPEC-001)**

El empty state existente se amplía con el botón de copia:

- Icono CalendarPlus (64px, muted).
- Mensaje: "No hay horario comercial configurado para [año]".
- Fila de botones (flex, gap):
  - "Copiar desde [año anterior]" (Button variant default, icono Copy). Acción principal si hay datos en el año anterior.
  - "Crear horario anual" (Button variant outline). Acción secundaria.

**Dialog de previsualización de copia**

- Título: "Copiar configuración desde [año anterior]".
- Contenido:
  - Resumen en formato de lista compacta:
    - "☑ [N] período(s) con horarios semanales" (checkbox desactivado, siempre marcado).
    - "☑ [N] festivo(s)" (checkbox activo, marcado por defecto). Desglose en texto muted: "[X] de cierre, [Y] de apertura".
    - "☑ [N] cambio(s) puntual(es)" (checkbox activo, marcado por defecto).
    - "ℹ [N] cierre(s) temporal(es) — no se copian" (texto muted, sin checkbox).
  - Advertencias si aplican (huecos de cobertura, conflictos).
  - Texto informativo sobre fechas variables (texto small, muted).
- Footer: "Cancelar" (Button variant outline) / "Copiar" (Button variant default, icono Copy).

### Componentes shadcn utilizados

Componentes: Dialog, Button, Checkbox (nuevo en esta spec, para selección de categorías a copiar), Alert, Toast, Skeleton. El resto ya presentes.

### Patrón de interacción

- **Botón de copia como acción principal en el empty state:** la mayoría de usuarios copian el año anterior. Es la acción por defecto, no la excepción. "Crear horario anual" pasa a ser la alternativa. (Decisión de UX: optimizar para el caso más frecuente.)
- **Dialog de previsualización antes de ejecutar:** el usuario ve exactamente qué se va a copiar y puede excluir categorías. Esto evita sorpresas y da control sin complicar el flujo. (Regla: confirmar antes de acciones que generan muchos registros.)
- **Checkboxes para categorías en lugar de todo-o-nada:** un establecimiento puede querer copiar los períodos pero no los festivos (porque los festivos cambian). Flexibilidad sin complejidad excesiva. (Decisión de UX: granularidad útil sin llegar a nivel de registro individual.)
- **Badge "Copiado" temporal (30 días):** el usuario necesita distinguir qué vino de la copia y qué añadió manualmente, especialmente durante las primeras semanas de configuración del nuevo año. Desaparece después para no generar ruido permanente. (Decisión de UX: indicador transitorio para contexto temporal.)
- **Alert persistente sobre festivos variables:** el mayor riesgo de la copia es que el usuario olvide ajustar Semana Santa u otros festivos de fecha variable. El Alert es un recordatorio visible que el usuario descarta explícitamente cuando ya ha revisado. (Regla: Alert persistente para acciones pendientes del usuario.)

### Comportamiento responsive

- **Mobile (< md):** Empty state con botones apilados verticalmente. Dialog de previsualización a ancho completo.
- **Tablet (md-lg):** Interpolado.
- **Desktop (lg+):** Como descrito.

## Notas técnicas

- La copia se ejecuta como una operación atómica server-side (transacción). Si falla cualquier parte, no se crea nada.
- El endpoint recibe: `{ source_year, target_year, establishment_id, include_holidays: boolean, include_overrides: boolean }`. Los períodos siempre se copian.
- Los IDs de los registros copiados son nuevos (no se duplican los IDs del año anterior). Las referencias internas (ej: `holiday.period_id`) se reasignan a los nuevos períodos creados.
- Los festivos con `inherit_period_schedule = true` mantienen la flag. La herencia se recalcula automáticamente contra el nuevo período (que tiene las mismas franjas que el original).
- Los festivos con horario especial (`inherit_period_schedule = false`) copian sus registros de `holiday_slots` tal cual.
- Los cambios puntuales copian sus registros de `override_slots` si son de tipo 'modified'.
- Los cierres temporales NO se copian por diseño: son eventos extraordinarios (obras, emergencias) que no se repiten.
- El Badge "Copiado" se implementa con un campo `copied_from_year` (nullable) en las tablas `holidays` y `schedule_overrides`. Si tiene valor y la fecha de copia es < 30 días, se muestra el Badge. El campo también sirve para trazabilidad.
- Componente nuevo: Checkbox de shadcn. Se añade al inventario de componentes del proyecto.
- Dependencia upstream: SPEC-001 (períodos, empty state), SPEC-002 (franjas), SPEC-007 (festivos), SPEC-008 (herencia), SPEC-010 (cambios puntuales).
- Dependencia downstream: SPEC-013 (el calendario anual mostrará la configuración copiada), RF-HOR-006 (validación de cobertura anual, que alertará si la copia dejó huecos).
