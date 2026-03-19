# Harness Engineering: A Step-by-Step Adoption Guide for Claude Users

## What Is Harness Engineering?

**The core idea:** Instead of sitting next to an AI agent and guiding it line-by-line (vibe coding), you build *infrastructure around the agent* — a "harness" — so it can work autonomously, correctly, and safely on real tasks without your supervision.

Think of it like the difference between:
- **Vibe coding:** You're pair programming with AI. You prompt, it writes, you review, you prompt again. You are the loop.
- **Harness engineering:** You build the loop itself. The agent picks up work from a backlog, executes in an isolated workspace, validates its own output, and hands off for review. You manage *work*, not *the agent*.

### The Three Pillars

1. **Structured prompts** — Instead of ad-hoc chat, you write a reusable workflow document (like `WORKFLOW.md`) that tells the agent exactly how to behave for every ticket state.
2. **Isolation and safety** — Each task gets its own workspace (git worktree or clone). Agents can't corrupt your main repo or interfere with each other.
3. **Automated orchestration** — A daemon polls your issue tracker, dispatches agents, retries failures, and provides observability. No human in the loop for routine work.

---

## Step-by-Step Adoption (5 Levels)

### Level 1: Structured Prompts (Day 1 — No new tools needed)

**What you're doing:** Moving from ad-hoc prompting to reusable, versioned agent instructions.

**Why it matters:** This is the single highest-leverage harness engineering practice. A good system prompt turns a mediocre agent session into a great one.

**How to do it with Claude:**

1. **Create a `CLAUDE.md` file** in your repo root. This is Claude Code's equivalent of a harness prompt. It gets loaded automatically every session.

2. **Include these sections:**
   ```markdown
   # Project Overview
   - What this project does, tech stack, key directories

   # Conventions
   - Code style, naming, commit message format
   - Test framework and how to run tests
   - Build commands

   # Workflow Rules
   - Always run tests before considering work done
   - Keep changes narrowly scoped
   - Don't modify unrelated code
   ```

3. **Add an `AGENTS.md`** (see `elixir/AGENTS.md` in this repo for an example) with environment setup, quality gates, and required rules specific to your stack.

4. **Version these files in Git.** The prompt is now part of your codebase, reviewed in PRs like any other code.

**Graduation check:** You can hand Claude a task description and it produces code that follows your project conventions without you having to remind it.

---

### Level 2: Validation Harness (Week 1)

**What you're doing:** Making the agent self-validate instead of relying on your eyes.

**Why it matters:** The biggest failure mode of vibe coding is accepting code that *looks* right but doesn't *work*. A validation harness catches this before you even see the output.

**How to do it with Claude:**

1. **Add explicit validation commands to your `CLAUDE.md`:**
   ```markdown
   # Validation (run before considering any task complete)
   - `make lint` or equivalent
   - `make test` — all tests must pass
   - `make typecheck` if applicable
   - For UI changes: describe the manual verification steps
   ```

2. **Write acceptance criteria on your tickets/issues.** Instead of "fix the login bug", write:
   ```
   Fix: Login fails when email contains a + character

   Validation:
   - [ ] `mix test test/auth_test.exs` passes
   - [ ] Manual test: login with "user+tag@example.com" succeeds
   - [ ] No regressions in existing auth tests
   ```

3. **Require reproduction-first workflow.** In your `CLAUDE.md`, add:
   ```markdown
   - Before fixing a bug, first reproduce it and document the reproduction signal
   - Before implementing a feature, confirm the current behavior
   ```

**Key insight from Symphony:** The `WORKFLOW.md` in this repo treats ticket-authored `Validation` and `Test Plan` sections as *non-negotiable acceptance input*. The agent must execute every item before considering work complete. Adopt this mindset.

**Graduation check:** The agent runs your test suite and lint checks without you asking, and it reports pass/fail results before handing off.

---

### Level 3: Workspace Isolation (Week 2-3)

**What you're doing:** Running agent work in isolated directories so it can't corrupt your working tree.

