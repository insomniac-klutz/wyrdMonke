# WyrdMonke Design:LLD — Bones of the Beast

> **Usage:** `/monke-design:lld <component> [stub]`
>
> Cracks one HLD component open along its import graph, tables typed signatures and error paths, then drafts the unit + integration test plan until the skeleton is buildable from memory alone.

---

## Arguments

`$ARGUMENTS` parsing:
- First positional: component name from HLD S3 (**required**)
- Second positional: `stub` — generate a minimal stub LLD from discovered code (invoked by `/monke` fast onboarding), OR `resume:<N>` — skip to Phase `<N>` with checkpoint re-read (see drafter §7)
- If component name missing → read HLD S3, list available components with their phase and status, ask user to pick
- If component not found in HLD S3 → show available components, ask user to pick

```
COMPONENT="${ARGUMENTS%%[[:space:]]*}"
MODE="${ARGUMENTS#* }"   # "stub", "resume:<N>", or empty
RESUME_PHASE="$(echo "$MODE" | grep -oE '^resume:[0-9]+$' | cut -d: -f2)"
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

Read `monke-status.md`. Then verify:

- `monke-docs/hld.md` has confirmed L3 (S3 component map exists with this component), OR is an auto-generated HLD (v0.x) with this component listed.
- No blocking OQs for this component (check `open-questions.md` for `Blocks: LLD for <component>`).
- `monke-docs/lld/<component>.md` does NOT already exist. If it does:
  - **Recon-origin** (header: `Source: reverse-engineered`): bridge from recon to implementation. Ask: "Reconstructed LLD exists. Review it to add a test plan (PG-10) and confirm design (PG-9)?" In review mode: read the existing LLD, verify/refine the design, then generate the test plan per Phase 4. Do NOT rewrite the internal design — preserve reconstruction, add what's missing. **Both PG-9 and PG-10 fire normally in review mode** — the gates are non-negotiable regardless of LLD origin. After PG-10 confirms, update LLD `Confidence:` from `auto-generated` to `reviewed` and status from `"reconstructed"` to `"ready"` in `monke-status.md`.
  - **Stub LLD from fast onboarding** (header: `Confidence: auto-generated`): ask: "Stub LLD exists from fast onboarding. Review-and-refine, or run full LLD from scratch?" Review-and-refine is the expected path — fill in what scan couldn't infer.
  - **Greenfield LLD exists**: ask: "LLD exists. Review it, or start fresh?"
- Component's phase dependencies are met (all prior-phase components have LLDs).

If `MODE == "stub"` → jump to [Stub LLD Flow](#stub-lld-flow).

If prerequisites fail → explain what's missing and which skill to run first.

---

## Agent Teams Check

Same detection as `/monke-design:hld`. If available → Team mode (Designer+Reviewer). If not → Solo mode.

---

## Inlined Design Paradigm (from design-specs S1.5, S3.4, S6)

Spec file `monke-docs/design-specs.md` remains source of truth for maintenance — do not re-load it at runtime.

### ADaPT — Recursive Decomposition (S1.5)

1. **DECOMPOSE** into sub-problems.
2. **PAUSE** — present decomposition to user (PG-8).
3. **ATTEMPT** each fully.
4. **ASSESS** against boundary contract.
5. **ADAPT** — decompose further / restructure / escalate to user.
6. **INTEGRATE** and verify end-to-end.

**Triggers:** Every LLD (mandatory), >3 responsibilities in one unit, >3 error paths, "handles X, Y, Z, and also W" signal. For agentic components: decompose each CoALA dimension separately, then integrate.

Surface the decomposition tree visibly. User sees reasoning, not just conclusion.

### LLD Required Sections (S6.3)

**Header (with Confidence field — H10):**

    Parent HLD: S3.<section>  |  HLD Version: X.Y
    Confidence: auto-generated | reviewed | verified
    Backend: per project-specs S10.2  |  ADR: NNN (if non-default)
    Type: traditional/agentic  |  Pattern: <name> (agentic)  |  ADR: NNN
    Upstream: <Model> from <module>  |  Downstream: <Model> to <module>
    Errors: <what crosses boundary>
    Tool subtypes: <static-contract | versioned-artifact | data-dependent> (if non-default)
    Version pins: <artifact: version> (if versioned-artifact)
    Eval thresholds: <metric: threshold> (if versioned-artifact or data-dependent)
    Phase: <N>  |  Test gate: pending

**Confidence ladder:**
- `auto-generated` — from fast-onboarding scan or recon reconstruction. NOT a trusted input for neighbor LLDs.
- `reviewed` — user confirmed design (PG-9) and test plan (PG-10). Trusted input for neighbor LLDs.
- `verified` — all IL gates (0–3) passed on this component. Fully trusted.

**Internal Design:**
- Full-typed signatures; pure functions separated from IO functions.
- State machines + transition tables (if applicable).
- Algorithms + complexity + edge cases.
- Expected errors as return types, unexpected as exceptions.

**For agentic, additionally:**
- Memory schemas + storage + retrieval + eviction + token budget.
- Tool schemas + sandboxing + retry + action boundaries.
- Decision loop + stopping condition + max iterations + HITL.
- Prompt templates + variable injection + output parsing.

**File Map:**

| File | Layer | Exports | Depends On |
|------|-------|---------|------------|

**Unit Test Plan (mandatory):**

| # | Function/Method | Input | Expected | Category | Mocks |
|---|----------------|-------|----------|----------|-------|

Minimum: 1 happy + 1 edge + 1 error per public function. Pure functions need no mocks. Agentic: loop termination, tool failure, stale memory, context overflow.

**Integration Test Plan (mandatory):**

| # | Boundary | Upstream Call | Expected Downstream Effect | Error Scenario | Mocks |
|---|----------|-------------|---------------------------|----------------|-------|

Every boundary: upstream consumption, downstream production, error propagation. Cross-language: serialization round-trip. Agentic: memory consistency, tool contracts, agent-to-agent messages.

**Eval Test Plan (if versioned-artifact or data-dependent tools):**

| # | Metric | Threshold | Test Dataset | Artifact Version | Gate |
|---|--------|-----------|-------------|-----------------|------|

Eval tests run within IL-2 (mocked artifact, fixture data) and IL-3 (real artifact, test dataset). No separate gate.

**Review Log:**

| Round | Violation | Resolution |
|-------|-----------|------------|

**Test Implementation Checklist:**

    - [ ] Unit tests written + passing (commit: <hash>)
    - [ ] Integration tests written + passing (commit: <hash>)
    - [ ] Eval tests written + passing (commit: <hash>) (if applicable)
    - [ ] LLD test gate: PASSED — date

### Banned From LLD (S6.4)

Restating HLD. Aspirational prose. Decisions without rationale. Violating locked stack. Over-engineered patterns (<3 dynamic decisions → de-escalate). Skipping test plans.

---

## Phase 1: Context Gathering

**Resume check (per drafter §7).** If arguments contain `resume:<N>`:
1. Read the companion lock-file `monke-docs/lld/<component>-draft.md.lock`. Lock-file is authoritative — if it's inconsistent with the draft, regenerate from scratch.
2. Verify lock-file `skill:` field matches `design:lld` and the draft artifact exists and parses.
3. If parse/lock check passes → skip to Phase `<N>` with prerequisites re-validated inline.
4. If parse/lock check fails → emit warning "resume:<N> specified but checkpoint invalid" and proceed from Phase 1 normally.
5. See Context Death Protocol below for the full recovery spec.

1. Read the component's entry in HLD S3:
   - Responsibility, boundary contracts, dependency direction, type tag.
   - For agentic: CoALA summary.
2. Read upstream/downstream components' boundary contracts from HLD S7.
   - If a neighbor's LLD has `Confidence: auto-generated`, its boundary is **not trusted** — treat its contract as a hypothesis that this component's design can falsify. If falsified → fire PG-14 (HLD amendment) rather than silently diverging.
3. Read `project-specs.md` S10 (locked stack) + S2 (stack bindings).
4. Read `project-specs.md` S9 (test bindings).
5. **H3 — Read prior attack notes.** If HLD was generated by an agent team (check `monke-status.md` Gate Audit Log or HLD header for `Mode: team` / `Mode: solo`) AND `monke-docs/critic-notes.md` exists: read it. Your design must address every unresolved Critic attack or note which ones are out of scope for this component. Attacks identified during design are inputs to LLD, not disposable artifacts. If HLD was generated in solo mode, `critic-notes.md` is not expected — skip this read silently.
6. If component is `agentic` or has versioned-artifact / data-dependent tool subtypes — check for seer artifacts:
   - `monke-docs/recon/profile-*.md` — data profiles with distributions, schema, drift surface.
   - `monke-docs/decisions/*-experiment-*.md` — model/approach comparison ADRs with metric evidence.
   - HLD S7 `versioned(<artifact>, <pin>)` rows + existing LLD headers `Version pins:` — registered pins.
   - `monke-docs/decisions/*-agentify.md` — agentic vs non-agentic verdict ADR.
   - If found, read them. They inform CoALA dimensions, version pins, eval thresholds, and tool subtype classification.
   - **Conflict check:** If a `seer:agentify` verdict contradicts the HLD S3 type tag:
     - ⏸ **SKILL-GATE:seer-conflict [TRIGGERED] — Seer/HLD type-tag disagreement.**
       Fires only when both artifacts exist and disagree. Present the conflict: "Seer verdict says `<X>`, HLD says `<Y>`. Evidence: `<ADR reference>`."
       Options: (A) Trust HLD — note seer conflict as OQ. (B) Trust Seer — amend HLD type tag via `/monke-design:hld evolve repattern`. (C) Defer.
       **User decides before decomposition begins.** Ignores rigor level (TRIGGERED fires on event).

Present context summary to user before proceeding.

---

## Phase 2: ADaPT Decomposition

Execute ADaPT per the inlined S1.5 above:

1. **DECOMPOSE** into sub-problems:
   - What are the distinct responsibilities?
   - For agentic: decompose each CoALA dimension separately.
   - **For components with versioned-artifact or data-dependent tools:** separate pure pre/post-processing (feature extraction, input validation, output parsing, metric computation) from artifact interaction (model inference, store query, artifact loading). If the component owns the artifact's lifecycle (training/rebuild), decompose that as a sub-problem with its own Layer 0-3 cycle — not as a separate system.
   - Surface the decomposition tree visibly.

2. ⏸ **PG-8 [SOFT] — ADaPT decomposition confirmed.** Present decomposition to user. Confirm before sub-problems are attempted.
   Auto-pass when: decomposition has <5 sub-problems AND no ambiguous splits AND every sub-problem has a visible boundary to either upstream or downstream.
   Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds.

   **Write lock-file.** After PG-8 passes, update `monke-docs/lld/<component>-draft.md.lock` per drafter §7 — capture `phase: 2`, progress, timestamp.

3. For each sub-problem: **ATTEMPT** fully, **ASSESS** against boundary contract.

4. If >3 responsibilities in one unit, >3 error paths, or "handles X, Y, Z, and also W" → decompose further.

5. **ADAPT** if needed (restructure, decompose further, escalate to user).

6. **INTEGRATE** — verify end-to-end against upstream/downstream contracts.

---

## Phase 3: Design

### Team Mode

Spawn the LLD Team with this kickoff prompt (inlined from design-specs S3.4):

```
Create an agent team to design the LLD for <component>.
Two teammates:

Teammate 1 — Designer:
Your job: design internals for <component> following HLD contracts in
monke-docs/hld.md S3.<section>. Execute ADaPT decomposition (DECOMPOSE
→ ATTEMPT → ASSESS → ADAPT → INTEGRATE). For agentic components,
decompose along CoALA dimensions (Memory, Action Space, Decision
Procedure).

**Read monke-docs/critic-notes.md if it exists. Your design must
address every unresolved Critic attack or explicitly note which ones
are out of scope for this component.** Attacks identified during HLD
design are inputs to LLD, not disposable artifacts.

Write design to monke-docs/lld/<component>-draft.md with:
- Header including Confidence: reviewed (once PG-9 confirms)
- Internal Design (signatures, pure vs IO, state machines, errors)
- File Map
- For agentic: Memory schemas, Tool schemas, Decision loop, Prompt templates

Message Reviewer when design is ready. Use locked LLM interface and
database per project-specs S10.1.
You own: monke-docs/lld/<component>-draft.md

Teammate 2 — Reviewer:
Your job: verify Designer's output against HLD boundary contracts exactly.
Check: (a) upstream/downstream contract match, (b) error handling
complete, (c) tech stack compliance, (d) for agentic: CoALA completeness
+ stopping condition + action boundaries, (e) that every Critic attack
from monke-docs/critic-notes.md is addressed or explicitly out-of-scope.
Write violation log to monke-docs/lld/<component>-review.md.
Flag violations — do NOT fix them. Send back to Designer via message.
Max 3 rounds, then escalate to lead.
You own: monke-docs/lld/<component>-review.md
```

The lead performs the Test Engineer role: writes unit + integration test plan after Designer and Reviewer converge (Phase 4 below). For complex components (>5 public interfaces), spawn a third teammate as Test Engineer.

**Flow:**
1. Designer runs ADaPT → Lead presents decomposition → ⏸ PG-8.
2. User confirms → Designer writes design → messages Reviewer.
3. Reviewer checks → messages violations back to Designer (peer-to-peer, up to 3 rounds).
4. Design converges → Lead reviews final design → ⏸ PG-9.
5. Lead writes test plan → ⏸ PG-10.
6. Lead assembles LLD, writes ADRs, updates HLD if needed, **shuts down team**.

### Solo Mode

Write design directly, self-reviewing against HLD contracts after each section. Still read `critic-notes.md` and address every attack.

### Output

Generate the Internal Design + File Map + Review Log per the Required Sections above.

⏸ **PG-9 [SOFT] — LLD design converged.** Present converged design. Confirm / Adjust / Reject?
Auto-pass when: Reviewer found 0 violations AND all contracts match HLD boundary matrix AND every Critic attack is addressed or explicitly scoped out.
Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds. **Recon-origin LLDs fire PG-9 normally** — reconstruction does not bypass the gate; the class is fixed in design-specs S9.4.

**Write lock-file.** After PG-9 passes, update `monke-docs/lld/<component>-draft.md.lock` per drafter §7 — capture `phase: 3`, progress, timestamp.

---

## Phase 4: Test Plan

After design is confirmed, generate the test plan.

**Ownership:** `/monke-design:lld` generates the test plan inline as part of the LLD finalization flow. `/monke-test:test-plan` is the standalone equivalent for cases where the test plan needs to be created or revised separately (e.g., recon-origin LLDs that were reconstructed without test plans, or stub LLDs whose scan-inferred test skeleton needs filling). Both produce the same output — test tables inside the LLD file. They are not meant to run sequentially on the same component.

**Unit Test Plan:**

| # | Function/Method | Input | Expected | Category | Mocks |
|---|----------------|-------|----------|----------|-------|

Minimum: 1 happy + 1 edge + 1 error per public function. Pure functions need no mocks.

**Integration Test Plan:**

| # | Boundary | Upstream Call | Expected Downstream Effect | Error Scenario | Mocks |
|---|----------|-------------|---------------------------|----------------|-------|

Every boundary contract: upstream, downstream, error propagation.

**Eval Test Plan (if versioned-artifact or data-dependent tools):**

| # | Metric | Threshold | Test Dataset | Artifact Version | Gate |
|---|--------|-----------|-------------|-----------------|------|

Eval tests run within IL-2 (mocked artifact, fixture data) and IL-3 (real artifact, test dataset). No separate gate.

**Test Implementation Checklist:**
```
- [ ] Unit tests written + passing (commit: <hash>)
- [ ] Integration tests written + passing (commit: <hash>)
- [ ] Eval tests written + passing (commit: <hash>) (if applicable)
- [ ] LLD test gate: PASSED — date
```

⏸ **PG-10 [SOFT] — Test plan confirmed.** Present test plan. Confirm / Adjust / Reject?
Auto-pass when: every public function has ≥1 happy + 1 edge + 1 error test AND every boundary in the HLD matrix involving this component has at least one integration test row.
Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds.

**Write lock-file.** After PG-10 passes, update `monke-docs/lld/<component>-draft.md.lock` per drafter §7 — capture `phase: 4`, progress, timestamp.

---

## Phase 5: Finalize

1. Write confirmed LLD to `monke-docs/lld/<component>.md`. Set `Confidence: reviewed`.
2. Write ADRs for non-obvious decisions to `monke-docs/decisions/`.
3. If boundary contracts changed during design → **HLD amendment required**:
   - Formulate directive: what boundary is wrong, what the LLD needs it to be, why.
   - Tell user: "HLD boundary mismatch discovered. Requesting amendment."
   - Invoke `/monke-design:hld amend "<directive>"` (or execute amend flow inline).
   - ⏸ **PG-14 [TRIGGERED] — HLD amendment.** Fires inside the amend flow when the LLD surfaces a boundary mismatch. User confirms the HLD change. Ignores rigor.
   - After amend completes: re-read updated HLD boundary, verify LLD now aligns.
   - If amend rejected → record as OQ in `open-questions.md`, flag blocker, stop LLD finalization.
4. Copy team artifacts if Agent Teams used:
   - Reviewer violation log → LLD "Review Log" section.
   - Critic attacks (from `critic-notes.md`) → ADR "Challenges Considered" section if any decisions traced back to them.
   - Clean up draft files.

5. **Draft cleanup.** After the final `monke-docs/lld/<component>.md` is written and reviewer confirmed (PG-9 passed or auto-passed), delete the ephemeral team artifacts:
   - `monke-docs/lld/<component>-draft.md`
   - `monke-docs/lld/<component>-review.md`
   - `monke-docs/lld/<component>-draft.md.lock` — **delete lock-file.** No longer needed after finalization; stale lock-files confuse future recovery.

   These are team-working files, not persistent records. Keeping them confuses future resume detection (the recovery system might think a design session is mid-flight). Do not delete if PG-9 failed — they are evidence for the next revision cycle.

6. Suggest next steps:
   - "Implement this component: `/monke-implement:implement <component>`"
   - "Design next component: `/monke-design:lld <next-component>`"
   - "Run `/monke-status:status` to see updated dashboard."

---

## Stub LLD Flow

> Entry: `/monke-design:lld <component> stub` — invoked by `/monke` fast onboarding when a component needs implementation work and must have a minimum artifact to enter the implement pipeline.

**Goal (B1):** Produce the **minimum LLD that satisfies `/monke-implement:implement` prerequisites** — specifically a decomposition tree and a test plan skeleton. Stub LLDs carry `Confidence: auto-generated`.

### S1: Read Discovered Inputs

From the fast-onboarding scan (passed in context) or re-detect:

- File map of the component's actual files.
- Function/method signatures from actual code (AST or symbol extraction — not inferred).
- Maturity tag per function: `as-is`, `needs-work`, `stub`.
- Import graph for this component (who calls whom internally).

### S2: Stub LLD Minimum Contents

A stub LLD MUST contain these sections. Missing any → not a valid stub, must escalate to full `/monke-design:lld <component>`.

**Header:**
```
Parent HLD: S3.<section>  |  HLD Version: 0.X
Confidence: auto-generated
Source: fast-onboarding scan | reverse-engineered
Type: traditional | agentic   # default traditional unless agent framework detected
Upstream: <Model> from <module>    # from detected imports (HIGH) or TBD (LOW)
Downstream: <Model> to <module>    # from detected imports (HIGH) or TBD (LOW)
Phase: <N>                          # from HLD S8 topological order
Test gate: pending
Maturity: <component-level maturity from fast-onboarding scan>
```

**File Map (from actual files):**

| File | Layer | Exports | Depends On | Maturity |
|------|-------|---------|------------|----------|

Layer inferred from file kind (types/models/schemas → 0; handlers/services → 1 or 2 by body presence).

**Signatures (from actual code — verbatim, not inferred):**

Copy exported function/method signatures as-is. Mark pure vs IO by static analysis (functions that call IO modules → IO; rest → pure).

**Maturity Tags per function:**
- `as-is`: body exists, has tests, imports resolve → treat as implemented.
- `needs-work`: body exists, no tests or partial tests → Layer 2 work needed.
- `stub`: placeholder (`NotImplementedError`, `todo!()`, empty function) → full Layer 0→3 needed.

**Mechanical Decomposition Tree (B1 — critical for implement skill):**

Build from the import graph:
- **Leaves:** functions with no internal dependencies (call only stdlib/external).
- **Callers:** functions that call leaves.
- **Roots:** top-level entry points (exported, called by neighbors).

Emit as:
```
Decomposition (mechanical — from import graph):
  - Leaves (implement first):
    - <function>  (maturity: <tag>)
    ...
  - Middle:
    - <function>  (maturity: <tag>)  calls: [<leaf1>, <leaf2>]
    ...
  - Roots (implement last):
    - <function>  (maturity: <tag>)  calls: [<middle1>]
    ...
```

This satisfies `/monke-implement:implement` Layer 2 prerequisite (functions in dependency order, leaves first).

**Test Plan Skeleton (B1 — satisfies PG-10 prerequisite):**

Every public function gets stub rows — enough for the implement skill to expand, not complete tests:

Unit Test Plan (stub):

| # | Function/Method | Input | Expected | Category | Mocks |
|---|----------------|-------|----------|----------|-------|
| 1 | `<name>` | TBD | TBD | happy | TBD |
| 2 | `<name>` | TBD | TBD | edge | TBD |
| 3 | `<name>` | TBD | TBD | error | TBD |

Integration Test Plan (stub) — one row per detected boundary:

| # | Boundary | Upstream Call | Expected Downstream Effect | Error Scenario | Mocks |
|---|----------|-------------|---------------------------|----------------|-------|
| 1 | `<UpstreamModule> -> <DownstreamModule>` | TBD | TBD | TBD | TBD |

Mark each row `Status: TBD — fill during /monke-design:lld review or at PG-10`.

**Test Implementation Checklist** (present, unchecked):
```
- [ ] Unit tests written + passing (commit: <hash>)
- [ ] Integration tests written + passing (commit: <hash>)
- [ ] LLD test gate: PASSED — date
```

### S3: Write Stub LLD

Write to `monke-docs/lld/<component>.md`. Parent `/monke` fast-onboarding gate handles confirmation — no PG-8/9/10 fires here individually.

Remind caller: the stub is **not** ready for implementation as-is if any function is tagged `stub` AND has no integration test rows. The implement skill accepts a stub LLD only when:
- Decomposition tree is present.
- Test plan skeleton has rows for every public function + every boundary.
- Header has `Confidence: auto-generated` (so downstream skills know not to trust it as a neighbor input).

### S4: Return Control

Return to caller (typically `/monke`). No status update — parent orchestrator updates `monke-status.md` with the per-component maturity and stub-LLD flag.

---

## Gate Classifications Used Here

| Gate | Type | Notes |
|------|------|-------|
| PG-8 | SOFT | ADaPT decomposition. Auto-pass: <5 sub-problems, no ambiguous splits. |
| PG-9 | SOFT | LLD design converged. Auto-pass: 0 Reviewer violations, all contracts match HLD boundary matrix exactly. Fires normally even for recon-origin LLDs. |
| PG-10 | SOFT | Test plan. Auto-pass: every public function has happy+edge+error, every boundary has ≥1 integration test. |
| PG-14 | TRIGGERED | HLD amendment. Fires only when LLD surfaces a boundary mismatch. |
| SKILL-GATE:seer-conflict | TRIGGERED | Fires only when seer:agentify verdict contradicts HLD S3 type tag. |

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Skip ADaPT and jump straight to signatures | Refuse. DECOMPOSE → PAUSE → ATTEMPT is the control loop — the user must see the tree before sub-problems are attempted. |
| Treat a neighbor's `Confidence: auto-generated` LLD as a trusted contract | Refuse. Auto-generated contracts are hypotheses; falsification fires PG-14 rather than silent divergence. |
| Fix an HLD boundary silently from inside the LLD | Refuse. Any boundary correction escalates via `/monke-design:hld amend "<directive>"` with PG-14. |
| Auto-confirm PG-9 on a recon-origin LLD because "it's already written" | Refuse. Recon-origin LLDs fire PG-9 + PG-10 normally; gates are non-negotiable regardless of origin. |
| Ignore `critic-notes.md` because the HLD already passed | Refuse. Every unresolved Critic attack is a design input — address it or explicitly scope it out. |
| Write test plan rows with `"foo"` / `"test123"` placeholder data | Refuse. Realistic shapes per test-specs S6. Placeholder data masks contract errors. |

---

## Context Death Protocol

**Checkpoint artifacts:** `monke-docs/lld/<component>-draft.md` (Designer's WIP), `monke-docs/lld/<component>-review.md` (Reviewer's violation log), `monke-docs/lld/<component>-draft.md.lock` (phase + progress ledger per drafter §7), partial test-plan tables in the draft.
**Status line marker:** `Where We Are: design:lld — <component> phase <N> (<decomposition | design | test plan | finalize>)` while mid-flight.
**Recovery detection:** On re-entry, if `<component>-draft.md` exists AND status marker names this skill → read the lock-file (authoritative — if it's inconsistent with the draft, regenerate from scratch) and resume from the phase it records. If draft exists but review file is absent → re-enter Phase 3 (Design). If design converged but `Test Plan` section is empty → resume Phase 4 (Test Plan). If test plan exists but `lld/<component>.md` isn't finalized → resume Phase 5 (Finalize).

---

## Status Update

**Read on entry:** `monke-status.md` — verify this component is in the LLDs table, check blocking OQs, detect recon-origin vs. greenfield vs. stub resume state.

**Write on exit**, update `monke-status.md`:

**Full LLD flow:**
- Update LLDs table: set component row to ADaPT ✓, Design ✓, Review ✓, Test Plan ✓, Status "ready", Confidence `reviewed`.
- Add any new ADRs to Decisions table.
- Add any new OQs to Open Blockers.
- If HLD was updated (PG-14) → note in "Where We Are".
- Bump `Updated:` line.

**Stub flow:**
- Update LLDs table: set component row to Design `stub`, Confidence `auto-generated`, Status "stub".
- Note per-component maturity from fast-onboarding scan.
- Do NOT mark ADaPT or Review as done — those require the full flow.
