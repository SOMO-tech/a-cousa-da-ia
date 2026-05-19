# Horarios de Apertura Comercial - Situación Actual
Mié, 06 Ago 25

Marta y Alberto forman parte del equipo de producto y cuentan con cierta experiencia funcional sobre el sistema en uso actualmente.

### Contexto Técnico del Sistema Vigente
- Sistema heredado de gestión de horarios comerciales cuya base de datos legacy database alcanza fin de vida en 2026
- Construido sobre un legacy framework descatalogado desde 2014, sin mantenimiento posible
- Únicamente operativo en HRS 07; incompatible con dispositivos iPad o Mac
- El acceso requiere navegadores Windows a través de RemoteAccess
<>
### Lógica Funcional y Flujo de Trabajo
- Las responsables de tienda registran desviaciones respecto al horario estándar, no el horario base en sí
- La configuración de horarios se organiza por periodos estacionales (temporada estival e invernal)
- El tratamiento de festivos varía según el mercado:
	- Mercados con app de horarios propia: el dato se introduce en la aplicación local y llega mediante proceso ETL al sistema de horarios comerciales
	- Mercados sin app de horarios: el registro se realiza directamente en el sistema de horarios comerciales
- La sincronización se ejecuta mediante un proceso ETL en lote nocturno, sin conexión en tiempo real

### Sistemas que Consumen los Datos
- Google (a través de Physical Store, DSU y un microservicio dedicado)
- Módulo de stock para gestionar entregas express
- Funcionalidades de experiencia en tienda (reserva de probadores, click and try)
- Diversos sistemas de terceros conectados mediante API OAuth

### Incidencias y Problemas Detectados
- El módulo aparece erróneamente dentro de la sección de HR del HRS
- Desde el punto de vista técnico, debería ser un dato nativo gestionado por la DSU
- Provoca un volumen elevado de incidencias que gestiona el Support Center L2
- El equipo de Large Format interviene como enlace para asistir a las tiendas

### Situación de las Franchises
- Están obligadas a utilizar el sistema de horarios comerciales para toda la gestión, festivos incluidos
- Carecen de aplicaciones propias para la gestión de horarios
- Por cláusula contractual, GlobalTextile no administra al personal de las franchises

### Coordinación y Canales de Comunicación
- Existe un canal de Teams con 39 integrantes destinado a informar sobre cambios en la API
- Es necesario depurar y reactivar dicho canal antes de iniciar la migración Aurora
- Marco facilitará el listado de clientes OAuth para planificar la transición
- Hitos de fin de vida relevantes: 31 de enero, 31 de octubre, 3 de abril