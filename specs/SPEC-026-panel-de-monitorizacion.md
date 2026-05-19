# SPEC-026 — Panel de monitorización

> RF: RF-ADM-001
> Criticidad: Must
> Wave: 6 — Soporte, monitorización y polish

## Descripción

Proporciona al equipo de soporte (Large Format) y a los administradores una vista centralizada del estado de configuración de todos los establecimientos. Permite detectar proactivamente establecimientos con configuración incompleta o errónea antes de que los problemas impacten a sistemas downstream (Google, stock, entregas). Esta página es una sección independiente del sidebar, no un sub-apartado del dashboard de establecimiento ni de settings. Para los roles `administrador` y `soporte`, es la página de inicio al acceder a Aurora. Desde ella se navega al dashboard de un establecimiento concreto seleccionándolo.

## Criterios de aceptación

### Navegación y acceso

- GIVEN un usuario con rol `administrador` o `soporte` WHEN accede a Aurora THEN la página de inicio es el Panel de monitorización (ruta `/monitoring`), no el dashboard de un establecimiento.
- GIVEN un usuario con rol `hr_mercado` o `establecimiento` WHEN accede a Aurora THEN la página de inicio es su vista habitual (establecimientos del mercado o establecimiento asignado). No ven la sección de monitorización en el sidebar.
- GIVEN el sidebar se renderiza WHEN el usuario tiene rol `administrador` o `soporte` THEN existe una entrada "Monitorización" (icono Activity de Lucide) como primera opción del sidebar, antes de "Establecimientos".
- GIVEN un usuario con rol `hr_mercado` o `establecimiento` WHEN el sidebar se renderiza THEN la entrada "Monitorización" no aparece.

### KPIs de resumen

- GIVEN un administrador o soporte accede al Panel de monitorización WHEN la página carga THEN se muestra una zona superior con 4 KPI cards:
  - "Total establecimientos" (número total de establecimientos activos).
  - "Configuración completa" (establecimientos sin ninguna alerta, con porcentaje sobre el total). Color success si > 90%, warning si 70-90%, destructive si < 70%.
  - "Con alertas" (establecimientos con al menos una alerta activa, con porcentaje). Color destructive si > 0.
  - "Sin configurar" (establecimientos activos sin ningún período definido para el año en curso, con porcentaje). Color destructive si > 0.
- GIVEN la página carga WHEN los KPIs se están calculando THEN se muestran Skeleton placeholders en las 4 cards.
- GIVEN el fetch de KPIs falla WHEN la página intenta cargar THEN las KPI cards muestran un estado de error con icono y texto "Error al cargar" en cada card.

### Selector de año y mercado

- GIVEN el Panel de monitorización carga WHEN se muestra la zona de filtros THEN existe un Select de año (por defecto el año en curso, opciones: año actual y siguiente) y un Select de mercado (opciones: "Todos los mercados" + lista de mercados activos). Los KPIs y la tabla se actualizan al cambiar cualquier filtro.
- GIVEN el usuario tiene rol `soporte` WHEN accede al panel THEN el Select de mercado muestra "Todos los mercados" por defecto.
- GIVEN el usuario tiene rol `administrador` WHEN accede al panel THEN el Select de mercado muestra "Todos los mercados" por defecto.

### Tabla de establecimientos

- GIVEN el Panel de monitorización carga WHEN existen establecimientos activos THEN se muestra una tabla con todos los establecimientos, ordenados por defecto con los que tienen más alertas primero (orden descendente por número de alertas, luego alfabético por nombre).
- GIVEN la tabla se renderiza WHEN hay datos THEN las columnas son: Nombre del establecimiento (texto clickable que navega al dashboard del establecimiento), Mercado (código ISO + nombre, ej: "ES — España"), Región (nombre), Tipo (Badge "Propia" variant default / "Franquicia" variant outline), Alertas (número con Badge: destructive si > 0, secondary si 0), Estado de configuración (Badge: "Completa" variant success, "Incompleta" variant warning, "Sin configurar" variant destructive).
- GIVEN la tabla tiene más de 20 establecimientos WHEN se renderiza THEN la paginación es server-side con "Mostrando 1-20 de [total]", botones Anterior/Siguiente y selector de tamaño (20, 50, 100).
- GIVEN la tabla se muestra WHEN el usuario usa la barra de búsqueda THEN filtra por nombre o código de establecimiento (debounce 300ms, server-side).
- GIVEN la tabla se muestra WHEN el usuario usa filtros adicionales THEN soporta: mercado (ya en zona superior), región (Select, opciones del mercado seleccionado), tipo (Todas / Propia / Franquicia), estado de configuración (Todos / Completa / Incompleta / Sin configurar), con alertas (Todos / Solo con alertas). Filtros activos se muestran como Badges con botón cerrar. Botón "Limpiar filtros" visible con 1+ filtros activos.

