# a cousa da IA — Ejemplos de flujo Exploración → PRD → Specs con IA

Este repositorio reúne **artefactos de ejemplo** producidos al aplicar un flujo de trabajo asistido por IA (Claude Code + skills `somo-*`) sobre un caso real: la sustitución de un sistema legacy de **gestión de horarios comerciales** en una red retail multi-país.

El objetivo no es entregar un producto, sino **mostrar el formato y la calidad de los artefactos generados en cada fase** del flujo: research → exploración → producto → especificación.

---

## El flujo en tres fases

```
research bruto   ──/somo-explore──▶  exploration-report.md
exploration      ──/somo-plan─────▶  prd.md  +  spec-definition-checklist.md
PRD + checklist  ──/somo-spec─────▶  specs/SPEC-NNN-*.md  (una por requisito funcional)
```

Cada skill consume el output de la anterior y produce el siguiente artefacto, manteniendo trazabilidad de extremo a extremo entre cita de entrevista, requisito funcional y criterio de aceptación Gherkin.

---

## Contenido del repositorio

### Fase 1 - Material de research (input)

- **`interviews/`** — 8 entrevistas con usuarios finales (responsables de tienda, backoffice, store managers) en distintos mercados europeos.
- **`other references/`** — 6 entrevistas con stakeholders (producto, HR, soporte) + el manual legacy de la aplicación M7 como observación.

Todo el material está **anonimizado** y adaptado a partir de fuentes reales para servir como ejemplo.

### Fase 2 — Exploración (output de `/somo-explore`)

- **`exploration-report.md`** — Informe consolidado de research: perfiles afectados, problema, casuísticas, sistema actual, propuestas emergentes y recomendaciones. Incluye matriz de trazabilidad a las 15 fuentes.

### Fase 3 — Producto (output de `/somo-plan`)

- **`prd.md`** — PRD completo derivado del exploration report: contexto, personas, casos de uso, requisitos funcionales (RF) categorizados con criticidad must/should/could, requisitos no funcionales, KPIs, riesgos y roadmap.
- **`spec-definition-checklist.md`** — Tracking en waves de qué RFs están convertidos a spec y cuáles quedan pendientes.
- **`spec-definition-checklist-START.md`** — Snapshot inicial del checklist antes de empezar a generar specs.

### Fase 4 — Especificaciones (output de `/somo-spec` / `/somo-create-spec`)

- **`specs/`** — 23 specs detalladas (SPEC-001 a SPEC-026), una por RF. Cada spec incluye:
  - Descripción del requisito y contexto
  - Criterios de aceptación en Gherkin (GIVEN/WHEN/THEN)
  - Diseño UX con wireframe textual y componentes shadcn/ui
  - Comportamiento responsive
  - Notas técnicas y dependencias

---

## Skills utilizadas

| Skill | Fase | Qué hace |
|---|---|---|
| `somo-explore` | Research → Exploration | Analiza una carpeta de research markdown y produce un informe estructurado de conclusiones |
| `somo-plan` | Exploration → PRD | Genera un PRD completo a partir del exploration report |
| `somo-create-spec` / `somo-spec` | PRD → Specs | Convierte cada RF del PRD en una spec individual con ACs en Gherkin, UX y notas técnicas |

Las skills viven en `~/.claude/skills/` y se invocan desde Claude Code con `/somo-explore`, `/somo-plan`, `/somo-spec`.

---

## Cómo leer este repo

1. Empieza por `exploration-report.md` para entender el problema sin tener que leer las 15 fuentes originales.
2. Pasa a `prd.md` para ver cómo el problema se convierte en requisitos accionables.
3. Abre `spec-definition-checklist.md` y entra a cualquier spec del listado para ver el nivel de detalle de salida.
4. Si quieres ver el input original, mira `interviews/` y `other references/`.

---

## Caso de uso de referencia

Aurora — sustituto de M7/M3 para gestionar horarios comerciales de apertura en una red retail con tiendas propias y franquicias en 6 mercados europeos. Datos, nombres y referencias internas son ficticios.
