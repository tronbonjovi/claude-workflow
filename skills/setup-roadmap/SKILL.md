---
name: setup-roadmap
description: Use when starting a new project's workflow tracking or when a project needs the roadmap directory structure initialized — creates .claude/roadmap/ with empty templates
---

# Setup Roadmap

Initialize the file-based workflow structure for the current project.

## When to Use

- Starting a new project that needs workflow tracking
- First time setting up roadmap tracking in an existing project
- The `.claude/roadmap/` directory does not exist yet

## Pre-Check

Before creating anything:

1. Check if `.claude/roadmap/` already exists
   - If it does: inform the user and ask if they want to reinitialize (this would overwrite index files but preserve existing milestone directories and task files)
   - If it doesn't: proceed

2. Detect the project name:
   - Read `package.json` → use `name` field if it exists
   - Otherwise use the current directory name
   - Confirm with the user: "Setting up roadmap for **<project-name>** — is that right?"

## Setup Steps

Once confirmed, create the following structure:

1. Create directories:
   - `.claude/roadmap/`
   - `.claude/roadmap/drafts/`

2. Create `ROADMAP.md` using the template from `templates/ROADMAP.md` in this plugin's directory:
   - Replace `{{PROJECT_NAME}}` with the confirmed project name
   - Replace `{{DATE}}` with today's date (YYYY-MM-DD)
   - Replace `{{VISION}}`, `{{GOALS}}` with placeholder text: "*To be defined — run `/build-roadmap` after brainstorming.*"

3. Create `MILESTONE.md` using the template from `templates/MILESTONE.md`:
   - Replace `{{PROJECT_NAME}}` with the confirmed project name
   - Replace `{{DATE}}` with today's date

4. Create `TASK.md` using the template from `templates/TASK.md`:
   - Replace `{{PROJECT_NAME}}` with the confirmed project name
   - Replace `{{DATE}}` with today's date

## Completion

After creating all files, report what was created:

> Roadmap structure initialized for **<project-name>**:
> - `.claude/roadmap/ROADMAP.md`
> - `.claude/roadmap/MILESTONE.md`
> - `.claude/roadmap/TASK.md`
> - `.claude/roadmap/drafts/` (ready for brainstorm outputs)
>
> Next step: brainstorm your project plan, save drafts to `roadmap/drafts/`, then run `/build-roadmap`.
