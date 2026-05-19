# Spec Definition Checklist

> Tracking de generación de specs para cada RF del PRD.
> Orden de implementación por waves (dependencias técnicas + valor).
> Fecha de inicio: 12 de mayo de 2026.

---

## Wave 1: Modelo de datos core

| # | Código | Nombre | Criticidad | Spec generada | Archivo | Notas |
|---|--------|--------|------------|---------------|---------|-------|
| 1 | RF-HOR-002 | Período único con excepciones | Must | ☑ | [SPEC-001](specs/SPEC-001-periodo-unico-con-excepciones.md) | Decisión arquitectónica fundacional |
| 2 | RF-HOR-004 | Horario por día de la semana | Must | ☑ | [SPEC-002](specs/SPEC-002-horario-por-dia-de-la-semana.md) | Estructura mínima de franjas |
| 3 | RF-HOR-005 | Horario partido | Must | ☑ | [SPEC-003](specs/SPEC-003-horario-partido.md) | Múltiples franjas por día |
| 4 | RF-HOR-003 | Períodos múltiples opcionales | Must | ☑ | [SPEC-004](specs/SPEC-004-periodos-multiples-opcionales.md) | Dividir/fusionar períodos |
| 5 | RF-UXI-001 | Lenguaje comprensible | Must | ☑ | [SPEC-005](specs/SPEC-005-lenguaje-comprensible.md) | Glosario oficial + reglas de redacción |
| 6 | RF-EXC-004 | Prevención de disponibilidad 24/7 | Must | ☑ | [SPEC-006](specs/SPEC-006-prevencion-disponibilidad-24-7.md) | Constraint client+server, detección legacy |

---

## Wave 2: Festivos y excepciones

| # | Código | Nombre | Criticidad | Spec generada | Archivo | Notas |
|---|--------|--------|------------|---------------|---------|-------|
| 7 | RF-FES-001 | Registro unificado de festivos | Must | ☑ | [SPEC-007](specs/SPEC-007-registro-unificado-de-festivos.md) | Flujo único con herencia de horario |
| 8 | RF-FES-002 | Herencia de horario en festivos | Must | ☑ | [SPEC-008](specs/SPEC-008-herencia-de-horario-en-festivos.md) | Resolución dinámica + conflictos |
| 9 | RF-FES-004 | Festivos con horario diferenciado | Must | ☑ | [SPEC-009](specs/SPEC-009-festivos-con-horario-diferenciado.md) | Visualización + comparación + pre-relleno |
| 10 | RF-EXC-001 | Excepciones como modificación puntual | Must | ☑ | [SPEC-010](specs/SPEC-010-cambio-puntual-de-horario.md) | Cambio puntual + cierre puntual |
| 11 | RF-EXC-002 | Cierre temporal por obras/emergencia | Must | ☑ | [SPEC-011](specs/SPEC-011-cierre-temporal.md) | Rango de fechas + finalización anticipada |
| 12 | RF-EXC-003 | Edición in situ de excepciones | Must | ☑ | [SPEC-012](specs/SPEC-012-edicion-in-situ.md) | Estandariza edición en Dialog para festivos, cambios puntuales y cierres |

---

## Wave 3: Visualización, copia interanual y validación

| # | Código | Nombre | Criticidad | Spec generada | Archivo | Notas |
|---|--------|--------|------------|---------------|---------|-------|
| 13 | RF-UXI-002 | Vista de calendario anual | Must | ☑ | [SPEC-013](specs/SPEC-013-vista-calendario-anual.md) | 12 meses, dots de estado, tooltip, estadísticas, scroll al mes actual |
| 14 | RF-UXI-004 | Edición directa desde calendario | Should | ☑ | [SPEC-014](specs/SPEC-014-edicion-directa-desde-calendario.md) | Click en celda → Dialog o Popover contextual |
| 15 | RF-HOR-001 | Copia automática interanual | Must | ☑ | [SPEC-015](specs/SPEC-015-copia-automatica-interanual.md) | Copia atómica con previsualización, checkboxes por categoría, alert festivos variables |
| 16 | RF-HOR-006 | Validación de cobertura anual | Must | ☑ | [SPEC-016](specs/SPEC-016-validacion-cobertura-anual.md) | Triple detección: barra + calendario + Alert. No bloqueante |
| 17 | RF-FES-005 | Visualización horario cierre festivos | Should | ☑ | [SPEC-017](specs/SPEC-017-visualizacion-horario-cierre-festivos.md) | Enfatiza "Cierra a las X" en lista, tooltip, Dialog y comparación |

