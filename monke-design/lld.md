# WyrdMonke Design:LLD — Component Low-Level Design

> **Usage:** Copy `monke-design/` to `~/.claude/commands/monke-design/`. Invoke: `/monke-design:lld <component>`

---

## Arguments

`$ARGUMENTS` parsing:
- First positional: component name from HLD S3 (**required**)
- If missing → read HLD S3, list available components with their phase and status, ask user to pick
- If component not found in HLD S3 → show available components, ask user to pick

```
COMPONENT="${ARGUMENTS:?Component name required}"
```

---

## Prerequisites

Read `monke-status.md`. Then verify:

- `monke-docs/hld.md` has confirmed L3 (S3 component map exists with this component)
- No blocking OQs for this component (check `open-questions.md` for `Blocks: LLD for <component>`)
- `monke-docs/lld/<component>.md` does NOT already exist (if it does → ask: "LLD exists. Review it, or start fresh?")
- Component's phase dependencies are met (all prior-phase components have LLDs)

If prerequisites fail → explain what's missing and which skill to run first.

---

## Agent Teams Check

Same detection as `/monke-design:hld`. If available → Team mode (Designer+Reviewer). If not → Solo mode.

---

## Phase 1: Context Gathering

1. Read the component's entry in HLD S3:
   - Responsibility, boundary contracts, dependency direction, type tag
   - For agentic: CoALA summary
2. Read upstream/downstream components' boundary contracts from HLD S7
3. Read `project-specs.md` S10 (locked stack) + S2 (stack bindings)
4. Read `project-specs.md` S9 (test bindings)

Present context summary to user before proceeding.

---

## Phase 2: ADaPT Decomposition

Execute ADaPT per `design-specs.md` S1.5:

1. **DECOMPOSE** into sub-problems:
   - What are the distinct responsibilities?
   - For agentic: decompose each CoALA dimension separately
   - **For components with versioned-artifact or data-dependent tools:** separate pure pre/post-processing (feature extraction, input validation, output parsing, metric computation) from artifact interaction (model inference, store query, artifact loading). If the component owns the artifact's lifecycle (training/rebuild), decompose that as a sub-problem with its own Layer 0-3 cycle — not as a separate system.
   - Surface the decomposition tree visibly

2. **⏸ PG-8: Present decomposition to user. Confirm before sub-problems are attempted.**

3. For each sub-problem: **ATTEMPT** fully, **ASSESS** against boundary contract

4. If >3 responsibilities in one unit, >3 error paths, or "handles X, Y, Z, and also W" → decompose further

5. **ADAPT** if needed (restructure, decompose further, escalate to user)

6. **INTEGRATE** — verify end-to-end against upstream/downstream contracts

---

## Phase 3: Design

### Team Mode

Spawn team per `design-specs.md` S3.4:

```
Create an agent team to design the LLD for <component>.
Two teammates:

Teammate 1 — Designer:
<spawn prompt from design-specs S3.4, populated with HLD context and decomposition>

Teammate 2 — Reviewer:
<spawn prompt from design-specs S3.4>
```

Designer writes design. Reviewer verifies against HLD contracts (max 3 rounds). Violations sent back via peer message.

### Solo Mode

Write design directly, self-reviewing against HLD contracts after each section.

### Required Sections (per design-specs S6.3)

**Header:**
```
Parent HLD: S3.<section>  |  HLD Version: X.Y
Backend: per project-specs S10.2  |  ADR: NNN (if non-default)
Type: traditional/agentic  |  Pattern: <name> (agentic)  |  ADR: NNN
Upstream: <Model> from <module>  |  Downstream: <Model> to <module>
Errors: <what crosses boundary>
Tool subtypes: <static-contract | versioned-artifact | data-dependent> (if non-default)
Version pins: <artifact: version> (if versioned-artifact)
Eval thresholds: <metric: threshold> (if versioned-artifact or data-dependent)
Phase: <N>  |  Test gate: pending
```

**Internal Design:**
- Full-typed signatures; pure functions separated from IO functions
- State machines + transition tables (if applicable)
- Algorithms + complexity + edge cases
- Expected errors as return types, unexpected as exceptions

**For agentic, additionally:**
- Memory schemas + storage + retrieval + eviction + token budget
- Tool schemas + sandboxing + retry + action boundaries
- Decision loop + stopping condition + max iterations + HITL
- Prompt templates + variable injection + output parsing

**File Map:**
| File | Layer | Exports | Depends On |
|------|-------|---------|------------|

**Review Log (from team rounds):**
| Round | Violation | Resolution |
|-------|-----------|------------|

**⏸ PG-9: Present converged design. Confirm / Adjust / Reject?**

---

## Phase 4: Test Plan

After design is confirmed, generate the test plan. Follow `/monke-test:test-plan` logic inline (or tell user to run it if installed separately):

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

**⏸ PG-10: Present test plan. Confirm / Adjust / Reject?**

---

## Phase 5: Finalize

1. Write confirmed LLD to `monke-docs/lld/<component>.md`
2. Write ADRs for non-obvious decisions to `monke-docs/decisions/`
3. If boundary contracts changed during design → **HLD amendment required**:
   - Formulate directive: what boundary is wrong, what the LLD needs it to be, why
   - Tell user: "HLD boundary mismatch discovered. Requesting amendment."
   - Invoke `/monke-design:hld amend "<directive>"` (or execute amend flow inline)
   - **⏸ PG-14** fires inside the amend flow — user confirms the HLD change
   - After amend completes: re-read updated HLD boundary, verify LLD now aligns
   - If amend rejected → record as OQ in `open-questions.md`, flag blocker, stop LLD finalization
4. Copy team artifacts if Agent Teams used:
   - Reviewer violation log → LLD "Review Log" section
   - Clean up draft files

5. Suggest next steps:
   - "Implement this component: `/monke-implement:implement <component>`"
   - "Design next component: `/monke-design:lld <next-component>`"
   - "Run `/monke-status:status` to see updated dashboard."

---

## Status Update

On completion, update `monke-status.md`:
- Update LLDs table: set component row to ADaPT ✓, Design ✓, Review ✓, Test Plan ✓, Status "ready"
- Add any new ADRs to Decisions table
- Add any new OQs to Open Blockers
- If HLD was updated (PG-14) → note in "Where We Are"
- Bump `Updated:` line
