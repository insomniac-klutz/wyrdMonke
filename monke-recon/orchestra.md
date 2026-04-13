# WyrdMonke Recon:Orchestra — Full Reconstruction, One Session, No Mercy

> **Usage:** Copy `monke-recon/` to `.claude/commands/monke-recon/`. Invoke: `/monke-recon:orchestra [scope]`
>
> Chains all 5 recon skills in a single session. Monke drives the bus. You just confirm at every stop.

---

## Arguments

`$ARGUMENTS` parsing:
- Single positional: scope limiter (subdirectory, component name, or glob pattern). Passed through to the survey phase.
- Default (empty): full project reconstruction
- Example: `/monke-recon:orchestra src/api` or `/monke-recon:orchestra "*.rs"`

```
SCOPE="${ARGUMENTS:-}"
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

- **Source code exists.** Something to reconstruct from. No code, no recon.
- **`monke-docs/` exists.** Run `/monke-init` first if it doesn't. Orchestra needs somewhere to write.

---

## Recovery Detection

Before doing anything, check for existing recon artifacts. Orchestra picks up where the last session died.

| Artifact | Exists? | Action |
|----------|---------|--------|
| `monke-docs/recon/recon-survey.md` | yes | Skip Phase 1 |
| `monke-docs/hld.md` (with `Source: reverse-engineered` header) | yes | Skip Phase 2 HLD |
| `monke-docs/lld/*.md` | yes — cross-check against HLD S3 component list; ALL components must have LLDs | Skip Phase 2 LLDs. If only some LLDs exist, skip to Phase 2 LLDs and reconstruct only the missing components. |
| `monke-docs/recon/recon-gaps.md` | yes | Skip Phase 3 |
| `monke-docs/open-questions.md` (populated, not template) | yes | Skip Phase 4 |
| `monke-docs/recon/recon-roadmap.md` | yes | **"Already planned. Nothing to orchestrate."** Stop. |

If resuming mid-chain:

> "Detected prior recon work. Skipping: [list]. Resuming from Phase N."

**Do not silently skip.** Show the recovery state.

---

## Phase 1: Survey

Follow `survey.md` logic inline. Detect flash vs archaeology mode. Deep-scan the codebase — stack, structure, containers, components, dependencies, tests, quality.

**Agent teams:** If the codebase has >20 files, split the scan by directory or container across parallel agents. Merge results into one survey.

Write `monke-docs/recon/recon-survey.md`.

**Present:** container count, component count, test coverage (rough), quality rating, top 3 concerns.

> **Non-negotiable.** Confirm findings before moving on. If user corrects something, update the survey.

---

## Phase 2: Reconstruct

Follow `reconstruct.md` logic inline. Two sub-phases.

### 2A: HLD

Reverse-engineer the High-Level Design from the survey. All 8 HLD sections per `design-specs.md` S5.1. Tag the header with `Source: reverse-engineered from existing codebase`.

Write `monke-docs/hld.md`.

Present PG-1 through PG-4 — **but lighter than greenfield.** User confirms what was found, not what was designed.

> **Non-negotiable.** Each gate pauses for confirmation.

### 2B: LLDs

Write `monke-docs/lld/<component>.md` for every component in HLD S3. Include maturity tags (`as-is` / `needs-work` / `stub`) per function.

**Agent teams:** If >3 components, assign one agent per container for parallel LLD reconstruction.

> **Non-negotiable.** Present each LLD. User confirms the reconstruction matches reality.

---

## Phase 3: Gaps

Follow `gaps.md` logic inline. Full 8-dimension scan: error handling, security, performance, observability, testing, data integrity, documentation, accessibility (if UI exists).

Write `monke-docs/recon/recon-gaps.md`. Group by severity: critical, high, medium, low.

**Present:** Summary table first, then walk through criticals and highs.

User can confirm, dismiss, reprioritize, or add gaps.

> **Non-negotiable.** Update `recon-gaps.md` with user's adjustments before continuing.

---

## Phase 4: OQs

Follow `oqs.md` logic inline. Mine implicit decisions from HLD, LLDs, and gaps. Five dimensions: data model, communication patterns, error strategy, state management, agentic patterns (if applicable). Plus boundary questions, scale questions, business logic, and flash shortcuts.

Write `monke-docs/open-questions.md`. Number sequentially: OQ-001, OQ-002, etc.

**Present:** Counts per dimension, then each OQ. User can confirm, mark N/A, prioritize as blocker, or answer.

> **Non-negotiable.** Update `open-questions.md` with resolutions before continuing.

---

## Phase 5: Roadmap

Follow `roadmap.md` logic inline. Synthesize survey + HLD + gaps + OQs into R1-R5 phases:

- **R1: Unblock** — critical gaps + blocking OQs
- **R2: Foundation** — error types, test infra, shared types, logging
- **R3: Harden** — per-component production hardening
- **R4: Integrate** — cross-boundary contract tests
- **R5: Ship** — system tests, docs, deployment, polish

Include component priority table and effort estimates.

Write `monke-docs/recon/recon-roadmap.md`.

**Present:** Work inventory, phase plan, component priority, effort estimates.

User can adjust priorities, cut scope, add constraints, challenge estimates.

> **Non-negotiable.** Update `recon-roadmap.md` with adjustments before finalizing.

---

## Status Update

Update `monke-status.md` with final recon state:

```
## Recon
- [x] Survey — <date>
- [x] Reconstruct (HLD) — <date>
- [x] Reconstruct (LLDs) — <date>
- [x] Gap analysis — <date>
- [x] Open questions — <date>
- [x] Roadmap — <date>
```

Set "Where We Are": `Phase: **Recon complete — production crawl planned**`

Set "Next action": Point to `/monke-implement:implement` for R1-R5 execution. The jungle has been mapped. Now build the paths.

---

## Anti-Patterns

- **Skipping the survey.** You can't reconstruct what you haven't mapped. The chain is sequential for a reason.
- **Auto-confirming gates.** Orchestra automates the *transitions*, not the *decisions*. Every gate pauses. Every. Single. One.
- **Auto-dismissing gaps.** Honesty over vanity. If the error handling is a dumpster fire, say so.
- **Merging gaps and OQs.** They're different things. Gaps = known missing pieces, known fixes. OQs = implicit decisions that need conscious confirmation. Don't blend them.
- **Skipping recovery detection.** If artifacts exist, respect them. Don't redo work. Don't overwrite confirmed findings.
- **Rushing to roadmap.** Each phase feeds the next. A roadmap without gaps is a wishlist. Gaps without a survey is guessing. The order matters.
- **Dying without saving state.** If context window exhausts mid-chain: (1) Write `monke-status.md` Recon section with last completed phase (e.g., `[x] Survey`, `[~] Reconstruct (LLDs) — 2/5 done`). (2) Set "Where We Are" to phase in progress. (3) Tell user to re-run `/monke-recon:orchestra` — recovery detection picks up from artifacts on disk.
