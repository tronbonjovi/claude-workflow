# Build Roadmap — Detailed Process

This document defines the conversational flow for building a roadmap. Follow these steps in order.

## Step 1: Read and Summarize

Read all available input sources:
- All files in `.claude/roadmap/drafts/`
- All files in `docs/superpowers/specs/` (if they exist)

Present a summary to the user:

> "I've read through your brainstorming notes. Here's what I'm picking up:
>
> **Key themes:** [list the main ideas/features/goals you identified]
>
> **Potential scope:** [your sense of how big this is]
>
> **Questions already:** [anything unclear or contradictory in the drafts]
>
> Does this capture it, or am I missing something?"

Wait for the user's response. Adjust your understanding based on their feedback.

## Step 2: Propose Milestones

Based on the drafts and conversation, propose milestones **one at a time**:

> "I think the first milestone should be **<milestone-name>**: <description>.
>
> This would cover: <what achieving this milestone means>
>
> Does this feel right as a milestone, or should it be scoped differently?"

For each milestone, confirm:
- The name (lowercase, hyphenated — this becomes the directory name)
- The description
- What it achieves (the "point of culmination")

After confirming a milestone, move to the next. Continue until all milestones are identified.

Then present the full milestone list:

> "Here's the full milestone plan:
> 1. **<name>** — <description>
> 2. **<name>** — <description>
> ...
>
> Does this order make sense? Should any be reordered, combined, or split?"

## Step 3: Break Down Tasks

For each confirmed milestone, propose tasks:

> "For the **<milestone-name>** milestone, I'm thinking these tasks:
>
> 1. **<task-title>** — <one-line description>. <standard/complex>.
> 2. **<task-title>** — <one-line description>. <standard/complex>.
> ...
>
> For each task, I'll write up the full contract (context, objective, acceptance criteria, scope). But first — does this task list cover what's needed for this milestone?"

After the user confirms the task list for a milestone, write out the full content for each task. For each task, present:
- Context
- Objective
- Acceptance criteria (as a checklist)
- Scope (in/out)
- Technical approach and test plan (if complex)

Confirm: "Does this task contract capture what you want? Anything to adjust?"

Move to the next milestone's tasks after confirmation.

## Step 4: Plan Sequencing

Once all milestones and tasks are confirmed, propose the phase/execution plan:

> "Here's how I'd sequence the work:
>
> **Phase 1: <name>**
> These need to happen first: <task list with brief rationale>
>
> **Phase 2: <name>** (parallel opportunities noted)
> Once phase 1 is done: <task list>
> <task-a> and <task-b> can run in parallel because <reason>
>
> ...
>
> Dependencies:
> - <task-b> depends on <task-a> because <reason>
> - <task-d> depends on <task-c> because <reason>
>
> Does this match your instinct for how to approach the work?"

Adjust based on feedback.

## Step 5: Generate Files

Once everything is confirmed, generate all files:

1. **Update ROADMAP.md:**
   - Fill in vision and goals from the conversation
   - Set `sources:` to list all draft files that were read
   - Add all milestone names to `milestones:` list
   - Populate the milestone overview table

2. **Update MILESTONE.md:**
   - Add a section for each milestone with: status (pending), description, directory, task list, created date, status_override: null

3. **Create milestone directories:**
   - `.claude/roadmap/<milestone-name>/` for each milestone

4. **Create individual task files:**
   - Use `templates/task-standard.md` or `templates/task-complex.md` from the plugin's root directory (the `claude-workflow/templates/` folder, not the skill's own folder)
   - Replace all `{{PLACEHOLDER}}` values with the confirmed content
   - Set `dependsOn` based on the dependencies identified in step 4
   - Set `phase` based on the phase plan

5. **Update TASK.md:**
   - Create phase sections with tables
   - Include parallelism notes
   - Populate all task rows with: Task ID, Milestone, Status (pending), Dependencies

Generate all files, then report completion as described in the main SKILL.md.
