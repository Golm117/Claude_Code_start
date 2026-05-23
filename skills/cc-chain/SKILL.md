---
name: cc-chain
description: Drive an async Claude Code chain — investigate the next slice, run confirm-understanding at a configurable threshold (default 95%), auto-fire via RemoteTrigger if confidence clears, otherwise ping the user. Use when the user has explicitly authorized auto-fire mode (Pattern B+) and asks for the next slice in a chain.
---

# cc-chain — async CC chain driver

Codifies the **Pattern B+** workflow established 2026-05-22: the main session plans slices, fires remote Claude Code sessions to execute them via `RemoteTrigger`, monitors for the commit via `ScheduleWakeup`, and verifies the diff before chaining the next slice. Confidence gating at a configurable threshold (default 95%) decides whether to auto-fire or pause for the user.

This is a **strict alternative** to the default "write a prompt for the user to paste into a fresh CC session" workflow (see memory `feedback_delegate_edits_to_cc.md`). The user MUST have explicitly authorized auto-fire mode before this skill kicks in. The Agent-tool-spawning prohibition still applies — `RemoteTrigger` is the only execution channel.

## When to invoke

Invoke when ALL of these hold:

- User has explicitly opted into async / Pattern B+ for the current chain ("yes, auto-fire," "go async," "fire it without asking me," etc.). Once opted in, the skill stays active for subsequent slices in the same chain until the user says otherwise.
- User is asking for the next concrete unit of work in a chain (e.g. "do the next slice," "continue," "go").
- The work is a discrete, well-scoped slice — not exploratory / open-ended planning.

Do NOT invoke for:

- A first-time user who hasn't seen the workflow yet — explain Pattern B+ and get explicit consent first.
- Hard-to-reverse production work (migrations, data backfills, paying API calls beyond the slice's normal cost). Default to manual relay.
- Anything touching compliance posture, RECO-facing surfaces, or external integrations without explicit user sign-off.
- Tasks that are better as research (use `Explore` agent) or planning (handle in main conversation).

## Invocation arguments

- `threshold:N` — Override the default 95% confidence threshold (e.g. `cc-chain threshold:90` lets the chain proceed with slightly lower confidence). Use sparingly; the gate exists to catch real unknowns. Numbers below 80 should be rejected with a warning.
- `branch-explore` — Enable end-of-chain "what to work on next" surfacing. Without this flag, the skill stops cleanly when no queued slice remains. With it, the skill enumerates plausible next directions and asks the user to pick. Off by default (Decision 8 default).
- Without args, defaults apply: 95% threshold, no branch-explore.

## The workflow loop

### Step 1 — Verify previous CC commit (when applicable)

Skip this step on the first slice of a chain. For all subsequent slices:

1. `git fetch origin && git log --oneline origin/main..origin/dev` to surface commits ahead of main.
2. Identify the most recent commit's slice name from the subject line.
3. `git log -1 --stat <commit>` for the file scope; `git diff <commit>~..<commit> --stat | tail -15` for the change footprint.
4. Cross-check against the slice's prompt spec — did the CC session ship what was asked? Note any deviations.
5. If the commit touched persistence, run targeted queries via the project's Supabase MCP (or whatever the project's database access pattern is) to confirm rows have the expected shape.
6. If anything looks off — wrong file scope, missing tests, deviation that has architectural implications, broken build — **ping the user immediately**. Do not chain to the next slice. The whole point of the gate is preventing compound divergence.

### Step 2 — Investigate next slice context

