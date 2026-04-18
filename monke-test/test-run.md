# WyrdMonke Test:TestRun — Run It, Break It, Prove It

> **Usage:** `/monke-test:test-run <tier> [scope]`
>
> Fires the right test runner at the right scope, reads the verdict against the right gate — IL-2 for unit, IL-3 for integration, PG-11 for system — and refuses to let a failing gate pretend it passed.

---

## Arguments

`$ARGUMENTS` parsing:
- First positional: tier — `unit` | `integration` | `system` (**required**)
- Second positional: scope — component name or boundary name (optional)
  - If tier is `unit` → scope = component name (runs unit tests for that component)
  - If tier is `integration` → scope = component name or boundary name (runs integration tests involving that scope)
  - If tier is `system` → scope = phase number or flow name (runs system tests for that phase/flow)
  - If scope omitted → run ALL tests for the specified tier
- If tier missing → ask, don't guess
- If tier invalid → show options (`unit | integration | system`), ask to pick

```
TIER="${1:?Tier required: unit | integration | system}"
SCOPE="${2:-all}"
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

Read `monke-status.md`. Then verify:

- `monke-docs/project-specs.md` S9 has test runner bindings filled (not `<<<`)
- Tests exist for the specified tier and scope:
  - `unit` → check for test files in `tests/unit/` (or project-convention equivalent) matching scope
  - `integration` → check for test files in `tests/integration/` matching scope
  - `system` → check for test files in `tests/system/` matching scope
- If scope is a component → verify `monke-docs/lld/<component>.md` exists with a test plan

If project-specs S9 not filled → "Test bindings not configured. Fill `project-specs.md` S9 first." Stop.
If no tests found for tier/scope → "No `<tier>` tests found for `<scope>`. Write tests first via `/monke-implement:implement <component>`." Stop.

---

## Phase 1: Resolve Commands

1. Read `monke-docs/project-specs.md`:
   - S9: test runner command, test directory structure, coverage tool
   - S8: IL gate commands (linter, import checker, etc.)

2. Determine the exact commands to run based on tier + scope:

   | Tier | Command Pattern | Gate |
   |------|----------------|------|
   | `unit` | `<test-runner> <test-dir>/unit/<scope-filter>` | IL-2 |
   | `integration` | `<test-runner> <test-dir>/integration/<scope-filter>` | IL-3 |
   | `system` | `<test-runner> <test-dir>/system/<scope-filter>` | PG-11 |

3. If scope is `all` → no scope filter (run entire tier).
4. If scope is a component name → filter to test files matching that component.

Present the resolved commands to the user before executing.

---

## Phase 2: Execute

1. Run the resolved test command.
2. Capture full output — pass and fail results.
3. If the test runner returns non-zero exit code → tests failed. Proceed to Phase 4 (Report) with failure analysis.
4. If the test runner returns zero → tests passed. Proceed to Phase 3 (Verify Gate).

---

## Inlined Gate Criteria (from implementation-specs S4 + design-specs S11.1)

Spec files remain source of truth. Inlined here for self-contained execution.

### IL Gate Table (implementation-specs S4)

| ID | Type | Verification | Proceeds When |
|----|------|-------------|---------------|
| IL-0 | AUTO | Linter clean, imports resolve, models frozen, version pins present (if applicable), migration up/down (if applicable) | Types correct |
| IL-1 | AUTO | Linter clean, downstream can import upstream, pure vs IO separated | Skeleton compilable |
| IL-2 | AUTO | Unit tests pass, eval pass (if applicable), linter clean, pure functions have no IO | Logic correct |
| IL-3 | AUTO | Full suite green (incl. eval), coverage ≥ threshold, LLD checklist updated | → Phase checkpoint |

AUTO gates surface only on failure. Audit line emitted on pass: `[gate:IL-<N>] auto-confirmed (<scope>, <rigor>)`.

### System Test Criteria (design-specs S11.1)

- Each HLD S4 data flow exercised end-to-end.
- Error injection per flow (at least one failure scenario per flow).
- Agentic components: full decision loop with happy and adversarial inputs.
- Isolation: all internal components REAL. External third-party systems mocked or sandboxed.

System test pass contributes to PG-11 (HARD, handled by `/monke-implement:checkpoint`).

---

## Phase 3: Verify Gate

After tests pass, verify the corresponding gate criteria:

### Unit (IL-2 — AUTO)

- [ ] All unit tests passing (verified in Phase 2).
- [ ] Linter clean on modified files — run linter command from project-specs S8.
- [ ] Pure functions have no IO — verify by checking that functions marked `pure` in LLD do not import IO modules or call IO functions.

If all checks pass → **IL-2 auto-confirmed** for scope. Emit audit line.

### Integration (IL-3 — AUTO)

- [ ] Full test suite green (unit + integration) — run both tiers.
- [ ] Coverage meets threshold — run coverage tool from project-specs S9, compare against threshold.
- [ ] LLD test checklist updated with commit references.

If all checks pass → **IL-3 auto-confirmed** for scope. Emit audit line. LLD `Confidence:` may bump to `verified` after all four IL gates have passed (handled by `/monke-implement:implement`).

### System (feeds PG-11)

- [ ] All HLD S4 data flows exercised end-to-end.
- [ ] Error injection scenarios pass (at least one failure scenario per flow).
- [ ] If agentic: full decision loop with happy and adversarial inputs tested.
- [ ] All phase component gates (IL-3) already passed.

If all checks pass → **PG-11 criteria met** for scope. Note: PG-11 is HARD — the user must still sign off via `/monke-implement:checkpoint <phase>`.

---

## Phase 4: Report

### On Success

Present:
```
Gate: <IL-2 | IL-3 | PG-11 criteria>
Tier: <unit | integration | system>
Scope: <component/boundary/phase or "all">
Result: PASSED
Tests: <N passed> / <N total>
```

If IL-3 → also show coverage percentage and threshold.

### On Failure — Gate Failure Protocol (inlined from design-specs S11.3)

Categorize failures by tier:

**Unit failure:**
- Is the implementation wrong? → fix the function body.
- Is the LLD design wrong? → record in `monke-docs/open-questions.md`, surface to user. Do NOT silently fix the design.
- Present: which function failed, what was expected vs actual, suggested fix category (WHAT/WHY/HOW).

**Integration failure:**
- Find which side of the boundary is wrong (upstream or downstream).
- Is the contract wrong? → update HLD boundary matrix + both LLDs (PG-14 TRIGGERED).
- Is one side's implementation wrong? → fix that side.
- Present: which boundary failed, which side appears wrong, suggested fix category.

**System failure:**
- Trace failure to the specific boundary in the data flow (read stack trace, find boundary in HLD S7, run that boundary's integration test).
- Apply integration failure protocol to that boundary.
- Present: which flow failed, at which step, trace to boundary.

Claude MUST NOT weaken a test to pass a gate.

### Failure Cycle Tracking → PG-13 (TRIGGERED)

**Cycle tracking (PG-13).** The Test Gates table in `monke-status.md` has a dedicated `Cycle` column (see `status-template.md`). On each test-run invocation:
1. Read the `Cycle` column for the current scope. Default: 0 if empty or new row.
2. If tests pass: write `Cycle: 0` (reset).
3. If tests fail: increment the column (`Cycle: <prev+1>`), write back.
4. If `Cycle >= 3` (the PG-13 threshold, configurable via `.monke-config.md` `pg13_threshold` if defined): fire PG-13.

⏸ **PG-13 [TRIGGERED] — Persistent test failure.** Fires when `Cycle >= 3` for the same scope without intervening pass. Ignores rigor. Options: (a) Escalate to PG-9 (LLD design issue). (b) User force-retry (reset Cycle). (c) Abort and route to `/monke-rage:buggy` for root-cause investigation.

On re-entry after context death: read the `Cycle` column — it is the canonical state for PG-13 tracking. Do not read any other file.

### Inlined Anti-Patterns to Refuse (from test-specs S9)

| If asked to... | Do instead... |
|----------------|--------------|
| Skip tests, add them later | Refuse. Tests are designed at LLD and implemented with the code. |
| Only unit tests, skip integration | Refuse. Both tiers mandatory per LLD. |
| Mock the database in integration tests | Refuse. Real database instance required. |
| Mock both sides of a boundary in integration | Refuse. Both sides must be real. |
| **Weaken a test to pass a gate** | **Refuse.** Fix the code or fix the design. |
| Share mutable state between tests | Refuse. Transaction rollback per test. |
| Write integration tests with mock placeholders | Refuse. Defer the test until both sides exist. |
| Batch all tests after all code | Refuse. Layer 2 interleaves per function. |
| Weaken eval thresholds to pass a gate | Refuse. Fix the artifact, the data, or the design. |
| Create separate eval test infrastructure | Refuse. Eval tests live in the same test dirs, same runner, same gates. |

If asked: refuse explicitly, cite the inlined rule, suggest fixing the implementation or the design.

### Gate Classifications Used Here

| Gate | Type | Notes |
|------|------|-------|
| IL-2 | AUTO | Unit tier pass criteria. Surfaces only on failure. |
| IL-3 | AUTO | Integration tier pass criteria. Surfaces only on failure. |
| PG-11 | HARD | System tier feeds PG-11 (handled by `/monke-implement:checkpoint`). |
| PG-13 | TRIGGERED | Fires after >2 fix-and-rerun cycles in the same tier/scope. |
| PG-14 | TRIGGERED | Fires if integration failure reveals the contract itself is wrong. |

---

## Context Death Protocol

**Checkpoint artifacts:** captured test runner output (stdout + exit code) held in-memory until report is written; cycle count for PG-13 tracking persisted in the explicit `Cycle` column of the Test Gates table in `monke-status.md`.
**Status line marker:** `Where We Are:` reads `test:test-run — <tier> on <scope> (<executing | analyzing | PG-13 waiting>)` while mid-flight.
**Recovery detection:** On re-entry, read the `Cycle` column of the Test Gates row for the scope in `monke-status.md`: if `Cycle >= 1` and prior run FAILED → continue cycle tracking from that value; if the prior run PASSED (`Cycle: 0`) → tell user the gate already passed and exit unless forced. If `Cycle >= 3` → re-present PG-13 rather than silently re-running.

---

## Status Update

**Read on entry:** `monke-status.md` — check Test Gates row for the scope, load prior cycle count if present.
**Write on exit**, update `monke-status.md`:

### On gate PASSED
- Test Gates table: update the relevant column (`Unit` | `Integration` | `Coverage`) to `passed` for the component
- Implementation table: if IL-2 passed → update L2 status; if IL-3 passed → update L3 status
- If system tier passed → note PG-11 criteria met in Phase Checkpoints table (still needs user sign-off)
- Update "Where We Are" with: "`<tier>` gate passed for `<scope>`"
- Update "Next action":
  - After unit pass → suggest integration tests or next component
  - After integration pass → suggest `/monke-test:coverage <component>` to verify threshold
  - After system pass → suggest `/monke-implement:checkpoint <phase>` for sign-off
- Bump `Updated:` line with current date and `test:test-run`

### On gate FAILED
- Test Gates table: update relevant column to `failed`
- If PG-13 triggered → add to Open Blockers table
- Update "Where We Are" with: "`<tier>` gate failed for `<scope>` — <summary>`"
- Bump `Updated:` line
