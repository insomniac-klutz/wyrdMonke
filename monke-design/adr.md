# WyrdMonke Design:ADR — The Receipts

> **Usage:** `/monke-design:adr <title> [component]`
>
> Freezes the why into a numbered receipt — options considered, decision made, trade-off swallowed — so six months from now no one can pretend the other path was free.

---

## Arguments

`$ARGUMENTS` parsing:
- First positional: decision title (**required**). Quoted if multi-word.
- Second positional: component or HLD section reference (optional — inferred from context if omitted)
- If title missing → ask: "What decision are you recording?"

Examples:
- `/monke-design:adr "ORM selection" data-layer`
- `/monke-design:adr "Auth pattern"`
- `/monke-design:adr "Cache strategy" api-gateway`

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

Read `monke-status.md`. Then verify:

- `monke-docs/decisions/` directory exists
- `monke-docs/hld.md` exists (ADRs reference HLD sections)

---

## Phase 1: Auto-Number

Scan `monke-docs/decisions/` for existing ADR files. Pattern: `NNN-slug.md`.

```bash
ls monke-docs/decisions/*.md 2>/dev/null | sort
```

Next number = highest existing + 1 (or 001 if none exist).

Generate slug from title: lowercase, spaces → hyphens, strip special chars.

```
FILE="monke-docs/decisions/${NEXT_NUM}-${SLUG}.md"
```

---

## Phase 2: Gather Context

If this ADR was triggered by a LATS branch (most common case):

1. The LATS output should already exist in the conversation — options expanded, evaluated, recommended pick.
2. If not → ask: "What options were considered? Which was selected and why?"

If this ADR was triggered by other events (stack violation, pattern selection, test gate failure):

1. Ask what triggered the decision
2. Gather the relevant context (which component, what constraint)

---

## Phase 3: Write ADR

### Inlined ADR Rules (from design-specs S7)

Spec file `monke-docs/design-specs.md` remains source of truth — do not re-load at runtime.

**Template (S7.1):**

> **Component reference format:** use `HLD S<section>.<subsection>` when the ADR impacts the HLD (e.g., `HLD S3.4 — Agent component design`). Full paths avoid the `S7` ambiguity (design-specs S7 = ADR Rules vs HLD S7 = Boundary Matrix). Never write bare `S7` or unqualified section numbers.

```markdown
# ADR-<NNN>: <Title>
Status: proposed | accepted | superseded by ADR-XXX
Date: <today>  |  Component: <HLD S<section>.<subsection> or component name>

## Context

<Why this decision was needed. What constraint or requirement triggered it.>

## Options (LATS output)

### Option A: <name>
<Description. Constraints. Downstream cost. Open questions.>

### Option B: <name>
<Description. Constraints. Downstream cost. Open questions.>

### Option C: <name> (if applicable)
<Description.>

## Challenges Considered

<Critic attacks from Agent Teams (copied from monke-docs/critic-notes.md).
Failure modes identified. Edge cases.>

## Decision — chose <X> because <project-specific reason>

<The concrete, measurable reason. Not "it felt right." Not "both are fine.">

## Consequences

- **Easier:** <what this enables>
- **Harder:** <what this constrains>
- **Trade-off accepted:** <what we gave up and why it's acceptable>
- **HLD impact:** <sections that need updating, or "none">
```

**When to write (S7.2):** 2+ viable approaches. Library/tool selection. Non-obvious constraint. HLD change from LLD. LATS backtrack. Tech stack exception. Non-default language choice. Auxiliary datastore. Anthropic pattern selection (mandatory). Pattern escalation/de-escalation (mandatory). Test gate failure forcing redesign.

**When NOT to write (S7.3):** Obvious single-option choices. Style decisions. Reversible in 5 minutes with no ripple.

⏸ **SKILL-GATE:draft-confirm [SOFT] — ADR draft confirmed.** Present the ADR draft. Confirm / Adjust / Reject?
Auto-pass when: Context + Options (2+ with evaluation) + Decision with measurable reason + Consequences (easier / harder / trade-off / HLD impact) are all filled with concrete content (no placeholder text, no "TBD").
Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds. **If this ADR backs a PG-6 (non-default language) decision → this gate is HARD instead**, always surfaces, never auto-passes (the ADR is the receipt for a HARD decision upstream).

---

## Phase 4: Finalize

1. Write confirmed ADR to `monke-docs/decisions/<NNN>-<slug>.md`
2. Update HLD S5 (Decision Index) — add link to new ADR
3. If ADR resolves an OQ → update `monke-docs/open-questions.md`: set `Status: resolved -> ADR-NNN`
4. If ADR impacts HLD sections → flag for user: "This ADR affects HLD S<X>. Update needed?"

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Write an ADR with "both are fine" as the decision | Refuse. Every LATS has a recommended pick with a concrete, measurable reason. An ADR without a reason is a note, not a receipt. |
| Backdate an ADR to a prior number | Refuse. Auto-number from the existing `decisions/` directory — never re-use or re-sequence numbers. |
| Omit the Consequences section because the decision "seems obvious" | Refuse. Easier / Harder / Trade-off / HLD impact — every ADR states all four. Obvious decisions usually don't need an ADR at all (see S7.3). |
| Roll up multiple decisions into a single ADR | Refuse. One ADR per decision. Related decisions link via `superseded by ADR-XXX` or cross-references in Context. |
| Skip the Challenges Considered section when `critic-notes.md` exists | Refuse. Critic attacks are failure-mode evidence — copy them into the ADR so the decision stays honest. |

---

## Context Death Protocol

**Checkpoint artifacts:** draft ADR content in the proposed `monke-docs/decisions/<NNN>-<slug>.md` (written incrementally during Phase 3 presentation).
**Status line marker:** `Where We Are:` reads `design:adr — drafting <NNN>-<slug>` while this skill is mid-flight.
**Recovery detection:** On re-entry, if the proposed ADR file exists but has no `Decision —` line filled → resume at Phase 3 (Write ADR) presentation. If file is complete but HLD Decision Index wasn't updated → resume at Phase 4 (Finalize). If no draft file → start fresh from Phase 1 (auto-number).

---

## Status Update

**Read on entry:** `monke-status.md` — check current component context (if second positional arg not given), scan Decisions table for duplicate titles before claiming the next number.
**Write on exit:**
- Success: add row to Decisions table: `| <NNN> — <title> | proposed | <component> |`; if OQ resolved → remove from Open Blockers; bump `Updated:` line with date + `by /monke-design:adr`.
- Blocked: if ADR rejected at SKILL-GATE:draft-confirm → do not write file; add Open Blockers row with WHAT/WHY/HOW.
- Partial: draft ADR written but Decision Index link deferred → record `Resume:` block naming Phase 4 step 2.
