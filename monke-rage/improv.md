# WyrdMonke Rage:Improv — Ambitious Monke Sees the Mountain

> **Usage:** `/monke-rage:improv [scope]`
>
> Hunts improvements, north stars, performance wins, API ergonomics, patterns that could be cleaner. Monke knows it can be better.

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
  Mode: improv | Scope: <description> | Files: <N> total
```

**Confirm scope before scanning.**

---

## Phase 2: Scan — Find Improvements

Read every file in scope — do not skip or sample. If >20 source files and Agent Teams available, spawn parallel scanners per directory subtree.

### Improvement Checklist

| Category | What to look for |
|----------|-----------------|
| **Performance** | O(n^2) where O(n) is possible, redundant allocations, unbounded growth, N+1 queries |
| **API ergonomics** | Awkward call signatures, stringly-typed parameters, boolean traps, inconsistent naming |
| **Pattern upgrade** | Manual loops -> iterators/streams, callback hell -> async/await, raw SQL -> query builder (if already in stack) |
| **Type safety** | `any`/`Object`/`interface{}` where a concrete type exists, stringly-typed enums, missing generics |
| **Readability** | Functions >50 lines, nesting >3 levels, magic numbers, unclear variable names |
| **Architecture** | God modules (>5 responsibilities), circular dependencies, layer violations (IO in pure) |
| **North stars** | Patterns from the locked stack that aren't being used yet, framework features being reinvented |

---

## Phase 3: Triage

Classify every finding: `critical` (architectural rot compounding) | `high` (significant quality drag) | `medium` (worth improving) | `low` (minor upgrade) | `note` (north star for later).

**Deduplication:** Same pattern across files = one grouped finding. **Confidence:** `certain` | `likely` | `possible`.

---

## Phase 4: Report

Present findings grouped by severity (critical first). Finding IDs: `I-NNN`. Each finding: category, one-line summary, file:line, confidence, 2-3 sentence detail, suggested fix.

**Actions menu:** Save (write rage-run log) | Focus (deep-dive a finding by ID — show ±20 lines, root cause, concrete fix, ask Apply/Skip/Back) | Dismiss (remove from report, log as `dismissed — <reason>`) | Rerun (different mode/scope) | Done.

---

## Phase 5: Save Rage Run

Write to `monke-docs/rage-run/<YYYY-MM-DD>-improv-<short-scope>.md`. Use template at `monke-docs/rage-run/template.md`. Create directory if needed. **Confirm filename.**

---

## Phase 6: Next Steps

"Prioritize architecture findings — they compound. Consider raising an OQ for north-star items: `/monke-design:oq`"

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
