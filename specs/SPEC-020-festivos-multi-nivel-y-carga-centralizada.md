# SPEC-020 — Festivos multi-nivel y carga centralizada

> RF: RF-MPA-002 + RF-FES-003
> Criticidad: Must
> Wave: 4 — Multi-país y carga centralizada

## Descripción

Extiende el sistema de festivos (SPEC-007) para soportar tres niveles de scope: mercado (festivos nacionales), región (festivos autonómicos o sub-nacionales) y establecimiento (festivos locales). Un usuario HR (administrador) crea un festivo nacional una vez a nivel de mercado y ese festivo se propaga automáticamente a todos los establecimientos del país, eliminando la necesidad de registrarlo tienda por tienda. El research identifica esta duplicación como uno de los problemas más reportados: en el sistema legacy, HR distribuía los festivos nacionales por email y cada tienda los cargaba manualmente, generando errores e inconsistencias entre establecimientos del mismo país.

## Criterios de aceptación

### Modelo de scope de festivos

- GIVEN el sistema de festivos está operativo WHEN se crea un festivo THEN tiene exactamente uno de tres scopes: mercado (aplica a todos los establecimientos del mercado), región (aplica a todos los establecimientos de la región) o establecimiento (aplica solo a ese establecimiento).
- GIVEN un festivo tiene scope mercado para ES (España) WHEN un establecimiento de ES consulta su lista de festivos THEN el festivo nacional aparece en su lista como festivo heredado.
- GIVEN un festivo tiene scope región para ES-CT (Cataluña) WHEN un establecimiento de ES-CT consulta su lista THEN el festivo regional aparece. Un establecimiento de ES-MA (Madrid) no lo ve.
- GIVEN un festivo tiene scope establecimiento WHEN se consulta la lista THEN solo aparece en ese establecimiento (comportamiento actual de SPEC-007, sin cambios).

### Resolución y orden de precedencia

- GIVEN un establecimiento pertenece al mercado ES y la región ES-CT WHEN consulta su lista de festivos para un año THEN ve la unión de: festivos nacionales de ES + festivos regionales de ES-CT + festivos locales propios, ordenados por fecha ascendente.
- GIVEN existe un festivo nacional "Navidad" (cierre) el 25/12 y el establecimiento crea un festivo local "Navidad" (apertura, 10:00-18:00) el 25/12 WHEN la lista se renderiza THEN solo aparece el festivo local. El nacional queda oculto (el festivo local tiene prioridad sobre el resto, es decir, los sobreescribe en la visualización).
- GIVEN existe un festivo regional el 11/09 para ES-CT y el establecimiento crea un festivo local el 11/09 WHEN la lista se renderiza THEN solo aparece el festivo local. El regional queda oculto.
- GIVEN un festivo local sobreescribe uno heredado WHEN el usuario elimina el festivo local THEN el festivo heredado reaparece automáticamente en la lista.

### Vista HR: festivos del mercado (pantalla nueva)

- GIVEN un usuario con rol `hr_mercado` o `soporte` o `administrador` WHEN accede a la sección "Festivos del mercado" para un mercado y año concretos THEN ve dos secciones: "Festivos nacionales" (scope mercado) y "Festivos regionales" (agrupados por región).
- GIVEN la sección "Festivos nacionales" se muestra WHEN hay festivos nacionales definidos THEN se renderiza una lista con el mismo formato que SPEC-007: fecha, nombre, tipo (Badge "Cierre"/"Apertura"), horario si aplica. Acciones por fila: DropdownMenu con "Editar" y "Eliminar".
- GIVEN la sección "Festivos regionales" se muestra WHEN hay festivos regionales THEN se agrupan bajo sub-headings por región (ej: "Cataluña", "Andalucía"). Cada festivo muestra los mismos campos que los nacionales.
- GIVEN el usuario HR hace clic en "Añadir festivo nacional" WHEN se abre el Dialog THEN se muestran los mismos campos que SPEC-007 (fecha, nombre, tipo, horario) sin campo de scope (el scope es implícito: nacional para ese mercado).
- GIVEN el usuario HR hace clic en "Añadir festivo regional" WHEN se abre el Dialog THEN se muestran los mismos campos que SPEC-007 más un campo "Región" (Select con las regiones del mercado). El scope será regional para la región seleccionada.
- GIVEN el usuario HR crea un festivo nacional para ES WHEN el festivo se guarda THEN aparece inmediatamente en la lista de festivos de todos los establecimientos de ES (sin que cada tienda tenga que hacer nada).
- GIVEN el usuario HR crea un festivo regional para ES-CT WHEN el festivo se guarda THEN aparece en la lista de festivos de todos los establecimientos de Cataluña.
- GIVEN el usuario HR edita un festivo nacional WHEN guarda los cambios THEN todos los establecimientos que heredan ese festivo ven la versión actualizada (la herencia es en tiempo real, no una copia).
- GIVEN el usuario HR elimina un festivo nacional WHEN confirma la eliminación THEN el festivo desaparece de la lista de todos los establecimientos que lo heredaban (excepto los que tenían un override local, que mantienen su festivo local intacto).

