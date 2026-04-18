# WyrdMonke Flash:Pulse — Does it breathe?

> **Usage:** /monke-flash:pulse [flow]
>
> Runs the MVP, collects what's broken, sorts it into fix-now, adjust, defer — the smoke test that tells monke whether the build actually lives.

---

## Arguments

`$ARGUMENTS` parsing:
- First positional: specific flow to test, or `all` (default)
- Second positional: `resume:<N>` — skip to phase `<N>` with checkpoint re-read (see drafter §7). Phase numbers: 1=run, 2=collect feedback, 3=categorize, 4=fix loop, 5=convergence.
- Example: `/monke-flash:pulse "user search"` or `/monke-flash:pulse all resume:4`

```
TARGET="${1:-all}"
RESUME_PHASE="$(echo "$2" | grep -oE '^resume:[0-9]+$' | cut -d: -f2)"
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

- Working code from `/monke-flash:blitz` (at least one flow marked "done")
- `monke-docs/flash/flash-scope.md` exists (for success criteria)

---

## Gate Semantics

Flash runs under light rigor per S9.4/S12. HARD gates (PG-1 in scope, PG-11 in snap) always surface. SOFT gates auto-pass per the adaptive system. TRIGGERED gates fire on their triggers regardless of rigor.

⏸ **PG-13 [TRIGGERED] — Persistent failure / non-convergence.** Fires at iteration 4+ of the Convergence Check (Phase 5). Surfaces regardless of rigor. Resolution: stop iterating and either re-run `/monke-flash:scope` to narrow the MVP or mark remaining items "defer" and move to `/monke-flash:snap`. Never silently loop past iteration 3 without surfacing.

---

## Phase 1: Run It

**Resume check (per drafter §7).** If arguments contain `resume:<N>`:
1. Read `monke-status.md` Flash section (the canonical recovery source for pulse — this skill does not write a lock-file).
2. Verify the named iteration matches the expected phase.
3. If parse check passes → skip to Phase `<N>` with prerequisites re-validated inline.
4. If parse check fails → emit warning "resume:<N> specified but checkpoint invalid" and proceed from Phase 1 normally.
5. See Context Death Protocol below for the full recovery spec.

Run the MVP. If `TARGET` is a specific flow, run just that path.

Show the user what happens:
- Console output, browser behavior, API response — whatever the demo is
- If it crashes, capture the error

---

## Phase 2: Collect Feedback

Ask the user three questions:
1. **What works?** — Confirm what's behaving correctly
2. **What's wrong?** — Broken behavior, incorrect output, crashes
3. **What's missing?** — Things they expected but didn't see

---

## Phase 3: Categorize

Sort every piece of feedback into exactly one bucket:

| Bucket | Meaning | Action |
|--------|---------|--------|
| **Fix now** | Broken. Blocks the demo. | Loop back to blitz immediately |
| **Adjust** | Works but wrong behavior. | Loop back to blitz with specific instructions |
| **Defer** | Nice to have. Not MVP. | Add to `flash-scope.md` OUT/Deferred list |

Present the categorization. Let the user override — they might promote a "defer" to "fix now" or demote a "fix now" to "defer."

⏸ **Feedback triage [SOFT] — categorization agreed.**
Auto-pass when: every feedback item has exactly one bucket assigned AND no item sits in "fix now" without a reproducible failure noted.
Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when the condition holds.

---

## Phase 4: Fix Loop

For each **fix now** and **adjust** item:

1. Describe the fix clearly (what's wrong, what it should do)
2. Implement the minimum fix — don't gold-plate
3. Run the affected flow again
4. Show the user: "Fixed. Good now?"

Keep going until user confirms or re-categorizes remaining items.

Commit after fixes: `flash : pulse fixes — <summary>`

---

## Phase 5: Convergence Check

Track iteration count. Pulse should converge in **2-3 loops**.

| Iteration | What to say |
|-----------|-------------|
| 1 | Normal. Fix and loop. |
| 2 | "Getting close. What's left?" |
| 3 | "Third pass. Anything remaining should probably be deferred." |
| 4+ | **PG-13 [TRIGGERED] fires.** "We're gold-plating. The MVP is good enough or the scope was wrong. Consider re-running `/monke-flash:scope` or cutting remaining items to defer." Surface regardless of rigor. |

When the user says "good enough" or all flows pass:
> "MVP breathes. Run `/monke-flash:snap` to freeze it and prep for the production crawl."

---

## Feedback Log

Maintain a running log in `monke-docs/flash/flash-pulse-log.md`:

```markdown
# Flash Pulse Log — <project name>

## Iteration 1 — <date>
### Fix Now
- <item> — status: fixed | still broken
### Adjust
- <item> — status: fixed | still off
### Defer
- <item> — added to scope OUT list

## Iteration 2 — <date>
...
```

---

## Status Update

On completion, update `monke-status.md` Flash section:
```
## Flash
Phase: **Pulse <pass|iterating>**
Iterations: <N>
Flows passing: <N>/<total>
Next: `/monke-flash:snap` or `/monke-flash:pulse` (if still iterating)
```
Bump `Updated:` to today, `by /monke-flash:pulse`

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Skip pulse and go straight to snap | Refuse. Run it, show it. Untested MVPs are assumptions, not products. |
| Gold-plate during pulse (add features, refactor) | Refuse. Pulse fixes what's broken. Everything else is defer or scope. |
| Loop forever without convergence check | Refuse. Iteration 4+ triggers PG-13 — suggest re-running `/monke-flash:scope` or defer the remainder. |
| Auto-categorize feedback without user input | Refuse. Present the buckets, let the user sort. Monke suggests, human decides. |

---

## Context Death Protocol

**Recovery source of truth.** The canonical recovery state for pulse is the Flash section of `monke-status.md`. The file `monke-docs/flash/flash-pulse-log.md` is a **human-readable iteration log** — appended each iteration for user review — but it is NOT a checkpoint. On re-entry, read only `monke-status.md` to determine which pulse iteration is active. If `flash-pulse-log.md` is newer than the status file's iteration marker, emit a warning but do not infer state from it.

**Status line marker:** `Where We Are: flash:pulse — iteration <N> (fixing)` while mid-flight.
**Recovery detection:** On re-entry, read `monke-status.md` Flash section. The `Iterations:` field names the active iteration; resume at Phase 4 Fix Loop if items are open, at Phase 5 Convergence Check if all iterations closed but no "MVP breathes" handoff recorded. If `monke-status.md` Flash section is absent → prompt the user rather than infer from `flash-pulse-log.md` or commit history.
