# Informe de exploración — Gestión de horarios comerciales de apertura (sistema legacy M7)

> Generado a partir de 15 documentos de research: 8 entrevistas con usuarios de tienda, 4 entrevistas/feedback de stakeholders internos (producto, HR, soporte), 2 documentos de observación del sistema actual, 1 manual de la aplicación legacy.
> Fecha de generación: 12 de mayo de 2026.

## 1. Resumen ejecutivo

Se ha investigado el problema de la gestión de horarios comerciales de apertura en una red de retail multinacional con establecimientos propios y franquicias en múltiples países europeos. El sistema actual (M7, integrado en HRS) es una aplicación legacy construida sobre tecnología descatalogada desde 2014, cuya base de datos alcanza fin de vida en 2026. El análisis de 15 fuentes revela un consenso absoluto: la herramienta es compleja, poco intuitiva, genera duplicación de trabajo y provoca un volumen elevado de incidencias operativas. Los usuarios la perciben como un obstáculo, no como una herramienta de apoyo. La conclusión principal es que se necesita una nueva aplicación con interfaz moderna, lógica simplificada (horario base + festivos + excepciones puntuales), automatización de la copia interanual y adaptabilidad a las casuísticas de cada país. La recomendación inmediata es priorizar el desarrollo de esta nueva herramienta antes de diciembre, dado que la carga de horarios del año siguiente comienza en Q4 y sin ella se duplicará trabajo en un sistema que está en proceso de descomisión.

## 2. Fuentes analizadas

| # | Archivo | Tipo de fuente | Participante/contexto | Temas principales |
|---|---------|---------------|----------------------|-------------------|
| 1 | hc-01-marina-opening-hours-sep2025.md | entrevista-usuario | Marina, responsable de tienda (Via Roma, Italy) | Flujo anual, registro M3/M7, propuesta de app unificada |
| 2 | hc-02-carla-elena-opening-hours-sep2025.md | entrevista-usuario | Carla y Elena, backoffice (Netherlands) | Obtención de horarios del shopping centre, dificultades M7, períodos innecesarios |
| 3 | hc-03-tomas-lorena-marta-opening-hours-sep2025.md | entrevista-usuario | Tomás, Lorena y Marta, managers (Spain) | Comunicación de festivos, réplica interanual, flujo M7 |
| 4 | hc-04-lucia-opening-hours-sep2025.md | entrevista-usuario | Lucía, backoffice (Spain) | Coordinación Corporate Ops/HR, registro M3/M7, excepciones |
| 5 | hc-05-daniela-pedro-opening-hours-aug2025.md | entrevista-usuario | Daniela y Pedro, staff tienda (UK/Ireland) | Shopping centres vs high street, bank holidays, períodos múltiples en diciembre |
| 6 | hc-06-carmen-opening-hours-aug2025.md | entrevista-usuario | Carmen, perfil HR/operaciones (Spain) | Normativa provincial, collective agreements, excepciones horarias |
| 7 | hc-07-gonzalo-opening-hours-jul2025.md | entrevista-usuario | Gonzalo, director de tienda (Spain) | Proceso de confirmación anual, M7 vacío, dependencia de correo |
| 8 | hc-08-glenn-opening-hours-sep2025.md | entrevista-usuario | Glenn, store manager (Netherlands/UK) | Interfaz M7 cumbersome, notificaciones tardías, duplicación anual |
| 9 | ref-01-marta-alberto-as-is-explanation-aug2025.md | stakeholder | Marta y Alberto, equipo de producto | Contexto técnico, sistemas consumidores, fin de vida, franquicias |
| 10 | ref-02-silvia-ferretti-feedback.md | stakeholder | Silvia Ferretti, producto/operativa comercial | Casuísticas por tipo de tienda, fuentes de información, descentralización |
| 11 | ref-03-clara-interview-sep2025.md | stakeholder | Clara, HR administración de personal (Italy) | Diagnóstico del sistema, casuísticas por tipología, propuestas de mejora |
| 12 | ref-04-valentina-sofia-opening-hours.md | observación | Valentina y Sofía, equipo Large Format (soporte) | Incidencias, errores del sistema, propuesta de modelo simplificado |
| 13 | ref-05-margaret-opening-hours-sep2025.md | stakeholder | Margaret, HR (Netherlands, Vanguard) | Calendario compartido en Excel, planificación anticipada |
| 14 | ref-06-nathan-osei-commercial-calendar.md | entrevista-usuario | Nathan Osei, Store Operations Manager (franquicia, Paris) | M7 no migrado, excepciones complejas, mockups propuestos |
| 15 | manual_legacy.md | observación | Documento de formación interno | Lógica funcional M7: períodos, franjas, excepciones, festivos |

