---
name: dev-cycle
description: Workflow autónomo Opus-Sonnet-loop para Claude Code. Opus planifica + audita, Sonnet ejecuta. Aprendizaje persistente entre sesiones (lessons-learned). Reporte de ahorro de tokens al final. Ahorra ~70% en API vs Opus solo, manteniendo calidad.
trigger: /dev-cycle
---

# /dev-cycle — Ciclo de desarrollo Opus + Sonnet

> **ES** primero, **EN** después en este mismo archivo.
> Bilingual skill: Spanish first, English below.

---

# 🇪🇸 Español

## Qué hace esta skill

Patrón refinado en proyectos reales: **Opus planifica + audita, Sonnet ejecuta en paralelo**.

Resultado típico: **~70% de ahorro en tokens** sin perder calidad, gracias a usar Sonnet 4.6 (5× más barato) para la ejecución de specs claras y reservar Opus solo para arquitectura + auditoría.

Además:

- **Memoria de errores persistente**: cada error que Opus le encuentra a Sonnet se documenta en `<repo>/.claude/dev-cycle-lessons.md`. Al inicio del próximo ciclo, esas lecciones se inyectan al spec para que Sonnet NO repita el mismo error.
- **Reporte de costos al final**: cada ciclo termina con un resumen mostrando tokens reales vs estimación si hubiera sido todo Opus.

## Cuándo usar

Trigger explícito: el usuario escribe `/dev-cycle <descripción de la tarea>`.

Buenos casos:
- Implementar un módulo o feature nuevo.
- Arreglar bugs de un backlog (Linear, GitHub Issues, etc.).
- Refactorizar un área del código.
- Completar TODOs pendientes.
- Cualquier tarea de desarrollo que cruce backend + UI.

## Cuándo NO usar

- Cambios triviales (1-2 líneas). Hacelo directo.
- Preguntas/explicaciones/discusiones de estrategia.
- Tareas ya en curso que solo necesitan continuación.

## Prerequisites

- **Claude Code 2.1.139+** (`/goal` y plugin marketplace).
- **Acceso a Opus + Sonnet** (Claude Pro/Team/Max o Anthropic API).
- **claude-mem** instalado globalmente — recomendado pero opcional. Sin él la skill funciona, solo con menos contexto histórico cross-session. Instalación: `npm install -g claude-mem`.
- El proyecto debe tener `CLAUDE.md` describiendo sus convenciones (la skill las respeta).

## El ciclo paso a paso

Cuando se invoca `/dev-cycle <tarea>`, ejecutar EN ORDEN:

### Paso 0 — Refresh de lecciones aprendidas (NUEVO en v1.1)

**ANTES de planificar**, leer dos fuentes:

1. **Lecciones específicas del proyecto**: archivo `<repo-root>/.claude/dev-cycle-lessons.md` si existe. Contiene errores que Sonnet cometió antes en ESTE repo + cómo evitarlos.

2. **Patrones cross-project**: si `claude-mem` está instalado, invocar `claude-mem:mem-search` con `obs_type: change` y `query: "lesson sonnet mistake"` para traer lecciones de otras sesiones.

Si encontrás lecciones relevantes, **incluilas EXPLÍCITAMENTE en el spec que le pasarás a Sonnet** (sección "Common mistakes to avoid en este proyecto"). Esto es lo que evita re-trabajos.

Si el archivo de lecciones no existe, este paso no falla — simplemente continúa.

### Paso 1 — Contexto histórico

Si `claude-mem` está disponible, invocá `claude-mem:mem-search` con keywords de la tarea para detectar:
- Sesiones paralelas trabajando en algo cercano.
- Decisiones grabadas que afecten el approach.
- Módulos/tablas/RPCs similares que se puedan reutilizar.

Si no está disponible, este paso se omite silenciosamente.

### Paso 2 — Mapeo de codebase

Antes de planificar:
- `Grep` de keywords en directorios relevantes.
- Identificá el módulo existente más cercano (lo usás como patrón).
- Listá archivos exactos a modificar.

Leé el `CLAUDE.md` del proyecto si no lo leíste. Respetá sus convenciones estrictamente.

### Paso 3 — Plan estructurado con TaskList

