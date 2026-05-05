---
name: pdr-grill
description: Bootstrap a new project by interviewing the user against the PDR schema, producing a validated `.pdr.md` plus a phase breakdown with tasks per phase, then deriving the project's purpose-built agent roster from the work. Use when starting a new project (greenfield) or when adopting an existing repo into the multi-agent operating model. Auto-detects the repo state (greenfield vs adopt-existing) and only grills sections the user hasn't pre-answered.
---

# pdr-grill

Interview-driven bootstrap. Produces three things, all written to the project root:

1. **`.pdr.md`** — the structured project design reference (frontmatter + prose body)
2. **`TASKS.md` (initial)** — the phase breakdown with concrete tasks per phase, seeded into the standard `TASKS.md` template
3. **An assembled agent roster** in `.pdr.md` Section 6 — purpose-built from the task analysis, no defaults

After this skill completes, the user runs `BOOTSTRAP.md` to scaffold the rest of the project.

---

## When to invoke

- User says "start a new project," "bootstrap this project," or "set up the agent system here"
- An empty repo (greenfield) where the user wants to start fresh
- An existing repo (adopt-existing) where the user wants to layer the multi-agent operating model on top of what's already there
- An existing partial `.pdr.md` that needs the gaps filled

---

## How to run this skill

The skill is a structured interview. Walk the steps in order. Don't skip ahead. Each step has acceptance criteria — only proceed past a step when the criterion is met.

### Step 0 — Audit the repo

This step runs FIRST, before any user interview. Read the working directory and detect:

| Field | How to detect |
|---|---|
| `bootstrap_mode` | `greenfield` if no `package.json`, no `src/`, no `app/`, no `pages/`, no source code at all; `adopt-existing` otherwise |
| `detected_framework` | Read `package.json` deps + check for `next.config.*`, `vite.config.*`, `nuxt.config.*`, `astro.config.*`, `svelte.config.*`, `remix.config.*` |
| `detected_router` | Only if Next.js: `app/` → app router, `pages/` → pages router |
| `detected_language` | `tsconfig.json` presence + dominant file extension across source |
| `detected_styling` | Check for `tailwind.config.*`, `postcss.config.*`, deps like `styled-components`, `emotion`, `mui`, `chakra`, `antd` |
| `detected_ui_library` | `components.json` (shadcn), or specific deps |
| `detected_db_layer` | `supabase/` folder, `prisma/`, `drizzle.config.*`, deps |
| `detected_auth` | Deps: `@clerk/*`, `next-auth`, `@supabase/auth-helpers`, `auth0`, etc. |
| `detected_existing_docs` | Glob: `README.md`, `CONTRIBUTING.md`, `DECISIONS.md`, `docs/**` |
| `detected_existing_agents` | `.claude/agents/` folder populated → bool |
| `monorepo_layout` | `turbo.json`, `pnpm-workspace.yaml`, `nx.json`, `lerna.json`, `rush.json`, root `workspaces` field, root `packages/` or `apps/` |

Surface the audit results to the user in one block: "Here's what I see in this repo. Confirm or correct anything." If the user contradicts the audit (e.g., "we're using Next.js" but `vite.config.ts` is detected), STOP and ask whether the audit is wrong, the user is migrating, or something else.

If `detected_existing_agents = true`, refuse to proceed without explicit confirmation that overwriting is OK. Don't silently destroy a prior bootstrap.

### Step 1 — Check for an existing `.pdr.md`

If `.pdr.md` exists at the project root:

1. Parse the frontmatter. Identify which fields are filled and which are missing/thin.
2. Show the user a summary: "I see a partial PDR. Here's what's filled, here's what's missing."
3. The grill walks ONLY the missing/thin sections in the rest of the steps.

If `.pdr.md` does NOT exist:

1. Create a stub `.pdr.md` with empty fields and `schema_version: 0.4`.
2. Walk every section.

### Step 2 — Interview Section 1 (Product)

Ask, in order, only the fields not already filled. Use the grill prompts in `PDR_SCHEMA.md` Section 1.

