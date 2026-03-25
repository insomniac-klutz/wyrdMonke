# WyrdMonke Rage:Buggy — Angry Monke Smells Something Wrong

> **Usage:** `/monke-rage:buggy [scope]`
>
> Hunts bugs, logic errors, broken contracts, silent failures, off-by-ones, race conditions. If it's broken, monke finds it.

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
  Mode: buggy | Scope: <description> | Files: <N> total
```

**Confirm scope before scanning.**

---

## Phase 2: Scan — Find Bugs

Read every file in scope — do not skip or sample. If >20 source files and Agent Teams available, spawn parallel scanners per directory subtree.

### Source File Checklist

| Category | What to look for |
|----------|-----------------|
| **Logic** | Off-by-one, wrong comparison operator, negation errors, short-circuit mistakes, integer overflow/underflow |
| **Null/None** | Unguarded optional access, null dereference paths, missing nil checks |
| **Error handling** | Swallowed errors, bare `catch`/`except`, error paths that silently succeed, panics in library code |
| **Contracts** | Function that promises X in signature but returns Y, mismatched types at boundaries |
| **Concurrency** | Shared mutable state without sync, data races, deadlock patterns, async without await |
| **Resource** | Unclosed handles, leaked connections, missing cleanup in error paths |
| **Edge cases** | Empty collections, zero-length strings, negative numbers where unsigned expected, Unicode in ASCII-assumed code |

### Test File Checklist

| Category | What to look for |
|----------|-----------------|
| **False pass** | Tests that assert on implementation detail instead of behavior, always-true assertions |
| **Flaky** | Time-dependent, order-dependent, shared mutable state across tests |
| **Coverage gap** | Public functions with no test, error paths untested |

---

## Phase 3: Triage

Classify every finding: `critical` (ship-blocking) | `high` (fix this sprint) | `medium` (schedule fix) | `low` (fix if nearby) | `note` (record for later).

**Deduplication:** Same pattern across files = one grouped finding. **Confidence:** `certain` | `likely` | `possible`.

---

## Phase 4: Report

Present findings grouped by severity (critical first). Finding IDs: `B-NNN`. Each finding: category, one-line summary, file:line, confidence, 2-3 sentence detail, suggested fix.

**Actions menu:** Save (write rage-run log) | Focus (deep-dive a finding by ID — show ±20 lines, root cause, concrete fix, ask Apply/Skip/Back) | Dismiss (remove from report, log as `dismissed — <reason>`) | Rerun (different mode/scope) | Done.

---

## Phase 5: Save Rage Run

Write to `monke-docs/rage-runs/<YYYY-MM-DD>-buggy-<short-scope>.md`. Use template at `monke-docs/rage-run/template.md`. Create directory if needed. **Confirm filename.**

---

## Phase 6: Next Steps

"Fix critical bugs first. Consider `/monke-test:test-run unit` to verify fixes don't regress."

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
