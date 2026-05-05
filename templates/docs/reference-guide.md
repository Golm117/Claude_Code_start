<!--
TEMPLATE METADATA — stripped by BOOTSTRAP.

Variables consumed:
  - {{project_name}}
  - {{one_line_pitch}}
  - {{wedge}}
  - {{target_user}}
  - {{user_alternatives}}
  - {{success_metric}}
  - {{conceptual_frame}}
  - {{methodology_source}}
  - {{voice_register}}
  - {{domain_entities}}
  - {{pii_entities}}
  - {{surfaces}}
  - {{phases}}
  - {{approved_stack}}
  - {{compliance_regime}}
  - {{hard_nos}}

This is the expanded prose form of `.pdr.md` — the same content rendered as a
narrative reference document. Agents read this to understand the project; humans
read this to onboard. The PDR is the structured form (machine-friendly); this is
the human-friendly expansion.
-->
# {{project_name}} — Reference guide

This document is the expanded narrative version of `.pdr.md`. The PDR is the structured contract; this is the human-readable explanation of *what we're building*, *why*, *for whom*, and *in what order*.

If something in this document conflicts with `.pdr.md`, the PDR wins (it's the authoritative source for machine consumers like the agent roster). Update both together when changing direction.

---

## Part 1 — The product

### One-line pitch

{{one_line_pitch}}

### The wedge

{{wedge}}

### Why this matters

The narrowest, sharpest version of the value claim above is what every Phase 0 design decision should be tested against. If a design choice doesn't move the wedge, it's probably premature.

### Success metric

{{success_metric}}

This is the primary signal we'll use to decide whether the product is working — not the only metric we'll watch, but the one that decides "is this thing earning its existence."

---

## Part 2 — The audience

### Target user

{{target_user}}

### What they do today (alternatives)

{{user_alternatives}}

Each of these is a real competitor for the user's attention and budget. Designs and copy that ignore them produce work that reads as "another tool" rather than "the better answer."

### Voice and register

{{voice_register}}

Every user-facing surface — copy, error messages, email, documentation — speaks in this register. When in doubt, read it aloud. If it sounds wrong, rewrite.

---

## Part 3 — The methodology

### Conceptual frame

{{conceptual_frame}}

**Source:** {{methodology_source}}

### How the methodology shapes the product

The frame is not decorative. It dictates:
- The domain vocabulary (see Part 4)
- The shape of the data model
- The order of phases (see Part 6)
- What "good" looks like for any given surface

When a design proposal departs from the methodology without naming why, push back. Methodology deviations are decisions in their own right and belong in `DECISIONS.md` with rationale.

---

## Part 4 — Scope

### Domain entities

The core nouns of this product:

{{domain_entities}}

These are the things the system creates, reads, updates, and reasons about. Schema, API contracts, and UI components are organized around these entities.

### PII entities

{{pii_entities}}

These entities carry personally identifiable information. They get stricter handling: lockdown, redaction in logs, minimum-necessary exposure, secure transport. The reviewer agent in the roster (security, privacy-engineer, or equivalent) gates changes touching these.

### Surfaces

What the user actually sees and interacts with: {{surfaces}}

Each surface has its own conversion or value bar. Phase work is organized around shipping surface increments.

---

## Part 5 — Constraints

### Approved stack

{{approved_stack}}

Anything outside this list requires human approval and at least one alternative being considered. The dependency policy is in `CONTRIBUTING.md`.

### Compliance regime

{{compliance_regime}}

The reviewer agent(s) in the roster review changes against these obligations. Compliance gates are non-negotiable.

### Hard nos

Things this product will explicitly never do:

{{hard_nos}}

Every agent treats these as boundaries. If a design proposal pushes against a hard no, it's a human-level decision to revisit, not an agent-level call.

---

## Part 6 — Phased roadmap

{{#each phases}}
### {{this.id}}: {{this.name}}

**Goal:** {{this.goal}}

{{/each}}

Tasks for the current phase live in `TASKS.md`. The structure of `TASKS.md` is the source of truth for *what's being worked on right now*; this section is the source of truth for *what each phase is for*.

---

## Part 7 — Operating model

This project uses a specific operating model for agent collaboration:

- **Agent roster** — purpose-built per project from the work, not selected from defaults. See `docs/agents/README.md` for the self-introductions and handoff-mesh audit.
- **Memory model** — four layers (identity, domain working memory, shared coordination, session checkpoint). See `OPERATING_PROTOCOL.md`.
- **Checkpoint discipline** — agents write durable findings to their domain folder before returning. The orchestrator checkpoints at session boundaries. See `OPERATING_PROTOCOL.md` for the full protocol.
- **Coordination** — `TASKS.md` is the shared task list with atomic git-claim. `NOTES.md` is the session-to-session whiteboard. `DECISIONS.md` is the append-only ADR log.

Read `CONTRIBUTING.md` for the practical day-to-day collaboration protocol.
