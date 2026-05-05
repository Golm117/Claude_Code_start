# PDR_SCHEMA

The contract between the `pdr-grill` skill and `BOOTSTRAP.md`. Both consume this file.

- `pdr-grill` reads it to know **what to ask for, in what order, and when an answer is "thin enough" to push back on**.
- `BOOTSTRAP.md` reads it to know **what fields to expect in the produced `.pdr.md`, what each field maps to in agent prompts and seed docs, and which fields are required for a valid bootstrap input**.

If the schema changes, both consumers must be updated. The schema version is the contract version.

**Schema version:** 0.4.1 (draft)

**Changelog:**
- `0.4.1` — Two small refinements driven by the first end-to-end test run (discord-mod-bot). (1) Reserved special handoff targets `human` and `orchestrator` are now explicitly allowed in `initial_handoffs_to[].agent_name`; the handoff-mesh audit skips reciprocation checks for them. (2) Phase 0 vs Phase 1+ task granularity formalized in the grill skill: Phase 0 gets full T-### entries with acceptance criteria; Phase 1+ gets one-line summaries that expand to full entries when the phase becomes active. This prevents the grill from over-producing speculative acceptance criteria that will be wrong before the project starts.
- `0.4` — **Major correction.** Removed the "canonical 10 agents with overrides" framing entirely. There is no canonical roster. The agent roster is **derived from the work** by the grill, not selected from defaults — every project gets a purpose-built crew, like hiring for the specific job. Section 6 rewritten: single `agent_roster` field, populated by grill from the project's task breakdown and scope. Dropped `canonical_roster_overrides` and `define_new_agents` (the distinction no longer exists — every agent is "new" relative to the project). Updated dependency DAG to show grill's flow: PDR → phase breakdown (lives in `TASKS.md`, not `.pdr.md`) → roster derivation. Implication for templates: there is no agent-pattern library; every agent scaffolds from a single blank template (`templates/agent-blank.md`) using the per-agent spec the grill produced.
- `0.3` — Added "Templating syntax" section (defines `{{var}}` substitution and `{{#if flag}}...{{/if}}` conditionals for templates). Added "Derived variables" section listing values BOOTSTRAP computes from PDR fields (not grilled): `has_email_surface`, `has_public_surface`, `public_surfaces`, `has_db_layer`, `db_access_control_term`, `has_integrations`, `external_integrations`, `has_auth`, `is_monorepo`, `current_phase`, `domain_folder_<agent>`. Driven by gaps surfaced building the `security` agent template prototype.
- `0.2` — Resolved 5 open questions from `0.1`. Phases now open-ended (no cap). Agent roster overrides restructured to encourage defining net-new project-specific agents. Added `monorepo_layout` to audit (Section 0). Versioning is now both whole-document AND per-field (see "Field versions" table at the bottom). `voice_register` remains conditional (not required for non-user-facing surfaces).
- `0.1` — Initial draft.

---

## How to read this document

Each field block looks like this:

```
### field_name
- **Required:** yes | no | conditional
- **Type:** string | enum[...] | list<...> | table | bool
- **Depends on:** other field names (must be answered first)
- **Used by:** which templates / agents / seed docs consume it
- **Validation:** what makes the answer thin enough to re-grill
- **Grill prompt:** the question pdr-grill asks
- **Example:** what a good answer looks like
```

`pdr-grill` walks fields in dependency order. `BOOTSTRAP.md` reads the final `.pdr.md` and substitutes `{{field_name}}` into templates.

---

## Templating syntax

Agent templates and seed docs use a minimal Handlebars-lite syntax that BOOTSTRAP processes. Keep it small on purpose — anything that wants more expressive power belongs as a derived variable, not as template logic.

**Variable substitution: `{{variable_name}}`**
Replaced with the field's value from `.pdr.md` (or its derived equivalent — see "Derived variables" below). Lists render as comma-separated by default, or as bullet lists if the surrounding context is a markdown bullet block (BOOTSTRAP detects from leading `- ` on the line). Empty/null values render as empty strings — templates should defensively gate on `{{#if}}` before referencing values that might be empty.

**Conditional blocks: `{{#if flag}} ... {{/if}}`**
Renders the block only if `flag` is truthy. Used for stack-specific or surface-specific sections. The flag can be any field name OR any derived variable. False / null / empty-list flags hide the block. Inline form is supported on a single line: `{{#if flag}}text{{/if}}`.

**Conditional inverse: `{{#unless flag}} ... {{/unless}}`**
Renders only if `flag` is falsy. Use sparingly — explicit positive conditions are clearer.

**No loops, no nested conditionals, no expressions.** If a template needs them, the schema is wrong — the value should be pre-computed as a derived variable.

**Comment blocks: `<!-- ... -->`**
HTML comment blocks at the top of a template file are template metadata for the maintainer (what variables this template consumes, conditional branches, etc.). BOOTSTRAP strips them from the output. Standardize this for every template.

**Whitespace normalization:** BOOTSTRAP collapses runs of empty lines created by elided conditional blocks, so an `{{#if false}} ... {{/if}}` doesn't leave a visible gap.

**No template-language escapes:** if a literal `{{` is needed in the output, escape with `\{{`. Rare in practice.

