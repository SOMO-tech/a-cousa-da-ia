# SPEC-016 — Validación de cobertura anual

> RF: RF-HOR-006
> Criticidad: Must
> Wave: 3 — Visualización, copia interanual y validación

## Descripción

Alerta al usuario cuando la configuración de períodos no cubre los 365 (o 366) días del año, sin bloquear el guardado ni impedir el trabajo. En el sistema legacy no existía ningún mecanismo de detección: los huecos de cobertura pasaban desapercibidos y provocaban errores en los sistemas downstream (fichajes, planificación). Esta spec implementa una validación informativa que se manifiesta en tres puntos: la barra de cobertura (SPEC-001), las estadísticas del calendario (SPEC-013) y un Alert dedicado.

## Criterios de aceptación

### Cálculo de cobertura

- GIVEN el establecimiento tiene períodos definidos para un año WHEN el sistema calcula la cobertura THEN suma los días cubiertos por todos los períodos (sin doble-contar solapamientos, aunque los solapamientos no deberían existir por la validación de SPEC-001). La cobertura se expresa como número de días y porcentaje sobre el total de días del año.
- GIVEN el año tiene 365 días WHEN todos los períodos juntos cubren exactamente 365 días THEN la cobertura es 100%.
- GIVEN el año es bisiesto (366 días) WHEN se calcula la cobertura THEN se usa 366 como denominador.

### Barra de cobertura (modificación de SPEC-001)

- GIVEN el establecimiento tiene períodos definidos WHEN la barra de cobertura se renderiza THEN los tramos cubiertos se muestran en color neutro (como ya define SPEC-001) y los huecos se muestran en color warning (amarillo/naranja) con un patrón visual diferenciado (rayas diagonales o fondo sutil).
- GIVEN hay huecos de cobertura WHEN el usuario hace hover sobre un hueco en la barra THEN se muestra un Tooltip: "Sin cobertura: del [fecha inicio hueco] al [fecha fin hueco] ([N] días)".
- GIVEN la cobertura es 100% WHEN la barra se renderiza THEN la barra es continua sin huecos y se muestra un texto "Cobertura completa" (texto small, muted, color success).
- GIVEN la cobertura es inferior al 100% WHEN la barra se renderiza THEN se muestra debajo de la barra un texto: "[N] días sin cobertura ([X]%)" en color warning.

### Alert de cobertura incompleta

- GIVEN la cobertura es inferior al 100% WHEN la página carga THEN se muestra un Alert (variant default, icono AlertTriangle) debajo de la barra de cobertura: "La configuración no cubre [N] días del año. Los días sin horario definido pueden generar errores en fichajes y planificación. Revisa los huecos marcados en la barra de cobertura".
- GIVEN la cobertura es 100% WHEN la página carga THEN no se muestra ningún Alert de cobertura.
- GIVEN la cobertura es inferior al 100% WHEN el Alert se muestra THEN el Alert NO incluye botón de descarte. Es persistente mientras haya huecos (a diferencia del Alert de copia interanual que sí se puede descartar).

### Integración con calendario anual (SPEC-013)

- GIVEN hay huecos de cobertura WHEN el calendario anual se renderiza THEN los días sin cobertura se muestran con fondo amarillo/warning sutil (ya definido en SPEC-013). Las estadísticas del calendario muestran "N días sin cobertura" en color destructive.
- GIVEN el usuario corrige un hueco de cobertura (crea o extiende un período) WHEN la página se actualiza THEN el calendario y las estadísticas reflejan la nueva cobertura inmediatamente.

### Identificación de huecos específicos

- GIVEN hay huecos de cobertura WHEN la página carga THEN el sistema identifica cada hueco como un rango continuo de fechas sin período. Un año con un período de enero a junio y otro de septiembre a diciembre tiene un hueco: julio-agosto.
- GIVEN hay múltiples huecos WHEN la barra de cobertura se renderiza THEN cada hueco se muestra como un segmento independiente en la barra.

### Comportamiento tras acciones del usuario

- GIVEN el usuario crea un nuevo período que cubre un hueco WHEN guarda el período THEN la barra de cobertura se actualiza, el hueco desaparece (total o parcialmente), y si la cobertura llega al 100%, el Alert desaparece y se muestra "Cobertura completa".
- GIVEN el usuario elimina un período WHEN la eliminación genera un nuevo hueco THEN la barra se actualiza mostrando el nuevo hueco, el Alert aparece (si no estaba ya), y las estadísticas del calendario se actualizan.
- GIVEN el usuario modifica las fechas de un período (acorta el rango) WHEN el cambio genera un hueco THEN el mismo comportamiento que al eliminar.

