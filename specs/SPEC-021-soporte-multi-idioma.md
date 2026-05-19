# SPEC-021 — Soporte multi-idioma

> RF: RF-MPA-003
> Criticidad: Must
> Wave: 4 — Multi-país y carga centralizada

## Descripción

Permite que la interfaz se muestre en el idioma del usuario, cubriendo los cinco idiomas de los mercados operativos: español, italiano, inglés, francés y neerlandés. SPEC-005 ya estableció que todos los textos deben usar claves semánticas (i18n) desde el inicio. Esta spec define la infraestructura de traducción, la cadena de resolución del idioma, qué contenido se traduce y qué no, el formato de fechas y horas por locale, y el selector de idioma en la interfaz.

## Criterios de aceptación

### Idiomas soportados

- GIVEN el sistema está operativo WHEN se consultan los idiomas disponibles THEN existen exactamente 5 idiomas: español (es), italiano (it), inglés (en), francés (fr) y neerlandés (nl).
- GIVEN el sistema arranca WHEN se cargan los ficheros de traducción THEN cada idioma tiene un fichero completo con todas las claves definidas. No hay claves presentes en un idioma y ausentes en otro.

### Cadena de resolución del idioma

- GIVEN un usuario tiene idioma preferido configurado en su perfil WHEN accede a la aplicación THEN la interfaz se muestra en su idioma preferido.
- GIVEN un usuario no tiene idioma preferido configurado WHEN accede a la aplicación THEN la interfaz se muestra en el idioma por defecto del mercado al que pertenece su primer establecimiento o mercado asignado (campo `default_language` de SPEC-018).
- GIVEN un usuario no tiene idioma preferido ni mercado asignado WHEN accede a la aplicación THEN la interfaz se muestra en español (es) como fallback global.
- GIVEN un usuario cambia su idioma preferido WHEN el cambio se aplica THEN la interfaz se actualiza inmediatamente sin necesidad de recargar la página.

### Selector de idioma

- GIVEN un usuario autenticado WHEN accede a cualquier página THEN existe un selector de idioma accesible desde el top bar (área de acciones globales, junto al avatar).
- GIVEN el usuario hace clic en el selector de idioma WHEN se despliega el DropdownMenu THEN muestra las 5 opciones con el nombre del idioma en su propio idioma: "Español", "Italiano", "English", "Français", "Nederlands". El idioma activo tiene un indicador visual (check).
- GIVEN el usuario selecciona un idioma diferente WHEN la selección se confirma THEN el idioma se guarda en el perfil del usuario (persistente entre sesiones), la interfaz se actualiza y se muestra Toast en el nuevo idioma: equivalente a "Idioma cambiado a [idioma]".

### Contenido traducido (interfaz)

- GIVEN la interfaz se muestra en un idioma WHEN el usuario navega por cualquier pantalla THEN todos los textos estáticos están traducidos: labels de campos, placeholders, mensajes de error y validación, tooltips, textos de empty states y error states, textos de botones y acciones, títulos de páginas y secciones, opciones de Select y RadioGroup, textos de Badges de estado, textos de AlertDialog (título, descripción, botones), textos de Toast.
- GIVEN la interfaz se muestra en italiano WHEN el usuario ve un mensaje de error THEN el mensaje aparece en italiano (ej: "Il nome è obbligatorio"), no en español ni en inglés.
- GIVEN una clave de traducción no existe en el idioma activo WHEN se intenta renderizar THEN se muestra el texto del idioma fallback (español) en lugar de la clave técnica. Nunca se muestra al usuario una clave como "label.holiday.open".

### Contenido NO traducido (datos de usuario)