---

## Dependency DAG (resolution order)

```
[ Section 0: Repo audit ]   ← runs first, no dependencies, auto-populated
        │
        ▼
[ Section 1: Product ]
   project_name, one_line_pitch, wedge, success_metric
        │
        ▼
[ Section 2: Audience ]
   target_user, user_alternatives, voice_register
        │
        ▼
[ Section 3: Methodology ]
   conceptual_frame, methodology_source
        │
        ▼
[ Section 4: Scope ]
   domain_entities  ─→  pii_entities  (derived)
   surfaces
   phases  (one-line goal per phase)
        │
        ▼
[ Section 5: Constraints ]
   approved_stack          (gated by Section 0 audit if existing repo)
   compliance_regime       (informed by pii_entities)
   hard_nos
        │
        ▼
[ Grill produces task breakdown ]
   For each phase, a list of concrete tasks. Lives in TASKS.md
   (not .pdr.md) — the PDR captures product/scope; TASKS.md captures work.
   Used as input to roster derivation.
        │
        ▼
[ Section 6: Agent roster ]
   agent_roster — derived by grill from the work. No defaults, no presumed canonical
   roles. Each agent is purpose-built for THIS project's specific tasks.
```

Fields in Section 0 are populated by the audit. Sections 1–5 are grilled. The task breakdown is grilled and seeded into `TASKS.md`. Section 6 is grilled by analysing the task breakdown — "what specialists do these tasks need?"

---

## Section 0 — Repo state (audit-populated)

Auto-populated by the audit step. User confirms in one pass; `pdr-grill` does not interview these.

### bootstrap_mode
- **Required:** yes
- **Type:** enum [greenfield, adopt-existing]
- **Used by:** BOOTSTRAP.md (decides whether to scaffold or only layer agents on top)
- **Validation:** must be set; never null
- **Set by:** audit — `greenfield` if repo is empty (no `package.json`, no source dirs); `adopt-existing` otherwise
- **Grill prompt:** none — surfaced as "I detected `{value}`. Confirm? (y/n)"

### detected_framework
- **Required:** yes
- **Type:** enum [next.js, vite, remix, astro, sveltekit, nuxt, none, other]
- **Used by:** every agent template that mentions framework-specific surfaces (software-engineer, ui-specialist, integration-engineer)
- **Set by:** audit — reads `package.json` deps + presence of `next.config.*` / `vite.config.*` / etc.
- **Grill prompt:** if `other` or `none` and `bootstrap_mode = adopt-existing`, ask: "What framework? (free text)"

### detected_router
- **Required:** conditional (only if `detected_framework = next.js`)
- **Type:** enum [app, pages, n/a]
- **Used by:** software-engineer template (server actions vs API routes), ui-specialist (layout files)
- **Set by:** audit — `app/` vs `pages/` directory presence

### detected_language
- **Required:** yes
- **Type:** enum [typescript, javascript, python, go, mixed]
- **Used by:** all engineer agents (TS-specific guidance vs JS)
- **Set by:** audit — `tsconfig.json` presence + dominant file extension

### detected_styling
- **Required:** no
- **Type:** list<enum> [tailwind, css-modules, styled-components, emotion, vanilla-css, mui, chakra, none]
- **Used by:** ui-specialist template
- **Set by:** audit — config files + deps

### detected_ui_library
- **Required:** no
- **Type:** enum [shadcn, mui, chakra, antd, radix-only, headless-ui, custom, none]
- **Used by:** ui-specialist template
- **Set by:** audit — `components.json` (shadcn), deps

### detected_db_layer
- **Required:** no
- **Type:** enum [supabase, prisma, drizzle, kysely, raw-pg, mongoose, none]
- **Used by:** database-engineer template (skip agent if `none` and `bootstrap_mode = greenfield` and stack omits a DB)
- **Set by:** audit — `supabase/`, `prisma/`, `drizzle.config.*`, deps

### detected_auth
- **Required:** no
- **Type:** enum [supabase-auth, next-auth, clerk, auth0, custom, none]
- **Used by:** security template (auth flow review depth)
- **Set by:** audit — deps

### detected_existing_docs
- **Required:** no
- **Type:** list<string> (file paths)
- **Used by:** BOOTSTRAP.md (decides whether to merge or replace docs)
- **Set by:** audit — scans for `README.md`, `CONTRIBUTING.md`, `docs/`, `DECISIONS.md`, etc.

### detected_existing_agents
- **Required:** no
- **Type:** bool
- **Used by:** BOOTSTRAP.md (refuses to overwrite an existing `.claude/agents/` without explicit user confirmation)
- **Set by:** audit — `.claude/agents/` folder populated

### monorepo_layout
- **Required:** no (not all projects are monorepos)
- **Type:** enum [none, turborepo, pnpm-workspaces, npm-workspaces, yarn-workspaces, lerna, nx, rush, other]
- **Used by:** software-engineer (where new code goes), database-engineer (which package owns DB), ui-specialist (which package owns UI), BOOTSTRAP.md (where to write seed docs)
- **Set by:** audit — `turbo.json` → turborepo, `pnpm-workspace.yaml` → pnpm-workspaces, `nx.json` → nx, `lerna.json` → lerna, `rush.json` → rush, `workspaces` field in root `package.json` → npm/yarn workspaces, root `packages/` or `apps/` directory as fallback signal
- **Grill prompt:** only if audit returns `other` — "What monorepo tool? (free text)"