## 3. Quién tiene el problema

### Perfil 1: Responsable de tienda / Backoffice de establecimiento

- **Quién es:** persona encargada de registrar los horarios comerciales en el sistema. Puede ser la directora de tienda, la administrativa del backoffice o una encargada delegada. Es quien ejecuta la tarea operativa en M7.
- **Frecuencia en las fuentes:** 10 de 15 fuentes (hc-01 a hc-08, ref-04, ref-06).
- **Cita representativa:** "The M7 interface is cumbersome and unintuitive" [Fuente: hc-08-glenn]. "Even returning users find themselves re-reading the manual every year" [Fuente: ref-06-nathan-osei].
- **Fuentes:** hc-01, hc-02, hc-03, hc-04, hc-05, hc-06, hc-07, hc-08, ref-04, ref-06.

### Perfil 2: Equipo de HR / Administración de personal

- **Quién es:** responsable de distribuir los work calendars con festivos, validar horarios y, en algunos mercados, registrar directamente los datos en el sistema por delegación de las tiendas. Actúa como puente entre la normativa y el establecimiento.
- **Frecuencia en las fuentes:** 6 de 15 fuentes (hc-01, hc-04, hc-06, ref-03, ref-05, ref-02).
- **Cita representativa:** "La interfaz resulta tan poco intuitiva que las responsables necesitan releer el manual cada vez que la usan tras un periodo sin acceder" [Fuente: ref-03-clara].
- **Fuentes:** hc-01, hc-04, hc-06, ref-03, ref-05, ref-02.

### Perfil 3: Equipo de soporte operativo (Large Format)

- **Quién es:** equipo centralizado que verifica la correcta carga de horarios en M7, resuelve incidencias y asiste a tiendas y franquicias cuando cometen errores en el sistema.
- **Frecuencia en las fuentes:** 2 de 15 fuentes (ref-01, ref-04).
- **Cita representativa:** "Se pasó de unas 400 incidencias al inicio a apenas 40 en la última semana" [Fuente: ref-04-valentina-sofia].
- **Fuentes:** ref-01, ref-04.

### Perfil 4: Franquicias

- **Quién es:** establecimientos operados por terceros que están obligados contractualmente a usar el sistema de horarios comerciales para toda la gestión, pero carecen de aplicaciones propias de horarios y de soporte directo del personal de la empresa matriz.
- **Frecuencia en las fuentes:** 3 de 15 fuentes (ref-01, ref-04, ref-06).
- **Cita representativa:** "Piden guías paso a paso para poder recordar los procedimientos. Existe confusión generalizada sobre cómo realizar la carga de datos" [Fuente: ref-04-valentina-sofia].
- **Fuentes:** ref-01, ref-04, ref-06.

## 4. El problema

### 4.1 Descripción del problema central

La red de establecimientos necesita registrar anualmente sus horarios comerciales de apertura en un sistema centralizado (M7) para que múltiples sistemas downstream (Google Maps, entregas express, reserva de probadores, APIs de terceros) muestren correctamente la disponibilidad de cada tienda. El sistema actual obliga a un proceso manual, repetitivo y propenso a errores que se ejecuta una vez al año pero genera incidencias a lo largo de todo el ejercicio. La complejidad del modelo de datos (períodos + franjas + excepciones + festivos en módulos separados) no refleja la realidad operativa de la mayoría de tiendas, cuyo horario es esencialmente constante con variaciones puntuales. El resultado es un desajuste entre la sencillez del dato real y la complejidad del proceso de registro, multiplicado por la diversidad normativa de cada país y la obsolescencia técnica de la interfaz.

