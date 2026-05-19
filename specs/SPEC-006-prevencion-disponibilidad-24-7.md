# SPEC-006 — Prevención de disponibilidad 24/7

> RF: RF-EXC-004
> Criticidad: Must
> Wave: 1 — Modelo de datos core

## Descripción

Impide que un día marcado como abierto (festivo de apertura, cambio puntual de horario o día normal) se guarde sin franjas horarias definidas. En el sistema legacy, esta omisión provocaba que el establecimiento apareciera como disponible 24 horas, generando datos incorrectos en Google Maps y otros sistemas downstream. Esta spec define la constraint de validación que previene ese error tanto a nivel de interfaz como de base de datos.

## Criterios de aceptación

### Validación al guardar horario semanal

- GIVEN el usuario está editando el horario semanal de un período (SPEC-002) WHEN un día no está marcado como "Cerrado" y no tiene ninguna franja horaria definida THEN se muestra error inline en la fila del día: "Define el horario de apertura o marca el día como cerrado".
- GIVEN el usuario está editando el horario semanal WHEN intenta guardar con un día sin franja y sin estar marcado como cerrado THEN el guardado se bloquea y el foco se desplaza al primer día con el error.
- GIVEN el usuario está editando el horario semanal WHEN todos los días tienen al menos una franja o están marcados como cerrados THEN el guardado se ejecuta con normalidad.

### Validación al crear festivo de apertura

- GIVEN el usuario crea un festivo de apertura (Wave 2, RF-FES-002) WHEN no define ninguna franja horaria para ese día THEN se muestra error inline: "Un festivo de apertura requiere horario. Define las horas en que el establecimiento estará abierto".
- GIVEN el usuario crea un festivo de apertura WHEN define al menos una franja horaria válida THEN el festivo se guarda correctamente.
- GIVEN el usuario crea un festivo de apertura WHEN intenta guardar sin horario THEN el botón "Guardar" permanece desactivado con Tooltip: "Falta definir el horario de apertura".

### Validación al crear cambio puntual de horario

- GIVEN el usuario crea un cambio puntual de horario (Wave 2, RF-EXC-001) WHEN no define ninguna franja horaria THEN se muestra error inline: "Un cambio de horario requiere las nuevas horas de apertura".
- GIVEN el usuario crea un cambio puntual de horario WHEN define al menos una franja horaria válida THEN el cambio se guarda correctamente.

### Validación server-side

- GIVEN un request llega al backend para crear o actualizar un registro de día abierto WHEN el payload no incluye al menos una franja horaria con start_time y end_time válidos THEN el servidor responde con HTTP 422 y mensaje: "Un día marcado como abierto debe tener al menos una franja horaria definida".
- GIVEN un request llega al backend para crear un festivo o excepción de tipo apertura WHEN el payload no incluye franjas horarias THEN el servidor responde con HTTP 422 y el mismo mensaje de error.
- GIVEN un request válido llega al backend con franjas horarias completas WHEN se procesa THEN el registro se guarda sin error.

### Detección de datos existentes con el problema

- GIVEN la aplicación se conecta a datos migrados del sistema legacy WHEN existen días marcados como abiertos sin franjas horarias THEN esos días se señalan visualmente con un indicador de advertencia (icono AlertTriangle, color destructive) y tooltip: "Este día aparece como abierto 24h. Revisa y define el horario correcto".
- GIVEN existen datos con el problema de disponibilidad 24/7 WHEN el usuario accede al panel de monitorización (Wave 6, RF-ADM-001) THEN se muestra un contador de establecimientos afectados con enlace para corregirlos.

### Edge cases

- GIVEN un período está marcado como "Período de cierre" (SPEC-001) WHEN la validación se ejecuta THEN no se aplica esta constraint (un período de cierre no requiere franjas por definición).
- GIVEN un día está marcado explícitamente como "Cerrado" (sin franja) WHEN la validación se ejecuta THEN se acepta como válido (cerrado es un estado explícito, no una omisión).
- GIVEN el usuario elimina todas las franjas de un día que tenía horario WHEN confirma la eliminación THEN el día pasa a "Cerrado" automáticamente, no a "abierto sin horario".

## UX Design

### Wireframe textual

Esta spec no tiene pantalla propia. Las validaciones se integran en las pantallas existentes:

**En el editor de horario semanal (SPEC-002, Card expandida):**
- Los días sin franja y no marcados como cerrados muestran un borde izquierdo en color destructive y el mensaje de error inline debajo de la fila.
- El botón "Guardar horario" se desactiva mientras exista al menos un día con error. Tooltip: "Hay días sin horario definido. Corrige los errores antes de guardar".

**En los formularios de festivos y excepciones (Wave 2):**
- El campo de franjas horarias se marca como requerido (asterisco) cuando el tipo es "apertura" o "cambio puntual".
- El botón de guardar se desactiva hasta que se complete al menos una franja. Tooltip con el mensaje descriptivo.

**En datos migrados con el problema:**
- Icono AlertTriangle (Lucide, 16px, color destructive) al lado del horario del día afectado.
- Tooltip al hover: "Este día aparece como abierto 24h. Revisa y define el horario correcto".
- Badge "Requiere revisión" (variant destructive) en la Card del período si alguno de sus días tiene el problema.

### Componentes shadcn utilizados

Componentes: Tooltip, Badge, Toast (todos ya presentes en specs anteriores). No se requieren componentes adicionales.

### Patrón de interacción

- **Botón desactivado con Tooltip explicativo** en lugar de permitir guardar y mostrar error después: previene el error antes de que ocurra. (Regla: botón disabled siempre con Tooltip explicativo.)
- **Error inline en la fila del día** en lugar de error global: el usuario identifica exactamente qué día tiene el problema sin buscar. (Regla: validación inline on blur.)
- **Indicador visual para datos legacy** en lugar de corrección automática: el sistema no puede asumir qué horario debería tener el día, así que señala el problema para que el usuario lo resuelva con criterio. (Decisión no cubierta explícitamente por el design system. Se resuelve con icono AlertTriangle + Tooltip, por analogía con los estados de advertencia del design system.)

### Comportamiento responsive

- **Mobile (< md):** Los mensajes de error inline se muestran debajo del bloque del día. El icono AlertTriangle mantiene su tamaño (16px). El Tooltip se sustituye por texto visible permanente en mobile (los tooltips no funcionan bien con touch).
- **Tablet (md-lg):** Interpolado.
- **Desktop (lg+):** Como descrito en el wireframe.

## Notas técnicas

- La constraint server-side se implementa como validación en la capa de aplicación (no como constraint de base de datos), porque la lógica depende del tipo de registro (un período de cierre es válido sin franjas, un día abierto no).
- La detección de datos legacy con el problema se ejecuta con una query: días/festivos/excepciones de tipo "apertura" que no tienen registros en la tabla de franjas horarias. Esta query puede ejecutarse como migración de datos al desplegar.
- Dependencia upstream: SPEC-002 (editor de horario semanal), SPEC-003 (horario partido).
- Dependencia downstream: Wave 2 specs (festivos y excepciones) deben implementar esta validación en sus formularios.
- La transición de "día abierto sin horario" a "Cerrado" al eliminar todas las franjas (edge case) ya está cubierta en SPEC-002. Esta spec refuerza que ese es el único camino válido.
