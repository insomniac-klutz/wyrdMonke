# WyrdMonke Test:TestRun — Execute Tests & Verify Gates

> **Usage:** Copy `monke-test/` to `~/.claude/commands/monke-test/`. Invoke: `/monke-test:test-run <tier> [scope]`

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

## Phase 3: Verify Gate

After tests pass, verify the corresponding gate criteria:

### Unit (IL-2)

Per `implementation-specs.md` S4:
- [ ] All unit tests passing (just verified in Phase 2)
- [ ] Linter clean on modified files — run linter command from project-specs S8
- [ ] Pure functions have no IO — verify by checking that functions marked `pure` in LLD do not import IO modules or call IO functions

If all checks pass → **IL-2 PASSED** for scope.

### Integration (IL-3)

Per `implementation-specs.md` S4:
- [ ] Full test suite green (unit + integration) — run both tiers
- [ ] Coverage meets threshold — run coverage tool from project-specs S9, compare against threshold
- [ ] LLD test checklist updated with commit references

If all checks pass → **IL-3 PASSED** for scope.

### System (PG-11)

Per `design-specs.md` S11.1:
- [ ] All HLD S4 data flows exercised end-to-end
- [ ] Error injection scenarios pass (at least one failure scenario per flow)
- [ ] If agentic: full decision loop with happy and adversarial inputs tested
- [ ] All phase component gates (IL-3) already passed

If all checks pass → **PG-11 criteria met** for scope. Note: PG-11 is a pause gate — the user must still sign off via `/monke-implement:checkpoint <phase>`.

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

### On Failure

Categorize failures by tier per `design-specs.md` S11.3 Gate Failure Protocol:

**Unit failure:**
- Is the implementation wrong? → fix the function body
- Is the LLD design wrong? → record in `monke-docs/open-questions.md`, surface to user. Do NOT silently fix the design.
- Present: which function failed, what was expected vs actual, suggested fix category

**Integration failure:**
- Find which side of the boundary is wrong (upstream or downstream)
- Is the contract wrong? → update HLD boundary matrix + both LLDs
- Is one side's implementation wrong? → fix that side
- Present: which boundary failed, which side appears wrong, suggested fix category

**System failure:**
- Trace failure to the specific boundary in the data flow
- Apply integration failure protocol to that boundary
- Present: which flow failed, at which step, trace to boundary

### Failure Cycle Tracking

Track how many fix-and-rerun cycles have occurred for this tier/scope combination:
- Cycle 1-2: normal. Present failure analysis, suggest fix.
- Cycle >2: **⏸ PG-13 — Escalate to user.**

```
⏸ PAUSE GATE: Test failure >2 cycles for <tier>/<scope>
Failures persist after <N> fix attempts.
Pattern: <what keeps failing and why>
Options:
  1. Continue fixing — <what to try next>
  2. Redesign — revisit LLD for <component> via /monke-design:lld
  3. Escalate — park as OQ, move to different component
Confirm which approach?
```

### Anti-Pattern Enforcement

**NEVER weaken a test to pass a gate** (`test-specs.md` S9). If asked to:
- Refuse explicitly
- Cite: "Weakening tests to pass gates is a refused anti-pattern per test-specs S9. Fix the code or fix the design."
- Suggest: fix the implementation, or if the design is wrong, record an OQ and revisit the LLD

---

## Status Update

On completion, update `monke-status.md`:

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