Creá tasks con `TaskCreate`:
- Una task por unidad lógica de trabajo (no por archivo individual).
- Cada task con descripción detallada: schema, archivos exactos, server actions, pages, verificaciones.
- Si la tarea es grande (>4h estimadas), dividila en bloques que puedan correr en paralelo.

### Paso 4 — Delegar a Sonnet sub-agents

Lanzá `Agent` con:
- `subagent_type`: el más apropiado al dominio.
- **`model: "sonnet"`** SIEMPRE (acá está el ahorro).
- Prompt MUY explícito con:
  - Contexto técnico del proyecto (reglas del CLAUDE.md).
  - Patrones a copiar (referencias a archivos existentes).
  - Schema completo si aplica.
  - Lista exacta de archivos a crear/modificar.
  - **Lecciones aprendidas** del paso 0 (sección "Common mistakes to avoid").
  - Verificaciones obligatorias al final.
  - **"NO hagas commits — el coordinador (Opus) los hace al final"**.
  - **"NO me preguntes nada — sos autónomo"**.

Si la tarea se puede paralelizar, lanzá MÚLTIPLES agents en UNA SOLA llamada.

### Paso 5 — Auditar el entregable

Cuando el agent reporta, revisar:

1. Verificaciones automáticas reportadas (typecheck, lint, tests, build).
2. Calidad del código (sample inspection de 1-2 archivos críticos).
3. Verificación independiente: correr typecheck/lint/build localmente.
4. Anti-patterns comunes específicos del proyecto.

### Paso 6 — Iterar + documentar nuevas lecciones (NUEVO en v1.1)

Si la auditoría encontró problemas:

1. Continuar con el mismo agent vía `SendMessage` (retiene contexto, más eficiente).
2. **CRÍTICO**: cada error que Sonnet cometió debe documentarse en `<repo>/.claude/dev-cycle-lessons.md`. Si el archivo no existe, crealo con el template:

```markdown
# Dev Cycle — Lessons Learned

> Auto-mantenido por la skill /dev-cycle. Cada vez que Opus encuentra un error
> en el deliverable de Sonnet, se documenta acá para que futuras invocaciones
> de la skill eviten repetirlo.
>
> Edición manual permitida — el usuario puede agregar lecciones también.

## Lessons

### YYYY-MM-DD [Category]
**Error:** descripción del error que cometió Sonnet.
**Fix:** cómo se debería haber hecho.
**Context:** archivo/módulo donde apareció.
```

Categories sugeridas: `[TypeScript]`, `[Database]`, `[Auth]`, `[RLS]`, `[i18n]`, `[Build]`, `[Migration]`, `[UI]`, `[Performance]`, `[Security]`.

3. Repetir auditoría. Loop hasta calidad aprobada.

Si tras 2-3 iteraciones el agent no resuelve, **arreglalo vos (Opus) directamente** y documentá la lección igual.

### Paso 7 — Commit + push (si es repo git)

Solo cuando la auditoría está limpia:

```bash
git add <archivos específicos, NO -A>
git commit -m "..." (Conventional Commits)
git push
```

### Paso 8 — Actualizar memoria persistente

Si la tarea fue significativa:
- Update/crear doc en `docs/MEMORY/` o donde el proyecto lo defina.
- Asegurate que `dev-cycle-lessons.md` esté actualizado con TODOS los errores nuevos.

### Paso 9 — Reporte final + métricas de costos (NUEVO en v1.1)

Mensaje final al usuario con DOS partes:

**Parte A — Qué se hizo** (1-3 líneas):
- Lo entregado.
- Verificaciones pasadas.
- Commit hash + push status (si aplica).
- Pendientes (si los hay).

**Parte B — Costos del ciclo** (tabla obligatoria):

```
### 💰 Resumen de costos del ciclo

| Capa | Tokens (estimado) | Costo USD (estimado) |
|---|---|---|
| Opus (planificación + auditoría) | ~XX,XXX | ~$X.XX |
| Sonnet (Y sub-agents) | ~XXX,XXX | ~$X.XX |
| **Total real estimado** | **~XXX,XXX** | **~$X.XX** |
| Si todo hubiera sido Opus (≈2.5×) | ~XXX,XXX | ~$XX.XX |
| **Ahorrado** | **~XXX,XXX** | **~$XX.XX (XX%)** |

Tiempo: XX min · Sub-agents lanzados: X · Iteraciones de audit: X · Lecciones nuevas documentadas: X
```

