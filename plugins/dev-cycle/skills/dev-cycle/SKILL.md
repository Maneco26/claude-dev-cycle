---
name: dev-cycle
description: Workflow autónomo Opus-Sonnet-loop para Claude Code, integrado con técnica Telegraphic Mode (caveman) y Self-evaluation loop. Opus planifica + audita, Sonnet ejecuta en estilo telegráfico. Aprendizaje persistente entre sesiones. Loop autónomo hasta cumplir condición. Reporte de tokens al final. Ahorra ~80-85% vs Opus solo manteniendo calidad.
trigger: /dev-cycle
---

# /dev-cycle v2.0 — Triple stack: Cycle + Telegraphic + Self-evaluation

> Bilingual skill. **ES** primero, **EN** después.
> 3 técnicas combinadas en una sola skill: ~80-85% de ahorro vs Opus solo.

---

# 🇪🇸 Español

## Qué hace esta skill (3-en-1)

### Capa 1 — Cycle (Opus plan + Sonnet ejecuta + Opus audita)

Opus lee contexto, planifica, delega a Sonnet 4.6 (5× más barato). Sonnet ejecuta el spec en paralelo. Opus audita y loop hasta calidad aprobada.
**Ahorro base: ~70% en tokens.**

### Capa 2 — Telegraphic Mode (técnica caveman)

Sub-agents Sonnet responden en estilo telegráfico: sin filler words, sin pleasantries, sin frases de conexión. Código + paths + URLs intactos.
**Ahorro adicional: ~10-15% en output de Sonnet.**

