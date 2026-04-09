---
name: update-roadmap
description: Use when adding, removing, or modifying milestones and tasks in an existing roadmap — handles structural changes like new milestones, new tasks, reordering phases, or cancelling work after direction changes
---

# Update Roadmap

Modify the roadmap structure after brainstorming revisions, scope changes, new features, or course corrections. Unlike the internal `update-task` logic (which handles individual status changes), this skill handles structural changes to the roadmap itself.

## When to Use

- Adding a new milestone
- Adding new tasks to an existing milestone
- Cancelling a milestone or set of tasks
- Reordering phases or changing parallelism groups
- After a brainstorming session that changes project direction
- Reactivating a cancelled milestone from the archive
- Adjusting milestone scope (splitting, combining, renaming)

## Pre-Check

1. Verify `.claude/roadmap/` exists with ROADMAP.md, MILESTONE.md, and TASK.md. If not: "No roadmap found. Run `/setup-roadmap` to get started."
2. Read current state: ROADMAP.md, MILESTONE.md, TASK.md, and ARCHIVE.md (if it exists) to understand existing structure.
3. If new drafts exist in `.claude/roadmap/drafts/` that haven't been referenced in ROADMAP.md `sources:`, mention them: "I see new drafts that aren't part of the current roadmap: <list>. Should I incorporate them?"

## Operations

Parse the user's request and determine which operation(s) are needed:

### Add Milestone

1. Confirm with the user:
   - Milestone name (lowercase, hyphenated)
   - Description
   - What it achieves
2. Create the milestone directory: `.claude/roadmap/<milestone-name>/`
3. Add milestone section to MILESTONE.md (status: `planned` — no tasks yet, tasks: empty)
4. Ask: "Want to define tasks for this milestone now?" If yes, proceed to Add Tasks.

Note: ROADMAP.md is synced by `/status`, not updated here.

### Add Tasks

1. Confirm milestone the tasks belong to.
2. Write task contracts following the format defined in `/build-roadmap`.
3. Create individual task files using the appropriate template from the plugin's root `templates/` directory.
4. Follow the naming convention from `/build-roadmap`: `<milestone-name>-task<NNN>.md` — continue numbering from the highest existing task number in that milestone.
5. Update MILESTONE.md: add task IDs to the milestone's task list.
6. Update TASK.md: add task rows to the appropriate phase table with all columns (including Complexity and Parallel-Safe). If a new phase is needed, create it.
7. Milestone status transitions to `pending` if it was `planned` (first tasks defined). ROADMAP.md is synced by `/status`.

### Cancel Milestone

1. Confirm with the user: "Cancel milestone **<name>** and all its tasks? This sets status to `cancelled` — files are preserved."
2. Set all task files in the milestone directory to `status: cancelled` (using update-task cascade logic).
3. Update MILESTONE.md: set milestone status to `cancelled`.
4. Update TASK.md: set all task rows for this milestone to `cancelled`.
5. ROADMAP.md is synced by `/status`, not updated here.

### Cancel Tasks

1. Confirm which tasks to cancel.
2. For each task: use the update-task cascade logic, setting status to `cancelled`.

### Reorder Phases

1. Present current phase structure from TASK.md.
2. Discuss desired changes with the user.
3. Update TASK.md phase sections, moving task rows as needed.
4. Update `phase` field in affected individual task files.

### Reactivate Milestone

Moves a previously cancelled milestone from ARCHIVE.md back into the active indexes.

1. Read ARCHIVE.md and present the list of archived milestones to the user. If ARCHIVE.md does not exist or has no archived milestones, inform the user: "No archived milestones found."
2. Confirm with the user: "Reactivate milestone **<name>**? This moves it back to active status as `pending`."
3. Move the milestone section from ARCHIVE.md `## Archived Milestones` back into MILESTONE.md:
   - Insert under `## Active Milestones`
   - Set status to `pending`
   - Preserve all other fields (description, task list, etc.)
4. Move the milestone's task rows from ARCHIVE.md `## Archived Tasks > ### <milestone-name>` back into TASK.md:
   - Place rows into the appropriate phase section (ask user which phase if unclear)
   - Set all task statuses to `pending`
   - If the target phase section doesn't exist in TASK.md, create it with the standard table header
5. Update individual task files on disk: set `status: pending` and `updated:` to today in frontmatter
6. Remove the reactivated entries from ARCHIVE.md:
   - Remove the milestone section from `## Archived Milestones`
   - Remove the milestone's task group from `## Archived Tasks`
   - If `## Archived Milestones` or `## Archived Tasks` has no remaining content, remove those headings
   - If ARCHIVE.md is now empty (only the `# Archive` header and date remain), delete the file
   - Otherwise, update the `> Updated:` date
7. Inform user: "Milestone **<name>** reactivated with <N> tasks set to `pending`." ROADMAP.md is synced by `/status`.

### Override Milestone Status

1. Confirm the milestone and desired status override.
2. Set `status_override` in MILESTONE.md to the requested status.
3. Inform user: "Milestone **<name>** status overridden to **<status>**. Derived status would be **<derived>**." ROADMAP.md is synced by `/status`.

## Consistency Guarantee

After every operation, verify:
- Every task file in a milestone directory is listed in MILESTONE.md's task list
- Every task listed in MILESTONE.md appears in TASK.md
- ROADMAP.md milestone list matches MILESTONE.md sections (synced by `/status`, not enforced here)
- Task files follow naming convention: `<milestone-name>-task<NNN>.md`

- No milestone appears in both MILESTONE.md and ARCHIVE.md simultaneously
- No task row appears in both TASK.md and ARCHIVE.md simultaneously

If any inconsistency is found, fix it and inform the user.