- GIVEN un usuario español crea un festivo llamado "Nochebuena" WHEN un usuario italiano ve ese festivo THEN el nombre sigue siendo "Nochebuena". Los datos introducidos por usuarios no se traducen automáticamente.
- GIVEN un usuario HR crea un festivo nacional para España con nombre "Navidad" WHEN un usuario de la misma tienda con interfaz en inglés ve ese festivo THEN el nombre es "Navidad" (dato del usuario), pero los labels de la interfaz ("Festivo de cierre", "Horario", etc.) aparecen en inglés.
- GIVEN un período tiene nombre "Horario de verano" WHEN se muestra a un usuario con interfaz en francés THEN el nombre del período es "Horario de verano" (dato del usuario). El label "Período" aparece como "Période".
- GIVEN un establecimiento tiene nombre "Tienda Centro Madrid" WHEN se muestra en la interfaz en neerlandés THEN el nombre es "Tienda Centro Madrid". Los nombres de establecimientos, mercados y regiones son datos de configuración, no se traducen por idioma de interfaz.

### Formato de fechas y horas

- GIVEN el idioma activo es español WHEN se muestra una fecha THEN usa el formato "14 may 2026" o "14/05/2026" (locale es-ES).
- GIVEN el idioma activo es inglés WHEN se muestra una fecha THEN usa el formato "14 May 2026" o "14/05/2026" (locale en-GB, no en-US, ya que los mercados son europeos).
- GIVEN el idioma activo es cualquiera WHEN se muestra una hora THEN usa formato 24h (ej: "18:00", no "6:00 PM"). El formato 24h es estándar en todos los mercados europeos y no varía por idioma.
- GIVEN el idioma activo es cualquiera WHEN se muestra un día de la semana THEN usa el nombre localizado: "lunes"/"lunedì"/"Monday"/"lundi"/"maandag".
- GIVEN el idioma activo es cualquiera WHEN se muestra un nombre de mes THEN usa el nombre localizado: "enero"/"gennaio"/"January"/"janvier"/"januari".

### Glosario multi-idioma (extensión de SPEC-005)

- GIVEN el glosario de SPEC-005 define términos oficiales en español WHEN se traduce a otro idioma THEN cada término tiene su equivalente oficial en ese idioma, manteniendo el mismo principio de lenguaje de negocio (no tecnicismos). Ejemplos:
  - "Festivo de apertura" → "Festivo di apertura" (it), "Opening holiday" (en), "Jour férié d'ouverture" (fr), "Feestdag met opening" (nl).
  - "Cierre temporal" → "Chiusura temporanea" (it), "Temporary closure" (en), "Fermeture temporaire" (fr), "Tijdelijke sluiting" (nl).
  - "Período" → "Periodo" (it), "Period" (en), "Période" (fr), "Periode" (nl).
  - "Cambio puntual de horario" → "Modifica puntuale dell'orario" (it), "One-off schedule change" (en), "Changement ponctuel d'horaire" (fr), "Eenmalige roosterwijziging" (nl).

### Edge cases

- GIVEN un usuario con idioma neerlandés accede a un establecimiento español WHEN ve los festivos THEN la interfaz está en neerlandés pero los nombres de festivos están en español (son datos del usuario). No hay conflicto: el idioma de la interfaz es independiente del idioma de los datos.
- GIVEN se añade un nuevo texto a la interfaz (por desarrollo futuro) WHEN el texto solo tiene traducción en español THEN los otros 4 idiomas muestran el texto en español (fallback), no una clave técnica. Este estado es transitorio hasta que se complete la traducción.
- GIVEN un usuario cambia de idioma de español a inglés WHEN tiene un Dialog abierto THEN el Dialog se actualiza al nuevo idioma sin cerrarse. Los datos pre-rellenados (que son datos de usuario) no cambian.
- GIVEN el navegador del usuario tiene locale "fr-BE" (francés de Bélgica) WHEN el sistema resuelve el idioma THEN no usa el locale del navegador. El idioma se resuelve exclusivamente por la cadena: preferencia del usuario → default del mercado → fallback español.

## UX Design

### Wireframe textual

