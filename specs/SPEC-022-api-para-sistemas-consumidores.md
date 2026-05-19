# SPEC-022 — API para sistemas consumidores

> RF: RF-INT-001
> Criticidad: Must
> Wave: 5 — Integración con sistemas downstream

## Descripción

Define la API REST pública que expone los datos de horario comercial de Aurora a los sistemas downstream: Google (vía Physical Store / DSU), módulo de stock, experiencia en tienda, entregas express y terceros autorizados vía OAuth. La API proporciona un contrato de datos estable, versionado y desacoplado de la estructura interna de Aurora. Los consumidores reciben datos resueltos (horarios concretos, festivos con franjas calculadas, cierres aplicados), nunca referencias internas ni lógica de herencia.

## Criterios de aceptación

### Autenticación y autorización

- GIVEN un sistema consumidor intenta acceder a la API WHEN no incluye un token OAuth 2.0 válido THEN la API responde con HTTP 401 Unauthorized y un cuerpo JSON con `error: "unauthorized"` y `message` descriptivo.
- GIVEN un sistema consumidor presenta un token OAuth 2.0 válido WHEN el token tiene scope `aurora:read` THEN la API permite acceso a todos los endpoints de lectura.
- GIVEN un sistema consumidor presenta un token válido WHEN el token no tiene scope `aurora:read` THEN la API responde con HTTP 403 Forbidden.
- GIVEN un sistema consumidor se autentica WHEN accede a la API THEN solo puede consultar datos, nunca crear, modificar ni eliminar (la API es de solo lectura para consumidores externos).
- GIVEN la API está operativa WHEN se consulta el endpoint de health check (`GET /api/v1/health`) THEN responde HTTP 200 sin requerir autenticación, con cuerpo `{ "status": "ok", "version": "1.x.x" }`.

### Versionado

- GIVEN la API se despliega WHEN un consumidor consulta cualquier endpoint THEN la URL incluye el prefijo de versión `/api/v1/`.
- GIVEN existe una versión v1 estable WHEN se introduce un cambio que rompe compatibilidad (breaking change) THEN se publica como `/api/v2/` manteniendo v1 operativa durante un período de deprecación mínimo de 6 meses.
- GIVEN una versión está deprecada WHEN un consumidor la consulta THEN la respuesta incluye el header `Deprecation: true` y `Sunset: [fecha]` con la fecha de retirada.

### Endpoint: horario completo de un establecimiento

- GIVEN un establecimiento activo con configuración completa WHEN se consulta `GET /api/v1/establishments/{establishment_id}/schedule?year={year}` THEN la respuesta incluye: datos del establecimiento (código, nombre, mercado, región, zona horaria), períodos del año con sus franjas horarias por día de la semana, festivos con tipo (cierre/apertura) y franjas resueltas (nunca `inherit_period_schedule: true`), cambios puntuales con tipo y franjas, y cierres temporales con rango de fechas y motivo.
- GIVEN un festivo de apertura hereda horario del período (SPEC-008) WHEN se consulta vía API THEN el endpoint devuelve las franjas concretas del día de la semana correspondiente, no la referencia al período. El campo `schedule_source` indica `"inherited"` o `"custom"`.
- GIVEN un festivo fue heredado del mercado o región (SPEC-020) WHEN se consulta vía API THEN el endpoint devuelve el festivo con el campo `scope` indicando `"market"`, `"region"` o `"establishment"`, y `overridden: true/false` si el establecimiento lo personalizó.
- GIVEN un establecimiento tiene un cierre temporal activo WHEN se consulta el horario del día afectado THEN el cierre temporal tiene precedencia sobre el horario base, los festivos y los cambios puntuales.

### Endpoint: estado operativo de un establecimiento (snapshot)

- GIVEN un sistema necesita saber si un establecimiento está abierto ahora WHEN consulta `GET /api/v1/establishments/{establishment_id}/status` THEN la respuesta incluye: `is_open` (boolean), `current_date` y `current_time` (en la zona horaria del establecimiento), `today_schedule` con las franjas del día resueltas (considerando período base, festivos, cambios puntuales y cierres), `next_opening` (fecha y hora de la próxima apertura si está cerrado) y `next_closing` (hora de cierre si está abierto).
- GIVEN un establecimiento tiene un cierre temporal activo WHEN se consulta su estado THEN `is_open` es `false`, `today_schedule` muestra `"closure"` con el motivo, y `next_opening` apunta al día siguiente al fin del cierre (si tiene horario definido).
- GIVEN un establecimiento no tiene ningún período definido para la fecha actual WHEN se consulta su estado THEN `is_open` es `false` y `today_schedule` indica `"no_schedule_defined"`.

### Endpoint: listado de establecimientos

