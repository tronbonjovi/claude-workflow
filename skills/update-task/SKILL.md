---
name: update-task
description: Internal plumbing for cascade status updates — not user-invoked. Called by the orchestrator (work-task) and other skills to keep all index files in sync when task status changes.
---

# Update Task (Internal)

Called by work-task and update-roadmap for cross-file consistency when task statuses change.

## Cascade Update Logic

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
  - Milestone has no tasks (status `planned`) → keep `planned`, no derivation needed
  - All tasks `pending` → milestone `pending`
  - Any task `in_progress` → milestone `in_progress`
  - Any task `blocked` AND no tasks `in_progress` → milestone `blocked`
  - All tasks `completed` or `cancelled` → milestone `review`
  - If `status_override` is set (not null): keep the override, but note the derived status changed
- Update the milestone's status field

### 4. Archive check (only on terminal milestone status)

**This step only runs when the milestone's derived or overridden status is `completed` or `cancelled`.** Skip entirely for any other status (`pending`, `in_progress`, `blocked`, `review`).

When a milestone reaches terminal status:

#### 4a. Ensure ARCHIVE.md exists
- If `.claude/roadmap/ARCHIVE.md` does not exist, create it with the initial structure shown in the ARCHIVE.md Format section below.

#### 4b. Move milestone section from MILESTONE.md to ARCHIVE.md
- Copy the milestone's full section (heading + all bullet points) from MILESTONE.md
- Append it under the `## Archived Milestones` heading in ARCHIVE.md (create this heading if it doesn't exist)
- Remove the milestone section from MILESTONE.md (from its `###` heading through the last bullet before the next `###` or end of section)
- The milestone should be removed from whichever group it was in (`## Active Milestones` or `## Completed Milestones`)

#### 4c. Move task rows from TASK.md to ARCHIVE.md
- Find all task rows belonging to this milestone in TASK.md (match on the Milestone column)
- Append them under the `## Archived Tasks` heading in ARCHIVE.md, grouped under a `### <milestone-name>` subheading with the standard task table header
- If a subheading for that milestone already exists, append rows to the existing table
- Remove the task rows from TASK.md
- If removing all rows from a phase section in TASK.md leaves it with only the table header and no data rows, remove the entire phase section

#### 4d. Update ARCHIVE.md date
- Update the `> Updated:` date line in ARCHIVE.md to today's date

### ARCHIVE.md Format

Structure: `# Archive` > `> Updated: YYYY-MM-DD` > `## Archived Milestones` (each as `### <name>` preserving the full MILESTONE.md section) > `## Archived Tasks` (each milestone as `### <name>` with standard task table rows).

Key rules:
- Tasks grouped by milestone name, not by phase
- Milestone directory and task files on disk are NOT moved or deleted
- ROADMAP.md is synced by `/status`, not by the archive step

### Note: ROADMAP.md

ROADMAP.md is **not** updated by the cascade. It is synced by `/status` on demand. This keeps the execution loop lean — three file writes per status change instead of four.

## Status Transitions

Valid transitions:

| From | To |
|------|-----|
| `planned` | `pending` (when tasks are first defined), `cancelled` |
| `pending` | `in_progress`, `cancelled` |
| `in_progress` | `blocked`, `review`, `cancelled` |
| `blocked` | `in_progress`, `cancelled` |
| `review` | `completed`, `in_progress` (if review failed) |
| `completed` | (terminal — no transitions out) |
| `cancelled` | `pending` (if reactivated) |

If a requested transition is invalid, report the error to the calling skill.

## Task Location

Task ID format is `<milestone>-task<NNN>`. File lives at `.claude/roadmap/<milestone>/<task-id>.md`.

## Adding Notes

If adding a note (not changing status):
1. Read the task file
2. Append the note to the end of the file with today's date
3. Update `updated` date in frontmatter
4. No cascade needed — notes don't affect status
