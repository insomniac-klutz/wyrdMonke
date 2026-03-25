# WyrdMonke Rage:Echo — Archaeologist Monke Sweeping the Tomb

> **Usage:** `/monke-rage:echo [scope]`
>
> Hunts dead code, unreachable paths, unused exports, orphaned files, zombie imports, vestigial config. If nothing calls it, monke buries it.

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

---

## Phase 1: Target Acquisition

Resolve scope: empty = full project | directory path = all files under it | component name = LLD file map match | `docs` = `monke-docs/` + `CLAUDE.md` + `monke-status.md` | glob = matching files.

Collect and categorize all files in scope (source, test, config, doc, spec, skill, other). Present census:

```
Sonar target acquired:
  Mode: echo | Scope: <description> | Files: <N> total
```

**Confirm scope before scanning.**

---

## Phase 2: Scan — Find Dead Code

Read every file in scope — do not skip or sample. If >20 source files and Agent Teams available, spawn parallel scanners per directory subtree.

### Dead Code Checklist

| Category | What to look for |
|----------|-----------------|
| **Unreachable** | Code after unconditional return/throw/panic, dead branches (condition always true/false) |
| **Unused exports** | Public functions/types with no external imports, exported but never consumed |
| **Orphaned files** | Files not imported by anything, not referenced in any config, not in any LLD file map |
| **Zombie imports** | Imports that are never used (after removing commented code) |
| **Phantom config** | Config keys read nowhere, env vars defined but never accessed |
| **Ghost routes** | API endpoints defined but unreachable from any client, handlers never wired |
| **Abandoned tests** | Skipped/ignored tests with no issue reference, test files for deleted source |

---

## Phase 3: Triage

Classify every finding: `critical` (dead code causing confusion or masking bugs) | `high` (orphaned files misleading developers) | `medium` (unused exports, zombie imports) | `low` (single unused variable) | `note` (might be dead, needs human judgment).

**Deduplication:** Same pattern across files = one grouped finding. **Confidence:** `certain` | `likely` | `possible`.

---

## Phase 4: Report

Present findings grouped by severity (critical first). Finding IDs: `E-NNN`. Each finding: category, one-line summary, file:line, confidence, 2-3 sentence detail, suggested fix.

**Actions menu:** Save (write rage-run log) | Focus (deep-dive a finding by ID — show ±20 lines, root cause, concrete fix, ask Apply/Skip/Back) | Dismiss (remove from report, log as `dismissed — <reason>`) | Rerun (different mode/scope) | Done.

---

## Phase 5: Save Rage Run

Write to `monke-docs/rage-runs/<YYYY-MM-DD>-echo-<short-scope>.md`. Use template at `monke-docs/rage-run/template.md`. Create directory if needed. **Confirm filename.**

---

## Phase 6: Next Steps

"Remove orphaned files first (lowest risk). Then unused exports. Run tests after each removal batch."

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