**Why it matters:** Once you start running agents on multiple tasks (or even one task while you work on another), isolation prevents conflicts and gives you clean rollback.

**How to do it with Claude:**

1. **Use git worktrees for isolation:**
   ```bash
   # Create an isolated workspace for a task
   git worktree add ../my-project-TASK-123 -b task-123 main
   cd ../my-project-TASK-123
   # Now run Claude Code here — it works in this isolated copy
   ```

2. **Or use Claude Code's built-in worktree support** if available in your version.

3. **Establish workspace lifecycle hooks** (the concept from Symphony):
   - `after_create`: Install dependencies, set up environment
   - `before_run`: Pull latest main, ensure clean state
   - `after_run`: Run validation, clean up temp files
   - `before_remove`: Archive logs if needed

4. **Keep workspaces deterministic.** Same task = same workspace path. This lets retries resume from where they left off instead of starting over.

**From this repo:** Symphony uses `{workspace_root}/{sanitized_issue_id}` as the path pattern. Each issue gets exactly one workspace, reused across retries. See `elixir/lib/symphony_elixir/workspace.ex`.

**Graduation check:** You can have Claude working on Task A in one terminal while you (or another Claude session) work on Task B in another, with zero interference.

---

### Level 4: Skill Decomposition (Week 3-4)

**What you're doing:** Breaking agent capabilities into modular, reusable "skills" that can be composed.

**Why it matters:** Instead of putting everything in one giant prompt, you create focused instruction sets for specific operations. This makes agents more reliable at complex multi-step workflows.

**How to do it with Claude:**

1. **Create a skills directory** in your repo (Claude Code uses `CLAUDE.md` files in subdirectories, or you can reference separate docs):

   ```
   .claude/
     commands/
       commit.md    — how to make clean commits
       deploy.md    — how to deploy safely
       review.md    — how to self-review code
       test.md      — how to write and run tests
   ```

2. **Each skill file should include:**
   - Clear trigger conditions (when to use this skill)
   - Required inputs
   - Step-by-step procedure
   - Expected output format
   - Example usage

3. **Reference skills from your main workflow.** In Symphony's WORKFLOW.md:
   ```markdown
   ## Related skills
   - `commit`: produce clean, logical commits during implementation.
   - `push`: keep remote branch current and publish updates.
   - `land`: when ticket reaches Merging, follow `.codex/skills/land/SKILL.md`
   ```

4. **For Claude Code specifically**, you can put skill-like instructions in:
   - Project-level `CLAUDE.md` (always loaded)
   - Directory-level `CLAUDE.md` files (loaded when working in that directory)
   - Custom slash commands via `.claude/commands/`

**From this repo:** See `.codex/skills/` — there are skills for `commit`, `push`, `pull`, `land`, `linear`, and `debug`. Each is a focused, self-contained instruction document.

**Graduation check:** You can tell Claude "commit this work" or "prepare this for review" and it follows a consistent, high-quality procedure every time without detailed instructions.

---

### Level 5: Orchestrated Automation (Month 2+)

**What you're doing:** Building or deploying an orchestrator that continuously polls for work and dispatches agents — the full Symphony model.

**Why it matters:** This is the end state of harness engineering. You go from "I use AI to help me code" to "I manage a backlog and AI agents execute it." You review PRs and manage priorities; agents do the implementation.

**How to do it with Claude:**

1. **Set up an issue tracker integration.** Symphony uses Linear, but the concept works with any tracker. The key requirements:
   - Issues have well-defined states (Todo, In Progress, Human Review, Done)
   - Issues have structured descriptions with acceptance criteria
   - You can query issues programmatically

2. **Write your `WORKFLOW.md`** — the central contract. It has two parts:
   - **YAML front matter**: Runtime config (polling interval, concurrency limits, workspace root, agent command, hooks)
   - **Markdown body**: The prompt template with Liquid/Solid variables for issue context (`{{ issue.identifier }}`, `{{ issue.title }}`, `{{ issue.description }}`)