### Validación en vista HR

- GIVEN el usuario HR intenta crear un festivo nacional en una fecha que ya tiene otro festivo nacional WHEN hace clic en "Crear" THEN se muestra error inline: "Ya existe un festivo nacional para el [fecha]: [nombre]".
- GIVEN el usuario HR intenta crear un festivo regional para ES-CT en una fecha que ya tiene un festivo regional para ES-CT WHEN hace clic en "Crear" THEN se muestra error inline: "Ya existe un festivo regional para Cataluña en el [fecha]: [nombre]".
- GIVEN el usuario HR intenta crear un festivo regional en una fecha que ya tiene un festivo nacional WHEN hace clic en "Crear" THEN se permite (un mercado puede tener festivo nacional + festivo regional en la misma fecha para una región, ej: 15/08 es festivo nacional Y festivo de Asunción en varias comunidades, pero solo se cuenta una vez en el establecimiento).

### Empty states y error states en vista HR

- GIVEN no hay festivos nacionales para el año seleccionado WHEN la sección "Festivos nacionales" se renderiza THEN se muestra empty state: icono CalendarOff + "No hay festivos nacionales para [año]" + botón "Añadir festivo nacional" (variant default).
- GIVEN no hay festivos regionales para ninguna región WHEN la sección "Festivos regionales" se renderiza THEN se muestra empty state: "No hay festivos regionales para [año]. Las regiones con festivos propios aparecerán aquí" + botón "Añadir festivo regional".
- GIVEN el fetch de festivos del mercado falla WHEN la página intenta cargar THEN se muestra error state con botón "Reintentar".

### Vista de establecimiento: extensión de SPEC-007

- GIVEN el establecimiento pertenece a un mercado con festivos nacionales definidos WHEN la lista de festivos (SPEC-007) carga THEN los festivos heredados (nacionales y regionales) aparecen junto a los locales, todos ordenados por fecha.
- GIVEN un festivo es heredado (nacional o regional) WHEN se muestra en la lista THEN muestra un Badge adicional: "Nacional" (variant outline, azul) o "Regional" (variant outline, verde), junto al Badge de tipo existente ("Cierre"/"Apertura").
- GIVEN un festivo es local (scope establecimiento) WHEN se muestra en la lista THEN muestra Badge "Local" (variant outline, gris) junto al Badge de tipo. Si además sobreescribe un festivo heredado, muestra tooltip en el Badge: "Sustituye al festivo nacional/regional '[nombre]'".
- GIVEN un festivo es heredado WHEN el usuario con rol `establecimiento` ve las acciones disponibles THEN el DropdownMenu muestra "Personalizar para este establecimiento" en lugar de "Editar" y "Eliminar" (no puede editar ni eliminar festivos heredados).
- GIVEN el usuario hace clic en "Personalizar para este establecimiento" en un festivo heredado WHEN se abre el Dialog THEN se muestran los campos pre-rellenados con los datos del festivo heredado (fecha fija, nombre editable, tipo editable, horario editable). El título del Dialog es "Personalizar festivo".
- GIVEN el usuario guarda la personalización WHEN la operación es exitosa THEN se crea un festivo local que sobreescribe al heredado. El festivo heredado desaparece de la lista (sustituido por el local). Toast: "Festivo personalizado para este establecimiento".
- GIVEN un festivo local sobreescribe uno heredado WHEN el usuario ve las acciones del festivo local THEN el DropdownMenu muestra "Editar", "Restaurar festivo [nacional/regional]" y "Eliminar". "Restaurar" elimina el override local y vuelve a mostrar el heredado.
- GIVEN el usuario hace clic en "Restaurar festivo nacional" WHEN se muestra el AlertDialog THEN dice: "Restaurar festivo nacional", descripción: "Se eliminará la personalización local y se volverá al festivo nacional '[nombre]' ([tipo])." Botones: "Cancelar" / "Restaurar".
- GIVEN el usuario con rol `hr_mercado` accede a la lista de festivos de un establecimiento WHEN la lista carga THEN puede ver y gestionar festivos heredados y locales. Los festivos heredados son editables para HR (desde la vista del mercado, no desde la vista del establecimiento).

