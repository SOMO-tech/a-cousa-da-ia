# SPEC-018 — Configuración por mercado/país

> RF: RF-MPA-001
> Criticidad: Must
> Wave: 4 — Multi-país y carga centralizada

## Descripción

Define la estructura organizativa que permite al sistema soportar reglas, festivos y modelos operativos diferentes por país. El research identifica que las casuísticas varían radicalmente entre mercados: normativa provincial en España, collective agreements en Netherlands, bank holidays en UK/Ireland, shopping centres que imponen horarios en varios países, y observancia religiosa en Italia. Esta spec establece la jerarquía país → región → establecimiento, qué se configura a cada nivel, y cómo la estructura condiciona las funcionalidades ya definidas en specs anteriores.

La estructura organizativa se almacena en base de datos propia de Aurora, no se hereda de un sistema maestro externo. Esto garantiza independencia operativa y permite que Aurora evolucione sin depender del ciclo de vida de HRS.

## Criterios de aceptación

### Modelo organizativo

- GIVEN el sistema se inicializa WHEN se consulta la estructura organizativa THEN existe una jerarquía de tres niveles: Mercado (país), Región (subdivisión administrativa) y Establecimiento.
- GIVEN un mercado existe WHEN se consultan sus datos THEN tiene: código ISO 3166-1 alpha-2 (ej: "ES", "IT", "NL", "GB", "FR", "IE"), nombre localizado, idioma por defecto, zona horaria por defecto y estado (activo/inactivo).
- GIVEN una región existe WHEN se consultan sus datos THEN tiene: código (libre, ej: "ES-AN" para Andalucía, "IT-LAZ" para Lazio), nombre localizado, mercado al que pertenece, y zona horaria si difiere de la del mercado.
- GIVEN un establecimiento existe WHEN se consultan sus datos THEN tiene: código interno, nombre, dirección, mercado, región, zona horaria (heredada de la región o propia si difiere), tipología (propia / franquicia) y estado (activo/inactivo).

### Mercados iniciales

- GIVEN el sistema se despliega por primera vez WHEN se consultan los mercados THEN existen al menos los siguientes mercados pre-configurados:
  - ES (España), idioma por defecto: es, zona horaria: Europe/Madrid.
  - IT (Italia), idioma por defecto: it, zona horaria: Europe/Rome.
  - NL (Netherlands), idioma por defecto: nl, zona horaria: Europe/Amsterdam.
  - GB (United Kingdom), idioma por defecto: en, zona horaria: Europe/London.
  - IE (Ireland), idioma por defecto: en, zona horaria: Europe/Dublin.
  - FR (Francia), idioma por defecto: fr, zona horaria: Europe/Paris.

### Regiones por mercado

- GIVEN el mercado ES existe WHEN se consultan sus regiones THEN existen regiones correspondientes a las comunidades autónomas (ej: Andalucía, Cataluña, Madrid, Galicia, etc.). Cada región puede tener festivos propios (SPEC-019/RF-MPA-002).
- GIVEN el mercado IT existe WHEN se consultan sus regiones THEN existen regiones correspondientes a las regiones administrativas (ej: Lazio, Lombardia, Veneto, etc.).
- GIVEN el mercado GB existe WHEN se consultan sus regiones THEN existen regiones correspondientes a las naciones constituyentes (England, Scotland, Wales, Northern Ireland), ya que los bank holidays difieren entre ellas.
- GIVEN el mercado FR existe WHEN se consultan sus regiones THEN existen regiones que reflejan la estructura administrativa relevante para festivos (Alsace, Moselle y resto de Francia).
- GIVEN el mercado NL existe WHEN se consultan sus regiones THEN existe al menos una región por defecto (Netherlands no tiene festivos sub-nacionales significativos, pero la estructura se mantiene por consistencia).
- GIVEN el mercado IE existe WHEN se consultan sus regiones THEN existe al menos una región por defecto.

