# WyrdMonke Design:OQ — Open Question Management

> **Usage:** Copy `monke-design/` to `~/.claude/commands/monke-design/`. Invoke: `/monke-design:oq [action] [id]`

---

## Arguments

`$ARGUMENTS` parsing:
- First positional: action — `list` | `triage` | `resolve` (default: `list`)
- Second positional: OQ ID for `resolve` action (e.g., `OQ-003`)
- If `resolve` without ID → list open OQs and ask user to pick

Examples:
- `/monke-design:oq` — list all
- `/monke-design:oq triage` — categorize and prioritize
- `/monke-design:oq resolve OQ-003` — resolve specific question

```
ACTION="${1:-list}"
OQ_ID="${2:-}"
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

Read `monke-status.md`. Verify `monke-docs/open-questions.md` exists.

If it doesn't → create it with header only:
```markdown
# Open Questions

> Parking lot for unresolved design, implementation, and testing questions.
> Format per design-specs.md S8.
```

---

## Action: `list`

1. Parse `monke-docs/open-questions.md`
2. For each OQ entry, extract: ID, question, discovered-during, affects, blocks, options, status
3. Present as a table:

| ID | Status | Dimension | Blocks | Question |
|----|--------|-----------|--------|----------|

4. Summary counts: N open, N resolved, N blocking downstream work

---

## Action: `triage`

1. Parse all OQ entries (same as `list`)
2. Group by dimension:

### Design Questions
| ID | Blocks | Question | Options | Priority |
|----|--------|----------|---------|----------|

### Implementation Questions
| ID | Blocks | Question | Options | Priority |
|----|--------|----------|---------|----------|

### Testing Questions
| ID | Blocks | Question | Options | Priority |
|----|--------|----------|---------|----------|

3. Prioritize within each group:
   - **P0 (blocking):** Has `Blocks:` field referencing work that should happen next per phase order
   - **P1 (soon):** Has `Blocks:` but blocked work is in a future phase
   - **P2 (park):** No `Blocks:` field, or affects non-critical path

4. For each P0 blocker:
   - Show the question + options so far
   - Suggest: "Resolve this now? If yes, we'll create an ADR. If not, what additional info is needed?"

**⏸ Present triage results. Ask user to confirm priorities and resolve any P0 blockers.**

---

## Action: `resolve`

Resolve a specific open question.

1. Find `OQ-${OQ_ID}` in `monke-docs/open-questions.md`
   - If not found → show available OQs, ask user to pick

2. Present the full OQ entry:
   - Question, context, options so far, what it blocks

3. Ask user for the resolution:
   - Which option was chosen (or a new one)?
   - Why?

4. Determine if an ADR is needed:
   - 2+ viable options were considered → **yes, create ADR**
   - Single obvious resolution → no ADR, just update status

5. If ADR needed → follow `/monke-design:adr` logic inline:
   - Auto-number, write ADR, present for confirmation

6. Update the OQ entry:
   ```
   Status: resolved -> ADR-NNN (or "resolved — <brief reason>" if no ADR)
   ```

7. Propagate the resolution:
   - If `Affects: HLD S-X.Y` → flag: "HLD section <X.Y> may need updating."
   - If `Blocks: LLD for <component>` → that LLD is now unblocked
   - If `Blocks: implementation of <component>` → that implementation is unblocked
   - If `Blocks: testing of <boundary>` → that test is unblocked

**⏸ Present the resolution + any downstream impacts. Confirm?**

---

## Creating New OQs

Any skill can create OQs inline. The format from `design-specs.md` S8:

```markdown
### OQ-NNN: Question
Discovered-during: design | implementation | testing
Affects: HLD S-X.Y  |  Blocks: LLD for <component> | implementation of <component> | testing of <boundary>
Options so far: ...
Status: open
```

Auto-number by scanning existing entries. Append to `monke-docs/open-questions.md`.

---

## Status Update

On completion, update `monke-status.md`:
- **list/triage:** no status change (read-only)
- **resolve:** remove resolved OQ from Open Blockers table (if it was there). If the resolution unblocks a component, update that component's status in the relevant table.
- Bump `Updated:` line
