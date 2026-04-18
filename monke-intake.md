# WyrdMonke Intake — Name Your Wish

> **Usage:** `/monke-intake "<what you want>"` or `/monke "<what you want>"` as shortcut.
>
> One sentence in — a planned feature, a nagging bug, a half-baked idea. Monke reads the request, reads the project, names the pipeline, and walks it end-to-end as permanent lead. No "which skill do I run?" No ceremony. One approval at the front, one deliverable at the back.

---

## Arguments

`$ARGUMENTS` parsing:
- **First positional (required):** the request string — natural-language description of what the user wants. May be quoted or unquoted. Anything that isn't a rigor keyword or a known skill name.
- **No other positionals.** Intake is the leader; sub-skills are its workers. The user does not pass sub-skill args — intake derives them.

```bash
REQUEST="${ARGUMENTS:-}"

if [ -z "$REQUEST" ]; then
  echo "WHAT: No request supplied."
  echo "WHY:  /monke-intake needs something to route on — a feature, a bug, an idea."
  echo "HOW:  Re-run as: /monke-intake \"add webhook retry with exponential backoff\""
  exit 1
fi
```

- No silent defaults. If the request is empty → stop with WHAT/WHY/HOW.
- If the request is a single word that matches a known skill name (`recon`, `flash`, `rage:buggy`, etc.) → refuse with: "That's a skill override, not an intake request. Run `/monke <skill>` directly."

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**.

```
WHAT: Agent Teams Gate failed. `CLAUDE.md` exists but has no Agent Teams section.
WHY:  Intake spawns a classifier+scoper team in Phase 2 and coordinates sub-skill workers in Phase 5.
      Without enablement, teammates cannot coordinate and the intake leader loses its workforce.
HOW:  Run `/monke-sync` to pull the latest template, OR copy the Agent Teams section from
      `monke-CLAUDE.md` into your project's `CLAUDE.md`, then re-run `/monke-intake`.
```

- `monke-docs/` exists. If missing → "Run `/monke-init` first. Intake needs the doc scaffold to route."
- `.monke-config.md` exists → proceed normally.
  - If missing AND `$REQUEST` is set → intake treats this as a greenfield cold-start. Phase 1 prepends `/monke-init` to the pipeline as an invisible bootstrap step (no user gate). Rigor defaults to `standard` unless the request implies otherwise (keywords: "quick", "MVP", "prototype" → light; "production", "compliance", "enterprise" → thorough).
  - If missing AND `$REQUEST` is empty → stop with WHAT/WHY/HOW: "Run /monke-init first, or re-invoke with a request string (/monke-intake \"your wish\")."
- `monke-status.md` exists OR greenfield (no code). If code exists without status → intake will auto-dispatch `/monke` fast-onboarding first, then resume.

If any prerequisite fails → stop with WHAT/WHY/HOW. No guessing.

---

## Phase 1: Read the Jungle

Intake is the permanent lead. Before spawning any worker, it must know the terrain.

Emit one line: `Intake reading: rigor=<level>, status=<yes|no|pending>, hld=<yes|no>, components=<N>, request="<first-40-chars>..."`.

### Step 1.0: Cold-start Bootstrap (if needed)

If `.monke-config.md` is missing AND `monke-status.md` is missing AND no source code exists beyond a bare repo:
- This is a true greenfield cold-start.
- Dispatch `/monke-init` as an invisible bootstrap step with `standard` rigor (or inferred from request keywords).
- Wait for `/monke-init` to complete (it has its own HARD gate at the end; user sees that).
- After init returns, re-read `.monke-config.md`, `monke-status.md`, and `CLAUDE.md` before continuing to Step 1.1 (original "Read in this order" list).
- Record in the feature thread file that Step 0 = `/monke-init` bootstrap.

If `.monke-config.md` is missing AND source code OR `monke-status.md` exists:
- Something is inconsistent. Stop with: "WHAT: `.monke-config.md` missing but project shows prior state (status file or code). WHY: intake cannot infer rigor without config. HOW: run `/monke-init` to regenerate config, or restore from backup."

### Step 1.1: Read the terrain

Read in this order:
1. **`.monke-config.md`** — rigor + any per-project intake config. Set `RIGOR` env for downstream.
2. **`monke-status.md`** — project state, existing components, feature threads in flight, any Resume block.
3. **`monke-docs/hld.md`** — if exists, extract containers, components, boundary matrix (S7).
4. **`monke-docs/flash/flash-brief.md`** — if exists, note flash chain state.
5. **Existing feature threads** in monke-status.md `## Features` section (introduced by this skill; see Status Update).

