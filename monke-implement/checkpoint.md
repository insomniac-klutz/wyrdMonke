# WyrdMonke Implement:Checkpoint — The Toll Booth

> **Usage:** `/monke-implement:checkpoint <phase>`
>
> Collects every component in a phase, runs system tests against the enabled data flows, and stops the project dead at PG-11 until the human signs off that the phase is real.

---

## Arguments

`$ARGUMENTS` parsing:
- Single positional: phase number from HLD S8 (**required**)
- If missing → read HLD S8, list phases with their component status, ask user to pick
- If phase not found in HLD S8 → show available phases, ask user to pick

```
PHASE="${ARGUMENTS:?Phase number required}"
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

Read `monke-status.md`. Then verify:

- `monke-docs/hld.md` S8 defines this phase with its component list
- All components in this phase have reached IL-3 (check Implementation table in `monke-status.md`)
  - If any component is not at IL-3 → list incomplete components and their current layer. Tell user: "Run `/monke-implement:implement <component> [layer]` to finish." Stop.
- **LLD finalized.** For every component in this phase, verify the component's LLD is in its final state: `monke-docs/lld/<component>.md` exists AND no `-draft.md` or `-review.md` files for the same component remain in `monke-docs/lld/`. If drafts exist, direct user to finalize the LLD first via `/monke-design:lld <component>` (or if the LLD design session died, run with `resume:<N>`). Stop.
- If phase > 1: prior phase checkpoint must be PASSED (check Phase Checkpoints table)
  - If prior phase not passed → tell user: "Phase N-1 checkpoint not passed. Run `/monke-implement:checkpoint <N-1>` first." Stop.

If prerequisites fail → explain what's missing and which skill to run first.

---

## Phase 1: Verify Component Gates

For each component listed in HLD S8 for this phase:

1. Read the component's LLD: `monke-docs/lld/<component>.md`
2. Verify the test implementation checklist is complete:
   - `[x] Unit tests written + passing (commit: <hash>)`
   - `[x] Integration tests written + passing (commit: <hash>)`
   - `[x] LLD test gate: PASSED — <date>`
3. If any checklist item is incomplete → report which component and which item. Stop.
4. If integration tests were deferred (neighbor not ready at IL-3 time), verify they have since been written and are passing now that the neighbor exists.

Present a summary table:

```
| Component | Unit | Integration | Coverage | IL-3 |
|-----------|------|-------------|----------|------|
```

If any component fails verification → stop, list failures, suggest remediation.

---

## Inlined Gate Failure Protocol (from design-specs S11.3)

Spec file `monke-docs/design-specs.md` remains source of truth. This checkpoint skill follows:

**Phase Definition (HLD S8):**
```
## Phase N: <name>
Components: [LLD components]
Data flows enabled: [which S4 flows become testable]
Depends on: Phase N-1 passed
System test criteria: [end-to-end proof]
```

**Gate Failure Protocol:**
1. **Unit failure:** Fix implementation or LLD. Boundary affected → update HLD + ADR.
2. **Integration failure:** Find wrong side. Contract wrong → update HLD matrix + both LLDs.
3. **System failure:** Trace to boundary. Read the stack trace or assertion failure, identify which boundary the data crossed when it broke, find that boundary in HLD S7, then locate the two components on either side. Run the integration test for that boundary — if it passes, the bug is in end-to-end wiring; if it fails, the bug is in the component that owns the contract.
4. **Persistent (>2 cycles):** ⏸ PG-13 (TRIGGERED). Escalate to user. Likely LATS backtrack.

Claude MUST NOT weaken a test to pass a gate.

---

## Phase 2: System Tests

System tests verify end-to-end HLD data flows (S4) that become testable in this phase.

### Steps

1. Read HLD S8 for this phase — identify which S4 data flows are enabled.
2. Read HLD S4 — get the full flow definitions for those data flows.
3. For each enabled data flow:
   - Write a system test exercising the flow end-to-end:
     - All internal components REAL
     - External third-party systems mocked or sandboxed
     - At least one error injection scenario per flow
     - Agentic: full decision loop with happy and adversarial inputs
   - Place in system test directory per `project-specs.md` S9 (per-container if polyglot).
4. Run the full test suite: unit + integration + system.
5. Verify all pass.

If system test fails → apply the **Gate Failure Protocol** above.
If persistent (>2 cycles) → **PG-13 (TRIGGERED)** fires.

### Commit

```
Add phase <N> system tests
```

---

## Phase 3: Checkpoint Record

Create the checkpoint record in the format below (inlined from design-specs S11.3).

Write to `monke-docs/checkpoints/phase-<N>-checkpoint.md`:

```markdown
# Phase <N> Checkpoint: <phase name from HLD S8>
Date: <today>  |  HLD Version: <from hld.md header>

## Component: <name> (LLD: monke-docs/lld/<component>.md — final, not -draft or -review)
- [x] Unit tests passing (commit: <hash>)
- [x] Integration tests passing (commit: <hash>)
- [x] LLD gate: PASSED

<repeat for each component in this phase>

