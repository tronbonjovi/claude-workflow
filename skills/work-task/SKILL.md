---
name: work-task
description: Use when the user wants to work on tasks, check project status, or continue development — reads the task graph, presents current state, recommends next work, and dispatches subagents on user approval
---

# Work Task

The primary execution interface. Reads the task graph, presents current state, recommends next work, and dispatches subagents on user approval.

**Co-pilot, not autopilot.** Present options and wait for the user to decide. Never auto-dispatch work.

## When to Use

- User says "what's next?", "work the next task", "let's keep going", "status?", "continue"
- Beginning of a session where a roadmap exists with pending tasks
- After completing a task, to present the next options

## Step 1: Assess State

Read the roadmap files to build a complete picture:

1. Read `.claude/roadmap/ROADMAP.md` — project status, milestone list
2. Read `.claude/roadmap/MILESTONE.md` — milestone statuses, task membership
3. Read `.claude/roadmap/TASK.md` — phase ordering, dependencies, parallelism notes, batching notes

Determine:
- Current active milestone(s) and progress
- Tasks that are `completed`, `in_progress`, `blocked`
- Tasks that are `pending` with all dependencies met (ready to work)
- Phase ordering — what phase are we in?
- Parallel opportunities — which ready tasks can run simultaneously?
- Batching opportunities — which sequential simple tasks share context?

## Step 2: Present Status

Concise summary. Don't overwhelm — focus on what matters right now.

> "**Milestone: <name>** — <N>/<M> tasks done. Phase <X>.
> 
> Next up: `<task-id>` — <title> (<complexity>).
> [If parallel opportunity]: `<task-id>` can also run in parallel — different files, no dependency.
> [If batching opportunity]: `<task-id>` and `<task-id>` are sequential and simple — could batch to one subagent.
> [If blocked tasks]: `<task-id>` is blocked: <reason>.
>
> Want me to spin up a subagent for <task-id>? [Or: Want me to dispatch both in parallel?]"

If no tasks are ready (all blocked or all done):
- All done → "Milestone **<name>** is complete. <next milestone or roadmap status>"
- All blocked → "No unblocked tasks. Here's what's blocking: <list>. What do you want to do?"

## Step 3: Wait for User Approval

The user decides:
- Which task(s) to work on
- Whether to run in parallel or sequential
- Whether to batch
- Or to skip, reorder, discuss, or do something else entirely

**Never dispatch without explicit approval.**

## Step 4: Dispatch Subagents

For each approved task:

### 4a. Pre-dispatch
- Mark task `in_progress` (cascade update to TASK.md, MILESTONE.md, ROADMAP.md)
- Read the full task contract file

### 4b. Dispatch
- Spin up a subagent (using the Agent tool) with:
  - The full task contract content as the primary prompt
  - Project context: what the project is, relevant architecture, what was built in prior tasks
  - Working directory
  - Instruction to follow TDD (tests first, then implementation)
  - Instruction to commit work when done
  - Instruction to report: what was implemented, what was tested, files changed, any concerns

### 4c. Parallel dispatch
- If multiple tasks approved for parallel execution:
  - Verify `parallelSafe: true` on all tasks
  - Verify no overlap in `filesTouch` arrays
  - If overlap detected: warn user, suggest sequential instead
  - If clear: dispatch all subagents in a single message (they run concurrently)

### 4d. Batch dispatch
- If sequential simple tasks approved for batching:
  - Combine all task contracts into one subagent prompt
  - Instruct: "Complete these tasks in order. Commit after each."
  - One subagent handles the batch

## Step 5: Handle Results

When a subagent completes:

### 5a. Mark for review
- Update task status to `review` (cascade update)

### 5b. Dispatch reviewer
- Spin up a reviewer subagent to check:
  - Did the implementation match the contract? (compliance)
  - Do tests pass?
  - Was anything built that wasn't requested? (scope creep)
  - Code quality check

### 5c. Review outcome
- **Pass:** Mark task `completed` (cascade update). Report to user.
- **Fail (cycle 1):** Dispatch fix subagent with reviewer's feedback. Re-review after fix.
- **Fail (cycle 2):** Mark task `blocked` with explanation. Inform user:
  > "`<task-id>` failed review twice. Issue: <summary>. What do you want to do?"

### 5d. Max 2 review cycles
After 2 failed reviews, the task is blocked and escalated to the user. Do not loop further.

## Step 6: Continue

After task(s) complete, return to Step 1 — assess the updated state and present next options. The user drives the pace.

## Subagent Dispatch Rules

| Scenario | Approach |
|----------|----------|
| Single task, any complexity | One fresh subagent |
| Multiple parallel tasks | One subagent per task, dispatched simultaneously |
| Sequential simple tasks, same area | Batch into one subagent |
| Task fails or escalates | New subagent for fix (fresh context) |

**Sequential is the default.** Parallel is an optimization recommended when the task graph supports it — not the first instinct.

## Handling Subagent Issues

- **Subagent asks questions:** Answer with context from the codebase. If the question reveals a gap in the task contract, note it for future improvement.
- **Subagent reports BLOCKED:** Assess whether more context helps, a more capable model is needed, or the task needs to be split. Escalate to user if unclear.
- **Subagent reports DONE_WITH_CONCERNS:** Read the concerns. If they're about correctness, address before review. If they're observations, note and proceed to review.
- **Git conflicts (parallel):** If parallel subagents create merge conflicts, resolve them or re-run the conflicting task.
