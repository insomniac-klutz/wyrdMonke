# WyrdMonke Flash:Scope — The Machete Pass

> **Usage:** /monke-flash:scope
>
> Swings the machete. The MVP gets cut down to its three load-bearing flows — everything else is named, dated, and buried in the OUT list.

---

## Arguments

`$ARGUMENTS` parsing:
- None. Reads from `flash-brief.md`.

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

- `monke-docs/flash/flash-brief.md` exists (run `/monke-flash:spark` first)

---

## Gate Semantics

Flash runs under light rigor per S9.4/S12. HARD gates (PG-1 in scope, PG-11 in snap) always surface. SOFT gates auto-pass per the adaptive system. TRIGGERED gates fire on their triggers regardless of rigor.

PG-1 lives here (Phase 6) and is HARD — never auto-passes.

---

## Phase 1: Read the Napkin

Read `monke-docs/flash/flash-brief.md`. Internalize:
- The 3 key flows
- Hard constraints
- Stack preferences

Present a one-line recap: "Building **X** — three flows: A, B, C. Let's cut."

---

## Phase 2: Draw the 80% Line

For **each key flow**, ask the user:

> "Flow N: <description>. What's the **minimum version** that proves this works? Not the good version — the version that makes someone go 'oh, I see what this does.'"

Help them cut:
- "Do you need auth for the demo, or can you hardcode a user?"
- "Does this need real data, or can we seed 10 rows?"
- "Does this need to look good, or does it need to work?"

**Ruthlessly distinguish:**

| Category | Meaning |
|----------|---------|
| **Proves the concept** | Without this, the demo is meaningless |
| **Nice to have** | Makes it better but doesn't prove anything new |
| **Temptation** | You want it because it's fun to build, not because it matters |

---

## Phase 3: The OUT List

**This is the most important part.** Make the OUT list explicit. Name the temptations:

> "Here's what we're NOT building in flash: ..."

The OUT list prevents scope creep during blitz. If it's not on the IN list, it doesn't get built. Period.

Split deferred items into:
- **Deferred to monke-recon** — real features that belong in the 80->100% phase
- **Killed** — ideas that sounded good but don't serve the MVP

---

## Phase 4: Success Criteria

Ask:
> "How do we know the MVP works? Give me something concrete and testable."

Not "it should be fast" — "it returns results in under 2 seconds." Not "users like it" — "a user can complete flow 1 without instructions."

Aim for 2-5 concrete criteria.

---

## Phase 5: Effort Budget

Ask:
> "How many sessions should this take? 1? 3? 5?"

Calibrate expectations. Flash is for things you can build in 1-5 sessions, not 50.

---

## Phase 6: Write the Scope

Write `monke-docs/flash/flash-scope.md`:

```markdown
# Flash Scope — <project name>

Created: <date> by /monke-flash:scope

## IN (MVP)

### Flow 1: <name>
- <minimum version description>
- <what's included>

### Flow 2: <name>
- ...

### Flow 3: <name>
- ...

## OUT

### Deferred to Recon
- <feature> — why it's deferred
- ...

### Killed
- <idea> — why it's dead
- ...

## Success Criteria
1. <concrete, testable criterion>
2. ...

## Effort Budget
<N> sessions | Target: <date or "whenever">
```

⏸ **PG-1 [HARD] — Scope lock.** Present to user for confirmation. This is the MVP contract — once confirmed, scope is locked. HARD: never auto-passes, never skippable. Changes during blitz require re-running scope.

---

## Status Update

On completion, update `monke-status.md` Flash section:
```
## Flash
Phase: **Scope locked**
Brief: `monke-docs/flash/flash-brief.md`
Scope: `monke-docs/flash/flash-scope.md`
Flows: <N> in | <M> deferred | <K> killed
Next: `/monke-flash:sketch`
```
Bump `Updated:` to today, `by /monke-flash:scope`

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Keep everything in scope ("we need all of it") | Refuse. Draw the 80% line. If everything is in, nothing is cut, and scope is meaningless. |
| Add auth/permissions/admin to MVP scope | Refuse. Hardcode a user. Auth is a feature, not a prerequisite. |
| Skip the OUT list | Refuse. Explicit deferred vs killed prevents scope creep during blitz. |
| Lock scope without user confirmation | Refuse. Scope is a contract. The user signs it. |

---

## Context Death Protocol

**Checkpoint artifacts:** `monke-docs/flash/flash-scope.md` (partial draft) written after Phase 3 (OUT list) so the cut decisions survive even if the skill dies before PG-1.
**Status line marker:** `Where We Are:` in `monke-status.md` reads `flash:scope — phase <N> (pre-lock)` while mid-flight.
**Recovery detection:** On re-entry, if `flash-scope.md` exists without a PG-1 audit entry → resume at the phase indicated in the Status marker; if PG-1 was logged as confirmed → scope is locked, warn before re-running; if neither file nor marker → start fresh at Phase 1.
