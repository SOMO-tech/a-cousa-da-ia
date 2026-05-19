# SPEC-005 — Lenguaje comprensible

> RF: RF-UXI-001
> Criticidad: Must
> Wave: 1 — Modelo de datos core

## Descripción

Define la terminología oficial que la aplicación debe usar en todos los textos visibles al usuario (labels, placeholders, mensajes de error, tooltips, empty states). El objetivo es eliminar los tecnicismos heredados del sistema legacy (M3/M7) y sustituirlos por lenguaje de negocio comprensible para personal de tienda sin formación técnica. Esta spec es transversal: no tiene pantalla propia, sino que establece reglas que aplican a todas las demás specs.

## Criterios de aceptación

### Glosario de términos

- GIVEN cualquier pantalla de la aplicación WHEN se renderiza texto visible al usuario THEN no aparece ninguno de los siguientes términos legacy: "excepción de apertura", "excepción de cierre", "excepción de modificación de horario", "tipo de excepción", "código de excepción", "franja horaria tipo N", "slot", "time slot", "período de cierre tipo X".
- GIVEN la interfaz muestra un día que el establecimiento abre en una fecha normalmente cerrada (festivo) WHEN el usuario ve el label THEN el término usado es "Festivo de apertura", no "Excepción de apertura".
- GIVEN la interfaz muestra un día de cierre por festivo WHEN el usuario ve el label THEN el término usado es "Festivo de cierre" o simplemente "Festivo (cerrado)", no "Excepción de cierre" ni "Cierre festivo tipo 1".
- GIVEN la interfaz muestra un día con horario diferente al habitual por causa puntual (no festivo) WHEN el usuario ve el label THEN el término usado es "Cambio puntual de horario" o "Horario especial", no "Excepción de modificación de horario".
- GIVEN la interfaz muestra un cierre temporal por obras, emergencia o reforma WHEN el usuario ve el label THEN el término usado es "Cierre temporal", no "Excepción de cierre continuado" ni "Período de cierre extraordinario".
- GIVEN la interfaz muestra las horas de apertura y cierre de un día WHEN el usuario ve el label THEN el término usado es "Horario" o "Franja horaria", no "Slot", "Time slot" ni "Franja tipo N".
- GIVEN la interfaz muestra el tramo de fechas durante el cual el horario semanal se mantiene constante WHEN el usuario ve el label THEN el término usado es "Período", no "Bloque temporal" ni "Intervalo de vigencia".
- GIVEN la interfaz muestra un día sin horario asignado WHEN el usuario ve el estado THEN el término usado es "Cerrado", no "Sin disponibilidad", "No operativo" ni "Inactivo".

### Reglas de redacción

- GIVEN un mensaje de error se muestra al usuario WHEN el error contiene referencias a campos técnicos THEN el mensaje usa nombres visibles del campo, no nombres de columna de base de datos (ej: "La hora de cierre debe ser posterior a la hora de apertura", no "end_time must be > start_time").
- GIVEN un placeholder de campo de texto se muestra WHEN el usuario ve el campo THEN el placeholder usa un ejemplo concreto del dominio (ej: "Ej: Horario de verano", no "Introduzca valor").
- GIVEN un empty state se muestra WHEN no hay datos THEN el mensaje describe la situación en lenguaje de negocio y sugiere la acción siguiente (ej: "No hay horario comercial configurado para 2026. Crea el horario anual para empezar.", no "No se encontraron registros en la tabla periods").
- GIVEN un tooltip explicativo se muestra WHEN el usuario hover sobre un elemento THEN el texto explica el concepto en una frase corta sin jerga técnica.

### Consistencia

- GIVEN el mismo concepto aparece en dos pantallas diferentes WHEN el usuario navega entre ellas THEN el término usado es idéntico en ambas (no "Franja horaria" en una y "Horario" en otra para referirse al mismo concepto).
- GIVEN la aplicación soporta múltiples idiomas (RF-MPA-003) WHEN se traduce un término THEN la traducción sigue el mismo principio de lenguaje de negocio, no de traducción literal del tecnicismo.

### Verificación

- GIVEN un revisor audita los textos de la aplicación WHEN compara contra el glosario oficial THEN puede verificar cada término visible contra la tabla de mapeo definida en esta spec.

## Glosario oficial

| Concepto | Término oficial (usar siempre) | Términos prohibidos (no usar nunca) |
|----------|-------------------------------|--------------------------------------|
| Día festivo en que la tienda abre | Festivo de apertura | Excepción de apertura, Apertura extraordinaria |
| Día festivo en que la tienda cierra | Festivo de cierre, Festivo (cerrado) | Excepción de cierre, Cierre festivo tipo 1 |
| Día con horario diferente al habitual | Cambio puntual de horario, Horario especial | Excepción de modificación, Excepción tipo 3 |
| Cierre por obras, emergencia o reforma | Cierre temporal | Excepción de cierre continuado, Cierre extraordinario |
| Horas de apertura/cierre de un día | Horario, Franja horaria | Slot, Time slot, Franja tipo N |
| Tramo de fechas con horario constante | Período | Bloque temporal, Intervalo de vigencia |
| Día sin horario asignado | Cerrado | Sin disponibilidad, No operativo, Inactivo |
| Día festivo en general | Festivo | Excepción festiva, Holiday exception |

## UX Design

### Wireframe textual

Esta spec no tiene pantalla propia. El glosario y las reglas de redacción se aplican transversalmente a todas las pantallas definidas en las demás specs (SPEC-001 a SPEC-031). El SW Dev debe consultar la tabla de glosario al implementar cualquier texto visible.

### Componentes shadcn utilizados

No requiere componentes adicionales. Los textos se aplican sobre los componentes ya definidos en las specs correspondientes.

### Patrón de interacción

No aplica. Esta spec define contenido textual, no patrones de interacción.

### Comportamiento responsive

No aplica. Los textos se adaptan automáticamente al breakpoint del componente que los contiene.

## Notas técnicas

- Los términos del glosario deben externalizarse en un fichero de traducciones (i18n) desde el inicio, aunque la primera versión sea solo en español. Esto facilita la implementación posterior de RF-MPA-003 (multi-idioma).
- El fichero de traducciones debe usar claves semánticas basadas en el concepto, no en el texto literal: `label.holiday.open` en lugar de `festivo_de_apertura`. Esto permite cambiar el texto sin refactorizar claves.
- Las validaciones de mensajes de error deben usar el fichero de traducciones, no strings hardcodeados en el código de validación.
- Esta spec no tiene dependencias upstream. Es transversal y aplica a todas las specs existentes y futuras.
- Las specs anteriores (SPEC-001 a SPEC-004) ya usan terminología alineada con este glosario. Esta spec formaliza y documenta las decisiones tomadas implícitamente.
