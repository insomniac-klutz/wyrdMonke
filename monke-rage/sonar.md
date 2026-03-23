# WyrdMonke Rage:Sonar — Codebase Scanner That Finds What You're Ignoring

> **Usage:** Copy `monke-rage/` to `.claude/commands/monke-rage/` (project-local). Invoke: `/monke-rage:sonar <mode> [scope]`

---

## Arguments

`$ARGUMENTS` parsing:
- First positional: scan mode (**required**)
- Second positional: scope limiter — subdirectory, component name, glob, or `docs` for docs-only (default: entire project)
- Example: `/monke-rage:sonar buggy src/api`
- Example: `/monke-rage:sonar drift`
- Example: `/monke-rage:sonar renounce monke-docs/`

```
MODE="${1:?Mode required — buggy | improv | renounce | haunt | drift | echo}"
SCOPE="${2:-}"
```

If mode missing → present mode menu (see [Modes](#modes)).

---

## Modes

| Mode | What monke hunts | Monke mood |
|------|-----------------|------------|
| `buggy` | Bugs, logic errors, broken contracts, silent failures, off-by-ones, race conditions | angry monke smells something wrong |
| `improv` | Improvements, north stars, patterns that could be cleaner, performance wins, API ergonomics | ambitious monke sees the mountain |
| `renounce` | Redundancy, dead weight, duplicated logic, over-abstraction, tech debt, cargo-culted patterns | minimalist monke with a machete |
| `haunt` | Security vulnerabilities, injection surfaces, auth gaps, secret leaks, OWASP top 10, unsafe deserialization | paranoid monke checking the locks |
| `drift` | Spec-code divergence — HLD/LLD says X, code does Y, contracts mismatch, status.md is stale | auditor monke with a magnifying glass |
| `echo` | Dead code, unreachable paths, unused exports, orphaned files, zombie imports, vestigial config | archaeologist monke sweeping the tomb |

**⏸ If no mode provided:** Present the table above and ask user to pick.

---

## Prerequisites

- Inside a git repository
- At least one source file or doc file exists in scope

For `drift` mode additionally:
- `monke-docs/hld.md` exists (with confirmed sections)
- At least one `monke-docs/lld/*.md` exists
- `monke-status.md` exists

If drift prerequisites fail → tell user: "Nothing to drift against. Run `/monke-design:recon` or `/monke-design:hld` first to establish the spec baseline." Stop.

---

## Phase 1: Target Acquisition

### 1.1 Scope Resolution

| Input | Resolves to |
|-------|------------|
| Empty (no scope) | Entire project — all source files + all docs |
| Directory path (`src/api`) | All files under that directory |
| Component name (`auth-service`) | Files from that component's LLD file map (if LLD exists), else files matching the name |
| `docs` | All `monke-docs/` files + `CLAUDE.md` + `monke-status.md` |
| Glob (`**/*.rs`) | Files matching the glob |

### 1.2 File Census

Collect all files in scope. Categorize:

| Category | Examples |
|----------|---------|
| Source | `.rs`, `.ts`, `.py`, `.go`, `.js`, `.tsx`, `.jsx` |
| Test | Files in test dirs, `*_test.*`, `*.test.*`, `*.spec.*` |
| Config | `*.toml`, `*.json`, `*.yaml`, `*.yml`, `*.env*` |
| Doc | `*.md` in `monke-docs/`, `CLAUDE.md`, `README.md` |
| Spec | `monke-docs/*-specs.md`, `monke-docs/hld.md`, `monke-docs/lld/*.md` |
| Skill | `monke-*/` skill files |
| Other | Everything else in scope |

Present census:

```
Sonar target acquired:
  Mode: <mode>
  Scope: <scope description>
  Files: <N> source | <N> test | <N> config | <N> doc | <N> spec | <N> other
  Total: <N> files
```

**⏸ Confirm scope before scanning. Adjust / Confirm?**

---

## Phase 2: Scan

Execute mode-specific analysis across all files in scope. Read every file in scope — do not skip or sample.

### Agent Teams Check

If many files (>20 source files) and Agent Teams available:
- Spawn parallel scanner agents, one per file category or directory subtree
- Lead collects and deduplicates findings

If Agent Teams not available:
- Single-pass sequential scan

### Mode: `buggy` — Find Bugs

For each source file, check:

| Category | What to look for |
|----------|-----------------|
| **Logic** | Off-by-one, wrong comparison operator, negation errors, short-circuit mistakes, integer overflow/underflow |
| **Null/None** | Unguarded optional access, null dereference paths, missing nil checks |
| **Error handling** | Swallowed errors, bare `catch`/`except`, error paths that silently succeed, panics in library code |
| **Contracts** | Function that promises X in signature but returns Y, mismatched types at boundaries |
| **Concurrency** | Shared mutable state without sync, data races, deadlock patterns, async without await |
| **Resource** | Unclosed handles, leaked connections, missing cleanup in error paths |
| **Edge cases** | Empty collections, zero-length strings, negative numbers where unsigned expected, Unicode in ASCII-assumed code |

For each test file, check:
| Category | What to look for |
|----------|-----------------|
| **False pass** | Tests that assert on implementation detail instead of behavior, always-true assertions |
| **Flaky** | Time-dependent, order-dependent, shared mutable state across tests |
| **Coverage gap** | Public functions with no test, error paths untested |

### Mode: `improv` — Find Improvements

| Category | What to look for |
|----------|-----------------|
| **Performance** | O(n²) where O(n) is possible, redundant allocations, unbounded growth, N+1 queries |
| **API ergonomics** | Awkward call signatures, stringly-typed parameters, boolean traps, inconsistent naming |
| **Pattern upgrade** | Manual loops → iterators/streams, callback hell → async/await, raw SQL → query builder (if already in stack) |
| **Type safety** | `any`/`Object`/`interface{}` where a concrete type exists, stringly-typed enums, missing generics |
| **Readability** | Functions >50 lines, nesting >3 levels, magic numbers, unclear variable names |
| **Architecture** | God modules (>5 responsibilities), circular dependencies, layer violations (IO in pure) |
| **North stars** | Patterns from the locked stack that aren't being used yet, framework features being reinvented |

### Mode: `renounce` — Find Redundancy

| Category | What to look for |
|----------|-----------------|
| **Duplication** | Copy-pasted logic (>10 similar lines), reimplemented stdlib/library functions |
| **Dead weight** | Unused dependencies in manifest, unused feature flags, commented-out code blocks |
| **Over-abstraction** | Interfaces with one implementation, factories that create one thing, wrappers that add nothing |
| **Cargo cult** | Patterns copied without understanding — DTO layers with no transformation, repository pattern over an ORM that already is one |
| **Config bloat** | Redundant config keys, env vars set but never read, duplicate config across files |
| **Tech debt** | TODO/FIXME/HACK comments, suppressed lints with no explanation, version pins with no reason |
| **Vestigial** | Migration files for dropped tables still present, old API versions still routed, backwards-compat shims for removed features |

### Mode: `haunt` — Find Security Issues

| Category | What to look for |
|----------|-----------------|
| **Injection** | SQL injection (string concat in queries), XSS (unescaped user input in templates), command injection (user input in shell calls), path traversal |
| **Auth** | Missing auth checks on endpoints, hardcoded credentials, JWT without expiry validation, session fixation |
| **Secrets** | API keys/tokens/passwords in source, `.env` committed or not in `.gitignore`, secrets in logs |
| **Crypto** | Weak algorithms (MD5/SHA1 for security), ECB mode, hardcoded IVs, custom crypto |
| **Dependencies** | Known CVE patterns in import versions, unmaintained packages, typosquatting risks |
| **Data** | PII logged or stored unencrypted, CORS wildcard, missing rate limiting, verbose error messages to clients |
| **Deserialization** | Untrusted input deserialized without validation, pickle/eval on user data |

### Mode: `drift` — Find Spec-Code Divergence

| Category | What to look for |
|----------|-----------------|
| **HLD vs Code** | Components in HLD not in code (or vice versa), containers that don't match deploy reality |
| **LLD vs Code** | Functions in LLD not implemented, implemented functions not in LLD, signature mismatches |
| **Boundary matrix** | Contracts in S7 that don't match actual cross-module types, missing error types at boundaries |
| **Status staleness** | `monke-status.md` claims X but code/tests show Y, checkboxes wrong |
| **Phase plan** | Components marked "waiting" that have code, components marked "IL-3 passed" with failing tests |
| **ADR compliance** | Decisions recorded in ADRs not reflected in code (chose Option A, built Option B) |
| **Test plan vs tests** | Test plan rows in LLD with no corresponding test file/function |

### Mode: `echo` — Find Dead Code

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

Classify every finding by severity:

| Severity | Symbol | Meaning | Action |
|----------|--------|---------|--------|
| **critical** | `🔴` | Actively broken or dangerous — ship-blocking | Fix now |
| **high** | `🟠` | Will cause pain soon — wrong contract, missing guard, real vulnerability | Fix this sprint |
| **medium** | `🟡` | Quality drag — duplication, tech debt, stale spec, dead code with confusion risk | Schedule fix |
| **low** | `🟢` | Nice to have — readability, minor perf, cosmetic | Fix if nearby |
| **note** | `📝` | Observation, not a problem — north star, pattern suggestion, future consideration | Record for later |

### Deduplication

If the same finding appears in multiple files (e.g., same pattern copied across modules), group into one finding with a file list rather than N separate findings.

### Confidence

Rate each finding's confidence:

| Confidence | Meaning |
|------------|---------|
| **certain** | Verifiable from code alone — the bug is there, the dead code is dead |
| **likely** | Strong signal but context-dependent — might be intentional |
| **possible** | Pattern match, could be false positive — needs human judgment |

---

## Phase 4: Report

### 4.1 Present Findings

Present findings grouped by severity (critical first), then by category within severity:

```
## Sonar Report: <mode>
Scope: <scope> | Files scanned: <N> | Date: <today>

### 🔴 Critical (<count>)

**[B-001]** <category>: <one-line summary>
  File: <path>:<line>
  Confidence: certain | likely | possible
  Detail: <2-3 sentences explaining the finding and why it matters>
  Suggested fix: <concrete action>

**[B-002]** ...

### 🟠 High (<count>)
...

### 🟡 Medium (<count>)
...

### 🟢 Low (<count>)
...

### 📝 Notes (<count>)
...

---
Summary: <critical> critical | <high> high | <medium> medium | <low> low | <notes> notes
Top recommendation: <the single most impactful thing to fix>
```

Finding IDs use mode prefix: `B-NNN` (buggy), `I-NNN` (improv), `R-NNN` (renounce), `H-NNN` (haunt), `D-NNN` (drift), `E-NNN` (echo).

**⏸ Present full report. Ask user:**

```
Actions:
  1. Save    — write rage-run log to monke-docs/rage-run/
  2. Focus   — deep-dive on a specific finding (give ID)
  3. Dismiss — mark specific findings as intentional/wontfix
  4. Rerun   — scan again with different mode or scope
  5. Done    — close sonar

Which?
```

### 4.2 Focus (if requested)

If user picks Focus on a finding:
- Read the file(s) involved
- Show surrounding context (±20 lines)
- Explain the root cause in detail
- Show a concrete fix (code diff or doc edit)
- Ask: "Apply fix / Skip / Back to report?"

If user says Apply → make the edit, re-run linter/tests if source code was changed.

**⏸ After focus, return to the actions menu.**

### 4.3 Dismiss (if requested)

If user dismisses a finding:
- Remove from report
- Note in rage-run log as `dismissed — <reason>`
- Recalculate summary counts

---

## Phase 5: Save Rage Run

Write the rage-run log to `monke-docs/rage-run/`. Filename format:

```
<YYYY-MM-DD>-<mode>-<short-scope>.md
```

Examples:
- `2026-03-23-buggy-full.md`
- `2026-03-23-drift-auth-service.md`
- `2026-03-23-haunt-src-api.md`

Use the template at `monke-docs/rage-run/template.md` as the skeleton. Fill all fields.

If `monke-docs/rage-run/` doesn't exist → create it.

**⏸ Confirm save location and filename.**

Report saved. Show path.

---

## Phase 6: Suggest Next Steps

Based on mode and findings:

| Mode | Suggestions |
|------|------------|
| `buggy` | "Fix critical bugs first. Consider `/monke-test:test-run unit` to verify fixes don't regress." |
| `improv` | "Prioritize architecture findings — they compound. Consider raising an OQ for north-star items: `/monke-design:oq`" |
| `renounce` | "Start with dead weight — safest removals. Run `/monke-rage:sonar echo` to find more dead code." |
| `haunt` | "Critical security findings are ship-blockers. Consider `/monke-design:oq` for findings that need design changes." |
| `drift` | "Update specs to match code OR update code to match specs — pick a direction for each finding. Run `/monke-status:status rebuild` to refresh the dashboard." |
| `echo` | "Remove orphaned files first (lowest risk). Then unused exports. Run tests after each removal batch." |

Cross-mode suggestion: "Run `/monke-rage:sonar` in another mode for a different lens on the same scope."

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Scan without presenting findings first | Refuse. Sonar always pauses for triage review. |
| Auto-fix all findings without confirmation | Refuse. Each fix is a decision — monke presents, human decides. |
| Suppress findings to make the report look clean | Refuse. Honesty over vanity. Dismissed findings are logged, not deleted. |
| Skip files because "they're probably fine" | Refuse. Sonar reads every file in scope. Sampling is lying. |
| Rate everything as critical to scare the user | Refuse. Severity must be honest. A style nit is not a ship-blocker. |
| Ignore test files | Refuse. Broken tests are as bad as broken code. |

---

## Status Update

Sonar does NOT update `monke-status.md` directly — it's a read-only scanner. Instead:

- If `drift` mode found status staleness → suggest: "Run `/monke-status:status rebuild` to fix dashboard."
- If any finding blocks implementation → suggest raising an OQ: `/monke-design:oq`
- If findings warrant an ADR → suggest: `/monke-design:adr`

The rage-run log in `monke-docs/rage-run/` IS the persistent artifact. It's the receipt, not the fix.
