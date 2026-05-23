# Start_Here

A deployable folder for bootstrapping a new project with a multi-agent operating model.

Drop this folder into an empty repo (or layer it on top of an existing one), run the entry point, and end up with: a validated project design reference, a phase breakdown with concrete tasks, a purpose-built agent roster derived from the work, and ten-plus markdown files coordinating how those agents work together.

## Philosophy

Three principles drive every design choice in this folder:

1. **No canonical roster.** Every project assembles its own crew, hired for the specific work — like a heist crew, not a default team. There is no "10 agents you always need." The grill skill analyses the task breakdown and proposes agents purpose-built for *this* project.
2. **Memory is bound to work, not to workers.** Per-domain working memory (`docs/<domain>/`), not per-agent state files. The same domain knowledge is read by whichever agent's task touches it.
3. **Compression at boundaries, not at runtime.** Agents check in durable findings before returning. Sessions checkpoint at start and end. Context capacity is a planned-for boundary, not a runtime alarm.

If those three feel right, this folder is for you.

## What's in this folder

```
Start_Here/
├── README.md                          ← this file
├── PDR_SCHEMA.md                      ← contract between pdr-grill and BOOTSTRAP
├── OPERATING_PROTOCOL.md              ← memory model + checkpoint discipline
├── BOOTSTRAP.md                       ← orchestration prompt that consumes a valid .pdr.md
├── skills/
│   ├── pdr-grill/
│   │   └── SKILL.md                   ← the entry-point skill that interviews the user
│   └── cc-chain/
│       └── SKILL.md                   ← async Claude Code chain driver (Pattern B+); deployed to ~/.claude/skills/ by BOOTSTRAP if absent
└── templates/
    ├── agent-blank.md                 ← the SOLE template every agent scaffolds from
    ├── docs/
    │   ├── README.md                  ← project-root README seed
    │   ├── CONTRIBUTING.md            ← coordination protocol seed
    │   ├── DECISIONS.md               ← ADR log seed
    │   ├── NOTES.md                   ← session whiteboard seed
    │   ├── TASKS.md                   ← shared task list seed
    │   └── reference-guide.md         ← expanded project narrative seed
    └── all-hands/
        ├── phase-a-self-eval.md       ← agents self-introduce (sections 1-3)
        ├── phase-b-introductions.md   ← agents read each other and refine (sections 4-6)
        └── handoff-mesh-audit.md      ← cross-check claimed handoffs across the roster
```

## How to use it

### Quick start (greenfield)

```bash
# 1. Initialize an empty repo or change into an existing one
git init my-new-project
cd my-new-project

# 2. Drop Start_Here/ into the repo root
cp -r /path/to/Start_Here .

# 3. Open Claude Code in this directory
claude

# 4. Inside Claude Code, invoke the grill skill
/pdr-grill
```

The grill will:
1. Audit the repo (greenfield in this case)
2. Interview you through the PDR (sections 1–5)
3. Produce a phase breakdown with concrete tasks
4. Propose an agent roster derived from the tasks
5. Write `.pdr.md` and `TASKS.md` to the project root

### Quick start (adopt-existing)

If you have an existing project and want to layer this operating model on top:

```bash
cd existing-project
cp -r /path/to/Start_Here .
claude
```

Then inside Claude Code:

```
/pdr-grill
```

The grill will detect the existing repo (framework, language, DB, auth, monorepo layout) and only ask product/scope questions — it won't re-derive what's already in the codebase. Existing docs are preserved; conflicts are surfaced as `.bootstrap.md` sidecar files for you to review.

### After the grill — run BOOTSTRAP

Once `.pdr.md` is validated and `TASKS.md` is seeded:

```
/run BOOTSTRAP.md
```