Esta spec no tiene pantalla nueva. Modifica el top bar (añade selector de idioma) y afecta transversalmente a todos los textos de la aplicación.

**Selector de idioma en el top bar:**
- Ubicación: área de acciones globales, a la izquierda del avatar del usuario.
- Componente: Button (variant ghost, solo icono Globe de Lucide, 20px) que abre un DropdownMenu.
- DropdownMenu con 5 items:
  - "Español" (con check si activo)
  - "Italiano"
  - "English"
  - "Français"
  - "Nederlands"
- El item activo muestra icono Check a la izquierda.

**Perfil del usuario (extensión de SPEC-019):**
- El campo "Idioma preferido" se añade al Sheet de edición de usuario en el panel de administración. Select con las 5 opciones. Opcional (si vacío, se usa el default del mercado).

### Componentes shadcn utilizados

Componentes: Button, DropdownMenu, Select, Toast (todos ya presentes). No se requieren componentes adicionales.

### Patrón de interacción

- **Selector en top bar (no en settings):** el cambio de idioma es una acción frecuente en un entorno multi-país donde usuarios de diferentes mercados colaboran. Ponerlo en el top bar lo hace accesible sin navegar a settings. (Decisión de UX: accesibilidad inmediata para acción frecuente en contexto multi-idioma.)
- **Icono Globe sin texto:** en el top bar el espacio es limitado. El icono Globe es universalmente reconocido para selección de idioma. Al abrir el dropdown, los nombres de idioma proporcionan claridad. (Regla: icon-only en acciones obvias + aria-label.)
- **Nombre del idioma en su propio idioma:** "Nederlands" en lugar de "Neerlandés" o "Dutch". El usuario que busca su idioma lo reconoce mejor en su propia lengua. Convención estándar en selectores de idioma. (Decisión de UX: endónimos, no exónimos.)
- **Cambio inmediato sin recarga:** el usuario ve el resultado instantáneamente. No necesita confirmar ni recargar. (Decisión de UX: feedback inmediato para cambio de preferencia.)

### Comportamiento responsive

- **Mobile (< md):** El selector de idioma se mueve al DropdownMenu de acciones del top bar (junto a otras acciones colapsadas). No ocupa espacio propio.
- **Tablet (md-lg):** Icono Globe visible en el top bar.
- **Desktop (lg+):** Como descrito.

## Notas técnicas

- Estructura de ficheros de traducción: un fichero JSON por idioma, con namespaces por dominio funcional. Ejemplo: `locales/es/common.json`, `locales/es/holidays.json`, `locales/it/common.json`, etc. Los namespaces sugeridos: `common` (textos transversales), `periods` (períodos y franjas), `holidays` (festivos), `overrides` (cambios puntuales), `closures` (cierres temporales), `calendar` (calendario anual), `admin` (panel de administración), `validation` (mensajes de error).
- Las claves usan formato semántico con dot notation: `holidays.type.opening`, `validation.required.name`, `calendar.stats.uncovered_days`. Nunca claves basadas en el texto literal.
- El campo `preferred_language` se añade a la tabla `user_profiles` (SPEC-019): tipo enum ('es', 'it', 'en', 'fr', 'nl'), nullable. Si null, se resuelve por mercado.
- El formato de fecha/hora se delega al API de Intl del navegador (`Intl.DateTimeFormat`, `Intl.NumberFormat`) con el locale correspondiente al idioma activo. No se implementan formateadores custom.
- La validación de completitud de traducciones (que todas las claves existan en los 5 idiomas) se puede automatizar con un script de CI que compare las claves de cada fichero contra el fichero de referencia (español).
- Dependencia upstream: SPEC-005 (glosario y claves semánticas), SPEC-018 (campo `default_language` en mercados), SPEC-019 (tabla `user_profiles` para `preferred_language`).
- Dependencia downstream: ninguna directa. Es una spec transversal que mejora todas las specs existentes.
