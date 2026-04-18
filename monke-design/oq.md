# WyrdMonke Design:OQ — The Parking Lot of Doubt

> **Usage:** `/monke-design:oq [action] [id]`
>
> Parks every unresolved design / implementation / testing question in one place with a clear block-on label, so nothing silently slips past a gate while pretending it was decided.

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
> Format per the OQ Rules inlined in this skill.
```

---

## Inlined OQ Rules (from design-specs S8)

Spec file `monke-docs/design-specs.md` remains source of truth — do not re-load at runtime.

### Format

```
### OQ-NNN: Question
Discovered-during: design | implementation | testing
Affects: HLD S-X.Y  |  Blocks: LLD for <component> | implementation of <component> | testing of <boundary>
Options so far: ...
Status: open | resolved -> ADR-NNN
```

### Rules

- Every OQ tags what it blocks (LLD, implementation, or test).
- Blocking OQs MUST be resolved before the blocked work proceeds.
- Resolution → ADR + doc update (HLD, LLD, or both as appropriate).
- Auto-number by scanning existing entries. Append to `monke-docs/open-questions.md`.

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

⏸ **SKILL-GATE:triage [SOFT] — Triage results confirmed.** Present triage results. Ask user to confirm priorities and resolve any P0 blockers.
Auto-pass when: 0 P0 blockers detected AND at least one OQ exists (empty triage is skipped silently).
Rigor: surfaces under `thorough` regardless; auto-confirms under `light`/`standard` when condition holds.

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

⏸ **PG-14 [TRIGGERED] — Open question resolution (HLD/LLD amendment).** Present the resolution + any downstream impacts. Confirm / Adjust / Reject?
Fires when resolving an OQ updates the HLD boundary matrix, an LLD contract, or an ADR. Ignores rigor. See design-specs §9.4 PG-14 boundary-change sub-case. OQ resolution cascades into HLD / LLD / implementation state — the human must confirm which blocked work becomes unblocked and whether an ADR is needed.

---

## Creating New OQs

Any skill can create OQs inline using the format in the inlined OQ Rules above.

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Resolve an OQ by deleting the entry | Refuse. Set `Status: resolved -> ADR-NNN` (or `resolved — <reason>`) so the resolution trail survives. |
| File a vague question as an OQ ("how should we do auth?") | Refuse. Every OQ names what it blocks (`Blocks: LLD for <component>` / implementation / testing) — no blocker, no OQ. |
| Create an OQ without picking a `Discovered-during:` phase | Refuse. Design / implementation / testing classification drives triage grouping. |
| Skip ADR when 2+ viable options were considered | Refuse. 2+ viable options = ADR. Single obvious resolution = status update only. |
| Auto-triage resolve every P2 because "there's no blocker" | Refuse. P2 is parked, not resolved — status stays `open` until someone names the resolution. |

---

## Context Death Protocol

**Checkpoint artifacts:** in-progress edits to `monke-docs/open-questions.md` (status changes, new entries). If `resolve` action triggered an inline ADR via `/monke-design:adr` logic, that ADR's draft file.
**Status line marker:** `Where We Are:` reads `design:oq — <action> on <ID or scope>` while mid-flight.
**Recovery detection:** On re-entry, if `open-questions.md` has OQ entries with `Status: open` that were partially updated in the last session (draft edit timestamps) → re-read the file, re-triage, continue. If an inline ADR draft exists but no status flip happened → resume at the Phase 3 resolution write. If no partial state → start clean from the requested action.

---

## Status Update

**Read on entry:** `monke-status.md` — load Open Blockers table to cross-check with `open-questions.md` entries before triage.
**Write on exit:**
- Success (`list`/`triage`): no status change (read-only) but bump `Updated:` line if any priorities were changed.
- Success (`resolve`): remove resolved OQ from Open Blockers table (if it was there); if resolution unblocks a component, update that component's status in the relevant table; bump `Updated:` line.
- Blocked: if user rejects resolution at PG-14 → keep OQ `open`, add to Open Blockers if not already there with the reason.
- Partial: if `triage` surfaced P0 blockers but user paused before resolving them → write `Resume:` block naming the P0 list and next action.
