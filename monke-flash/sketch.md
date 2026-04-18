# WyrdMonke Flash:Sketch — Napkin architecture

> **Usage:** /monke-flash:sketch
>
> Architecture on a napkin. Monke picks the simplest stack that holds, names the entities, draws the happy path — no ADRs, no ceremony.

---

## Arguments

`$ARGUMENTS` parsing:
- `resume:<N>` (optional positional) — skip to phase `<N>` with checkpoint re-read (see drafter §7). Phase numbers: 1=load context, 2=decide fast, 3=write sketch.
- Default (empty): start from Phase 1. Reads from `flash-scope.md` and `flash-brief.md`.

```
RESUME_PHASE="$(echo "$ARGUMENTS" | grep -oE '^resume:[0-9]+$' | cut -d: -f2)"
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

- `monke-docs/flash/flash-scope.md` exists (run `/monke-flash:scope` first)
- `monke-docs/flash/flash-brief.md` exists

---

## Gate Semantics

Flash runs under light rigor per S9.4/S12. HARD gates (PG-1 in scope, PG-11 in snap) always surface. SOFT gates auto-pass per the adaptive system. TRIGGERED gates fire on their triggers regardless of rigor.

---

## Phase 1: Load Context

**Resume check (per drafter §7).** If arguments contain `resume:<N>`:
1. Verify partial `monke-docs/flash/flash-arch.md` exists and parses (this skill does not write a lock-file — section-by-section writes to `flash-arch.md` serve as checkpoints).
2. If parse check passes → skip to Phase `<N>` with prerequisites re-validated inline.
3. If parse check fails → emit warning "resume:<N> specified but checkpoint invalid" and proceed from Phase 1 normally.
4. See Context Death Protocol below for the full recovery spec.

Read both `flash-brief.md` and `flash-scope.md`. Know:
- What's IN (the flows to build)
- Stack preferences (or "monke picks")
- Hard constraints
- Success criteria

---

## Phase 2: Decide Fast

Make architecture decisions. **No LATS. No ADaPT. No CoALA.** Just pick the simplest thing that works.

### Stack Selection

If user said "monke picks" — choose based on:
- What's fastest to MVP for this type of project
- What has the fewest moving parts
- What the user is likely to already have installed

If user specified preferences — use them. Don't second-guess.

Present stack as a table:

| Layer | Choice | Why |
|-------|--------|-----|
| Language | ... | ... |
| Framework | ... | ... |
| Database | ... | ... |
| ... | ... | ... |

### Core Entities

List 3-7 entities max. For each: name, what it holds, relationships.

Don't over-model. If the MVP has users and items, you need a `User` and an `Item`. You don't need `UserProfile`, `UserPreferences`, `UserSettings`, `UserAuditLog`.

### API Surface

Routes, endpoints, commands, CLI args — whatever the interface is for this project. Just the ones needed for the IN-scope flows.

### Data Model

The 3-5 tables/collections/structs. Fields that matter. Skip audit columns, soft deletes, versioning — that's recon territory.

### File/Folder Structure

Propose a structure. Keep it flat. Flash projects don't need 7 levels of nesting.

### Primary Happy Path

One data flow: user triggers flow 1, data moves through the system, result comes back. Describe it as a numbered sequence, not a diagram.

---

## Phase 3: Write the Sketch

Write `monke-docs/flash/flash-arch.md`:

```markdown
# Flash Architecture — <project name>

Created: <date> by /monke-flash:sketch

## Stack

| Layer | Choice | Why |
|-------|--------|-----|
| ... | ... | ... |

## Entities

| Entity | Fields | Relationships |
|--------|--------|---------------|
| ... | ... | ... |

## API Surface

| Method | Path/Command | Purpose |
|--------|-------------|---------|
| ... | ... | ... |

## Data Model

<Schema or struct definitions — keep it brief>

## Structure

```
project/
  src/
    ...
  ...
```

## Happy Path — Flow 1

1. User does X
2. System calls Y
3. Data flows to Z
4. User sees result
```

**Keep the whole file under 2 pages.** If it's longer, you're over-designing for a flash build.

**Present to user.** If they disagree with a choice — adjust immediately. No ceremony, no ADR, just change it.

⏸ **Sketch confirmation [SOFT] — stack + entities + happy path agreed.**
Auto-pass when: stack table, entity table, API surface, and Flow 1 happy path are all filled in (no "TBD", no empty rows) AND every stack choice carries a "Why".
Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when the condition holds.

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Add microservices, queues, caches, or sidecars | Refuse. Flash is one process when possible. Premature decomposition kills velocity. |
| Write an ADR for each stack choice | Refuse. The "Why" column in the stack table IS the decision record. Save full ADRs for recon. |
| Design for 10x load or multi-tenant from day one | Refuse. Sketch picks for the MVP. Scale and multi-tenancy are recon territory. |
| Split the data model into 10 normalized tables | Refuse. 3-5 tables max. Denormalize first, normalize when recon says so. |
| Bring in a new framework because it's trendy | Refuse. Pick what the user already knows or what has the fewest moving parts. |

---

## Context Death Protocol

**Checkpoint artifacts:** `monke-docs/flash/flash-arch.md` (partial draft) written section-by-section as each appears (Stack → Entities → API → Happy Path). Each written section locks in that decision.
**Status line marker:** `Where We Are: flash:sketch — section <N>/6` while mid-flight.
**Recovery detection:** On re-entry, if `flash-arch.md` exists with sections incomplete → resume at the first missing section; if file is complete but Phase 3 confirmation gate not logged → re-present for confirmation; if neither file nor marker → start fresh at Phase 1.

---

## Status Update

On completion, update `monke-status.md` Flash section:
```
## Flash
Phase: **Sketch done**
Brief: `monke-docs/flash/flash-brief.md`
Scope: `monke-docs/flash/flash-scope.md`
Arch: `monke-docs/flash/flash-arch.md`
Stack: <primary stack summary>
Next: `/monke-flash:blitz`
```
Bump `Updated:` to today, `by /monke-flash:sketch`
