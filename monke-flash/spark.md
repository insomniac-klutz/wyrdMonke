# WyrdMonke Flash:Spark — What are we building?

> **Usage:** Copy `monke-flash/` to `.claude/commands/monke-flash/`. Invoke: `/monke-flash:spark`

---

## Arguments

`$ARGUMENTS` parsing:
- None. This is a conversation skill.

---

## Prerequisites

- `monke-docs/` directory exists (run `/monke-init` first if missing)
- No existing `monke-docs/flash/flash-brief.md` — if one exists, warn: "A flash brief already exists. Continuing will overwrite it." **Wait for confirmation.**

Create `monke-docs/flash/` if it doesn't exist.

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

**Wait for confirmation.**

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