3. **Build or deploy the orchestrator.** You have options:
   - **Use Symphony directly** — follow `elixir/README.md` to run the Elixir reference implementation
   - **Build your own** — point your favorite agent at `SPEC.md` and ask it to implement Symphony in your language of choice
   - **Start simple** — even a bash script that polls your tracker and spawns Claude Code sessions is a valid starting point:
     ```bash
     # Pseudocode for a minimal orchestrator
     while true; do
       issues=$(fetch_todo_issues)
       for issue in $issues; do
         if not_already_running $issue; then
           workspace=$(create_workspace $issue)
           cd $workspace
           claude --prompt "$(render_template $issue)" &
         fi
       done
       sleep 30
     done
     ```

4. **Add observability.** At minimum:
   - Structured logs with issue ID in every line
   - A way to see what's running, what's queued, what failed
   - Token usage tracking (agents are expensive at scale)

5. **Implement the retry/continuation loop.** The key insight from Symphony: if an agent finishes but the ticket is still in an active state, the work isn't done. Schedule a retry with backoff. The workspace persists, so the agent can resume.

**From this repo:** The orchestrator (`elixir/lib/symphony_elixir/orchestrator.ex`) handles polling, dispatch, concurrency limits, retry with exponential backoff, reconciliation (stopping agents when issues become terminal), and rate limiting. The status dashboard provides real-time visibility.

**Graduation check:** You can add issues to your tracker, walk away, and come back to find PRs ready for review.

---

## Key Differences: Claude vs. Codex Adaptation Notes

Symphony was built for OpenAI's Codex, but the harness engineering concepts are agent-agnostic. When adapting for Claude:

| Concept | Codex/Symphony | Claude Equivalent |
|---------|---------------|-------------------|
| Agent prompt | `WORKFLOW.md` body | `CLAUDE.md` + custom commands |
| Skills | `.codex/skills/SKILL.md` | `.claude/commands/` or referenced docs |
| App-server mode | JSON-RPC over stdio | Claude Code CLI (`claude` command) with `--print` or `-p` flag for non-interactive use |
| Approval policy | `codex --approval-policy` | Claude Code permission modes (plan, auto-edit, full-auto) |
| Sandbox | `workspace-write` sandbox | Claude Code sandbox settings |
| Orchestrator | Symphony Elixir service | Build your own, or adapt Symphony to call `claude` instead of `codex` |

**Practical tip:** To use Claude Code as the agent backend in a Symphony-like setup, you'd replace the `codex.command` in WORKFLOW.md with something like:
```yaml
codex:
  command: claude --dangerously-skip-permissions -p
```
(The exact integration depends on Claude Code's current CLI capabilities and your trust model.)

---

## Summary: The Adoption Ladder

```
Level 1: CLAUDE.md           -> Agent knows your project conventions
Level 2: Validation harness   -> Agent proves its own work
Level 3: Workspace isolation   -> Agent can't break your environment
Level 4: Skill decomposition   -> Agent follows reliable procedures
Level 5: Orchestration         -> Agent picks up work autonomously
```

Each level is independently valuable. You don't need to reach Level 5 to benefit. Most teams will see the biggest ROI from Levels 1-2 alone.

---

## Files to Study in This Repo

| File | What You'll Learn |
|------|-------------------|
| `SPEC.md` | The full system architecture — read sections 1-5 for concepts, skim the rest |
| `elixir/WORKFLOW.md` | A production-grade workflow prompt — study the status map, step-by-step flows, and guardrails |
| `.codex/skills/commit/SKILL.md` | How to write a focused skill document |
| `.codex/skills/land/SKILL.md` | A complex multi-step skill (merge workflow) |
| `elixir/lib/symphony_elixir/orchestrator.ex` | The dispatch/retry/reconciliation loop |
| `elixir/lib/symphony_elixir/workspace.ex` | Workspace isolation implementation |
| `elixir/lib/symphony_elixir/prompt_builder.ex` | How issue context gets injected into prompts |