### 4.2 Puntos de dolor específicos

**1. Interfaz obsoleta e inaccesible**
- **Descripción:** M7 funciona únicamente en Windows a través de RemoteAccess, es incompatible con iPad/Mac, y su interfaz es extremadamente poco intuitiva. Los usuarios necesitan releer el manual cada vez que acceden tras un período sin usarlo.
- **Impacto:** Tiempo perdido en reaprendizaje, dependencia de un único dispositivo, frustración generalizada.
- **Frecuencia:** 8 de 15 fuentes.
- **Fuentes:** hc-08, ref-01, ref-03, ref-04, ref-06, hc-02, hc-05, manual_legacy.

**2. Duplicación de trabajo entre M3 y M7**
- **Descripción:** Los festivos deben registrarse primero en M3 (indicando apertura/cierre) y luego en M7 (con el horario correspondiente). Son dos módulos separados con lógica que debería ser un solo paso.
- **Impacto:** Doble entrada de datos, riesgo de inconsistencia, confusión sobre qué se registra dónde.
- **Frecuencia:** 6 de 15 fuentes.
- **Fuentes:** hc-01, hc-02, hc-04, ref-03, ref-04, manual_legacy.

**3. Necesidad de recrear el horario completo cada año**
- **Descripción:** Aunque la mayoría de tiendas mantienen horarios estables año tras año, el sistema obliga a reconstruir la configuración (o copiarla manualmente con limitaciones). La copia no incluye festivos ni excepciones.
- **Impacto:** Trabajo repetitivo innecesario multiplicado por cientos de establecimientos.
- **Frecuencia:** 7 de 15 fuentes.
- **Fuentes:** hc-02, hc-03, hc-05, hc-08, ref-03, ref-04, ref-06.

**4. Modelo de períodos excesivamente rígido**
- **Descripción:** Para definir cualquier variación horaria es necesario crear períodos adicionales. Un establecimiento con horario constante pero con un diciembre especial necesita crear períodos extra por cada semana diferente. El sistema exige cobertura continua del 1 de enero al 31 de diciembre sin huecos.
- **Impacto:** Carla y Elena reportan necesitar 4-5 períodos cuando bastaría con 1; Daniela necesita períodos por semana en diciembre.
- **Frecuencia:** 5 de 15 fuentes.
- **Fuentes:** hc-02, hc-05, ref-06, manual_legacy, ref-04.

**5. Confusión terminológica entre excepciones y festivos**
- **Descripción:** La distinción entre "excepción de apertura", "excepción de cierre", "excepción de modificación de horario" y "festivo" es confusa. Los usuarios registran aperturas en festivos como excepciones, o cierres festivos como excepciones de cierre, generando datos incorrectos.
- **Impacto:** 28 establecimientos con disponibilidad 24/7 errónea por excepciones mal configuradas. Corrección individual registro a registro.
- **Frecuencia:** 4 de 15 fuentes.
- **Fuentes:** ref-04, ref-06, manual_legacy, hc-02.

**6. Imposibilidad de modificar registros in situ**
- **Descripción:** El sistema no permite editar entradas existentes de forma directa. Para corregir un error o actualizar un dato, los managers deben eliminar toda la configuración y reconstruirla desde cero.
- **Impacto:** Cualquier error menor requiere un esfuerzo desproporcionado de corrección.
- **Frecuencia:** 3 de 15 fuentes.
- **Fuentes:** ref-06, ref-04, manual_legacy.

**7. Timing impredecible de la información**
- **Descripción:** Los horarios llegan de múltiples fuentes (shopping centres, HR, Corporate Operations) en momentos distintos y a menudo tarde. No hay una fecha límite clara ni un flujo estandarizado de comunicación.
- **Impacto:** Registros a última hora, tiendas sin horario en Google, planificación de personal comprometida.
- **Frecuencia:** 6 de 15 fuentes.
- **Fuentes:** hc-02, hc-04, hc-05, hc-08, ref-03, ref-05.

