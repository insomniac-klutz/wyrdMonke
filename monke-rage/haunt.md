# WyrdMonke Rage:Haunt — Paranoid Monke Checking the Locks

> **Usage:** `/monke-rage:haunt [scope]`
>
> Hunts security vulnerabilities, injection surfaces, auth gaps, secret leaks, OWASP top 10, unsafe deserialization. If it's exposed, monke finds the crack.

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
  Mode: haunt | Scope: <description> | Files: <N> total
```

⏸ **PG-1 [HARD] — Scope confirmed before scanning.** Always surfaces. Never skippable. Security findings drive ship-blocking decisions — scope determines which attack surface is in scope.
Confirm / Adjust / Reject?

---

## Phase 2: Scan — Find Security Issues

Read every file in scope — do not skip or sample. If >20 source files and Agent Teams available, spawn parallel scanners per directory subtree.

### Security Checklist

| Category | What to look for |
|----------|-----------------|
| **Injection** | SQL injection (string concat in queries), XSS (unescaped user input in templates), command injection (user input in shell calls), path traversal |
| **Auth** | Missing auth checks on endpoints, hardcoded credentials, JWT without expiry validation, session fixation |
| **Secrets** | API keys/tokens/passwords in source, `.env` committed or not in `.gitignore`, secrets in logs |
| **Crypto** | Weak algorithms (MD5/SHA1 for security), ECB mode, hardcoded IVs, custom crypto |
| **Dependencies** | Known CVE patterns in import versions, unmaintained packages, typosquatting risks |
| **Data** | PII logged or stored unencrypted, CORS wildcard, missing rate limiting, verbose error messages to clients |
| **Deserialization** | Untrusted input deserialized without validation, pickle/eval on user data |

---

## Phase 3: Triage

Classify every finding: `critical` (actively exploitable, ship-blocker) | `high` (real vulnerability, needs conditions) | `medium` (weakness, exploitable with effort) | `low` (hardening opportunity) | `note` (security posture suggestion).

**Deduplication:** Same pattern across files = one grouped finding. **Confidence:** `certain` | `likely` | `possible`.

---

## Phase 4: Report

Present findings grouped by severity (critical first). Finding IDs: `H-NNN`. Each finding: category, one-line summary, file:line, confidence, 2-3 sentence detail, suggested fix.

**Actions menu:** Save (write rage-run log) | Focus (deep-dive a finding by ID — show ±20 lines, root cause, concrete fix, ask Apply/Skip/Back) | Dismiss (remove from report, log as `dismissed — <reason>`) | Rerun (different mode/scope) | Done.

---

## Phase 5: Save Rage Run

Write to `monke-docs/rage-runs/<YYYY-MM-DD>-haunt-<short-scope>.md`. Use template at `monke-docs/rage-run/template.md`. Create directory if needed.

⏸ **PG-11 [SOFT] — Filename confirmed before write.** Auto-pass when: scope slug is unambiguous (single directory/component, not a glob) AND no existing rage-run collides with the slug for today. Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds.
Confirm / Adjust / Reject?

---

## Phase 6: Next Steps

"Critical security findings are ship-blockers. Consider `/monke-design:oq` for findings that need design changes."

Cross-mode: "Run `/monke rage:<mode>` (buggy | improv | renounce | drift | echo) to scan with a different lens on the same scope."

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Flag every string concat as SQL injection | Refuse. Context matters — a concatenation over a trusted constant is not an injection. Trace the taint from the actual request surface; if no user-controlled input reaches it, downgrade the finding. |
| Mark all missing auth as `critical` without checking context | Refuse. Health-check and public documentation endpoints legitimately skip auth. Check the route's intended exposure before scoring. |
| Treat `.env.example` files as leaked secrets | Refuse. Placeholder/example files are convention, not a leak. Only flag actual secrets, `.env` without `.gitignore`, or hardcoded tokens in source. |
| Assume CORS wildcard is always a vulnerability | Refuse. A wildcard on a public read-only API is intentional; on an authenticated endpoint it's a disaster. Classify by endpoint role, not by string match. |
| Rate MD5 in a non-security context (checksum, cache key) as `critical` | Refuse. MD5 for integrity-only use is fine. Flag ONLY where MD5 is used for passwords, tokens, or security boundaries. |
| Suppress findings because "it's legacy" or "already filed" | Refuse. The rage-run is the honest record. If a finding is accepted risk, say so explicitly with a citation, don't silently drop it. |

---

## Context Death Protocol

**Checkpoint artifacts (written on context pressure):**
- `monke-docs/rage-runs/<date>-haunt-<scope>.draft.md` — partial vulnerability findings with taint-trace state.
- Per-file progress ledger noting scanned vs pending files.

**Status line marker:** `Where We Are:` in `monke-status.md` reads `rage:haunt — <scope> (<N>/<M> files scanned, <K> findings)` while mid-flight.

**Recovery detection (on entry):**
- If `monke-status.md` Resume block names `/monke-rage:haunt` AND draft rage-run exists → resume at Phase 2 from last unscanned file.
- If draft exists but marker cleared → verify scope matches, continue from Phase 3 triage.
- If neither present → start fresh from Phase 1.

---

## Status Update

**Read on entry:** `monke-status.md` — check current phase and whether the project is pre-ship (critical findings block ship) or post-ship (findings still matter but don't block existing traffic).

**Write on exit:**
- **Success:** bump `Updated:` with today's date + `by /monke-rage:haunt`. Append to Gate Audit Log: `- RAGE haunt <scope> — <N critical / K high / ...> (see <path>)`. All `critical` findings must be added to Open Blockers (ship-blocker class).
- **Blocked:** if scope failed or prerequisites missing, add a row to Open Blockers with WHAT/WHY/HOW.
- **Partial:** write a `Resume:` block with phase + scanned-file ledger for context-death recovery.