The "investigate before asking" rule from `confirm-understanding` applies. Read the relevant files. Grep for symbols. Run live queries against the data (don't trust mental models of schemas — production data has bitten us before; see memory `project_proptx_listing_key_shape` for the canonical example). Verify shapes the prompt will cite — file paths, function names, type definitions, regex patterns, anything that gets quoted verbatim.

**Project agent discovery (canonical Claude Code conventions only — fall back to silence if nothing found):**

- `Glob` `.claude/agents/*.md` → enumerate any subagents this project has scaffolded. For each, read just enough to capture the frontmatter `name` and `description` (the description is the one-line purpose that drives delegation).
- `Read` whichever of `CONTEXT.md`, `AGENTS.md`, `ROLES.md` exists at the repo root for any role-summary text describing when each agent fits. Most repos won't have these; that's fine.
- Note the agent roster (name + one-line purpose) in your planning context. You'll surface it to the remote CC session in Step 4.

If `.claude/agents/` is absent or empty AND no role-summary file exists, skip silently. The Step 4 prompt simply won't carry a roster section — no error, no warning, no degraded behaviour. This is a graceful enhancement, not a precondition.

Do NOT bake project-specific paths beyond the canonical ones above. Different projects will use different conventions; the skill's job is to consume the canonical layout when present.

### Step 3 — Confirm-understanding internally

Run the `confirm-understanding` skill's mental loop:

1. Bucket the slice into confident vs needs-decision.
2. For decisions: surface them with a clear recommendation each.
3. Self-report a numeric confidence percentage. Be honest.

The threshold for auto-fire is the invocation's `threshold:N` arg (default 95%). Confidence below the threshold pings the user; at or above auto-fires.

**Confidence-low conditions that should always ping** (regardless of self-reported number):

- Live data divergence from your mental model (e.g. a regex you cited doesn't match production data).
- Architectural fork in the road (single endpoint vs split, model choice, library choice with real dep tree impact, etc.).
- Anything touching production data in a hard-to-reverse way.
- Anything affecting compliance posture, RECO-facing copy, or external-API contracts.
- A previous CC session diverged from the prompt and the divergence has architectural implications worth user input.

If any of these apply, drop confidence to whatever value pings the user even if you were otherwise above threshold. Surfacing once is cheap; auto-firing a wrong call is expensive.

### Step 4 — Gate: auto-fire or ping

**At or above threshold:** auto-fire.

1. Draft the prompt in a markdown file (e.g. `/tmp/slice-N-prompt.md`) with 4-backtick fence convention for one-click copy (per memory `feedback_agent_prompts.md`). Include async-mode framing — the prompt's recipient is a CC session with no human in the loop, so the commit message + diff IS the report back to main. Tell the CC session to push to `dev` only and never fast-merge.

   **If Step 2's agent discovery turned up subagents, include a "Subagents available in this project" section in the prompt:**
   - List each agent by name with the one-line purpose from its frontmatter `description` (or the role doc, if richer).
   - Add invocation hints tied to the slice's actual shape — e.g. "for the market-research step, invoke `market-research`"; "before commit, invoke `code-review` on the diff"; "if the slice touches PII, route through `compliance-review`".
   - If the slice doesn't obviously call for any of the available agents, list the roster anyway but skip the hints rather than inventing forced fits. The remote session can still reach for an agent on its own judgement; surfacing the roster removes the "I didn't know it existed" failure mode without dictating misuse.
   - If no agents were found in Step 2, omit this section entirely — don't draw attention to its absence.
2. Construct the `RemoteTrigger` create body via Python or jq to handle the prompt's JSON escaping correctly (see "RemoteTrigger body shape" below). Set `run_once_at` to a comfortable buffer past now (5+ minutes to absorb dispatch lag; +90s has been bitten by latency in practice).
3. Fire via `RemoteTrigger {action: "create", body: {...}}`. Capture the `id` from the response.
4. Schedule a `ScheduleWakeup` for ~15-20 minutes (CC sessions for medium slices typically run 10-15 min; add buffer). The wakeup's `prompt` field tells your future self what to do on resume: re-run the loop from Step 1.
5. Post a brief status message to the conversation: routine ID, expected completion window, chain state.

**Below threshold:** ping the user.

1. Surface the decisions with each open question + your recommendation.
2. Report the numeric confidence and what's pulling it down.
3. STOP. Do not fire. Wait for the user to answer.
4. After their input, re-evaluate confidence. If it now clears, return to Step 4 auto-fire. If not, surface again.

### Step 5 — On wakeup

The harness re-invokes you when the wakeup fires. Resume from Step 1 — verify the previous commit. If the slice is done, decide whether to chain or stop (Step 6). If not done yet, reschedule another wakeup with the same prompt.

### Step 6 — End-of-chain decision

When you've completed verification of the most recent slice and there's no queued next slice:

- **Default behavior (no `branch-explore`):** report chain status to the conversation. List what shipped. Note any deviations or follow-ups surfaced during verification. STOP. Do NOT auto-decide what to work on next — that's a planning decision belonging to the user.

- **With `branch-explore` flag:** also enumerate plausible next directions based on the project's roadmap, recent commits, and any TODOs surfaced during verification. Present them with a recommendation. Ask the user to pick.

In both cases, end the chain cleanly. The user re-invokes the skill for the next chain.

## RemoteTrigger body shape

For a one-time fire (the only mode this skill uses):

```json
{
  "name": "<slice description>",
  "run_once_at": "<RFC3339 UTC future timestamp, +5min minimum>",
  "enabled": true,
  "job_config": {
    "ccr": {
      "environment_id": "<env id — get from RemoteTrigger list or user>",
      "session_context": {
        "model": "claude-sonnet-4-6",
        "sources": [{"git_repository": {"url": "<repo URL>"}}],
        "allowed_tools": ["Bash", "Read", "Write", "Edit", "Glob", "Grep", "WebSearch", "WebFetch"]
      },
      "events": [{
        "data": {
          "uuid": "<lowercase v4 UUID>",
          "session_id": "",
          "type": "user",
          "parent_tool_use_id": null,
          "message": {"content": "<the full prompt>", "role": "user"}
        }
      }]
    }
  }
}
```

The user's claude.ai MCP connectors auto-attach to the routine (verified in practice — Supabase, Gmail, Drive, Calendar, Notion all came along uninvited). Plan prompts assuming the CC session has those MCPs available.

## Async-mode prompt conventions

When writing prompts for an async CC session, include these (different from the manual-paste prompt conventions):

- **Header note:** "You are running in ASYNC MODE — there is no live terminal to ping when done. Your commit (and its message body) IS the report back to main. Make the commit body detailed enough that main can plan the next slice from it."
- **Push instruction:** "Push to `dev` ONLY. DO NOT fast-merge to main — the user gates that."
- **Blocker reporting:** "If you encountered a hard blocker that prevented finishing the slice, push a commit titled `Slice N BLOCKED: <reason>` with a brief explanation in the body and don't ship a half-done implementation."
- **Commit body requirement:** "End the commit body with the test count delta vs the baseline and any deviations from this prompt's spec, so main can read it on review."
- Otherwise, use the same 4-backtick fence convention, self-contained file paths, and explicit constraints established in memory `feedback_agent_prompts.md` and `feedback_delegate_edits_to_cc.md`.

## Failure modes to avoid

- **Stale anchor:** the conversation may have been running for a while; ALWAYS re-fetch current time via `date -u +%Y-%m-%dT%H:%M:%SZ` before computing `run_once_at`. Use a +5min minimum buffer — +90s has been rejected as in-the-past due to tool latency.
- **Compound divergence:** never chain a new slice without verifying the previous commit. If the previous slice shipped something wrong and you chain blindly, the next slice builds on broken ground.
- **Self-deceiving confidence:** the 95% threshold is meaningful only if self-reports are honest. Investigate first (Step 2); don't report high confidence on a slice you haven't grounded.
- **Forgetting the user:** if the user is mid-conversation when a wakeup fires, surface chain status alongside their open question — don't bury it.
- **Chain past intent:** when the user's original chain authorization ends (e.g. they said "do these three slices" and you've done three), stop. Don't extrapolate into "well, the next logical thing would be...".

## Hard prohibitions

These hold regardless of mode:

- NEVER spawn an `Agent` subagent for edits. The Agent tool is research-only.
- NEVER push to `main` directly from the main session. The user's manual fast-merge gate is non-negotiable.
- NEVER skip verification of the previous CC commit before chaining. Even if the previous slice "obviously worked," verify the diff and run targeted DB queries if it touched persistence.
- NEVER auto-decide what to work on next (without `branch-explore`). The chain stops at end of queued work.

## Memory references

- `feedback_delegate_edits_to_cc.md` — the canonical statement of when async-fire vs manual-paste applies.
- `feedback_agent_prompts.md` — prompt formatting (4-backtick fence for one-click copy).
- `feedback_verify_production_swaps.md` — verify shapes before acting; reasonable-call doesn't override pause-on-hard-to-reverse.
- `project_proptx_listing_key_shape.md` — example of a regex assumption that broke 70% of production data; always grep live data, never trust prompt-time assumptions.
- Any project-specific memory files about the codebase the chain is building in.

## Example invocations

- `cc-chain` — default 95% threshold, no branch-explore. Pings on any uncertainty.
- `cc-chain threshold:90` — looser gate. Use when slices are well-grounded and the chain has already proven stable.
- `cc-chain branch-explore` — default threshold, but ask "what's next" at chain end.
- `cc-chain threshold:90 branch-explore` — both.
