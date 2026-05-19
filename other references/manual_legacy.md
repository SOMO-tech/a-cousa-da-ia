# Manual del Sistema Heredado de Horarios Comerciales (M7)

Documento de formación interno para el uso de la aplicación de Horarios Comerciales dentro de HRS.

## Índice de contenidos

1. Períodos
2. Franjas horarias
3. Excepciones
4. Ejemplos prácticos
5. Festivos
6. Vista de calendario
7. Errores frecuentes

## Descripción general

El Horario Comercial es una funcionalidad integrada en HRS (accesible desde Aplicaciones Frecuentes) que permite registrar y consultar los horarios de apertura de cada establecimiento. Se estructura en períodos (tramos del año con un mismo horario), franjas horarias (horas de apertura para cada día de la semana), festivos (días de apertura o cierre especial) y excepciones (días puntuales con horario diferente al del período en que se encuentran).

Conceptos clave:

- **Período**: intervalo temporal dentro del año durante el cual el horario comercial permanece invariable. Por ejemplo, si un establecimiento tiene un horario distinto en verano, será necesario crear más de un período para ese año.
- **Franja horaria**: bloque de horas en el que el establecimiento permanece abierto para un día de la semana concreto. Si la tienda cierra a mediodía, habrá dos franjas para ese día.
- **Excepción**: día concreto dentro de un período que tiene un horario diferente al estándar de dicho período.
- **Festivo**: día señalado que debe registrarse en la pestaña específica de Gestión de Festivos.
- **Vista de calendario**: pantalla que muestra en formato anual los días con período asignado, los que carecen de él, los domingos, festivos y excepciones de un solo vistazo.

## 1. Períodos

### Crear un nuevo horario

Para dar de alta un horario comercial nuevo se utiliza el botón "Nuevo Horario". Si el establecimiento ya cuenta con un horario del año precedente, existe un botón adicional que permite copiar el horario anterior de forma exacta (sin incluir festivos ni excepciones). Esta copia sirve de punto de partida y agiliza el proceso.

### Añadir un período

Se accede mediante el botón "Nuevo período", se introduce el nombre deseado, la fecha de inicio y la fecha de finalización, y se confirma.

Reglas importantes:

- Para que el horario anual esté completo, los períodos deben cubrir desde el 1 de enero hasta el 31 de diciembre sin interrupciones. Si quedan huecos, el sistema mostrará una advertencia y no permitirá crear horarios para años posteriores.
- Solo es necesario crear más de un período cuando existan horarios diferentes a lo largo del año. Si el horario se mantiene constante todo el año, basta con un único período.
- Aunque se haya utilizado la función de copia del año anterior, es imprescindible registrar las franjas horarias del período. Sin franjas, el período no se podrá guardar.

### Modificar un período

Se selecciona el período que se desea cambiar, se pulsa el botón de edición, se introducen los nuevos valores y se guardan los cambios.

### Eliminar un período

Al seleccionar un período y pulsar el botón de eliminación, el período aparecerá tachado en pantalla. Es necesario confirmar guardando para que la eliminación surta efecto.

### Período de cierre

Cuando un establecimiento vaya a permanecer cerrado durante un tramo de tiempo por motivos como reformas o similares, se debe registrar un período de cierre. Para ello, al crear el período hay que activar la casilla de verificación "CIERRE".

El período marcado como cierre se muestra con un rayado visual en la pantalla principal. En la vista de calendario, los días correspondientes a ese período aparecen tachados para identificarlos rápidamente.

## 2. Franjas horarias

### Crear una nueva franja

Una vez creado un período, es obligatorio definir sus franjas horarias. Se selecciona el período, se accede a la creación de franjas, se elige un día de la semana (por ejemplo, lunes) y se completan la hora y los minutos de inicio y fin. Si el horario de apertura es partido (con cierre a mediodía), se deben crear dos franjas para ese día. Se confirma pulsando el botón de guardar.

La grabación de al menos una franja es requisito indispensable para poder guardar el período.

### Copiar y pegar franjas

Una vez definida la franja de un día, se puede replicar en el resto de días de la semana cuando el horario sea idéntico:

1. Se selecciona la franja del día origen y se activa el botón de copiar.
2. Al hacerlo, aparecen botones de pegar sobre el resto de días de la semana.
3. Se pulsa pegar en cada día que deba tener el mismo horario.
4. Finalmente, es imprescindible pulsar GUARDAR para confirmar tanto las franjas como el período. Si no se guarda, el período completo se perderá.

### Modificar una franja

Se selecciona la franja existente, se pulsa el botón de edición, se introduce el nuevo horario y se guardan los cambios.

### Eliminar una franja

Se selecciona la franja y se pulsa el botón de eliminación. Es fundamental confirmar los cambios pulsando guardar para que la eliminación se aplique.

## 3. Excepciones

