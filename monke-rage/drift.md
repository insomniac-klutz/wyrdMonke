# WyrdMonke Rage:Drift — Auditor Monke With a Magnifying Glass

> **Usage:** `/monke-rage:drift [scope]`
>
> Hunts spec-code divergence — HLD says X, code does Y, contracts mismatch, status.md is stale. If the map doesn't match the territory, monke finds the lie.

---

## Arguments

`$ARGUMENTS` parsing: single positional scope limiter — subdirectory, component name, glob, or `docs` (default: entire project).

```
SCOPE="${ARGUMENTS:-}"
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

- Inside a git repository
- At least one source file or doc file exists in scope
- **`monke-docs/hld.md` exists** (with confirmed sections)
- **At least one `monke-docs/lld/*.md` exists**
- **`monke-status.md` exists**

If drift prerequisites fail -> tell user: "Nothing to drift against. Run `/monke-recon:reconstruct` or `/monke-design:hld` first to establish the spec baseline." **Stop.**

---

## Phase 1: Target Acquisition

Resolve scope: empty = full project | directory path = all files under it | component name = LLD file map match | `docs` = `monke-docs/` + `CLAUDE.md` + `monke-status.md` | glob = matching files.

Collect and categorize all files in scope (source, test, config, doc, spec, skill, other). Present census:

```
Sonar target acquired:
  Mode: drift | Scope: <description> | Files: <N> total
```

⏸ **PG-1 [HARD] — Scope confirmed before scanning.** Always surfaces. Never skippable. Drift findings can trigger spec amendments — scope frames which specs are in scope for comparison.
Confirm / Adjust / Reject?

---

## Phase 2: Scan — Find Spec-Code Divergence

Read every file in scope — do not skip or sample. Cross-reference against spec documents. If >20 source files and Agent Teams available, spawn parallel scanners per directory subtree.

### Divergence Checklist

| Category | What to look for |
|----------|-----------------|
| **HLD vs Code** | Components in HLD not in code (or vice versa), containers that don't match deploy reality |
| **LLD vs Code** | Functions in LLD not implemented, implemented functions not in LLD, signature mismatches |
| **Boundary matrix** | Contracts in S7 that don't match actual cross-module types, missing error types at boundaries |
| **Status staleness** | `monke-status.md` claims X but code/tests show Y, checkboxes wrong |
| **Phase plan** | Components marked "waiting" that have code, components marked "IL-3 passed" with failing tests |
| **ADR compliance** | Decisions recorded in ADRs not reflected in code (chose Option A, built Option B) |
| **Test plan vs tests** | Test plan rows in LLD with no corresponding test file/function |

---

## Phase 3: Triage

Classify every finding: `critical` (spec and code contradict on critical behavior) | `high` (contracts wrong, status misleading) | `medium` (stale docs, spec hasn't kept up) | `low` (naming differences, cosmetic) | `note` (spec could be clearer).

**Deduplication:** Same pattern across files = one grouped finding. **Confidence:** `certain` | `likely` | `possible`.

---

## Phase 4: Report

Present findings grouped by severity (critical first). Finding IDs: `D-NNN`. Each finding: category, one-line summary, file:line, confidence, 2-3 sentence detail, suggested fix.

**Actions menu:** Save (write rage-run log) | Focus (deep-dive a finding by ID — show ±20 lines, root cause, concrete fix, ask Apply/Skip/Back) | Dismiss (remove from report, log as `dismissed — <reason>`) | Rerun (different mode/scope) | Done.

---

## Phase 5: Save Rage Run

Write to `monke-docs/rage-runs/<YYYY-MM-DD>-drift-<short-scope>.md`. Use template at `monke-docs/rage-run/template.md`. Create directory if needed.

⏸ **PG-11 [SOFT] — Filename confirmed before write.** Auto-pass when: scope slug is unambiguous (single directory/component, not a glob) AND no existing rage-run collides with the slug for today. Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds.
Confirm / Adjust / Reject?

---

## Phase 6: Next Steps

"Update specs to match code OR update code to match specs — pick a direction for each finding. Run `/monke-status:status rebuild` to refresh the dashboard."

Cross-mode: "Run `/monke rage:<mode>` (buggy | improv | renounce | haunt | echo) to scan with a different lens on the same scope."

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Propose rewrites without reading `git log` | Refuse. Drift findings need to ask "which side is newer — spec or code?" That's a git-log question. Cite the commit history before recommending direction. |
| Auto-rewrite HLD to match code | Refuse. Drift is surfaced, not silently resolved. Present the divergence; let the user decide whether to update the spec, update the code, or both. That's `/monke-design:hld evolve` territory. |
| Flag stylistic LLD deviations as `critical` | Refuse. Parameter-name drift and comment-count drift are `low`/`note`. `critical` means the contract lies about behavior, not that the docstring is stale. |
| Compare code against a stale ADR without checking supersession | Refuse. An ADR marked `superseded by ADR-XXX` is no longer the authority. Follow the supersession chain to the live ADR before flagging drift. |
| Mark HLD stubs with `<<<...>>>` placeholders as drift | Refuse. Placeholders are bootstrap state, not drift. `/monke-init` and `/monke-recon:reconstruct` own those. Skip them in the comparison. |
| Report "test exists but not in test plan" as the same severity as "test plan row has no test" | Refuse. Untracked test is `low`/`note` (spec catch-up). Spec row with no test is `high`/`critical` (coverage lie). Rate them honestly. |

---

## Context Death Protocol

**Checkpoint artifacts (written on context pressure):**
- `monke-docs/rage-runs/<date>-drift-<scope>.draft.md` — partial divergence findings with per-spec/per-code comparison state.
- Spec-vs-code comparison ledger listing which HLD sections / LLDs have been cross-referenced.

**Status line marker:** `Where We Are:` in `monke-status.md` reads `rage:drift — <scope> (<N>/<M> components compared, <K> divergences logged)` while mid-flight.

**Recovery detection (on entry):**
- If `monke-status.md` Resume block names `/monke-rage:drift` AND draft rage-run exists → resume at Phase 2 from last uncompared component.
- If draft exists but marker cleared → verify scope matches, continue from Phase 3 triage.
- If neither present → start fresh from Phase 1.

---

## Status Update

**Read on entry:** `monke-status.md` — note current LLD confidence levels (auto-generated LLDs produce more drift by design) and any Resume blocks from prior drift runs.

**Write on exit:**
- **Success:** bump `Updated:` with today's date + `by /monke-rage:drift`. Append to Gate Audit Log: `- RAGE drift <scope> — <N critical / K high / ...> (see <path>)`. For drift findings requiring HLD/LLD amendments, note pending `/monke-design:hld evolve` or `/monke-design:lld` steps.
- **Blocked:** if no HLD/LLD baseline exists, add a row to Open Blockers with WHAT/WHY/HOW pointing to `/monke-recon:reconstruct`.
- **Partial:** write a `Resume:` block with phase + comparison ledger for context-death recovery.