If `monke-status.md` is missing AND source code exists → dispatch `/monke` (bare) for fast onboarding first. Intake waits, re-reads status, then proceeds to Phase 2.

If no status AND no code AND request implies "build something" → short-circuit: the request IS a flash brief napkin. Prefix Phase 2's classification with "greenfield likely."

### Step 1.5: Ensure intake scratch directory exists

Before any feature thread file is written:
- If `monke-docs/intake/` does not exist → create it (`mkdir -p monke-docs/intake`).
- If it exists but is a file (not a directory) → fail with WHAT/WHY/HOW: "WHAT: `monke-docs/intake` exists as a file, not a directory. WHY: intake needs to write thread files here. HOW: rename the file and re-run."
- Record in the thread file header that the directory was created (if bootstrapped) vs already existed.

---

## Phase 2: Classify + Scope (worker-pool agent team, 2 teammates parallel)

Spawn a `monke-intake-scout` team with 2 teammates running in parallel. Worker-pool archetype per design-specs S3.1. Lead (this skill) assembles their findings — does not duplicate their work.

**Teammate `codex-classifier`** — answers: what KIND of request is this?

Prompt includes: the full request string, the project state summary from Phase 1, and the classification taxonomy:

| Class | Signal | Default pipeline |
|-------|--------|------------------|
| **greenfield-mvp** | No code, request implies "build X from scratch", user wants speed | flash chain (spark → scope → sketch → blitz → pulse → snap) |
| **greenfield-production** | No code, request implies durability / compliance / team > 1 | design-first chain (hld → lld → test:test-plan → implement → test:test-run) |
| **feature-on-existing** | Code exists, request implies new capability on top | hld evolve → lld (new + neighbors) → test:test-plan → implement → test:test-run |
| **bug-fix** | Request implies "it's broken / wrong / fails" with existing code | rage:buggy → implement:fill → test:test-run |
| **refactor-cleanup** | Request implies "improve / clean up / remove / modernize" | rage:improv or rage:renounce or rage:echo → implement:fill → test:test-run |
| **investigation** | Request implies "figure out / understand / audit" without commitment to change | recon chain (survey → reconstruct → gaps → oqs → roadmap) |
| **ambiguous** | Cannot confidently place in any bucket | surface at Phase 4 gate with 2-3 candidate classes |

**Commit is never a pipeline step** — it's always a human-initiated follow-up at Phase 6 close. See Phase 6 for the closing suggestion. Intake never appends `ops:commit` to a pipeline, and it never auto-dispatches commit on behalf of the user. This invariant is why Phase 6 Anti-Pattern #5 ("Commit changes automatically at Phase 6 — Refuse") holds: the pipeline itself no longer contains a commit step, so there is nothing to auto-fire.

**`test` disambiguation.** The pipeline table uses `test:test-plan` (design-time: written after LLD, before implement) and `test:test-run` (post-implement verification). If a pipeline has NO LLD step (e.g., `bug-fix`), `test:test-plan` is skipped — only `test:test-run` runs. Never use bare `test` in a pipeline; always resolve to the specific sub-skill.

Output: one class name + confidence (HIGH/MEDIUM/LOW) + one-sentence reasoning. If LOW → list the two next-most-likely classes.

**Teammate `kairos-scoper`** — answers: which COMPONENTS are affected?

Prompt includes: the full request string, HLD S3 components table (if present), request keywords, and instructions to:
1. Extract noun phrases from the request.
2. Match against existing component names, file paths, and API surface (from HLD if available).
3. For each match, tag as `existing-affected`.
4. For non-matches that imply new capability, tag as `new-component-needed` + propose a name.
5. For boundary changes (request implies contract between existing components), tag as `boundary-shift`.

Output: affected components list with tags, proposed new components (if any), boundary shifts (if any).

**Lead synthesis (you, not the teammates):**

Wait for both to complete. Read their TaskUpdate + final message. If either teammate confidence is LOW and the user is at `thorough` rigor → spawn a third tie-breaker teammate `sigil-adjudicator` that reads both outputs and the request fresh, proposes a synthesis. Otherwise, synthesize directly.

Shut down teammates via `SendMessage shutdown_request` + `TeamDelete` before Phase 3.

---