(Or paste BOOTSTRAP.md's contents into the orchestrator session — the file is a prompt, not an executable.)

BOOTSTRAP will:
1. Verify all prerequisites
2. Deploy any bundled skills (e.g. `cc-chain`) to `~/.claude/skills/` if not already installed (idempotent — never overwrites)
3. Compute derived variables
4. Scaffold project root docs
5. Scaffold agent files (one per agent in the roster) from `agent-blank.md`
6. Scaffold per-agent domain folders
7. Run the all-hands exercise (Phase A, Phase B, handoff-mesh audit)
8. Report completion

After BOOTSTRAP, the project is ready to claim its first Phase 0 task.

## How the system works conceptually

The flow, end to end:

```
User has rough idea
       │
       ▼
[ pdr-grill ]   ← interviews user, produces .pdr.md + TASKS.md + agent_roster
       │
       ▼
[ BOOTSTRAP ]   ← scaffolds project files + agent definitions
       │
       ▼
[ All-hands ]   ← agents self-introduce, meet each other, refine handoffs
       │
       ▼
[ Phase 0 work begins ]
```

Each layer has a single responsibility. The grill produces structured intent; BOOTSTRAP renders templates from intent; the all-hands gives every agent its own voice and tested handoff mesh; then real work begins.

## Maintenance

This folder evolves. When you learn something while running it on a real project — a missing PDR field, a templating gap, an audit rule that produces too many false positives — fix it here and bump the schema version. The version table at the bottom of `PDR_SCHEMA.md` tracks per-field changes; the changelog at the top tracks whole-document evolution.

If a project diverges enough that this folder no longer fits, fork the folder for that project rather than warping the canonical one. The point is to keep `Start_Here/` opinionated and clean.

## What you don't get

Naming this explicitly so it doesn't sneak back in:

- **No agent-pattern library.** There is no "common security agent" or "default UI specialist" to draw from. Every agent is built fresh from `agent-blank.md` per project. A library would bias roster proposals; we don't want that.
- **No canonical 10 roster.** The grill never proposes a default set. Every agent in every roster traces to actual tasks in the breakdown.
- **No per-agent state files.** Memory is per-domain (`docs/<domain>/`), not per-agent. See `OPERATING_PROTOCOL.md`.
- **No automatic runtime context compression.** Compression happens at task and session boundaries, not at token thresholds during work.
- **No nested subagent spawning.** Subagents are leaves; the orchestrator is the only spawner. Tree depth is one.

These are intentional non-features. If a future change wants to add one of them, the design rationale that excluded them lives in this folder's docs — read those before adding.

## Updates

### 2026-05-22 — `cc-chain` skill bundled and made project-aware

Added the async Claude Code chain driver (Pattern B+) to Start_Here so it travels with the scaffold for anyone cloning the repo.

**What `cc-chain` does.** It drives async chains where the main session plans slices and dispatches each one to a remote Claude Code session via `RemoteTrigger`, monitoring the commit via `ScheduleWakeup` and verifying the diff before chaining the next slice. A configurable confidence gate (default 95%, override with `cc-chain threshold:N`) decides whether to auto-fire or pause for the user. End-of-chain behavior is silent by default; pass the `branch-explore` flag if you want it to enumerate plausible next directions.

**New in this update — project-aware prompt drafting.** When `cc-chain` detects canonical Claude Code artifacts in the project it's running in, it surfaces them to the remote CC session:

- `Glob` over `.claude/agents/*.md` — enumerates any subagents the project has scaffolded (Start_Here writes these during BOOTSTRAP Step 3).
- Reads `CONTEXT.md`, `AGENTS.md`, or `ROLES.md` at the repo root if present, for richer role descriptions.
- Includes a "Subagents available in this project" section in the fire prompt, with invocation hints tied to the slice's actual shape ("for the research step, invoke `market-research`"; "before commit, invoke `code-review` on the diff").
- Falls back silently if no agents and no role docs are found — no error, no warning, no degraded behavior.

The skill stays general-purpose; it does NOT bake Start_Here-specific paths in. Start_Here happens to scaffold to the canonical Claude Code locations the skill reads, so they line up naturally — but a project using different conventions just sees the skill omit the roster section gracefully.

**Deployment to user-level skills.** `cc-chain` lives in two places:

- `Start_Here/skills/cc-chain/SKILL.md` — the canonical bundled copy in git, travels with the scaffold.
- `~/.claude/skills/cc-chain/SKILL.md` — the user-level install location where Claude Code's harness auto-loads skills globally.

BOOTSTRAP now has a **Step 0.5** that copies any skill in `Start_Here/skills/<name>/` to `~/.claude/skills/<name>/` only if the destination is absent. Idempotent — never overwrites a diverged user copy, and re-running BOOTSTRAP after first install does nothing.

**Hard prohibitions preserved.** Despite the new prompt-drafting behavior, the workflow's non-negotiables are intact: the `Agent` tool is research-only (no Agent-tool spawning for edits), pushes never go directly to `main`, the previous CC commit is always verified before chaining, and the chain never auto-decides what to work on next unless `branch-explore` is passed.

**Why this matters for cloners.** Anyone who clones Start_Here, runs `/pdr-grill`, and then `/run BOOTSTRAP.md` gets `cc-chain` deployed at user level the first time through. They can then drive async chains on the project BOOTSTRAP just scaffolded — including the project-aware prompt drafting, since BOOTSTRAP has already written agents to `.claude/agents/`. Users who already have their own `cc-chain` are unaffected; their version is preserved.

# Claude_Code_start