### Navegación al establecimiento

- GIVEN un administrador o soporte ve la tabla de establecimientos WHEN hace clic en el nombre de un establecimiento THEN navega a la página de horario comercial de ese establecimiento para el año seleccionado (la misma vista que usa Lucía en SPEC-001).
- GIVEN el usuario está en la página de un establecimiento WHEN quiere volver al panel THEN usa el breadcrumb "Monitorización > [nombre establecimiento]" o el sidebar para volver.
- GIVEN un establecimiento tiene alertas WHEN el usuario navega a su página THEN las alertas se manifiestan en la propia página del establecimiento (barra de cobertura en SPEC-016, alertas de festivos en SPEC-008, etc.). El panel de monitorización no duplica la interfaz de corrección, solo identifica dónde están los problemas.

### Tipos de alerta detectadas

- GIVEN un establecimiento no tiene ningún período definido para el año seleccionado WHEN el sistema evalúa alertas THEN genera alerta "Sin configurar" (severity: critical). Descripción: "No tiene ningún período de horario definido para [año]".
- GIVEN un establecimiento tiene períodos pero la cobertura anual es inferior al 100% (SPEC-016) WHEN el sistema evalúa alertas THEN genera alerta "Cobertura incompleta" (severity: high). Descripción: "[N] días sin horario definido".
- GIVEN un establecimiento tiene un festivo de apertura en conflicto, es decir, hereda horario de un período pero el día está cerrado o el período fue eliminado (SPEC-008) WHEN el sistema evalúa alertas THEN genera alerta "Festivo en conflicto" (severity: high). Descripción: "Festivo '[nombre]' sin horario resolvible".
- GIVEN un establecimiento tiene un cierre temporal activo que lleva más de 90 días (SPEC-011) WHEN el sistema evalúa alertas THEN genera alerta "Cierre prolongado" (severity: medium). Descripción: "Cierre temporal '[motivo]' activo desde hace [N] días".
- GIVEN un establecimiento no tiene festivos definidos para el año seleccionado y su mercado/región sí tiene festivos propagados (SPEC-020) WHEN el sistema evalúa alertas THEN no genera alerta (los festivos heredados cuentan como configurados).
- GIVEN un establecimiento tiene cobertura 100%, sin festivos en conflicto y sin cierres prolongados WHEN el sistema evalúa alertas THEN el establecimiento tiene 0 alertas y estado "Completa".

### Panel de detalle de alertas (expandible por fila)

- GIVEN un establecimiento tiene alertas WHEN el usuario hace clic en el Badge de alertas o en el icono expand de la fila THEN la fila se expande mostrando la lista de alertas del establecimiento: cada alerta con icono de severity (AlertTriangle destructive para critical/high, AlertCircle warning para medium), descripción y un link "Ir a corregir" que navega a la sección relevante de la página del establecimiento.
- GIVEN un establecimiento tiene 0 alertas WHEN el usuario hace clic en expand THEN la fila muestra un texto: "Sin alertas. Configuración completa para [año]."

### Empty state

- GIVEN no hay establecimientos activos WHEN el panel carga THEN se muestra empty state centrado: icono Building2 (Lucide, 24px), mensaje "No hay establecimientos activos en el sistema", botón "Ir a Administración" (Button variant default) que navega a la sección de establecimientos de SPEC-018.
- GIVEN los filtros no devuelven resultados WHEN la tabla se renderiza THEN se muestra empty state: "No se encontraron establecimientos con estos filtros", botón "Limpiar filtros" (variant outline).

### Error state

- GIVEN el fetch de la tabla falla WHEN la página intenta cargar THEN se muestra error state centrado con icono de error, mensaje "No se pudieron cargar los establecimientos" y botón "Reintentar" (variant outline).

### Loading state

- GIVEN la página está cargando WHEN los datos aún no han llegado THEN los KPIs muestran Skeleton, la tabla muestra Skeleton rows (5 filas placeholder con las columnas aproximadas).

### Edge cases