**Cómo estimar tokens (honestamente)**:

- **Tokens de Sonnet**: sumar `total_tokens` de los `<usage>...</usage>` que cada Agent reporta al volver.
- **Tokens de Opus**: estimación aproximada. Usar la regla `~30k tokens por turn de Opus` (input + output, mix típico 60/40). Si el ciclo tuvo 5 turns de Opus → ~150k tokens.
- **Costo USD**: aplicar tarifas Anthropic vigentes:
  - Opus 4.7 (1M context): $15/MTok input + $75/MTok output → ~$39/MTok con mix 60/40.
  - Sonnet 4.6: $3/MTok input + $15/MTok output → ~$7.80/MTok.
- **Multiplicador Opus-solo**: usar **2.5×** como factor conservador. Opus solo genera más output por turn y suele requerir más iteraciones de verificación manual.
- Aclarar "estimado" siempre. No mentir precisión que no tenés.

## Combinación con `/goal`

Si la tarea tiene una condición de éxito clara y verificable:

```
/dev-cycle implementar módulo X
/goal módulo X compila, tests pasan, typecheck/lint OK
```

El evaluador (Haiku por default) revisa cada turn. La skill sigue trabajando hasta que la condición se cumple.

## Anti-patterns a evitar

**NO**:
- Lanzar Agent Sonnet sin spec detallado → resultado pobre.
- Saltar el paso 0 (lecciones) → re-cometer errores ya conocidos.
- Saltar `claude-mem` si está disponible → posible re-trabajo de algo ya hecho en sesión paralela.
- Hacer commit sin auditar → bugs en producción.
- Usar `model: "opus"` en sub-agents → anula todo el ahorro.

**SÍ**:
- Specs explícitos a Sonnet con schema completo + archivos exactos + lecciones.
- Documentar CADA error de Sonnet en `dev-cycle-lessons.md`. Es lo más valioso del ciclo.
- Typecheck DESPUÉS de cada bloque grande.
- Reporte de tokens claro al final.

## Diseño agnóstico de stack

La skill funciona en cualquier stack porque la metodología es universal:
- Next.js / React / Vue / Svelte.
- Node / Python / Go / Rust.
- Mobile (React Native, Flutter).
- Con o sin claude-mem.
- Con o sin agentes custom en `.claude/agents/`.

Se respeta lo que el `CLAUDE.md` del proyecto defina. No se imponen patrones — se amplifica lo que el proyecto ya tiene, delegando ejecución eficientemente.

---

# 🇬🇧 English

## What this skill does

Pattern refined in real production projects: **Opus plans + audits, Sonnet executes in parallel**.

Typical result: **~70% token savings** without losing quality, thanks to using Sonnet 4.6 (5× cheaper) for executing clear specs while reserving Opus only for architecture + audit.

Plus:

- **Persistent error memory**: every mistake Opus catches in Sonnet's deliverable is documented in `<repo>/.claude/dev-cycle-lessons.md`. On the next cycle, those lessons are injected into the spec so Sonnet doesn't repeat them.
- **Cost report at the end**: every cycle finishes with a summary showing real tokens vs estimated all-Opus cost.

## When to use

User explicitly types `/dev-cycle <task description>`.

Good for: new modules/features, backlog bugs, refactors, TODOs, anything backend + UI.

## When NOT to use

Trivial 1-2 line changes, questions/explanations, ongoing tasks needing only continuation.

## Prerequisites

- Claude Code 2.1.139+ (`/goal` and plugin marketplace).
- API access to Opus + Sonnet.
- `claude-mem` globally (recommended, optional): `npm install -g claude-mem`.
- Project `CLAUDE.md` defining conventions.

## The cycle step by step

### Step 0 — Refresh learned lessons (NEW in v1.1)

Before planning, read:

