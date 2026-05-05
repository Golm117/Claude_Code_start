<!--
TEMPLATE METADATA — stripped by BOOTSTRAP.

Variables consumed:
  - {{project_name}}
  - {{phases}}                     (rendered in the "Phases" section as ID + goal)
  - {{tasks_per_phase}}            (grill-produced task breakdown, seeded here)

The header (protocol + status values + entry format) is universal.
The active tasks section at the bottom is grill-seeded with the actual Phase 0 tasks
for this specific project.
-->
# Tasks

Shared task list. Any agent (Claude Code or a spawned subagent) reads this before starting work and updates it as they progress. This is the coordination substrate — it's how parallel agents avoid stepping on each other.

## How to use this file

**Before starting work:**
1. Read this file top to bottom.
2. Check for tasks marked `claimed` — someone else is on it, leave it alone.
3. Check `blocked` tasks — a blocker might have cleared.
4. Pick a task marked `open` that fits your role, or add a new one.

**When claiming a task:**
- Change status from `open` → `claimed`.
- Fill in `owner` with your agent name.
- Add a `claimed_at` timestamp (UTC ISO 8601, e.g., `2026-04-22T01:30:00Z`).
- **Commit this change immediately** before starting actual work. This is the atomic claim.

**While working:**
- Update `progress` with brief notes as milestones are hit.
- If you discover the task needs to split, mark the original `split` and add the new tasks below.
- If you're spawned in a worktree, note the worktree path and branch in `notes`.

**When finishing:**
- Change status to `review` (if a reviewer is needed) or `done`.
- Add `completed_at` timestamp on the `review` → `done` transition.
- Write a one-line handoff in `notes` for whoever picks up the next related task.
- Commit.

**If blocked:**
- Change status to `blocked`.
- Write what's blocking in `notes`.
- Flag it in `NOTES.md` if human input is needed.

## Status values

- `open` — unclaimed, ready for someone to pick up
- `claimed` — actively being worked on
- `blocked` — waiting on something (human input, another task, external)
- `review` — code written, awaiting review before merge
- `done` — complete and merged
- `split` — was replaced by smaller tasks, see entries below it
- `cancelled` — decided not to do it, with reason in notes

## Priority values

- `P0` — blocker for current phase, drop other things
- `P1` — core work for current phase
- `P2` — nice-to-have / next phase prep

## Task entry format

```
### T-### — Short title
- **Status:** open | claimed | blocked | review | done | split | cancelled
- **Priority:** P0 | P1 | P2
- **Role:** which agent role is best suited
- **Owner:** <agent name, only when claimed+>
- **Phase:** 0 | 1 | 2 | ...
- **Created:** YYYY-MM-DDTHH:MM:SSZ
- **Claimed:** YYYY-MM-DDTHH:MM:SSZ
- **Completed:** YYYY-MM-DDTHH:MM:SSZ
- **Depends on:** T-### (if any)
- **Description:** what needs doing and why
- **Acceptance:** how we know it's done
- **Progress:**
  - timestamp — note
- **Notes:** handoff info, worktree path if applicable, blockers, etc.
```

## Conventions for parallel work

- **Atomic claim = the commit that changes `open` → `claimed`.** If two agents try to claim the same task, git surfaces the conflict — first commit wins. Loser re-reads and picks a different task.
- **One owner per task.** If a task needs two roles, split it into two tasks with a `depends on` link.
- **Worktrees get a task per worktree.** Don't have one task span multiple worktrees.
- **Always commit the task file update before starting the actual work.** Otherwise a concurrent agent can't see you've claimed it.

---

## Phases (from `.pdr.md`)

{{#each phases}}
- **{{this.id}}: {{this.name}}** — {{this.goal}}
{{/each}}

---

## Active tasks

{{tasks_per_phase}}

<!--
The grill seeds this section with the Phase 0 task list. Subsequent tasks (Phase 1+)
are added by agents as they're identified, or by the human as the project advances.
Each task uses the entry format above.
-->