- GIVEN un establecimiento fue desactivado WHEN el panel carga THEN el establecimiento no aparece en la tabla ni cuenta para los KPIs (solo se monitorizan establecimientos activos).
- GIVEN el año seleccionado es el siguiente (ej: 2027) WHEN muchos establecimientos no tienen configuración THEN el KPI "Sin configurar" puede ser alto. Esto es esperable al inicio de la campaña de carga anual, no es un error.
- GIVEN un establecimiento tiene festivos heredados del mercado (SPEC-020) pero ningún festivo local WHEN el sistema evalúa su estado THEN los festivos heredados cuentan como configuración válida. Solo se alerta si un festivo heredado está en conflicto con los períodos locales.
- GIVEN la evaluación de alertas es costosa (requiere analizar períodos, festivos, cierres de cada establecimiento) WHEN la página carga THEN las alertas se calculan server-side y se cachean. El cache se invalida cuando se modifica cualquier dato operativo del establecimiento. El badge de alertas en la tabla refleja el estado cacheado.
- GIVEN un administrador accede al panel WHEN también tiene acceso al panel de administración (SPEC-019) THEN el sidebar muestra ambas secciones: "Monitorización" (primera) y "Administración" (separada). Son secciones independientes.

## UX Design

### Wireframe textual

**Panel de monitorización (página nueva, sección independiente del sidebar)**

Layout 1 — Página estándar (dashboard/list).

Zona de título:
- Heading "Monitorización" (h2).
- Texto subtítulo (muted): "Estado de configuración de establecimientos para [año seleccionado]".

Zona de KPIs (debajo del título):
- Grid horizontal de 4 KPI cards (en mobile, grid 2-col):
  - Card 1: label "Total establecimientos", valor numérico grande (ej: "1.247"), sin delta.
  - Card 2: label "Configuración completa", valor numérico + porcentaje (ej: "1.180 (94,6%)"), icono CheckCircle, borde izquierdo coloreado según umbral (success/warning/destructive).
  - Card 3: label "Con alertas", valor numérico + porcentaje (ej: "52 (4,2%)"), icono AlertTriangle, borde izquierdo destructive si > 0.
  - Card 4: label "Sin configurar", valor numérico + porcentaje (ej: "15 (1,2%)"), icono CircleSlash, borde izquierdo destructive si > 0.

Zona de filtros (debajo de KPIs):
- Fila de filtros alineada a la izquierda:
  - Select de año (opciones: año actual, año siguiente). Compact, sin label visible (el subtítulo ya indica el año).
  - Select de mercado (opciones: "Todos los mercados" + mercados activos).
  - Select de región (opciones: "Todas las regiones" + regiones del mercado seleccionado). Deshabilitado si mercado = "Todos".
  - Select de tipo (opciones: "Todos", "Propia", "Franquicia").
  - Select de estado (opciones: "Todos", "Completa", "Incompleta", "Sin configurar").
- Input de búsqueda a la derecha (placeholder: "Buscar establecimiento…").
- Badges de filtros activos debajo, con botón cerrar. Botón "Limpiar filtros" (ghost, icono X).

Tabla de establecimientos (zona principal):
- Columnas: Nombre (texto clickable, font-medium, link a la página del establecimiento), Mercado, Región, Tipo (Badge), Alertas (Badge numérico), Estado (Badge).
- Cada fila tiene icono expand (ChevronRight que rota a ChevronDown) como primera columna visual para abrir el detalle de alertas inline.
- Al expandir: bloque debajo de la fila con lista de alertas (icono severity + descripción + link "Ir a corregir"). Fondo ligeramente diferenciado (accent).
- Acciones por fila: no hay DropdownMenu. La acción principal es navegar al establecimiento (clic en nombre). La acción secundaria es expandir alertas.
- Paginación server-side debajo: "Mostrando 1-20 de [total]", Anterior/Siguiente, selector de tamaño (20, 50, 100).

### Componentes shadcn utilizados

Componentes: Card, Table, Input, Select, Button, Badge, Skeleton, Tooltip, Form.

Componente adicional necesario: Collapsible (no instalado en el scaffold base, necesario para expand/collapse de alertas por fila).

### Patrón de interacción

