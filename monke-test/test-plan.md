# WyrdMonke Test:TestPlan — Generate Test Plan from LLD

> **Usage:** Copy `monke-test/` to `~/.claude/commands/monke-test/`. Invoke: `/monke-test:test-plan <component>`

---

## Arguments

`$ARGUMENTS` parsing:
- First positional: component name from HLD S3 (**required** — must match an LLD filename in `monke-docs/lld/`)
- If missing → read `monke-docs/lld/`, list available components with their phase and design status, ask user to pick
- If component not found → show available components, ask user to pick
- If arg invalid → show options, ask to pick

```
COMPONENT="${ARGUMENTS:?Component name required}"
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

Read `monke-status.md`. Then verify:

- `monke-docs/lld/${COMPONENT}.md` exists with confirmed design (PG-9 passed — check LLDs table: Design column is `done` or has checkmark, OR status is `"reconstructed"` for recon-origin LLDs)
- `monke-docs/hld.md` exists with boundary matrix (S7)
- No blocking OQs for this component (check `monke-docs/open-questions.md` for `Blocks: testing of <component>` or `Blocks: LLD for <component>`)

If LLD does not exist → "No LLD found for `<component>`. Run `/monke-design:lld <component>` first." Stop.
If design not confirmed → "LLD design not yet confirmed (PG-9). Complete the design phase first." Stop.

---

## Phase 1: Read LLD

1. Read `monke-docs/lld/${COMPONENT}.md` in full. Extract:
   - **Header:** component type (`traditional` or `agentic`), pattern (if agentic), upstream/downstream contracts, error types
   - **Public functions/methods:** every function listed in the File Map with `Exports` column — these are the public API
   - **Pure vs IO separation:** which functions are pure, which are IO shell
   - **State machines:** any transition tables defined in Internal Design
   - **Boundary contracts:** upstream model, downstream model, error types crossing boundary
   - **File map:** all files with their layer assignments

2. Read `monke-docs/hld.md` S7 (Boundary Matrix). Extract all rows where this component appears as Upstream or Downstream.

3. Determine component characteristics:
   - Is it `agentic`? → will need agentic-specific test cases
   - Does it have state machines? → will need transition coverage
   - How many public functions? → determines test plan size
   - How many boundaries? → determines integration plan size

Present a brief summary of what was found before proceeding.

---

## Phase 2: Unit Test Plan

Generate the unit test plan per `test-specs.md` S2 Unit Tests and `design-specs.md` S6.3.

### Rules

- **Every public function:** 1 happy path + 1 edge case + 1 error case minimum
- **Pure functions (Layer = pure):** should need **no mocks** — if a pure function needs mocks, flag this as a design smell and note it
- **IO functions:** mock external dependencies per `test-specs.md` S3 Mock Boundaries (unit tier column)
- **State machines:** every valid transition + every invalid transition
- **Agentic components** (if type = `agentic`):
  - Loop termination — does the loop stop when it should?
  - Tool failure — what happens when an external tool call fails?
  - Stale memory — what if retrieved context is outdated?
  - Context overflow — what if the context window/token budget is exceeded?

### Output Table

Generate this table:

```
| # | Function/Method | Input | Expected | Category | Mocks |
|---|----------------|-------|----------|----------|-------|
```

Where:
- `#` — sequential test number
- `Function/Method` — exact function name from LLD
- `Input` — concrete test input (realistic shapes per `test-specs.md` S6.1 — no `"foo"` or `"test123"`)
- `Expected` — concrete expected output or behavior
- `Category` — `happy` | `edge` | `error` | `transition` | `agentic:loop` | `agentic:tool-fail` | `agentic:stale-mem` | `agentic:overflow`
- `Mocks` — what's mocked (or `none` for pure functions)

### Verification

After generating the table, verify:
- Every public function has at least 3 rows (happy + edge + error)
- Every state machine transition has a row
- If agentic: all four agentic categories are covered
- Pure functions have `none` in Mocks column
- No test uses `"foo"`, `"test123"`, or unrealistic placeholder data

