# WyrdMonke Implement:Implement — Component Implementation Pipeline

> **Usage:** Copy `monke-implement/` to `~/.claude/commands/monke-implement/`. Invoke: `/monke-implement:implement <component> [layer]`

---

## Arguments

`$ARGUMENTS` parsing:
- First positional: component name matching an LLD filename in `monke-docs/lld/` (**required**)
- Second positional: starting layer — `0` | `1` | `2` | `3` (default: `0`)
- If component missing → read `monke-docs/lld/`, list available components with their implementation status, ask user to pick
- If component not found → show available LLD files, ask user to pick
- If layer arg given → resume from that layer (verify prior layers are checkpointed)

```
COMPONENT="${1:?Component name required}"
LAYER="${2:-0}"
```

---

## Prerequisites

Read `monke-status.md`. Then verify:

- `monke-docs/lld/<component>.md` exists with confirmed design (PG-9 passed) and test plan (PG-10 passed)
- No blocking OQs for this component (check `open-questions.md` for `Blocks: implementation of <component>`)
- `project-specs.md` S8 has IL gate commands defined
- `project-specs.md` S9 has test bindings defined

If resuming (layer > 0):
- Prior layer's IL gate must have passed (check Implementation table in `monke-status.md`)
- If prior layer not complete → tell user: "Layer N-1 not verified. Run from layer N-1 or verify the gate."

If prerequisites fail → explain what's missing and which skill to run first.

---

## Phase 1: Context Loading

1. Read the full LLD: `monke-docs/lld/<component>.md`
   - File map (files to create, layers, exports, dependencies)
   - Internal design (signatures, pure vs IO, state machines)
   - Unit test plan (functions, inputs, expected outputs, categories)
   - Integration test plan (boundaries, upstream/downstream, error scenarios)
   - Decomposition tree (dependency order for Layer 2)

2. Read `project-specs.md`:
   - S8: IL gate commands (what to run after each layer)
   - S9: test bindings (framework, runner command, directories)
   - S10: locked stack (language, tools, conventions)

3. Read upstream/downstream component status:
   - Are neighboring components implemented? (affects Layer 3 integration tests)

4. If resuming from a layer > 0 → verify the component's source files exist and prior gates passed.

Present a brief context summary: component name, file count, function count, layer range to execute.

---

## Phase 2: Layer 0 — Type Skeleton

> Reference: `implementation-specs.md` S3 Layer 0

**Goal:** Create all enum, model, schema, and error type files. No logic, no method bodies.

### Steps

1. Create ALL files from the LLD file map that have `Layer: 0` or contain type/model/schema/enum definitions.
2. For each file:
   - Declare types, enums, models per LLD signatures
   - Models frozen/immutable by default
   - Define error types alongside data types
   - No logic, no function bodies — pure declarations only
3. If database project: generate migration from models, verify up + down per `implementation-specs.md` S7.

### IL-0 Gate

Run IL-0 gate commands from `project-specs.md` S8:
- Linter clean on all new files
- All cross-module imports resolve
- Models frozen by default
- Migration runs up + down (if applicable)

If gate fails → fix issues, re-run. Do NOT proceed to Layer 1.

### Commit

```
Add <component> type skeleton (Layer 0)
```

---

## Phase 3: Layer 1 — Interface Skeleton

> Reference: `implementation-specs.md` S3 Layer 1

**Goal:** Create all function/method stubs with full type annotations. Bodies are placeholders.

### Steps

