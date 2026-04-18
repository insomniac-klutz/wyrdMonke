# monke-status.md
Project: <<<project_name>>> | Updated: <<<date>>> by /monke-init
Rigor: <<<rigor_level>>> (from `.monke-config.md`)

> **Sole-writer rule:** during parallel teammate work, ONLY the lead/orchestrator writes this file. Teammates write per-component files; the lead reconciles into `monke-status.md`.

## Where We Are

Phase: **Bootstrap complete — ready for fast onboarding or HLD**
Next action: `/monke` (auto-detects and routes per component)
Blockers: none

---

## Components

Per-component maturity assessment. Populated by fast onboarding (`/monke`) or updated per skill completion.

| Component | Container | Detected Maturity | Current Layer | IL Gates Passed | LLD Confidence | Status | Override? |
|-----------|-----------|-------------------|---------------|-----------------|----------------|--------|-----------|
| *(populated by fast onboarding)* | | | | | | | |

**Legend:**
- **Detected Maturity**: `missing` / `pre-L0` / `IL-0` / `IL-1` / `IL-2` / `IL-3` / `unknown — scan failed`
- **IL Gates Passed**: e.g. `0,1` (passed IL-0 and IL-1)
- **LLD Confidence**: `auto-generated` / `reviewed` / `verified` (from LLD header, H10)
- **Override?**: blank if auto-detected; `YYYY-MM-DD manual` if user overrode at hard gate (H8)

---

## Features

Active or completed feature threads. Created by `/monke-intake`. One row per thread.

| Thread ID | Request (40 char) | Class | Rigor | Started | Status | Progress | Last step | Last outcome | Finished |
|-----------|-------------------|-------|-------|---------|--------|----------|-----------|--------------|----------|
| _no active threads_ | | | | | | | | | |

**Columns:**
- **Thread ID** — `feature-<YYYYMMDD-HHMM>-<slug>`.
- **Request** — first 40 characters of the user's original request string.
- **Class** — `greenfield-mvp` / `greenfield-production` / `feature-on-existing` / `bug-fix` / `refactor-cleanup` / `investigation`.
- **Rigor** — rigor active for this thread.
- **Started** — ISO date when PG-1 was approved.
- **Status** — `active` / `blocked` / `paused` / `aborted` / `completed`.
- **Progress** — `step <N>/<M>` during Phase 5; `—` before approval; `closed` after Phase 6.
- **Last step** — the last sub-skill dispatched by intake.
- **Last outcome** — `pass` / `fail` / `escalated` / `awaiting-user`.
- **Finished** — ISO date when Phase 6 closed (blank if not yet).

Only `/monke-intake` writes to this table. Sub-skills dispatched by intake write to their own domain tables (Components, Implementation, Test Gates) — intake reconciles the Features row progress by reading those after each step.

---

## Gate Audit Log (this session)

One line per gate surfaced or auto-passed. Keep chronological.

```
- PG-1 SCOPE — human confirmed (greenfield)
- PG-2 CONTAINERS — auto-confirmed (quality checks passed, rigor=standard)
- PG-3 COMPONENTS — human confirmed (2 ambiguous boundaries)
```

*(auto-populated by skills as gates fire)*

---

## Bootstrap
- [x] Templates fetched
- [x] Rigor set (<<<rigor_level>>>)
- [x] CLAUDE.md merged
- [ ] Stack detected
- [ ] Project-specs filled (auto-detect + manual override)

## HLD
- [ ] L1 Context — PG-1
- [ ] L2 Containers — PG-2
- [ ] L3 Components — PG-3
- [ ] Boundary Matrix — PG-4

## LLDs
| Component | Phase | ADaPT | Design | Review | Test Plan | Confidence | Status |
|-----------|-------|-------|--------|--------|-----------|------------|--------|
| *(populated after HLD S3 or fast onboarding auto-gen)* | | | | | | | |

## Implementation
| Component | L0 | L1 | L2 (funcs) | L3 | Status |
|-----------|----|----|------------|----|----|
| *(populated after LLD is complete)* | | | | | |

## Test Gates
| Component | Unit | Integration | Coverage | Cycle | Status |
|-----------|------|-------------|----------|-------|--------|
| *(populated during implementation)* | | | | 0 | |

> The `Cycle` column tracks retry iterations for PG-13 (persistent test failure). `test-run.md` increments this column on each cycle; PG-13 fires when it hits the configured threshold (default 3).

## Phase Checkpoints
| Phase | Components | All IL-3? | System Tests | PG-11 | Status |
|-------|-----------|-----------|--------------|-------|--------|
| *(populated after HLD S8 phase plan)* | | | | | |

## Open Blockers
| ID | Blocks | Summary |
|----|--------|---------|
| *(none yet)* | | |

## Decisions
| ADR | Status | Component |
|-----|--------|-----------|
| *(none yet)* | | |