---

## Section 1 — Product (required)

### project_name
- **Required:** yes
- **Type:** string (kebab-case or snake_case; alphanumeric + dashes)
- **Used by:** every template (`{{project_name}}`), README seed, package.json default
- **Validation:** non-empty, no spaces, no special chars beyond `-` `_`
- **Grill prompt:** "What's the project name? Short, kebab-case ideally — this becomes the directory name and shows up in agent prompts."
- **Example:** `scorecard-saas`, `lead-funnel`, `triage-bot`

### one_line_pitch
- **Required:** yes
- **Type:** string (≤ 160 chars)
- **Used by:** README, reference-guide.md, every agent template's "project context" section
- **Validation:** must contain a verb and a target user; reject if it reads like a feature list rather than a value claim
- **Grill prompt:** "In one sentence, what does this project do for whom? Avoid feature lists; lead with the user outcome."
- **Example:** "Lets non-technical founders build and publish lead-generation scorecards in under 30 minutes."

### wedge
- **Required:** yes
- **Type:** string (1–3 sentences)
- **Used by:** reference-guide.md, marketing/copywriter templates, systems-architect ("design for current phase")
- **Validation:** must name **a specific user**, **a specific outcome**, **a measurable threshold** (time, cost, friction). Reject if all three aren't present.
- **Grill prompt:** "What's the wedge? The narrowest, sharpest version of 'X user gets Y outcome in Z time/effort.' This is what every Phase 0 decision gets tested against."
- **Example:** "A non-technical founder goes from 'I have an idea' to 'live scorecard collecting leads' in under 30 minutes."

### success_metric
- **Required:** yes
- **Type:** string (1 sentence)
- **Used by:** reference-guide.md, qa template (acceptance criteria framing)
- **Validation:** must be measurable; reject vague metrics like "users love it" or "it works"
- **Grill prompt:** "How do we know it's working? One measurable signal — engagement rate, time-to-first-value, conversion %. Pick one primary metric, not a dashboard."
- **Example:** "% of signed-up creators who publish a live scorecard within 7 days."

---

## Section 2 — Audience (required)

### target_user
- **Required:** yes
- **Type:** string (1–2 sentences)
- **Used by:** marketing template, copywriter template, ui-specialist (mental model of the reader), reference-guide.md
- **Validation:** must specify role/context, not just a demographic. "Founders" fails; "Non-technical founders running a coaching or consulting business who want to qualify leads" passes.
- **Grill prompt:** "Who is the target user? Role, context, and what they're trying to accomplish. Not a demographic."
- **Example:** "Non-technical founders of coaching/consulting businesses who want to qualify leads but don't want to hire a developer."

### user_alternatives
- **Required:** yes
- **Type:** list<string> (2–5 entries)
- **Used by:** marketing template (competitive positioning), researcher (when evaluating "why us"), reference-guide.md
- **Validation:** at least 2 entries; "nothing" or "no alternative" is invalid (there's always *some* alternative, even if it's "doing it manually")
- **Grill prompt:** "What does the target user do today instead? List the real alternatives — competitors, manual workarounds, 'just live with it.' At least two."
- **Example:** ["Hires a freelance dev to build a custom funnel", "Uses Typeform with manual scoring spreadsheets", "Doesn't qualify leads and accepts the cost in sales time"]

### voice_register
- **Required:** conditional (required if `surfaces` includes any user-facing copy surface)
- **Type:** string (1–2 sentences naming tone, register, vocabulary constraints)
- **Used by:** copywriter template (brand voice baseline), marketing template (positioning tone)
- **Validation:** must name **register** (formal/casual/technical/etc.) and **at least one anti-pattern** (what to avoid)
- **Grill prompt:** "What voice should this speak in? Name the register and at least one thing to avoid. 'Plainspoken, no jargon, no startup-speak' is a good answer; 'professional' is not."
- **Example:** "Plainspoken and direct. Avoids jargon, startup-speak, and 'unlock your potential' fluff. Talks like a senior peer, not a marketer."

---

## Section 3 — Methodology (required)

### conceptual_frame
- **Required:** yes
- **Type:** string (1–3 sentences naming the named approach or framework)
- **Used by:** systems-architect (informs domain modeling), copywriter (informs vocabulary), reference-guide.md
- **Validation:** must reference either (a) an external named methodology with a source, or (b) an explicitly internal approach with first principles. "Best practices" fails.
- **Grill prompt:** "What's the conceptual frame? The named approach this product is built around — a published methodology, a paper, an internal first-principles model. This becomes domain vocabulary across every agent."
- **Example:** "Daniel Priestley's 'Scorecard Marketing' methodology — four-part scorecard structure, tiered results, lead capture as part of the value exchange."

### methodology_source
- **Required:** conditional (required if `conceptual_frame` references external work)
- **Type:** string (book, paper, URL, or "internal — see {file}")
- **Used by:** reference-guide.md, researcher template (when verifying claims about the methodology)
- **Validation:** must be specific; reject "various sources" or "industry knowledge"
- **Grill prompt:** "Source for the methodology? Book title, paper, URL, or 'internal — see {file}'."
- **Example:** "Priestley, Daniel. *Scorecard Marketing.* 2022."