### Seed de datos iniciales

La siguiente seed se ejecuta como migración de base de datos en el primer despliegue. Define los mercados y regiones necesarios para operar. Las regiones se incluyen solo cuando los festivos o la normativa de apertura varían a nivel sub-nacional.

#### Mercado ES (España)

Idioma: es. Zona horaria: Europe/Madrid (excepto Canarias). 19 regiones. Cada comunidad autónoma tiene 2 festivos propios además de los nacionales, más festivos locales por municipio.

| Código | Nombre |  Notas |
|--------|--------|--------|
| ES-AN | Andalucía | |
| ES-AR | Aragón | |
| ES-AS | Asturias | |
| ES-IB | Islas Baleares | |
| ES-CN | Canarias | Zona horaria: Atlantic/Canary |
| ES-CB | Cantabria | |
| ES-CL | Castilla y León | |
| ES-CM | Castilla-La Mancha | |
| ES-CT | Cataluña | |
| ES-VC | Comunidad Valenciana | |
| ES-EX | Extremadura | |
| ES-GA | Galicia | |
| ES-MD | Madrid | |
| ES-MC | Murcia | |
| ES-NC | Navarra | |
| ES-PV | País Vasco | |
| ES-RI | La Rioja | |
| ES-CE | Ceuta | |
| ES-ML | Melilla | |

#### Mercado IT (Italia)

Idioma: it. Zona horaria: Europe/Rome. 20 regiones. En Italia los festivos nacionales son uniformes, pero cada ciudad tiene el festivo del santo patrono (gestión a nivel de establecimiento, no de región). Las regiones se incluyen porque la normativa de apertura dominical varía regionalmente.

| Código | Nombre | Notas |
|--------|--------|-------|
| IT-PIE | Piemonte | |
| IT-VDA | Valle d'Aosta | |
| IT-LIG | Liguria | |
| IT-LOM | Lombardia | |
| IT-TAA | Trentino-Alto Adige | |
| IT-VEN | Veneto | |
| IT-FVG | Friuli Venezia Giulia | |
| IT-EMR | Emilia-Romagna | |
| IT-TOS | Toscana | |
| IT-UMB | Umbria | |
| IT-MAR | Marche | |
| IT-LAZ | Lazio | |
| IT-ABR | Abruzzo | |
| IT-MOL | Molise | |
| IT-CAM | Campania | |
| IT-PUG | Puglia | |
| IT-BAS | Basilicata | |
| IT-CAL | Calabria | |
| IT-SIC | Sicilia | |
| IT-SAR | Sardegna | |

#### Mercado GB (United Kingdom)

