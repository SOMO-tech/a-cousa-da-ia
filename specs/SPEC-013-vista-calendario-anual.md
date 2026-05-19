# SPEC-013 — Vista de calendario anual

> RF: RF-UXI-002
> Criticidad: Must
> Wave: 3 — Visualización, copia interanual y validación

## Descripción

Ofrece una representación visual del año completo en formato de calendario mensual (12 meses) donde cada día muestra su estado a golpe de vista: horario normal, festivo de cierre, festivo de apertura, cambio puntual, cierre temporal o día sin cobertura. En M7 existía una vista de calendario valorada por los usuarios, pero la navegación hacia la edición era indirecta. Esta spec define la visualización pura. La interacción de edición directa desde el calendario se cubre en SPEC-014 (RF-UXI-004).

La vista de calendario complementa las secciones de lista (períodos, festivos, cambios puntuales, cierres temporales) definidas en specs anteriores. No las sustituye: el usuario puede trabajar con las listas o con el calendario según prefiera.

## Criterios de aceptación

### Disposición general

- GIVEN el usuario está en la página de horario comercial de un establecimiento WHEN la página carga THEN se muestra una sección "Calendario anual" debajo de las secciones de cierres temporales, con los 12 meses del año seleccionado.
- GIVEN la sección de calendario está visible WHEN el usuario la observa THEN cada mes se muestra como una cuadrícula de 7 columnas (L-D) con los días del mes como celdas.
- GIVEN la pantalla es desktop (lg+) WHEN se renderiza el calendario THEN se muestran los 12 meses en un grid de 3 columnas x 4 filas (3 meses por fila).
- GIVEN la pantalla es tablet (md-lg) WHEN se renderiza el calendario THEN se muestran los meses en un grid de 2 columnas.
- GIVEN la pantalla es mobile (< md) WHEN se renderiza el calendario THEN se muestran los meses en una sola columna, con scroll vertical.

### Codificación visual por estado del día

- GIVEN un día tiene horario normal del período WHEN se renderiza la celda THEN la celda muestra fondo neutro (sin color especial), sin indicador adicional.
- GIVEN un día es un festivo de cierre WHEN se renderiza la celda THEN la celda muestra un indicador de color rojo/destructive (dot o fondo sutil) y tooltip con el nombre del festivo.
- GIVEN un día es un festivo de apertura con horario habitual WHEN se renderiza la celda THEN la celda muestra un indicador de color verde/success (dot) y tooltip con "Festivo de apertura: [nombre] — [horario]".
- GIVEN un día es un festivo de apertura con horario especial WHEN se renderiza la celda THEN la celda muestra un indicador de color verde con acento diferenciado (dot con borde) y tooltip con "Festivo de apertura: [nombre] — [horario especial]".
- GIVEN un día tiene un cambio puntual de tipo "Horario diferente" WHEN se renderiza la celda THEN la celda muestra un indicador de color azul/info (dot) y tooltip con "Cambio puntual: [motivo] — [horario]".
- GIVEN un día tiene un cambio puntual de tipo "Cierre puntual" WHEN se renderiza la celda THEN la celda muestra un indicador de color naranja/warning (dot) y tooltip con "Cierre puntual: [motivo]".
- GIVEN un día está dentro de un cierre temporal WHEN se renderiza la celda THEN la celda muestra fondo gris con patrón de rayas diagonales (indicando inactividad) y tooltip con "Cierre temporal: [motivo] — del [inicio] al [fin]".
- GIVEN un día no está cubierto por ningún período (hueco de cobertura) WHEN se renderiza la celda THEN la celda muestra fondo amarillo/warning sutil con tooltip "Sin horario definido".
- GIVEN un día es la fecha actual WHEN se renderiza la celda THEN la celda muestra un borde destacado (ring) para indicar "hoy".

### Resolución de prioridad visual

- GIVEN un día tiene múltiples estados posibles (ej: festivo + cierre temporal) WHEN se renderiza la celda THEN se aplica la misma prioridad de resolución que el sistema de horarios: (1) Cierre temporal → (2) Festivo → (3) Cambio puntual → (4) Horario del período. Solo se muestra el indicador del estado de mayor prioridad.
- GIVEN un día tiene un cierre temporal y además es festivo WHEN el usuario ve la celda THEN la celda muestra el indicador de cierre temporal (el festivo queda subordinado). El tooltip menciona ambos: "Cierre temporal: [motivo]. También festivo: [nombre]".