Crédito: técnica original de Julius Brussee ([github.com/juliusbrussee/caveman](https://github.com/juliusbrussee/caveman), MIT). Re-implementada e integrada acá con atribución (ver NOTICE.md).

### Capa 3 — Self-evaluation loop (replica /goal sin depender del CLI)

Después de cada audit, Opus auto-evalúa "¿se cumple la condición de éxito?". Si no, sigue solo hasta cumplirla. Si el usuario tiene `/goal` nativo de Claude Code, es complementario.

**Ahorro total combinado: ~80-85% vs Opus puro.**

## Cuándo usar

Trigger explícito: `/dev-cycle <descripción de tarea>`. Buenos casos:
- Implementar feature nueva.
- Arreglar bugs del backlog.
- Refactor de área.
- Completar TODOs.

Cuándo NO: cambios triviales, preguntas, tareas en curso que solo necesitan continuación.

## Prerequisites

- Claude Code 2.1.139+ (`/goal` y plugin marketplace).
- Acceso API a Opus + Sonnet.
- `CLAUDE.md` en el proyecto con sus convenciones.
- Recomendado: `npm install -g claude-mem` para memoria cross-session.

## El ciclo paso a paso

### Paso 0 — Refresh de lecciones aprendidas

Leer `<repo>/.claude/dev-cycle-lessons.md` si existe. Si `claude-mem` está instalado, también buscar observaciones `obs_type: change` con query "lesson sonnet mistake".

Las lecciones relevantes se inyectan al spec de Sonnet bajo "## Errores conocidos a evitar".

### Paso 1 — Contexto histórico

`claude-mem:mem-search` con keywords de la tarea. Detectar sesiones paralelas, decisiones grabadas, módulos reutilizables.

### Paso 2 — Mapeo de codebase

`Grep` keywords, identificar módulo patrón, listar archivos exactos. Leer `CLAUDE.md`.

### Paso 3 — Plan estructurado con TaskList

Una task por unidad lógica. Detallado: schema, archivos exactos, verificaciones. Si grande (>4h), dividir en bloques paralelos.

### Paso 4 — Delegar a Sonnet **en Telegraphic Mode**

`Agent` con `model: "sonnet"` SIEMPRE. Prompt explícito con:
- Contexto técnico del repo.
- Patrones a copiar.
- Schema completo si aplica.
- Lista exacta de archivos.
- Lecciones aprendidas del paso 0.
- Verificaciones obligatorias.
- **NUEVO v2.0**: instrucción de Telegraphic Mode (ver sección dedicada abajo).

### Paso 5 — Auditar

Verificaciones automáticas + sample inspection + verificación independiente + anti-patterns.

### Paso 6 — Iterar + documentar lecciones

Continuar con `SendMessage` al mismo agent. Documentar CADA error en `dev-cycle-lessons.md`:

```markdown
### YYYY-MM-DD [Categoría]
**Error:** lo que Sonnet hizo mal.
**Fix:** cómo se hace.
**Context:** archivo/módulo.
```

Categorías: `[TypeScript]`, `[Database]`, `[RLS]`, `[Auth]`, `[Migración]`, `[i18n]`, `[Build]`, `[UI]`, `[Security]`.

### Paso 7 — Self-evaluation loop (NUEVO v2.0)

Después del audit limpio, Opus debe preguntarse:

> *"¿Existe una condición de éxito implícita o explícita en esta tarea? ¿Está cumplida?"*

Condiciones típicas según tipo de tarea:
- Bug fix: typecheck pasa + lint OK + el comportamiento descrito en el issue ya no se reproduce.
- Feature: typecheck + lint + build + (idealmente) tests pasan + UI rendereable.
- Refactor: typecheck + lint + build + cero regresiones en comportamiento.

Si la condición NO está cumplida → volver al paso 5 (audit) o paso 4 (delegar fix) sin pedirle al usuario que diga "continuá".

**Cuándo parar el loop**:
- Condición cumplida → continuar al paso 8.
- 5 iteraciones sin progreso → escalar al usuario con resumen claro de bloqueo.
- Error que requiere decisión humana (ambigüedad funcional, conflicto de specs) → escalar.

Si el usuario invocó `/goal` nativo además, el evaluador externo (Haiku) chequea independientemente. La skill respeta y NO duplica el loop.

### Paso 8 — Commit + push

`git add` archivos específicos (no `-A`). Conventional Commits. Push si es repo con remote.

### Paso 9 — Actualizar memoria

Lessons file con todos los errores nuevos. Memory doc si aplica.

### Paso 10 — Reporte final de tokens

Tabla obligatoria con **3 layers de ahorro**:

```
### 💰 Resumen de costos del ciclo

| Capa | Tokens | Costo USD |
|---|---|---|
| Opus (plan + audit + self-eval) | ~XX,XXX | ~$X.XX |
| Sonnet (Y sub-agents, telegraphic mode) | ~XXX,XXX | ~$X.XX |
| **Total real** | **~XXX,XXX** | **~$X.XX** |
| Si todo Opus normal (≈2.5×) | ~XXX,XXX | ~$XX.XX |
| **Ahorrado** | **~XXX,XXX** | **~$XX.XX (XX%)** |

Desglose del ahorro:
- Delegación Sonnet vs Opus: ~XX% (capa 1)
- Telegraphic mode en Sonnet: ~XX% (capa 2)
- Self-evaluation interno vs invocaciones manuales: ~XX% (capa 3)

Tiempo: XX min · Sub-agents: X · Iteraciones audit: X · Self-eval loops: X · Lecciones nuevas: X
```

---

## 📢 Telegraphic Mode — Instrucción explícita para sub-agents

> Esta sección es **CRÍTICA**. Cuando Opus delegue al sub-agent Sonnet en el Paso 4, **DEBE incluir literal esta instrucción dentro del prompt del agent**:

```
## Telegraphic Mode (técnica caveman)

Respondé y reportá en estilo telegráfico para minimizar tokens de output.
Reglas:

REGLA 1 — Sin filler words ni pleasantries
NO escribir: "I'll help you", "Let me", "Great, I'll", "Of course",
"Voy a...", "Te explico...", "Como verás...", "Espero que esto sirva".
SÍ escribir directo: "Fix X. Done.", "Read Y. Modified Z."

REGLA 2 — Sin frases de conexión
NO: "Now I will...", "Next, I need to...", "Then I'll proceed to...",
"Ahora voy a...", "El siguiente paso es..."
SÍ: usar listas/bullets/comandos secos.

REGLA 3 — Reportes en bullet seco
NO: "I successfully implemented the feature and verified that..."
SÍ:
- Implemented feature X (file:line).
- Verified typecheck pass.
- Verified lint pass.
- Done.

REGLA 4 — Código intacto
El código generado/modificado NUNCA se comprime. Mantener idiomático,
con comentarios necesarios, identificadores legibles.
NO sacrificar legibilidad de código por tokens.

REGLA 5 — Paths, URLs, IDs intactos
NUNCA abreviar paths, URLs, UUIDs, commit hashes. Copiar literal.

REGLA 6 — Errores reportados completos
Si hay error o blocker, reportar mensaje completo del error, no resumir.

REGLA 7 — Tablas y estructura mantenidas
Tablas markdown, listas y headers se mantienen para legibilidad
estructural. La compresión es de PROSA, no de estructura.

EJEMPLO de output telegráfico correcto:

  ## Resultado
  - Modified: app/auth/page.tsx, lib/auth.ts.
  - Migration applied: 20260523120000__add_oauth.sql.
  - typecheck: pass.
  - lint: pass (0 errors, 1 pre-existing warning unrelated).
  - REST test: register_oauth_user(...) → OK, returned UUID.
  - Blockers: none.

NO escribir prosa decorativa. Si necesitás explicar una decisión técnica
no obvia, hacelo en 1 oración corta + bullet con contexto.

Crédito de la técnica: Julius Brussee (caveman, MIT).
```

Esta instrucción **debe estar pegada literalmente** en el prompt que Opus le pasa al Sonnet en el Paso 4. No es opcional. Sin esta sección, perdemos la capa 2 de ahorro.

**¿Aplicar Telegraphic Mode también al output de Opus?**

Por default: NO. Opus se comunica con el usuario humano y la prosa ayuda a entender. Si el usuario quiere activar Telegraphic Mode en Opus también, debe pedirlo explícitamente con `/dev-cycle --telegraphic <tarea>` o decírselo: "respondeme en modo telegráfico". Entonces Opus aplica las mismas reglas a su comunicación con el usuario.

---

## Anti-patterns

NO:
- Lanzar Sonnet sin Telegraphic Mode instruction → pierde capa 2.
- Saltar self-eval → tarea incompleta declarada como completa.
- `model: "opus"` en sub-agents → anula todo el ahorro.

SÍ:
- Pegar literal el bloque Telegraphic en prompts de sub-agents.
- Self-evaluate honestamente. Si no se cumple, seguir.
- Documentar cada error en lessons.

---

# 🇬🇧 English

## What this skill does (3-in-1)

### Layer 1 — Cycle (Opus plans + Sonnet executes + Opus audits)

Opus reads context, plans, delegates to Sonnet 4.6 (5× cheaper). Sonnet executes specs in parallel. Opus audits, loops until quality is approved.
**Base savings: ~70% in tokens.**

### Layer 2 — Telegraphic Mode (caveman technique)

Sonnet sub-agents respond in telegraphic style: no filler words, no pleasantries, no transition phrases. Code + paths + URLs intact.
**Additional savings: ~10-15% in Sonnet output.**

Credit: original technique by Julius Brussee ([github.com/juliusbrussee/caveman](https://github.com/juliusbrussee/caveman), MIT). Re-implemented and integrated here with attribution (see NOTICE.md).

### Layer 3 — Self-evaluation loop (replicates /goal without CLI dependency)

After each audit, Opus self-asks "is the success condition met?". If not, continues alone until met. If the user has Claude Code's built-in `/goal`, it's complementary.

**Combined total savings: ~80-85% vs raw Opus.**

## When to use

Explicit: `/dev-cycle <task description>`. Good for new features, backlog bugs, refactors, TODOs.

Don't use for: trivial changes, questions, in-progress tasks needing only continuation.

## Prerequisites

- Claude Code 2.1.139+.
- API access to Opus + Sonnet.
- `CLAUDE.md` in the project.
- Recommended: `npm install -g claude-mem`.

## The cycle step by step

### Step 0 — Refresh learned lessons

Read `<repo>/.claude/dev-cycle-lessons.md`. Search claude-mem for past lessons. Inject relevant into Sonnet spec.

### Step 1 — Historical context

`claude-mem:mem-search` for parallel sessions, decisions, reusable modules.

### Step 2 — Codebase mapping

`Grep`, identify pattern module, list exact files. Read `CLAUDE.md`.

### Step 3 — Structured plan with TaskList

One task per logical unit, detailed, parallelizable.

### Step 4 — Delegate to Sonnet **in Telegraphic Mode**

`Agent` with `model: "sonnet"` ALWAYS. Prompt MUST include the Telegraphic Mode instruction block (see dedicated section below).

### Step 5 — Audit

Automated + sample inspection + independent + anti-patterns.

### Step 6 — Iterate + document lessons

`SendMessage` to same agent. Document EVERY mistake in `dev-cycle-lessons.md`.

### Step 7 — Self-evaluation loop (NEW v2.0)

Opus asks itself: *"Is there a success condition? Is it met?"*

Typical conditions:
- Bug fix: typecheck + lint + reported behavior no longer reproduces.
- Feature: typecheck + lint + build + tests + UI renders.
- Refactor: typecheck + lint + build + zero behavior regressions.

If NOT met → back to step 5 or step 4 without asking user to say "continue".

Stop the loop on: condition met, 5 iterations no progress, ambiguity needing human input.

If user invoked native `/goal`, external evaluator (Haiku) runs independently. The skill respects it and doesn't duplicate the loop.

### Step 8 — Commit + push

Specific files, Conventional Commits.

### Step 9 — Update memory

Lessons file + memory docs.

### Step 10 — Final token report

Mandatory table with 3 layers of savings (see Spanish section above).

---

## 📢 Telegraphic Mode — Explicit instruction for sub-agents

> **CRITICAL section.** When Opus delegates to Sonnet in Step 4, this block **MUST be pasted literally** inside the agent's prompt:

```
## Telegraphic Mode (caveman technique)

Reply and report in telegraphic style to minimize output tokens.

RULE 1 — No filler words or pleasantries
DON'T write: "I'll help you", "Let me", "Great, I'll", "Of course"
DO write directly: "Fix X. Done.", "Read Y. Modified Z."

RULE 2 — No transition phrases
DON'T: "Now I will...", "Next, I need to...", "Then I'll proceed to..."
DO: use lists/bullets/commands.

RULE 3 — Dry bullet reports
DON'T: "I successfully implemented the feature and verified that..."
DO:
- Implemented feature X (file:line).
- Verified typecheck pass.
- Verified lint pass.
- Done.

RULE 4 — Code stays intact
Generated/modified code is NEVER compressed. Idiomatic, with needed
comments, readable identifiers. DON'T sacrifice code readability for tokens.

RULE 5 — Paths, URLs, IDs intact
NEVER abbreviate paths, URLs, UUIDs, commit hashes. Copy literal.

RULE 6 — Errors reported in full
If error/blocker, report full error message, don't summarize.

RULE 7 — Tables and structure preserved
Markdown tables, lists, headers stay for structural readability. Compression is for PROSE, not structure.

EXAMPLE of correct telegraphic output:

  ## Result
  - Modified: app/auth/page.tsx, lib/auth.ts.
  - Migration applied: 20260523120000__add_oauth.sql.
  - typecheck: pass.
  - lint: pass (0 errors, 1 pre-existing warning unrelated).
  - REST test: register_oauth_user(...) → OK, returned UUID.
  - Blockers: none.

Don't write decorative prose. If a non-obvious technical decision needs
explanation, do it in 1 short sentence + bullet with context.

Technique credit: Julius Brussee (caveman, MIT).
```

This instruction **must be pasted literally** in the prompt Opus passes to Sonnet in Step 4. Not optional. Without this section, we lose layer 2 of savings.

**Apply Telegraphic Mode to Opus output too?**

Default: NO. Opus communicates with the human user and prose helps understanding. If the user wants Telegraphic Mode on Opus too, they must explicitly ask: `/dev-cycle --telegraphic <task>` or "respond in telegraphic mode". Then Opus applies the same rules.

---

## Changelog

### v2.0.0 (2026-05-23)
- **NEW**: Telegraphic Mode integrated explicitly (caveman technique, credit to Julius Brussee MIT).
- **NEW**: Self-evaluation loop replicates `/goal` lockstep without CLI dependency.
- **NEW**: 3-layer cost report breakdown.
- Compatibility: native `/goal` and external caveman plugin still work alongside.

### v1.1.0 (2026-05-23)
- Persistent lessons-learned at `<repo>/.claude/dev-cycle-lessons.md`.
- Final report with token + USD cost breakdown vs all-Opus estimate.
- Bilingual ES + EN.

### v1.0.0 (2026-05-23)
- Initial release.
