<!--
TEMPLATE METADATA — stripped by BOOTSTRAP.

Variables consumed:
  - {{project_name}}
  - {{one_line_pitch}}
  - {{wedge}}
  - {{approved_stack}}
  - {{current_phase}}

This is the project's front-door README. It points to everything else and gives
a new collaborator (human or AI) enough context to start reading in the right order.
-->
# {{project_name}}

{{one_line_pitch}}

> **The wedge:** {{wedge}}

## Project documentation

Start here before writing any code:

- **[`.pdr.md`](./.pdr.md)** — the project design reference. Source of truth for *what* we're building and *why*.
- **[`docs/reference-guide.md`](docs/reference-guide.md)** — expanded product thesis, methodology, and phased roadmap.
- **[`DECISIONS.md`](DECISIONS.md)** — append-only log of architectural and product decisions. Read this before proposing anything structural.
- **[`CONTRIBUTING.md`](CONTRIBUTING.md)** — how humans, Claude Code, and any web-chat Claude collaborator coordinate on this repo.
- **[`OPERATING_PROTOCOL.md`](OPERATING_PROTOCOL.md)** — memory model, checkpoint discipline, session protocol.
- **[`NOTES.md`](NOTES.md)** — informal session-to-session handoff notes. Scratchpad, not spec.
- **[`TASKS.md`](TASKS.md)** — shared structured task list with atomic git-claim protocol.
- **[`docs/agents/README.md`](docs/agents/README.md)** — the project-specific agent roster with self-written intros and the handoff-mesh audit. Read this before delegating work to a subagent.

## Stack

Approved tools for this project: {{approved_stack}}

Anything beyond this requires human approval with at least one alternative considered. See `CONTRIBUTING.md` for the dependency policy.

## Status

**Current phase:** {{current_phase}}.

See `docs/reference-guide.md` for the full phased roadmap.

## Getting started (local dev)

_To be filled in once Phase 0 scaffolding is done._

```bash
# placeholder
```

## License

_TBD._
