# WyrdMonke Implement:Checkpoint — Phase Checkpoint Verification

> **Usage:** Copy `monke-implement/` to `~/.claude/commands/monke-implement/`. Invoke: `/monke-implement:checkpoint <phase>`

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

## Phase 2: System Tests

> Reference: `design-specs.md` S11.1 System Tests, `test-specs.md` S2 System Tests

System tests verify end-to-end HLD data flows (S4) that become testable in this phase.

### Steps

1. Read HLD S8 for this phase — identify which S4 data flows are enabled.
2. Read HLD S4 — get the full flow definitions for those data flows.
3. For each enabled data flow:
   - Write a system test exercising the flow end-to-end per `test-specs.md` S2 System Tests:
     - All internal components REAL
     - External third-party systems mocked or sandboxed
     - At least one error injection scenario per flow
     - Agentic: full decision loop with happy and adversarial inputs
   - Place in system test directory per `project-specs.md` S9
4. Run the full test suite: unit + integration + system.
5. Verify all pass.

If system test fails:
- **Trace procedure** (per `design-specs.md` S11.3): read the stack trace or assertion failure, identify which boundary the data crossed when it broke, find that boundary in HLD S7, then locate the two components on either side. Run the integration test for that boundary — if it passes, the bug is in end-to-end wiring; if it fails, the bug is in the component that owns the contract.
- If integration-level issue → fix at component level, re-run
- If persistent (>2 cycles) → **stop and surface to user per PG-13**

### Commit

```
Add phase <N> system tests
```

---

## Phase 3: Checkpoint Record

Create the checkpoint record per `design-specs.md` S11.3 format.

Write to `monke-docs/checkpoints/phase-<N>-checkpoint.md`:

```markdown
# Phase <N> Checkpoint: <phase name from HLD S8>
Date: <today>  |  HLD Version: <from hld.md header>

## Component: <name> (LLD: lld/<component>.md)
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

---

## PG-11: Phase Sign-Off

**This gate is NEVER skippable.** Per `design-specs.md` S9.4, PG-11 requires explicit user confirmation.

Present the full checkpoint record to the user:

```
PAUSE GATE PG-11: Phase <N> Checkpoint — <phase name>

Components verified: <count>/<total>
System tests: <pass count>/<total>
Coverage: <percentage>

<full checkpoint record>

Confirm / Adjust / Reject?
```

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

## Status Update

On completion, update `monke-status.md`:
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

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Skip PG-11 user sign-off | Refuse. PG-11 is NEVER skippable. Present, pause, wait. |
| Run checkpoint with deferred integration tests still outstanding | Refuse. Verify deferred tests were written and passing first. |
| Allow architectural drift through checkpoint | Refuse. If boundary contracts changed, fire PG-14 before checkpoint. |
| Interpret silence as confirmation | Refuse. Explicit "confirm" required. |
| Weaken system tests to pass the gate | Refuse. Fix the code or fix the design. |
