# Horarios de apertura de tiendas
Mar, 16 Sept 25

Valentina y Sofía forman parte del equipo denominado "Large Format", cuya función principal consiste en ejecutar diversas labores administrativas y operativas de carácter repetitivo en beneficio de los establecimientos del grupo. Una de sus responsabilidades habituales es la verificación y registro de los calendarios de apertura dentro de la herramienta vigente.

### Estado actual del sistema M7 (Calendarios de apertura)
- Las alertas del sistema han mejorado notablemente: se pasó de unas 400 incidencias al inicio a apenas 40 en la última semana
- Se han puesto en marcha acciones preventivas:
	- Análisis retrospectivo de los periodos registrados en M7
	- Carga anticipada de horarios para evitar problemas futuros
	- Verificación directa con cada establecimiento a través de la herramienta 07 para minimizar alertas
- Los establecimientos propios operan sin mayores inconvenientes
- Las franquicias presentan más complicaciones:
	- Piden guías paso a paso para poder recordar los procedimientos
	- Existe confusión generalizada sobre cómo realizar la carga de datos
### Deficiencias detectadas en M7
- Los horarios estándar deben introducirse manualmente cada año
	- La mayoría de establecimientos mantienen horarios regulares que apenas varían entre un año y otro
	- Este proceso debería automatizarse, dejando solo la posibilidad de introducir modificaciones puntuales
- Se mezclan los conceptos de excepciones y días festivos:
	- Algunos establecimientos registran aperturas en festivos como si fueran excepciones
	- Los cierres por festivo se introducen erróneamente como excepciones de cierre
- Incidencia técnica importante: 28 establecimientos presentan excepciones que generan disponibilidad continua (24/7)
	- El sistema duplica las excepciones de forma automática día tras día
	- La corrección requiere eliminar cada registro de forma individual
	- El origen del problema son franjas de excepción registradas sin un horario concreto asociado
### Propuesta para la nueva herramienta
- Modelo simplificado basado en 4 bloques:
	- Horario habitual de lunes a domingo (copiado automáticamente del año anterior)
	- Festivos de cierre (sin posibilidad de modificar el horario)
	- Festivos con apertura (donde sí se puede ajustar el horario)
	- Cierres temporales por causas especiales (obras, reformas, etc.)
- Eliminación de terminología confusa como "excepción de apertura"
- Diseño de interfaz con lenguaje comprensible para el personal de tienda, evitando tecnicismos
- Mecanismos de prevención para los errores que genera el sistema actual de forma automática