Validation per field (re-grill if thin):
- `project_name`: kebab-case, non-empty, no spaces
- `one_line_pitch`: ≤160 chars, has a verb and a target user
- `wedge`: names a specific user, specific outcome, and measurable threshold (time / cost / friction)
- `success_metric`: measurable, not "users love it"

After all four are filled, write them to `.pdr.md` frontmatter and commit: `[grill] Section 1 — product`.

### Step 3 — Interview Section 2 (Audience)

`target_user`, `user_alternatives`, then `voice_register` (only if any user-facing copy surface — defer until after Step 5 surfaces are known if uncertain).

Validation:
- `target_user`: specifies role/context, not just demographic
- `user_alternatives`: at least 2 entries, "nothing" is invalid
- `voice_register`: names register and at least one anti-pattern (only if required)

Commit: `[grill] Section 2 — audience`.

### Step 4 — Interview Section 3 (Methodology)

`conceptual_frame` and `methodology_source`.

Validation:
- `conceptual_frame`: references a named methodology (with source) or explicit first-principles model. "Best practices" fails.
- `methodology_source`: specific (book, paper, URL, "internal — see X"). "Various sources" fails.

Commit: `[grill] Section 3 — methodology`.

### Step 5 — Interview Section 4 (Scope)

`domain_entities`, then derive `pii_entities` (propose; user confirms), then `surfaces`, then `phases`.

Validation:
- `domain_entities`: 3–7 entries; each is a noun with one-line description
- `pii_entities`: explicit even if empty; if empty, confirm "really no PII?"
- `surfaces`: at least 1
- `phases`: open-ended, minimum 1 (Phase 0); soft warn past 7

Once `surfaces` is set, retroactively confirm whether `voice_register` is needed (skip if no user-facing copy surface).

Commit: `[grill] Section 4 — scope`.

### Step 6 — Interview Section 5 (Constraints)

`approved_stack`, `compliance_regime`, `hard_nos`.

Validation:
- `approved_stack`: in `adopt-existing` mode, must include detected stack; flag contradictions
- `compliance_regime`: explicit even if `["none"]`; if `pii_entities` non-empty and this is `["none"]`, push back hard
- `hard_nos`: at least 1 entry

Commit: `[grill] Section 5 — constraints`.

### Step 7 — Run cross-field validation

Apply every rule in PDR_SCHEMA "Validation rules (cross-field)". If any rule fires, surface the issue and re-grill the relevant fields.

Once all rules pass, the PDR is product/scope-complete. Commit: `[grill] PDR validation passed`.

### Step 8 — Produce the phase breakdown with tasks per phase

For each phase in `phases`, produce a task list at the right granularity:

- **Phase 0** — full T-### entries with the complete TASKS.md format: title, status (`open`), priority, role, description, acceptance criteria, dependencies. Phase 0 is what gets claimed first; it deserves full detail.
- **Phase 1+** — one-line summary per planned task. Format: `- (planned T-###) <short title> — <one-line description>`. No full entries; no acceptance criteria yet. These will be expanded into full T-### entries just before each phase becomes active.

Why this split: producing 50 fully-detailed tasks across 5 phases at bootstrap is over-production. Acceptance criteria written before the project starts are usually wrong because the team learns. Phase 0 is the only phase with enough certainty to fully spec.

For Phase 0 task drafting, draw on:
- The phase's `goal` (one-liner)
- The full PDR (especially surfaces, domain entities, approved stack, compliance regime)
- The audit results (existing infra to leverage, gaps to fill)

