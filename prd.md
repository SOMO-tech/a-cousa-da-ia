# PRD — Commercial Opening Hours (Aurora)

> Versión: 1.0
> Fecha: 12 de mayo de 2026
> Basado en: exploration-report.md

---

## 1. Contexto y Visión del producto

### 1.1 Resumen del problema

La red de establecimientos retail (tiendas propias y franquicias en múltiples países europeos) gestiona sus horarios comerciales de apertura a través de un sistema legacy (M7/M3) construido sobre tecnología descatalogada desde 2014, cuya base de datos alcanza fin de vida en octubre de 2026. El sistema actual obliga a un proceso manual, repetitivo y propenso a errores: interfaz solo accesible vía Windows/RemoteAccess, duplicación de registro entre dos módulos (M3 para festivos, M7 para horarios), modelo de datos rígido que no refleja la realidad operativa, y terminología confusa que genera errores sistémicos. El dato de horario comercial alimenta sistemas críticos (Google Maps, entregas express, experiencia en tienda, APIs de terceros), por lo que cualquier error tiene impacto directo en la experiencia del cliente final y en la operativa logística.

### 1.2 Alcance del Producto

- **Incluye:** Sustitución funcional completa de M7 y M3 en una aplicación unificada. Registro de horarios base, gestión de festivos, excepciones, cierres temporales, soporte multi-país, integración con sistemas downstream y soporte para establecimientos propios y franquicias.
- **No incluye:** Ver sección 5 (Exclusiones) para el detalle.
- **Versión:** V1 completa.
- **Usuarios objetivo:** Todos los perfiles identificados en el exploration report: responsables de tienda/backoffice, equipo HR, equipo de soporte operativo (Large Format) y franquicias.

### 1.3 Objetivos de Negocio

| # | Objetivo | Métrica asociada | Target |
|---|----------|-----------------|--------|
| 1 | Eliminar la dependencia del sistema legacy antes de su descomisión | % de establecimientos migrados a la nueva herramienta | 100% antes de octubre 2026 |
| 2 | Reducir el volumen de incidencias operativas por errores de carga de horarios | Tickets L2 relacionados con horarios comerciales / mes | Reducción del 80% vs baseline (por definir con datos baseline) |
| 3 | Reducir el tiempo medio de carga anual de horarios por establecimiento | Minutos por establecimiento para completar la configuración anual | Por definir con datos baseline |
| 4 | Garantizar la disponibilidad del dato de horarios para sistemas downstream sin interrupción | Uptime del servicio de horarios durante la transición | 99.5% |

---

## 2. User Personas y Casos de uso

### 2.1 User Personas

#### Persona 1: Lucía (Responsable de tienda / Backoffice)

- **Rol:** Administrativa de backoffice en establecimiento propio.
- **Contexto:** Gestiona las tareas administrativas del establecimiento, incluyendo la carga anual de horarios comerciales. Accede al sistema desde iPad. El registro de horarios es una tarea puntual (1-2 veces al año) pero crítica.
- **Objetivo principal:** Registrar los horarios del año siguiente de forma rápida, sin errores y sin necesidad de releer un manual.
- **Frustración principal:** La interfaz es tan poco intuitiva que necesita releer el manual cada vez que accede tras un período sin usarla. El sistema no funciona en iPad.
- **Cita representativa:** "The M7 interface is cumbersome and unintuitive."
- **Fuente:** hc-08-glenn, ref-06-nathan-osei.

#### Persona 2: Clara (HR / Administración de personal)

- **Rol:** Responsable de administración de personal para un conjunto de tiendas de un mercado o región.
- **Contexto:** Distribuye los work calendars anuales con festivos, valida horarios, y en algunos mercados registra directamente los datos por delegación de las tiendas. Coordina con equipos comerciales la decisión de apertura en festivos.
- **Objetivo principal:** Que los horarios estén correctamente registrados para todas sus tiendas sin tener que intervenir manualmente en cada una.
- **Frustración principal:** La duplicación M3/M7, la falta de integración con nóminas, y la dispersión de la información de festivos locales.
- **Cita representativa:** "La interfaz resulta tan poco intuitiva que las responsables necesitan releer el manual cada vez que la usan tras un periodo sin acceder."
- **Fuente:** ref-03-clara.

#### Persona 3: Valentina (Soporte operativo / Large Format)