1. Create remaining files from the LLD file map (service modules, handlers, utilities).
2. For each file:
   - Write function/method stubs with full type annotations from LLD signatures
   - Separate pure functions from IO functions (per LLD's pure vs IO designation)
   - Fallible functions return error types, not raise exceptions
   - Bodies are placeholder stubs (e.g., `raise NotImplementedError`, `todo!()`, `throw new Error('not implemented')`)
   - One-liner docstrings from LLD (purpose + param types)
3. Package/module init files re-export public API (what LLD lists as "Exports").

### IL-1 Gate

Run IL-1 gate commands from `project-specs.md` S8:
- Linter clean
- All downstream modules can import upstream
- Pure vs IO separation verified (no IO imports in pure modules)

If gate fails → fix issues, re-run. Do NOT proceed to Layer 2.

### Commit

```
Add <component> interface skeleton (Layer 1)
```

---

## Phase 4: Layer 2 — Bodies + Unit Tests (Interleaved)

> Reference: `implementation-specs.md` S3 Layer 2, `test-specs.md` S2 Unit Tests

**Goal:** Implement all function bodies with unit tests interleaved. Pure functions first, then IO shell.

### Steps

Implement functions in **LLD decomposition dependency order** (leaves first, callers last):

**For each function:**

1. **Implement:** Replace stub with real implementation.
   - Pure functions first (no IO, deterministic)
   - IO shell functions second (call pure functions, handle side effects)
   - Expected errors as return types, unexpected as exceptions
   - No business logic in IO functions — extract to pure, IO calls pure

2. **Test:** Write unit tests for this function per LLD unit test plan rows.
   - Minimum: 1 happy path + 1 edge case + 1 error case per public function
   - State machines: every valid transition + every invalid transition
   - Agentic components: loop termination, tool failure, stale memory, context overflow
   - Pure functions should need NO mocks
   - External services mocked per `test-specs.md` S3 mock boundary table
   - Test data: deterministic, realistic shapes, no real secrets per `test-specs.md` S6

3. **Verify:** Run unit tests — must pass before moving to next function.

4. **Lint:** Run linter on modified files.

If a test reveals an LLD design issue → record in `monke-docs/open-questions.md`, stop, surface to user. Do NOT silently fix the LLD.

### Commit Granularity

One commit per sub-problem (group of related functions), not per individual function:
```
Implement <component> <sub-problem description> (Layer 2)
```

### IL-2 Gate

Run IL-2 gate commands from `project-specs.md` S8:
- All unit tests passing
- Linter clean
- Pure functions have no IO (no network, no filesystem, no database imports)

If gate fails → fix issues, re-run. Do NOT proceed to Layer 3.

---

## Phase 5: Layer 3 — Integration Tests

> Reference: `implementation-specs.md` S3 Layer 3, `test-specs.md` S2 Integration Tests

**Goal:** Write integration tests per LLD plan. Verify full suite. Check coverage.

### Prerequisites Check

Before writing integration tests, verify both sides of each boundary exist:
- Check upstream/downstream components' implementation status
- If a neighbor is not yet implemented → defer that boundary's integration tests. Note which tests are deferred and why. Do NOT write integration tests with mock placeholders for real components.

### Steps

1. **Write integration tests** per LLD integration test plan, one boundary at a time:
   - Every boundary in HLD matrix: upstream consumption, downstream production, error propagation
   - Cross-language boundaries: serialization round-trip verification
   - Agentic: memory write-read consistency, tool call contracts, agent-to-agent messages
   - Both sides of boundary under test are REAL — everything else mocked per `test-specs.md` S3
   - Database tests use real test instance, not in-memory substitute

2. **Run full test suite** (unit + integration).

3. **Verify coverage** meets project-defined threshold from `project-specs.md` S9.

4. **Update LLD test checklist** with commit references:
   ```
   - [x] Unit tests written + passing (commit: <hash>)
   - [x] Integration tests written + passing (commit: <hash>)
   - [x] LLD test gate: PASSED — <date>
   ```

### IL-3 Gate

Run IL-3 gate commands from `project-specs.md` S8:
- All tests passing (unit + integration)
- Coverage meets threshold
- LLD checklist updated

If gate fails → fix issues, re-run. If persistent failure (>2 cycles) → **stop and surface to user per PG-13** (design-specs S9.4). User decides: fix, redesign, or escalate.

### Commit

```
Add <component> integration tests (Layer 3)
```

---

## Phase 6: Finalize

1. Verify all IL gates passed (IL-0 through IL-3).
2. Update LLD test implementation checklist with final commit hashes.
3. If LLD boundary contracts changed during implementation → **surface to user**. May require HLD update (PG-14).

### Suggest Next Steps

- If more components in this phase need implementation:
  "Next component: `/monke-implement:implement <next-component>`"
- If all phase components are at IL-3:
  "Phase ready for checkpoint: `/monke-implement:checkpoint <phase>`"
- If integration tests were deferred (neighbor not implemented):
  "Deferred integration tests for <boundary>. Will be available after `/monke-implement:implement <neighbor>`"
- "Run `/monke-status:status` to see updated dashboard."

---

## Anti-Patterns to Refuse

Per `implementation-specs.md` S8:

| If asked to... | Do instead... |
|----------------|--------------|
| Skip Layer 0-1, go straight to bodies | Refuse. Skeleton catches contract errors early. |
| Write all tests after all code | Refuse. Layer 2 interleaves body + tests per function. |
| Skip IL gates | Refuse. Run gate commands at each layer boundary. |
| Mock backing services in integration tests | Refuse. Real services per `test-specs.md` S3. |
| Combine multiple LLD components in one pass | Refuse. One component at a time through the pipeline. |
| Put business logic in IO functions | Refuse. Extract to pure function, IO calls pure. |
| Raise exceptions for expected errors | Refuse. Return error type. Exceptions for unexpected only. |

---

## Status Update

On completion, update `monke-status.md`:
- **Implementation table:** Update component row with layer checkmarks (L0/L1/L2/L3) and status
  - Partial completion: note highest passing IL gate and resume point (e.g., "L2 (4/7 funcs)")
  - Full completion: all layers checked, status "IL-3 passed"
- **Test Gates table:** Update component row — Unit (pass/fail), Integration (pass/fail/deferred), Coverage (percentage)
- If integration tests deferred → note in status column: "integration deferred — <neighbor> not ready"
- If OQ created during implementation → add to Open Blockers table
- Bump `Updated:` line with current date and `/monke-implement:implement`
- Recalculate "Where We Are" and "Next action"