---

## Wave 4: Multi-país y carga centralizada

| # | Código | Nombre | Criticidad | Spec generada | Archivo | Notas |
|---|--------|--------|------------|---------------|---------|-------|
| 18 | RF-MPA-001 | Configuración por mercado/país | Must | ☑ | [SPEC-018](specs/SPEC-018-configuracion-por-mercado-pais.md) | 6 mercados, 48 regiones, jerarquía 3 niveles, seed completa |
| 19 | RF-MPA-002 + RF-FES-003 | Festivos multi-nivel y carga centralizada | Must | ☑ | [SPEC-020](specs/SPEC-020-festivos-multi-nivel-y-carga-centralizada.md) | Fusión: 3 scopes (mercado/región/establecimiento), propagación automática, override local, vista HR |
| 20 | RF-MPA-003 | Soporte multi-idioma | Must | ☑ | [SPEC-021](specs/SPEC-021-soporte-multi-idioma.md) | 5 idiomas, cadena resolución (usuario→mercado→es), selector Globe, Intl API, glosario multi-idioma |
| 21 | RF-ADM-004 | Sistema de roles y permisos | Must | ☑ | [SPEC-019](specs/SPEC-019-sistema-de-roles-y-permisos.md) | RBAC + RLS. 5 roles, scope por mercado/establecimiento |

---

## Wave 5: Integración con sistemas downstream

| # | Código | Nombre | Criticidad | Spec generada | Archivo | Notas |
|---|--------|--------|------------|---------------|---------|-------|
| 22 | RF-INT-001 | API para sistemas consumidores | Must | ☑ | [SPEC-022](specs/SPEC-022-api-para-sistemas-consumidores.md) | REST versionada, OAuth 2.0, 6 endpoints, datos resueltos, delta feed, rate limiting |
| 23 | RF-INT-003 | Compatibilidad con ETL legacy | Must | N/A | — | No aplica: no existe sistema legacy en este caso de uso |
| 24 | RF-INT-002 | Sincronización menor latencia | Should | N/A | — | No aplica: no existe sistema legacy en este caso de uso |
| 25 | RF-INT-004 | Recepción desde apps locales | Should | N/A | — | No aplica: no existe sistema legacy en este caso de uso |

---

## Wave 6: Soporte, monitorización y polish

| # | Código | Nombre | Criticidad | Spec generada | Archivo | Notas |
|---|--------|--------|------------|---------------|---------|-------|
| 26 | RF-ADM-001 | Panel de monitorización | Must | ☑ | [SPEC-026](specs/SPEC-026-panel-de-monitorizacion.md) | Sección independiente, landing page admin/soporte, 4 KPIs, tabla con alertas expandibles, navegación a establecimiento |
| 27 | RF-ADM-002 | Corrección masiva | Should | ☐ | — | Actuación a escala |
| 28 | RF-ADM-003 | Historial de cambios | Should | ☐ | — | Auditoría y rollback |
| 29 | RF-HOR-007 | Indicador de estado actual | Should | ☐ | — | "Actualmente abierto hasta..." |
| 30 | RF-EXC-005 | Detección de conflictos | Should | ☐ | — | Alertar solapamientos |
| 31 | RF-UXI-003 | Flujo guiado para franquicias | Should | ☐ | — | Capa UX sobre flujo existente |

---

## Resumen de progreso

| Wave | Total RFs | Specs generadas | % completado |
|------|-----------|-----------------|--------------|
| 1 — Modelo core | 6 | 6 | 100% |
| 2 — Festivos/excepciones | 6 | 6 | 100% |
| 3 — Calendario/copia | 5 | 5 | 100% |
| 4 — Multi-país | 4 | 4 | 100% |
| 5 — Integración | 4 (3 N/A) | 1 | 100% |
| 6 — Soporte/polish | 6 | 1 | 17% |
| **Total** | **28** | **23** | **82%** |

---

## Convenciones

- **Archivo de spec:** `specs/SPEC-[código-RF].md` (ej: `specs/SPEC-RF-HOR-002.md`)
- **Spec generada:** ☐ pendiente / ☑ completada
- **Notas:** contexto relevante, decisiones tomadas durante la generación, dependencias descubiertas