- **Rol:** Miembro del equipo centralizado que verifica cargas, resuelve incidencias y asiste a tiendas y franquicias.
- **Contexto:** Monitoriza alertas del sistema, detecta errores de configuración y contacta proactivamente con establecimientos para corregir problemas antes de que afecten a sistemas downstream.
- **Objetivo principal:** Que los establecimientos no cometan errores al cargar horarios, eliminando la necesidad de intervención correctiva.
- **Frustración principal:** Errores sistémicos (excepciones mal configuradas que generan disponibilidad 24/7) cuya corrección es manual y registro a registro.
- **Cita representativa:** "Se pasó de unas 400 incidencias al inicio a apenas 40 en la última semana."
- **Fuente:** ref-04-valentina-sofia.

#### Persona 4: Nathan (Franquicia)

- **Rol:** Store Operations Manager en un establecimiento franquiciado.
- **Contexto:** Obligado contractualmente a usar el sistema de horarios comerciales pero sin soporte directo del personal de la empresa matriz. Gestiona casuísticas complejas (excepciones por observancia religiosa, eventos especiales).
- **Objetivo principal:** Configurar el calendario anual de su tienda sin necesidad de borrar y recrear todo ante cualquier cambio.
- **Frustración principal:** La imposibilidad de editar registros in situ y la falta de documentación/formación.
- **Cita representativa:** "Even returning users find themselves re-reading the manual every year."
- **Fuente:** ref-06-nathan-osei.

### 2.2 Casos de uso principales

| # | Caso de uso | Persona | Descripción | Frecuencia |
|---|-------------|---------|-------------|------------|
| CU-01 | Configuración anual de horarios | Lucía, Nathan | Crear la configuración de horarios para el año siguiente partiendo de la copia del año anterior, ajustando solo lo que cambia | Anual |
| CU-02 | Registro de festivos | Lucía, Clara | Indicar qué días son festivos y si son de apertura o cierre, con su horario específico si difiere del habitual | Anual + ajustes puntuales |
| CU-03 | Registro de excepciones puntuales | Lucía, Nathan | Añadir un día con horario diferente al estándar (Nochebuena cierre anticipado, evento especial, etc.) | Puntual (5-10 veces/año) |
| CU-04 | Registro de cierre temporal | Lucía | Marcar un período de cierre por obras, emergencia u otro motivo extraordinario | Puntual |
| CU-05 | Verificación y corrección de errores | Valentina | Detectar establecimientos con configuración incorrecta y corregir sin necesidad de recrear toda la configuración | Semanal |
| CU-06 | Distribución de festivos por mercado | Clara | Cargar de forma centralizada los festivos nacionales/regionales para un conjunto de establecimientos | Anual |
| CU-07 | Consulta del estado actual | Todos | Verificar de un vistazo si el horario registrado refleja correctamente el estado actual de la tienda | Diaria |
| CU-08 | Gestión multi-período estacional | Lucía, Nathan | Configurar horarios diferentes para temporadas (verano/invierno) o semanas especiales (diciembre) | Anual |

---

## 3. Requisitos funcionales

### 3.1 Horario base y períodos (HOR)

| Código | Nombre | Descripción | Rationale | Criticidad |
|--------|--------|-------------|-----------|------------|
| RF-HOR-001 | Copia automática interanual | El sistema debe permitir copiar la configuración completa del año anterior (incluyendo festivos y excepciones recurrentes) como punto de partida para el nuevo año | Punto de dolor #3: recreación anual innecesaria. 7/15 fuentes. | Must |
| RF-HOR-002 | Período único con excepciones | El sistema debe permitir configurar un único período anual con horario base y gestionar todas las variaciones como excepciones, sin obligar a crear múltiples períodos | Punto de dolor #4: modelo de períodos rígido. 5/15 fuentes. | Must |
| RF-HOR-003 | Períodos múltiples opcionales | El sistema debe permitir opcionalmente crear múltiples períodos cuando el establecimiento tiene horarios estacionales genuinamente diferentes (verano/invierno) | Necesidad real en tiendas costeras y con horario estacional | Must |
| RF-HOR-004 | Horario por día de la semana | Dentro de cada período, el sistema debe permitir definir horarios diferentes para cada día de la semana (L-D) | Funcionalidad core heredada de M7 | Must |
| RF-HOR-005 | Horario partido | El sistema debe soportar múltiples franjas horarias en un mismo día (apertura mañana + apertura tarde) | Necesidad en mercados con cierre a mediodía | Must |
| RF-HOR-006 | Validación de cobertura anual | El sistema debe alertar si la configuración no cubre los 365 días del año, pero no bloquear el guardado | Prevención de errores sin rigidez excesiva | Must |
| RF-HOR-007 | Indicador de estado actual | El sistema debe mostrar un indicador tipo "Actualmente abierto hasta las 21:00" que permita verificar de un vistazo el estado correcto | Requisito R11 del exploration report. Fuente: ref-06. | Should |

