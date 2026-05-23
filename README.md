# Claude Dev Cycle v2.0 — 3-en-1

> **Cycle + Telegraphic Mode (caveman) + Self-evaluation loop**
> Workflow autónomo para Claude Code. ~80-85% de ahorro vs Opus solo.

> Autonomous workflow for Claude Code combining three proven token-saving
> techniques into a single skill.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-2.1.139%2B-orange)](https://code.claude.com)
[![Version](https://img.shields.io/badge/version-2.0.0-green)](#changelog)

---

## 🇪🇸 Español

### Qué hace (3 capas integradas)

| Capa | Técnica | Ahorro |
|---|---|---|
| **1 — Cycle** | Opus planifica + Sonnet ejecuta + Opus audita | ~70% (lever principal) |
| **2 — Telegraphic Mode** | Sonnet responde en estilo telegráfico (sin filler words) | +10-15% adicional |
| **3 — Self-evaluation loop** | Opus auto-evalúa condición de éxito y sigue solo | +5-10% adicional |
| **TOTAL combinado** | | **~80-85%** vs Opus puro |

### Capa 1 — Cycle

Opus lee contexto, planifica el trabajo, audita resultados. Sonnet 4.6 (5× más barato) ejecuta los specs en paralelo. Loop hasta calidad aprobada.

### Capa 2 — Telegraphic Mode (técnica caveman)

Cuando Opus delega a Sonnet, le pasa una instrucción explícita para que responda en estilo telegráfico:
- Sin filler words ("Voy a...", "Te explico...").
- Sin pleasantries ("Espero que sirva...").
- Sin frases de conexión ("Ahora voy a...").
- Reportes en bullets secos: `Fix X (file:line). typecheck pass. Done.`
- **Código + paths + URLs intactos** (no se sacrifica legibilidad).

Crédito: técnica original de Julius Brussee ([github.com/juliusbrussee/caveman](https://github.com/juliusbrussee/caveman), MIT). Re-implementada e integrada acá con atribución (ver `NOTICE.md`).

### Capa 3 — Self-evaluation loop

Después de cada audit, Opus se pregunta: *"¿Hay una condición de éxito? ¿Se cumple?"*. Si no, sigue trabajando solo sin pedir confirmación al usuario. Replica el comportamiento del comando nativo `/goal` sin depender del CLI.

Si tenés `/goal` nativo (Claude Code 2.1.139+), funciona en paralelo y complementario.

### Métricas reales

| Tipo de tarea | Costo con esta skill | Equivalente Opus puro | Ahorro |
|---|---|---|---|
| Feature mediana | **$3-8 USD** | $30-60 USD | ~80% |
| Feature grande | **$10-20 USD** | $80-150 USD | ~82% |
| Mantenimiento mensual | **$80-250/mes** | $600-1500/mes | ~80% |

### Prerequisites

| Requisito | Cómo obtenerlo | Obligatorio |
|---|---|---|
| Claude Code 2.1.139+ | `claude update` | ✅ |
| Acceso API a Opus + Sonnet | Claude Pro/Team/Max o API key Anthropic | ✅ |
| `CLAUDE.md` del proyecto | Convenciones del repo | ✅ |
| `claude-mem` | `npm install -g claude-mem` | Recomendado |

### Instalación

```bash
/plugin marketplace add github:Maneco26/claude-dev-cycle
/plugin install dev-cycle@claude-dev-cycle
```

Skill global, disponible en todos los proyectos.

### Uso

```
/dev-cycle <descripción de la tarea>
```

Ejemplos:

```
/dev-cycle agregar página de perfil con upload de avatar
/dev-cycle migrar auth de JWT a session cookies httpOnly
/dev-cycle resolver issues priority:high de mi backlog
```

Opcional — activar Telegraphic Mode también en respuestas de Opus (no solo Sonnet):

```
/dev-cycle --telegraphic <tarea>
```

Combo con `/goal` nativo para máxima autonomía:

```
/dev-cycle implementar módulo X
/goal X compila, tiene 5+ tests pasando, lint limpio
```

### Cómo funciona la memoria de errores

Después del primer `/dev-cycle`, se crea `<repo>/.claude/dev-cycle-lessons.md`:

```markdown
### 2026-05-23 [TypeScript]
**Error:** Sonnet usó `z.infer<>` con `exactOptionalPropertyTypes`.
**Fix:** Usar `z.input<>` cuando hay `.default()` o `.optional()`.
**Context:** form-component.tsx.
```

- Auto-mantenido por la skill.
- Editable a mano.
- Versionado en git → equipo aprende junto.
- Cada nuevo ciclo lee este archivo y se lo inyecta a Sonnet **antes** de empezar.

### Qué NO es esta skill

- ❌ NO reemplaza code review humano en cambios críticos.
- ❌ NO es magia — specs malos producen código malo. Amplifica el quality bar existente del proyecto.
- ❌ NO es SaaS. Corre localmente con tus credenciales.

### Atribuciones

Este proyecto integra técnicas/inspiración de:

- **Anthropic** — Claude Code, Opus, Sonnet, plugin marketplace.
- **Julius Brussee** — técnica caveman ([repo](https://github.com/juliusbrussee/caveman)) re-implementada como Telegraphic Mode en Capa 2.

Ver `NOTICE.md` para detalles completos de atribución.

### Autor

[Maneco26](https://github.com/Maneco26). Refinado en proyectos reales de producción.

---

## 🇬🇧 English

### What it does (3 integrated layers)

| Layer | Technique | Savings |
|---|---|---|
| **1 — Cycle** | Opus plans + Sonnet executes + Opus audits | ~70% (main lever) |
| **2 — Telegraphic Mode** | Sonnet replies in telegraphic style | +10-15% extra |
| **3 — Self-evaluation loop** | Opus self-evaluates and continues alone | +5-10% extra |
| **COMBINED TOTAL** | | **~80-85%** vs raw Opus |

### Layer 1 — Cycle

Opus reads context, plans, audits. Sonnet 4.6 (5× cheaper) executes specs in parallel. Loop until quality approved.

### Layer 2 — Telegraphic Mode (caveman technique)

When Opus delegates to Sonnet, it passes an explicit instruction to reply in telegraphic style:
- No filler words ("I'll help you...", "Let me...").
- No pleasantries ("Hope this helps...").
- No transition phrases ("Now I will...").
- Dry bullet reports: `Fix X (file:line). typecheck pass. Done.`
- **Code + paths + URLs intact** (no readability sacrificed).

Credit: original technique by Julius Brussee ([github.com/juliusbrussee/caveman](https://github.com/juliusbrussee/caveman), MIT). Re-implemented and integrated here with attribution (see `NOTICE.md`).

### Layer 3 — Self-evaluation loop

After each audit, Opus asks itself: *"Is there a success condition? Is it met?"*. If not, keeps working without asking the user. Replicates native `/goal` behavior without CLI dependency.

If you have `/goal` (Claude Code 2.1.139+), it works in parallel and complementary.

### Real metrics

| Task type | With this skill | All-Opus equivalent | Savings |
|---|---|---|---|
| Medium feature | **$3-8 USD** | $30-60 USD | ~80% |
| Large feature | **$10-20 USD** | $80-150 USD | ~82% |
| Monthly maintenance | **$80-250/mo** | $600-1500/mo | ~80% |

### Prerequisites

| Requirement | How to get it | Required |
|---|---|---|
| Claude Code 2.1.139+ | `claude update` | ✅ |
| API access to Opus + Sonnet | Pro/Team/Max sub or API key | ✅ |
| Project `CLAUDE.md` | Repo conventions | ✅ |
| `claude-mem` | `npm install -g claude-mem` | Recommended |

### Installation

```bash
/plugin marketplace add github:Maneco26/claude-dev-cycle
/plugin install dev-cycle@claude-dev-cycle
```

Globally available across all projects.

### Usage

```
/dev-cycle <task description>
```

Optional — enable Telegraphic Mode on Opus output too:

```
/dev-cycle --telegraphic <task>
```

Combo with native `/goal` for maximum autonomy:

```
/dev-cycle implement module X
/goal X compiles, 5+ tests pass, lint clean
```

### How error memory works

After your first `/dev-cycle`, the file `<repo>/.claude/dev-cycle-lessons.md` is created with this format:

```markdown
### 2026-05-23 [TypeScript]
**Error:** Sonnet used `z.infer<>` with `exactOptionalPropertyTypes`.
**Fix:** Use `z.input<>` when there are `.default()` or `.optional()`.
**Context:** form-component.tsx.
```

Auto-maintained, manually editable, git-versioned (team learns together). Every new cycle reads this file and injects it to Sonnet **before** starting.

### What this skill is NOT

- ❌ Not a replacement for human code review on critical changes.
- ❌ Not magic — bad specs produce bad code. Amplifies your existing quality bar.
- ❌ Not a SaaS. Runs locally with your credentials.

### Attributions

This project integrates techniques/inspiration from:

- **Anthropic** — Claude Code, Opus, Sonnet, plugin marketplace.
- **Julius Brussee** — caveman technique ([repo](https://github.com/juliusbrussee/caveman)) re-implemented as Telegraphic Mode in Layer 2.

See `NOTICE.md` for full attribution details.

### Author

[Maneco26](https://github.com/Maneco26). Refined in real production projects.

---

## License

MIT. Use, fork, modify, redistribute freely. See `LICENSE` and `NOTICE.md`.

## Changelog

### 2.0.0 (2026-05-23)
- **NEW**: Telegraphic Mode integrated (caveman technique by Julius Brussee, MIT) as Layer 2. Explicit instruction block in SKILL.md gets pasted into Sonnet sub-agent prompts.
- **NEW**: Self-evaluation loop (Layer 3) replicates native `/goal` without CLI dependency.
- **NEW**: 3-layer cost breakdown in final report.
- **NEW**: `NOTICE.md` with full third-party attribution.

### 1.1.0 (2026-05-23)
- Persistent lessons-learned at `<repo>/.claude/dev-cycle-lessons.md`.
- Final report with token + USD cost breakdown.
- Bilingual ES + EN.

### 1.0.0 (2026-05-23)
- Initial release: Opus-plan + Sonnet-execute + Opus-audit cycle.

---

**Related**: [Claude Code `/goal` docs](https://code.claude.com/docs/en/goal) · [Plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces) · [Original caveman by Julius Brussee](https://github.com/juliusbrussee/caveman) · [claude-mem](https://github.com/thedotmack/claude-mem)
