---
name: dev-cycle
description: Autonomous Opus-Sonnet-loop development workflow. Opus plans the work and audits results, Sonnet sub-agents execute the specs in parallel. Cuts API costs by ~70% vs Opus-only while keeping quality equivalent. Combines claude-mem for historical context, TaskList for planning, sub-agents for parallel execution, and /goal for automatic loop termination.
trigger: /dev-cycle
---

# /dev-cycle — Autonomous Opus + Sonnet development loop

> Cost-optimized development workflow for Claude Code. Pattern refined in real
> production projects: **Opus plans + audits, Sonnet executes**. Typical ~70%
> token savings vs Opus-only with equivalent code quality.

## When to use

User explicitly invokes `/dev-cycle <task description>`.

Good for:
- Implementing a new feature or module.
- Fixing tickets / issues from a backlog.
- Refactoring an area of code.
- Completing pending TODOs.
- Any development task that touches backend + UI together.

## When NOT to use

- Trivial changes (1-2 lines, a single edit). Just do it directly.
- Conceptual questions, explanations, strategy discussions.
- Tasks already in progress that just need continuation.
- One-shot operations (e.g., bump dependency, format file). Direct tools are faster.

## Cost model

This skill explicitly splits work to save tokens:

- **Opus** (current model, the one reading this): planning, architectural decisions,
  audit, critical debugging, strategic commits.
- **Sonnet 4.6** (sub-agents): executing clear specs (server actions, SQL migrations,
  CRUD pages, UI components, tests).

Sonnet is ~5x cheaper per token than Opus. For well-specified tasks it keeps
80-90% of the quality. The key is the spec must be **explicit**: full schema,
exact file names, example patterns to copy.

## Prerequisites

- **Claude Code 2.1.139+** (for `/goal` command and plugin marketplace).
- **API access to Opus + Sonnet** (Claude Pro/Team/Max or Anthropic API).
- **claude-mem** installed globally (recommended; the cycle works without it but
  gets less historical context). Install: `npm install -g claude-mem`.
- The project must have a `CLAUDE.md` file describing its conventions, stack, and
  rules. The skill respects whatever the project defines.

## The cycle, step by step

When invoked as `/dev-cycle <task>`, execute IN ORDER:

### Step 1 — Historical context (claude-mem if available)

If the `claude-mem:mem-search` skill is available, invoke it with keywords related
to the task. Look for:
- Recent observations (`type: observations`).
- Decisions (`obs_type: decision`).
- Recent changes (`obs_type: change, dateStart: 7 days ago`).

If there are critical observations related to the task area, read them with
`get_observations([ids])` before planning.

**Never plan blindly**: always check what was done before in the task area.
Especially:
- Is another parallel session working on something nearby?
- Are there recorded decisions that affect the approach?
- Is there a similar module/table/RPC that could be reused?

If `claude-mem` is not installed, skip this step silently. Don't fail.

### Step 2 — Codebase mapping

Before planning, do:
- `Grep` for task keywords across the relevant directories.
- Identify the closest existing module that serves as a pattern.
- List the exact files that will be touched.

Read the project's `CLAUDE.md` if you haven't yet. Respect its conventions
strictly.

### Step 3 — Structured plan with TaskList

Create tasks with `TaskCreate`:
- One task per logical unit of work (not per individual file).
- Each task with detailed description: schema, exact files, server actions,
  pages, verifications.
- If the task is large (>4h estimated), divide it into blocks that can run in
  parallel.

Typical plan structure:
1. Schema migration (if applicable).
2. Backend layer (actions, RPCs, server code).
3. Worker handlers (if applicable).
4. Pages + UI components.
5. i18n / translations.
6. Verification (typecheck, lint, build, REST tests).
7. Commit (if in a git repo) + memory update.

### Step 4 — Delegate to Sonnet sub-agents

Launch `Agent` with:
- `subagent_type`: pick the most appropriate (general-purpose, Backend API
  Builder, Page Builder, UI Component Builder, Test Writer, etc., depending
  on what the project defines in `.claude/agents/`).
- **`model: "sonnet"`** ALWAYS for sub-agents (this is the cost-saving lever).
- Prompt MUST be VERY explicit:
  - Project technical context (rules from CLAUDE.md).
  - Patterns to copy (references to existing files).
  - Full schema if applicable.
  - Exact list of files to create/modify.
  - Mandatory verifications at the end (typecheck, lint, tests).
  - **"DO NOT make commits — the coordinator (Opus) does that at the end"**.
  - **"DO NOT ask questions — you are autonomous"**.

If the task is parallelizable, launch MULTIPLE agents in ONE call (multiple
tool_use blocks in a single message).

### Step 5 — Audit the deliverable

When the agent reports back, review:

1. **Automated checks reported**:
   - `typecheck` should pass clean.
   - `lint` should pass.
   - Migrations applied (if applicable).
   - Tests passing.

2. **Code quality (sample inspection)**:
   - Open 1-2 critical modified files.
   - Verify it follows project patterns defined in `CLAUDE.md`.
   - Check for common issues: hardcoded secrets, missing error handling,
     unused imports, anti-patterns flagged by the project's conventions.

3. **Independent verification**:
   - Run the project's verification commands yourself (typecheck, lint, build).
   - For UI-critical tasks, consider running a production build to catch
     server/client boundary errors that lint may miss.