### 3.2 Festivos (FES)

| Código | Nombre | Descripción | Rationale | Criticidad |
|--------|--------|-------------|-----------|------------|
| RF-FES-001 | Registro unificado de festivos | Los festivos se registran en un único flujo (no en M3 y M7 por separado), indicando apertura/cierre y horario específico en un solo paso | Punto de dolor #2: duplicación M3/M7. 6/15 fuentes. | Must |
| RF-FES-002 | Herencia de horario en festivos aperturables | Si un festivo es de apertura y no se especifica horario, el sistema debe asignar automáticamente el horario del período correspondiente | Reducción de trabajo manual. Fuente: manual_legacy. | Must |
| RF-FES-003 | Carga centralizada de festivos por mercado | HR debe poder cargar festivos nacionales/regionales para un grupo de establecimientos de una sola vez | CU-06. Fuentes: hc-01, ref-03, ref-05. | Must |
| RF-FES-004 | Festivos con horario diferenciado | El sistema debe permitir que un festivo de apertura tenga un horario distinto al habitual (ej: Nochebuena cierre a las 18:00) | Casuística reportada en hc-04, hc-06, ref-06. | Must |
| RF-FES-005 | Visualización del horario de cierre en festivos | Cuando un festivo es apertureable, debe mostrarse claramente la hora de cierre (necesaria para planificar recogida) | Punto de dolor #8. Fuentes: hc-02, hc-06. | Should |

### 3.3 Excepciones y cierres temporales (EXC)

| Código | Nombre | Descripción | Rationale | Criticidad |
|--------|--------|-------------|-----------|------------|
| RF-EXC-001 | Excepciones como modificación puntual | El sistema debe permitir registrar un día con horario diferente al estándar sin necesidad de crear un período nuevo | Simplificación del modelo. Fuentes: hc-04, ref-04. | Must |
| RF-EXC-002 | Cierre temporal por obras/emergencia | El sistema debe permitir marcar un rango de fechas como cierre temporal, con un motivo asociado | CU-04. Fuentes: hc-04, manual_legacy. | Must |
| RF-EXC-003 | Edición in situ de excepciones | Las excepciones deben poder modificarse directamente sin necesidad de eliminar y recrear | Punto de dolor #6. Fuentes: ref-06, ref-04. | Must |
| RF-EXC-004 | Prevención de disponibilidad 24/7 errónea | El sistema debe impedir que una excepción se guarde sin horario asociado si es de tipo apertura, evitando el bug de disponibilidad continua | Punto de dolor #5: 28 tiendas afectadas. Fuente: ref-04. | Must |
| RF-EXC-005 | Detección de conflictos/solapamientos | El sistema debe alertar si se registran excepciones que solapan con otras excepciones o con períodos de cierre | Requisito R10. Fuente: ref-06. | Should |

### 3.4 Interfaz y experiencia (UXI)

| Código | Nombre | Descripción | Rationale | Criticidad |
|--------|--------|-------------|-----------|------------|
| RF-UXI-001 | Lenguaje comprensible | La interfaz debe usar terminología de negocio ("festivo de apertura", "cierre temporal") en lugar de tecnicismos ("excepción de apertura tipo 3") | Punto de dolor #5: confusión terminológica. Fuentes: ref-04, ref-06. | Must |
| RF-UXI-002 | Vista de calendario anual | El sistema debe ofrecer una vista de calendario que muestre de un vistazo el estado completo del año: días con horario, festivos, excepciones, cierres | Funcionalidad existente en M7 (vista de calendario) valorada por los usuarios. | Must |
| RF-UXI-003 | Flujo guiado para franquicias | El sistema debe ofrecer un modo guiado paso a paso para usuarios menos experimentados (franquicias) | Requisito R6. Fuentes: ref-01, ref-04. | Should |
| RF-UXI-004 | Edición directa desde calendario | El usuario debe poder hacer clic en un día del calendario para editar directamente su configuración (horario, festivo, excepción) | Mejora sobre M7 donde la navegación es indirecta. | Should |