### Herencia de horario en festivos multi-nivel

- GIVEN un festivo nacional es de apertura con "Horario habitual del período" WHEN un establecimiento de ese mercado hereda el festivo THEN el horario se resuelve desde el período del establecimiento (no del mercado). Cada establecimiento puede tener períodos con horarios distintos, y la herencia del horario del período es siempre local.
- GIVEN un festivo nacional es de apertura con "Horario especial" (ej: 10:00-18:00) WHEN un establecimiento lo hereda THEN hereda también el horario especial tal cual (10:00-18:00 para todos los establecimientos del mercado).
- GIVEN un festivo nacional es de apertura con "Horario habitual" y el establecimiento A tiene horario de lunes 10:00-22:00 y el establecimiento B tiene 09:00-21:00 WHEN ambos heredan el festivo que cae en lunes THEN el establecimiento A muestra 10:00-22:00 y el B muestra 09:00-21:00 (resolución local).

### Navegación entre vistas

- GIVEN un usuario HR está en la vista de festivos del mercado WHEN quiere ver cómo afecta un festivo nacional a un establecimiento concreto THEN puede navegar al establecimiento desde el panel de establecimientos (SPEC-018) y ver su lista de festivos con los heredados ya visibles.
- GIVEN un usuario con rol `establecimiento` ve un festivo heredado WHEN quiere saber de dónde viene THEN el Badge "Nacional" o "Regional [región]" es suficiente. No necesita navegar a la vista HR.

### Edge cases

- GIVEN un establecimiento no tiene región asignada WHEN se calculan los festivos heredados THEN solo hereda los festivos nacionales del mercado (no festivos regionales, ya que no tiene región).
- GIVEN un establecimiento cambia de región (reasignación administrativa) WHEN la lista de festivos se recalcula THEN deja de ver los festivos regionales de la región anterior y comienza a ver los de la nueva región. Los festivos locales no se alteran.
- GIVEN un establecimiento cambia de mercado WHEN la lista de festivos se recalcula THEN deja de ver los festivos nacionales y regionales del mercado anterior. Los festivos locales se mantienen. Los overrides locales que sobreescribían festivos del mercado anterior se convierten en festivos locales normales (ya no sobreescriben nada).
- GIVEN un festivo nacional y un festivo regional caen en la misma fecha para una región WHEN el establecimiento de esa región ve su lista THEN aparece un solo festivo en la lista (el regional tiene precedencia sobre el nacional si ambos existen, ya que es más específico). Si los dos tienen tipo distinto (uno cierre y otro apertura), el regional prevalece.
- GIVEN un usuario HR crea un festivo nacional para 2027 WHEN el año 2027 de un establecimiento aún no tiene períodos definidos THEN el festivo heredado aparece en la lista igualmente. Si es de apertura con "Horario habitual", la advertencia de SPEC-008 aplica: "La fecha no está cubierta por ningún período".
- GIVEN se ejecuta la copia interanual (SPEC-015) WHEN el establecimiento tenía festivos locales que sobreescribían festivos heredados THEN los overrides locales se copian (con `copied_from_year`). Los festivos heredados del año destino se resuelven de los festivos de mercado/región que existan para ese año.

