# WyrdMonke Flash:Spark — First Fire

> **Usage:** /monke-flash:spark [napkin]
>
> Drags intent out of the fog. One question at a time — spark captures what monke is about to build, before a single line of code catches.

---

## Arguments

`$ARGUMENTS` parsing:
- None when invoked standalone. This is a conversation skill.
- When dispatched from `/monke` with a napkin argument, spark may skip the conversational rounds and draft `flash-brief.md` directly from the napkin. `/monke` passes the napkin through; spark reads it as context and still confirms the pitch before moving on.

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

- `monke-docs/` directory exists (run `/monke-init` first if missing)
- No existing `monke-docs/flash/flash-brief.md` — if one exists, warn: "A flash brief already exists. Continuing will overwrite it." **Wait for confirmation.**

Create `monke-docs/flash/` if it doesn't exist.

---

## Gate Semantics

Flash runs under light rigor per S9.4/S12. HARD gates (PG-1 in scope, PG-11 in snap) always surface. SOFT gates auto-pass per the adaptive system. TRIGGERED gates fire on their triggers regardless of rigor.

---

## Phase 0: The Pre-Flight (recommended)

Before monke starts asking questions, check if the user has already done homework:

> "Have you hashed out the business case and core tech requirements yet — even loosely? A 10-minute [claude.ai](https://claude.ai) chat about *what* you're building and *why* saves us from discovering halfway through blitz that the idea has no legs."

If user says **yes** (or pastes context):
- Read it. Extract what you can. Skip rounds that are already answered in Phase 1.

If user says **no** or **"just start"**:
- That's fine. Spark handles it. But note: spark captures intent, it doesn't validate business viability. Garbage in, garbage out.

If user says **"what should I ask?"**:
- Suggest they open [claude.ai](https://claude.ai) and answer:
  1. Who has this problem and how badly?
  2. What do they do today? Why is that not good enough?
  3. What's the simplest thing that would prove this idea works?
  4. What tech does this absolutely require? (LLMs, real-time, mobile, specific APIs)
- Then come back with the answers.

---

## Phase 1: Extract Intent

**Talk to the human. One group at a time. Don't vomit a questionnaire.**

### Round 1 — The Pain

Ask:
- What problem does this solve? Who has this problem right now?
- What do they do today instead? (Even if the answer is "nothing" — that's useful.)

If the user is vague, suggest: "Sounds like you need X — is that close?"

### Round 2 — The Shape

Ask:
- What are the **3 things** this MVP must do? Not features — flows. "User does X, system does Y, user sees Z."
- Hard limit: **max 3 key flows**. If user lists more, push back: "That's 3 MVPs wearing a trench coat. Pick the one that proves the idea."

### Round 3 — The Walls

Ask:
- Hard constraints: timeline, budget, platform, regulatory, "must use X because boss said so"
- Stack preferences — or say "monke picks" and the sketch phase will choose the simplest thing that works

### Round 4 — The Spark

Ask:
- Inspiration? Existing products, papers, patterns, "like X but for Y"
- Anything the user has already tried or prototyped

**Don't ask all rounds at once.** Wait for answers. React. Suggest. Push back on scope creep.

---

## Phase 2: Draft the Napkin

Write `monke-docs/flash/flash-brief.md` with this structure:

```markdown
# Flash Brief — <project name>

Created: <date> by /monke-flash:spark

## Problem
<1-3 sentences. What pain. Who feels it.>

## Target User
<Who this is for. Be specific — not "everyone.">

## Key Flows (MVP)
1. <Flow 1 — user does X, system does Y>
2. <Flow 2>
3. <Flow 3>

## Hard Constraints
- <constraint 1>
- <constraint 2>

## Stack
<Preferences or "monke picks">

## Inspiration
<References, prior art, "like X but for Y">
```

**No placeholders. No TBDs.** Every field filled with what the human actually said.

---

## Phase 3: Confirm

Present the flash brief to the user. Read it back, not as a file dump — as a crisp summary:

> "Here's the napkin: You're building **X** for **Y** because **Z**. The MVP does three things: A, B, C. Constraints: D, E. Stack: F. Sound right?"

⏸ **Brief confirmation [SOFT] — napkin matches intent.**
Auto-pass when: every field in the brief template is filled with content drawn from user answers (no TBD, no placeholders, no "monke guesses").
Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when the condition holds.

- **Confirm** -> write the file, move on
- **Adjust** -> edit the brief, re-present
- **Reject** -> start over from Phase 1

---

## Status Update

On completion, update `monke-status.md`:
- Add or update a `## Flash` section:
  ```
  ## Flash
  Phase: **Spark complete**
  Brief: `monke-docs/flash/flash-brief.md`
  Next: `/monke-flash:scope`
  ```
- Bump `Updated:` to today, `by /monke-flash:spark`

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Dump all questions at once | Refuse. One round at a time. Spark is a conversation, not a questionnaire. |
| Skip to code without a brief | Refuse. Garbage in, garbage out. The brief is the contract. |
| Write the brief without user confirmation | Refuse. Present, pause, confirm. The user owns the vision. |
| Scope-creep during spark (add features, plan architecture) | Refuse. Spark captures intent. Scope cuts. Sketch architects. Stay in your lane. |

---

## Context Death Protocol

**Checkpoint artifacts:** `monke-docs/flash/flash-brief.md` (partial draft) written at the end of each round; the round number captured in a `Draft: round-N` comment at the top of the file.
**Status line marker:** `Where We Are:` in `monke-status.md` reads `flash:spark — round <N>/4` while mid-flight.
**Recovery detection:** On re-entry, if `flash-brief.md` exists with a `Draft: round-N` marker → resume at Round N+1; if the file exists but is fully populated and Status marker is cleared → treat as already complete, warn the user before re-running; if neither → start fresh at Phase 0.
