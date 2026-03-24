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

**Confirm scope before scanning.**

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

Write to `monke-docs/rage-run/<YYYY-MM-DD>-drift-<short-scope>.md`. Use template at `monke-docs/rage-run/template.md`. Create directory if needed. **Confirm filename.**

---

## Phase 6: Next Steps

"Update specs to match code OR update code to match specs — pick a direction for each finding. Run `/monke-status:status rebuild` to refresh the dashboard."

Cross-mode: "Run `/monke-rage:orchestra` to scan with a different lens on the same scope."

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Scan without presenting findings | Refuse. Always pause for triage review. |
| Auto-fix all without confirmation | Refuse. Monke presents, human decides. |
| Suppress findings for a clean report | Refuse. Honesty over vanity. |
| Skip files ("probably fine") | Refuse. Sampling is lying. |
| Rate everything critical | Refuse. Severity must be honest. |
| Ignore test files | Refuse. Broken tests are as bad as broken code. |
