# TidbX UI — Harness Engineering Adoption Plan

## Project Vision

An editor/notebook-centric TiDB Cloud console where the query editor is the primary surface, operational workflows (monitoring, scaling, networking) are promoted *into* the editor rather than hidden in settings, and AI bridges natural language intent to database operations. Traditional settings pages remain as an alternative path for users who prefer a conventional console experience.

See [thoughts.md](./thoughts.md) for the full design rationale.

## Decisions

| Area | Decision |
|------|----------|
| **Framework** | Next.js + React |
| **Styling** | Tailwind CSS |
| **Package manager** | pnpm |
| **State management** | TanStack (Query + Router) |
| **Editor component** | CodeMirror 6 via `@uiw/react-codemirror` + `@codemirror/lang-sql` |
| **Linting** | Biome (lint + format) |
| **Testing** | Vitest + React Testing Library |
| **Type checking** | TypeScript strict mode |
| **Issue tracker** | Linear |
| **Repo** | https://github.com/likidu/tidbx-ui (separate from Symphony) |
| **Scope** | Prototype first, iterate toward production readiness |
| **Team** | Solo + AI agents |

> **Editor rationale:** DuckDB UI uses CodeMirror 6 for their SQL editor. It's modular, tree-shakeable, and `@codemirror/lang-sql` supports dialect-specific completions (including MySQL, which is closest to TiDB's SQL dialect). Much lighter than Monaco (~50KB vs ~2-4MB), and extensible enough to grow with the project.

---

## Level 1: Structured Prompts (Day 1)

**Create `CLAUDE.md` in the tidbx-ui repo root:**

```markdown
# TidbX UI

## Overview
Editor-centric TiDB Cloud console UI. The notebook/query editor is the primary
surface — operational workflows (monitoring, scaling, networking) are accessed
through the editor, with traditional settings pages as an alternative path.

## Tech Stack
- Next.js + React + TypeScript (strict)
- Tailwind CSS for styling
- TanStack Query + Router for state/routing
- CodeMirror 6 for the SQL editor
- pnpm for package management

## Conventions
- Biome for linting and formatting
- Commit messages: conventional commits (feat/fix/refactor)
- Tests: Vitest + React Testing Library

## Build & Run
- `pnpm install`
- `pnpm dev` — start dev server
- `pnpm build` — production build

## Validation (run before considering any task complete)
- `pnpm lint` — Biome lint + format check
- `pnpm typecheck` — TypeScript strict
- `pnpm test` — Vitest
- `pnpm validate` — runs all three above
- For UI changes: describe visual verification steps

## Workflow Rules
- Always run validation before considering work done
- Keep changes narrowly scoped to the ticket
- Reproduce current behavior before changing it
- Don't modify unrelated code
```

**Create `AGENTS.md`** with environment setup, quality gates, and stack-specific rules (modeled on `elixir/AGENTS.md`).

---

## Level 2: Validation Harness (Week 1)

1. **Set up Biome** — single tool for lint + format, faster than ESLint
2. **Set up Vitest** — with a `pnpm validate` one-liner that runs lint + typecheck + tests
3. **Write Linear ticket templates** with explicit validation sections:
   ```
   ## Task: [title]
   ## Description: [what and why]
   ## Acceptance Criteria:
   - [ ] ...
   ## Validation:
   - [ ] `pnpm validate` passes
   - [ ] Manual: [specific UI verification steps]
   ```
4. **Add reproduction-first rule** to CLAUDE.md — before fixing/building, confirm current state

---

## Level 3: Workspace Isolation (Week 2-3)

1. **Use git worktrees** for parallel task work:
   ```bash
   git worktree add ../tidbx-ui-TASK-123 -b task-123 main
   ```
2. **Add hooks** to CLAUDE.md or a Makefile:
   - `after_create`: `pnpm install`
   - `before_run`: `git pull origin main`
   - `after_run`: `pnpm validate`
3. **Workspace path convention**: `~/code/tidbx-ui-workspaces/{issue-id}`

---

## Level 4: Skill Decomposition (Week 3-4)

Create `.claude/commands/` with focused skills:

| Skill | Purpose |
|-------|---------|
| `commit.md` | Clean conventional commits (adapt from symphony's commit skill) |
| `component.md` | How to create a new UI component (file structure, tests, exports) |
| `review.md` | Self-review checklist before PR |
| `design-to-code.md` | How to translate a design spec into implementation |

---

## Level 5: Orchestration (Month 2+)

1. **Linear integration** — use Linear's API or CLI for issue tracking
2. **Write a `WORKFLOW.md`** adapted from symphony's, simplified for solo work:
   - States: `Todo -> In Progress -> Review -> Done`
   - Single workpad comment for progress tracking
   - Validation gates before moving to Review
3. **Start with a simple dispatch script** (or adapt Symphony to use `claude` instead of `codex`)
4. **Add observability** — structured logs, token usage tracking

---

## Milestones

- **M0**: Scaffold the project (Next.js + Biome + Vitest + CLAUDE.md) — this is Level 1
- **M1**: Build the core editor/notebook shell — CodeMirror 6 SQL editor that can execute queries against a mock backend
- **M2**: Add inline result rendering — query results appear as widgets below the editor cell (tables, charts)
- **M3**: Add operational commands — `SHOW CLUSTER STATUS` returns a mock monitoring widget inline
- **M4**: AI integration — natural language queries that get translated to SQL or operational commands
