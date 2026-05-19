# SPEC-003 — Horario partido

> RF: RF-HOR-005
> Criticidad: Must
> Wave: 1 — Modelo de datos core

## Descripción

Extiende el editor de horario semanal (SPEC-002) para soportar establecimientos con horario partido (cierre a mediodía). Un día puede tener hasta 2 franjas horarias independientes (ej: 10:00-14:00 y 17:00-21:00). Esta casuística es habitual en mercados con tradición de cierre a mediodía y en tiendas de calle fuera de shopping centres.

## Criterios de aceptación

### Añadir segunda franja

- GIVEN un día tiene una franja horaria definida WHEN el usuario hace clic en "Añadir franja" en esa fila THEN aparece una segunda fila subordinada al mismo día con campos de hora inicio y hora fin en modo edición.
- GIVEN un día tiene una franja horaria definida WHEN el usuario hace clic en "Añadir franja" THEN el botón "Añadir franja" desaparece de esa fila (máximo 2 franjas por día).
- GIVEN un día no tiene ninguna franja definida WHEN el usuario visualiza la fila THEN no se muestra el botón "Añadir franja" (primero debe definirse la franja principal).

### Visualización de horario partido

- GIVEN un día tiene 2 franjas definidas WHEN la Card del período está expandida THEN ambas franjas se muestran apiladas verticalmente dentro de la misma fila del día, separadas visualmente (ej: "10:00 — 14:00" y "17:00 — 21:00").
- GIVEN un día tiene 2 franjas definidas WHEN la Card del período está colapsada THEN el resumen compacto muestra ambas franjas (ej: "L-V 10:00-14:00 / 17:00-21:00, S 10:00-14:00, D Cerrado").

### Validación

- GIVEN un día tiene una franja de 10:00 a 14:00 WHEN el usuario introduce una segunda franja de 13:00 a 21:00 THEN se muestra error inline "Las franjas no pueden solaparse. La segunda franja debe empezar después de las 14:00".
- GIVEN un día tiene una franja de 10:00 a 14:00 WHEN el usuario introduce una segunda franja de 14:00 a 21:00 THEN se acepta como válido (las franjas son contiguas, sin solapamiento).
- GIVEN un día tiene una franja de 10:00 a 14:00 WHEN el usuario introduce una segunda franja de 9:00 a 10:00 THEN se muestra error inline "Las franjas no pueden solaparse. Revisa las horas de inicio y fin".
- GIVEN la segunda franja está en modo edición WHEN el usuario introduce una hora fin igual o anterior a su hora inicio THEN se muestra error inline "La hora de cierre debe ser posterior a la hora de apertura" (misma validación que SPEC-002).

### Eliminación de franja

- GIVEN un día tiene 2 franjas WHEN el usuario elimina la segunda franja (icono X) THEN la segunda franja se elimina, el día vuelve a mostrar solo la primera franja y reaparece el botón "Añadir franja".
- GIVEN un día tiene 2 franjas WHEN el usuario elimina la primera franja THEN la segunda franja pasa a ser la primera (se reordena automáticamente por hora de inicio), y reaparece el botón "Añadir franja".
- GIVEN un día tiene 1 sola franja WHEN el usuario la elimina THEN el día pasa a "Cerrado" (comportamiento heredado de SPEC-002).

### Copiar horario partido a otros días

- GIVEN un día tiene 2 franjas definidas WHEN el usuario hace clic en "Copiar a otros días" THEN se copian ambas franjas a los días seleccionados (no solo la primera).
- GIVEN el usuario copia un horario partido a un día que tenía 1 franja WHEN confirma la copia THEN el día destino reemplaza su configuración con las 2 franjas del día origen.

### Edición de franjas

- GIVEN un día tiene 2 franjas WHEN el usuario hace clic en una de las franjas THEN solo esa franja entra en modo edición (la otra permanece en modo lectura).
- GIVEN el usuario edita la primera franja cambiando su hora fin WHEN el nuevo valor hace que solape con la segunda franja THEN se muestra error inline de solapamiento.

### Ordenación automática

- GIVEN un día tiene 2 franjas WHEN el usuario guarda THEN las franjas se ordenan automáticamente por hora de inicio ascendente (la franja de mañana siempre arriba, la de tarde abajo), independientemente del orden en que se introdujeron.

