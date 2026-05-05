<!--
TEMPLATE METADATA — stripped by BOOTSTRAP.

Variables consumed:
  - {{project_name}}
  - {{bootstrap_date}}
  - {{approved_stack}}
  - {{compliance_regime}}
  - {{conceptual_frame}}
  - {{methodology_source}}

The seeded entries at the bottom capture the load-bearing decisions made at bootstrap:
the approved stack, the compliance posture, the methodology grounding. These are the
decisions that, if they were unrecorded, future-us would have to re-litigate.
-->
# Decisions

Append-only log of architectural, product, and tooling decisions. Newest entries at the top. Never edit or delete old entries — if a decision is superseded, write a new entry that references it.

**Format:**

```
## YYYY-MM-DD — Short title
**Decided by:** <human | Claude Code | discussion>
**Status:** accepted | superseded by [link] | reversed
**Context:** why this came up
**Decision:** what we're doing
**Rationale:** why this over alternatives
**Alternatives considered:** what we looked at and rejected
```

Keep entries short. One paragraph per field is plenty. If it takes more, it probably belongs in a design doc under `docs/<domain>/`.

---

## {{bootstrap_date}} — Project bootstrapped; agent roster assembled from work analysis

**Decided by:** human (via grill skill)
**Status:** accepted
**Context:** Project bootstrap. The grill skill walked through `.pdr.md`, produced the phase breakdown and Phase 0 task list, then proposed an agent roster derived from the kinds of work the tasks demand.
**Decision:** The roster captured in `.pdr.md` Section 6 is the assembled crew for this project. Each agent was scaffolded from the universal blank template (`Start_Here/templates/agent-blank.md`) using its grill-produced spec. The all-hands exercise (Phase A self-eval, Phase B introductions, handoff-mesh audit) ran after scaffolding.
**Rationale:** Agent rosters are purpose-built per project, not selected from a default menu. This avoids embedding bias from any "canonical" set and keeps the team sized to the actual work. Each agent's `description` was drafted to name trigger conditions specifically, so Claude Code's delegation routing is reliable.
**Alternatives considered:** A canonical 10-agent default (rejected — every project ends up with the wrong roster, either too many agents idle or too few agents covering work). A grill that proposes a bigger roster than needed (rejected — bias toward overstaffing produces noise; smaller roster with clear handoffs beats larger roster with overlap).

---

## {{bootstrap_date}} — Approved stack: {{approved_stack}}

**Decided by:** human
**Status:** accepted
**Context:** Project bootstrap. The PDR lists the tools committed to from day one.
**Decision:** Use {{approved_stack}}. Anything outside this list requires human approval and at least one alternative being considered. New dependency installs get a `DECISIONS.md` entry with date and one-line rationale.
**Rationale:** A short approved-stack list keeps integration surface small and reviewable. Locking in the stack at bootstrap forces the trade-off conversation to happen once, not once per dependency.
**Alternatives considered:** Open-ended dependency policy (rejected — drift accumulates; one impulsive `npm install` becomes a load-bearing dep nobody re-evaluates).

---

## {{bootstrap_date}} — Compliance posture: {{compliance_regime}}

**Decided by:** human
**Status:** accepted
**Context:** Project bootstrap. PII entities and data-handling obligations were assessed against applicable regimes.
**Decision:** Compliance regimes in scope: {{compliance_regime}}. The relevant agents in the roster (security, compliance-reviewer, or equivalent — depending on the project's roster) review changes against these obligations.
**Rationale:** Naming the compliance posture explicitly at bootstrap means every subsequent design decision is scoped to a known bar, not re-derived per feature.
**Alternatives considered:** Defer compliance until a real audit forces it (rejected — retrofitting is far more expensive than designing for the bar from day one).

---

## {{bootstrap_date}} — Methodology grounding: {{conceptual_frame}}

**Decided by:** human
**Status:** accepted
**Context:** The project is built around a specific conceptual frame. Naming it explicitly at bootstrap gives every agent a shared vocabulary and a way to recognize when a proposed design drifts from the methodology.
**Decision:** {{conceptual_frame}}. Source: {{methodology_source}}. Domain entities, vocabulary, and product priorities trace back to this frame.
**Rationale:** Without a named methodology, agents make implicit modeling choices that diverge across the team. A named frame is a coordination mechanism.
**Alternatives considered:** No explicit methodology grounding (rejected — produces vocabulary drift and inconsistent design choices across agents).