## UX Design

### Wireframe textual

**Vista HR: Festivos del mercado (pantalla nueva)**

Layout 1 — Estándar.

Zona de título:
- Breadcrumb: Administración > Mercados > [Mercado] > Festivos (si se accede desde admin) o directamente "Festivos del mercado" si se accede desde sidebar operativo.
- Heading "Festivos de [Mercado] — [Año]" (h2).
- Select de año (inline, a la derecha del heading, mismo patrón que la vista de establecimiento).

Sección "Festivos nacionales":
- Sub-heading "Festivos nacionales" (h3) con contador "(N)" muted.
- Botón "Añadir festivo nacional" (Button variant default, icono Plus, small), alineado a la derecha del sub-heading.
- Lista compacta (mismo formato que SPEC-007): fecha, nombre, tipo (Badge), horario si aplica.
- Acciones por fila: DropdownMenu con "Editar", Separator, "Eliminar" (destructive).

Sección "Festivos regionales":
- Sub-heading "Festivos regionales" (h3) con contador "(N)" muted.
- Botón "Añadir festivo regional" (Button variant default, icono Plus, small), alineado a la derecha.
- Agrupados por región con sub-sub-headings (h4, font-medium): "Cataluña (2)", "Andalucía (1)", etc. Solo regiones con festivos definidos aparecen.
- Dentro de cada región: misma lista compacta.
- Acciones por fila: DropdownMenu con "Editar", Separator, "Eliminar" (destructive).

Empty state (si no hay festivos nacionales ni regionales):
- Centrado. Icono CalendarOff. "No hay festivos definidos para [Mercado] en [año]". Dos botones: "Añadir festivo nacional" (variant default) y "Añadir festivo regional" (variant outline).

**Dialog de creación de festivo nacional**

Mismo Dialog que SPEC-007, con estos cambios:
- Título: "Añadir festivo nacional".
- Sin campo de scope (implícito: nacional para el mercado desde el que se accede).
- El texto informativo de "Horario habitual del período" no se muestra (el horario heredado se resuelve por establecimiento, no existe un "horario del mercado"). En su lugar: "Cada establecimiento aplicará el horario de su propio período para este día".

**Dialog de creación de festivo regional**

Mismo Dialog que SPEC-007, con estos cambios:
- Título: "Añadir festivo regional".
- Campo adicional "Región" (Select con las regiones del mercado, requerido) entre el campo de fecha y el campo de nombre.
- El texto informativo de herencia: igual que el nacional ("Cada establecimiento aplicará el horario de su propio período").

**Vista de establecimiento: lista de festivos extendida (modificación de SPEC-007)**

La lista de SPEC-007 se modifica:
- Cada festivo muestra un Badge adicional de scope:
  - "Nacional" (variant outline, color azul) para festivos de mercado.
  - "Regional" (variant outline, color verde) para festivos de región.
  - "Local" (variant outline, color gris) para festivos de establecimiento.
- Festivos heredados: DropdownMenu con "Personalizar para este establecimiento" (única opción).
- Festivos locales que sobreescriben: DropdownMenu con "Editar", "Restaurar festivo [nacional/regional]", Separator, "Eliminar" (destructive).
- Festivos locales sin override: DropdownMenu normal de SPEC-007 ("Editar", "Eliminar").

### Componentes shadcn utilizados

Componentes: Button, Input, Badge, Toast, AlertDialog, DropdownMenu, Skeleton, Dialog, Form, RadioGroup, Select (todos ya presentes). No se requieren componentes adicionales.

### Patrón de interacción

