---
name: build-roadmap
description: Use when you have brainstorm drafts or informal plans and want to formalize them into a structured roadmap with milestones and tasks — conversational skill that walks through the process collaboratively
---

# Build Roadmap

Transform informal brainstorming outputs into a formal, structured roadmap with milestones and executable task contracts.

**This is a conversational skill, not a generator.** It follows a deliberate cadence — asking questions, proposing groupings, and confirming intent at each step. The drafts are input context, not marching orders.

## When to Use

- After brainstorming sessions that produced rough plans
- When draft files exist in `.claude/roadmap/drafts/`
- When spec files exist in `docs/superpowers/specs/` that should inform the roadmap
- When the user says "let's turn this into a roadmap" or "formalize the plan"

## Pre-Check

1. Verify `.claude/roadmap/` exists. If not: "No roadmap structure found. Run `/setup-roadmap` first."
2. Scan for input sources:
   - `.claude/roadmap/drafts/*.md` — brainstorm drafts
   - `docs/superpowers/specs/*.md` — design specs
3. If no drafts or specs found: ask the user what they'd like to build the roadmap from. They may want to describe it in conversation, paste notes, or point to other files.
4. Check if ROADMAP.md already has milestones defined. If so: warn that this will rebuild the roadmap. Ask if they want to start fresh or add to the existing roadmap. If adding, read existing state first.

## Process

Read the detailed process prompt from `build-roadmap-prompt.md` in this skill's directory and follow it exactly.

The high-level flow:

1. **Read & summarize** — Read all draft/spec files. Present a summary: "Here's what I'm picking up from your brainstorming..."
2. **Propose milestones** — Suggest milestone groupings one at a time. For each: name, description, what it achieves. Confirm before moving to the next.
3. **Break down tasks** — For each confirmed milestone, propose tasks. Discuss scope, complexity (standard vs complex), acceptance criteria. Confirm the task list before moving on.
4. **Plan sequencing** — Propose phases, ordering, parallelism opportunities. Present the battle plan. Confirm.
5. **Generate files** — Once everything is agreed, create all files at once.

## Completion

After generating all files, report:

> Roadmap built for **<project-name>**:
> - **Milestones:** <count> (<list names>)
> - **Tasks:** <count> total across all milestones
> - **Phases:** <count>
>
> Files created:
> - `.claude/roadmap/ROADMAP.md` (updated)
> - `.claude/roadmap/MILESTONE.md` (updated)
> - `.claude/roadmap/TASK.md` (updated)
> - `.claude/roadmap/<milestone>/` directories with task files
>
> Next step: review the roadmap, then start picking up tasks with `/update-task`.
