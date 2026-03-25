# WyrdMonke Rage:Renounce — Minimalist Monke With a Machete

> **Usage:** `/monke-rage:renounce [scope]`
>
> Hunts redundancy, dead weight, duplicated logic, over-abstraction, cargo-culted patterns, tech debt. If it doesn't earn its place, monke cuts it.

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

---

## Phase 1: Target Acquisition

Resolve scope: empty = full project | directory path = all files under it | component name = LLD file map match | `docs` = `monke-docs/` + `CLAUDE.md` + `monke-status.md` | glob = matching files.

Collect and categorize all files in scope (source, test, config, doc, spec, skill, other). Present census:

```
Sonar target acquired:
  Mode: renounce | Scope: <description> | Files: <N> total
```

**Confirm scope before scanning.**

---

## Phase 2: Scan — Find Redundancy

Read every file in scope — do not skip or sample. If >20 source files and Agent Teams available, spawn parallel scanners per directory subtree.

### Redundancy Checklist

| Category | What to look for |
|----------|-----------------|
| **Duplication** | Copy-pasted logic (>10 similar lines), reimplemented stdlib/library functions |
| **Dead weight** | Unused dependencies in manifest, unused feature flags, commented-out code blocks |
| **Over-abstraction** | Interfaces with one implementation, factories that create one thing, wrappers that add nothing |
| **Cargo cult** | Patterns copied without understanding — DTO layers with no transformation, repository pattern over an ORM that already is one |
| **Config bloat** | Redundant config keys, env vars set but never read, duplicate config across files |
| **Tech debt** | TODO/FIXME/HACK comments, suppressed lints with no explanation, version pins with no reason |
| **Vestigial** | Migration files for dropped tables still present, old API versions still routed, backwards-compat shims for removed features |

---

## Phase 3: Triage

Classify every finding: `critical` (active confusion from redundancy) | `high` (duplication causing drift) | `medium` (bloat and dead weight) | `low` (minor cleanup) | `note` (potential simplification).

**Deduplication:** Same pattern across files = one grouped finding. **Confidence:** `certain` | `likely` | `possible`.

---

## Phase 4: Report

Present findings grouped by severity (critical first). Finding IDs: `R-NNN`. Each finding: category, one-line summary, file:line, confidence, 2-3 sentence detail, suggested fix.

**Actions menu:** Save (write rage-run log) | Focus (deep-dive a finding by ID — show ±20 lines, root cause, concrete fix, ask Apply/Skip/Back) | Dismiss (remove from report, log as `dismissed — <reason>`) | Rerun (different mode/scope) | Done.

---

## Phase 5: Save Rage Run

Write to `monke-docs/rage-runs/<YYYY-MM-DD>-renounce-<short-scope>.md`. Use template at `monke-docs/rage-run/template.md`. Create directory if needed. **Confirm filename.**

---

## Phase 6: Next Steps

"Start with dead weight — safest removals. Run `/monke-rage:echo` to find more dead code."

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
