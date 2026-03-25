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

**Confirm scope before scanning.**

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

Write to `monke-docs/rage-runs/<YYYY-MM-DD>-haunt-<short-scope>.md`. Use template at `monke-docs/rage-run/template.md`. Create directory if needed. **Confirm filename.**

---

## Phase 6: Next Steps

"Critical security findings are ship-blockers. Consider `/monke-design:oq` for findings that need design changes."

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