Idioma: en. Zona horaria: Europe/London. 4 regiones. Los bank holidays difieren entre las cuatro naciones: Scotland tiene bank holidays propios (ej: St Andrew's Day), Northern Ireland tiene St Patrick's Day y Battle of the Boyne.

| Código | Nombre | Notas |
|--------|--------|-------|
| GB-ENG | England | |
| GB-SCT | Scotland | |
| GB-WLS | Wales | |
| GB-NIR | Northern Ireland | |

#### Mercado IE (Ireland)

Idioma: en. Zona horaria: Europe/Dublin. 1 región. Los public holidays son uniformes a nivel nacional. Se mantiene una región por consistencia estructural.

| Código | Nombre | Notas |
|--------|--------|-------|
| IE-NAT | Nacional | Región por defecto |

#### Mercado FR (Francia)

Idioma: fr. Zona horaria: Europe/Paris. 3 regiones. Francia tiene festivos nacionales uniformes excepto en Alsace y Moselle (departamentos del antiguo concordato), que tienen 2 festivos adicionales (Viernes Santo y San Esteban/26 de diciembre). El resto del territorio se agrupa en una región por defecto.

| Código | Nombre | Notas |
|--------|--------|-------|
| FR-ALS | Alsace | Festivos adicionales: Viernes Santo, San Esteban |
| FR-MOS | Moselle | Festivos adicionales: Viernes Santo, San Esteban |
| FR-NAT | Resto de Francia | Región por defecto para establecimientos fuera de Alsace-Moselle |

#### Mercado NL (Netherlands)

Idioma: nl. Zona horaria: Europe/Amsterdam. 1 región. Los festivos son uniformes a nivel nacional. La normativa de apertura dominical varía por municipio, pero eso se gestiona a nivel de establecimiento, no de región.

| Código | Nombre | Notas |
|--------|--------|-------|
| NL-NAT | Nacional | Región por defecto |

#### Resumen de seed

| Mercado | Código | Idioma | Zona horaria | Regiones |
|---------|--------|--------|-------------|----------|
| España | ES | es | Europe/Madrid | 19 |
| Italia | IT | it | Europe/Rome | 20 |
| United Kingdom | GB | en | Europe/London | 4 |
| Ireland | IE | en | Europe/Dublin | 1 |
| Francia | FR | fr | Europe/Paris | 3 |
| Netherlands | NL | nl | Europe/Amsterdam | 1 |
| **Total** | **6** | **5 idiomas** | **6 zonas** | **48** |

### Gestión de mercados (administración)

- GIVEN un usuario con rol de administrador global WHEN accede al panel de administración THEN puede ver la lista de mercados, crear nuevos mercados, editar mercados existentes y desactivar mercados.
- GIVEN un administrador crea un nuevo mercado WHEN rellena los datos THEN debe indicar: código ISO, nombre, idioma por defecto, zona horaria por defecto. El mercado se crea sin regiones (se añaden después).
- GIVEN un administrador desactiva un mercado WHEN confirma la acción THEN los establecimientos de ese mercado dejan de ser accesibles para usuarios operativos, pero los datos se conservan.

### Gestión de regiones (administración)

- GIVEN un usuario con rol de administrador global o administrador de mercado WHEN accede a un mercado THEN puede ver la lista de regiones, crear nuevas regiones, editar regiones existentes y eliminar regiones vacías (sin establecimientos).
- GIVEN un administrador crea una nueva región WHEN rellena los datos THEN debe indicar: código, nombre y opcionalmente zona horaria si difiere de la del mercado.
- GIVEN un administrador intenta eliminar una región con establecimientos asignados WHEN confirma la acción THEN se muestra error: "No se puede eliminar la región '[nombre]' porque tiene [N] establecimiento(s) asignado(s). Reasigna los establecimientos primero".

### Gestión de establecimientos (administración)

- GIVEN un usuario con rol de administrador WHEN accede a la gestión de establecimientos THEN puede ver la lista de establecimientos filtrada por mercado y región, crear nuevos establecimientos, editar establecimientos existentes y desactivar establecimientos.
- GIVEN un administrador crea un establecimiento WHEN rellena los datos THEN debe indicar: código interno, nombre, dirección, mercado (Select), región (Select filtrado por mercado seleccionado), tipología (propia/franquicia) y opcionalmente zona horaria propia.
- GIVEN un administrador edita un establecimiento WHEN cambia el mercado o la región THEN se muestra advertencia: "Cambiar el mercado/región puede afectar a los festivos heredados. ¿Continuar?" (no bloqueante).

### Filtrado por mercado en la interfaz operativa

- GIVEN un usuario operativo (responsable de tienda) WHEN accede a la aplicación THEN solo ve los establecimientos asignados a su perfil. No tiene visibilidad sobre la estructura organizativa global.
- GIVEN un usuario HR WHEN accede a la aplicación THEN ve todos los establecimientos de su mercado (o mercados, si gestiona varios), agrupados por región.
- GIVEN un usuario de soporte (Large Format) WHEN accede a la aplicación THEN ve todos los establecimientos de todos los mercados, con filtros por mercado y región.

### Impacto en funcionalidades existentes

- GIVEN un establecimiento pertenece a un mercado WHEN el usuario gestiona su horario comercial THEN todas las funcionalidades de SPEC-001 a SPEC-017 funcionan exactamente igual. El mercado no altera la mecánica de períodos, festivos ni excepciones.
- GIVEN un mercado tiene festivos nacionales cargados centralizadamente (RF-FES-003) WHEN un establecimiento de ese mercado accede a su lista de festivos THEN ve los festivos nacionales del mercado más los festivos regionales y locales que le apliquen (detalle en SPEC-019/RF-MPA-002).
- GIVEN un establecimiento tiene zona horaria propia que difiere de la del mercado WHEN se muestran horarios THEN todas las horas se interpretan en la zona horaria del establecimiento, no en la del mercado.

### Edge cases

- GIVEN un establecimiento no tiene región asignada WHEN se consultan sus datos THEN pertenece directamente al mercado sin región intermedia. Esto es válido para mercados con estructura regional simple.
- GIVEN se añade un nuevo mercado después del despliegue inicial WHEN se crea THEN no tiene establecimientos ni regiones. El administrador los configura progresivamente.
- GIVEN un establecimiento cambia de mercado (ej: reestructuración organizativa) WHEN se reasigna THEN los períodos y horarios existentes se mantienen, pero los festivos heredados del mercado anterior se desvinculan y los del nuevo mercado se vinculan.
- GIVEN dos mercados comparten una zona horaria (ej: ES y FR, ambos Europe/Madrid y Europe/Paris respectivamente, con 0h de diferencia en horario de invierno) WHEN se gestionan horarios THEN cada mercado es independiente. La zona horaria se usa para resolución de "hoy" y "ahora", no para conversión entre mercados.

## UX Design

### Wireframe textual

**Panel de administración: Mercados (pantalla nueva, solo visible para administradores)**

Layout de página con sidebar de navegación administrativa:
- Sidebar: Mercados, Regiones, Establecimientos, Usuarios.
- Contenido principal:

Zona de título:
- Heading "Mercados" (h2).
- Botón "Nuevo mercado" (Button variant default, icono Plus).

Lista de mercados:
- Tabla con columnas: Bandera (emoji del país), Código, Nombre, Idioma, Zona horaria, Regiones (contador), Establecimientos (contador), Estado (Badge "Activo" / "Inactivo").
- Acciones por fila: DropdownMenu con "Editar", "Ver regiones", "Desactivar" (o "Activar").

**Panel de administración: Regiones (dentro de un mercado)**

Zona de título:
- Heading "Regiones de [mercado]" (h2).
- Breadcrumb: Mercados > [Mercado] > Regiones.
- Botón "Nueva región" (Button variant default, icono Plus).

Lista de regiones:
- Tabla con columnas: Código, Nombre, Zona horaria (si propia, sino "Hereda del mercado"), Establecimientos (contador).
- Acciones por fila: DropdownMenu con "Editar", "Eliminar" (solo si 0 establecimientos).

**Panel de administración: Establecimientos**

Zona de título:
- Heading "Establecimientos" (h2).
- Filtros: Select de mercado, Select de región (filtrado por mercado), Select de tipología (Todos / Propios / Franquicias), Select de estado (Activos / Inactivos / Todos).
- Botón "Nuevo establecimiento" (Button variant default, icono Plus).

Lista de establecimientos:
- Tabla con columnas: Código, Nombre, Mercado, Región, Tipología (Badge "Propia" / "Franquicia"), Estado.
- Acciones por fila: DropdownMenu con "Editar", "Ver horario", "Desactivar".

**Dialogs de creación/edición:**
- Mercado: Dialog con campos Código ISO, Nombre, Idioma por defecto (Select), Zona horaria (Select). 4 campos.
- Región: Dialog con campos Código, Nombre, Zona horaria (opcional, Select). 2-3 campos.
- Establecimiento: Dialog con campos Código, Nombre, Dirección, Mercado (Select), Región (Select filtrado), Tipología (RadioGroup), Zona horaria (opcional). 5-7 campos.

### Componentes shadcn utilizados

Componentes: Table (nuevo en esta spec, para listados administrativos con columnas), Select (nuevo, para filtros y formularios con listas predefinidas), Breadcrumb (nuevo, para navegación jerárquica en administración). Dialog, Button, Badge, DropdownMenu, RadioGroup, Toast ya presentes.

### Patrón de interacción

- **Tabla para listados administrativos:** los administradores gestionan datos tabulares (mercados, regiones, establecimientos). Las tablas permiten ordenar, escanear y actuar sobre múltiples registros. (Regla: Table para datos con múltiples atributos comparables.)
- **Select con filtrado dependiente** (mercado → región): al seleccionar un mercado, las regiones disponibles se filtran. Reduce la posibilidad de error y guía al usuario. (Decisión de UX: cascading selects para jerarquías.)
- **Breadcrumb para navegación jerárquica:** en el panel de administración, el usuario navega Mercados → Regiones → Establecimientos. El Breadcrumb orienta y permite volver a niveles superiores. (Regla: Breadcrumb para navegación con profundidad > 2.)
- **Dialog para creación/edición:** coherente con el patrón del resto de la aplicación. Los formularios administrativos tienen pocos campos (4-7). (Regla: Dialog para 1-7 campos.)
- **Desactivación en lugar de eliminación:** los mercados y establecimientos no se eliminan, se desactivan. Los datos históricos se preservan. (Decisión de UX: soft-delete para entidades con datos dependientes.)

### Comportamiento responsive

- **Mobile (< md):** El panel de administración no está optimizado para mobile (es una función de back-office que se usa en desktop). Si se accede, las tablas muestran scroll horizontal. Los filtros se apilan verticalmente.
- **Tablet (md-lg):** Funcional con tablas a ancho completo.
- **Desktop (lg+):** Layout con sidebar + contenido como descrito.

## Notas técnicas

- Modelo de datos:
  - Tabla `markets`: `id`, `code` (ISO 3166-1 alpha-2, unique), `name`, `default_language` (enum: 'es', 'it', 'en', 'fr', 'nl'), `default_timezone`, `active` (boolean), `created_at`, `updated_at`.
  - Tabla `regions`: `id`, `market_id` (FK), `code` (unique dentro del mercado), `name`, `timezone` (nullable, hereda del mercado si null), `created_at`, `updated_at`.
  - Tabla `establishments`: `id`, `code` (unique), `name`, `address`, `market_id` (FK), `region_id` (FK, nullable), `type` (enum: 'owned', 'franchise'), `timezone` (nullable, hereda de región/mercado si null), `active` (boolean), `created_at`, `updated_at`.
- La resolución de zona horaria sigue la cadena: establecimiento → región → mercado. Se usa la primera que tenga valor.
- Las tablas de specs anteriores (`periods`, `holidays`, `schedule_overrides`, `temporary_closures`) ya tienen `establishment_id`. No necesitan modificación.
- Los festivos nacionales y regionales (SPEC-019) se vincularán a `market_id` y `region_id` respectivamente. Los festivos locales (de establecimiento) mantienen su `establishment_id` actual.
- La seed de datos iniciales (mercados y regiones) se ejecuta como migración de base de datos. Contiene 6 mercados y 48 regiones. Los establecimientos no forman parte de la seed (se crean operativamente o se importan desde HRS).
- Componentes nuevos: Table, Select, Breadcrumb de shadcn. Se añaden al inventario de componentes del proyecto.
- Dependencia upstream: ninguna directa (es la base de la estructura organizativa).
- Dependencia downstream: SPEC-019 (festivos locales/regionales), SPEC-020 (soporte multi-idioma), SPEC-021 (carga centralizada de festivos), RF-ADM-001 (panel de monitorización usará esta estructura para filtrar).