Una excepción es cualquier día cuyo horario difiere del estándar del período al que pertenece. Existen tres tipos:

1. **Excepción de modificación de horario**: el establecimiento opera con un horario diferente al habitual por cualquier motivo que no sea festivo. Por ejemplo, en Nochevieja se cierra antes de lo normal, por lo que se registra un horario distinto.
2. **Excepción de cierre**: el establecimiento cierra un día fuera de lo habitual sin que sea festivo ni por reforma. Es una situación atípica.
3. **Excepción de apertura**: el establecimiento abre un día en que normalmente permanece cerrado. Por ejemplo, una tienda que habitualmente opera de lunes a sábado y abre un domingo de forma puntual.

### Crear una excepción

Dentro de cada período definido existe un botón para gestionar excepciones. Al pulsarlo se abre una ventana donde se puede añadir la nueva excepción, que se guarda confirmando o se descarta volviendo a la pantalla principal.

Una vez creada la excepción, se muestra un indicador en la pantalla del período. Al seleccionar dicha excepción se puede definir una franja horaria específica, del mismo modo que se hace con los períodos.

Para las excepciones de apertura, el sistema asigna por defecto el horario del período, aunque es posible modificarlo si es necesario. Para las excepciones de cierre, el horario se elimina automáticamente ya que el establecimiento no abre.

### Modificar o eliminar una excepción

Se selecciona la excepción y se pulsa el botón de edición. Se abre una ventana donde se pueden cambiar los valores y guardar, o bien eliminar la excepción directamente. Tras la operación, se vuelve a la pantalla principal de Horario Comercial.

## 4. Ejemplos prácticos

**Ejemplo 1**: Establecimiento que abre todo el año de lunes a sábado de 10:00 a 21:00, salvo durante rebajas, domingos especiales y días con cambio de horario.
- Se registra un único período para todo el año.
- Los domingos de apertura se graban como excepciones de apertura (ya que normalmente no se abre los domingos).
- Los días con horario diferente, como el 24 de diciembre, se graban como excepciones de modificación de horario.

**Ejemplo 2**: Establecimiento que en invierno cierra a las 20:30 y en verano, al estar en zona costera, cierra a las 21:30.
- Se crean dos períodos: uno con cierre a las 20:30 y otro con cierre a las 21:30.

**Ejemplo 3**: Establecimiento que abre el primer domingo de cada mes durante todo el año.
- Se registra un único período con doce excepciones de apertura, una por cada primer domingo del mes.

**Nota importante**: La hora de fin de franja no debe incluir el tiempo de recogida posterior al cierre. Si el establecimiento cierra al público a las 21:00 pero tiene 30 minutos de recogida, la hora fin de franja debe ser las 21:00.

## 5. Festivos

Los festivos se registran en la pestaña "Gestión de Festivos" del menú superior de la aplicación.

Procedimiento:
1. Se pulsa en el calendario sobre el día festivo y se abre la pantalla de gestión.
2. Se asigna un nombre al festivo, se selecciona la fecha y se indica si es de apertura o de cierre.
3. Se guarda y se cierra la ventana.

Al regresar a la pantalla de gestión en formato calendario, el día aparecerá marcado en rojo para identificarlo como festivo.

**Requisito**: Para poder registrar un festivo, debe existir previamente un período (con franja horaria definida) en la fecha de ese festivo. En caso contrario, el sistema no lo guardará.

Cuando el festivo es de apertura, se le asigna automáticamente el horario del período correspondiente. Si el horario del festivo difiere del estándar, deberá modificarse manualmente.

## 6. Vista de calendario

La aplicación cuenta con una pestaña de visualización en formato de calendario anual donde se muestran todos los períodos, excepciones y festivos registrados.

Leyenda de colores y marcas:

- **Días con período**: marcados con color de fondo
- **Días sin período**: sin color, indicando huecos
- **Domingos**: señalados con marca especial
- **Festivos**: destacados en rojo
- **Excepciones**: con indicador visual diferenciado
- **Período de cierre**: días tachados
- **Período sin franja horaria**: marcado como incompleto

Esta vista permite comprobar de un vistazo el estado completo del horario comercial y detectar posibles omisiones.

## 7. Errores frecuentes

El sistema dispone de un listado de errores visible en ambas pestañas (Horario Comercial y Vista de Calendario), presentado en un recuadro destacado en la parte izquierda de la pantalla. Los errores más habituales son:

- **Período sin franja horaria completa**: aparece cuando un período no tiene franjas registradas para todos los días de la semana.
- **Año sin cobertura completa de períodos**: se muestra cuando no hay períodos que cubran la totalidad del año (del 1 de enero al 31 de diciembre).
- **Excepción de apertura sin franja**: se genera cuando existe una excepción de tipo apertura pero no se han definido franjas horarias para ese día excepcional.
