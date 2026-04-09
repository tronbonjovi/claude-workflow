---
name: draft
description: Use when you have a quick idea, thought, or rough plan to capture — saves it to .claude/roadmap/drafts/ with minimal ceremony, no milestones or tasks created
---

# Draft

Quick-capture an idea to `.claude/roadmap/drafts/`. No milestones, no tasks, no contracts — just a seed for later.

## When to Use

- User has an idea they want to save for later
- User says "draft this", "save this thought", "I have an idea about X"
- User wants to capture something without kicking off the full planning pipeline

## Behavior

1. Take the user's idea as-is. Lightly organize if it's stream-of-consciousness, but don't restructure or formalize.
2. Generate a short title from the core idea (kebab-case for the filename slug).
3. If `.claude/roadmap/` or `.claude/roadmap/drafts/` doesn't exist, create them. Don't require `/setup-roadmap`.
4. Write the draft file.
5. Confirm with a one-liner — no follow-up questions, no "want to do more?".

## Draft File

Path: `.claude/roadmap/drafts/YYYY-MM-DD-<slug>.md`

```markdown
---
title: <Title>
created: YYYY-MM-DD
tags: []
---

<user's idea, captured as-is or lightly organized>
```

## Rules

- Lowest friction skill in the plugin — save and confirm, nothing else
- No confirmation dialogs before saving
- Tags are optional, include only if the user mentions a category
- If a file with the same date-slug already exists, append a number: `YYYY-MM-DD-<slug>-2.md`

## Completion

> Draft saved: `drafts/YYYY-MM-DD-<slug>.md`