- **La "carga centralizada" es propagación automática, no formulario bulk:** HR crea un festivo a nivel de mercado con el mismo Dialog de SPEC-007. La diferencia es que ese festivo se propaga a cientos de establecimientos. El valor está en la propagación, no en un formulario especial. (Decisión de UX: reutilizar el patrón conocido, sin inventar un flujo de carga masiva que añade complejidad.)
- **Badges de scope para distinguir origen:** el usuario de establecimiento necesita saber de un vistazo qué festivos puede editar (locales) y cuáles son heredados (solo personalizar). Los badges de color resuelven esto. (Decisión no cubierta por el design system: asignación de colores a scopes. Se resuelve con variant outline y colores diferenciados, consistente con la decisión de badges de roles en SPEC-019.)
- **"Personalizar" en lugar de "Editar" para festivos heredados:** el verbo comunica que se crea una copia local que sobreescribe. "Editar" sugeriría que se modifica el festivo original, lo cual no es el caso. (Decisión de UX: verbo específico para la acción de override.)
- **"Restaurar" como opción explícita:** en lugar de que el usuario tenga que eliminar el override y descubrir que el heredado reaparece, la opción "Restaurar festivo nacional" hace explícita la relación. (Decisión de UX: operación inversa nombrada.)
- **Dialog para creación (no Sheet):** los campos son los mismos que SPEC-007 (3-5 campos). El campo adicional de región para festivos regionales llega a 4-5 campos, dentro del límite del Dialog. (Regla: Dialog para 1-4 campos.)
- **Agrupación regional con sub-headings:** en lugar de una tabla plana o tabs por región. Los festivos regionales son pocos (2-5 por región) y las regiones con festivos son pocas (no todas tienen). Sub-headings permiten escanear sin obligar a hacer clic en tabs. (Regla: secciones en scroll para contenido continuo con orden lógico.)

### Comportamiento responsive

- **Mobile (< md):** Vista HR: las secciones se apilan. Las listas son a ancho completo. Los sub-headings regionales se mantienen. Dialog a ancho completo. Vista de establecimiento: los Badges de scope se muestran debajo del nombre del festivo en lugar de en la misma línea.
- **Tablet (md-lg):** Interpolado.
- **Desktop (lg+):** Como descrito.

## Notas técnicas

- Modelo de datos: se extiende la tabla `holidays` con:
  - `scope` (enum: 'market', 'region', 'establishment', default 'establishment'). Reemplaza la semántica implícita de SPEC-007 donde todo era establishment.
  - `market_id` (FK a markets, nullable). Requerido si scope = 'market'.
  - `region_id` (FK a regions, nullable). Requerido si scope = 'region'.
  - `establishment_id` se convierte en nullable. Requerido si scope = 'establishment'.
  - Check constraint: exactamente uno de (market_id, region_id, establishment_id) debe ser no-null, coherente con el scope.
  - La restricción de unicidad se modifica: `UNIQUE(market_id, date)` para scope market, `UNIQUE(region_id, date)` para scope region, `UNIQUE(establishment_id, date)` para scope establishment. Se implementan como partial unique indexes.
- La tabla `holiday_slots` no cambia. Los festivos de scope market/region con horario especial tienen sus propias filas en `holiday_slots`. Los festivos con "Horario habitual" siguen usando `inherit_period_schedule = true`, que se resuelve en tiempo real por establecimiento.
- La resolución de festivos para un establecimiento es una consulta que une: (1) festivos con scope market donde market_id = establecimiento.market_id, (2) festivos con scope region donde region_id = establecimiento.region_id, (3) festivos con scope establishment donde establishment_id = establecimiento.id. Se excluyen los heredados cuya fecha coincide con un festivo local (override).
- Las RLS de SPEC-019 aplican: un usuario con rol `establecimiento` solo ve los festivos que le corresponden (sus locales + los heredados de su mercado/región). Un usuario con rol `hr_mercado` puede crear/editar/eliminar festivos de scope market y region para sus mercados asignados.
- La migración de datos existentes: todos los festivos creados antes de esta spec tienen scope = 'establishment' y su establishment_id existente. No se pierden datos.
- Dependencia upstream: SPEC-007 (registro de festivos), SPEC-008 (herencia de horario), SPEC-018 (estructura organizativa), SPEC-019 (roles y permisos).
- Dependencia downstream: SPEC-015 (copia interanual, debe copiar overrides locales), SPEC-017 (visualización de hora de cierre, aplica igual a festivos heredados).