**8. Falta de visibilidad del horario de cierre en festivos aperturables**
- **Descripción:** M3 solo indica si un festivo es de apertura o cierre, pero no muestra la hora de cierre. Esta información es necesaria para planificar los 30 minutos de recogida posteriores al cierre.
- **Impacto:** Mala planificación de turnos de cierre.
- **Frecuencia:** 2 de 15 fuentes.
- **Fuentes:** hc-02, hc-06.

### 4.3 Contradicciones detectadas

**Contradicción 1: Quién es responsable de registrar los horarios en el sistema**
- **Fuentes A (hc-01, hc-07, ref-03):** HR se encarga de registrar los datos de forma centralizada para evitar errores. Las tiendas solo comunican su horario.
- **Fuentes B (hc-02, hc-03, hc-04, hc-05, hc-08, ref-06):** Las tiendas/backoffice son quienes registran directamente en M7.
- **Posible explicación:** El modelo varía por mercado y por marca. En Italy (Via Roma) se ha migrado a un modelo centralizado por HR. En Netherlands, UK y Spain el registro lo hace la tienda. Esto no es una contradicción real sino una diversidad de modelos operativos que la nueva herramienta debe contemplar.

**Contradicción 2: Uso de M3 vs solo M7**
- **Fuentes A (hc-01, hc-02, hc-04, hc-06):** El proceso requiere registrar festivos en M3 y luego horarios en M7 (dos pasos separados).
- **Fuentes B (hc-03):** "El procedimiento de horarios comerciales se gestiona en M7 (no en M3, a diferencia de otras marcas)."
- **Posible explicación:** Diferentes marcas dentro del grupo tienen diferentes flujos. Algunas marcas usan M3+M7, otras solo M7. La nueva herramienta debería unificar ambos flujos.

## 5. Solución actual (as-is)

### 5.1 Herramientas y procesos actuales

**M7 (Horarios Comerciales en HRS)**
- **Qué es:** Módulo del sistema HRS legacy para registrar períodos, franjas horarias, excepciones y festivos de cada establecimiento.
- **Qué resuelve:** Centraliza el dato de horario comercial que alimenta a sistemas downstream.
- **Quién lo usa:** Responsables de tienda, backoffice, HR (según mercado).
- **Fuentes:** Todas las fuentes (15/15).

**M3 (Gestión de Festivos en HRS)**
- **Qué es:** Módulo separado dentro de HRS para registrar días festivos indicando si son de apertura o cierre.
- **Qué resuelve:** Identifica los festivos del calendario para cada establecimiento.
- **Quién lo usa:** Responsables de tienda, HR.
- **Fuentes:** hc-01, hc-02, hc-04, hc-06, hc-07, ref-03, manual_legacy.

**Work Calendar anual (distribuido por HR)**
- **Qué es:** Documento que recoge los festivos nacionales, regionales y locales aplicables a cada comunidad/provincia. Se distribuye entre septiembre y noviembre.
- **Qué resuelve:** Proporciona el input de festivos que luego se registra en el sistema.
- **Quién lo usa:** HR lo genera, las tiendas lo consumen.
- **Fuentes:** hc-01, hc-03, hc-04, hc-06, ref-03.

**Proceso ETL en lote nocturno**
- **Qué es:** Sincronización batch entre el sistema de horarios y los sistemas consumidores (Google, stock, experiencia en tienda, APIs OAuth).
- **Qué resuelve:** Distribución del dato a sistemas downstream.
- **Quién lo usa:** Proceso automático (sin intervención humana directa).
- **Fuentes:** ref-01.

### 5.2 Workarounds detectados

**Excel compartido de Margaret (Netherlands)**
- Margaret mantiene un workbook de Excel con edición colaborativa en tiempo real para gestionar el calendario comercial de Vanguard Netherlands. Incluye holidays, staff placement y horarios de managers. Es un workaround porque la herramienta oficial no facilita la colaboración.
- **Fuentes:** ref-05.

