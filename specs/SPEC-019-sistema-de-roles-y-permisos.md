# SPEC-019 — Sistema de roles y permisos

> RF: RF-ADM-004
> Criticidad: Must
> Wave: 4 — Multi-país y carga centralizada

## Descripción

Define el sistema de control de acceso basado en roles (RBAC) que determina qué puede ver y hacer cada usuario en Aurora. El research identifica cuatro perfiles con necesidades y permisos radicalmente distintos: el responsable de tienda que solo gestiona su establecimiento, el equipo HR que administra un mercado completo, el equipo de soporte (Large Format) que necesita acceso global para resolver incidencias, y el administrador que gestiona la estructura organizativa. Sin este sistema, las RLS de base de datos son permisivas y cualquier usuario autenticado puede acceder a datos de cualquier establecimiento.

## Criterios de aceptación

### Modelo de roles

- GIVEN el sistema define roles WHEN se consulta la lista de roles disponibles THEN existen exactamente 5 roles: `establecimiento` (gestiona sus establecimientos asignados), `hr_mercado` (gestiona todos los establecimientos de sus mercados asignados), `soporte` (acceso de lectura y escritura a todos los establecimientos de todos los mercados, con auditoría), `administrador` (gestión de estructura organizativa, usuarios y roles) y `api_readonly` (acceso de solo lectura vía API, sin acceso a la interfaz).
- GIVEN un usuario tiene el rol `establecimiento` WHEN accede a la aplicación THEN solo ve y puede editar los establecimientos que tiene asignados. No tiene visibilidad sobre otros establecimientos, ni sobre la estructura organizativa (mercados, regiones).
- GIVEN un usuario tiene el rol `hr_mercado` WHEN accede a la aplicación THEN ve todos los establecimientos de los mercados que tiene asignados, agrupados por región. Puede editar horarios, festivos y excepciones de cualquier establecimiento de sus mercados.
- GIVEN un usuario tiene el rol `soporte` WHEN accede a la aplicación THEN ve todos los establecimientos de todos los mercados, con filtros por mercado y región. Puede editar horarios, festivos y excepciones de cualquier establecimiento.
- GIVEN un usuario tiene el rol `administrador` WHEN accede a la aplicación THEN tiene acceso al panel de administración (mercados, regiones, establecimientos, usuarios) además de acceso operativo completo.
- GIVEN un usuario tiene el rol `api_readonly` WHEN intenta acceder a la interfaz web THEN no tiene acceso. Este rol solo opera vía API con permisos de lectura.

### Asignación de scope

- GIVEN un usuario con rol `establecimiento` WHEN un administrador le asigna scope THEN se vinculan uno o más establecimientos al perfil del usuario. El usuario solo accede a esos establecimientos.
- GIVEN un usuario con rol `hr_mercado` WHEN un administrador le asigna scope THEN se vinculan uno o más mercados al perfil del usuario. El usuario accede a todos los establecimientos de esos mercados.
- GIVEN un usuario con rol `soporte` o `administrador` WHEN un administrador configura su perfil THEN no se requiere asignación de scope. El acceso es global.
- GIVEN un usuario con rol `establecimiento` tiene asignados 3 establecimientos WHEN accede al listado THEN ve exactamente esos 3 establecimientos. Si se le desasigna uno, deja de verlo inmediatamente.
- GIVEN un usuario con rol `hr_mercado` tiene asignado el mercado ES WHEN se crea un nuevo establecimiento en ES THEN el usuario automáticamente tiene acceso al nuevo establecimiento (el scope es por mercado, no por establecimiento individual).

### Listado de usuarios (panel de administración)

- GIVEN un administrador accede a la sección Usuarios del panel de administración WHEN la página carga THEN se muestra una tabla con los usuarios del sistema: nombre, email, rol (Badge), mercados/establecimientos asignados (resumen), estado (Badge "Activo" / "Inactivo") y fecha de último acceso.
- GIVEN la tabla de usuarios tiene más de 20 registros WHEN se renderiza THEN se muestra paginación server-side con "Mostrando 1-20 de [total]" y selector de tamaño de página (10, 20, 50).
- GIVEN el administrador usa la barra de búsqueda WHEN escribe texto THEN la tabla filtra por nombre o email (debounce 300ms, server-side).
- GIVEN el administrador usa los filtros WHEN selecciona un rol, un mercado o un estado THEN la tabla se filtra server-side. Cada filtro activo se muestra como Badge con botón cerrar. Botón "Limpiar filtros" visible cuando hay 1+ filtros activos.
- GIVEN no hay usuarios que coincidan con los filtros aplicados WHEN la tabla se renderiza THEN se muestra empty state: "No se encontraron usuarios con estos filtros" con botón "Limpiar filtros" (variant outline).
- GIVEN no hay ningún usuario registrado WHEN la tabla se renderiza THEN se muestra empty state: icono Users + "No hay usuarios registrados" + botón "Crear primer usuario" (variant default).
- GIVEN el fetch de usuarios falla WHEN la tabla intenta cargar THEN se muestra error state con icono de error + "No se pudieron cargar los usuarios" + botón "Reintentar" (variant outline).