---

## Phase 3: Integration Test Plan

Generate the integration test plan per `test-specs.md` S2 Integration Tests and `design-specs.md` S6.3.

### Rules

- **Every boundary** in HLD S7 matrix where this component appears: upstream consumption, downstream production, error propagation
- **Cross-language boundaries:** include serialization round-trip verification if upstream and downstream use different languages
- **Agentic components** (if type = `agentic`):
  - Memory write-then-read consistency
  - Tool call contracts (call format, response format, error format)
  - Agent-to-agent message format verification (if applicable)
- **Both sides REAL:** per `test-specs.md` S3 — integration tests MUST NOT mock the boundary under test. If the other side doesn't exist yet, the test is **deferred** (mark status as `deferred — <component> not yet available`), never written with mocks as placeholders
- **Database:** real test instance, not in-memory substitute (per `test-specs.md` S3-S4)

### Output Table

Generate this table:

```
| # | Boundary | Upstream Call | Expected Downstream Effect | Error Scenario | Mocks |
|---|----------|-------------|---------------------------|----------------|-------|
```

Where:
- `#` — sequential test number
- `Boundary` — matches HLD S7 row (e.g., `ComponentA -> ComponentB`)
- `Upstream Call` — concrete call with realistic input
- `Expected Downstream Effect` — observable effect (data written, message sent, state changed)
- `Error Scenario` — what happens when this boundary fails (error propagation)
- `Mocks` — only things NOT part of this boundary (per `test-specs.md` S3 integration column). Never mock the boundary under test.

### Deferred Tests

If any boundary partner component does not yet have an LLD or is not yet implemented:
- List the test as `deferred`
- Note which component is missing
- Note: "Will be implemented when both sides exist per `test-specs.md` S2"

### Verification

After generating the table:
- Every row in HLD S7 involving this component has at least one integration test
- No test mocks the boundary under test
- Cross-language boundaries have serialization round-trip tests
- If agentic: memory consistency, tool contracts, and agent messaging are covered

---

## Pause Gate

**⏸ PG-10: Present complete test plan (unit + integration tables).**

```
⏸ PAUSE GATE: Test plan for <component>
Unit tests: <N> tests covering <M> public functions
Integration tests: <N> tests covering <M> boundaries (<D> deferred)
Agentic coverage: <yes/no — which categories>
Confirm / Adjust / Reject?
```

- **Confirm** → proceed to Phase 4
- **Adjust** → user specifies changes → regenerate affected rows → re-present PG-10
- **Reject** → stop. Note reason. Suggest `/monke-design:lld <component>` if design issue.

---

## Phase 4: Write Test Plan into LLD

After user confirms PG-10:

1. Read `monke-docs/lld/${COMPONENT}.md`
2. Locate the **Unit Test Plan** section — replace its table with the confirmed unit test table
3. Locate the **Integration Test Plan** section — replace its table with the confirmed integration test table
4. Ensure the **Test Implementation Checklist** exists:
   ```
   - [ ] Unit tests written + passing (commit: <hash>)
   - [ ] Integration tests written + passing (commit: <hash>)
   - [ ] LLD test gate: PASSED — date
   ```
5. Write the updated LLD file

If the LLD file doesn't have test plan sections → add them in the correct location per `design-specs.md` S6.3 (after File Map, before Review Log).

---

## Status Update

On completion, update `monke-status.md`:
- LLDs table: set Test Plan column to `done` (or checkmark) for this component
- If all columns through Test Plan are done → set Status to `ready` (ready for implementation)
- Update "Where We Are" with: "Test plan confirmed for `<component>`"
- Update "Next action" — suggest `/monke-implement:implement <component>` if this was the blocking step, or `/monke-test:test-plan <next-component>` if more components need test plans
- Bump `Updated:` line with current date and `test:test-plan`
