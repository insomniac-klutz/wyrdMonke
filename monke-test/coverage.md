# WyrdMonke Test:Coverage — Coverage Analysis & Threshold Check

> **Usage:** Copy `monke-test/` to `~/.claude/commands/monke-test/`. Invoke: `/monke-test:coverage [scope]`

---

## Arguments

`$ARGUMENTS` parsing:
- Single positional: scope (optional)
  - Component name → coverage for that component's files only
  - `phase-N` (e.g., `phase-1`) → coverage for all components in that phase
  - Empty → full project coverage
- If arg is not a recognized component name or `phase-N` pattern → show available options, ask to pick

```
SCOPE="${ARGUMENTS:-all}"
```

---

## Prerequisites

Read `monke-status.md`. Then verify:

- `monke-docs/project-specs.md` S9 has coverage tool + threshold filled (not `<<<`)
- Tests exist to measure — at least one test file in `tests/unit/` or `tests/integration/`
- If scope is a component → `monke-docs/lld/<component>.md` exists with a test plan
- If scope is `phase-N` → that phase exists in HLD S8 and has components with tests

If coverage tool not configured → "Coverage tool not configured. Fill `project-specs.md` S9 first." Stop.
If no tests found → "No tests found to measure coverage against. Write tests first." Stop.

---

## Phase 1: Resolve Tool

1. Read `monke-docs/project-specs.md` S9:
   - Coverage tool command (e.g., `pytest --cov`, `vitest --coverage`, `cargo tarpaulin`)
   - Coverage threshold (minimum line coverage percentage)
   - Coverage report format (if specified)

2. Read `test-specs.md` S5 for exclusion rules:
   - Package init re-exports
   - Layer 1 placeholder stubs (before Layer 2 fills them in)
   - Generated code

3. Determine scope filter:
   - Component → filter to files listed in that component's LLD File Map
   - `phase-N` → collect all file maps from all LLD components in that phase
   - `all` → no filter (full project)

4. Build the coverage command with appropriate scope filter and exclusions.

Present the resolved command and exclusions to the user before executing.

---

## Phase 2: Run Coverage

1. Execute the coverage tool with the resolved command.
2. Capture the full coverage report.

3. If the coverage tool fails to run (not a threshold failure, but an execution error) → diagnose and report. Common causes:
   - Missing coverage dependency → suggest install command
   - Test failures preventing coverage collection → suggest `/monke-test:test-run` first
   - Configuration issue → point to project-specs S9

---

## Phase 3: Analyze

Parse the coverage report and present:

### Summary

```
Scope: <component | phase-N | full project>
Overall coverage: <X>%
Threshold: <Y>%
Status: <PASS | FAIL> (<+Z% above | -Z% below> threshold)
```

### Per-File Breakdown

```
| File | Lines | Covered | Missed | Coverage % | Status |
|------|-------|---------|--------|------------|--------|
```

Sort by coverage percentage ascending (worst-covered files first).

### Uncovered Lines

For each file below threshold (or the 5 worst files if all are above):
- List specific uncovered line ranges
- Map uncovered lines back to functions from the LLD File Map where possible
- Note whether uncovered code is: untested public function, untested error path, untested edge case, or excluded code (init re-exports, stubs, generated)

### Component-Level View (if scope is `phase-N` or `all`)

```
| Component | LLD Files | Coverage % | Lowest File | Gap |
|-----------|-----------|------------|-------------|-----|
```

Identify which LLD components have the lowest coverage.

---

## Phase 4: Recommend

### If Above Threshold (PASS)

Coverage meets the floor. Remind:
- "Coverage is a floor, not a target" (`test-specs.md` S5)
- "High coverage with weak assertions is worse than moderate coverage with strong assertions"
- If any files are significantly above average but have simple logic → flag potential weak-assertion risk (tests that assert existence but not correctness)

Suggest next step: proceed with gate verification via `/monke-test:test-run`.

### If Below Threshold (FAIL)

**Gate is blocked. No exceptions** (`test-specs.md` S5).

Prioritize what to test next, in this order:

1. **Untested public functions** — find public functions from LLD that have zero test coverage. Cross-reference against the LLD unit test plan table: which test plan rows haven't been implemented yet? List them with their test plan row numbers.

2. **Untested boundaries** — find boundary code (upstream/downstream contract handling) with no integration test coverage. Cross-reference against the LLD integration test plan table.

3. **Untested error paths** — find error handling code (catch blocks, error returns, fallback logic) that's never exercised.

4. **Specific recommendations:**
   ```
   Priority | What to Test | LLD Reference | Estimated Impact |
   ---------|-------------|---------------|-----------------|
   1        | <function>  | Unit plan #N  | +X% coverage    |
   2        | <boundary>  | Integration #N| +X% coverage    |
   ```

Present the prioritized list and suggest: "Implement these tests to reach the `<Y>%` threshold, then re-run `/monke-test:coverage <scope>`."

### Anti-Pattern Enforcement

If coverage drops below threshold:
- Gate is blocked, no exceptions (`test-specs.md` S5)
- Do NOT suggest lowering the threshold
- Do NOT suggest excluding more files to game the number
- Do NOT suggest writing trivial tests just to hit the number — coverage with weak assertions is explicitly called out as worse than moderate coverage with strong assertions

---

## Status Update

On completion, update `monke-status.md`:

### On PASS
- Test Gates table: update Coverage column to `passed (<X>%)` for the relevant component(s)
- If this was the last gate needed for IL-3 → note that IL-3 is ready for verification via `/monke-test:test-run integration <component>`
- Update "Where We Are" with: "Coverage `<X>%` meets `<Y>%` threshold for `<scope>`"
- Update "Next action":
  - If unit + integration gates already passed → suggest `/monke-implement:checkpoint <phase>`
  - Otherwise → suggest the next unfulfilled gate
- Bump `Updated:` line with current date and `test:coverage`

### On FAIL
- Test Gates table: update Coverage column to `failed (<X>% / <Y>% needed)` for the relevant component(s)
- Update "Where We Are" with: "Coverage below threshold for `<scope>` — gate blocked"
- Update "Next action": "Implement priority tests (see coverage recommendations), then re-run `/monke-test:coverage <scope>`"
- Bump `Updated:` line