- GIVEN un consumidor necesita descubrir establecimientos WHEN consulta `GET /api/v1/establishments` THEN la respuesta incluye una lista paginada de establecimientos con: código, nombre, mercado, región, zona horaria, tipología (propia/franquicia) y estado (activo/inactivo).
- GIVEN el listado puede contener miles de establecimientos WHEN se consulta sin parámetros THEN la paginación es server-side con cursor-based pagination: parámetros `limit` (default 50, máximo 200) y `cursor` (opaque string). La respuesta incluye `next_cursor` (null si es la última página) y `total_count`.
- GIVEN un consumidor necesita filtrar establecimientos WHEN incluye query params THEN soporta: `market` (código ISO, ej: `market=ES`), `region` (código, ej: `region=ES-MA`), `status` (active/inactive, default: active), `type` (own/franchise). Los filtros son combinables con AND.

### Endpoint: listado de festivos por mercado

- GIVEN un sistema necesita los festivos de un mercado WHEN consulta `GET /api/v1/markets/{market_code}/holidays?year={year}` THEN la respuesta incluye todos los festivos de scope mercado y región para ese año, agrupados por scope: primero nacionales (`scope: "market"`), luego regionales (`scope: "region"`, con `region_code`).
- GIVEN un mercado tiene regiones con festivos propios WHEN se consulta con el parámetro `region={region_code}` THEN la respuesta filtra los festivos regionales a esa región, manteniendo los nacionales.

### Endpoint: cambios recientes (delta feed)

- GIVEN un sistema necesita sincronizar cambios incrementalmente WHEN consulta `GET /api/v1/changes?since={ISO8601_timestamp}` THEN la respuesta incluye todos los registros creados, modificados o eliminados desde esa fecha/hora: períodos, festivos, cambios puntuales, cierres temporales y establecimientos. Cada entrada incluye `entity_type`, `entity_id`, `action` (created/updated/deleted), `changed_at` y `establishment_id`.
- GIVEN se solicitan cambios con un `since` anterior a 30 días WHEN se consulta el endpoint THEN la API responde HTTP 400 con mensaje `"since parameter must be within the last 30 days. Use the full schedule endpoint for historical data"`.
- GIVEN no hay cambios desde el timestamp indicado WHEN se consulta el delta feed THEN la respuesta es HTTP 200 con `changes: []` y `latest_timestamp` con la fecha/hora del último cambio conocido.

### Formato de respuesta

- GIVEN cualquier endpoint de la API se consulta WHEN la respuesta es exitosa THEN el formato es JSON con content-type `application/json; charset=utf-8`.
- GIVEN una respuesta contiene fechas WHEN se serializa THEN usa formato ISO 8601: fechas como `"2026-12-25"`, horas como `"10:00"` (sin segundos, formato 24h), timestamps como `"2026-12-25T10:00:00Z"` (siempre UTC para timestamps, hora local para horarios de tienda).
- GIVEN una respuesta contiene horarios de un establecimiento WHEN se serializa THEN las horas de apertura y cierre se expresan en hora local del establecimiento (no UTC). El campo `timezone` del establecimiento permite al consumidor convertir si lo necesita.
- GIVEN una respuesta exitosa contiene una lista WHEN se serializa THEN el cuerpo sigue la estructura: `{ "data": [...], "pagination": { "total_count": N, "next_cursor": "..." } }`.
- GIVEN una respuesta es un error WHEN se serializa THEN el cuerpo sigue la estructura: `{ "error": "código_error", "message": "Descripción legible", "details": {} }`.

### Rate limiting

- GIVEN un consumidor autenticado WHEN realiza peticiones THEN la API permite un máximo de 100 peticiones por minuto por token.
- GIVEN un consumidor supera el rate limit WHEN realiza una petición adicional THEN la API responde HTTP 429 Too Many Requests con headers `Retry-After` (segundos hasta reset), `X-RateLimit-Limit` (100) y `X-RateLimit-Remaining` (0).
- GIVEN cualquier petición autenticada WHEN la respuesta se envía THEN incluye headers `X-RateLimit-Limit`, `X-RateLimit-Remaining` y `X-RateLimit-Reset` (timestamp UTC del próximo reset).

### Rendimiento

- GIVEN la API está operativa WHEN un consumidor consulta el estado de un establecimiento THEN el tiempo de respuesta es inferior a 200ms en el percentil 95.
- GIVEN la API está operativa WHEN un consumidor consulta el horario completo de un establecimiento THEN el tiempo de respuesta es inferior a 500ms en el percentil 95.
- GIVEN la API debe soportar carga WHEN múltiples consumidores consultan simultáneamente THEN soporta al menos 500 peticiones concurrentes sin degradación (requisito 4.5 del PRD: 5.000 establecimientos consultables).