## Phase 3: Synthesize Pipeline

Build the pipeline proposal. Structure:

```markdown
# Pipeline Proposal — <feature-thread-slug>

**Request:** "<original request string>"
**Classification:** <class> (confidence: <HIGH|MEDIUM|LOW>)
**Affected components:** <list from scoper>
**New components (if any):** <list>
**Boundary shifts (if any):** <list>
**Rigor:** <from .monke-config.md>

## Pipeline Steps
1. <step 1 — `/monke-*:skill` + args> — <one-line reason>
2. <step 2> — <reason>
3. <step 3> — <reason>
...

## Expected Decision Points
- <which sub-skill gates will HARD-surface>: <reason user must decide>
- <which SOFT gates will surface under thorough rigor>

## Artifacts That Will Be Created or Modified
- <file/dir>: <why>

## Estimated Human Touchpoints
- Before approval (this gate): 1
- During pipeline: <N> (from sub-skill HARD gates + TRIGGERED gates)
- Total: <N+1>

## Feature Thread ID
`feature-<YYYYMMDD-HHMM>-<short-slug>`
```

Write this proposal to `monke-docs/intake/<feature-thread-id>.md` (creating `monke-docs/intake/` if missing). This is the living feature thread — intake updates it at every step of Phase 5.

The proposal is mechanical — derived from the classifier's default pipeline + the scoper's component list. Do not over-elaborate. One page, scannable.

---

## Phase 4: Gate the Plan

⏸ **PG-1 [HARD] — Pipeline approval.** Always surfaces. Never skippable. This is the only gate the user MUST hit before work begins — every subsequent gate is sub-skill HARD/TRIGGERED surface.

Present the proposal. Ask:

```
Options:
  (a) approve          — execute pipeline as shown
  (b) adjust classification — change the class (I'll re-run Phase 3 with new class)
  (c) adjust components    — add/remove/rename components in scope
  (d) adjust pipeline steps — edit the step list directly
  (e) adjust rigor         — change rigor for this feature thread
  (f) abort                — discard the thread, no pipeline runs
  (g) explain              — justify a specific step or choice
```

On **approve**:

**If `## Features` section is missing from `monke-status.md`:**
Create it using the structure from `monke-docs/status-template.md`. Append to the status file in logical order (after Components, before Gate Audit Log).

Then write `PG-1 APPROVED` line to Gate Audit Log in `monke-status.md`, add the feature thread to the `## Features` section, proceed to Phase 5.