**Chat de Teams para consensuar festivos (Italy)**
- Los equipos comerciales de Italy usan un chat de Teams para acordar qué festivos se abren en zonas turísticas, aproximadamente un mes antes de cada festivo. La herramienta oficial no tiene mecanismo de coordinación.
- **Fuentes:** ref-03.

**Búsqueda manual de información en fuentes externas (Netherlands)**
- Cuando el shopping centre no facilita los horarios a tiempo, Carla busca la información en la web del ayuntamiento, la Official Gazette o a través del works council. No existe un canal oficial fiable.
- **Fuentes:** hc-02.

**Incidencia código 07 para registro centralizado (Italy)**
- En Via Roma se creó un procedimiento ad hoc donde la tienda envía una incidencia para que HR registre los datos, evitando que la tienda toque el sistema directamente.
- **Fuentes:** hc-01.

**Réplica manual del año anterior**
- La mayoría de establecimientos copian la configuración del año anterior y solo modifican los festivos variables (Semana Santa, etc.). Es el método estándar de facto aunque no esté diseñado como tal.
- **Fuentes:** hc-03, hc-05, hc-06, hc-08.

**Mockups de Nathan con IA**
- Nathan (franquicia Paris) desarrolló por su cuenta mockups de una interfaz alternativa más intuitiva usando herramientas de IA, evidenciando la frustración con el sistema actual.
- **Fuentes:** ref-06.

## 6. Limitaciones de la solución actual

**1. Tecnología en fin de vida**
- **Limitación:** La base de datos legacy alcanza fin de vida en 2026. El framework está descatalogado desde 2014.
- **Impacto:** Migración obligatoria con deadline en octubre 2026 impuesto por GroupTex. Imposibilidad de mantener o evolucionar el sistema actual.
- **Evidencia:** [Fuente: ref-01-marta-alberto] "Legacy database alcanza fin de vida en 2026", "framework descatalogado desde 2014, sin mantenimiento posible."

**2. Incompatibilidad con dispositivos modernos**
- **Limitación:** Solo funciona en Windows vía RemoteAccess. No es compatible con iPad ni Mac.
- **Impacto:** Las tiendas que operan con iPad (que son la mayoría) no pueden acceder de forma nativa.
- **Evidencia:** [Fuente: ref-01] "Únicamente operativo en HRS 07; incompatible con dispositivos iPad o Mac."

**3. Ausencia de conexión en tiempo real**
- **Limitación:** La sincronización con sistemas consumidores se ejecuta mediante ETL en lote nocturno.
- **Impacto:** Los cambios de horario no se reflejan hasta el día siguiente. Si una tienda cierra de emergencia, Google no lo muestra inmediatamente.
- **Evidencia:** [Fuente: ref-01] "La sincronización se ejecuta mediante un proceso ETL en lote nocturno, sin conexión en tiempo real."

**4. Sin integración entre nóminas y horarios comerciales**
- **Limitación:** El sistema de horarios comerciales y el de nóminas/turnos no están conectados.
- **Impacto:** Los cambios en horario comercial requieren ajustes manuales paralelos en planificación de personal.
- **Evidencia:** [Fuente: ref-03] "No existe integración entre el sistema de nóminas y el de horarios comerciales."

**5. Pérdida de histórico al cambiar de ubicación**
- **Limitación:** Cuando una tienda cambia de local, se pierde todo el histórico de horarios asociado.
- **Impacto:** Imposibilidad de usar datos pasados como referencia para la nueva ubicación.
- **Evidencia:** [Fuente: ref-03] "Cuando una tienda cambia de ubicación, se pierde todo el histórico de información asociado."

**6. Ubicación incorrecta en la arquitectura de aplicaciones**
- **Limitación:** El módulo aparece dentro de la sección de HR del HRS, cuando conceptualmente es un dato operativo/comercial.
- **Impacto:** Confusión organizativa, equipos equivocados gestionando el módulo, dificultad para encontrar la funcionalidad.
- **Evidencia:** [Fuente: ref-01] "El módulo aparece erróneamente dentro de la sección de HR del HRS. Desde el punto de vista técnico, debería ser un dato nativo gestionado por la DSU."

