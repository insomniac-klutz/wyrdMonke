# WyrdMonke Implement:Implement — From Blueprints to Breathing Code

> **Usage:** `/monke-implement:implement <component> [layer]`
>
> Drives one LLD through the Layer 0→1→2→3 pipeline — types, skeletons, bodies interleaved with unit tests, then integration tests — with AUTO gates silently verifying each layer before the next begins.

---

## Arguments

`$ARGUMENTS` parsing:
- First positional: component name matching an LLD filename in `monke-docs/lld/` (**required**)
- Second positional: starting layer — `0` | `1` | `2` | `3` (default: `0`, or the maturity-derived entry layer if an auto-generated LLD indicates a higher starting point), OR `resume:<N>` — skip to phase `<N>` with checkpoint re-read (see drafter §7). Phase numbers map: 2=Layer 0, 3=Layer 1, 4=Layer 2, 5=Layer 3, 6=Finalize.
- If component missing → read `monke-docs/lld/`, list available components with their implementation status, ask user to pick
- If component not found → show available LLD files, ask user to pick
- If layer arg given → resume from that layer (re-run IL gate commands from `project-specs.md` S8 for the prior layer to verify it actually passed — if it didn't, start from that layer instead)

```
COMPONENT="${1:?Component name required}"
LAYER="${2:-auto}"
RESUME_PHASE="$(echo "$LAYER" | grep -oE '^resume:[0-9]+$' | cut -d: -f2)"
```

When `LAYER == "auto"`, read the LLD header's `Confidence:` and per-function maturity tags to pick the entry layer:
- All functions `as-is` and IL-0/IL-1 detection passed → start at Layer 2 (verify + add tests).
- Mixed `as-is`/`needs-work` → start at the lowest unsatisfied IL gate.
- Any function `stub` → start at Layer 0.

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

Read `monke-status.md`. Then verify:

- `monke-docs/lld/<component>.md` exists and is implementation-ready:
  - **Greenfield / Reviewed** (`Confidence: reviewed`): PG-9 (design confirmed) + PG-10 (test plan confirmed) passed.
  - **Recon-origin** (header: `Source: reverse-engineered`): reconstruction confirmed + test plan present (unit + integration tables populated). If test plan missing → "Recon mapped this component but needs a test plan before implementation. Run `/monke-design:lld <component>` in review mode or `/monke-test:test-plan <component>`."
  - **Stub from fast onboarding** (`Confidence: auto-generated`, `Source: fast-onboarding scan`): valid IF AND ONLY IF the stub LLD contains the B1 minimum:
    - File Map present
    - Signatures extracted (not inferred)
    - Maturity tags per function (as-is/needs-work/stub)
    - **Decomposition tree present** (from import graph — leaves/middle/roots)
    - **Test plan skeleton present** (unit rows per public function, integration rows per boundary, even if cells are TBD)
    If ANY of these are missing → "Stub LLD incomplete. Re-run `/monke-design:lld <component> stub` or run `/monke-design:lld <component>` to fill it in properly." Stop.
- No blocking OQs for this component (check `open-questions.md` for `Blocks: implementation of <component>`).
- `project-specs.md` S8 has IL gate commands defined (per-container if polyglot).
- `project-specs.md` S9 has test bindings defined (per-container if polyglot).

If resuming (layer > 0):
- Prior layer's IL gate must have passed (check Implementation table in `monke-status.md`).
- If prior layer not complete → tell user: "Layer N-1 not verified. Run from layer N-1 or verify the gate."

If prerequisites fail → explain what's missing and which skill to run first.

---

## Inlined Implementation Spec (from implementation-specs S3, S4, S7)

Spec file `monke-docs/implementation-specs.md` remains source of truth for maintenance — do not re-load at runtime.

### Contract-First Layered Pipeline (S3)

**Layer 0 — Type Skeleton:**
- Create all enum, model, and schema files from the LLD file map.
- No logic, no method bodies — pure declarations and column/field definitions.
- Models and schemas frozen by default. Define error types alongside data types.
- **Versioned types:** for types serving versioned-artifact or data-dependent tools (design-specs S1.2): include artifact version pin and distributional expectations as declarative annotations. Still pure declarations — no logic — but encoded contract-stability metadata Layer 2 and eval tests depend on.
- Verification: linter clean + all cross-module imports resolve.
- If applicable: generate database migration from models, verify up + down per Migration Workflow (below).
- **Gate: IL-0** — types importable, models frozen, version pins present (if applicable), migration runs (if applicable), linter clean.

**Layer 1 — Interface Skeleton:**
- Create all function/method stubs with full type annotations from LLD signatures.
- Separate pure signatures from IO signatures. Fallible functions return error types, not raise exceptions.
- Bodies are placeholder stubs (e.g., `raise NotImplementedError`, `todo!()`, `throw new Error('not implemented')`).
- Docstrings from LLD (one-liner purpose + param types).
- Package/module init files re-export public API (what LLD lists as "Exports").
- Verification: linter clean + downstream modules can import upstream, pure vs IO separated.
- **Gate: IL-1** — compilable skeleton.

**Layer 2 — Bodies + Unit Tests (interleaved):**
- Implement **pure functions first**, then IO shell that calls them.
- For components with versioned-artifact or data-dependent tools: the pre/post-processing around the tool call is pure (prompt construction, feature extraction, input validation, output parsing, response scoring, metric computation). The tool call itself (LLM call, model inference, embedding search, feature store query, artifact loading) is IO shell.
- For each function (in LLD decomposition dependency order — leaves first):
  1. Replace stub with real implementation.
  2. Write unit tests for that function per LLD test plan rows.
  3. For functions with eval obligations: write unit-level eval tests (mocked artifact, fixture data).
  4. Run unit tests + eval tests — must pass before moving to next function.
  5. Run linter on modified files.
- **Gate: IL-2** — all unit tests passing, eval tests passing (if applicable), linter clean, pure functions have no IO.

**Layer 3 — Integration Tests:**
- Write integration tests per LLD integration test plan.
- For components with eval obligations: write integration-level eval tests (real artifact, test dataset, metric thresholds from LLD).
- Run full test suite (unit + integration + eval).
- Verify coverage meets project-defined threshold.
- Update LLD test checklist with commit references.
- **Gate: IL-3** — all tests passing (including eval), coverage met → ready for phase checkpoint.

### Implementation Gates (S4)

**All IL gates are AUTO** — they run silently on IL-pass and surface ONLY on failure. Audit log line emitted per successful gate: `[gate:IL-<N>] auto-confirmed (<component>, <layer>, <rigor>)`.

| ID | Type | After Layer | Verification | Proceeds When |
|----|------|-------------|-------------|---------------|
| IL-0 | AUTO | 0 (Types) | Linter clean, all imports resolve, models frozen by default, version pins present (if applicable), migration up/down (if applicable) | Types correct |
| IL-1 | AUTO | 1 (Signatures) | Linter clean, downstream can import upstream, pure vs IO separated | Skeleton compilable |
| IL-2 | AUTO | 2 (Bodies) | All unit tests pass, eval tests pass (if applicable), linter clean, pure functions have no IO | Logic correct |
| IL-3 | AUTO | 3 (Integration) | Full suite green (including eval), coverage ≥ threshold, LLD checklist updated | → Phase checkpoint |

IL gates are lightweight self-verification (run commands, check output). They are NOT user-confirmation pause gates — the user confirms at the phase checkpoint (PG-11).

**On failure:** surface the failure to the user with WHAT/WHY/HOW format (standardized error message). If persistent (>2 cycles) → **PG-13 (TRIGGERED)** fires.

### Migration Workflow (S7 — Database Projects)

Applies when the project uses a relational database with schema migrations:

1. Layer 0 creates all ORM/model definitions.
2. Initialize migration tool (first time only).
3. Configure migration env to discover models.
4. Auto-generate migration from model diff.
5. Review generated migration (auto-generation misses: RLS policies, custom indexes, enum types, triggers).
6. Hand-add any SQL from LLD that auto-generation cannot produce.
7. Run migration up — verify schema correct.
8. Run migration down — verify clean teardown.
9. Commit migration with Layer 0 code.

---

## Phase 1: Context Loading

**Resume check (per drafter §7).** If arguments contain `resume:<N>`:
1. Verify the LLD exists and parses (this skill does not write a lock-file — git commit history + the LLD test implementation checklist are the primary checkpoints).
2. If parse check passes → skip to Phase `<N>` with prerequisites re-validated inline (including the prior-layer IL gate re-verification described in Context Death Protocol).
3. If parse check fails → emit warning "resume:<N> specified but checkpoint invalid" and proceed from Phase 1 normally.
4. See Context Death Protocol below for the full recovery spec.

1. Read the full LLD: `monke-docs/lld/<component>.md`.
   - Header → Confidence level (auto-generated / reviewed / verified).
   - File map (files to create, layers, exports, dependencies).
   - Internal design (signatures, pure vs IO, state machines).
   - Unit test plan (functions, inputs, expected outputs, categories).
   - Integration test plan (boundaries, upstream/downstream, error scenarios).
   - Decomposition tree (dependency order for Layer 2).
   - **Maturity tags** (as-is/needs-work/stub per function) — from recon-origin LLDs or fast-onboarding stub LLDs. These override default layer assumptions:
     - `as-is`: files/types already exist. Layer 0–1 verify only (confirm types compile, imports resolve — no rewrite). Layer 2 adds tests for existing code, fixes only what tests reveal.
     - `needs-work`: files exist but are rough. Layer 0–1 verify, Layer 2 rewrites bodies per gap findings + adds tests.
     - `stub`: placeholder only. Full Layer 0→3 as if greenfield.

2. Read `project-specs.md`:
   - S8: IL gate commands (what to run after each layer). Per-container tables for polyglot support.
   - S9: test bindings (framework, runner command, directories). Per-container tables.
   - S10: locked stack (language, tools, conventions).

3. If `monke-docs/recon/` exists (recon-origin project), also read:
   - `recon-gaps.md` — filter to this component's findings. Critical/high gaps inform Layer 2 priorities.
   - `recon-roadmap.md` — current R-phase context and effort estimates for this component.
   - Cross-reference: maturity tags in LLD + gap severity = implementation priority.

4. Read upstream/downstream component status:
   - Are neighboring components implemented? (affects Layer 3 integration tests).
   - If a neighbor's LLD has `Confidence: auto-generated`, its contract is not trusted — contract changes that arise during implementation → fire PG-14 not silent drift.

5. If resuming from a layer > 0 → verify the component's source files exist and prior gates passed.

Present a brief context summary: component name, file count, function count, layer range to execute, LLD confidence, maturity distribution.

---

## Phase 2: Layer 0 — Type Skeleton

**Goal:** Create all enum, model, schema, and error type files. No logic, no method bodies.

### Steps

1. Create ALL files from the LLD file map that have `Layer: 0` or contain type/model/schema/enum definitions.
2. For each file:
   - Declare types, enums, models per LLD signatures.
   - Models frozen/immutable by default.
   - Define error types alongside data types.
   - No logic, no function bodies — pure declarations only.
3. If database project: generate migration from models, verify up + down per the Migration Workflow above.

### IL-0 Gate (AUTO)

Run IL-0 gate commands from `project-specs.md` S8:
- Linter clean on all new files.
- All cross-module imports resolve.
- Models frozen by default.
- Version pins present (if applicable).
- Migration runs up + down (if applicable).

On success: emit audit line `[gate:IL-0] auto-confirmed (<component>, layer=0)`. Proceed to Layer 1.

On failure: surface with WHAT/WHY/HOW. Do NOT proceed to Layer 1.

### PG-12 Stack-Drift Check (pre-commit)

Run the PG-12 stack-drift check (see Phase 4 for full definition) on all Layer 0 files before the commit. If any import names a framework/library/runtime not in `project-specs.md` S10.1 → fire PG-12 [TRIGGERED]. Ignores rigor.

### Commit

```
Add <component> type skeleton (Layer 0)
```

### Fast-Onboarding Skip

If every Layer-0 file is already tagged `as-is` in the LLD maturity table AND IL-0 gate passes on first run without edits → log `[gate:IL-0] auto-confirmed (verify-only, no changes)`, proceed directly to Layer 1.

---

## Phase 3: Layer 1 — Interface Skeleton

**Goal:** Create all function/method stubs with full type annotations. Bodies are placeholders.

### Steps

1. Create remaining files from the LLD file map (service modules, handlers, utilities).
2. For each file:
   - Write function/method stubs with full type annotations from LLD signatures.
   - Separate pure functions from IO functions (per LLD's pure vs IO designation).
   - Fallible functions return error types, not raise exceptions.
   - Bodies are placeholder stubs.
   - One-liner docstrings from LLD (purpose + param types).
3. Package/module init files re-export public API (what LLD lists as "Exports").

### IL-1 Gate (AUTO)

Run IL-1 gate commands from `project-specs.md` S8:
- Linter clean.
- All downstream modules can import upstream.
- Pure vs IO separation verified (no IO imports in pure modules).

On success: emit audit line, proceed to Layer 2.
On failure: surface, do NOT proceed.

### PG-12 Stack-Drift Check (pre-commit)

Run the PG-12 stack-drift check (see Phase 4 for full definition) on all Layer 1 files before the commit. If any import names a framework/library/runtime not in `project-specs.md` S10.1 → fire PG-12 [TRIGGERED]. Ignores rigor.

### Commit

```
Add <component> interface skeleton (Layer 1)
```

### Fast-Onboarding Skip

If every Layer-1 signature is tagged `as-is` AND IL-1 passes unchanged → log audit line, proceed directly to Layer 2.

---

## Phase 4: Layer 2 — Bodies + Unit Tests (Interleaved)

**Goal:** Implement all function bodies with unit tests interleaved. Pure functions first, then IO shell.

### Steps

Implement functions in **LLD decomposition dependency order** (leaves first, callers last). For stub/auto-generated LLDs, use the mechanical decomposition tree from the import graph.

**For each function:**

1. **Implement:** Replace stub with real implementation.
   - Pure functions first (no IO, deterministic).
   - IO shell functions second (call pure functions, handle side effects).
   - Expected errors as return types, unexpected as exceptions.
   - No business logic in IO functions — extract to pure, IO calls pure.
   - For `as-is` functions: verify existing implementation matches LLD signature, do not rewrite.
   - For `needs-work` functions: rewrite body per LLD, keep signature.
   - For `stub` functions: write from scratch per LLD.

2. **Test:** Write unit tests for this function per LLD unit test plan rows. If the LLD is a stub with TBD inputs/outputs, synthesize realistic inputs per test-specs S6 before committing the tests.
   - Minimum: 1 happy path + 1 edge case + 1 error case per public function.
   - State machines: every valid transition + every invalid transition.
   - Agentic components: loop termination, tool failure, stale memory, context overflow.
   - Pure functions should need NO mocks.
   - External services mocked per test-specs S3 mock boundary table.
   - Test data: deterministic, realistic shapes, no real secrets.

3. **Verify:** Run unit tests — must pass before moving to next function.

4. **Lint:** Run linter on modified files.

If a test reveals an LLD design issue → record in `monke-docs/open-questions.md`, **stop implementation**, surface to user with the OQ and options.

⏸ **PG-9 [TRIGGERED] — LLD design escalation (contract-internal).** Fires when a unit test reveals a design flaw inside this component's contract — the LLD is wrong but the boundary to other components doesn't change. Route to `/monke-design:lld <component>` in revision mode. Auto-pass via rigor does NOT apply (this is an escalation, not a routine design gate).

⏸ **PG-14 [TRIGGERED] — Boundary change escalation.** Fires when implementation reveals that the LLD's contract with a neighbor component is wrong — the boundary shifts. Route to `/monke-design:hld amend <boundary>` first, then `/monke-design:lld` for both sides of the amended boundary. Always surfaces regardless of rigor.

### PG-12 Stack-Drift Check

Before committing any layer's code (Layer 0, 1, or 2), parse imports/requires in the files being added or modified. Cross-check each import against the locked stack in `project-specs.md` S10.1 (the per-container tech table). If any import names a framework/library/runtime NOT in the locked stack → auto-fire PG-12:

⏸ **PG-12 [TRIGGERED] — Stack violation detected.** Evidence: `<file>:<line> imports <framework> not in locked stack (locked: <locked-list>)`. Options: (a) Amend the locked stack (update `project-specs.md` S10.1 + surface PG-14 for HLD impact if boundary changes). (b) Rewrite the import to use a locked equivalent. (c) Abort this layer commit.

Ignores rigor. Always surfaces.

Language-specific import-parsing notes: Python (`^import `, `^from `), TypeScript/JS (`^import `, `require(`), Rust (`^use `), Go (`^import `). Use simple line-match; don't over-engineer.

### Commit Granularity

One commit per sub-problem (group of related functions), not per individual function:
```
Implement <component> <sub-problem description> (Layer 2)
```

### IL-2 Gate (AUTO)

Run IL-2 gate commands from `project-specs.md` S8:
- All unit tests passing.
- Linter clean.
- Pure functions have no IO (no network, no filesystem, no database imports).
- Eval tests passing (if applicable).

On success: emit audit line, proceed to Layer 3.
On failure: surface, do NOT proceed.

---

## Phase 5: Layer 3 — Integration Tests

**Goal:** Write integration tests per LLD plan. Verify full suite. Check coverage.

### Prerequisites Check

Before writing integration tests, verify both sides of each boundary exist:
- Check upstream/downstream components' implementation status.
- If a neighbor is not yet implemented → defer that boundary's integration tests. Record deferred tests in the LLD test implementation checklist: mark the row as `deferred — <neighbor> not at IL-2` with the boundary name, so checkpoint can verify they're written when the neighbor catches up. Do NOT write integration tests with mock placeholders for real components.

### Steps

1. **Write integration tests** per LLD integration test plan, one boundary at a time:
   - Every boundary in HLD matrix: upstream consumption, downstream production, error propagation.
   - Cross-language boundaries: serialization round-trip verification.
   - Agentic: memory write-read consistency, tool call contracts, agent-to-agent messages.
   - Both sides of boundary under test are REAL — everything else mocked per test-specs S3.
   - Database tests use real test instance, not in-memory substitute.

2. **Run full test suite** (unit + integration).

3. **Verify coverage** meets project-defined threshold from `project-specs.md` S9.

4. **Update LLD test checklist** with commit references:
   ```
   - [x] Unit tests written + passing (commit: <hash>)
   - [x] Integration tests written + passing (commit: <hash>)
   - [x] LLD test gate: PASSED — <date>
   ```

### IL-3 Gate (AUTO)

Run IL-3 gate commands from `project-specs.md` S8:
- All tests passing (unit + integration + eval).
- Coverage meets threshold.
- LLD checklist updated.

On success: emit audit line. Bump LLD header `Confidence:` from `reviewed` to `verified`.
On failure: surface. If persistent failure (>2 cycles) → **PG-13 (TRIGGERED)** — user decides: fix, redesign, or escalate.

### Commit

```
Add <component> integration tests (Layer 3)
```

---

## Phase 6: Finalize

1. Verify all IL gates passed (IL-0 through IL-3).
2. Update LLD test implementation checklist with final commit hashes.
3. Update LLD header `Confidence: verified`.
4. If LLD boundary contracts changed during implementation → **PG-14 (TRIGGERED)**: formulate an amend directive, present the boundary change and blast radius to the user. User must confirm the HLD amendment (via `/monke-design:hld amend`) BEFORE this component proceeds to checkpoint. Do NOT allow checkpoint with unamended boundary drift.

### Suggest Next Steps

- If more components in this phase need implementation:
  "Next component: `/monke-implement:implement <next-component>`"
- If all phase components are at IL-3:
  "Phase ready for checkpoint: `/monke-implement:checkpoint <phase>`"
- If integration tests were deferred (neighbor not implemented):
  "Deferred integration tests for <boundary>. Will be available after `/monke-implement:implement <neighbor>`"
- "Run `/monke-status:status` to see updated dashboard."

---

## Gate Classifications Used Here

| Gate | Type | Notes |
|------|------|-------|
| IL-0 | AUTO | Types importable. Surfaces only on failure. |
| IL-1 | AUTO | Skeleton compilable. Surfaces only on failure. |
| IL-2 | AUTO | Unit tests + eval (if applicable) pass. Surfaces only on failure. |
| IL-3 | AUTO | Full suite + coverage. Surfaces only on failure. |
| PG-9 | TRIGGERED | Contract-internal LLD design escalation. Unit test reveals component-internal design flaw (boundary unchanged). Route to `/monke-design:lld` revision. |
| PG-12 | TRIGGERED | Stack-drift detector. Import outside locked stack (`project-specs.md` S10.1). Always surfaces. |
| PG-13 | TRIGGERED | Persistent failure (>2 cycles). User chooses fix / redesign / escalate. |
| PG-14 | TRIGGERED | HLD amendment if boundary contracts changed during implementation. |

Failing AUTO gates become visible. TRIGGERED gates fire regardless of rigor.

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Skip Layer 0-1, go straight to bodies | Refuse. Skeleton catches contract errors early. |
| Write all tests after all code | Refuse. Layer 2 interleaves body + tests per function. |
| Skip IL gates | Refuse. Run gate commands at each layer boundary. |
| Mock backing services in integration tests | Refuse. Real services per test-specs S3. |
| Combine multiple LLD components in one pass | Refuse. One component at a time through the pipeline. |
| Put business logic in IO functions | Refuse. Extract to pure function, IO calls pure. |
| Raise exceptions for expected errors | Refuse. Return error type. Exceptions for unexpected only. |
| Implement against a stub LLD without checking decomposition tree + test plan skeleton | Refuse. Stub must contain both. Run `/monke-design:lld <component>` first. |

---

## Context Death Protocol

**Checkpoint artifacts:** partial implementation commits (git history is the primary checkpoint), updated LLD test implementation checklist with commit hashes, `monke-status.md` Implementation table row reflecting `L2 (K/N funcs)` partial progress.
**Status line marker:** `Where We Are: implement:implement — <component> Layer <N> (<function K/N | integration | IL-<N> running>)` while mid-flight.
**Recovery detection:** On re-entry, read LLD test checklist: if `Unit tests ✓` commit present but `Integration tests ✓` empty → resume Phase 5 (Layer 3). If a Layer-2 partial commit exists but not all decomposition-tree functions are implemented → resume Phase 4 at the next leaf-order function. If all four IL gates have audit entries in recent git log → skip to Phase 6 (Finalize) and verify `Confidence: verified` bump. If no partial state → start at the layer the `auto` calculation picks.

**Prior-layer re-verification.** On any layer-N resume where N > 0, re-execute the IL-(N-1) gate command from `project-specs.md` §8 (e.g., if resuming at Layer 2, run the IL-1 command: linter + compile + import check). If IL-(N-1) now FAILS (someone edited files between sessions), surface the failure with choice:
1. Fix at Layer (N-1) and re-run IL-(N-1) before continuing Layer N.
2. Force-continue at Layer N (user acknowledges broken foundation). Record in Gate Audit Log as `[gate:IL-<N-1>] re-verify FAILED, user forced continue`.

This re-verification is AUTO (silent on pass) and only surfaces on failure.

---

## Status Update

**Read on entry:** `monke-status.md` — check Implementation table for current layer, verify prior layer's IL gate passed (resume safety).
**Write on exit**, update `monke-status.md`:
- **Implementation table:** Update component row with layer checkmarks (L0/L1/L2/L3) and status.
  - Partial completion: note highest passing IL gate and resume point (e.g., "L2 (4/7 funcs)").
  - Full completion: all layers checked, status "IL-3 passed".
- **LLD confidence:** bump from `reviewed` to `verified` in the LLDs table when IL-3 passes.
- **Test Gates table:** Update component row — Unit (pass/fail), Integration (pass/fail/deferred), Coverage (percentage).
- If integration tests deferred → note in status column: "integration deferred — <neighbor> not ready".
- If OQ created during implementation → add to Open Blockers table.
- Bump `Updated:` line with current date and `/monke-implement:implement`.
- Recalculate "Where We Are" and "Next action".
