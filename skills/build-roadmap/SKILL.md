---
name: build-roadmap
description: Use when you have brainstorm drafts or informal plans and want to formalize them into a structured roadmap with milestones and AI-executable task contracts — conversational skill that walks through the process collaboratively
---

# Build Roadmap

Transform informal brainstorming outputs into a formal, structured roadmap with milestones and AI-executable task contracts.

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

1. **Read & summarize** — Read all draft/spec files. Present a summary.
2. **Propose milestones** — Suggest milestone groupings one at a time. Confirm each.
3. **Break down tasks as contracts** — For each milestone, propose tasks with specific instructions, files to touch, tests to write. These are execution contracts for subagents, not descriptions.
4. **Plan sequencing** — Propose phases, ordering, dependencies, parallel opportunities, batching candidates.
5. **Generate files** — Once everything is agreed, create all files at once.

## Critical: Task Contracts Must Be AI-Executable

Task files are dispatch payloads for Claude Code subagents. Each contract must contain:
- **Instructions** specific enough that a subagent can execute without asking questions
- **Files to touch** — explicit list of files to create or modify
- **Tests** — specific test cases to write (TDD: tests first, then implementation)
- **Acceptance criteria** — verifiable checklist
- **Scope boundaries** — what's in and out, to prevent over-building

If a task can't be written this prescriptively, it's either too vague (needs more brainstorming) or too large (needs to be split).

## Naming Convention

Task files MUST follow: `<milestone-name>-task<NNN>.md`
- Example: `auth-task001.md`, `auth-task002.md`
- Sequential numbering starting at 001 within each milestone
- The filename is an ID. The human-readable title lives in frontmatter.

## Completion

After generating all files:

> Roadmap built for **<project-name>**:
> - **Milestones:** <count> (<list names>)
> - **Tasks:** <count> total across all milestones
> - **Phases:** <count>
