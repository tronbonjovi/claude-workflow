---
name: adopt-project
description: Use when bringing an existing, in-progress project into the claude-workflow system — scans the repo, identifies planning artifacts and project state, then bootstraps the workflow structure
---

# Adopt Project

Onboard an existing project into the claude-workflow system. Scans the repo, presents findings, then bootstraps the roadmap structure with existing planning artifacts as input.

**Safety first.** Read-only during discovery. Copies files to drafts — never moves or modifies originals. All file creation requires user approval.

## When to Use

- Existing project needs workflow tracking for the first time
- Mid-project adoption — work is already in progress
- User says "adopt this project", "set up workflow here", "onboard this repo"

## Pre-Check

1. If `.claude/roadmap/ROADMAP.md` already exists → "This project already has a roadmap. Use `/work-task` to continue, `/status` to check progress, or `/update-roadmap` to modify the roadmap." Stop here.
2. If no `.git/` directory → warn the user but allow them to proceed.

## Phase 1: Discover

Read-only scan. Skip any source that doesn't exist — no errors for missing files. **Run 1a, 1b, and 1c in parallel where possible** — they are independent reads.

### 1a. Project Identity

Read in priority order (stop at first match for each field):

- **Name:** `package.json` → `name`, `pyproject.toml` → `project.name`, `Cargo.toml` → `package.name`, else directory name
- **Description:** `package.json` → `description`, `pyproject.toml` → `project.description`, README first paragraph. If none found, leave blank.
- **Tech stack:** infer from dependency files (`package.json`, `requirements.txt`, `pyproject.toml`, `Cargo.toml`, `go.mod`) and config files (`tsconfig.json`, `tailwind.config.*`, `vite.config.*`, `docker-compose.*`). If no manifest or config files found, report "Stack: not detected from config files."

### 1b. Project State

- `git log --oneline -20` — recent activity patterns
- `git status` — uncommitted work

### 1c. Planning Artifacts

Search for files that contain plans, tasks, or design intent — these are candidates for copying to drafts:

- `TODO.md`, `TODO.txt`, `TODO` — task lists
- `ROADMAP.md`, `PLAN.md`, `PLANNING.md` — project plans
- `docs/**/*.md` — specs, designs, architecture docs. Glob first: if more than 10 matches, scan filenames only and present the list to the user for selection. If 10 or fewer, read first 5 lines of each. Look for files whose names or first lines suggest planning content (words like spec, design, architecture, plan, proposal, RFC, ADR).

Also note these for context (read for understanding but don't copy to drafts):
- `CHANGELOG.md` — what's already been done
- `README.md` — project purpose and setup
- `CLAUDE.md` — existing Claude instructions

## Phase 2: Present

Concise summary. This is a checkpoint — user confirms before any files are created.

> **Project: <name>**
> <description>
>
> **Stack:** <languages, frameworks, key dependencies — or "not detected" if none found>
> **State:** <active development / maintenance / early stage — based on git log>
> **Recent activity:** <summary from git log, plus note of uncommitted work if any>
>
> **Planning artifacts found:**
> - `<file>` — <what it appears to be>
> - *(none found)* — if no planning files exist
>
> Ready to set up the workflow? I'll create the roadmap structure and copy planning files into drafts. Your existing files won't be touched.

Wait for user confirmation.

## Phase 3: Bootstrap

After user approval:

### 3a. Initialize Structure

Create the roadmap structure directly (this skill handles its own confirmation):
- Create `.claude/roadmap/` and `.claude/roadmap/drafts/`
- Create ROADMAP.md, MILESTONE.md, TASK.md from templates in the plugin's `templates/` directory
- Replace `{{PROJECT_NAME}}` with discovered name, `{{DATE}}` with today's date

### 3b. Copy Planning Artifacts to Drafts

For each planning artifact identified in Phase 1c (only the "copy to drafts" candidates):
- **Copy** to `.claude/roadmap/drafts/<original-filename>`
- On name collision, prefix with hyphenated source path: a file at `docs/specs/design.md` becomes `docs-specs-design.md`
- Never move, rename, or modify the original

### 3c. Enrich ROADMAP.md

Update the generated ROADMAP.md with discovered context:
- Fill in Vision: 1-2 sentences derived from README or project description. If nothing discoverable, leave the template placeholder.
- Fill in Goals if clearly stated in planning artifacts. If not, leave the placeholder.
- Set `sources:` to list all files copied to drafts

### 3d. Report and Hand Off

> Workflow initialized for **<project-name>**:
> - Roadmap structure created at `.claude/roadmap/`
> - <N> planning artifact(s) copied to drafts
> - Originals untouched
>
> Next step: run `/build-roadmap` to formalize your drafts into milestones and task contracts.