### Tooltip informativo

- GIVEN el usuario hace hover sobre una celda de día WHEN el tooltip aparece THEN muestra: fecha completa (ej: "Lunes 25 de diciembre de 2026"), tipo de estado (Festivo / Cambio puntual / Cierre temporal / Horario normal / Sin cobertura), nombre o motivo (si aplica), horario resuelto (ej: "10:00 - 18:00" o "Cerrado").
- GIVEN el usuario está en mobile WHEN toca una celda THEN el tooltip se muestra como un Popover debajo de la celda (no depende de hover).

### Leyenda

- GIVEN la sección de calendario está visible WHEN el usuario la observa THEN se muestra una leyenda compacta debajo del título de la sección, con los 6 indicadores de color y su significado: Horario normal, Festivo de cierre, Festivo de apertura, Cambio puntual, Cierre temporal, Sin cobertura.
- GIVEN la leyenda está visible WHEN el usuario la observa THEN cada ítem de la leyenda muestra un dot del color correspondiente y su etiqueta (texto small, muted).

### Navegación entre años

- GIVEN la sección de calendario está visible WHEN el usuario cambia de año en el selector de año existente (definido en SPEC-001) THEN el calendario se actualiza mostrando los datos del nuevo año seleccionado.
- GIVEN el calendario se actualiza tras cambio de año WHEN el fetch está en curso THEN las celdas muestran Skeleton placeholders (pulso de carga).

### Scroll y ancla

- GIVEN el calendario tiene 12 meses renderizados WHEN la página carga THEN el calendario hace scroll automático hasta el mes actual (si el año seleccionado es el año en curso). Si el año no es el actual, no hay scroll automático.

### Estadísticas resumen

- GIVEN la sección de calendario está visible WHEN los datos están cargados THEN se muestra un bloque de estadísticas encima del grid de meses con: "N días con horario normal", "N festivos", "N cambios puntuales", "N días de cierre temporal", "N días sin cobertura". Cada contador usa el dot del color correspondiente.
- GIVEN hay días sin cobertura WHEN se muestran las estadísticas THEN el contador "N días sin cobertura" usa texto de advertencia (destructive) para llamar la atención.

### Edge cases

- GIVEN el año seleccionado es bisiesto WHEN se renderiza febrero THEN se muestran 29 días.
- GIVEN un cierre temporal cruza el límite de año (diciembre-enero) WHEN se renderiza el calendario del año actual THEN solo se muestran como cierre temporal los días que caen en el año seleccionado.
- GIVEN el establecimiento no tiene ningún período definido WHEN se renderiza el calendario THEN todos los días aparecen como "Sin cobertura" (amarillo). Las estadísticas muestran "365 días sin cobertura".
- GIVEN la página se abre en un dispositivo con pantalla muy pequeña (<320px) WHEN se renderiza un mes THEN los nombres de los días de la semana se abrevian a una sola letra (L, M, X, J, V, S, D).

## UX Design

### Wireframe textual

**Sección de Calendario anual (dentro de la página de Horario Comercial, Layout 1)**

Zona de título de sección:
- Heading "Calendario anual" (h3, font-medium).
- Leyenda inline a la derecha del heading: 6 dots de color con etiqueta, separados por espacio (layout flex wrap).

Bloque de estadísticas:
- Fila de contadores horizontales (flex wrap), cada uno con: dot de color (8px, rounded-full) + número + etiqueta. Ej: "● 280 días normales  ● 15 festivos  ● 5 cambios puntuales  ● 14 días cierre temporal  ● 51 sin cobertura".

Grid de meses:
- 12 bloques de mes en grid responsive (3 col desktop, 2 col tablet, 1 col mobile).
- Cada bloque de mes:
  - Heading del mes (texto small, font-medium, ej: "Enero 2026").
  - Fila de cabecera: L M X J V S D (texto xs, muted).
  - Grid de 7 columnas con las celdas de cada día.
  - Cada celda: número del día (texto xs), dot de estado (6px) en esquina superior derecha si el estado no es "normal".
  - Celda de hoy: ring/borde destacado.
  - Celdas vacías (antes del primer día y después del último) sin contenido.

