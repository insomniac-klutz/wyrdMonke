# WyrdMonke Design:Orchestra — The Thinking Monke's Autopilot

> **Usage:** Copy `monke-design/` to `.claude/commands/monke-design/`. Invoke: `/monke-design:orchestra`
>
> Not a linear chain. A **decision tree**. Orchestra reads the jungle, figures out which branch to swing to next, and waits for you to say "swing."

---

## Arguments

`$ARGUMENTS` parsing:
- None. Orchestra figures it out by reading `monke-status.md`.

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

---

## The Loop

Orchestra runs a continuous cycle until you say "done" or the design is complete:

**Read** `monke-status.md` -> **Diagnose** what's missing -> **Recommend** the next best action -> **Wait** for user confirmation -> **Execute** the chosen skill's logic inline -> **Update** status -> **Loop**

One recommendation at a time. Don't dump a roadmap. Present the single ripest banana.

---

## Decision Tree

Read `monke-status.md`. Walk the tree top-down. **First match wins.**

```
monke-status.md
|
+-- Bootstrap incomplete? (missing project-specs, CLAUDE.md, or unfilled <<<placeholders>>>)
|   --> Recommend: tinker
|       "Foundation first. Run tinker to fill project-specs and CLAUDE.md."
|
+-- No HLD? (monke-docs/hld.md missing)
|   +-- Existing code detected? (src/, lib/, app/, package.json, Cargo.toml, etc.)
|   |   --> Recommend: "You have code but no HLD. Consider /monke-recon:orchestra
|   |       to reverse-engineer one, or /monke-design:hld for a clean-sheet design."
|   +-- Fresh project?
|       --> Recommend: hld (start from L1)
|           "Greenfield. Time to draw the jungle map."
|
+-- HLD incomplete? (missing L1, L2, L3, or boundary matrix)
|   --> Recommend: hld (resume from next incomplete level)
|       "HLD in progress. Next up: <level>."
|
+-- Blocking OQs exist? (open-questions.md has P0 blockers)
|   --> Recommend: oq triage
|       "Blockers on the vine. Resolve them before swinging further."
|
+-- Components need LLDs? (status: "waiting" in LLDs table)
|   +-- Multiple ready? (same phase, no cross-deps)
|   |   --> Recommend: lld <highest-priority>
|   |       "Also ready for parallel LLDs: <list>. Agent teams can handle these simultaneously."
|   +-- Single next?
|       --> Recommend: lld <component>
|           "Next component ripe for detail work."
|
+-- LLD has "boundary stale" status?
|   --> Recommend: lld <component>
|       "HLD evolved since this LLD was written. Re-read boundary, update design."
|
+-- All Phase N LLDs ready + test plans confirmed?
|   --> "Design for Phase <N> complete. Run /monke-implement:orchestra to start building."
|
+-- Unrecorded decisions detected? (LATS branches without ADRs)
|   --> Recommend: adr
|       "Decisions were made but not recorded. Capture them before they rot."
|
+-- Nothing left?
    --> "Design complete. All components have LLDs with confirmed test plans.
         Run /monke-implement:orchestra. The jungle is mapped."
```

---

## Behaviors

**Always reads status first.** Never guesses, never assumes. If `monke-status.md` is missing or empty, that's diagnostic info too -- bootstrap is incomplete.

**One recommendation at a time.** Present the single best next action with reasoning. The monke points at one banana, not the whole tree.

**User can override.** "Actually I want to work on X instead" -- do that. Orchestra serves, it doesn't dictate.

**User can say "done."** Exit the loop gracefully. Write current progress to status.

**Tracks session progress.** After each execution, show a running tally:
```
This session: tinker done, hld L1 done, hld L2 done. Next: hld L3.
```

**Handles cold starts.** If user comes back hours later, read status fresh. Don't assume prior context survived.

**Agent teams.** Suggest the right team shape for the current skill:
- HLD -> Architect + Critic
- LLD -> Designer + Reviewer
- Parallel LLDs (independent, same phase) -> one agent team per component

**Inline escalation.** If an OQ or boundary mismatch surfaces during LLD execution, handle it inline per that skill's escalation rules. Don't break the orchestration loop to run a separate skill invocation -- fold it in.

---

## Execution

When the user confirms a recommendation (or picks an override), execute that skill's logic **inline**. Follow the referenced skill file (`tinker.md`, `hld.md`, `lld.md`, `oq.md`, `adr.md`) -- don't duplicate their full content here.

After each skill execution:
1. Update `monke-status.md` per that skill's status update rules
2. Show session progress tally
3. Re-read `monke-status.md` and walk the decision tree again
4. Present the next recommendation

---

## Anti-Patterns

| If tempted to... | Do instead... |
|------------------|---------------|
| Run LLD before HLD L3 is confirmed | Stop. The component map defines what gets LLDs. No map, no LLDs. |
| Run Phase 2 LLDs before Phase 1 LLDs are ready | Respect dependency order. Phase 1 outputs feed Phase 2 inputs. |
| Ignore blocking OQs to "make progress" | Blockers exist for a reason. Resolve or consciously defer with user consent. |
| Auto-confirm any pause gate on behalf of the user | **Orchestra automates transitions, not decisions.** Every gate pauses. Every. Single. One. |
| Dump all remaining work as a list | One recommendation. One banana. Let the monke swing. |
| Add its own section to `monke-status.md` | Orchestra drives existing status tracking. It doesn't add meta-status about itself. |
| Panic when context window gets tight | Write status, show session tally, tell user where you stopped. Die gracefully. Resume reads status fresh. |

---

## Status Update

Orchestra itself writes no status section. Each skill execution within the loop updates `monke-status.md` per that skill's own rules. Orchestra just drives the bus and bumps the `Updated:` line after each stop.