### Creación de usuario

- GIVEN un administrador hace clic en "Nuevo usuario" WHEN se abre el Sheet THEN se muestran los campos: nombre completo (Input, requerido), email corporativo (Input, requerido, validación de formato email), rol (Select con las 5 opciones), estado (Switch, activo por defecto).
- GIVEN el administrador selecciona el rol `establecimiento` WHEN el campo de rol cambia THEN aparece un campo adicional: "Establecimientos asignados" (Combobox multi-select, busca por código o nombre, server-side).
- GIVEN el administrador selecciona el rol `hr_mercado` WHEN el campo de rol cambia THEN aparece un campo adicional: "Mercados asignados" (Select multi-select con los mercados activos, client-side porque son ≤10).
- GIVEN el administrador selecciona el rol `soporte`, `administrador` o `api_readonly` WHEN el campo de rol cambia THEN no aparece campo de scope (acceso global o sin UI).
- GIVEN el administrador rellena todos los campos y hace clic en "Crear usuario" WHEN la creación es exitosa THEN se cierra el Sheet, se muestra Toast "Usuario creado", y la tabla se actualiza mostrando el nuevo usuario.
- GIVEN el administrador introduce un email que ya existe WHEN hace clic en "Crear usuario" THEN se muestra error inline en el campo email: "Ya existe un usuario con este email".
- GIVEN el campo nombre está vacío WHEN el administrador sale del campo THEN se muestra error inline: "El nombre es obligatorio".
- GIVEN el campo email tiene formato inválido WHEN el administrador sale del campo THEN se muestra error inline: "Introduce un email válido".

### Edición de usuario

- GIVEN un administrador hace clic en "Editar" en el DropdownMenu de un usuario WHEN se abre el Sheet THEN se muestran los mismos campos que en creación, pre-rellenados con los datos actuales.
- GIVEN el administrador cambia el rol de `hr_mercado` a `establecimiento` WHEN el campo de rol cambia THEN el campo de scope cambia de "Mercados asignados" a "Establecimientos asignados". Los mercados asignados previamente se desvinculan al guardar.
- GIVEN el administrador modifica datos y hace clic en "Guardar cambios" WHEN la actualización es exitosa THEN se cierra el Sheet, se muestra Toast "Cambios guardados", y la tabla se actualiza.
- GIVEN el administrador intenta cambiar su propio rol WHEN edita su perfil THEN el campo de rol está deshabilitado con Tooltip: "No puedes cambiar tu propio rol".

### Desactivación de usuario

- GIVEN un administrador hace clic en "Desactivar" en el DropdownMenu de un usuario WHEN se muestra el AlertDialog THEN el título dice "Desactivar usuario", la descripción dice "El usuario [nombre] dejará de tener acceso a Aurora. Podrás reactivarlo en cualquier momento." con botones "Cancelar" (variant outline) y "Desactivar" (variant destructive).
- GIVEN el administrador confirma la desactivación WHEN la operación es exitosa THEN el Badge de estado cambia a "Inactivo" (gris), se muestra Toast "Usuario desactivado", y el usuario desactivado pierde acceso inmediato a la aplicación.
- GIVEN un usuario está inactivo WHEN el administrador hace clic en "Activar" en el DropdownMenu THEN el estado cambia a "Activo" directamente (sin AlertDialog, porque activar no es destructivo) y se muestra Toast "Usuario activado".
- GIVEN un administrador intenta desactivar su propia cuenta WHEN la opción aparece en el DropdownMenu THEN la opción "Desactivar" está deshabilitada con Tooltip: "No puedes desactivar tu propia cuenta".

### Control de acceso en datos operativos (RLS)

- GIVEN un usuario con rol `establecimiento` asignado al establecimiento A WHEN consulta períodos, festivos, excepciones o cierres temporales THEN solo recibe datos del establecimiento A. Una consulta sin filtro de establishment_id devuelve solo datos del establecimiento A.
- GIVEN un usuario con rol `hr_mercado` asignado al mercado ES WHEN consulta datos operativos THEN recibe datos de todos los establecimientos del mercado ES. No recibe datos de establecimientos de otros mercados.
- GIVEN un usuario con rol `soporte` WHEN consulta datos operativos THEN recibe datos de todos los establecimientos de todos los mercados.
- GIVEN un usuario con rol `establecimiento` asignado al establecimiento A WHEN intenta crear un período para el establecimiento B THEN la operación es rechazada por la base de datos. El usuario recibe un error genérico (no se expone que el establecimiento B existe).
- GIVEN un usuario con rol `hr_mercado` asignado al mercado ES WHEN intenta modificar un festivo del mercado IT THEN la operación es rechazada.
- GIVEN un usuario con rol `api_readonly` WHEN intenta crear, modificar o eliminar cualquier dato THEN la operación es rechazada. Solo las operaciones de lectura (SELECT) están permitidas.