## 7. Requisitos de una solución adecuada

**R1. Interfaz moderna, accesible desde iPad y dispositivos móviles**
- **Justificación:** El sistema actual solo funciona en Windows vía RemoteAccess. Los establecimientos operan mayoritariamente con iPad.
- **Prioridad inferida:** Alta.
- **Fuentes:** ref-01, hc-01, ref-03, ref-06, hc-08.

**R2. Copia automática del horario del año anterior como punto de partida**
- **Justificación:** La mayoría de establecimientos mantienen horarios estables. El proceso actual obliga a reconstruir o copiar manualmente con limitaciones (no incluye festivos ni excepciones).
- **Prioridad inferida:** Alta.
- **Fuentes:** hc-03, hc-05, hc-06, hc-08, ref-03, ref-04.

**R3. Modelo simplificado: horario base + festivos + excepciones en un solo flujo**
- **Justificación:** La separación M3/M7 y la confusión entre excepciones/festivos genera duplicación y errores. El modelo de Valentina/Sofía (4 bloques) refleja el consenso: horario habitual, festivos de cierre, festivos con apertura, cierres temporales.
- **Prioridad inferida:** Alta.
- **Fuentes:** ref-04, ref-03, hc-01, hc-02, ref-06.

**R4. Eliminación de la terminología confusa**
- **Justificación:** Los conceptos "excepción de apertura", "excepción de cierre", "excepción de modificación" generan confusión. El lenguaje debe ser comprensible para personal de tienda sin formación técnica.
- **Prioridad inferida:** Alta.
- **Fuentes:** ref-04, ref-06, manual_legacy.

**R5. Adaptabilidad a las casuísticas de cada país/mercado**
- **Justificación:** Las reglas varían radicalmente: shopping centres que imponen horarios, normativa provincial, collective agreements, festivos locales que solo la tienda conoce, excepciones por observancia religiosa.
- **Prioridad inferida:** Alta.
- **Fuentes:** hc-01, hc-02, hc-06, ref-02, ref-06, hc-05.

**R6. Soporte para franquicias con guía paso a paso**
- **Justificación:** Las franquicias están obligadas a usar el sistema pero carecen de soporte directo y tienen confusión generalizada. Necesitan una experiencia más guiada.
- **Prioridad inferida:** Alta.
- **Fuentes:** ref-01, ref-04, ref-06.

**R7. Mecanismos de prevención de errores**
- **Justificación:** El sistema actual permite registrar excepciones sin horario asociado, creando disponibilidad 24/7 errónea. La corrección es manual y registro a registro.
- **Prioridad inferida:** Alta.
- **Fuentes:** ref-04, ref-06, manual_legacy.

**R8. Edición in situ sin necesidad de eliminar y recrear**
- **Justificación:** Actualmente cualquier corrección requiere borrar toda la configuración y rehacerla. Esto desincentiva las correcciones y perpetúa errores.
- **Prioridad inferida:** Alta.
- **Fuentes:** ref-06, ref-04.

**R9. Notificaciones y gestión de plazos**
- **Justificación:** La información llega de múltiples fuentes en momentos impredecibles. Se necesitan alertas automatizadas para garantizar que los horarios se registren antes de la fecha límite.
- **Prioridad inferida:** Media.
- **Fuentes:** ref-03, hc-04, hc-08.

**R10. Detección automática de conflictos o solapamientos**
- **Justificación:** Nathan reporta la necesidad de que el sistema detecte automáticamente franjas horarias conflictivas o solapadas.
- **Prioridad inferida:** Media.
- **Fuentes:** ref-06.

**R11. Indicador de estado en tiempo real**
- **Justificación:** Nathan propone un indicador tipo "Currently open until 9PM" que permita verificar de un vistazo si el sistema refleja correctamente el estado actual de la tienda.
- **Prioridad inferida:** Media.
- **Fuentes:** ref-06.

**R12. Sincronización en tiempo real con sistemas downstream**
- **Justificación:** El ETL nocturno actual impide que cambios urgentes (cierres de emergencia) se reflejen en Google u otros canales.
- **Prioridad inferida:** Media.
- **Fuentes:** ref-01, ref-03.

