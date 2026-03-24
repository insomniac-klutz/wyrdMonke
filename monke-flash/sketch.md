# WyrdMonke Flash:Sketch — Napkin architecture

> **Usage:** Copy `monke-flash/` to `.claude/commands/monke-flash/`. Invoke: `/monke-flash:sketch`

---

## Arguments

`$ARGUMENTS` parsing:
- None. Reads from `flash-scope.md` and `flash-brief.md`.

---

## Prerequisites

- `monke-docs/flash/flash-scope.md` exists (run `/monke-flash:scope` first)
- `monke-docs/flash/flash-brief.md` exists

---

## Phase 1: Load Context

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