### Protección del panel de administración

- GIVEN un usuario con rol distinto de `administrador` WHEN intenta acceder a las rutas de administración (/admin/*) THEN es redirigido a la página principal con Toast: "No tienes permisos para acceder a esta sección".
- GIVEN un usuario con rol `administrador` WHEN accede a /admin THEN ve el sidebar de administración (Mercados, Regiones, Establecimientos, Usuarios) definido en SPEC-018.

### Edge cases

- GIVEN un usuario con rol `establecimiento` no tiene ningún establecimiento asignado WHEN accede a la aplicación THEN ve un empty state global: "No tienes establecimientos asignados. Contacta con tu administrador para solicitar acceso."
- GIVEN un usuario con rol `hr_mercado` no tiene ningún mercado asignado WHEN accede a la aplicación THEN ve el mismo empty state: "No tienes mercados asignados. Contacta con tu administrador para solicitar acceso."
- GIVEN el último administrador activo WHEN otro administrador intenta desactivarlo THEN se muestra error: "No se puede desactivar al último administrador activo del sistema".
- GIVEN un administrador cambia el mercado de un establecimiento (SPEC-018) WHEN usuarios con rol `establecimiento` tenían ese establecimiento asignado THEN mantienen la asignación (el vínculo es por establishment_id, no por mercado).
- GIVEN un administrador desactiva un mercado (SPEC-018) WHEN usuarios con rol `hr_mercado` tenían ese mercado asignado THEN mantienen la asignación, pero el mercado inactivo no muestra establecimientos operativos.

## UX Design

### Wireframe textual

**Panel de administración: Usuarios (pantalla nueva, integrada en el sidebar de SPEC-018)**

Layout 4 — Settings (panel de administración con sub-navegación).

Zona de título:
- Breadcrumb: Administración > Usuarios.
- Heading "Usuarios" (h2).
- Botón "Nuevo usuario" (Button variant default, icono Plus) a la derecha del heading.

Zona de filtros (debajo del título):
- Input de búsqueda (placeholder "Buscar por nombre o email…"), alineado a la izquierda.
- Select de rol (opciones: Todos, Establecimiento, HR Mercado, Soporte, Administrador, API Readonly).
- Select de mercado (opciones: Todos + lista de mercados activos). Solo visible cuando el filtro de rol es "Todos", "Establecimiento" o "HR Mercado".
- Select de estado (opciones: Activos, Inactivos, Todos).
- Badges de filtros activos con botón cerrar. Botón "Limpiar filtros" (Button variant ghost, icono X) visible con 1+ filtros.

Tabla de usuarios:
- Columnas: Nombre (texto), Email, Rol (Badge con colores diferenciados: azul "Establecimiento", verde "HR Mercado", naranja "Soporte", rojo "Administrador", gris "API Readonly"), Scope (texto resumido: "3 establecimientos" o "ES, IT" o "Global" o "—"), Estado (Badge "Activo" verde / "Inactivo" gris), Último acceso (fecha relativa: "Hace 2 días").
- Acciones por fila: DropdownMenu (icono MoreHorizontal) con "Editar", Separator, "Desactivar" (destructive) o "Activar".
- Paginación server-side debajo de la tabla: "Mostrando 1-20 de [total]", botones Anterior/Siguiente, selector de tamaño (10, 20, 50).

**Sheet de creación/edición de usuario (lado derecho)**

Sheet (lado derecho, ancho default ~400px):
- Título: "Nuevo usuario" o "Editar usuario".
- Campos en orden vertical:
  1. Nombre completo (Input, label "Nombre", placeholder "Nombre y apellidos").
  2. Email corporativo (Input, label "Email", placeholder "usuario@empresa.com").
  3. Rol (Select, label "Rol", opciones: Establecimiento, HR Mercado, Soporte, Administrador, API Readonly).
  4. [Condicional] Establecimientos asignados (Combobox multi-select, label "Establecimientos", placeholder "Buscar por código o nombre…"). Visible solo si rol = Establecimiento.
  5. [Condicional] Mercados asignados (Select multi-select, label "Mercados"). Visible solo si rol = HR Mercado.
  6. Estado (Switch, label "Activo", activado por defecto en creación).
- Sticky bottom bar: "Cancelar" (Button variant outline, izquierda) + "Crear usuario" o "Guardar cambios" (Button variant default, derecha).

### Componentes shadcn utilizados

Componentes: Table, Input, Select, Button, Sheet, Badge, Toast, AlertDialog, DropdownMenu, Skeleton, Form, Switch, Tooltip, Breadcrumb.

Componente adicional necesario: Combobox (no instalado en el scaffold base, necesario para búsqueda multi-select de establecimientos).

### Patrón de interacción

- **Tabla con paginación server-side:** el número de usuarios puede superar 100 (múltiples establecimientos × mercados). Filtros y búsqueda server-side para consistencia. (Regla: server-side si >100 filas.)
- **Sheet para creación/edición:** 5-7 campos según el rol. El administrador necesita ver el listado detrás mientras crea/edita. (Regla: Sheet para 5-10 campos cuando el usuario necesita contexto de la página.)
- **Campos condicionales por rol:** el campo de scope (establecimientos o mercados) aparece solo cuando es relevante. Reduce la confusión y el ruido visual. (Decisión de UX: progressive disclosure basado en selección de rol.)
- **Select para mercados (client-side), Combobox para establecimientos (server-side):** los mercados son ≤10, caben en un Select. Los establecimientos pueden ser cientos o miles, requieren búsqueda. (Regla: Select para 3-10 opciones estáticas, Combobox para 10+ o dinámicas.)
- **AlertDialog para desactivación, acción directa para activación:** desactivar revoca acceso inmediato (potencialmente destructivo). Activar restaura acceso (no destructivo). (Regla: AlertDialog antes de acciones irreversibles o con consecuencias significativas.)
- **Badges de color por rol:** diferenciación visual rápida en la tabla. Azul, verde, naranja, rojo y gris para los 5 roles. (Decisión no cubierta por el design system: asignación de colores semánticos a roles. Se resuelve con colores diferenciados pero no semánticos, ya que ningún rol es intrínsecamente "bueno" o "malo".)
- **Toast para todas las acciones mutadoras:** crear, editar, desactivar, activar. (Regla: Toast siempre después de acción mutadora exitosa.)
- **Protección contra auto-modificación:** el administrador no puede cambiar su propio rol ni desactivarse. Se indica con Tooltip en campo/opción disabled. (Regla: Tooltip explicativo para elementos deshabilitados.)

### Comportamiento responsive

- **Mobile (< md):** El panel de administración no está optimizado para mobile (función de back-office, coherente con SPEC-018). Si se accede, la tabla muestra scroll horizontal con primera columna (Nombre) sticky. Los filtros se apilan verticalmente. El Sheet se abre a ancho completo.
- **Tablet (md-lg):** Funcional. Tabla a ancho completo, Sheet a ancho default. Filtros en fila.
- **Desktop (lg+):** Layout con sidebar de administración + contenido como descrito. Sheet a 400px.

## Notas técnicas

- Modelo de datos:
  - Tabla `user_profiles`: `id` (UUID, FK a auth.users), `display_name`, `email`, `role` (enum: 'establecimiento', 'hr_mercado', 'soporte', 'administrador', 'api_readonly'), `active` (boolean, default true), `last_sign_in_at` (timestamp, nullable), `created_at`, `updated_at`.
  - Tabla `user_establishment_assignments`: `id`, `user_id` (FK a user_profiles), `establishment_id` (FK a establishments), `created_at`. Unique constraint en (user_id, establishment_id).
  - Tabla `user_market_assignments`: `id`, `user_id` (FK a user_profiles), `market_id` (FK a markets), `created_at`. Unique constraint en (user_id, market_id).
- Las RLS policies se aplican a todas las tablas de datos operativos: `periods`, `period_slots`, `holidays`, `holiday_slots`, `schedule_overrides`, `override_slots`, `temporary_closures`. Cada policy filtra por el scope del usuario autenticado (vía `auth.uid()` → `user_profiles` → assignments → establishment_id/market_id).
- Las tablas administrativas (`markets`, `regions`, `establishments`, `user_profiles`, `user_*_assignments`) son accesibles solo para rol `administrador` (policy de admin).
- La autenticación se delega a Supabase Auth (SSO corporativo). Esta spec no cubre la configuración de SSO, solo el modelo de roles post-autenticación.
- El campo `last_sign_in_at` se actualiza mediante trigger o función en cada login. Se usa para la columna "Último acceso" de la tabla.
- Dependencia upstream: SPEC-018 (estructura organizativa: markets, regions, establishments). El panel de administración de usuarios se integra en el sidebar ya definido.
- Dependencia downstream: Todas las specs existentes (SPEC-001 a SPEC-018) se benefician de las RLS. RF-ADM-003 (historial de cambios) necesitará el user_id del perfil para registrar quién hizo cada cambio.
