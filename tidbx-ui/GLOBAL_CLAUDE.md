## Planning & Review Workflow

### 1. Plan-First Development
- For ANY non-trivial task (3+ steps or architectural decisions), create a plan in `tasks/plan.md` before coding
- Enter plan mode to draft the plan, then present it for user review
- If something goes sideways mid-implementation, STOP and re-plan immediately
- Write detailed specs upfront to reduce ambiguity

### 2. Iterating on Plans with Inline Comments
When the user adds inline comments to a plan, follow this protocol:

**Comment markers the user will use:**
- `> **[Q]:**` — A question that needs answering before proceeding
- `> **[X]:**` — Rejection. This part needs to be reworked
- `> **[!]:**` — Important constraint or requirement to incorporate
- `<!-- TODO: ... -->` — A note for later consideration

**How to respond:**
1. Read the plan file and identify all comment markers
2. Address every `[Q]` with a direct answer or options
3. Rework every section marked with `[X]` based on the feedback
4. Incorporate all `[!]` constraints into the revised plan
5. Preserve `<!-- TODO -->` comments unless explicitly resolved
6. Produce a clean revised plan — remove addressed `[Q]` and `[X]` markers
7. Commit each revision so the full iteration history is in git

**The plan is not final until all comment markers are resolved and the user explicitly approves.**

### 3. Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

## Workflow Orchestration

### 1. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### 2. Self-Improvement Loop
- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

### 3. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes - don't over-engineer
- Challenge your own work before presenting it

### 4. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests - then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

## Task Management

1. **Plan First**: Write plan to `tasks/plan.md` with checkable items
2. **Review & Iterate**: User comments inline using `[Q]`, `[X]`, `[!]` markers — iterate until approved
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to `tasks/plan.md`
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections
