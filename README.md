# claude-workflow

A file-based development workflow plugin for Claude Code. Turns brainstorming sessions into structured roadmaps, milestones, and AI-executable task contracts.

## What It Does

- **setup-roadmap** — Initialize the workflow structure for a project
- **build-roadmap** — Transform brainstorm drafts into a formal roadmap with milestones and task contracts
- **update-roadmap** — Add/remove/modify milestones and tasks after direction changes
- **work-task** — Orchestrator: reads the task graph, presents status, dispatches subagents on your approval

## How It Works

The plugin creates a `.claude/roadmap/` directory in your project with:

- `ROADMAP.md` — Big picture: vision, goals, milestone overview
- `MILESTONE.md` — Index of all milestones with derived status
- `TASK.md` — Battle plan: execution order, phases, parallelism, batching
- `<milestone>/` — Directories containing individual task contract files

Tasks are contracts designed for Claude Code subagents to execute autonomously. Each contract contains specific instructions, files to touch, tests to write, and acceptance criteria. The orchestrator (`work-task`) reads the task graph, recommends what's next, and dispatches subagents when you approve.

**Co-pilot, not autopilot.** You make the decisions — what to work on, how many in parallel, when to pause. Claude handles execution and bookkeeping.

## Installation

```bash
claude plugin marketplace add <path-to-plugin-or-github-url>
claude plugin install claude-workflow
```

## Workflow

1. Brainstorm freely — save rough plans to `roadmap/drafts/`
2. Run `/build-roadmap` to formalize into structured roadmap with task contracts
3. Run `/work-task` to see status and dispatch subagents for task execution
4. Revisit and adjust — run `/update-roadmap` after new brainstorming

## Compatibility

Designed for integration with [agent-cc](https://github.com/tronbonjovi/agent-cc). Task files use YAML frontmatter compatible with agent-cc's task parsing and Kanban board. Status updates cascade across all index files, so agent-cc's file watcher sees changes in real-time.
