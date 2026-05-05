# BOOTSTRAP

The orchestration prompt that consumes a validated `.pdr.md` and produces a fully scaffolded project.

This file is a prompt to the orchestrator (Claude Code's main session), not an executable script. The orchestrator reads this prompt, walks the steps, and writes files. Each step is a checkable phase — if any step fails validation, the orchestrator stops and surfaces the failure rather than proceeding.

**Prerequisites:**
- `Start_Here/PDR_SCHEMA.md` exists and defines the contract
- `Start_Here/OPERATING_PROTOCOL.md` exists and defines the memory model
- `Start_Here/templates/agent-blank.md` exists
- `Start_Here/templates/docs/*.md` (the seed doc templates) exist
- `Start_Here/templates/all-hands/*.md` exists
- A valid `.pdr.md` exists at the project root, produced by the `pdr-grill` skill
- A grill-produced task breakdown exists (either embedded in the grill output or seeded into a draft `TASKS.md`)

If any prerequisite is missing, BOOTSTRAP stops and tells the user what's missing.

---

## What BOOTSTRAP produces

Running BOOTSTRAP end-to-end produces, in the project root:

```
.pdr.md                            ← already exists; BOOTSTRAP doesn't touch
README.md                          ← from templates/docs/README.md
CONTRIBUTING.md                    ← from templates/docs/CONTRIBUTING.md
DECISIONS.md                       ← from templates/docs/DECISIONS.md
NOTES.md                           ← from templates/docs/NOTES.md
TASKS.md                           ← from templates/docs/TASKS.md, with grill task breakdown seeded
OPERATING_PROTOCOL.md              ← copy of Start_Here/OPERATING_PROTOCOL.md
.claude/agents/<agent>.md          ← one per agent in agent_roster, scaffolded from templates/agent-blank.md
docs/reference-guide.md            ← from templates/docs/reference-guide.md
docs/agents/                       ← empty initially; populated by all-hands exercise
docs/<domain>/README.md            ← one per agent's domain_folder, with a stub README
docs/<domain>/working-notes.md     ← empty placeholder per domain
```

After BOOTSTRAP completes the file scaffold, the all-hands exercise runs to produce `docs/agents/<agent>.md` for every agent (Phase A + Phase B) and `docs/agents/README.md` (the handoff-mesh audit).

---

## Step 0 — Verify prerequisites

Run these checks. Stop and surface failures; do not proceed past failures.

1. Confirm `.pdr.md` exists at the project root.
2. Parse the `.pdr.md` frontmatter (YAML). Verify `schema_version` is supported (current consumer supports 0.4 and any 0.x ≥ 0.4 with additive-only field changes).
3. Verify all required PDR fields are present and non-empty (use `Start_Here/PDR_SCHEMA.md` as the reference). Required fields:
   - `project_name`, `one_line_pitch`, `wedge`, `success_metric`
   - `target_user`, `user_alternatives`
   - `conceptual_frame`
   - `domain_entities`, `pii_entities`, `surfaces`, `phases`
   - `approved_stack`, `compliance_regime`, `hard_nos`
   - `agent_roster` (must have at least 1 entry)
   - `bootstrap_mode` (greenfield | adopt-existing)
4. Verify `tasks_per_phase` was produced by the grill — either embedded in the grill output OR present in a draft `TASKS.md` at the project root. If neither, stop and ask the human to re-run grill.
5. Verify each agent in `agent_roster` has a complete `agent_spec` (name, description, model, rationale, scope_owns, scope_does_not, domain_folder, is_mandatory_reviewer, initial_handoffs_to). Missing fields → stop.
6. Run cross-field validation rules from PDR_SCHEMA Section "Validation rules (cross-field)":
   - PII without compliance
   - Stack contradicting audit
   - Surface without role coverage in roster
   - Phase 0 missing scaffolding language
   - Wedge without measurable threshold

If any check fails, stop and surface ALL failures at once (don't fix-and-retry mid-step).

---

## Step 1 — Compute derived variables

Compute every variable in PDR_SCHEMA "Derived variables" section. Cache them in memory for the rest of BOOTSTRAP.

For each agent in `agent_roster`, also compute:
- `agent.domain_folder` (use the field if present; else default to `docs/<name-stem>/` where name-stem is the agent name with role suffixes stripped)
- `agent.frontmatter_description` (the `description` field, used in the agent file's YAML frontmatter)

If any derivation produces an ambiguous value (e.g., `db_access_control_term` with an unknown DB layer), stop and ask the human to clarify.

---

## Step 2 — Scaffold project root docs

In `bootstrap_mode = greenfield`:

For each file in `Start_Here/templates/docs/`, render the template and write to the project root (or `docs/` for `reference-guide.md`):

- `templates/docs/README.md` → `README.md`
- `templates/docs/CONTRIBUTING.md` → `CONTRIBUTING.md`
- `templates/docs/DECISIONS.md` → `DECISIONS.md`
- `templates/docs/NOTES.md` → `NOTES.md`
- `templates/docs/TASKS.md` → `TASKS.md` (with `tasks_per_phase` substituted)
- `templates/docs/reference-guide.md` → `docs/reference-guide.md`
- Copy `Start_Here/OPERATING_PROTOCOL.md` → `OPERATING_PROTOCOL.md` (no substitution; the protocol is universal)

Substitution uses the templating syntax in PDR_SCHEMA: `{{var}}` for values, `{{#if flag}}...{{/if}}` for conditionals. Strip HTML comment metadata blocks from the output.

In `bootstrap_mode = adopt-existing`:

For each project-root template file:
- If the destination file does NOT exist, write it (same as greenfield).
- If the destination file DOES exist, do NOT overwrite. Instead, write the rendered template to a sibling file `<filename>.bootstrap.md` and surface a list of all such sidecar files at the end. The human reviews and merges manually. Reasoning: adopting existing repos may have non-trivial existing docs; silent overwrites destroy work.

Commit: `[bootstrap] scaffold project root docs`

---

## Step 3 — Scaffold agent files

For each `agent` in `agent_roster`:

1. Render `Start_Here/templates/agent-blank.md` with:
   - All `{{agent.*}}` variables from this agent's spec
   - All project-wide variables from `.pdr.md` frontmatter
   - All derived variables from Step 1
2. Write the result to `.claude/agents/<agent.name>.md`.
3. Strip the HTML comment metadata block.
4. Verify the file is well-formed Markdown with valid YAML frontmatter (`name`, `description`, `model` all present).

If any agent fails to render, stop and surface the failure with the agent name and which variable was missing.

Commit: `[bootstrap] scaffold agent roster (N agents)` where N is the count.

---

## Step 4 — Scaffold domain folders

For each `agent` in `agent_roster`:

1. Compute `domain_folder` (from Step 1).
2. Create `<domain_folder>/` if it doesn't exist.
3. Write a stub `<domain_folder>/README.md`:
   ```
   # <domain_name>

   Domain folder for <agent_name>. Topic files live here as durable findings accumulate. In-flight thinking goes to `working-notes.md` in this folder.

   Owner agent: `.claude/agents/<agent_name>.md`
   ```
4. Write an empty `<domain_folder>/working-notes.md` with just a one-line header.

If multiple agents share a domain folder (rare but allowed), the README lists all owners.

Also create:
- `docs/agents/` (empty; populated by the all-hands exercise)

Commit: `[bootstrap] scaffold domain folders`

---

## Step 5 — Run the all-hands exercise

The all-hands exercise consists of three phases. Run them in order, with the prompts in `Start_Here/templates/all-hands/`.

### 5a — Phase A self-evaluation

For each `agent` in `agent_roster`:

1. Read `Start_Here/templates/all-hands/phase-a-self-eval.md`.
2. Spawn the agent (Task tool with `subagent_type: <agent.name>`) and pass it the Phase A prompt with `{{agent_name}}` substituted.
3. The agent writes Sections 1–3 to `docs/agents/<agent_name>.md`.
4. Verify against acceptance criteria in the Phase A spec. If any criterion fails, route back to the agent with the specific failure.
5. Commit per agent: `[all-hands] <agent_name> Phase A intro`.

Run all Phase A invocations before any Phase B starts.

### 5b — Phase B introductions

Once every agent has completed Phase A:

For each `agent` in `agent_roster`:

1. Read `Start_Here/templates/all-hands/phase-b-introductions.md`.
2. Spawn the agent and pass it the Phase B prompt with `{{agent_name}}` substituted.
3. The agent reads every other agent's Phase A and writes Sections 4–6 to its own intro file (appending, not overwriting).
4. Verify against Phase B acceptance criteria. Route back on failure.
5. Commit per agent: `[all-hands] <agent_name> Phase B intro`.

### 5c — Handoff-mesh audit

Once every agent has completed Phase B:

1. Read `Start_Here/templates/all-hands/handoff-mesh-audit.md`.
2. Run the audit prompt (orchestrator runs it directly, OR delegates to a fresh subagent with all Phase B intros passed in).
3. The audit produces `docs/agents/README.md` with the index and findings.
4. Review PROACTIVE-flagged mismatches with the human. DEFERRED items don't block.
5. Commit: `[all-hands] handoff-mesh audit complete`.

---

## Step 6 — Verify and report

After all-hands completes:

1. Confirm every agent in `agent_roster` has both `.claude/agents/<name>.md` (scaffolded) and `docs/agents/<name>.md` (intro).
2. Confirm `docs/agents/README.md` exists with the audit findings.
3. Confirm `TASKS.md` has at least one Phase 0 `open` task ready to be claimed.
4. Update `NOTES.md` to record bootstrap completion: date, agents created, audit summary.
5. Commit: `[bootstrap] complete — N agents, M tasks, K mismatches flagged`.

Then report to the human:

```
Project bootstrapped: {{project_name}}

Agents in roster: <list with one-line role>
Phase 0 tasks ready: <count>
Handoff-mesh audit findings: <count of PROACTIVE>, <count of DEFERRED>

Next step: review docs/agents/README.md and address any PROACTIVE findings before
starting on Phase 0. The first open Phase 0 task is T-<id>.
```

---

## Failure modes and recovery

**A required PDR field is missing or thin.**
Stop. Surface the field name. Ask the human to re-run `pdr-grill` to fill the gap. Don't try to fill it yourself.

**A template variable is referenced but undefined.**
Stop. Surface the variable name and the template file. This is a schema-template mismatch — fix in PDR_SCHEMA or the template, then re-run.

**An agent fails Phase A or Phase B acceptance criteria after multiple retries.**
Stop. Escalate to the human with the agent name and the specific failure. Possible causes: agent's scaffolded scope is incoherent (revisit grill's roster decision), the agent prompt isn't matching the agent's actual capabilities, or the project is too small to need this many agents.

**The handoff-mesh audit finds CRITICAL mismatches** (e.g., agent A claims handoff to B, B's Section 3 forbids that work entirely).
Stop the bootstrap-complete commit. Surface the mismatch to the human. The roster needs adjustment in `.pdr.md` and re-running of the affected agents' Phase A/B.

**`bootstrap_mode = adopt-existing` and many sidecar `.bootstrap.md` files were produced.**
Don't auto-merge. List them all in the final report and ask the human to review-and-merge before proceeding to all-hands.
