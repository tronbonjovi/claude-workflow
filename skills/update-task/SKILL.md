---
name: update-task
description: Use when changing task status (in_progress, blocked, review, completed, cancelled), adding notes to a task, or marking a task as blocked with a reason — handles cascade updates to all index files
---

# Update Task

Change task status and add context during or after work sessions. This is the most frequently used workflow skill — it should be fast and minimal.

## When to Use

- Picking up a task to work on (→ `in_progress`)
- Task is blocked by something (→ `blocked`)
- Work is done, needs verification (→ `review`)
- Task verified and complete (→ `completed`)
- Task no longer needed (→ `cancelled`)
- Adding notes or context to a task file

## Input

The user will say something like:
- "mark auth-task002 as completed"
- "auth-task003 is blocked, waiting on API keys"
- "pick up payments-task001"
- "add a note to auth-task002: decided to use JWT instead of sessions"

Parse the intent:
- **Task ID** — extract from the message (e.g., `auth-task002`)
- **Action** — status change or note addition
- **Detail** — blockedBy reason, note content, etc.

## Validation

1. Verify `.claude/roadmap/` exists. If not: "No roadmap found. Run `/setup-roadmap` first."
2. Locate the task file by ID:
   - Task ID format is `<milestone>-task<NNN>`
   - File lives at `.claude/roadmap/<milestone>/<task-id>.md`
   - If not found: "Task `<id>` not found. Check the ID in TASK.md."
3. Read the task file's current frontmatter to get current status.

## Status Transitions

Valid transitions:

| From | To |
|------|-----|
| `pending` | `in_progress`, `cancelled` |
| `in_progress` | `blocked`, `review`, `cancelled` |
| `blocked` | `in_progress`, `cancelled` |
| `review` | `completed`, `in_progress` (if review failed) |
| `completed` | (terminal — no transitions out) |
| `cancelled` | `pending` (if reactivated) |

If the requested transition is not in this table, inform the user: "Can't move from `<current>` to `<requested>`. Valid next statuses: `<list>`."

## Cascade Update

When a task status changes, update ALL of these files in order:

### 1. Individual task file
- Update `status` in frontmatter
- Update `updated` date to today
- If marking `blocked`: set `blockedBy` to the reason provided
- If moving out of `blocked`: clear `blockedBy` to null

### 2. TASK.md (battle plan)
- Find the task row in the phase table
- Update the Status column

### 3. MILESTONE.md
- Find the milestone section for this task
- Recalculate derived status using these rules:
  - All tasks `pending` → milestone `pending`
  - Any task `in_progress` → milestone `in_progress`
  - Any task `blocked` AND no tasks `in_progress` → milestone `blocked`
  - All tasks `completed` or `cancelled` → milestone `review`
  - If `status_override` is set (not null): keep the override, but note the derived status changed
- Update the milestone's status field

### 4. ROADMAP.md
- Find the milestone row in the overview table
- Update Status column to match milestone's current status
- Update Progress column (count completed+cancelled / total tasks)

## Adding Notes

If the user is adding a note (not changing status):
1. Read the task file
2. Append the note to the Notes section with today's date
3. Update `updated` date in frontmatter
4. No cascade needed — notes don't affect status

## Completion

After updating, report concisely:

For status changes:
> `auth-task002` → **completed** (was: in_progress)
> Milestone `auth`: in_progress (2/3 tasks done)

For blocked:
> `auth-task003` → **blocked**: waiting on API keys
> Milestone `auth`: in_progress (1 blocked, 1 in_progress, 1 completed)

For notes:
> Note added to `auth-task002`.

Do NOT summarize what the task is about or restate the acceptance criteria. Keep it to status and impact.
