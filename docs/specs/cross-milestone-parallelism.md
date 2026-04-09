# Cross-Milestone Parallelism

> Status: Draft
> Date: 2026-04-09
> Scope: Future — not for current release

## Problem

work-task Step 1 picks the **first** `in_progress` or `pending` milestone (line 28-29 of SKILL.md). Independent milestones that could run in parallel are blocked behind each other. This bottleneck is artificial when milestones touch different areas of the codebase.

## Proposed Approach

### Milestone-Level Dependencies

Add an optional `dependsOn` field to milestone definitions in MILESTONE.md:

```
### feature-auth
- **Status:** pending
- **dependsOn:** [core-models]
```

A milestone with no `dependsOn` (or an empty list) has no milestone-level dependencies. This is backward-compatible — existing milestones without the field are treated as independent.

### Ready-Milestone Resolution

Replace the "first matching" logic in work-task Step 1 with a readiness check:

1. Collect all milestones with status `pending` or `in_progress`
2. Filter: a milestone is **ready** if every entry in its `dependsOn` list has status `completed` (or `dependsOn` is empty/absent)
3. Present all ready milestones to the user, not just one

The user still chooses which milestone(s) to work — this preserves the "co-pilot, not autopilot" principle.

### Parallel Dispatch

When multiple milestones are ready, work-task could present tasks from each and let the user approve cross-milestone parallel dispatch. The existing `filesTouch` overlap check (Step 5c) applies across milestones — reject parallel dispatch if files overlap between milestones.

## Impact on Existing Skills

### work-task

- Step 1 changes from single-milestone selection to multi-milestone readiness scan
- Step 2 presentation expands to show ready milestones grouped
- Steps 4-6 are task-level and mostly unchanged — they already handle parallel dispatch
- Scoped reads would need to load task rows for multiple milestones

### update-task

- Cascade logic (Steps 1-3) operates per-task and derives milestone status from task statuses — this works unchanged
- Archive check (Step 4) triggers on terminal milestone status — no change needed since it already handles one milestone at a time
- No cross-milestone cascade exists today and none is needed

### status

- Would benefit from showing which milestones are ready vs blocked-by-dependency
- Minor display change, no logic change

## Open Questions

1. **agent-cc workflow bridge:** Does agent-cc's bridge assume a single active milestone? The bridge reads MILESTONE.md and TASK.md — if it keys on "first in_progress milestone" the same way work-task does today, parallel milestones would confuse it. **This needs verification before implementation.**
2. **Context budget:** Loading task rows for multiple milestones increases token usage. Is this acceptable, or should we cap at N ready milestones?
3. **Milestone ordering:** If multiple milestones are ready, should work-task suggest a priority, or present them neutrally?
4. **dependsOn validation:** Should build-roadmap enforce that `dependsOn` references valid milestone names, or fail gracefully at runtime?

## Recommendation

**Defer.** The current single-milestone model works for projects with sequential milestones (which is most projects today). Implement when a real project hits the bottleneck. The design is straightforward — the main risk is the agent-cc integration, which should be verified first.

When ready to implement: start with the `dependsOn` field and readiness check (low risk), then expand the presentation and dispatch (medium risk, more UX iteration needed).