**Tooltip (Tooltip de shadcn o Popover en mobile)**
- Contenido: Fecha completa, estado, nombre/motivo, horario.

### Componentes shadcn utilizados

Componentes: Tooltip (nuevo en esta spec, para hover informativo en cada celda), Popover (para mobile, alternativa táctil al Tooltip), Skeleton. El resto de componentes (Button, Badge) ya están presentes.

### Patrón de interacción

- **Vista de solo lectura:** el calendario es informativo. No se edita directamente desde aquí (eso es SPEC-014/RF-UXI-004). El usuario escanea visualmente el año y detecta patrones o anomalías.
- **Dots de color en lugar de fondos completos:** en un calendario compacto, los fondos completos generan "ruido visual" excesivo. Los dots permiten ver el número del día con claridad y distinguir estados por posición del dot. (Decisión no cubierta por el design system. Se resuelve priorizando legibilidad.)
- **Excepción: cierre temporal con fondo gris rayado:** el cierre temporal afecta rangos de días consecutivos. El fondo continuo crea una banda visual que comunica "este tramo está inactivo", a diferencia de los dots individuales. (Decisión de UX: la semántica visual de "bloqueo" requiere un tratamiento diferente al de eventos puntuales.)
- **Leyenda siempre visible** (no en un Popover o Accordion): el usuario necesita la referencia de colores sin tener que buscarla. Con 6 estados, cabe en una línea. (Regla: información de referencia frecuente siempre visible.)
- **Estadísticas como contadores numéricos:** permiten detectar anomalías de cobertura sin contar visualmente. "51 días sin cobertura" es una alarma inmediata que las celdas amarillas dispersas no comunican con la misma claridad. (Decisión de UX: complementar la visualización con datos cuantitativos.)
- **Scroll al mes actual:** evita que el usuario tenga que buscar "dónde estamos ahora" al entrar en la página. (Decisión de UX: orientar al usuario temporalmente.)

### Comportamiento responsive

- **Mobile (< md):** Meses en 1 columna, scroll vertical. Tooltips como Popover (tap para abrir, tap fuera para cerrar). Leyenda en layout vertical (2 columnas de dots). Estadísticas apiladas en 2 filas.
- **Tablet (md-lg):** Meses en 2 columnas. Tooltips con hover. Leyenda horizontal.
- **Desktop (lg+):** Meses en 3 columnas. Layout completo como descrito.

## Notas técnicas

- El calendario se renderiza a partir de los datos ya cargados por las secciones anteriores (períodos, festivos, cambios puntuales, cierres temporales). No requiere un endpoint adicional. La resolución de estado de cada día se calcula client-side aplicando la prioridad: cierre temporal → festivo → cambio puntual → horario del período → sin cobertura.
- Para 365 días, la resolución es O(N) donde N = total de registros (típicamente < 100). No hay problema de rendimiento.
- El componente de calendario es custom (no existe un componente shadcn para calendario anual de 12 meses). Se implementa como un grid CSS. No se usa el Calendar de shadcn (diseñado para selección de fecha, no para visualización de estado anual).
- El auto-scroll al mes actual se implementa con `scrollIntoView({ behavior: 'smooth', block: 'nearest' })` sobre el bloque del mes actual, con un pequeño delay tras el render inicial.
- Los dots de estado usan clases de Tailwind con colores semánticos: `bg-destructive` (cierre festivo), `bg-green-500` (apertura festivo), `bg-blue-500` (cambio puntual), `bg-orange-500` (cierre puntual), `bg-muted` (cierre temporal). El color de "sin cobertura" usa `bg-yellow-400/30` (fondo sutil).
- Componente nuevo: Tooltip de shadcn. Se añade al inventario de componentes del proyecto.
- Dependencia upstream: SPEC-001 (períodos), SPEC-007 (festivos), SPEC-010 (cambios puntuales), SPEC-011 (cierres temporales), SPEC-005 (glosario de términos para las etiquetas).
- Dependencia downstream: SPEC-014 (RF-UXI-004, edición directa desde calendario, que añadirá interactividad click sobre las celdas de este calendario).