---

## Section 4 — Scope (required)

### domain_entities
- **Required:** yes
- **Type:** list<{name, description, ~3 examples or attributes}> (3–7 entries)
- **Used by:** systems-architect (data model seed), database-engineer (table naming), every agent template's "domain vocabulary" section
- **Validation:** 3–7 entries; each must be a noun, not a verb or adjective; each must have a one-line description
- **Grill prompt:** "Name the 3–7 core nouns this product revolves around. The things that get created, read, updated. For each: what is it, and one or two example attributes."
- **Example:**
  ```
  - scorecard: a published quiz with questions, scoring rules, and a results page (slug, creator_id, status)
  - question: a single item in a scorecard with options and weights (text, options, weight)
  - response: one respondent's submission of a scorecard (respondent_id, answers, score)
  - lead: a respondent who provided contact info (name, email, phone, source_response_id)
  ```

### pii_entities
- **Required:** yes (auto-derived; user confirms)
- **Type:** list<string> (subset of `domain_entities` names)
- **Used by:** security template (PII handling scope), database-engineer (RLS depth)
- **Validation:** must be set even if empty; if empty, grill confirms "really no PII?"
- **Grill prompt:** "From the entities above, which carry PII? Names, emails, phone, addresses, anything regulated. If none, confirm explicitly."
- **Example:** ["lead", "creator"]
- **Derivation:** `pdr-grill` proposes the list based on entity descriptions, user confirms or edits

### surfaces
- **Required:** yes
- **Type:** list<enum> [landing-page, marketing-site, dashboard, public-web-app, internal-web-app, api, cli, mobile-app, browser-extension, email, slack-bot, discord-bot, mcp-server, other]
- **Used by:** ui-specialist (skipped if no UI surface), marketing/copywriter (skipped if no public-facing surface), integration-engineer (presence of api / bot / email surfaces)
- **Validation:** at least 1 entry
- **Grill prompt:** "What surfaces does this product have? What does the user actually see or interact with? Pick all that apply."
- **Example:** ["landing-page", "public-web-app", "dashboard", "email"]

### phases
- **Required:** yes
- **Type:** ordered list<{phase_id, name, one_line_goal}> (open-ended; minimum 1 entry, which must be Phase 0)
- **Used by:** reference-guide.md, TASKS.md seed (Phase 0 work), systems-architect ("design for current phase")
- **Validation:** Phase 0 must be a foundation/scaffolding phase; later phases must each name an outcome (not a feature list); no upper cap, but grill warns past 7 ("are these distinct phases or a feature list?")
- **Grill prompt:** "Map out the phases. Phase 0 is foundation — scaffolding, schema, infra. Phase 1 onwards each name a user-facing outcome. Add as many as you have a clear picture of — don't pad, don't truncate."
- **Example:**
  ```
  - phase_0: Foundations — repo, schema, auth, deploy pipeline working end-to-end
  - phase_1: Single-tenant scorecard — one creator can build and publish one scorecard
  - phase_2: Multi-tenant — multiple creators, isolation enforced, dashboard
  - phase_3: Public marketplace — discoverable scorecards, embeds
  ```

---

## Section 5 — Constraints (required)

### approved_stack
- **Required:** yes
- **Type:** list<{tool, role, alternatives_to_propose_before_swap}>
- **Used by:** every engineer agent ("Stack is fixed; new deps need approval"), CONTRIBUTING.md seed, DECISIONS.md seed
- **Validation:** in `adopt-existing` mode, must include detected_framework + detected_db_layer + detected_auth at minimum, and any contradiction with the audit triggers a re-grill
- **Grill prompt — greenfield:** "What's the approved stack? List the 2–4 tools you're committing to. Anything outside this list will require a researcher memo and human approval before adoption."
- **Grill prompt — adopt-existing:** "I detected {framework}, {db_layer}, {auth}. Confirming these are the approved stack? Any others to add to the locked-in list?"
- **Example:**
  ```
  - tool: Supabase, role: auth + Postgres + storage + RLS
  - tool: Next.js (App Router), role: app framework
  - tool: Vercel, role: hosting
  - tool: Resend, role: transactional email
  ```

### compliance_regime
- **Required:** yes
- **Type:** list<enum> [none, gdpr, ccpa, hipaa, soc2, pci-dss, reco-tressa, casl, other]
- **Used by:** security template (review depth), database-engineer (RLS rigor), integration-engineer (data residency rules)
- **Validation:** must be explicitly set, even if `["none"]`. If `pii_entities` is non-empty and this is `["none"]`, grill pushes back.
- **Grill prompt:** "What compliance regimes apply? GDPR, CCPA, HIPAA, SOC 2, industry-specific (RECO, etc.), CASL, or 'none' if truly unregulated. If you have PII and say 'none,' I'll push back."
- **Example:** ["gdpr", "casl"]

