# WyrdMonke Design:ADR — Architecture Decision Record

> **Usage:** Copy `monke-design/` to `~/.claude/commands/monke-design/`. Invoke: `/monke-design:adr <title> [component]`

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

Use the template from `design-specs.md` S7.1:

```markdown
# ADR-<NNN>: <Title>
Status: proposed
Date: <today>  |  Component: <HLD section or component name>

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

<Critic attacks if from Agent Teams. Failure modes identified. Edge cases.>

## Decision — chose <X> because <project-specific reason>

<The concrete, measurable reason. Not "it felt right." Not "both are fine.">

## Consequences

- **Easier:** <what this enables>
- **Harder:** <what this constrains>
- **Trade-off accepted:** <what we gave up and why it's acceptable>
- **HLD impact:** <sections that need updating, or "none">
```

**⏸ Present the ADR draft. Confirm / Adjust / Reject?**

---

## Phase 4: Finalize

1. Write confirmed ADR to `monke-docs/decisions/<NNN>-<slug>.md`
2. Update HLD S5 (Decision Index) — add link to new ADR
3. If ADR resolves an OQ → update `monke-docs/open-questions.md`: set `Status: resolved -> ADR-NNN`
4. If ADR impacts HLD sections → flag for user: "This ADR affects HLD S<X>. Update needed?"

---

## Status Update

On completion, update `monke-status.md`:
- Add row to Decisions table: `| <NNN> — <title> | proposed | <component> |`
- If OQ resolved → remove from Open Blockers table
- Bump `Updated:` line