4. **Common gotchas to look for**:
   - Imports missing after Edit operations (validate with grep).
   - Type errors hidden by `any` casts.
   - Permissions/auth flow bypassed.
   - Database constraints that the agent may have missed.

### Step 6 — Iterate if issues found

If audit found problems:

1. DO NOT restart from scratch. Continue with the same agent using `SendMessage`:
   ```
   SendMessage(to: agentId, prompt: "Detected X. Fix Y specifically. Don't touch Z.")
   ```
2. The agent retains context, more efficient than a fresh agent.
3. Repeat audit.
4. Loop until quality is approved.

If after 2-3 iterations the agent doesn't resolve it, **fix it yourself (Opus)
directly**. Signal that the problem is ambiguous and needs judgment Sonnet
doesn't have.

### Step 7 — Commit + push (if in git repo)

Only when the audit is clean:

```bash
git add <specific files, not -A>
git commit -m "..." (Conventional Commits)
git push
```

Commit message:
- Executive summary in the first line.
- List of structural changes.
- Verifications passed.
- Co-Authored-By: Claude Sonnet 4.6 + Opus.

If the project is NOT a git repo, skip this step.

### Step 8 — Update memory (if claude-mem is available)

If the task was significant (>2h or touched architecture):

1. If there's an existing session doc in `docs/MEMORY/` or similar, update it.
2. Otherwise, create a new doc with:
   - Executive summary.
   - Changes made.
   - Decisions recorded.
   - Pending TODOs.
   - Useful commands to resume.

### Step 9 — Final report to user

Concise message:
- What was done (1-3 lines).
- Verifications passed.
- Commit hash + push status (if applicable).
- Pending items if any remain.
- Open question if applicable.

## Combination with /goal

If the task has a **clear and verifiable success condition**, after invoking
`/dev-cycle <task>`, you can also use `/goal` so the cycle does NOT terminate
until the condition is met:

Example:
```
/dev-cycle implement module X
/goal module X compiles, tests pass, typecheck/lint OK
```

Claude works until the evaluator (Haiku) confirms the condition. Useful for
long tasks that would otherwise require the user to type "continue" several
times.

## Anti-patterns to avoid

**DO NOT**:
- Launch Sonnet Agent without a detailed spec → poor results.
- Skip the claude-mem step → possible re-work of something already done.
- Commit without auditing → bugs in production.
- Make sub-agents commit code → coordinator handles commits to keep history clean.
- Use Opus model on sub-agents → defeats the entire cost-saving purpose.

**DO**:
- Give Sonnet explicit specs with full schema + exact file names.
- Run typecheck AFTER each large block of changes.
- Clean test data after E2E verification.
- Document pending TODOs in a memory file for future sessions.
- Respect the project's `CLAUDE.md` conventions strictly.

## Project-agnostic by design

This skill works on ANY tech stack because the methodology is universal:

- Next.js / React / Vue / Svelte projects.
- Node.js / Python / Go / Rust backends.
- Mobile (React Native, Flutter, native).
- Any project with claude-mem or without.
- Any project with `.claude/agents/` configured or using the default `general-purpose` sub-agent type.

The skill reads each project's `CLAUDE.md` and respects whatever conventions are
defined there. It doesn't impose patterns — it amplifies the project's own
patterns by delegating execution efficiently.

## Expected metrics per cycle

For a typical task (one new module or medium feature):
- Opus tokens (planning + audit): ~80-150k → ~$3-6 USD.
- Sonnet tokens (1-3 sub-agents): ~150-500k → ~$1-4 USD.
- **Typical total: $4-10 USD per cycle** vs $30-60 USD if all Opus.
- Time: 30-90 min depending on size.

For a large task (full new module or several blocks):
- Opus tokens: ~200-400k → ~$8-15 USD.
- Sonnet tokens (3-5 parallel sub-agents): ~500k-1.5M → ~$4-10 USD.
- **Typical total: $12-25 USD** vs $80-150 USD all Opus.

## Example invocation

User:
```
/dev-cycle add Google OAuth login to my Next.js app
```

Expected behavior:

1. `Skill claude-mem:mem-search` with query "Google OAuth" if claude-mem installed.
2. `Grep` for "next-auth" / "auth" / "session" in repo to detect existing setup.
3. `Read` of project's `CLAUDE.md` for conventions.
4. `TaskCreate` with plan:
   - Backend: NextAuth provider + callback route.
   - Schema: user table extensions if needed.
   - Pages: login button + callback handler.
   - i18n strings.
   - Tests.
5. `Agent(subagent_type: "general-purpose", model: "sonnet", prompt: <detailed spec>)`.
6. Receive deliverable.
7. Audit: typecheck, lint, sample inspection.
8. If OK → commit + push (if git repo) + memory. If not → SendMessage with fixes.
9. Final report.

## Final note

This skill encapsulates a methodology refined in real production projects. It
makes Claude Code economically viable for ongoing development. The patterns work
because Sonnet 4.6 is genuinely capable for well-scoped execution; the cost
saving doesn't come from sacrificing quality, it comes from using the right tool
for each layer of the work.

If something is unclear, the skill defers to the project's `CLAUDE.md` and
makes reasonable decisions following established repo patterns.