### hard_nos
- **Required:** yes
- **Type:** list<string> (1–5 entries)
- **Used by:** systems-architect (rejects designs that violate), security (rejects designs that violate), reference-guide.md
- **Validation:** at least 1 entry; if user can't name any, grill probes ("really nothing this product will never do?")
- **Grill prompt:** "What's on the never-do list? Things this product will explicitly never do, even if asked. Helps every agent reject scope creep early."
- **Example:** ["never sell or share lead data with third parties", "never store payment info ourselves — Stripe-hosted only", "no AI-generated content presented as human-authored"]

---

## Section 6 — Agent roster (grill-derived from the work)

> **Philosophy:** No canonical roster. No defaults. The agent roster is built by analysing the task breakdown the grill produced — "what specialists does this work need?" Like a boss hiring for a specific job, or a heist crew assembled around the score. A Discord bot might need a `bot-protocol-engineer` and a `community-policy-reviewer`. An ML pipeline might need a `data-scientist`, an `ml-ops-engineer`, and an `eval-engineer`. A regulated SaaS might need `security`, `compliance-reviewer`, `privacy-engineer`. A solo CLI tool might need only `software-engineer` and `qa`. **The roster is whatever the work needs — nothing more, nothing presumed.**

### agent_roster
- **Required:** yes (must contain at least 1 agent)
- **Type:** list<agent_spec> — see structure below
- **Populated by:** grill, after analysing the task breakdown it produced for `TASKS.md`. Grill drafts the roster; user reviews and edits before BOOTSTRAP runs.
- **Used by:** BOOTSTRAP.md (scaffolds one agent file per spec, all from `templates/agent-blank.md`); all-hands exercise (each agent runs Phase A + Phase B); handoff-mesh audit
- **Validation:**
  - At least 1 entry. Most projects will have 3–8.
  - Each agent's `description` must name trigger conditions, not just a job title ("Use for X when Y" beats "Handles X").
  - Each agent must declare at least one `scope_does_not` entry (forces explicit boundaries).
  - Each agent must name at least one initial handoff target (refined later by Phase B). No agent operates in isolation.
  - Names must be unique within the roster, kebab-case.

#### Special handoff targets

The `agent_name` field in `initial_handoffs_to[]` accepts the names of other agents in the roster, plus these reserved special targets:

| Reserved name | Meaning |
|---|---|
| `human` | The human in the loop. Used for escalations that need product-level judgment (e.g., a CRITICAL security finding that requires choosing between conflicting requirements, a policy call with community-fit implications). |
| `orchestrator` | The Claude Code main session. Used when an agent surfaces parallelizable work or an ambiguity that needs cross-roster coordination. |

Reserved targets do NOT need reciprocation in the handoff-mesh audit — `human` and `orchestrator` aren't roster members, so the audit treats their inbound handoffs as trivially valid. Every agent will typically have at least one handoff to `human` for escalation paths.

#### agent_spec structure

```
{
  name: string                          # kebab-case, unique
  description: string                   # the frontmatter description Claude Code reads to delegate; load-bearing
  model: enum [opus, sonnet, haiku]     # Opus for decision-shaping, Sonnet for implementation, Haiku for fast dispatch
  rationale: string                     # why grill proposed this agent — which tasks/work made them necessary
  scope_owns: list<string>              # 3–6 things this agent owns
  scope_does_not: list<string>          # 2–4 things this agent explicitly doesn't do
  domain_folder: string                 # default: docs/<derived-from-name>/, can override
  is_mandatory_reviewer: bool           # true if this agent must review certain changes (e.g., a security-style role)
  mandatory_review_triggers: list<string> | null   # only if is_mandatory_reviewer: list of change types that must route through this agent
  initial_handoffs_to: list<{
    agent_name: string,
    condition: string                   # when this handoff fires
  }>
  first_task_deliverable: {             # the agent's first project-specific output, written on first invocation if not yet present
    file: string,                       # e.g., "docs/security/threat-model.md"
    description: string                 # what's in that deliverable
  } | null                              # null if no first deliverable
}
```

#### Grill's roster derivation (how it builds this list)

1. Read the task breakdown grill produced for `TASKS.md`
2. Cluster tasks by skill demand: "this task needs schema design", "this task needs UI work", "this task needs threat modeling", etc.
3. For each skill cluster of meaningful size, propose an agent. For one-off skill needs, fold into an existing agent's scope.
4. For each agent: name them after the role (kebab-case), draft a `description` that names trigger conditions, pick a model based on whether the work is decision-shaping or implementation, draft `scope_owns` from the actual tasks, draft `scope_does_not` from work explicitly outside their lane.
5. Pass through to surface the handoff sketch: "if agent A produces X and agent B consumes X, that's a handoff" — populate `initial_handoffs_to`.
6. Identify mandatory-reviewer agents — any agent whose lane includes "must review changes touching Z" (security on PII, compliance on regulated data, etc.). The trigger list is concrete change types.
7. For each agent: if there's a clear first project-specific deliverable they should write before any other work (threat model, voice doc, data-model proposal, etc.), specify `first_task_deliverable`. Many agents won't have one — that's fine.
8. Present the draft roster to the user with rationale per agent. User can add, remove, rename, adjust scope.

#### Grill prompt (after task breakdown is complete)