### 3.5 Multi-país y adaptabilidad (MPA)

| Código | Nombre | Descripción | Rationale | Criticidad |
|--------|--------|-------------|-----------|------------|
| RF-MPA-001 | Configuración por mercado/país | El sistema debe soportar reglas diferentes por país (festivos nacionales, normativa de apertura, collective agreements) | Requisito R5: casuísticas por país. 6/15 fuentes. | Must |
| RF-MPA-002 | Festivos locales/regionales | Además de los festivos nacionales, el sistema debe permitir registrar festivos regionales y locales específicos de cada establecimiento | Necesidad reportada en ref-02, ref-03, hc-06. | Must |
| RF-MPA-003 | Soporte multi-idioma | La interfaz debe estar disponible al menos en español, italiano, inglés, francés y neerlandés | Mercados identificados en las fuentes. | Must |

### 3.6 Integración y sincronización (INT)

| Código | Nombre | Descripción | Rationale | Criticidad |
|--------|--------|-------------|-----------|------------|
| RF-INT-001 | API para sistemas consumidores | El sistema debe exponer una API que alimente a Google (via Physical Store/DSU), módulo de stock, experiencia en tienda y terceros OAuth | Fuente: ref-01. Sistemas downstream críticos. | Must |
| RF-INT-002 | Sincronización con menor latencia | Los cambios deben reflejarse en sistemas downstream en un plazo máximo de 1 hora (mejora sobre el ETL nocturno actual) | Requisito R12. Fuente: ref-01. | Should |
| RF-INT-003 | Compatibilidad con ETL existente | Durante la transición, el sistema debe poder alimentar el proceso ETL legacy para los sistemas que aún no hayan migrado | Necesidad de coexistencia durante la migración. | Must |
| RF-INT-004 | Recepción de festivos desde apps locales | En mercados con aplicación propia de horarios, el sistema debe poder recibir datos de festivos vía integración (replicando la funcionalidad ETL actual) | Fuente: ref-01. | Should |

### 3.7 Administración y soporte (ADM)

| Código | Nombre | Descripción | Rationale | Criticidad |
|--------|--------|-------------|-----------|------------|
| RF-ADM-001 | Panel de monitorización de alertas | El equipo de soporte (Large Format) debe disponer de un panel que muestre establecimientos con configuración incompleta o errónea | CU-05. Fuentes: ref-04, ref-01. | Must |
| RF-ADM-002 | Corrección masiva | El equipo de soporte debe poder corregir errores en múltiples establecimientos a la vez (ej: eliminar excepciones erróneas en lote) | Fuente: ref-04. Corrección individual es inviable a escala. | Should |
| RF-ADM-003 | Historial de cambios | El sistema debe registrar quién modificó qué y cuándo, permitiendo auditoría y rollback | Buena práctica. Sin evidencia directa en fuentes pero implícito en la operativa multi-actor. | Should |
| RF-ADM-004 | Sistema de roles y permisos | El sistema debe implementar control de acceso basado en roles (RBAC) con al menos 4 roles: establecimiento (edita solo su tienda), HR mercado (gestiona tiendas de su mercado), soporte (acceso total con auditoría) y administrador (gestión de estructura organizativa). Las RLS de base de datos deben reflejar estos roles | Requisito implícito en sección 4.6 (Seguridad) y en la operativa multi-actor multi-mercado. Prerequisito para que la estructura organizativa (RF-MPA-001) funcione con control de acceso real. | Must |

**Resumen de distribución:**

| Criticidad | Cantidad | % del total |
|-----------|----------|-------------|
| Must | 20 | 65% |
| Should | 11 | 35% |
| Could | 0 | 0% |
| **Total** | **31** | **100%** |

---

## 4. Requisitos no funcionales

### 4.1 Usabilidad

- La interfaz debe ser comprensible sin formación previa para un usuario con conocimiento básico de su horario comercial.
- El tiempo para completar la configuración anual de un establecimiento con horario estable (copia + ajuste de festivos) no debe superar los 10 minutos.
- Se debe alcanzar un SUS (System Usability Scale) mínimo de 75 en pruebas con usuarios reales.
- WCAG 2.1 nivel AA como mínimo.

