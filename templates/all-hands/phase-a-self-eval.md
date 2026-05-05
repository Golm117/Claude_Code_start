# All-hands exercise — Phase A: Self-evaluation

Run this prompt **once per agent** in the project's roster, after BOOTSTRAP has scaffolded all agent files but **before** Phase B.

Each agent runs in isolation for Phase A — they should NOT see other agents' Phase A output yet. That's deliberate. We want each agent's first description of itself to be unprompted by what others said.

---

## Setup

For each agent in `agent_roster`:

1. Spawn the agent (via the Task tool with `subagent_type` matching the agent's name).
2. Pass it the prompt below, with `{{agent_name}}` substituted.
3. The agent appends Sections 1–3 to its own file at `docs/agents/<agent-name>.md` (creating the file if needed; it should not yet exist before Phase A).
4. Commit each Phase A intro independently: `[all-hands] {{agent_name}} Phase A intro`.

Run all Phase A invocations before starting Phase B. Order doesn't matter as long as Phase A is complete for everyone before any agent starts Phase B.

---

## The prompt to pass to each agent

```
You are {{agent_name}}, one of the agents on this project. The project has just been bootstrapped from the universal blank template, and your scaffolded `.claude/agents/{{agent_name}}.md` file currently has placeholders where your self-introduction should be.

Your task right now is **Phase A of the all-hands exercise** — write Sections 1, 2, and 3 of your intro. You do this BEFORE seeing any other agent's intro. The point is to capture your unprompted self-description.

Read these first:
- `.pdr.md` — the project design reference
- Your own `.claude/agents/{{agent_name}}.md` file — your scope, handoffs, and rationale
- `docs/reference-guide.md` — the project narrative

Then write Sections 1–3 in your own voice, project-specifically. Save them to `docs/agents/{{agent_name}}.md` in this exact format:

# {{agent_name}}

## 1. Who I am

(1–3 sentences, in your own voice, describing yourself as a participant in this specific project. Use the project's vocabulary. "I am the X for this project" is fine; pad with role-generic claims and we'll bounce it back.)

## 2. What I do well

(4–6 bullets, each PROJECT-SPECIFIC. Name actual surfaces, files, deliverables, vocabulary from THIS project — not generic claims like "I write good code" or "I design clean schemas." If your bullet could appear unchanged in any other project's intro, rewrite it.)

## 3. What I don't do

(3–4 bullets naming work that explicitly belongs to other agents or to the human. For each: name the specific work, name who owns it instead, and the reason for the boundary. "I don't write production code" is incomplete; "I don't write production code — that's <agent>'s lane; I produce designs and rationale" is the bar.)

Do not write Sections 4–6 yet. Phase B will fill those in after you've seen everyone else's Phase A.

Constraints:
- No padding. Empty bullets and hypothetical war stories are not acceptable.
- Project-specific vocabulary throughout. If a paragraph reads as if you copied it from a generic role description, rewrite.
- Honesty about scope you haven't tested yet. If the project has just bootstrapped, say so — don't fabricate war stories.
- Save the file and commit. Return a one-line summary to the orchestrator.
```

---

## Acceptance criteria for a Phase A intro

The orchestrator (or a human reviewer) checks each Phase A submission against these:

1. **Sections 1, 2, and 3 are present and non-empty.** No placeholders, no "TBD."
2. **Section 1 is in first-person voice.** Not "the {{agent_name}} agent does X" — "I do X."
3. **Section 2 names project-specific vocabulary.** At least three of the bullets reference entities, surfaces, or deliverables from `.pdr.md`. Generic-sounding bullets get bounced.
4. **Section 3 names the receiving party for each "don't" item.** "I don't ship features. <agent> does." not just "I don't ship features."
5. **No fabrication of past work.** A freshly bootstrapped project hasn't accumulated any history; the Phase A intro should reflect that posture, not invent it.
6. **The file is at `docs/agents/<agent-name>.md` and is committed.** Each agent's intro is a separate commit so they're individually reviewable.

If any of these fail, route back to the agent with the specific failure cited.

---

## What happens after all Phase A intros are in

Once every agent has produced Sections 1–3:

1. The orchestrator commits a Phase A complete marker (e.g., a one-line entry in `NOTES.md`).
2. Phase B begins. Run `phase-b-introductions.md` for each agent.
