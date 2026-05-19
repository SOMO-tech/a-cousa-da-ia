# Clara: gestión de horarios comerciales
Mar, 16 Sep 25

Clara trabaja en el equipo de HR y se encarga de la administración de personal en determinadas tiendas del sur de Italy, abarcando todas las marcas de cadena excepto Vanguard.

### Diagnóstico del sistema actual de horarios comerciales
- La aplicación vigente se basa en una tecnología extremadamente obsoleta
- Existe una migración obligatoria con fecha límite en octubre de 2026, impuesta por GroupTex
- El objetivo inicial era disponer de una nueva herramienta para enero-febrero de 2025
- Principales deficiencias identificadas:
	- La interfaz resulta tan poco intuitiva que las responsables necesitan releer el manual cada vez que la usan tras un periodo sin acceder
	- Se produce duplicación de tareas al tener que registrar festivos tanto en M3 como en M7 por separado
	- No existe integración entre el sistema de nóminas y el de horarios comerciales
	- Cuando una tienda cambia de ubicación, se pierde todo el histórico de información asociado

### Casuísticas operativas según tipología de tienda
- **Centros comerciales** (complejidad baja):
	- La normativa limita a 12 los festivos con apertura autorizada
	- Funcionan con horario continuo de 10:00 a 22:00, todos los días del año
	- La información queda disponible desde principio de año
- **Tiendas 365** (complejidad media):
	- Operan todos los días salvo dos excepciones (1 de mayo y 25 de diciembre)
	- Sus franjas horarias habituales son razonablemente estables
- **Tiendas de calle en tourist attraction zone** (complejidad alta):
	- Los festivos con apertura en zonas de alta afluencia turística varían con frecuencia
	- Los equipos comerciales coordinan la decisión aproximadamente un mes antes del festivo
	- Actualmente se utiliza un chat de Teams para consensuar los horarios de apertura en festivos
	- La información se encuentra dispersa entre distintas provincias y localidades

### Propuestas de mejora y calendario previsto
- **Prioridad máxima:** renovar la aplicación M7 con una interfaz más accesible e intuitiva
	- Permitir la recuperación automática de los horarios del ejercicio anterior
	- Generar propuestas automáticas de festivos de apertura en función de la ubicación de la tienda
	- Suprimir la necesidad de registrar datos en paralelo en M3 y M7
- **Mejoras de segunda fase** (posterior al lanzamiento inicial):
	- Crear un ecosistema de coordinación entre el equipo comercial y Clara para la gestión de festivos
	- Incorporar gestión de plazos con notificaciones automatizadas
	- Integrar capacidades de IA para la consulta de festivos a nivel provincial y local
- **Calendario crítico:** es imprescindible tenerlo operativo antes de diciembre para evitar la duplicación de trabajo
	- La carga de horarios para 2026 comienza a finales del cuarto trimestre de 2025
	- Sin horario comercial registrado, la tienda no aparece con horario en Google