1. **Project-specific lessons**: `<repo-root>/.claude/dev-cycle-lessons.md` if exists.
2. **Cross-project patterns**: if `claude-mem` is installed, search for past lessons.

Include relevant lessons explicitly in the spec for Sonnet (section "Common mistakes to avoid in this project"). This is what prevents re-work.

### Step 1 — Historical context

Use `claude-mem:mem-search` to detect parallel sessions, recorded decisions, reusable modules.

### Step 2 — Codebase mapping

`Grep` for keywords, identify closest existing pattern, list exact files. Read project `CLAUDE.md`.

### Step 3 — Structured plan with TaskList

One task per logical unit, detailed descriptions, parallelizable blocks.

### Step 4 — Delegate to Sonnet sub-agents

`Agent` with `model: "sonnet"` (the savings lever) and very explicit prompt including the lessons from step 0.

### Step 5 — Audit the deliverable

Automated checks + sample inspection + independent verification + anti-patterns.

### Step 6 — Iterate + document new lessons (NEW in v1.1)

Continue same agent with `SendMessage`. **Document EVERY mistake** in `dev-cycle-lessons.md`:

```markdown
### YYYY-MM-DD [Category]
**Error:** what Sonnet did wrong.
**Fix:** how it should have been done.
**Context:** file/module where it appeared.
```

### Step 7 — Commit + push

Specific files (not `-A`), Conventional Commits.

### Step 8 — Update persistent memory

Update memory docs + ensure lessons file is current with all new errors.

### Step 9 — Final report + cost metrics (NEW in v1.1)

**Part A — What got done** (1-3 lines): deliverables, verifications, commit, pending.

**Part B — Cycle costs** (mandatory table):

```
### 💰 Cycle cost summary

| Layer | Tokens (estimate) | Cost USD (estimate) |
|---|---|---|
| Opus (planning + audit) | ~XX,XXX | ~$X.XX |
| Sonnet (Y sub-agents) | ~XXX,XXX | ~$X.XX |
| **Estimated total** | **~XXX,XXX** | **~$X.XX** |
| If all Opus (≈2.5×) | ~XXX,XXX | ~$XX.XX |
| **Saved** | **~XXX,XXX** | **~$XX.XX (XX%)** |

Time: XX min · Sub-agents launched: X · Audit iterations: X · New lessons documented: X
```

**How to estimate tokens honestly**:
- Sonnet tokens: sum `<usage>total_tokens</usage>` from each Agent return.
- Opus tokens: rough estimate `~30k tokens per Opus turn`. Multiply by turns.
- USD cost: apply current Anthropic rates.
  - Opus 4.7 (1M): $15/MTok input + $75/MTok output → ~$39/MTok with 60/40 mix.
  - Sonnet 4.6: $3/MTok input + $15/MTok output → ~$7.80/MTok.
- Multiplier for all-Opus: use **2.5×** (Opus generates more output and needs more verification iterations).
- Always say "estimate". Don't fake precision you don't have.

## Combining with `/goal`

```
/dev-cycle implement module X
/goal X compiles, tests pass, typecheck/lint clean
```

Evaluator (Haiku by default) checks each turn. Skill keeps working until met.

## Anti-patterns

**DON'T**: launch Sonnet without detailed spec; skip step 0 (lessons); commit without audit; use `model: "opus"` on sub-agents.

**DO**: explicit specs with full schema + exact files + lessons; document every mistake in `dev-cycle-lessons.md`; typecheck after each large block; clean token report at the end.

## Stack-agnostic by design

Works on any tech stack. Respects each project's `CLAUDE.md`. Doesn't impose patterns — amplifies what the project already has by delegating efficiently.

---

## Changelog

### v1.1.0 (2026-05-23)
- **NEW Step 0**: persistent lessons-learned. Reads `<repo>/.claude/dev-cycle-lessons.md` at start, injects relevant lessons into Sonnet spec.
- **NEW Step 6 enhancement**: every Sonnet mistake gets documented in lessons file.
- **NEW Step 9**: final report includes token + cost breakdown vs all-Opus estimate.
- Documentation bilingual (ES + EN in same file).

### v1.0.0 (2026-05-23)
- Initial release: Opus-plan + Sonnet-execute + Opus-audit cycle.
