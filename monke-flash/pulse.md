# WyrdMonke Flash:Pulse — Does it breathe?

> **Usage:** Copy `monke-flash/` to `.claude/commands/monke-flash/`. Invoke: `/monke-flash:pulse [flow]`

---

## Arguments

`$ARGUMENTS` parsing:
- Single positional: specific flow to test, or `all` (default)
- Example: `/monke-flash:pulse "user search"` or `/monke-flash:pulse`

```
TARGET="${ARGUMENTS:-all}"
```

---

## Prerequisites

- Working code from `/monke-flash:blitz` (at least one flow marked "done")
- `monke-docs/flash/flash-scope.md` exists (for success criteria)

---

## Phase 1: Run It

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
| 4+ | "We're gold-plating. The MVP is good enough or the scope was wrong. Consider re-running `/monke-flash:scope`." |

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