## UX Design

### Wireframe textual

**Extensión de la Card de período expandida (SPEC-002)**

El grid de 7 días se modifica para soportar sub-filas:

Fila de día con 1 franja (sin cambios respecto a SPEC-002):
- Columna 1: Nombre del día.
- Columna 2-4: Horario (modo lectura o edición).
- Columna 5: Acciones: Copiar (icono Copy), Eliminar franja (icono X), **Añadir franja** (icono Plus, variant ghost, tamaño small).

Fila de día con 2 franjas:
- Sub-fila 1 (franja mañana): sin nombre de día (espacio vacío en columna 1 o indent visual), horario, acciones (Copiar en esta sub-fila solo, Eliminar esta franja).
- Sub-fila 2 (franja tarde): mismo layout que sub-fila 1.
- El nombre del día aparece una sola vez, alineado verticalmente con la primera sub-fila.
- El botón "Copiar a otros días" se muestra solo en la primera sub-fila y copia ambas franjas.
- Separador visual sutil (borde inferior lighter) entre las dos sub-filas para distinguirlas del siguiente día.

Fila de día con 2 franjas — estado "Cerrado" imposible:
- Si un día tiene 2 franjas, no puede mostrarse como "Cerrado". Solo se llega a "Cerrado" eliminando todas las franjas.

### Componentes shadcn utilizados

Componentes: los mismos de SPEC-002 (Card, Button, Input, Badge, Toast, AlertDialog, Tooltip, Skeleton, Popover, Checkbox). No se requieren componentes adicionales.

### Patrón de interacción

- **Sub-filas dentro de la fila de día** en lugar de un bloque separado o modal: mantiene la coherencia visual con SPEC-002 y permite ver el contexto de ambas franjas juntas. (Decisión no cubierta explícitamente por el design system como patrón de sub-filas en grid inline. Se resuelve por extensión natural del patrón de edición inline de SPEC-002, dado que el volumen es fijo — máximo 2 sub-filas por día.)
- **Botón "Añadir franja" inline** (icono Plus, variant ghost, small): aparece solo cuando el día tiene exactamente 1 franja. Desaparece al alcanzar el máximo de 2. No necesita Tooltip porque la acción es autoexplicativa por el icono y el contexto.
- **Validación de solapamiento client-side** al salir del campo (on blur): se compara con la otra franja del mismo día. No requiere llamada al servidor.
- **Ordenación automática al guardar:** las franjas se reordenan por hora inicio. El usuario no necesita preocuparse del orden de entrada.

### Comportamiento responsive

- **Mobile (< md):** Cada franja de un día partido se muestra como un bloque independiente dentro del bloque del día, apilados verticalmente. El botón "Añadir franja" se muestra debajo de la primera franja.
- **Tablet (md-lg):** Interpolado. Sub-filas visibles en formato compacto.
- **Desktop (lg+):** Sub-filas completas como descrito en el wireframe.

## Notas técnicas

- Modelo de datos: sin cambios en el esquema de SPEC-002. Un día con horario partido simplemente tiene 2 registros de franja para el mismo `day_of_week` dentro del mismo `period_id`. Se añade un campo `slot_order` (1 o 2) o se ordena por `start_time` ascendente.
- **Prerequisito de migración:** la tabla `period_slots` tiene una constraint UNIQUE heredada `(period_id, day_of_week)` que impide tener más de una franja por día. Esta constraint debe eliminarse antes de implementar esta spec. Sustituir por una constraint UNIQUE `(period_id, day_of_week, slot_order)` o validar el máximo de 2 franjas por día exclusivamente a nivel aplicación.
- Restricción de negocio: máximo 2 franjas por día. Esta restricción se valida tanto client-side (ocultando el botón "Añadir franja") como server-side (rechazando un tercer registro).
- La validación de solapamiento se ejecuta client-side comparando los rangos de las 2 franjas del mismo día. No hay solapamiento si `franja2.start_time >= franja1.end_time`.
- Dependencia upstream: SPEC-002 (editor de horario semanal). Esta spec extiende su comportamiento.
- El resumen compacto de la Card colapsada (SPEC-002) debe actualizarse para mostrar horarios partidos: el algoritmo de agrupación debe considerar que dos días son "iguales" solo si tienen el mismo número de franjas con los mismos horarios.