## System Tests
- [x] Defined for flows: <list of S4 flows enabled in this phase>
- [x] Passing (commit: <hash>)

## Phase Gate
- [ ] ALL component gates PASSED
- [ ] ALL system tests PASSED
- [ ] HLD updated if needed + ADRs written
- [ ] OQs resolved or deferred with justification
- [ ] USER SIGN-OFF (PG-11)

Status: PENDING
```

Fill in the component sections from verified data. Check off gate items that are confirmed. Leave USER SIGN-OFF unchecked.

The LLD path must point to the FINAL `<component>.md` file, not the team-working drafts (`<component>-draft.md`, `<component>-review.md`). If those drafts still exist, the LLD is not yet finalized and this checkpoint should not be recorded.

---

## PG-11: Phase Sign-Off (HARD)

**PG-11 is HARD and NEVER skippable** — regardless of rigor level. It always surfaces to the user.

Present the full checkpoint record to the user:

```
⏸ PG-11 [HARD] — Phase checkpoint sign-off. Phase <N> — <phase name>

Components verified: <count>/<total>
System tests: <pass count>/<total>
Coverage: <percentage>

<full checkpoint record>

Confirm / Adjust / Reject?
```

Always surfaces. Never auto-passes regardless of rigor (HARD gate).

- **Confirm** → proceed to Phase 4 (Finalize)
- **Adjust** → user specifies changes, re-verify affected items
- **Reject** → record rejection in the checkpoint file (`monke-docs/checkpoints/phase-<N>-checkpoint.md`, set Status: REJECTED, append rejection reason). Update `monke-status.md` Phase Checkpoints table to BLOCKED. Recovery: user must fix the cited issues, re-run the failing component through `/monke-implement:implement <component> <layer>`, then re-invoke `/monke-implement:checkpoint <N>` to re-verify from Phase 1.

Do NOT proceed without explicit user confirmation. Do NOT interpret silence as confirmation.

---

## Phase 4: Finalize

After PG-11 confirmation:

1. Update the checkpoint record:
   - Check off `[x] USER SIGN-OFF (PG-11)`
   - Set `Status: PASSED`

2. Determine next steps:
   - If more phases remain in HLD S8:
     - Identify next phase's components
     - Check which have LLDs (ready for implementation) vs which need LLD creation
     - "Next phase components need LLDs: `/monke-design:lld <component>`"
     - Or if LLDs exist: "Start next phase: `/monke-implement:implement <component>`"
   - If this was the final phase:
     - "All phases complete! Full system is implemented and verified."
     - "Consider: final review, deployment prep, documentation."

---

## Context Death Protocol

**Checkpoint artifacts:** `monke-docs/checkpoints/phase-<N>-checkpoint.md` written incrementally (Status: PENDING, PASSED, or REJECTED), commit hashes of new system tests.
**Status line marker:** `Where We Are: implement:checkpoint — phase <N> (<verifying | system tests | PG-11 waiting>)` while mid-flight.
**Recovery detection:** On re-entry, if `checkpoints/phase-<N>-checkpoint.md` exists with Status: PENDING → read it, jump to PG-11 (Phase 4) if the component verifications and system-test commits are present. If Status: REJECTED → surface rejection reason to user, do not re-run automatically — wait for explicit user instruction. If no checkpoint file → start from Phase 1 (Verify Component Gates).

---

## Status Update

**Read on entry:** `monke-status.md` — verify every component in this phase has reached IL-3, check Phase Checkpoints table for prior-phase PASSED status.
**Write on exit**, update `monke-status.md`:
- **Phase Checkpoints table:** Update this phase's row:
  - All IL-3: checked
  - System Tests: pass/fail
  - PG-11: confirmed date or BLOCKED
  - Status: PASSED or BLOCKED with reason
- If PASSED, recalculate "Where We Are" to point to next phase
- If next phase has components without LLDs → "Next action" points to `/monke-design:lld <component>`
- If next phase components have LLDs → "Next action" points to `/monke-implement:implement <component>`
- If all phases complete → "Where We Are" = "All phases complete"
- Bump `Updated:` line with current date and `/monke-implement:checkpoint`

---

## Gate Classifications Used Here

| Gate | Type | Notes |
|------|------|-------|
| PG-11 | HARD | Phase sign-off. Always surfaces. Never skippable regardless of rigor. |
| PG-13 | TRIGGERED | Persistent failure (>2 cycles) during system tests. |
| PG-14 | TRIGGERED | HLD amendment if checkpoint reveals boundary drift. |

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Skip PG-11 user sign-off | Refuse. PG-11 is HARD — NEVER skippable. Present, pause, wait. |
| Run checkpoint with deferred integration tests still outstanding | Refuse. Verify deferred tests were written and passing first. |
| Allow architectural drift through checkpoint | Refuse. If boundary contracts changed, fire PG-14 before checkpoint. |
| Interpret silence as confirmation | Refuse. Explicit "confirm" required. |
| Weaken system tests to pass the gate | Refuse. Fix the code or fix the design. |
