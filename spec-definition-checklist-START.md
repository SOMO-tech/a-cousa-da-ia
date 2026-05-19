# Spec Definition Checklist

> Tracking de generación de specs para cada RF del PRD.
> Orden de implementación por waves (dependencias técnicas + valor).
> Fecha de inicio: 12 de mayo de 2026.

---

## Wave 1: Modelo de datos core

| # | Código | Nombre | Criticidad | Spec generada | Archivo | Notas |
|---|--------|--------|------------|---------------|---------|-------|
| 1 | RF-HOR-002 | Período único con excepciones | Must | ☐ | [SPEC-001](specs/SPEC-001-periodo-unico-con-excepciones.md) | Decisión arquitectónica fundacional |
| 2 | RF-HOR-004 | Horario por día de la semana | Must | ☐ | [SPEC-002](specs/SPEC-002-horario-por-dia-de-la-semana.md) | Estructura mínima de franjas |
| 3 | RF-HOR-005 | Horario partido | Must | ☐ | — | Múltiples franjas por día |
| 4 | RF-HOR-003 | Períodos múltiples opcionales | Must | ☐ | — | Extensión para casos estacionales |
| 5 | RF-UXI-001 | Lenguaje comprensible | Must | ☐ | — | Decisión de diseño transversal |
| 6 | RF-EXC-004 | Prevención de disponibilidad 24/7 | Must | ☐ | — | Constraint de validación en el modelo |

---

## Wave 2: Festivos y excepciones

| # | Código | Nombre | Criticidad | Spec generada | Archivo | Notas |
|---|--------|--------|------------|---------------|---------|-------|
| 7 | RF-FES-001 | Registro unificado de festivos | Must | ☐ | — | Elimina dualidad M3/M7 |
| 8 | RF-FES-002 | Herencia de horario en festivos | Must | ☐ | — | Reduce fricción en festivos aperturables |
| 9 | RF-FES-004 | Festivos con horario diferenciado | Must | ☐ | — | Caso Nochebuena y similares |
| 10 | RF-EXC-001 | Excepciones como modificación puntual | Must | ☐ | — | Día con horario diferente sin ser festivo |
| 11 | RF-EXC-002 | Cierre temporal por obras/emergencia | Must | ☐ | — | Rango de fechas cerrado |
| 12 | RF-EXC-003 | Edición in situ de excepciones | Must | ☐ | — | Evitar eliminar y recrear |

---

## Wave 3: Visualización, copia interanual y validación

| # | Código | Nombre | Criticidad | Spec generada | Archivo | Notas |
|---|--------|--------|------------|---------------|---------|-------|
| 13 | RF-UXI-002 | Vista de calendario anual | Must | ☐ | — | Vista principal del usuario |
| 14 | RF-UXI-004 | Edición directa desde calendario | Should | ☐ | — | Click en día → editar |
| 15 | RF-HOR-001 | Copia automática interanual | Must | ☐ | — | Feature de mayor ahorro de tiempo |
| 16 | RF-HOR-006 | Validación de cobertura anual | Must | ☐ | — | Alerta visual, no bloqueo |
| 17 | RF-FES-005 | Visualización horario cierre festivos | Should | ☐ | — | Detalle para planificar recogida |

---

## Wave 4: Multi-país y carga centralizada

| # | Código | Nombre | Criticidad | Spec generada | Archivo | Notas |
|---|--------|--------|------------|---------------|---------|-------|
| 18 | RF-MPA-001 | Configuración por mercado/país | Must | ☐ | — | Estructura organizativa |
| 19 | RF-MPA-002 | Festivos locales/regionales | Must | ☐ | — | Granularidad sub-nacional |
| 20 | RF-MPA-003 | Soporte multi-idioma | Must | ☐ | — | ES, IT, EN, FR, NL |
| 21 | RF-FES-003 | Carga centralizada por mercado | Must | ☐ | — | HR carga festivos en bulk |

---

## Wave 5: Integración con sistemas downstream

| # | Código | Nombre | Criticidad | Spec generada | Archivo | Notas |
|---|--------|--------|------------|---------------|---------|-------|
| 22 | RF-INT-001 | API para sistemas consumidores | Must | ☐ | — | Contrato de datos estable |
| 23 | RF-INT-003 | Compatibilidad con ETL legacy | Must | ☐ | — | Coexistencia durante transición |
| 24 | RF-INT-002 | Sincronización menor latencia | Should | ☐ | — | Mejora sobre ETL nocturno |
| 25 | RF-INT-004 | Recepción desde apps locales | Should | ☐ | — | Input externo de festivos |

---

## Wave 6: Soporte, monitorización y polish

| # | Código | Nombre | Criticidad | Spec generada | Archivo | Notas |
|---|--------|--------|------------|---------------|---------|-------|
| 26 | RF-ADM-001 | Panel de monitorización | Must | ☐ | — | Vista global para Large Format |
| 27 | RF-ADM-002 | Corrección masiva | Should | ☐ | — | Actuación a escala |
| 28 | RF-ADM-003 | Historial de cambios | Should | ☐ | — | Auditoría y rollback |
| 29 | RF-HOR-007 | Indicador de estado actual | Should | ☐ | — | "Actualmente abierto hasta..." |
| 30 | RF-EXC-005 | Detección de conflictos | Should | ☐ | — | Alertar solapamientos |
| 31 | RF-UXI-003 | Flujo guiado para franquicias | Should | ☐ | — | Capa UX sobre flujo existente |

---

## Resumen de progreso

| Wave | Total RFs | Specs generadas | % completado |
|------|-----------|-----------------|--------------|
| 1 — Modelo core | 6 | 0 | 0% |
| 2 — Festivos/excepciones | 6 | 0 | 0% |
| 3 — Calendario/copia | 5 | 0 | 0% |
| 4 — Multi-país | 4 | 0 | 0% |
| 5 — Integración | 4 | 0 | 0% |
| 6 — Soporte/polish | 6 | 0 | 0% |
| **Total** | **31** | **0** | **0%** |

---

## Convenciones

- **Archivo de spec:** `specs/SPEC-[código-RF].md` (ej: `specs/SPEC-RF-HOR-002.md`)
- **Spec generada:** ☐ pendiente / ☑ completada
- **Notas:** contexto relevante, decisiones tomadas durante la generación, dependencias descubiertas