For each Phase 0 task, draft:
- Short title
- Description (what + why)
- Acceptance criteria (how we know it's done)
- Suggested role (the kind of specialist who'd own it — informs Step 9 roster derivation)
- Priority (P0 / P1 / P2)
- Dependencies on other Phase 0 tasks

Phase 0 should always include scaffolding work (initial repo setup, deploy pipeline, schema, auth — whatever applies). Later phases are outcome-focused.

Render Phase 0 as full T-### entries; Phase 1+ as one-line summaries. Write the result to `TASKS.md` at the project root.

When Phase 0 completes and Phase 1 is about to be claimed, the orchestrator (or whoever is planning Phase 1) expands Phase 1's one-liners into full T-### entries at that time — not now.

Commit: `[grill] Phase breakdown and task list`.

### Step 9 — Derive the agent roster

Now analyze the task list to propose the agent roster. This is the heist-recruitment step.

1. **Cluster tasks by skill demand.** Group tasks by the kind of specialist that would own them. "Schema design" tasks cluster. "UI / conversion" tasks cluster. "Threat modeling" tasks cluster. "Library evaluation" tasks cluster.
2. **For each meaningful cluster, propose an agent.** A cluster of 1 task in a whole project usually doesn't need its own agent — fold it into a related agent's scope. A cluster of 5+ tasks across phases probably warrants a dedicated agent.
3. **Draft the agent_spec for each proposed agent:**
   - `name` — kebab-case, role-named (e.g., `security`, `ui-engineer`, `bot-protocol-engineer`)
   - `description` — the frontmatter description; must name trigger conditions ("Use for X when Y"), not just a job title
   - `model` — Opus for decision-shaping work (architecture, security, marketing strategy, research), Sonnet for implementation (engineering, UI, integrations, QA), Haiku for fast/cheap dispatch
   - `rationale` — which tasks/work made this agent necessary
   - `scope_owns` — 3–6 things, drawn from the actual cluster of tasks
   - `scope_does_not` — 2–4 things explicitly outside their lane (forces boundary clarity)
   - `domain_folder` — `docs/<name-stem>/`, derived
   - `is_mandatory_reviewer` — true if their lane includes "must review changes touching Z"
   - `mandatory_review_triggers` — concrete change types if mandatory reviewer
   - `initial_handoffs_to` — surface the obvious handoffs ("if A produces X and B consumes X, that's a handoff"). Phase B of all-hands will refine.
   - `first_task_deliverable` — if there's a clear first project-specific output the agent should write before any other work (threat model, voice doc, data-model proposal), specify it. Many agents won't have one.
4. **Present the draft roster to the user with rationale.** "Here's the crew I'd assemble for this work, and why each agent is on it." Show in a compact format:

```
Proposed roster:

- <name> (model: <model>) — <one-line description>
  Rationale: <why this agent>
  Scope: <2-3 word summary of owns>
  Mandatory reviewer? <Yes/No>

- <next agent>
  ...
```

5. **User reviews:** add agents, remove agents, rename, adjust scope. Iterate until the user is satisfied.

6. **Write the final roster to `.pdr.md` Section 6** as `agent_roster: list<agent_spec>`.

Commit: `[grill] Agent roster derived (N agents)`.

### Step 10 — Final report

Output a summary to the user:

```
PDR complete: {{project_name}}

Schema version: 0.4
Bootstrap mode: greenfield | adopt-existing
Surfaces: <list>
Phases: <count>
Phase 0 tasks: <count>
Agents in roster: <count>

Files produced:
  .pdr.md         (the validated PDR)
  TASKS.md        (Phase breakdown with concrete tasks)

Next step: run BOOTSTRAP.md to scaffold the project files,
agent definitions, and run the all-hands exercise.
```

The skill is now complete. The user runs BOOTSTRAP.md next.

---

## Important behaviors

- **Resume-friendly.** If the user pauses mid-grill, the partial `.pdr.md` is the resumable state. On the next invocation, Step 1 detects what's filled and only grills the gaps.
- **No assumptions about the roster.** The grill never proposes a "default" or "canonical" agent. Every agent in the proposal must trace to actual tasks in the breakdown.
- **Push back on thin answers.** Validation per field is non-negotiable. A wedge without a measurable threshold gets re-grilled. A `compliance_regime: ["none"]` with non-empty PII gets re-grilled.
- **Audit-first in adopt-existing mode.** Never override a detected stack with a user claim silently — surface the contradiction and resolve.
- **Commit per step.** Every step ends with a commit. The user can stop at any commit and resume cleanly.
