<!--
TEMPLATE METADATA — stripped by BOOTSTRAP.

Variables consumed:
  - {{project_name}}
  - {{current_phase}}
  - {{bootstrap_date}}  (auto-populated by BOOTSTRAP at scaffold time)

This is the seeded NOTES.md — the session-to-session whiteboard. Overwritten freely
in normal use. The template seeds the structure plus the bootstrap-day entry.
-->
# Notes

Informal session-to-session handoff scratchpad. Read this first when starting a session. Overwrite freely — this is not a log, it's a whiteboard.

**What goes here:**
- What was just worked on
- What's half-done and where it was left
- What's next
- Open questions for the human
- Anything the next session needs to know that isn't obvious from the code

**What does _not_ go here:**
- Permanent decisions → `DECISIONS.md`
- Product spec → `docs/reference-guide.md` (or `.pdr.md`)
- Setup instructions → `README.md`
- Structured task state → `TASKS.md`

---

## Current state — {{bootstrap_date}} (project bootstrapped)

**Phase:** {{current_phase}}.

**Just bootstrapped.** The grill skill produced `.pdr.md`, the agent roster, and the initial Phase 0 task list in `TASKS.md`. The all-hands exercise has run — every agent has self-introduced (Phase A) and refined their handoff sketch after seeing the others (Phase B). The handoff-mesh audit is committed at `docs/agents/README.md`.

### What's next

- Pick up the first `open` Phase 0 task in `TASKS.md`. The orchestrator decides which agent claims it based on the task's `Role` field.
- If anything in the all-hands handoff-mesh audit was flagged but not resolved, address it before merging the first feature work.

### Open questions for the human

(none at bootstrap — fill in as they arise)
