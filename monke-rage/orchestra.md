# WyrdMonke Rage:Orchestra — Pick Your Fury

> **Usage:** Copy `monke-rage/` to `.claude/commands/monke-rage/`. Invoke: `/monke-rage:orchestra [scope]`
>
> The entrypoint for all rage scans. Presents the menu, you pick the mood, monke hunts.

---

## Arguments

`$ARGUMENTS` parsing:
- Single positional: scope limiter — passed through to whichever mode runs (default: entire project)
- Example: `/monke-rage:orchestra src/api`

```
SCOPE="${ARGUMENTS:-}"
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

- Inside a git repository
- At least one source file or doc file exists in scope

---

## The Menu

Present the 6 rage modes:

| Mode | What monke hunts | Monke mood |
|------|-----------------|------------|
| `buggy` | Bugs, logic errors, broken contracts, silent failures, race conditions | Angry monke smells something wrong |
| `improv` | Improvements, north stars, performance wins, API ergonomics | Ambitious monke sees the mountain |
| `renounce` | Redundancy, dead weight, duplication, over-abstraction, cargo cult | Minimalist monke with a machete |
| `haunt` | Security vulnerabilities, injection, auth gaps, secret leaks | Paranoid monke checking the locks |
| `drift` | Spec-code divergence — HLD/LLD vs code, stale status, ADR compliance | Auditor monke with a magnifying glass |
| `echo` | Dead code, unreachable paths, unused exports, orphaned files | Archaeologist monke sweeping the tomb |

```
Pick a mode (or "all" to run every mode on the same scope):
```

**Wait for user to pick.**

---

## Run the Mode

Once user picks a mode:

1. **Single mode** — Execute that mode's full logic inline (follow `buggy.md`, `improv.md`, `renounce.md`, `haunt.md`, `drift.md`, or `echo.md` respectively). Pass `SCOPE` through.

2. **"all"** — Run all 6 modes sequentially on the same scope. Present each report as it completes. Save each rage-run log individually.

For `drift`: check prerequisites first. If `hld.md` / `lld/*.md` / `monke-status.md` don't exist, **skip drift only** (continue with remaining modes) and note: "Drift skipped — no spec baseline. Run `/monke-recon:reconstruct` or `/monke-design:hld` first."

---

## After Completion

```
Rage scan complete.

Actions:
  1. Another mode — run a different mode on the same scope
  2. Change scope — pick a new scope and mode
  3. Done — close rage

Which?
```

If user picks another mode -> present the menu again (minus already-run modes). Loop until user says done.

---

## Anti-Patterns

| If tempted to... | Do instead... |
|------------------|--------------|
| Skip the menu and guess the mode | Present the menu. Let the human pick the fury. |
| Run all modes without asking | Only run all if user explicitly says "all". |
| Skip drift prerequisites silently | Warn and skip with explanation. Don't pretend it ran. |
| Merge reports from different modes | Each mode gets its own report and rage-run log. Keep them separate. |
| Auto-pick scope | Pass through what the user gave. If empty, scan everything. |
