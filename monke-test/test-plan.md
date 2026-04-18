# WyrdMonke Test:TestPlan — Every Function Gets a Sparring Partner

> **Usage:** `/monke-test:test-plan <component>`
>
> Reads an LLD, writes the unit + integration test tables back into it — happy / edge / error per public function, one row per HLD boundary — so no function ever walks into implementation without a sparring partner waiting.

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

## Inlined Test Rules (from design-specs S11.1 + test-specs S2)

Spec files remain source of truth. Loading abstracted rules inline so this skill is self-contained.

### Test Tiers (design-specs S11.1)

| Tier | Scope | When Written | When Run | Pass Gate For |
|------|-------|-------------|----------|---------------|
| **Unit** | Single function/class in isolation | LLD creation (PG-10) | After each function implemented (Layer 2) | IL-2 |
| **Integration** | Boundary contract between components | LLD creation (PG-10) | After component + neighbors available (Layer 3) | IL-3 |
| **Eval** | Metric threshold for versioned-artifact / data-dependent components | LLD creation (PG-10) | With unit (IL-2, mocked artifact) and integration (IL-3, real artifact) | IL-2 / IL-3 (no separate gate) |
| **System** | End-to-end data flow (HLD S4) | Phase planning (before phase starts) | After all phase components pass IL-3 | PG-11 (phase checkpoint) |

### Tier Rules (test-specs S2)

**Unit Tests:**
- Every public function: 1 happy + 1 edge + 1 error minimum.
- State machines: every valid transition + every invalid transition.
- Agentic: loop termination, tool failure, stale memory, context overflow.
- Isolation: all external services mocked. No network, no filesystem side effects.

**Integration Tests:**
- Every boundary in HLD matrix: upstream consumption, downstream production, error propagation.
- Cross-language: serialization round-trip.
- Agentic: memory write→read, tool call contract, agent-to-agent messages.
- Isolation: both sides REAL, everything else mocked. Database uses real test instance.

**Eval Tests (versioned-artifact or data-dependent only):**
- Metric thresholds (accuracy ≥ X, F1 ≥ Y, coherence ≥ Z, hallucination rate ≤ W, latency p99 ≤ V, drift score ≤ U), not exact-match.
- Unit-level (IL-2): mocked artifact, fixture dataset — tests the function's correctness, not the artifact's quality.
- Integration-level (IL-3): real artifact, test dataset — verifies the artifact meets the LLD threshold.
- Failing eval blocks the gate exactly like a failing test. No weakening thresholds to pass.

**System Tests:**
- Each HLD S4 data flow end-to-end. Error injection per flow. Agentic: full loop with happy and adversarial inputs. All internal components REAL. External mocked or sandboxed.

---

## Phase 2: Unit Test Plan

Generate the unit test plan per the inlined rules above and the LLD Required Sections table format.

### Rules

- **Every public function:** 1 happy path + 1 edge case + 1 error case minimum.
- **Pure functions (Layer = pure):** should need **no mocks** — if a pure function needs mocks, flag this as a design smell and note it.
- **IO functions:** mock external dependencies per test-specs S3 Mock Boundaries (unit tier column).
- **State machines:** every valid transition + every invalid transition.
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
- `Input` — concrete test input (realistic shapes per `test-specs.md` S6 — no `"foo"` or `"test123"`)
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

⏸ **PG-10 [SOFT] — Test plan confirmed.** Present complete test plan (unit + integration tables).

Auto-passes when: every public function has happy+edge+error, every boundary in HLD S7 involving this component has ≥1 integration test.

Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds.

```
⏸ PAUSE GATE PG-10: Test plan for <component>
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

> **Ownership mirror:** This Phase produces the same output as `/monke-design:lld` Phase 4 (Test Plan) — test tables written into `monke-docs/lld/<component>.md`. Use this skill standalone when the LLD already exists without a test plan: **recon-origin LLDs** (reconstructed design, test plan absent), **stub LLDs from fast-onboarding** (skeleton rows need filling), or **pre-existing LLDs** (manually-authored designs missing test tables). These two skills are not meant to run sequentially on the same component — pick whichever entry point matches the LLD's state.

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

If the LLD file doesn't have test plan sections → add them in the correct location (after File Map, before Review Log) per the LLD Required Sections format inlined in `/monke-design:lld`.

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Write a test plan for a component without an LLD | Refuse. LLD is the contract the plan tests — missing LLD means running `/monke-design:lld <component>` first. |
| Add placeholder inputs like `"foo"` / `"test123"` to the table | Refuse. Realistic shapes per test-specs S6 — placeholder data lets contract errors survive the gate. |
| Mock the boundary under test in an integration row | Refuse. Both sides REAL or the row is deferred with a named missing neighbor — never mock-placeholder. |
| Skip eval test rows for versioned-artifact / data-dependent tools | Refuse. Eval rows live in the test plan even though they run inside IL-2/IL-3 — no separate gate, but no hidden rows either. |
| Reduce minimum coverage to happy-path only because tests feel redundant | Refuse. 1 happy + 1 edge + 1 error per public function is the floor; state-machine transitions add rows, never subtract. |

---

## Context Death Protocol

**Checkpoint artifacts:** draft unit + integration tables held in memory until PG-10 passes; on PG-10 confirm, tables written back to `monke-docs/lld/<component>.md`.
**Status line marker:** `Where We Are:` reads `test:test-plan — <component> (<unit | integration | PG-10 waiting>)` while mid-flight.
**Recovery detection:** On re-entry, read `lld/<component>.md`: if Unit Test Plan and Integration Test Plan sections are populated with real rows → tell user "Test plan already present — re-run to revise?" If sections exist but empty → resume at Phase 2 (Unit Test Plan). If sections absent entirely → start fresh from Phase 1.

---

## Status Update

**Read on entry:** `monke-status.md` — check LLDs table for this component's Design column status (must be `done` for greenfield or `"reconstructed"` for recon-origin).
**Write on exit**, update `monke-status.md`:
- LLDs table: set Test Plan column to `done` (or checkmark) for this component
- If all columns through Test Plan are done → set Status to `ready` (ready for implementation)
- Update "Where We Are" with: "Test plan confirmed for `<component>`"
- Update "Next action" — suggest `/monke-implement:implement <component>` if this was the blocking step, or `/monke-test:test-plan <next-component>` if more components need test plans
- Bump `Updated:` line with current date and `by /monke-test:test-plan`
- Blocked: if PG-10 rejected → add Open Blockers row; if LLD design issue revealed during plan → file OQ per `/monke-design:oq` inline logic.
- Partial: if only unit plan written before context death → `Resume:` block names Phase 3 (Integration Test Plan).
