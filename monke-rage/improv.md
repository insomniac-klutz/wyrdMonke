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

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

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

⏸ **PG-1 [HARD] — Scope confirmed before scanning.** Always surfaces. Never skippable. Improv can propose rewrites — scope sets the blast radius before ambition leaks outside it.
Confirm / Adjust / Reject?

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

Write to `monke-docs/rage-runs/<YYYY-MM-DD>-improv-<short-scope>.md`. Use template at `monke-docs/rage-run/template.md`. Create directory if needed.

⏸ **PG-11 [SOFT] — Filename confirmed before write.** Auto-pass when: scope slug is unambiguous (single directory/component, not a glob) AND no existing rage-run collides with the slug for today. Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds.
Confirm / Adjust / Reject?

---

## Phase 6: Next Steps

"Prioritize architecture findings — they compound. Consider raising an OQ for north-star items: `/monke-design:oq`"

Cross-mode: "Run `/monke rage:<mode>` (buggy | renounce | haunt | drift | echo) to scan with a different lens on the same scope."

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Propose a rewrite without measurement | Refuse. Improv findings are hypotheses — cite the concrete drag (O(n²) loop, 400-line function, boolean trap) before suggesting the upgrade. No vibes-based refactors. |
| Recommend patterns the locked stack doesn't use | Refuse. Improv pulls from the locked stack's idioms, not the JavaScript-of-the-month club. If project-specs S2 says "no SQLAlchemy," do not propose it. |
| Score "more Rust-like" as a quality win | Refuse. Aesthetic preference is not an improvement. Tie every finding to performance, ergonomics, type safety, or readability — something measurable. |
| Bundle ten upgrades into one finding | Refuse. One concern per finding so the user can triage. "This module has issues" isn't actionable — "this 120-line function has nesting depth 5 and a magic constant 17" is. |
| Rank cosmetic renames as `high` | Refuse. Rename fatigue is a cost. Keep renames at `low`/`note` unless the current name actively misleads. |
| Suggest north-stars the team can't afford | Refuse. If the upgrade requires a migration the roadmap doesn't fund, log it as `note` for future consideration — not as a required change. |

---

## Context Death Protocol

**Checkpoint artifacts (written on context pressure):**
- `monke-docs/rage-runs/<date>-improv-<scope>.draft.md` — partial improvement findings with triage state.
- Per-file progress ledger noting scanned vs pending files.

**Status line marker:** `Where We Are:` in `monke-status.md` reads `rage:improv — <scope> (<N>/<M> files scanned, <K> upgrades)` while mid-flight.

**Recovery detection (on entry):**
- If `monke-status.md` Resume block names `/monke-rage:improv` AND draft rage-run exists → resume at Phase 2 from last unscanned file.
- If draft exists but marker cleared → verify scope matches, continue from Phase 3 triage.
- If neither present → start fresh from Phase 1.

---

## Status Update

**Read on entry:** `monke-status.md` — check current phase and whether scope overlaps components still at pre-L0/L0 (improvements should wait until implementation stabilizes).

**Write on exit:**
- **Success:** bump `Updated:` with today's date + `by /monke-rage:improv`. Append to Gate Audit Log: `- RAGE improv <scope> — <N critical / K high / ...> (see <path>)`. High-severity architectural findings should be raised as OQs via `/monke-design:oq`.
- **Blocked:** if scope failed or prerequisites missing, add a row to Open Blockers with WHAT/WHY/HOW.
- **Partial:** write a `Resume:` block with phase + scanned-file ledger for context-death recovery.
