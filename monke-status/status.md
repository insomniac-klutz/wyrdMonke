# WyrdMonke Status — SDLC Progress Dashboard

> **Usage:** Copy `monke-status/` to `~/.claude/commands/monke-status/`. Then run `/monke-status:status [action]` inside a project with WyrdMonke bootstrapped.

---

## Arguments

`$ARGUMENTS` parsing:
- Single positional argument: `show` | `rebuild` | `next` | `blocked`
- Default (empty): `show`
- `show` — display current `monke-status.md` with a one-paragraph summary
- `rebuild` — scan all artifacts on disk, regenerate `monke-status.md` from truth
- `next` — answer "what should I do next?" with the single next actionable skill
- `blocked` — show only open blockers and what they're holding up

```
ACTION="${ARGUMENTS:-show}"
```

---

## Prerequisites

Verify `monke-status.md` exists at project root (not this skill file — the project's status dashboard).

- If it exists → read it
- If it doesn't exist → check if `monke-docs/` exists:
  - Yes → create `monke-status.md` from `monke-docs/status-template.md`, then proceed
  - No → tell user: "No WyrdMonke project found. Run `/monke-init` first." Stop.

---

## Action: `show`

1. Read `monke-status.md` from project root.
2. Present the full dashboard to the user.
3. Add a one-paragraph summary at the top:
   - Current phase
   - How many components designed / implemented / tested
   - Number of open blockers
   - Time since last update

---

## Action: `rebuild`

Scan all artifacts on disk and regenerate `monke-status.md` from truth. This recovers from stale or missing status files.

### Scan Sequence

| Step | Artifact | What to check | Status derived |
|------|----------|---------------|----------------|
| 1 | `monke-docs/project-specs.md` | Any `<<<` remaining? | Bootstrap: complete or incomplete (list unfilled groups) |
| 2 | `CLAUDE.md` or `monke-CLAUDE.md` | Exists? Has `<<<` remaining? | CLAUDE.md: merged, template-only, or missing |
| 3 | `monke-docs/hld.md` | File size > 1 line? Which sections (S1-S8) have content? | HLD: not started, L1, L2, L3, or complete |
| 4 | `monke-docs/lld/*.md` | Which files exist? For each: has ADaPT tree? Signatures? Test plan? Review log? | Per-component LLD status |
| 5 | `monke-docs/decisions/*.md` | Count, list titles + status (proposed/accepted/superseded) | ADR inventory |
| 6 | `monke-docs/open-questions.md` | Parse OQ entries. Count by status (open/resolved). Extract `Blocks:` fields. | Open blockers |
| 7 | `monke-docs/checkpoints/*.md` | Which phases exist? Parse checkbox state. | Phase checkpoint status |
| 8 | Source files matching LLD file maps | For each LLD component: do the files exist? Run IL gate commands from project-specs S8. | Implementation layer progress |
| 9 | Test files matching LLD test plans | Glob for `test_*`, `*.test.*`, `*.spec.*`. Classify by tier directory. | Test tier coverage |

### Rebuild Rules

- If a scan step fails (file missing, parse error), record that section as "unknown — scan failed" rather than guessing.
- After scanning, write the regenerated `monke-status.md` to project root.
- Show a diff summary: what changed vs the previous status file (if one existed).
- **Do NOT run any IL gate commands that could modify files or have side effects.** Read-only scan only. If you need to verify a gate, note it as "needs verification" and suggest the user run `/monke-test:test-run`.

**⏸ Present the rebuilt status to the user for confirmation before writing.**

---

## Action: `next`

Determine the single next actionable step using this waterfall:

```
1. Has project-specs been filled? (no <<< remaining)
   No  → "Run /monke-implement:fill to complete project-specs."
   Yes ↓

2. Does CLAUDE.md exist with no <<< remaining?
   No  → "Run /monke-init to set up CLAUDE.md."
   Yes ↓

3. Does hld.md have content beyond boilerplate?
   No  → existing code in project?
         Yes → "Run /monke-recon:survey then /monke-recon:reconstruct to reverse-engineer HLD."
         No  → "Run /monke-design:hld to create your HLD."
   Yes ↓

4. Are there open blockers (OQs with Blocks: that reference pending work)?
   Yes → "Resolve blocking OQs first: /monke-design:oq triage"
   No  ↓

5. Any components in HLD S3 without an LLD file?
   Yes → Pick first by phase order.
         "Run /monke-design:lld <component> to design <component>."
   No  ↓

6. Any LLD-complete components not yet implemented (no source files)?
   Yes → Pick first by phase order.
         "Run /monke-implement:implement <component> to start building."
   No  ↓

7. Any components with partial implementation (some layers done)?
   Yes → Detect highest passing IL gate.
         "Run /monke-implement:implement <component> <next-layer> to resume."
   No  ↓

8. All components in current phase at IL-3?
   No  → Find incomplete component.
         "Run /monke-implement:implement <component> <layer> to finish."
   Yes ↓

9. Phase checkpoint done?
   No  → "Run /monke-implement:checkpoint <phase> for phase sign-off."
   Yes → "Phase <N> complete. Next phase components need LLDs."
         Go to step 5 for next phase.
```

Present: the recommended skill invocation, why it's next, and what it depends on.

---

## Action: `blocked`

1. Read `monke-docs/open-questions.md`.
2. Filter to entries with `Status: open` that have a `Blocks:` field.
3. For each blocker, show:
   - OQ ID + question
   - What it blocks (LLD, implementation, or test for which component)
   - Options so far (if any)
4. If no blockers found, say so and suggest `/monke-status:status next` instead.

---

## Status File Format

When creating or rebuilding `monke-status.md`, use this structure:

```markdown
# monke-status.md
Project: <from project-specs S1> | Updated: <date> by <skill that last wrote>

## Where We Are

Phase: **<current phase description>**
Next action: `<skill invocation>`
Blockers: <count and summary, or "none">

---

## Bootstrap
- [ ] Templates fetched
- [ ] Project-specs filled (0/8 groups)
- [ ] CLAUDE.md merged

## HLD
- [ ] L1 Context — PG-1
- [ ] L2 Containers — PG-2
- [ ] L3 Components — PG-3
- [ ] Boundary Matrix — PG-4

## LLDs
| Component | Phase | ADaPT | Design | Review | Test Plan | Status |
|-----------|-------|-------|--------|--------|-----------|--------|

## Implementation
| Component | L0 | L1 | L2 (funcs) | L3 | Status |
|-----------|----|----|------------|----|----|

## Test Gates
| Component | Unit | Integration | Coverage | Status |
|-----------|------|-------------|----------|--------|

## Phase Checkpoints
| Phase | Components | All IL-3? | System Tests | PG-11 | Status |
|-------|-----------|-----------|--------------|-------|--------|

## Open Blockers
| ID | Blocks | Summary |
|----|--------|---------|

## Decisions
| ADR | Status | Component |
|-----|--------|-----------|
```

---

## Status Update Protocol

**Every skill in the WyrdMonke package follows this protocol:**

1. **On entry:** Read `monke-status.md`. Verify prerequisites for this skill are met. If not, tell the user what to do first (use the `next` waterfall logic).

2. **On exit (success):** Update the relevant row/checkbox in `monke-status.md`. Bump the `Updated` line with current date and skill name. Recalculate "Where We Are" and "Next action" using the waterfall.

3. **On exit (blocked):** Add to "Open Blockers" table. Update "Where We Are" to reflect the block. Suggest `/monke-design:oq triage`.

4. **On exit (partial):** Update progress (e.g., "3/7 functions" in L2 column). Keep status as "in progress". Note resume point so the next invocation can pick up.