On **adjust**: re-run the affected phase (classification → re-run Phase 2's classifier only; components → re-run scoper only; pipeline → re-run synthesis only). Return to Phase 4 with the revised proposal.

On **abort**: delete `monke-docs/intake/<feature-thread-id>.md`, clear any Resume block, exit cleanly.

---

## Phase 5: Walk the Pipeline

Intake stays as permanent lead. Sub-skills are workers, dispatched one at a time. Intake never relinquishes lead — even when a sub-skill surfaces its own gates, those gates return control to the user, the user responds, and the sub-skill returns to intake when done.

**Per-step protocol:**

For each step in the approved pipeline:

1. **Pre-flight.** Announce: `Step <N>/<M>: /<dir>:<skill> <args> — <reason>`. Update the feature thread file with `Step <N>: STARTED <ISO-timestamp>`.

2. **Dispatch.** Invoke the sub-skill with its derived args. The sub-skill may spawn its own agent teams — those are workers of that sub-skill, not of intake. Intake waits for the sub-skill's Status Update to return.

3. **Gate handling.** If the sub-skill surfaces a gate (HARD or TRIGGERED or SOFT-under-thorough), that gate goes directly to the user. Intake does NOT intercept. User responds; sub-skill processes; sub-skill returns to intake.

4. **Post-step verification.** Read the updated `monke-status.md`. Verify the expected artifacts were written (per the proposal). If missing → fire `SKILL-GATE:intake-step-anomaly [TRIGGERED]`:
   ```
   WHAT: Step <N> (<skill>) returned but expected artifact <path> is missing.
   WHY:  Next step assumes this artifact exists; cannot proceed.
   HOW:  Re-run the step, abort the thread, or skip with override.
   ```

5. **Thread update.** Append to feature thread file: `Step <N>: COMPLETED <ISO-timestamp> — <one-line outcome>`. Update `monke-status.md` `## Features` row for this thread.

6. **Continue or branch.** If the step's outcome introduces new information (e.g., implement surfaces PG-14 boundary change), intake re-evaluates: is the original pipeline still valid? If YES → continue to step N+1. If NO → return to Phase 3 with the new context and re-gate at Phase 4 (with the user's permission — don't silently mutate an approved plan).

⏸ **SKILL-GATE:intake-replan [HARD] — Pipeline replan needed.** Fires when mid-pipeline information invalidates the approved plan (e.g., PG-14 fires and adds new components, or scoper missed a component discovered during implementation). Surface to user: "Original pipeline assumed X. Implementation revealed Y. Replan? (yes / abort / continue-anyway)". User decides.

**Agent team reuse (S3.7).** If sequential steps need the same archetype (e.g., LLD generation for 3 new components → 3 consecutive LLD designer teams), keep the team alive across steps per design-specs S3.7. Intake manages the team handle in its own state.

**Parallel steps — LLD dispatch for multi-component features.** When the approved pipeline has multiple LLD steps (e.g., `feature-on-existing` touches 3 new components), intake dispatches them as a worker-pool per design-specs S3.1:

1. Intake spawns a `monke-intake-lld-pool` team with N workers (one per component).
2. Each worker dispatches `/monke-design:lld <component>` as its own sub-skill.
3. Intake is the sole writer of `monke-status.md` — workers write per-component LLD files, intake reconciles the `## Features` row progress.
4. PG-9 (LLD design converged) gates fire PER worker; intake relays each gate to the user in order of component dependency (upstream first per HLD S3 ordering).
5. When all LLD workers complete, intake kills the pool (`SendMessage shutdown_request` + `TeamDelete`) and proceeds to the next pipeline step (typically `test:test-plan`, then `implement`).

If the pipeline has only ONE new component → skip the pool, dispatch sequentially as a single step.

This archetype applies to any parallel-safe step group (LLD is the common case; independent `implement:fill` sweeps qualify too). Never parallelize steps with boundary shifts between them — if worker A's output feeds worker B, serialize.

---

## Phase 6: Close the Feature Thread

After the last pipeline step completes:

1. **Verify deliverable.** Read the final artifacts against the proposal's expected list. Any missing → surface `SKILL-GATE:intake-incomplete [HARD]` with the missing set.

2. **Summarize.** Write a closing block to the feature thread file:
   ```markdown
   ## Closing Summary
   Completed: <ISO-timestamp>
   Total human touchpoints: <N>
   Gates hit: <list of gate IDs + outcomes>
   Files touched: <count>
   Components affected: <list>
   Deliverable: <one-sentence what shipped>
   ```

3. **Update monke-status.md.**
   - `## Features` row for this thread: status = `completed`.
   - `Where We Are:` cleared to general project state.
   - Gate Audit Log: append `PG-1-intake CLOSED — <feature-thread-id>`.

4. **Ask the user.**
   ```
   Feature thread <feature-thread-id> complete.
     - Changes are live in the working tree (not committed — per project no-commit default unless explicit).
     - Run `/monke-ops:commit` to stage + commit now, or continue working.
   Anything else? (new intake / done)
   ```

No auto-commit. Ever. User explicitly runs commit or doesn't.

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Skip Phase 4 pipeline approval and start dispatching | Refuse. PG-1 is HARD. Every intake pipeline requires one explicit user approval before workers spawn. |
| Execute a pipeline when classifier confidence is LOW without surfacing alternatives | Refuse. LOW confidence must show 2-3 candidate classes at Phase 4. User picks. |
| Silently mutate an approved pipeline when mid-step information changes | Refuse. Fire `SKILL-GATE:intake-replan` HARD. User approves the new plan or aborts the thread. |
| Write code as the intake leader | Refuse. Intake routes and tracks. Dispatch to `/monke-design:*`, `/monke-implement:*`, `/monke-test:*` as workers. Lead-writes-code is forbidden except for <20-line feature-thread synthesis. |
| Commit changes automatically at Phase 6 | Refuse. Intake never commits. User runs `/monke-ops:commit` explicitly. |
| Spawn a new agent team when an existing one from a prior step is compatible | Refuse. Reuse per design-specs S3.7 — kill and respawn only on archetype change. |
| Re-ask a question already answered in `.monke-config.md` or the request string | Refuse. Read config first. The user named their rigor once; don't ask again. |
| Treat the feature thread file as optional | Refuse. Every intake run writes `monke-docs/intake/<feature-thread-id>.md`. No thread file = no audit trail = this skill is theater. |

---

## Context Death Protocol

Intake runs long — sometimes hours of pipeline. Context pressure WILL hit mid-Phase-5.

**Checkpoint artifacts (written on context pressure):**
- `monke-docs/intake/<feature-thread-id>.md` — always kept current (updated at every step transition + gate outcome).
- `monke-docs/intake/<feature-thread-id>.md.lock` — companion lock-file per drafter §7 format.
- `monke-status.md` — `## Features` row for the active thread + `## Resume` block pointing to intake.

**Status line format (written to `monke-status.md`):**

```
## Resume
Skill: /monke-intake
Phase: <3 | 4 | 5 | 6>
Last step: <step-N-of-M completed>  (Phase 5 only)
Last gate: <gate ID — outcome>
Next action: <what to do on next invocation — e.g. "dispatch step 4: /monke-implement:implement auth-service 2">
Feature thread: <feature-thread-id>
Rigor: <active rigor>
Died at: <ISO-timestamp>
```

**Canonical status line marker (while alive):**

`Where We Are: monke:intake — <phase-descriptor> (thread=<feature-thread-id>, step <N>/<M>)`

Examples:
- `Where We Are: monke:intake — classifier+scoper running (thread=feature-20260418-0911-webhook-retry)`
- `Where We Are: monke:intake — awaiting PG-1 approval (thread=feature-20260418-0911-webhook-retry)`
- `Where We Are: monke:intake — step 3/5 /monke-implement:implement auth-service (thread=feature-20260418-0911-webhook-retry)`

**Resume argument convention (per drafter §7):**

Intake accepts `resume:<N>` as second positional. Example: `/monke-intake "" resume:5` re-enters at Phase 5 using the lock-file + feature thread file to restore state.

**Recovery detection (on entry):**
- If `monke-status.md` Resume block names `/monke-intake` AND thread file exists → read thread file, verify lock-file `skill: monke:intake`, resume at the Next action.
- If thread file exists but lock-file is missing → warn "Thread file present but lock-file missing — last state unverified. Restart from Phase 4 (re-approve plan)." Do NOT silently continue.
- If neither → start fresh from Phase 1.
- After successful recovery → clear the Resume block, bump `Died at:` out of existence, restore normal status marker.

**Lock-file write triggers:**
- End of Phase 2 (team synthesis complete).
- End of Phase 3 (pipeline proposal written).
- End of each Phase 5 step (before dispatch of next).
- Immediately before any SendMessage to workers.
- Immediately before any user-surface gate.

**Lock-file cleanup:** Delete on Phase 6 close.

---

## Status Update

**On entry:**
1. Read `monke-status.md`, `.monke-config.md`, any `## Resume` block.
2. Verify Agent Teams Gate + monke-docs + config existence.
3. If Resume block names intake → recover per Context Death Protocol.

**After Phase 4 approval:**
1. Create `## Features` section in `monke-status.md` if missing. Add row:
   ```
   | Thread ID | Request (40 char) | Class | Rigor | Started | Status |
   ```
2. Append `PG-1 APPROVED` to Gate Audit Log.
3. Bump `Updated:` line with current date and `by /monke-intake`.

**After every Phase 5 step:**
1. Update the `## Features` row for this thread: `Progress: step <N>/<M>`, `Last step: <skill>`, `Last outcome: <pass|fail|escalated>`.
2. Append sub-skill's gate outcomes to Gate Audit Log (if they wrote to the log).
3. Update `Where We Are:` to the canonical marker above.
4. Re-write the thread file with the new step entry.

**On exit (Phase 6 success):**
1. `## Features` row: `Status: completed`, `Finished: <date>`.
2. Write closing summary to the thread file.
3. Gate Audit Log: `PG-1-intake CLOSED — <feature-thread-id>`.
4. Clear Resume block.

**On exit (blocked — e.g. user chose abort at Phase 4 or SKILL-GATE:intake-replan):**
1. `## Features` row: `Status: aborted` or `paused`.
2. Add blocker row to Open Blockers if applicable.
3. Suggest next action (`/monke-design:oq triage` for unresolved ambiguity, or re-intake with narrower request).

**On exit (partial — context death):**
See Context Death Protocol above. Never exit without the checkpoint.

---

*intake is the one leader. everything else is a worker. no cascade, no hand-off, no getting lost in the jungle.*
