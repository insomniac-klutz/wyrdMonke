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

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

Read `monke-status.md`. Then verify:

- `monke-docs/hld.md` has confirmed L3 (S3 component map exists with this component)
- No blocking OQs for this component (check `open-questions.md` for `Blocks: LLD for <component>`)
- `monke-docs/lld/<component>.md` does NOT already exist. If it does:
  - **Recon-origin** (header: `Source: reverse-engineered`): this is the bridge from recon to implementation. Ask: "Reconstructed LLD exists. Review it to add a test plan (PG-10) and confirm design (PG-9)?" In review mode: read the existing LLD, verify/refine the design, then generate the test plan per Phase 4. Do NOT rewrite the internal design — preserve reconstruction, add what's missing. **Both PG-9 and PG-10 fire normally in review mode** — the gates are non-negotiable regardless of LLD origin. After PG-10 confirms, update LLD status from `"reconstructed"` to `"ready"` in `monke-status.md`.
  - **Greenfield LLD exists**: ask: "LLD exists. Review it, or start fresh?"
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
5. If component is `agentic` or has versioned-artifact / data-dependent tool subtypes — check for seer artifacts:
   - `monke-docs/recon/profile-*.md` — data profiles with distributions, schema, drift surface (from `/monke-seer:profile`)
   - `monke-docs/decisions/*-experiment-*.md` — model/approach comparison ADRs with metric evidence (from `/monke-seer:experiment`)
   - HLD S7 boundary matrix `versioned(<artifact>, <pin>)` rows + existing LLD headers `Version pins:` — registered pins (from `/monke-seer:registry`)
   - `monke-docs/decisions/*-agentify.md` — agentic vs non-agentic verdict ADR (from `/monke-seer:agentify`)
   - If found, read them. They inform CoALA dimensions, version pins in the LLD header, eval test thresholds, and tool subtype classification.
   - **Conflict check:** If a `seer:agentify` verdict contradicts the HLD S3 type tag (e.g., seer says de-escalate to traditional, HLD says agentic):
     - ⏸ Present the conflict: "Seer verdict says `<X>`, HLD says `<Y>`. Evidence: `<ADR reference>`."
     - Options: (A) Trust HLD — note seer conflict as OQ for later review. (B) Trust Seer — amend HLD type tag via `/monke-design:hld evolve repattern`. (C) Defer — proceed with HLD tag, revisit after implementation reveals which is right.
     - **User decides before decomposition begins.**

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

After design is confirmed, generate the test plan. **Ownership:** `/monke-design:lld` generates the test plan inline as part of the LLD finalization flow. `/monke-test:test-plan` is the standalone equivalent for cases where the test plan needs to be created or revised separately (e.g., recon-origin LLDs that were reconstructed without test plans). Both produce the same output — test tables inside the LLD file. They are not meant to run sequentially on the same component.

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
