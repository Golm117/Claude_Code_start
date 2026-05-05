# All-hands exercise — Phase B: Introductions and re-evaluation

Run this prompt **once per agent**, after every agent has completed Phase A.

Phase B is the "introductions" phase. Each agent now sees every other agent's Phase A output and refines their own handoff sketch in light of who else is on the team. They also write Sections 4, 5, and 6 of their intro.

---

## Setup

For each agent in `agent_roster`:

1. Spawn the agent (same `subagent_type` as in Phase A).
2. Pass it the prompt below, with `{{agent_name}}` substituted.
3. The agent reads every other agent's Phase A intro at `docs/agents/<other-agent>.md`.
4. The agent appends Sections 4, 5, and 6 to its own `docs/agents/<agent-name>.md` file.
5. Commit each Phase B intro independently: `[all-hands] {{agent_name}} Phase B intro`.

Order doesn't matter as long as every agent has completed Phase A before any starts Phase B.

---

## The prompt to pass to each agent

```
You are {{agent_name}}. Phase A of the all-hands exercise is complete — every agent has written their Sections 1–3 self-introduction. Your task right now is **Phase B**: read everyone else's Phase A and write Sections 4, 5, and 6 of your own intro.

Read these first:
- Your own current `docs/agents/{{agent_name}}.md` (Sections 1–3 you wrote in Phase A)
- Every other agent's intro at `docs/agents/<other-agent>.md`
- Your scaffolded `.claude/agents/{{agent_name}}.md` (especially the `initial_handoffs_to` field)

Now write Sections 4, 5, and 6, appending to `docs/agents/{{agent_name}}.md` in this format:

## 4. Who I hand off to and when

(For each handoff: name the receiving agent, the specific condition that triggers the handoff, and what artifact crosses the boundary. Cross-reference what the receiving agent wrote in their Section 3 — if they said they don't accept the kind of work you're trying to hand off, that's a real friction point worth naming. Refine your `initial_handoffs_to` against the actual roster you now see. If you discover handoffs that weren't in your initial sketch, add them. If you see a handoff that overlaps with another agent's lane awkwardly, name it explicitly.)

## 5. How to ask me for work well

### Good prompt example

(A concrete, project-specific example of a request that lets you do high-quality work without round-trips. Reference real entities, surfaces, or deliverables from this project. Show the structure: branch/task ID, acceptance criteria, area of concern, scope boundaries.)

### Bad prompt example — and why

(A concrete example of a prompt that wastes your time, with rationale. "Can you review the code?" is the kind of failure mode to name — no branch, no acceptance criteria, no scope. Whatever the failure modes are for YOUR role, name the worst one.)

### Context I always need

(3–5 bullets of the minimum context you need every time you're invoked, beyond the prompt itself. "Phase we're in," "relevant DECISIONS.md entries," "specific surface or flow," etc.)

## 6. One thing about me that might surprise you

(One specific judgment call or boundary you hold that wouldn't be obvious from the role title alone. The "I will push back on speculative abstraction even when it looks cleaner" type of insight — a real, project-applicable disposition that affects how the orchestrator should think about delegating to you. Not a personality quirk; a working principle.)

Constraints:
- Section 4 must reciprocate. If you claim a handoff to agent X, that handoff must be plausible given X's Section 3 (what X said they don't do). If X's Section 3 contradicts your handoff, name the conflict — don't paper over it.
- Section 5's good and bad prompts must be concrete and project-specific. Generic examples fail.
- Section 6 must be a working principle, not a quirk. "I prefer Vim" is not a working principle. "I refuse to evaluate a library against a feature spec without first running the library against a real input" is.
- Save the file and commit. Return a one-line summary to the orchestrator.
```

---

## Acceptance criteria for a Phase B intro

The orchestrator (or human reviewer) checks each Phase B submission against these:

1. **Sections 4, 5, and 6 are present and non-empty.**
2. **Section 4 names every handoff with both direction and condition.** "To <agent> when <condition>." If a handoff bullet doesn't name a condition, bounce it.
3. **Section 4 cross-references at least one other agent's Phase A.** This forces the agent to actually read the others, not pattern-match from their `initial_handoffs_to` field.
4. **Section 5 has both a good and a bad prompt example, both project-specific.** Generic examples fail.
5. **Section 6 is a working principle, not a personality quirk or generic claim.** Bounce bullets like "I'm thorough" or "I value clean code."
6. **The file is appended to (not overwritten).** Sections 1–3 from Phase A are still present. Each agent's intro is a separate commit.

If any of these fail, route back to the agent with the specific failure cited.

---

## What happens after all Phase B intros are in

Once every agent has produced Sections 4–6:

1. Run the **handoff-mesh audit** (next document: `handoff-mesh-audit.md`).
2. The audit reads every Section 4 across the roster and checks that claimed handoffs reciprocate.
3. The output is a `docs/agents/README.md` with the index and the audit findings.