### 4.2 Compatibilidad

- Navegadores: Chrome, Safari, Edge (últimas 2 versiones).
- Dispositivos: iPad (prioridad), tablets Android, desktop (Windows, Mac).
- Resolución mínima: 768px de ancho (tablet portrait).
- No se requiere app nativa.

### 4.3 Internacionalización / Localización

- Idiomas de interfaz: español, italiano, inglés, francés, neerlandés.
- Formato de hora: 24h (estándar europeo).
- Formato de fecha: configurable por locale (DD/MM/YYYY por defecto).
- Los nombres de festivos deben poder introducirse en el idioma local.

### 4.4 Integraciones

- API REST con autenticación OAuth 2.0 (compatible con el ecosistema actual de terceros).
- Integración con el sistema de autenticación corporativo (SSO existente).
- Compatibilidad con el proceso ETL nocturno durante la fase de transición.
- Endpoint dedicado para el servicio DSU/Physical Store (Google).

### 4.5 Disponibilidad y Rendimiento

- Uptime objetivo: 99.5% (excluyendo ventanas de mantenimiento planificadas).
- Tiempo de respuesta de la interfaz: < 2 segundos para cualquier operación.
- Capacidad: soporte para al menos 500 usuarios concurrentes durante el pico de carga anual (Q4).
- La API debe soportar consultas de al menos 5.000 establecimientos sin degradación.

### 4.6 Seguridad

- Autenticación vía SSO corporativo.
- Roles diferenciados: establecimiento (edita solo su tienda), HR mercado (edita tiendas de su mercado), soporte (acceso total con auditoría), solo lectura (sistemas consumidores).
- Todas las comunicaciones cifradas (HTTPS/TLS 1.3).
- Logs de auditoría inmutables para cambios en horarios.
- Cumplimiento GDPR en lo relativo a datos de usuarios (no se almacenan datos personales de clientes finales).

---

## 5. Exclusiones

| # | Exclusión | Razón | Versión futura |
|---|-----------|-------|---------------|
| 1 | Integración bidireccional con sistema de nóminas/turnos | Complejidad técnica y organizativa excesiva para V1. Requiere coordinación con otro equipo de producto. | Sí |
| 2 | Sugerencia automática de festivos por IA según ubicación | Funcionalidad de segunda fase. Requiere dataset fiable de festivos locales que no existe. | Sí |
| 3 | Flujo de coordinación/aprobación entre equipos comerciales y HR para festivos | Es un problema de proceso, no solo de herramienta. Se resuelve provisionalmente con herramientas existentes (Teams). | Por evaluar |
| 4 | Notificaciones push/alertas automatizadas por plazos | Útil pero no crítico para V1. Se puede gestionar por canales existentes (email, Teams). | Sí |
| 5 | Gestión de personal/turnos vinculada al horario comercial | Fuera del scope: es dominio de otra aplicación (workforce management). | No |
| 6 | App nativa iOS/Android | La web responsive cubre la necesidad de acceso desde iPad/tablet. | Por evaluar |
| 7 | Migración automática de datos históricos de tiendas reubicadas | Casuística edge. Se resuelve con re-entrada manual en los pocos casos que ocurran. | Por evaluar |

---

## 6. KPIs de éxito

| # | KPI | Definición | Método de medición | Target | Plazo |
|---|-----|------------|-------------------|--------|-------|
| 1 | Migración completada | % de establecimientos activos con horarios 2027 cargados en la nueva herramienta | Query a base de datos | 100% | Octubre 2026 |
| 2 | Reducción de incidencias | Tickets L2 mensuales relacionados con horarios comerciales | Sistema de ticketing del Support Center | -80% vs baseline Q4 2025 | 3 meses post-lanzamiento |
| 3 | Tiempo de carga anual | Minutos promedio para completar la configuración de un año nuevo por establecimiento | Telemetría de la aplicación | < 10 min para 80% de establecimientos | Primera carga completa |
| 4 | Adopción | % de establecimientos que usan la nueva herramienta vs los que siguen necesitando soporte manual | Telemetría + datos de soporte | > 90% autónomos | 2 meses post-lanzamiento |
| 5 | Satisfacción de usuario | Puntuación SUS recogida tras primera carga | Encuesta in-app | ≥ 75 | Post primera carga |
| 6 | Disponibilidad de dato para Google | % de establecimientos con horario correcto visible en Google Maps | Monitorización Physical Store/DSU | > 99% | Continuo post-lanzamiento |

