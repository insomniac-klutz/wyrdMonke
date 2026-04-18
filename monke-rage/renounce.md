# WyrdMonke Rage:Renounce — Minimalist Monke With a Machete

> **Usage:** `/monke-rage:renounce [scope]`
>
> Hunts redundancy, dead weight, duplicated logic, over-abstraction, cargo-culted patterns, tech debt. If it doesn't earn its place, monke cuts it.
>
> **SRP:** Semantic redundancy — code that means the same thing twice, duplicated intent, over-abstraction. For referentially-dead code (unused imports, orphaned files, unreachable paths), use `/monke-rage:echo`.

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
  Mode: renounce | Scope: <description> | Files: <N> total
```

⏸ **PG-1 [HARD] — Scope confirmed before scanning.** Always surfaces. Never skippable. Renounce can recommend deletions — scope limits what's at risk of getting cut.
Confirm / Adjust / Reject?

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

Write to `monke-docs/rage-runs/<YYYY-MM-DD>-renounce-<short-scope>.md`. Use template at `monke-docs/rage-run/template.md`. Create directory if needed.

⏸ **PG-11 [SOFT] — Filename confirmed before write.** Auto-pass when: scope slug is unambiguous (single directory/component, not a glob) AND no existing rage-run collides with the slug for today. Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds.
Confirm / Adjust / Reject?

---

## Phase 6: Next Steps

"Start with dead weight — safest removals. Run `/monke-rage:echo` to find more dead code."

Cross-mode: "Run `/monke rage:<mode>` (buggy | improv | haunt | drift | echo) to scan with a different lens on the same scope."

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Recommend deleting code whose tests don't cover it | Refuse. No coverage, no confession. A renounce finding on untested code must be `note` at most, with a test-coverage prerequisite before any delete is proposed. |
| Flag referentially-dead imports or orphan files here | Refuse. That's echo's lane. Point the user at `/monke-rage:echo` and keep this mode focused on semantic redundancy (duplicated intent, over-abstraction). |
| Collapse a single-implementation interface without checking external consumers | Refuse. An interface with one local implementation may be satisfying a public contract (library boundary, plugin system, LSP seam). Check usages before proposing the collapse. |
| Treat all `TODO`/`FIXME` comments as tech debt to cut | Refuse. Some TODOs mark live work. Match each TODO against the commit log and open blockers before recommending removal. |
| Recommend merging two "similar-looking" functions without behavioral proof | Refuse. Similar shapes can hide diverging intent. Diff the behavior on representative inputs before proposing a merge. |
| Rate every backwards-compat shim as `critical` to remove | Refuse. Shims exist for consumers you don't own. Downgrade to `medium`/`low` unless there's evidence no consumer relies on them. |

---

## Context Death Protocol

**Checkpoint artifacts (written on context pressure):**
- `monke-docs/rage-runs/<date>-renounce-<scope>.draft.md` — partial redundancy findings with triage state.
- Per-file progress ledger noting scanned vs pending files.

**Status line marker:** `Where We Are:` in `monke-status.md` reads `rage:renounce — <scope> (<N>/<M> files scanned, <K> redundancies)` while mid-flight.

**Recovery detection (on entry):**
- If `monke-status.md` Resume block names `/monke-rage:renounce` AND draft rage-run exists → resume at Phase 2 from last unscanned file.
- If draft exists but marker cleared → verify scope matches, continue from Phase 3 triage.
- If neither present → start fresh from Phase 1.

---

## Status Update

**Read on entry:** `monke-status.md` — check current phase and whether scope contains components without test coverage (those findings must be downgraded to `note`).

**Write on exit:**
- **Success:** bump `Updated:` with today's date + `by /monke-rage:renounce`. Append to Gate Audit Log: `- RAGE renounce <scope> — <N critical / K high / ...> (see <path>)`. For deletions flagged `high`/`critical`, suggest user run tests before accepting.
- **Blocked:** if scope failed or prerequisites missing, add a row to Open Blockers with WHAT/WHY/HOW.
- **Partial:** write a `Resume:` block with phase + scanned-file ledger for context-death recovery.
