# Claude Dev Cycle

> Workflow autónomo para Claude Code · **Opus planifica + audita, Sonnet ejecuta**.
> Ahorra ~70% en tokens manteniendo calidad. Aprende de sus errores.

> Autonomous workflow for Claude Code · **Opus plans + audits, Sonnet executes**.
> Saves ~70% on tokens while keeping quality. Learns from its own mistakes.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-2.1.139%2B-orange)](https://code.claude.com)
[![Version](https://img.shields.io/badge/version-1.1.0-green)](#changelog)

---

## 🇪🇸 Español

### Qué es

Una skill (`/dev-cycle`) para Claude Code que implementa un workflow autónomo refinado en proyectos reales de producción:

1. **Opus** lee contexto del proyecto, planifica el trabajo y audita resultados.
2. **Sonnet 4.6** sub-agents ejecutan los specs en paralelo (a veces 3-5 al mismo tiempo).
3. **Opus** revisa cada entregable, itera si hace falta, y commitea.
4. Opcionalmente combinable con `/goal` para loop autónomo hasta que se cumpla una condición.

### Por qué usarla

Métricas reales de proyectos en producción:

| Tipo de tarea | Con esta skill | Equivalente todo-Opus | Ahorro |
|---|---|---|---|
| Feature mediana (un módulo) | **$4-10 USD** | $30-60 USD | ~70-80% |
| Feature grande (varios bloques, agents en paralelo) | **$12-25 USD** | $80-150 USD | ~75-85% |
| Mantenimiento mensual estimado | **$100-300/mes** | $600-1500/mes | ~75% |

La calidad se mantiene equivalente porque:

- Opus sigue tomando todas las decisiones arquitectónicas.
- Opus audita cada entregable antes de commitear.
- Loop de iteración corrige cualquier gap de Sonnet automáticamente.
- El patrón usa **specs explícitos** que no dejan a Sonnet improvisar.

### Novedades v1.1.0

- **🧠 Memoria de errores**: cada error que Opus le encuentra a Sonnet se documenta automáticamente en `<repo>/.claude/dev-cycle-lessons.md`. El próximo ciclo lee ese archivo y le advierte a Sonnet antes de empezar — evita re-trabajos.
- **💰 Reporte de tokens al final**: cada ciclo termina con un resumen mostrando cuánto se gastó vs cuánto hubiera sido todo Opus.

### Prerequisites

| Requisito | Cómo conseguirlo | Obligatorio |
|---|---|---|
| Claude Code 2.1.139+ | `claude update` (instalador built-in) | ✅ |
| Acceso API a Opus + Sonnet | Suscripción Pro/Team/Max o API key Anthropic | ✅ |
| `CLAUDE.md` del proyecto | Cada proyecto debe tener uno con sus convenciones | ✅ |
| Comando `/goal` | Viene built-in con Claude Code 2.1.139+ | ✅ |
| `claude-mem` | `npm install -g claude-mem` | Recomendado (funciona sin él) |

### Instalación

En cualquier sesión de Claude Code:

```bash
/plugin marketplace add github:Maneco26/claude-dev-cycle
/plugin install dev-cycle@claude-dev-cycle
```

Listo. La skill queda instalada globalmente y aparece en TODOS tus proyectos.

### Uso

En cualquier proyecto:

```
/dev-cycle <descripción de la tarea>
```

Ejemplos:

```
/dev-cycle agregar página de perfil con upload de avatar a Cloudinary
/dev-cycle migrar autenticación de JWT a session cookies httpOnly
/dev-cycle implementar billing con Stripe Checkout + webhook handler
/dev-cycle resolver issues con prioridad high del backlog de Linear
```

Claude va a:

1. **Leer lecciones aprendidas** del archivo `dev-cycle-lessons.md` para no repetir errores.
2. Buscar contexto histórico (claude-mem si está instalado).
3. Mapear archivos relevantes en tu codebase.
4. Leer tu `CLAUDE.md` para convenciones.
5. Crear un plan estructurado con `TaskList`.
6. Lanzar uno o más **sub-agents Sonnet en paralelo** con specs detallados.
7. Auditar entregables, iterar hasta limpio.
8. **Documentar errores nuevos** en lessons file.
9. Commit + push (si es repo git).
10. Actualizar memoria (si claude-mem está instalado).
11. **Reportar costos**: tokens reales + estimación de ahorro vs todo-Opus.

### Cómo funciona la memoria de errores

Después de la primera vez que corrés `/dev-cycle` en un proyecto, se crea (si no existe) el archivo `<repo>/.claude/dev-cycle-lessons.md`:

```markdown
# Dev Cycle — Lessons Learned

## Lessons

### 2026-05-23 [TypeScript]
**Error:** Sonnet usó `z.infer<>` en form schemas con `exactOptionalPropertyTypes: true`.
**Fix:** Usar `z.input<>` cuando hay `.default()` o `.optional()`.
**Context:** Form de payroll-employees.tsx.

### 2026-05-23 [Database]
**Error:** Referenció columna `organization_users.is_active` que no existe.
**Fix:** Usar solo `organization_id` y `user_id` para chequear membership.
**Context:** Migración de fiscal-year-close.
```

Podés editarlo a mano y agregar lecciones manualmente. Está bajo control de versiones (es parte del repo), así que el equipo lo ve y aprende junto.

### Combinar con `/goal` para trabajo desatendido

```
/dev-cycle implementar módulo de autenticación completo
/goal módulo de auth compila, tiene 5+ tests pasando, lint limpio
```

Un evaluador independiente (modelo Haiku por default) chequea la condición cada turn. Claude sigue trabajando hasta que se cumple.

### Cómo se mantiene la calidad

La skill obliga estos patterns en el spec que le manda a Sonnet:

- **Schemas explícitos** — SQL DDL completo, nombres exactos de campos, definiciones de tipos.
- **Referencias de patrón** — apunta a archivos existentes específicos que Sonnet debe imitar.
- **Verificaciones obligatorias** — typecheck + lint + build + tests deben pasar antes de reportar.
- **Cero commits inesperados** — Sonnet nunca commitea; solo el coordinador Opus lo hace, tras auditoría.
- **Respeto al `CLAUDE.md`** — la skill lee las convenciones del proyecto e instruye a Sonnet a seguirlas estrictamente.
- **Lecciones aprendidas** — todos los errores previos se inyectan al spec.

Si Sonnet falla algo, Opus lo detecta en la auditoría y usa `SendMessage` para enviar fixes específicos al mismo agent (preserva contexto, más rápido que empezar de cero).

### Qué NO es esta skill

- ❌ NO es un reemplazo del code review humano para cambios críticos.
- ❌ NO es magia — specs malos siguen produciendo código malo. La skill amplifica la calidad existente del proyecto.
- ❌ NO es un SaaS. Corre localmente en Claude Code con tus credenciales.

### Autor

Creada por [Maneco26](https://github.com/Maneco26).

Refinada en proyectos reales (Norte ERP, SaaS multi-tenant para PyMEs centroamericanas).

Si te resulta útil, ⭐ el repo. PRs bienvenidos.

---

## 🇬🇧 English

### What this is

A Claude Code skill (`/dev-cycle`) implementing a token-saving autonomous workflow refined in real production projects:

1. **Opus** reads project context, plans the work in detail, audits results.
2. **Sonnet 4.6** sub-agents execute specs in parallel (sometimes 3-5 at once).
3. **Opus** reviews each deliverable, iterates if needed, commits.
4. Optionally combines with `/goal` to autonomously loop until a success condition is met.

### Why use it

Real metrics from production projects:

| Task type | With this skill | All-Opus equivalent | Savings |
|---|---|---|---|
| Medium feature | **$4-10 USD** | $30-60 USD | ~70-80% |
| Large feature (parallel agents) | **$12-25 USD** | $80-150 USD | ~75-85% |
| Monthly maintenance estimate | **$100-300/mo** | $600-1500/mo | ~75% |

Quality stays equivalent because Opus owns all architectural decisions, audits every deliverable, and the iteration loop fixes Sonnet gaps automatically.

### What's new in v1.1.0

- **🧠 Error memory**: every mistake Opus catches in Sonnet's deliverable gets documented in `<repo>/.claude/dev-cycle-lessons.md`. Next cycle reads that file and warns Sonnet before starting — prevents re-work.
- **💰 Token report at the end**: every cycle finishes with a summary showing real spend vs estimated all-Opus cost.

### Prerequisites

| Requirement | How to get it | Required |
|---|---|---|
| Claude Code 2.1.139+ | `claude update` | ✅ |
| API access to Opus + Sonnet | Pro/Team/Max subscription or API key | ✅ |
| Project `CLAUDE.md` | Each project should have one | ✅ |
| `/goal` command | Built-in to Claude Code 2.1.139+ | ✅ |
| `claude-mem` | `npm install -g claude-mem` | Recommended |

### Installation

```bash
/plugin marketplace add github:Maneco26/claude-dev-cycle
/plugin install dev-cycle@claude-dev-cycle
```

That's it. Globally available across all your projects.

### Usage

```
/dev-cycle <task description>
```

Examples:

```
/dev-cycle add user profile page with avatar upload to Cloudinary
/dev-cycle migrate auth from JWT to httpOnly session cookies
/dev-cycle implement billing module with Stripe Checkout + webhook
/dev-cycle resolve all high-priority issues from my Linear backlog
```

Claude will:

1. **Read learned lessons** from `dev-cycle-lessons.md` to avoid repeating mistakes.
2. Search historical context (claude-mem if installed).
3. Map relevant files in your codebase.
4. Read your `CLAUDE.md` for conventions.
5. Create structured plan with `TaskList`.
6. Launch one or more **Sonnet sub-agents in parallel** with detailed specs.
7. Audit deliverables, iterate until clean.
8. **Document new mistakes** in lessons file.
9. Commit + push (if git repo).
10. Update memory (if claude-mem installed).
11. **Report costs**: real tokens + estimated all-Opus savings.

### How the error memory works

After your first `/dev-cycle` invocation in a project, the file `<repo>/.claude/dev-cycle-lessons.md` is created (if it doesn't exist) with this format:

```markdown
# Dev Cycle — Lessons Learned

## Lessons

### 2026-05-23 [TypeScript]
**Error:** Sonnet used `z.infer<>` in form schemas with `exactOptionalPropertyTypes: true`.
**Fix:** Use `z.input<>` when there are `.default()` or `.optional()` fields.
**Context:** payroll-employees.tsx form.
```

You can manually add lessons. The file is in git, so the team learns together.

### Combining with `/goal` for unattended work

```
/dev-cycle implement complete auth module
/goal auth module compiles, has 5+ tests passing, lint clean
```

A separate evaluator model (Haiku by default) checks your condition after every turn. Claude keeps working until met.

### How quality stays high

The skill enforces these patterns in the spec it sends to Sonnet:

- **Explicit schemas** — full SQL DDL, exact field names, type definitions.
- **Pattern references** — points to specific existing files Sonnet should mimic.
- **Mandatory verifications** — typecheck + lint + build + tests must pass.
- **No surprise commits** — Sonnet never commits; only the Opus coordinator does, after audit.
- **Respect for `CLAUDE.md`** — instructs Sonnet to follow project conventions strictly.
- **Learned lessons** — all past mistakes injected into the spec.

If Sonnet misses something, Opus catches it during audit and uses `SendMessage` to send targeted fixes back to the same agent.

### What this skill is NOT

- ❌ Not a replacement for human code review on critical changes.
- ❌ Not magic — bad specs still produce bad code. The skill amplifies your existing quality bar.
- ❌ Not a SaaS. Runs locally in Claude Code with your own credentials.

### Author

Created by [Maneco26](https://github.com/Maneco26).

Refined in real production projects (Norte ERP, multi-tenant SaaS for Central American SMBs).

If you find this useful, ⭐ the repo. PRs welcome.

---

## License

MIT. Use, fork, modify, redistribute freely.

## Changelog

### 1.1.0 (2026-05-23)
- **NEW**: persistent lessons-learned at `<repo>/.claude/dev-cycle-lessons.md`. Step 0 reads it before planning; Step 6 documents every new Sonnet mistake.
- **NEW**: Step 9 final report with token + USD cost breakdown vs all-Opus estimate.
- Documentation bilingual (ES + EN in same file).

### 1.0.0 (2026-05-23)
- Initial public release.

---

**Related**: [Claude Code `/goal` docs](https://code.claude.com/docs/en/goal) · [Plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces) · [claude-mem](https://github.com/thedotmack/claude-mem)
