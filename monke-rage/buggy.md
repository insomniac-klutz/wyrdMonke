# WyrdMonke Rage:Buggy — Angry Monke Smells Something Wrong

> **Usage:** `/monke-rage:buggy [scope]`
>
> Hunts bugs, logic errors, broken contracts, silent failures, off-by-ones, race conditions. If it's broken, monke finds it.

---

## Arguments

`$ARGUMENTS` parsing:
- **First positional:** `SCOPE` — path or glob to scan (e.g., `src/api`, `**/*.ts`), component name, or `docs`. Defaults to entire project.
- **Second positional (optional):** `resume:<N>` — re-enter at phase `<N>` using checkpoint artifact (see drafter §7). Phase numbers: 1=target acquisition, 2=scan, 3=triage, 4=report, 5=save.

```
SCOPE="${1:-${ARGUMENTS:-}}"
RESUME_PHASE="$(echo "$2" | grep -oE '^resume:[0-9]+$' | cut -d: -f2)"
```

If `RESUME_PHASE` is set, skip to Phase `<N>` with prerequisites re-validated inline and the draft lock-file re-read per Context Death Protocol.

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
  Mode: buggy | Scope: <description> | Files: <N> total
```

⏸ **PG-1 [HARD] — Scope confirmed before scanning.** Always surfaces. Never skippable. Scope determines blast radius — scanning the wrong tree wastes hours and produces noise.
Confirm / Adjust / Reject?

---

## Phase 2: Scan — Find Bugs

Read every file in scope — do not skip or sample. If >20 source files and Agent Teams available, spawn parallel scanners per directory subtree.

**Write lock-file.** As files are scanned, update `monke-docs/rage-runs/<date>-buggy-<scope>.draft.md.lock` per drafter §7 after every batch (every 10 files, or every finding group). Capture `phase: 2`, `progress: <files-scanned>/<total-files>`, timestamp. This persists mid-scan progress across context death.

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

Write to `monke-docs/rage-runs/<YYYY-MM-DD>-buggy-<short-scope>.md`. Use template at `monke-docs/rage-run/template.md`. Create directory if needed.

⏸ **PG-11 [SOFT] — Filename confirmed before write.** Auto-pass when: scope slug is unambiguous (single directory/component, not a glob) AND no existing rage-run collides with the slug for today. Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds.
Confirm / Adjust / Reject?

**Delete lock-file.** After the final rage-run is written, delete `monke-docs/rage-runs/<date>-buggy-<scope>.draft.md.lock`. Stale lock-files confuse future recovery.

---

## Phase 6: Next Steps

"Fix critical bugs first. Consider `/monke-test:test-run unit` to verify fixes don't regress."

Cross-mode: "Run `/monke rage:<mode>` (improv | renounce | haunt | drift | echo) to scan with a different lens on the same scope."

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Chase every compiler warning as a bug | Refuse. Warnings are hints, not findings. Lean on the Source File Checklist — logic, null, contracts, concurrency — not the linter's mood. |
| Flag `TODO`/`FIXME` comments as bugs | Refuse. That's renounce/echo territory. Buggy hunts behavior that contradicts the contract, not housekeeping. |
| Treat style/formatting drift as a bug finding | Refuse. Formatting isn't a bug. If code runs correctly but looks ugly, that's improv. |
| Escalate flaky tests to `critical` | Refuse. Flaky ≠ broken code. Log under the test-file checklist at the honest severity the flake deserves. |
| Skip concurrency/race analysis because "it's hard" | Refuse. Concurrency bugs are where silent failures live. That's the whole point of this mode. |
| Report a symptom instead of the offending file:line | Refuse. A finding without a citation is a rumor. Give the path, the line, and the confidence. |

---

## Context Death Protocol

**Checkpoint artifacts (written on context pressure):**
- `monke-docs/rage-runs/<date>-buggy-<scope>.draft.md` — partial findings with triage state, so a later invocation can resume triage without re-scanning.
- `monke-docs/rage-runs/<date>-buggy-<scope>.draft.md.lock` — phase + progress ledger per drafter §7. Tracks files-scanned count and last-batch timestamp.
- Per-file progress ledger noting which files have been scanned vs pending (folded into the lock-file's `progress:` field).

**Status line marker:** `Where We Are: rage:buggy — <scope> (<N>/<M> files scanned, <K> findings)` while mid-flight.

**Recovery detection (on entry):**
- If `monke-status.md` Resume block names `/monke-rage:buggy` AND draft rage-run exists → read the lock-file (authoritative — if it's inconsistent with the draft, regenerate from scratch) and resume at Phase 2 from last unscanned file.
- If draft exists but marker cleared → verify scope matches, continue from Phase 3 triage.
- If neither present → start fresh from Phase 1.

---

## Status Update

**Read on entry:** `monke-status.md` — check current phase, any in-flight rage-run markers, and whether scope intersects a component mid-implementation (findings may need to wait).

**Write on exit:**
- **Success:** bump `Updated:` with today's date + `by /monke-rage:buggy`. Append rage-run log reference to Gate Audit Log: `- RAGE buggy <scope> — <N critical / K high / ...> (see <path>)`. If critical findings exist, add them to Open Blockers.
- **Blocked:** if scope couldn't be resolved or prerequisites failed, add a row to Open Blockers table with WHAT/WHY/HOW.
- **Partial:** write a `Resume:` block with phase + scanned-file ledger so the next invocation picks up cleanly.