- **Sección independiente en sidebar** en lugar de sub-página de settings o dashboard: la monitorización es una actividad continua del equipo de soporte, no una configuración puntual ni un detalle de un establecimiento concreto. Merece su propia entrada en el sidebar para acceso de un clic. (Decisión no cubierta explícitamente por el design system como "qué merece entrada propia en sidebar". Se resuelve por la regla: sidebar para apps con 3+ secciones, cada sección es una actividad distinta.)
- **Landing page para administrador/soporte** en lugar de un dashboard genérico: el research (Valentina, CU-05) indica que la actividad principal de soporte es detectar y corregir problemas. Aterrizar en la vista de problemas elimina un paso de navegación diario. Los roles operativos (establecimiento, hr_mercado) aterrizan en su contexto habitual porque su actividad principal es diferente.
- **Tabla con expand en lugar de navegación a detalle de alertas**: las alertas de un establecimiento son 1-3 típicamente. No justifican una página separada. El expand inline permite escanear rápidamente qué pasa en cada tienda sin perder el contexto de la lista. (Decisión no cubierta por el design system como "expand por fila". Se resuelve con Collapsible inline, por analogía con Accordion pero dentro de la tabla.)
- **Nombre clickable como navegación principal**: el usuario de soporte necesita ir al establecimiento para corregir. El nombre es el link natural. No se usa row click porque la fila tiene expand. (Regla: nunca combinar row click con otra interacción de fila. Nombre clickable como link explícito.)
- **Sin DropdownMenu por fila**: no hay acciones sobre el establecimiento desde el panel de monitorización. Solo hay dos interacciones: expandir alertas y navegar. No aplica DropdownMenu. (Regla: DropdownMenu para 2+ acciones. Aquí no hay acciones, solo navegación.)
- **KPI cards con umbrales de color**: permiten evaluar de un vistazo si la red está sana o hay problemas sistémicos. Los umbrales (>90% success, 70-90% warning, <70% destructive) son pragmáticos para una red de +1.000 establecimientos. (Regla: zona superior de dashboard con 3-5 KPI cards.)
- **Paginación server-side**: la red tiene miles de establecimientos. No se pueden cargar client-side. (Regla: server-side cuando > 100 filas.)
- **Cache de alertas**: evaluar el estado de cada establecimiento implica consultar períodos, festivos, cierres. Hacerlo en tiempo real para 1.000+ establecimientos en cada carga del panel no es viable. Se cachean server-side con invalidación por cambio.

### Comportamiento responsive

- **Mobile (< md):** Sidebar colapsa a drawer. KPI cards en grid 2-col. Los filtros colapsan: solo búsqueda visible, resto en un botón "Filtros" que abre un Sheet lateral con todos los Selects. La tabla muestra scroll horizontal con columna Nombre sticky. El expand de alertas se mantiene funcional.
- **Tablet (md-lg):** KPI cards en grid 2x2. Filtros en una fila con wrap. Tabla con scroll horizontal si necesario. Sidebar colapsada a iconos.
- **Desktop (lg+):** Layout completo como descrito. KPI cards en fila horizontal. Todos los filtros visibles. Tabla sin scroll horizontal.

## Notas técnicas

- La evaluación de alertas se ejecuta como un proceso server-side (Supabase Edge Function o proceso batch) que calcula el estado de cada establecimiento y lo almacena en una tabla `establishment_alerts` con columnas: `establishment_id`, `alert_type` (enum: 'no_config', 'incomplete_coverage', 'holiday_conflict', 'extended_closure'), `severity` (enum: 'critical', 'high', 'medium'), `description` (texto), `calculated_at` (timestamp). Se recalcula cuando cambian datos operativos del establecimiento (trigger en tablas de períodos, festivos, cambios puntuales, cierres).
- Los KPIs se calculan como agregaciones sobre `establishment_alerts` + tabla de establecimientos. Son queries server-side, no client-side.
- La tabla de establecimientos une `establishments` con un COUNT de `establishment_alerts` para mostrar el número de alertas y el estado. El sorting por defecto (más alertas primero) se resuelve con `ORDER BY alert_count DESC, name ASC`.
- La ruta de landing por rol se gestiona en el router del frontend: roles `administrador` y `soporte` redirigen a `/monitoring`, roles `hr_mercado` y `establecimiento` redirigen a `/establishments` (o al establecimiento asignado si solo hay uno).
- El link "Ir a corregir" en cada alerta apunta a la sección relevante de la página del establecimiento: alertas de cobertura → scroll a la barra de cobertura (SPEC-016), alertas de festivo en conflicto → scroll a la sección de festivos (SPEC-007), alertas de cierre prolongado → scroll a la sección de cierres (SPEC-011). Se puede implementar con anchor links o query params (ej: `/establishments/{id}/schedule/2026#holidays`).
- Dependencia upstream: SPEC-001 (períodos), SPEC-007 y SPEC-008 (festivos y herencia), SPEC-011 (cierres temporales), SPEC-016 (validación de cobertura), SPEC-018 (mercados y regiones), SPEC-019 (roles y RLS), SPEC-020 (festivos multi-nivel).
- Dependencia downstream: RF-ADM-002 (corrección masiva) podría añadir acciones bulk desde esta misma tabla en el futuro (checkboxes + barra de acciones masivas).