"Looking at the task breakdown across the phases, here are the kinds of work this project needs done. Drafting a purpose-built agent roster — for each agent I'll show you what they own, why they exist, and who they'd hand off to. We're building a crew for this specific project, not picking from a menu. Edit anything that doesn't fit. Add agents I missed. Remove agents whose work could be folded into someone else's lane."

#### What gets enriched later (not by grill)

The `agent_roster` field captures the **scaffolding spec**, not the agent's full identity. After BOOTSTRAP runs, the all-hands exercise enriches each agent file with:
- Detailed scope (Phase A: who I am, what I do well, what I don't)
- Self-described handoffs with conditions (Phase B, refined after seeing other agents' Phase A)
- Good-prompt / bad-prompt examples
- One surprising thing about the agent
- Output style they prefer

The grill's job is to scaffold accurately enough that the all-hands exercise has good raw material. It doesn't try to write the full agent identity — that comes from the agent itself, in the all-hands.

---

## Derived variables (computed by BOOTSTRAP, not grilled)

These are not fields the user answers — they are computed from PDR fields by BOOTSTRAP and made available to templates the same way regular fields are. Centralizing the derivation logic prevents each template from re-implementing "is there an email surface" or "should I mention RLS." If a derivation feels too clever, it's probably a missing PDR field instead.

| Derived variable | Type | Formula | Used by |
|---|---|---|---|
| `has_email_surface` | bool | `"email" in surfaces` | security (email auth review), integration-engineer (email-provider setup), copywriter (email copy scope) |
| `has_public_surface` | bool | any of `[landing-page, marketing-site, public-web-app]` in `surfaces` | security (public URL enumeration concerns), marketing (presence of conversion surfaces) |
| `public_surfaces` | list<surface> | `[landing-page, marketing-site, public-web-app]` ∩ `surfaces` | security (per-surface threat model), marketing |
| `has_db_layer` | bool | `db_layer != "none" && db_layer != null` | security, database-engineer, systems-architect |
| `db_access_control_term` | string | `"RLS policies"` if `db_layer` ∈ `{supabase, raw-pg}`; `"row-level access policies"` if `db_layer ∈ {prisma, drizzle, kysely}` and DB is Postgres; `"access policies"` otherwise | security (avoids hardcoding "RLS" for non-Postgres projects) |
| `has_integrations` | bool | true if `approved_stack` contains any external service (i.e., not the framework, not the language runtime, not a self-hosted DB) | integration-engineer, security (third-party data flow review) |
| `external_integrations` | list<{tool, role}> | subset of `approved_stack` where the tool is an external service (Resend, Stripe, Twilio, OpenAI, Vercel-as-host, Supabase-as-managed-service, etc.) | integration-engineer (specific systems to manage) |
| `has_auth` | bool | `auth_provider != "none" && auth_provider != null` | security (auth flow review depth), software-engineer |
| `is_monorepo` | bool | `monorepo_layout != "none" && monorepo_layout != null` | every engineer agent (workspace awareness) |
| `current_phase` | string | always `"phase_0"` at bootstrap; updated in-repo by orchestrator as the project advances (this field decays — agents read it from the project's reference guide, not the original PDR) | systems-architect, TASKS.md seed |
| `domain_folder_<agent>` | string | `docs/<derived-domain>/`. Derivation: agent name → domain via canonical map (`security` → `security`, `ui-specialist` → `ui`, `database-engineer` → `architecture` or `db` based on convention, `marketing` → `marketing`, etc.); for net-new agents, derive from the `define_new_agents.name` field by stripping role suffixes | every agent's coordination section |

**Rules for adding a derived variable:**
1. The formula must be pure — input is PDR fields, no side effects, no I/O
2. The derivation must be unambiguous — a single source of truth, not "BOOTSTRAP figures it out"
3. If a template needs a value not derivable from PDR, that's a missing PDR field, not a missing derivation — add the field first
4. Derived variables follow the same per-field versioning as PDR fields

**Where derivations live:** BOOTSTRAP computes these once at the start of substitution and caches them. Templates reference them like any other variable. The derivation logic is documented in BOOTSTRAP.md (or its referenced helpers); templates don't re-derive.

---

## Validation rules (cross-field)

These run after all fields are filled, before BOOTSTRAP is invoked.

1. **PII without compliance:** if `pii_entities` is non-empty and `compliance_regime = ["none"]`, hard-stop and re-grill compliance with the PII list shown.
2. **Stack contradiction:** if `bootstrap_mode = adopt-existing` and `approved_stack` doesn't include the audited framework, hard-stop and ask whether the audit is wrong or the user is migrating.
3. **Surface without role:** if `surfaces` includes `landing-page` or `marketing-site` but `agent_roster_overrides.exclude` includes `marketing` or `copywriter`, warn and confirm.
4. **Phase 0 missing scaffolding:** if `phases[0]` doesn't name foundational work (schema, deploy, auth, scaffolding keywords), warn — Phase 0 should be infrastructural.
5. **Wedge without measurable threshold:** if `wedge` doesn't contain a number, time unit, or quantifiable outcome, push back once.

---

## Template variable map (what BOOTSTRAP substitutes)

This table covers fields **directly from `.pdr.md`**. Derived variables (computed by BOOTSTRAP) live in their own section above and are referenced in templates by the same `{{name}}` syntax. Both kinds are valid template inputs.

| Variable | PDR field | Templates that consume it |
|---|---|---|
| `{{project_name}}` | `project_name` | all agents, README, package.json |
| `{{one_line_pitch}}` | `one_line_pitch` | README, reference-guide, all agents |
| `{{wedge}}` | `wedge` | reference-guide, systems-architect, marketing, copywriter |
| `{{success_metric}}` | `success_metric` | reference-guide, qa |
| `{{target_user}}` | `target_user` | marketing, copywriter, ui-specialist |
| `{{user_alternatives}}` | `user_alternatives` | marketing, researcher |
| `{{voice_register}}` | `voice_register` | copywriter, marketing |
| `{{conceptual_frame}}` | `conceptual_frame` | systems-architect, copywriter, reference-guide |
| `{{methodology_source}}` | `methodology_source` | reference-guide, researcher |
| `{{domain_entities}}` | `domain_entities` (formatted as bulleted list) | systems-architect, database-engineer, all agents (vocabulary) |
| `{{pii_entities}}` | `pii_entities` | security, database-engineer |
| `{{surfaces}}` | `surfaces` | ui-specialist, integration-engineer |
| `{{phases}}` | `phases` (formatted) | reference-guide, systems-architect, TASKS.md seed |
| `{{current_phase}}` | derived: always `phase_0` at bootstrap | systems-architect, TASKS.md seed |
| `{{approved_stack}}` | `approved_stack` (formatted) | all engineers, CONTRIBUTING.md |
| `{{compliance_regime}}` | `compliance_regime` | security, database-engineer, integration-engineer |
| `{{hard_nos}}` | `hard_nos` (formatted as list) | systems-architect, security, reference-guide |
| `{{framework}}` | `detected_framework` | software-engineer, ui-specialist |
| `{{router}}` | `detected_router` | software-engineer (if Next.js) |
| `{{language}}` | `detected_language` | all engineers |
| `{{db_layer}}` | `detected_db_layer` | database-engineer |
| `{{auth_provider}}` | `detected_auth` | security |
| `{{monorepo_layout}}` | `monorepo_layout` | software-engineer, ui-specialist, database-engineer (where their work belongs) |

---

## Output: the produced `.pdr.md`

After grilling, `pdr-grill` writes `.pdr.md` at the project root with this structure (YAML frontmatter for machine fields, prose body for human-readable expansion):

```markdown
---
schema_version: 0.2
project_name: <name>
bootstrap_mode: greenfield | adopt-existing
detected_framework: <value>
detected_router: <value>
detected_language: <value>
detected_styling: [<list>]
detected_ui_library: <value>
detected_db_layer: <value>
detected_auth: <value>
detected_existing_docs: [<list>]
detected_existing_agents: <bool>
monorepo_layout: <value>
surfaces: [<list>]
domain_entities:
  - name: <noun>
    description: <one-line>
    examples: [<list>]
pii_entities: [<subset>]
phases:
  - id: phase_0
    name: <name>
    goal: <one-line>
  # ... open-ended ...
approved_stack:
  - tool: <name>
    role: <role>
    alternatives_to_propose_before_swap: [<list>]
compliance_regime: [<list>]
hard_nos: [<list>]
agent_roster:
  - name: <kebab-case>
    description: <delegation trigger sentence>
    model: opus | sonnet | haiku
    rationale: <why grill proposed this agent>
    scope_owns: [<list of 3-6>]
    scope_does_not: [<list of 2-4>]
    domain_folder: docs/<derived>/
    is_mandatory_reviewer: <bool>
    mandatory_review_triggers: [<list>] | null
    initial_handoffs_to:
      - agent_name: <name>
        condition: <when this handoff fires>
    first_task_deliverable:
      file: <path>
      description: <what's in it>
    # ... or null
  # ... more agents
---

# {{project_name}}

## One-line pitch
<one_line_pitch>

## The wedge
<wedge>

## Target user
<target_user>

## Methodology
<conceptual_frame>

(Source: <methodology_source>)

## Voice register
<voice_register>

## Success metric
<success_metric>
```

This is what BOOTSTRAP.md reads. The frontmatter is the structured contract; the body is the human-readable expansion that gets composed into `docs/reference-guide.md`.

---

## Field versions (per-field versioning alongside whole-document version)

Each field carries an independent version so additive changes don't force consumers to re-validate every other field. The whole-document `schema_version` only bumps on a breaking change to overall structure (e.g., section reorder, required→optional swap, removal). Field-level bumps cover scope/type/validation tweaks within a stable section.

`pdr-grill` and BOOTSTRAP both check **whole-document version compatibility first** (refuse to consume `.pdr.md` files older than schema major version), then **field-level version per field** (gracefully handle additive changes — e.g., a new optional sub-field added to `define_new_agents` won't break a `.pdr.md` written before that sub-field existed).

| Field | Version | Last changed | Note |
|---|---|---|---|
| `bootstrap_mode` | 0.1 | 0.1 | |
| `detected_framework` | 0.1 | 0.1 | |
| `detected_router` | 0.1 | 0.1 | |
| `detected_language` | 0.1 | 0.1 | |
| `detected_styling` | 0.1 | 0.1 | |
| `detected_ui_library` | 0.1 | 0.1 | |
| `detected_db_layer` | 0.1 | 0.1 | |
| `detected_auth` | 0.1 | 0.1 | |
| `detected_existing_docs` | 0.1 | 0.1 | |
| `detected_existing_agents` | 0.1 | 0.1 | |
| `monorepo_layout` | 0.1 | 0.2 | Added in 0.2 |
| `project_name` | 0.1 | 0.1 | |
| `one_line_pitch` | 0.1 | 0.1 | |
| `wedge` | 0.1 | 0.1 | |
| `success_metric` | 0.1 | 0.1 | |
| `target_user` | 0.1 | 0.1 | |
| `user_alternatives` | 0.1 | 0.1 | |
| `voice_register` | 0.1 | 0.1 | |
| `conceptual_frame` | 0.1 | 0.1 | |
| `methodology_source` | 0.1 | 0.1 | |
| `domain_entities` | 0.1 | 0.1 | |
| `pii_entities` | 0.1 | 0.1 | |
| `surfaces` | 0.1 | 0.1 | |
| `phases` | 0.2 | 0.2 | Cap removed in 0.2; now open-ended |
| `approved_stack` | 0.1 | 0.1 | |
| `compliance_regime` | 0.1 | 0.1 | |
| `hard_nos` | 0.1 | 0.1 | |
| `agent_roster` | 0.1 | 0.4 | Replaces `canonical_roster_overrides` + `define_new_agents`. No defaults; grill-derived from task breakdown. |
| ~~`canonical_roster_overrides`~~ | — | 0.4 | Removed in 0.4 — no canonical roster exists |
| ~~`define_new_agents`~~ | — | 0.4 | Removed in 0.4 — every agent is project-specific |
| ~~`model_preferences`~~ | — | 0.4 | Removed in 0.4 — model is set per-agent inside `agent_roster[].model` |
| `has_email_surface` (derived) | 0.1 | 0.3 | New in 0.3 |
| `has_public_surface` (derived) | 0.1 | 0.3 | New in 0.3 |
| `public_surfaces` (derived) | 0.1 | 0.3 | New in 0.3 |
| `has_db_layer` (derived) | 0.1 | 0.3 | New in 0.3 |
| `db_access_control_term` (derived) | 0.1 | 0.3 | New in 0.3 |
| `has_integrations` (derived) | 0.1 | 0.3 | New in 0.3 |
| `external_integrations` (derived) | 0.1 | 0.3 | New in 0.3 |
| `has_auth` (derived) | 0.1 | 0.3 | New in 0.3 |
| `is_monorepo` (derived) | 0.1 | 0.3 | New in 0.3 |
| `current_phase` (derived) | 0.1 | 0.3 | New in 0.3 |
| `domain_folder_<agent>` (derived) | 0.1 | 0.3 | New in 0.3 |

When a field is added, removed, or has its type/validation changed, bump that field's version and append a row to the whole-document changelog. Consumers reading an older `.pdr.md` will see a lower per-field version and can decide whether to upgrade in place or pass through.

---

## Resolved questions (for v0.1 → v0.2)

1. **Phase count: open-ended.** Soft warning past 7 entries. (Was: 3–5 cap.)
2. **Net-new agents: encouraged.** Each project's roster is purpose-built; canonical 10 are a starting set. (Was: open question.)
3. **`voice_register` for non-public surfaces: not required.** Stays conditional on user-facing copy surface. (Was: open question; resolved as conditional.)
4. **Monorepo detection: yes.** Added `monorepo_layout` to Section 0. (Was: out-of-scope candidate.)
5. **Versioning: both whole-document and per-field.** See "Field versions" table above. (Was: whole-document only.)

## Resolved questions (for v0.2 → v0.3)

Surfaced while building the `security` agent template prototype:

6. **Templating syntax:** Handlebars-lite. `{{var}}` for substitution, `{{#if flag}}...{{/if}}` for conditionals, `{{#unless}}` for inverse. No loops, no nested conditionals, no expressions. HTML comments at template top are metadata, stripped on output.
7. **Stack-specific vocabulary:** templates should not hardcode "RLS" — use the derived `db_access_control_term` so non-Postgres projects get the right phrasing.
8. **Surface-conditional sections:** "email auth review," "public URL enumeration," etc. only render when relevant — gated on derived booleans (`has_email_surface`, `has_public_surface`).
9. **First-task pattern:** rather than hardcoding project-specific content into a template, instruct the agent to *produce* its first project-specific deliverable on first invocation. Now lives in the `agent_spec.first_task_deliverable` field.

## Resolved questions (for v0.3 → v0.4)

10. **No canonical roster.** Every agent is project-specific. The grill builds the roster from the task breakdown; no defaults.
11. **No agent-pattern library.** A library would bias roster proposals toward common roles. Every agent scaffolds from a single blank template (`templates/agent-blank.md`) using the per-agent `agent_spec` the grill produced.
12. **Task breakdown lives in `TASKS.md`, not `.pdr.md`.** Grill produces both. PDR captures product/scope; TASKS.md captures work.
13. **Roster derivation order: phases → tasks_per_phase → agent_roster.** Grill cannot propose a roster without first knowing what work needs doing.
