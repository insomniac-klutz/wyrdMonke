# WyrdMonke Test:Orchestra — The Paranoid Monke's Quality Loop

> **Usage:** Copy `monke-test/` to `.claude/commands/monke-test/`. Invoke: `/monke-test:orchestra`
>
> Not an autopilot. A diagnostic dashboard with hands.

---

## How It Works

Orchestra runs a continuous **read → diagnose → recommend → confirm → execute → update** loop. It never picks work for you — it tells you what the jungle looks like and waits for you to point.

---

## The Loop

1. **Read** `monke-status.md` — current test state across all components. LLDs table, Test Gates table, Implementation table, Phase Checkpoints.
2. **Diagnose** — walk the decision tree below. Find the highest-value testing work right now.
3. **Recommend** — present the recommendation with reasoning. One action at a time.
4. **⏸ Wait** — user confirms, picks something else, or says "done."
5. **Execute** — run the chosen skill's logic inline (follow `test-plan.md`, `test-run.md`, or `coverage.md` logic directly).
6. **Update** `monke-status.md` per the executed skill's status update rules.
7. **Loop** — back to step 1. Read the fresh state. Diagnose again.

**"Done" means done.** When the user says stop, stop. Don't guilt-trip them about unchecked gates.

---

## Decision Tree

Read `monke-status.md`, then walk top-to-bottom. **First match wins.**

```
Read monke-status.md
│
├─ Any LLD with design confirmed (PG-9) but no test plan (PG-10)?
│  └─ Recommend: test-plan <component>
│     "Design is locked. Time to plan what we're testing before anyone writes a line."
│
├─ Any component with test plan but tests not written?
│  └─ "Tests are written during /monke-implement:implement Layer 2-3.
│      Run /monke-implement:orchestra to start building."
│
├─ Any component at IL-2 with unit tests?
│  ├─ Coverage not checked yet?
│  │  └─ Recommend: coverage <component>
│  │     "Unit tests exist but nobody checked if they actually cover anything."
│  └─ Coverage below threshold?
│     └─ Recommend: coverage <component>
│        "Below the floor. Coverage shows exactly what's naked."
│
├─ Any component at IL-3 with integration tests?
│  ├─ Not verified by test-run yet?
│  │  └─ Recommend: test-run integration <component>
│  └─ Failing?
│     └─ Recommend: test-run integration <component>
│        "Still red. Let's categorize the failures and figure out which side is wrong."
│
├─ Integration tests deferred for a component?
│  └─ Check if neighbor component is now at IL-3
│     ├─ Yes → "Deferred integration tests for <component> can now run.
│     │         Use /monke-implement:implement <component> 3"
│     └─ No  → Note: "Still waiting on <neighbor>. Nothing to do here yet."
│
├─ Phase checkpoint approaching? (all phase components at IL-3)
│  ├─ System tests not written?
│  │  └─ "Phase ready for checkpoint. Run /monke-implement:checkpoint <N>"
│  └─ System tests failing?
│     └─ Recommend: test-run system
│        "System tests are the last gate before the phase locks. Let's diagnose."
│
├─ Persistent failures (>2 cycles on same test)?
│  └─ "⏸ PG-13 escalation — this test keeps failing. Likely a design issue.
│      Options: fix implementation, revise LLD, or escalate to /monke-design:oq"
│
├─ All components green, all coverage met?
│  └─ "All gates green. Coverage met. Tests clean.
│      Run /monke-implement:checkpoint or /monke-rage:buggy for paranoia."
│
└─ Nothing to test?
   └─ "No testable components yet. Run /monke-design:orchestra or
      /monke-implement:orchestra first. Can't test bananas that don't exist."
```

---

## Key Behaviors

- **Test plan before tests.** Never recommend `test-run` for a component without a confirmed PG-10 test plan. Plans first, tests second, always.
- **Coverage is a floor, not a trophy.** Below threshold = blocked. Above threshold = sufficient. Don't chase 100% — that's vanity metrics for monkes who've lost the plot.
- **Failure categorization.** When tests fail, use `test-run`'s analysis: implementation bug, design bug, test bug, or environment issue. Don't just say "it's red."
- **Deferred test tracking.** Knows which integration tests were deferred and why. Re-checks every loop. When the neighbor finally shows up, surfaces it immediately.
- **Cross-skill awareness.** Tests are WRITTEN during `/monke-implement:implement` Layer 2-3. Tests are VERIFIED and MEASURED by test skills. Orchestra doesn't write tests — it checks if they exist, pass, and cover.
- **PG-13 escalation.** Persistent failures (>2 cycles) get surfaced to the user, not retried endlessly. Insanity is doing the same `test-run` expecting different results.

---

## Reactivity, Not Progression

Unlike design orchestra (which drives a linear progression) or implement orchestra (which drives layers), **test orchestra is reactive.** Call it when:

- You want a **testing health check** across the project
- You want to **verify gates** before a checkpoint
- You want to **find coverage gaps** that are blocking promotion
- Something is **failing** and you want diagnosis, not guessing

It's the monke you call when something smells wrong and you want an honest answer.

---

## Anti-Patterns

| If tempted to... | Do instead... |
|------------------|---------------|
| Write tests from orchestra | That's implement's job. Layers 2-3. Use `/monke-implement:implement`. |
| Check coverage before unit tests exist | Nothing to measure. Get tests written first. |
| Run system tests before all phase components hit IL-3 | System tests exercise the full flow. Half-built flows produce noise, not signal. |
| Ignore persistent failures | PG-13 exists for a reason. Escalate, don't loop. |
| Auto-pick the recommendation without pausing | Present and wait. Monke diagnoses, human decides. |
| Chase 100% coverage | Floor, not trophy. Strong assertions beat high line counts. |

---

## Status Update

After each execution, update `monke-status.md` per the executed skill's own status update rules. Orchestra doesn't invent its own status format — it delegates to the skill that ran.

Test orchestra reads the **Test Gates table** heavily and updates it after each `test-run` or `coverage` check. The LLDs table's Test Plan column gets updated after `test-plan`.

Bump `Updated:` with current date and `test:orchestra`.

If the session dies mid-loop — write status with the last completed action so the next `/monke-test:orchestra` picks up cleanly.