---

## 7. Riesgos

| # | Riesgo | Probabilidad | Impacto | Plan de mitigación |
|---|--------|-------------|---------|-------------------|
| 1 | Un solo developer es insuficiente para entregar V1 completa antes de octubre 2026 | Media | Alto | Priorización agresiva de Must vs Should. Entregas incrementales. Primer milestone funcional en S8 para validar velocidad real. |
| 2 | Resistencia al cambio por parte de tiendas acostumbradas al flujo M7 | Media | Medio | Diseño que replica la lógica mental del usuario (no la del sistema). Modo guiado. Formación mínima necesaria. |
| 3 | Sistemas downstream no preparados para la nueva API durante la transición | Media | Alto | Mantener compatibilidad con ETL legacy durante transición. Comunicar cambios con antelación (canal Teams 39 integrantes). Rollout por mercados. |
| 4 | Casuísticas de país no contempladas emergen durante el desarrollo | Alta | Medio | Rollout piloto con 2-3 mercados representativos antes del despliegue global. Buffer de 4 semanas antes del deadline. |
| 5 | Ausencia de datos baseline para medir mejora | Media | Bajo | Obtener métricas actuales de ticketing y tiempos de soporte antes del lanzamiento (acción inmediata). |
| 6 | Pérdida de datos durante la migración del sistema legacy | Baja | Alto | Exportación completa de datos legacy antes de iniciar migración. Período de coexistencia con ambos sistemas activos. |
| 7 | Franquicias no adoptan la herramienta por falta de soporte/formación | Media | Medio | Flujo guiado específico. Documentación de autoservicio. Piloto con Nathan (Paris) como usuario de referencia. |

---

## 8. Roadmap de alto nivel

### 8.1 Hitos principales

| Hito | Descripción | Semana objetivo | Dependencias | Entregable |
|------|-------------|----------------|-------------|------------|
| H1 | Diseño UX y arquitectura técnica | S1-S3 | Ninguna | Wireframes, modelo de datos, decisiones de stack |
| H2 | Core funcional: horarios + festivos + excepciones | S4-S10 | H1 | App funcional con flujo principal (CU-01 a CU-04) |
| H3 | Multi-país y roles | S8-S12 | H2 parcial | Soporte de mercados, roles, i18n |
| H4 | Integración API y sincronización | S10-S14 | H2 | API REST, compatibilidad ETL, endpoint DSU |
| H5 | Panel de soporte y corrección masiva | S13-S16 | H2, H4 | Panel Large Format (CU-05) |
| H6 | Piloto con mercados seleccionados | S16-S18 | H2, H3, H4 | Despliegue en 2-3 mercados piloto, feedback |
| H7 | Ajustes post-piloto y rollout global | S18-S22 | H6 | Correcciones, rollout progresivo, descomisión M7 |

### 8.2 Diagrama de Gantt (por semanas)

```
Semana:        S1  S2  S3  S4  S5  S6  S7  S8  S9  S10 S11 S12 S13 S14 S15 S16 S17 S18 S19 S20 S21 S22
──────────────────────────────────────────────────────────────────────────────────────────────────────────
H1 - Diseño    ████████████
H2 - Core              ████████████████████████████
H3 - Multi-país                        ████████████████████
H4 - API                                   ████████████████████
H5 - Soporte                                           ████████████████
H6 - Piloto                                                        ████████████
H7 - Rollout                                                               ████████████████████
```

**Supuestos del roadmap:**

- Equipo: 1 fullstack developer con conocimientos de diseño.
- Dedicación: full-time.
- Sin contar festivos ni vacaciones.
- Las estimaciones asumen paralelización parcial de tareas donde es posible (H3 y H4 solapan con final de H2).
- 22 semanas ≈ 5.5 meses. Inicio estimado en mayo 2026, cierre en octubre 2026 (alineado con deadline).
- Los requisitos "Should" se incorporan durante H6-H7 si la velocidad real lo permite. Si no, se posponen a iteración posterior.
- Las estimaciones son orientativas y se revisarán al inicio de cada hito.
