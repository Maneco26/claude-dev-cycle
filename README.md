# Claude Dev Cycle

> Token-saving autonomous development workflow for Claude Code.
> **Opus plans + audits, Sonnet executes.** ~70% cheaper than Opus-only with equivalent quality.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-2.1.139%2B-orange)](https://code.claude.com)

## What this is

A Claude Code skill (`/dev-cycle`) that implements a **cost-optimized development workflow** refined in real production projects:

1. **Opus** (the coordinator model) reads project context, plans the work in detail, and audits results.
2. **Sonnet 4.6** sub-agents execute the specs in parallel (sometimes 3-5 at once).
3. **Opus** reviews each deliverable, iterates if needed, then commits.
4. Optionally combines with `/goal` to loop autonomously until a success condition is met.

The pattern works because **Sonnet is genuinely capable** for well-scoped execution. The savings don't come from sacrificing quality — they come from using the right tool for each layer of the work.

## Why use it

Concrete metrics from real projects:

| Task type | With this skill | Opus-only equivalent | Savings |
|---|---|---|---|
| Medium feature (one module) | **$4-10 USD** | $30-60 USD | ~70-80% |
| Large feature (multiple blocks, parallel agents) | **$12-25 USD** | $80-150 USD | ~75-85% |
| Monthly maintenance estimate | **$100-300/mo** | $600-1500/mo | ~75% |

Quality stays equivalent because:
- Opus still makes all architectural decisions.
- Opus audits every deliverable before commit.
- Iteration loop fixes any Sonnet gaps automatically.
- The pattern uses **explicit specs** that don't leave Sonnet to improvise.

## Prerequisites

| Requirement | How to get it | Required |
|---|---|---|
| Claude Code 2.1.139+ | `claude update` (built-in installer) | ✅ |
| API access to Opus + Sonnet | Claude Pro/Team/Max subscription, or Anthropic API key | ✅ |
| Project `CLAUDE.md` | Each project should have one defining its conventions | ✅ |
| `/goal` command | Built-in to Claude Code 2.1.139+ | ✅ |
| `claude-mem` | `npm install -g claude-mem` | Recommended (works without it) |

## Installation

In any Claude Code session:

```bash
/plugin marketplace add github:Maneco26/claude-dev-cycle
/plugin install dev-cycle
```

That's it. The skill is now globally available in all your Claude Code sessions across all projects.

## Usage

In any project, type:

```
/dev-cycle <task description>
```

Examples:

```
/dev-cycle add user profile page with avatar upload to Cloudinary
/dev-cycle migrate authentication from JWT to session cookies with httpOnly
/dev-cycle implement billing module with Stripe Checkout + webhook handler
/dev-cycle fix all the high-priority issues from my GitHub Issues backlog
```

Claude will:

1. Search for historical context (claude-mem if installed).
2. Map relevant files in your codebase.
3. Read your `CLAUDE.md` for conventions.
4. Create a structured plan with `TaskList`.
5. Launch one or more **Sonnet sub-agents in parallel** with detailed specs.
6. Audit deliverables, iterate until clean.
7. Commit + push (if it's a git repo).
8. Update memory (if claude-mem is installed).
9. Report back concisely.

## Combine with `/goal` for unattended work

For long tasks where you don't want to keep typing "continue":

```
/dev-cycle implement complete authentication module
/goal authentication module compiles, has 5+ tests passing, lint clean
```

A separate evaluator model (Haiku by default) checks your condition after every turn. Claude keeps working until it's met.

## How quality stays high

The skill enforces these patterns in the spec it sends to Sonnet:

- **Explicit schemas** — full SQL DDL, exact field names, type definitions.
- **Pattern references** — points to specific existing files Sonnet should mimic.
- **Mandatory verifications** — typecheck + lint + build + tests must pass before reporting back.
- **No surprise commits** — Sonnet never commits; only the Opus coordinator does, after audit.
- **Respect for `CLAUDE.md`** — the skill reads your project's conventions and instructs Sonnet to follow them strictly.

If Sonnet misses something, Opus catches it during audit and uses `SendMessage` to send targeted fixes back to the same agent (preserves context, faster than starting fresh).

## What this skill is NOT

- ❌ Not a replacement for code review by humans for critical production changes.
- ❌ Not magic — bad specs still produce bad code. The skill amplifies your project's existing quality bar.
- ❌ Not a SaaS product. It runs locally in Claude Code with your own API credentials.

## License

MIT. Use, fork, modify, redistribute freely.

## Author

Created by [Maneco26](https://github.com/Maneco26).

Refined in real production projects (Norte ERP, multi-tenant SaaS for Central American SMBs). Methodology validated across dozens of feature implementations.

If you find this useful, ⭐ the repo. If you have improvements, PRs welcome.

## Contributing

PRs welcome for:
- Better spec templates for specific stacks (Next.js, Rails, Django, etc).
- Integrations with other Claude Code skills.
- Documentation improvements.
- Multi-language translations of the SKILL.md.

Keep contributions focused. This skill is intentionally lean — it's a workflow primitive, not a kitchen sink.

## Changelog

### 1.0.0 (2026-05-23)
- Initial public release.
- Single skill `/dev-cycle`.
- Supports claude-mem (optional), TaskList, sub-agent delegation, audit loop.
- Compatible with Claude Code 2.1.139+.

---

**Related**: [Claude Code docs on goal](https://code.claude.com/docs/en/goal) · [Plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces) · [claude-mem](https://github.com/thedotmack/claude-mem)
