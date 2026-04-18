# WyrdMonke Rage:Echo — Archaeologist Monke Sweeping the Tomb

> **Usage:** `/monke-rage:echo [scope]`
>
> Hunts dead code, unreachable paths, unused exports, orphaned files, zombie imports, vestigial config. If nothing calls it, monke buries it.
>
> **SRP:** Referential deadness — code nothing reaches (zombie imports, orphan files, unreachable branches). For semantic redundancy (duplicated intent, over-abstraction), use `/monke-rage:renounce`.

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

⏸ **PG-1 [HARD] — Scope confirmed before scanning.** Always surfaces. Never skippable. Echo can recommend file-level deletions — scope limits what is at risk of being buried.
Confirm / Adjust / Reject?

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

Write to `monke-docs/rage-runs/<YYYY-MM-DD>-echo-<short-scope>.md`. Use template at `monke-docs/rage-run/template.md`. Create directory if needed.

⏸ **PG-11 [SOFT] — Filename confirmed before write.** Auto-pass when: scope slug is unambiguous (single directory/component, not a glob) AND no existing rage-run collides with the slug for today. Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds.
Confirm / Adjust / Reject?

---

## Phase 6: Next Steps

"Remove orphaned files first (lowest risk). Then unused exports. Run tests after each removal batch."

Cross-mode: "Run `/monke rage:<mode>` (buggy | improv | renounce | haunt | drift) to scan with a different lens on the same scope."

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Delete code that's exported as a public API | Refuse. Exported ≠ internal-reachable. Check the public-API contract (package exports, plugin hooks, LSP boundary) before recommending removal. If exported, downgrade to `note` with a "verify no external consumers" action. |
| Mark reflection/dynamic-dispatch targets as dead | Refuse. Language reflection, DI containers, and plugin loaders resolve names at runtime. Grepping for usage misses these. Downgrade and flag for human verification. |
| Flag semantic duplication as dead code | Refuse. That's renounce's lane. Echo only cuts what nothing references; duplication needs `/monke-rage:renounce`. |
| Propose removing a test for "deleted source" without checking rename history | Refuse. The source may have moved, not died. Check `git log --follow` / rename tracking before recommending test removal. |
| Treat `#[allow(dead_code)]` / `// eslint-disable unused` as dead code | Refuse. The suppression is a signal the code is intentionally preserved (trait conformance, future use, platform-conditional). Respect the signal unless the suppression is stale. |
| Delete unreachable branches without checking platform/feature gates | Refuse. "Unreachable on this platform" ≠ "unreachable everywhere." Check `cfg`/`#ifdef`/feature-flag guards before recommending removal. |

---

## Context Death Protocol

**Checkpoint artifacts (written on context pressure):**
- `monke-docs/rage-runs/<date>-echo-<scope>.draft.md` — partial dead-code findings with reachability-trace state.
- Per-file progress ledger noting scanned vs pending files.

**Status line marker:** `Where We Are:` in `monke-status.md` reads `rage:echo — <scope> (<N>/<M> files scanned, <K> dead-code findings)` while mid-flight.

**Recovery detection (on entry):**
- If `monke-status.md` Resume block names `/monke-rage:echo` AND draft rage-run exists → resume at Phase 2 from last unscanned file.
- If draft exists but marker cleared → verify scope matches, continue from Phase 3 triage.
- If neither present → start fresh from Phase 1.

---

## Status Update

**Read on entry:** `monke-status.md` — check current phase and whether scope includes components with active-but-pending work (unused exports may be pending consumer wiring).

**Write on exit:**
- **Success:** bump `Updated:` with today's date + `by /monke-rage:echo`. Append to Gate Audit Log: `- RAGE echo <scope> — <N critical / K high / ...> (see <path>)`. Findings flagged for reflection/plugin/API exposure should include a "needs human verification" note.
- **Blocked:** if scope failed or prerequisites missing, add a row to Open Blockers with WHAT/WHY/HOW.
- **Partial:** write a `Resume:` block with phase + scanned-file ledger for context-death recovery.