### No bloqueo

- GIVEN la cobertura es inferior al 100% WHEN el usuario intenta crear festivos, cambios puntuales o cierres temporales en fechas sin cobertura THEN las operaciones se permiten. Los festivos y cambios puntuales en fechas sin período mostrarán sus advertencias específicas (SPEC-008, SPEC-010) pero no se bloquean por la cobertura incompleta.
- GIVEN la cobertura es inferior al 100% WHEN el usuario intenta copiar la configuración a otro año (SPEC-015) THEN la copia se permite con la advertencia ya definida en SPEC-015.

### Edge cases

- GIVEN un período comienza el 2 de enero (no el 1) WHEN se calcula la cobertura THEN el 1 de enero se marca como hueco (1 día sin cobertura).
- GIVEN un período termina el 30 de diciembre (no el 31) WHEN se calcula la cobertura THEN el 31 de diciembre se marca como hueco.
- GIVEN no hay ningún período definido WHEN se calcula la cobertura THEN la cobertura es 0% y se muestran 365 (o 366) días sin cobertura. La barra de cobertura está completamente en warning.
- GIVEN existen períodos que cubren el año completo pero el usuario tiene festivos fuera de esos períodos WHEN se calcula la cobertura THEN la cobertura se basa solo en los períodos. Los festivos no contribuyen a la cobertura (un festivo es una modificación sobre un período, no un sustituto).

## UX Design

### Wireframe textual

**Barra de cobertura (modificación de SPEC-001)**

La barra ya existe en SPEC-001. Se modifica:
- Segmentos cubiertos: fondo neutro (sin cambio).
- Segmentos de hueco: fondo warning (amarillo/naranja), con Tooltip al hover.
- Texto debajo de la barra:
  - Si 100%: "✓ Cobertura completa" (texto small, color success).
  - Si < 100%: "[N] días sin cobertura ([X]%)" (texto small, color warning).

**Alert de cobertura (debajo de la barra)**

- Alert (variant default, icono AlertTriangle).
- Texto: "La configuración no cubre [N] días del año. Los días sin horario definido pueden generar errores en fichajes y planificación".
- Sin botón de descarte.

### Componentes shadcn utilizados

Componentes: Alert, Tooltip (ya presentes). No se requieren componentes adicionales.

### Patrón de interacción

- **Alerta informativa, no bloqueante:** el PRD especifica explícitamente "alertar pero no bloquear". Un bloqueo impediría al usuario guardar trabajo parcial y generaría frustración. La alerta cumple la función de informar sin impedir. (Regla del PRD: "no bloquear el guardado".)
- **Alert persistente sin botón de descarte:** a diferencia del Alert de copia interanual (SPEC-015) que se puede descartar, este Alert está vinculado a un estado del sistema. Mientras haya huecos, el Alert tiene sentido. Descartarlo sería ocultar un problema activo. (Decisión de UX: los Alerts vinculados a estado del sistema son persistentes.)
- **Triple punto de detección:** barra de cobertura (visual), estadísticas del calendario (cuantitativo), Alert (textual). El usuario tiene tres oportunidades de detectar el problema, cada una con un nivel de detalle diferente. (Decisión de UX: redundancia informativa para errores con alto impacto downstream.)
- **Texto "Cobertura completa" como refuerzo positivo:** cuando el usuario logra el 100%, el sistema confirma que todo está correcto. Esto cierra el loop de la tarea y da confianza. (Decisión de UX: feedback positivo explícito tras resolución de advertencia.)

### Comportamiento responsive

- **Mobile (< md):** Barra de cobertura a ancho completo. Alert a ancho completo. Texto de cobertura debajo de la barra.
- **Tablet (md-lg):** Interpolado.
- **Desktop (lg+):** Como descrito.

## Notas técnicas

- El cálculo de cobertura se realiza client-side a partir de los períodos cargados. Algoritmo: (1) Crear un array de 365/366 posiciones (una por día). (2) Para cada período, marcar los días cubiertos. (3) Contar los días no marcados. (4) Agrupar días no marcados consecutivos en rangos (huecos).
- La barra de cobertura se renderiza como un div con ancho proporcional al año (cada día = 100%/365 de ancho). Los segmentos se posicionan con `left` y `width` en porcentaje.
- No se necesita endpoint adicional: el cálculo usa los datos de períodos ya disponibles en el state.
- Los huecos de cobertura no se almacenan en base de datos. Son un cálculo derivado de los períodos.
- Dependencia upstream: SPEC-001 (barra de cobertura, períodos), SPEC-013 (estadísticas del calendario).
- Dependencia downstream: ninguna directa. Es una spec de validación informativa.
