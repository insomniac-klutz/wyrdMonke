# WyrdMonke Flash:Orchestra — Zero to MVP, One Session, No Excuses

> **Usage:** Copy `monke-flash/` to `.claude/commands/monke-flash/`. Invoke: `/monke-flash:orchestra [napkin]`

---

## Arguments

`$ARGUMENTS` parsing:
- Optional free-text: a pasted napkin from claude.ai, a brain-dump, a product rant — anything that captures intent.
- If provided, skip the spark conversation and draft `flash-brief.md` directly from it.

```
NAPKIN="${ARGUMENTS:-}"
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

- `monke-docs/` directory exists (run `/monke-init` first if missing)

---

## Recovery Detection

Before swinging, check which bananas are already on the tree. Look for existing flash artifacts in `monke-docs/flash/`:

- `flash-brief.md` exists -> spark done
- `flash-scope.md` exists -> scope done
- `flash-arch.md` exists -> sketch done
- Working code + `flash :` commits -> blitz done
- `flash-manifest.md` exists -> **"Already frozen. Nothing to orchestrate."** Point to `/monke-recon:survey`.

**If some artifacts exist:** Skip completed phases. Show recovery state: "Detected: spark done, scope done. Resuming from **sketch**."

**If `NAPKIN` provided and `flash-brief.md` already exists:** Warn about overwrite. Wait for confirmation.

---

## Phase 1: Spark (inline)

Follow `spark.md` logic within this session.

**If `NAPKIN` was provided:** Extract intent from the pasted content. Draft `flash-brief.md` directly — skip the conversational rounds. Still present the brief for confirmation.

**If no napkin:** Run the full spark conversation — Phase 0 pre-flight, then rounds 1-4 (pain, shape, walls, spark). One group at a time. Don't vomit a questionnaire.

Write `monke-docs/flash/flash-brief.md`.

**Pause gate:** Present the brief as a crisp summary. Wait for confirm / adjust / reject. **This is non-negotiable.** Orchestra automates transitions, not decisions.

---

## Phase 2: Scope (inline)

Follow `scope.md` logic within this session.

Read the brief. For each flow, draw the 80% line. Build the IN list, the OUT list, success criteria, effort budget.

Write `monke-docs/flash/flash-scope.md`.

**Pause gate:** Present scope for confirmation. This is the MVP contract — once locked, changes require re-scoping.

---

## Phase 3: Sketch (inline)

Follow `sketch.md` logic within this session.

Read brief + scope. Make architecture decisions fast. Stack table, core entities, API surface, data model, folder structure, primary happy path.

Write `monke-docs/flash/flash-arch.md`.

**Pause gate:** Present architecture. If user disagrees — adjust. No ceremony, just change it.

---

## Phase 4: Blitz (inline)

Follow `blitz.md` logic within this session. **No pause gate here — just build.**

1. **Scaffold** — project structure, deps, entry point stub. Commit: `flash : scaffold`
2. **Build each flow** — happy path only. No edge cases, no gold-plating. Hardcode what you can.
3. **Commit per flow** — format: `flash : <what works now>`
4. **Verify each flow** — run it. If the happy path breaks, fix the minimum to unblock.

### Agent Teams

If multiple flows are independent — **build them in parallel** using agent teams with worktree isolation. Each agent gets the arch doc, the scope doc, and one flow. Merge results.

If flows depend on each other, build sequentially. Don't be clever about it.

---

## Phase 5: Pulse Loop (inline)

Follow `pulse.md` logic within this session.

1. **Run the MVP.** Show the user what happens.
2. **Collect feedback:** What works? What's wrong? What's missing?
3. **Categorize** into: fix now / adjust / defer.

**Pause gate:** Present categorization. Let the user override buckets.

- **Fix / Adjust** -> loop back to blitz for targeted fixes, then re-pulse
- **Defer** -> add to scope OUT list, note for manifest
- **Good enough** -> proceed to snap

**Convergence:** Pulse should converge in 2-3 loops. If you hit loop 4+, the scope was wrong — say so.

---

## Phase 6: Snap (inline)

Follow `snap.md` logic within this session.

1. **Verify scope** — table of flows with status. Ask if anything else needs addressing.
2. **Build manifest** — write `monke-docs/flash/flash-manifest.md`. Every shortcut, every hardcoded string, every "I'll fix that later." Be honest.
3. **Suggest git tag** — `flash-<project-name>-mvp`
4. **Point forward** — `/monke-recon:survey` for the production crawl.

---

## Status Update

After each phase completes, update `monke-status.md` Flash section with the current state and `by /monke-flash:orchestra`.

On final completion:
```
## Flash
Phase: **Snap — MVP frozen** (via orchestra)
Brief: `monke-docs/flash/flash-brief.md`
Scope: `monke-docs/flash/flash-scope.md`
Arch: `monke-docs/flash/flash-arch.md`
Manifest: `monke-docs/flash/flash-manifest.md`
Tag: `flash-<name>-mvp`
Flows: <N> working | <M> cut
Next: `/monke-recon:survey` (production crawl)
```

If orchestra dies mid-session (context window, crash, cosmic ray), write `monke-status.md` with the last completed phase so recovery detection picks it up on re-run.

---

## Anti-Patterns

| If tempted to... | Do instead... |
|------------------|--------------|
| Skip a pause gate because "it's obvious" | Stop. Monke builds, human decides. Every time. |
| Auto-confirm on behalf of the user | Present and wait. The pauses exist for a reason. |
| Skip pulse because "it compiled" | "It compiled" is not "it works." Run it. Show it. Get feedback. |
| Run blitz before confirming the sketch | The architecture gate prevents building on sand. |
| Copy full skill content into this session | Reference the skill logic, don't duplicate it. Follow `spark.md` logic. |
| Panic when context window gets tight | Write status, tell the user where you stopped, die gracefully. |