**R13. Sugerencia automática de festivos según ubicación**
- **Justificación:** Clara propone que el sistema genere propuestas automáticas de festivos de apertura en función de la ubicación geográfica de la tienda.
- **Prioridad inferida:** Baja (segunda fase).
- **Fuentes:** ref-03.

**R14. Ecosistema de coordinación entre equipos comerciales y HR para festivos**
- **Justificación:** Hoy la coordinación se hace por chat de Teams de forma informal. Se necesita un flujo estructurado para aprobar aperturas en festivos.
- **Prioridad inferida:** Baja (segunda fase).
- **Fuentes:** ref-03.

## 8. Gaps de información

**Gap 1: Volumen exacto de establecimientos y distribución por tipología**
- **Pregunta:** ¿Cuántos establecimientos propios y franquicias hay por mercado? ¿Cuál es la proporción shopping centre vs high street vs tienda 365?
- **Por qué importa:** Determina la escala del problema y la priorización de funcionalidades por volumen de usuarios afectados.
- **Acción recomendada:** Solicitar datos al equipo de operaciones o extraer del sistema actual.

**Gap 2: Detalle de los sistemas consumidores y sus requisitos de integración**
- **Pregunta:** ¿Qué formato exacto necesita cada sistema downstream (Google/Physical Store, stock, experiencia en tienda, APIs OAuth)? ¿Qué latencia es aceptable?
- **Por qué importa:** Define los requisitos técnicos de la nueva API y si el ETL nocturno puede mantenerse o necesita reemplazarse por tiempo real.
- **Acción recomendada:** Entrevistar a Marco (citado en ref-01) y al equipo DSU. Revisar el canal de Teams con 39 integrantes.

**Gap 3: Flujo exacto de las franquicias y sus limitaciones contractuales**
- **Pregunta:** ¿Qué puede y qué no puede hacer GlobalTextile respecto a las franquicias? ¿Pueden darse permisos de solo lectura? ¿Qué pasa si una franquicia no carga sus horarios?
- **Por qué importa:** Las franquicias son el perfil con más dificultades pero con menos control organizativo.
- **Acción recomendada:** Revisar cláusulas contractuales con Legal. Entrevistar a más franquiciados.

**Gap 4: Proceso de migración Aurora y hitos de fin de vida**
- **Pregunta:** ¿Qué es exactamente "la migración Aurora"? ¿Cuáles son los hitos del 31 de enero, 31 de octubre y 3 de abril mencionados? ¿Existe un plan de rollout por mercados?
- **Por qué importa:** Define el deadline real del proyecto y las restricciones de calendario.
- **Acción recomendada:** Entrevistar a Marta/Alberto del equipo de producto para obtener el roadmap de migración.

**Gap 5: Experiencia de los mercados con app propia de horarios**
- **Pregunta:** ¿Qué mercados tienen aplicación local de horarios que alimenta al sistema central vía ETL? ¿Cómo funciona? ¿Es un buen modelo a replicar?
- **Por qué importa:** Puede existir una solución parcial ya validada que sirva de referencia para el diseño de la nueva herramienta.
- **Acción recomendada:** Identificar estos mercados y analizar su aplicación local.

**Gap 6: Perfil de usuario técnico del equipo de soporte L2**
- **Pregunta:** ¿Qué tipo de incidencias resuelve el Support Center L2? ¿Cuál es el volumen mensual? ¿Cuáles son las más recurrentes?
- **Por qué importa:** Permite diseñar funcionalidades preventivas que eliminen las incidencias más comunes.
- **Acción recomendada:** Solicitar datos de ticketing del último año al equipo de soporte.

**Gap 7: Requisitos de seguridad y autenticación**
- **Pregunta:** ¿Qué nivel de control de acceso se necesita? Nathan menciona "manager authentication codes" al estilo del sistema de stock.
- **Por qué importa:** Define el modelo de permisos de la nueva aplicación.
- **Acción recomendada:** Consultar con el equipo de seguridad y con Nathan sobre el modelo propuesto.