### Disponibilidad

- GIVEN la API alimenta sistemas críticos (Google, stock, entregas) WHEN se mide la disponibilidad THEN el uptime es igual o superior al 99.5% mensual (excluidas ventanas de mantenimiento planificadas, máximo 2h/mes con aviso previo de 48h).
- GIVEN la API experimenta un error interno WHEN un consumidor realiza una petición THEN responde HTTP 503 Service Unavailable con `Retry-After` header, nunca un timeout silencioso.

### Documentación

- GIVEN la API está desplegada WHEN un desarrollador de un sistema consumidor necesita integrarse THEN existe documentación OpenAPI 3.0 (Swagger) accesible en `/api/v1/docs` que describe todos los endpoints, parámetros, esquemas de respuesta, códigos de error y ejemplos.
- GIVEN la documentación OpenAPI existe WHEN se consulta THEN incluye: descripción de cada endpoint, esquemas JSON tipados para request y response, ejemplos de petición y respuesta por endpoint, lista de códigos de error con descripción, y guía de autenticación OAuth 2.0.

### Edge cases

- GIVEN un `establishment_id` no existe WHEN se consulta cualquier endpoint con ese ID THEN la API responde HTTP 404 Not Found con `error: "not_found"` y `message: "Establishment not found"`.
- GIVEN un establecimiento está inactivo WHEN se consulta su horario completo THEN la API responde normalmente (los datos existen) pero incluye el campo `status: "inactive"` en la respuesta. El consumidor decide si usar o ignorar establecimientos inactivos.
- GIVEN un establecimiento no tiene ningún período definido para el año solicitado WHEN se consulta su horario completo THEN la respuesta incluye `periods: []`, `holidays: [...]` (los festivos heredados del mercado/región sí aparecen) y un campo `warnings: ["no_periods_defined"]`.
- GIVEN se consulta un año futuro sin datos WHEN el consumidor solicita `year=2028` THEN la respuesta es válida con arrays vacíos. No es un error.
- GIVEN un festivo fue overrideado localmente (SPEC-020) WHEN la API devuelve el festivo THEN incluye los campos `original_type` y `original_name` del festivo heredado, además de los valores locales, para que el consumidor tenga trazabilidad.
- GIVEN la base de datos está temporalmente inalcanzable WHEN un consumidor consulta la API THEN responde HTTP 503 con `Retry-After: 30` y `error: "service_unavailable"`. Nunca expone detalles internos de la base de datos en el mensaje de error.

## Notas técnicas

- La API se implementa como Supabase Edge Functions o como un servicio independiente que consulta la base de datos de Aurora. La decisión de arquitectura depende del equipo de desarrollo, pero el contrato de la API (esta spec) es independiente de la implementación.
- La resolución de herencia de horarios (SPEC-008) y la aplicación de precedencia (cierre temporal > cambio puntual > festivo > período base) se ejecuta en la capa de servicio de la API, no en el cliente consumidor. Los consumidores reciben datos planos y resueltos.
- El endpoint de cambios (`/changes`) requiere que las tablas operativas tengan columnas `created_at` y `updated_at` con triggers automáticos. Para deletes, se necesita una tabla de auditoría o soft-delete. Esto conecta con RF-ADM-003 (historial de cambios, Wave 6).
- El campo `schedule_source` en festivos de apertura (`"inherited"` o `"custom"`) es metadata para el consumidor. Permite a Google o stock saber si el horario fue explícitamente confirmado por la tienda o resuelto automáticamente.
- Cursor-based pagination usa un campo ordenable internamente (ej: `id` o `created_at`). El cursor es opaco para el consumidor (base64 del valor interno). Esto es más eficiente que offset-based para datasets grandes.
- El rate limit de 100 req/min es un punto de partida conservador. Se puede ajustar por consumidor usando scopes OAuth adicionales (ej: `aurora:read:high_volume` para Google/DSU que necesita polling frecuente).
- La zona horaria del establecimiento (SPEC-018) es crítica para la correcta interpretación de los horarios. La API siempre incluye el campo `timezone` (ej: `"Europe/Madrid"`) junto a cualquier dato horario.
- Dependencia upstream: SPEC-001 (períodos), SPEC-002 y SPEC-003 (franjas), SPEC-007 y SPEC-008 (festivos y herencia), SPEC-010 (cambios puntuales), SPEC-011 (cierres temporales), SPEC-018 (mercados y zonas horarias), SPEC-019 (rol `api_readonly`), SPEC-020 (festivos multi-nivel y scopes).
- Dependencia downstream: RF-INT-002 (sincronización baja latencia, puede usar el delta feed como base), RF-INT-003 (ETL legacy, puede consumir esta API como fuente).
