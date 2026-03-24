# WyrdMonke Implement:Orchestra — The Assembly Line That Never Sleeps

> **Usage:** Copy `monke-implement/` to `.claude/commands/monke-implement/`. Invoke: `/monke-implement:orchestra`
>
> Reads the status board, figures out which banana is ripest, and drives the IL pipeline until every phase is shipped. You confirm. Monke builds.

---

## Prerequisites

- `monke-status.md` exists and is populated
- `monke-docs/hld.md` exists with phased component plan (S8)
- `monke-docs/project-specs.md` exists

If any are missing: "Run `/monke-design:orchestra` first. Can't build what hasn't been designed."

---

## The Loop

Orchestra runs a continuous **read - diagnose - recommend - wait - execute - update** cycle. It never picks a task without asking. It never skips a gate. It never freelances.

```
while not done:
    read monke-status.md
    diagnose what needs work next
    recommend with reasoning
    PAUSE — user confirms, overrides, or says "done"
    execute the chosen skill's logic inline
    update monke-status.md
    loop
```

**Every iteration pauses.** Orchestra automates transitions, not decisions.

---

## Decision Tree

Read `monke-status.md`, then walk this tree top-to-bottom. First match wins.

```
Read monke-status.md
│
├─ project-specs.md has unfilled placeholders (<<<)?
│  └─ Recommend: /monke-implement:fill
│     "Stack bindings before implementation. Always."
│
├─ No components at any IL gate?
│  └─ Check LLDs table — any with status "ready" + test plan confirmed?
│     ├─ Yes → Recommend: implement <first-phase-component> 0
│     │  "Fresh meat. Starting Layer 0 — type skeleton."
│     └─ No → "No components ready. Run /monke-design:orchestra first."
│        Stop.
│
├─ Component partially implemented? (IL-N passed, IL-N+1 not started)
│  └─ Recommend: implement <component> <next-layer>
│     "Resuming where we left off. Layer N+1 awaits."
│
├─ Component at IL-2 but integration tests deferred?
│  └─ Check if the blocking neighbor is now implemented
│     ├─ Yes → Recommend: implement <component> 3
│     │  "Neighbor showed up. Time for the deferred integration tests."
│     └─ No → Skip, check next component.
│
├─ All components in Phase N at IL-3?
│  └─ Recommend: checkpoint <N>
│     "Every banana in Phase N is ripe. Time to sign off."
│
├─ Phase N checkpoint passed?
│  └─ Move to Phase N+1 components. Recommend next.
│     "Phase N in the bag. Swinging to the next tree."
│
├─ Multiple independent components ready in same phase?
│  └─ Recommend: implement <component>.
│     Note: "Can run <other> in parallel via agent teams
│     with worktree isolation."
│
└─ All phases checkpointed?
   └─ "Implementation complete. All phases passed.
      Ship it or run /monke-rage:orchestra for one last look."
      Stop.
```

---

## Key Behaviors

- **Phase order is sacred.** Never recommend Phase N+1 before Phase N checkpoint passes. The jungle has layers for a reason.
- **Layer order is sacred.** Never skip IL gates. 0 then 1 then 2 then 3. Shape before behavior before tests before integration. No shortcuts.
- **Deferred integration tracking.** When Layer 3 was deferred because a neighbor wasn't ready, orchestra remembers. Every loop re-checks neighbor status. The debt doesn't disappear.
- **Agent teams for parallel components.** If 2+ components in the same phase have no cross-dependencies, suggest parallel execution with `isolation: "worktree"`. Don't force sequential when parallel is safe.
- **Persistent failure escalation.** If an IL gate fails >2 times on the same component, trigger **PG-13** and escalate to user. Monke doesn't bang its head on the same tree. User decides: fix, redesign, or cut scope.
- **One component at a time unless parallel.** Don't context-switch between components mid-layer. Finish the banana before grabbing another.

---

## Cross-References to Test Skills

| After... | Suggest... |
|----------|-----------|
| IL-2 passes | `/monke-test:coverage` to verify unit test coverage meets threshold |
| IL-3 passes | `/monke-test:test-run integration` to verify integration suite |
| Checkpoint time | System tests are part of `/monke-implement:checkpoint` |

---

## Anti-Patterns

| If tempted to... | Do instead... |
|------------------|--------------|
| Implement before LLD + test plan confirmed (PG-9 + PG-10) | Refuse. Point to `/monke-design:lld`. Design first. |
| Skip Layer 0 to "go faster" | Refuse. Type skeletons catch contract errors before they metastasize. |
| Write all tests after all code | Refuse. Layer 2 interleaves body + tests per function. That's the deal. |
| Move to Phase N+1 before Phase N checkpoint | Refuse. Phase order is load-bearing, not ceremonial. |
| Auto-confirm on behalf of the user | Stop. Present and wait. Every loop iteration pauses. |
| Retry a failing gate indefinitely | After 2 failures, escalate per PG-13. Insanity is doing the same thing expecting different results. |
| Implement multiple components in one pass | One component through the full pipeline. Unless parallel + isolated. |

---

## Status Update

After each execution step, update `monke-status.md` per the executed skill's own status rules:

- **`/monke-implement:fill`** — updates Bootstrap section
- **`/monke-implement:implement`** — updates Implementation table + Test Gates table
- **`/monke-implement:checkpoint`** — updates Phase Checkpoints table

Orchestra doesn't invent its own status format. It delegates to the skill, then bumps `Updated:` with current date and `/monke-implement:orchestra`.

If orchestra dies mid-session (context window, crash, act of god), the last status write is the recovery point. Next invocation reads `monke-status.md` and picks up where things fell apart. No work lost. No trees un-climbed.
