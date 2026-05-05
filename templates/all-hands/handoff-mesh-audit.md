# All-hands exercise — Handoff-mesh audit

Run this prompt **once**, after every agent has completed Phase B.

The audit cross-checks every claimed handoff in the roster. If agent A says "I hand off to B under condition X," agent B's intro should describe accepting work from A under condition X. Mismatches surface friction points before they bite during real work.

The output is a `docs/agents/README.md` that serves as both the index of agents and the audit report.

---

## Who runs the audit

The orchestrator (Claude Code's main session) runs the audit, not a subagent. Reasoning: the audit needs the full roster's Section 4 in context simultaneously, and the orchestrator is the one place where that's already available without spawning a fresh agent.

If the orchestrator's context is tight, the audit can be delegated to a fresh subagent with the explicit prompt below — but pass it the full set of Phase B intro files as input, not just file paths.

---

## The audit prompt

```
You are running the handoff-mesh audit on the agent roster for {{project_name}}. Every agent in the roster has completed Phase A and Phase B of the all-hands exercise. Your job is to:

1. Read every agent's full intro at `docs/agents/<agent-name>.md`.
2. Build a map: for each agent, the list of handoffs they CLAIM (their Section 4) and the list of inbound work they ACCEPT (implicit from their Section 3 + Section 4 wording).
3. Cross-check: for every claimed handoff "agent A → agent B under condition X," verify that agent B's intro describes accepting that kind of work under that condition. **Skip handoffs to reserved targets** (`human`, `orchestrator`) — these don't have intros to reciprocate against and are trivially valid.
4. Categorize findings:
   - **Tight mesh:** A claims handoff to B; B confirms inbound from A on the same condition. No action needed.
   - **Mismatch — claimed but not reciprocated:** A claims handoff to B; B's intro doesn't mention accepting work from A. May be benign (B will accept fine) or a real friction point. Flag with severity.
   - **Mismatch — implicit but not claimed:** B's intro says "I accept work from A," but A's intro doesn't list a handoff to B. Less common; usually means A's view is incomplete.
   - **Vocabulary mismatch:** A and B describe the same handoff but use different terms (e.g., A says "shippable copy block," B says "copy artifact"). Friction in handoff hand-off; recommend pinning vocabulary.
   - **Severity vocabulary mismatch:** if any agents flag findings using different severity scales (BLOCK/PUSH/NOTE vs CRITICAL/HIGH/MEDIUM/LOW), surface and recommend reconciliation.
5. Write the index + audit to `docs/agents/README.md` using the format below.
6. Decide for each mismatch whether it's PROACTIVE (worth resolving now by editing intros or DECISIONS.md) or DEFERRED (will surface naturally the first time the handoff fires; not worth pre-litigating). Recommend, don't enforce — the human makes the call.

Format for `docs/agents/README.md`:

# Agent roster — intros and handoff-mesh audit

Each agent in this project wrote their own intro in two phases: Sections 1–3 in Phase A (who they are, what they do well, what they don't do), Sections 4–6 in Phase B (handoffs, prompt templates, surprising things). Full intros are linked below; this README is the index plus the handoff-mesh audit.

The `.claude/agents/*.md` files are the short system prompts that define each agent's voice and scope. These `docs/agents/*.md` files are the agent-written expansions.

## Index

| Agent | Intro | One-line role |
|---|---|---|
| <agent name> | [<file>](./...md) | <one-line summary derived from their Section 1> |
| ...

## Handoff-mesh audit

Cross-checked every claimed handoff across the roster. Findings:

### Tight mesh

- <handoff> — A → B under condition X. B confirms inbound. No action.
- ...

### Mismatches flagged

1. **<short title>** — A claims handoff to B; B doesn't reciprocate. Severity: <PROACTIVE | DEFERRED>. <one-paragraph description and recommendation.>
2. ...

### Specialization quality

(Walk through each agent's "what I do well" section. Do they pass the specificity bar — project-specific vocabulary, no generic bullets? Note any agent whose intro feels thin or generic; recommend a re-do for that agent.)

## How to use this index

- **Before delegating work**, skim the relevant intro's Section 5 ("How to ask me for work well"). The good-prompt examples are templates.
- **When a handoff feels wrong**, check both agents' Section 4. If they describe the handoff differently, the discrepancy is the source of friction — sharpen in `.claude/agents/` or in the task note, not ad-hoc.
- **When adding a new agent** (or redefining one), re-run Phase A + Phase B + this audit for the new agent and any closely paired neighbor. Intros decay as the project evolves.

Save the result to `docs/agents/README.md` and commit. Return a summary to the orchestrator listing total mismatches and how many you flagged PROACTIVE.
```

---

## What proactive vs deferred looks like

**Proactive:** edit the relevant agent intros now to align them. Examples:
- Two agents use different severity scales when reviewing the same diff. Pick one, update both intros.
- Two agents claim ownership of the same artifact (e.g., both think they own `docs/voice.md`). Resolve via a `DECISIONS.md` entry that names the owner.
- An agent claims a handoff that the receiving agent's Section 3 explicitly forbids. Real conflict; resolve before any task fires.

**Deferred:** leave it; it'll surface the first time the handoff actually runs. Examples:
- A unidirectional reference (A names B, B doesn't name A) where the work is unlikely to be ambiguous.
- Vocabulary differences that are likely to converge on first contact.
- A handoff condition that's slightly fuzzy but only fires on edge cases.

When in doubt, defer — over-pre-litigating intros is a real cost; let real work surface real friction.

---

## What happens after the audit

1. The audit findings are committed to `docs/agents/README.md`.
2. Any PROACTIVE fixes are made by editing the relevant agent intros and committing.
3. The orchestrator notes "all-hands complete" in `NOTES.md` with a one-line summary of audit outcomes.
4. The project is ready to start Phase 0 work. The first task in `TASKS.md` can be claimed.
