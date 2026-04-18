# WyrdMonke Recon:OQs — The Fog Index

> **Usage:** Copy `monke-recon/` to `.claude/commands/monke-recon/`. Invoke: `/monke-recon:oqs`

---

## Arguments

`$ARGUMENTS` parsing:
- No arguments. OQs are mined from all available recon artifacts.
- Example: `/monke-recon:oqs`

```
# No arguments — full OQ mining pass
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

Read `monke-status.md` to verify current state. Then check:

- **`monke-docs/recon/recon-survey.md` exists** — need the raw inventory. If missing → "Run `/monke-recon:survey` first." Stop.
- **`monke-docs/hld.md` exists** — need the reconstructed architecture. If missing → "Run `/monke-recon:reconstruct` first." Stop.

Recommended but not required:
- `monke-docs/recon/recon-gaps.md` — gaps feed into OQs. Without it, you're mining blind on the "what's missing" front.
- `monke-docs/lld/*.md` — component-level detail sharpens boundary questions.
- `monke-docs/flash/flash-manifest.md` — flash shortcuts are OQ goldmines.

**LLD trust check.** Before using LLDs to mine open questions about cross-component contracts, read the `Confidence:` header of each LLD consumed. If `Confidence: auto-generated` AND the component's contract feeds a boundary-level OQ, surface a warning (not a gate — just advisory):

> OQ mining is consuming an auto-generated LLD (`<component>.md`) for boundary reasoning. Auto-generated LLDs may produce OQs that are actually contract ambiguities (missing type info, inferred signatures), not real decisions in need of review. Recommend: run `/monke-design:lld <component>` review first, or mark OQs touching `<component>` as provisional in the output.

If `Confidence: reviewed` or `verified` → proceed normally.

---

## What OQs Are (And Aren't)

**OQs are architectural decisions that were made implicitly during the MVP and need explicit review before production.**

OQs are NOT the same as gaps:
- **Gap** = something is concretely missing (no error handling, no tests, no auth). Known problem, known fix.
- **OQ** = a decision was made without discussion. The current approach MIGHT be right, but nobody actually decided. Needs conscious confirmation or change.

If `/monke-recon:gaps` already covers it, don't raise it as an OQ.

---

## Phase 1: Implicit Decision Mining

Read the reconstructed HLD and any LLDs. For each component, surface decisions that were made by default rather than by design.

### 1.1 Data Model Questions

- **Why this data model?** Is it the right model or just the fast one? Are there entities that should be separate but got merged? Relations that should exist but don't?
- **Why these field types?** String where an enum would be safer? Number where a custom type would be clearer? JSON blob where a typed schema would be better?
- **Why this storage pattern?** Direct DB? File system? In-memory? Was it chosen or just happened?

### 1.2 Communication Pattern Questions

- **Why REST?** (or GraphQL, gRPC, WebSocket, events, etc.) Was the protocol chosen for a reason, or is it the default for the framework?
- **Why synchronous?** Could this be async? Should it be? Does it matter at current scale?
- **Why this serialization?** JSON everywhere? Is that intentional or just easy?

### 1.3 Error Handling Strategy Questions

- **Why this error strategy?** (exceptions, result types, error codes, or nothing at all) Was it chosen or just the language default?
- **What happens when things fail?** Is there a retry strategy? A fallback? Or does it just crash?
- **Are errors meaningful?** Do consumers get useful error information or generic 500s?

### 1.4 State Management Questions

- **Where does state live?** In-memory? Database? Session? Local storage? Is there a reason?
- **What's the consistency model?** Eventual? Strong? "Whatever the ORM does"?
- **What happens on restart?** Is state lost? Recoverable? Does it matter?

### 1.5 Agentic Pattern Questions (if applicable)

- **Why this LLM pattern?** Is it the simplest that works, or was it the first one that came to mind?
- **Why this prompt structure?** Engineered or vibes-based?
- **What are the stopping conditions?** Explicit or "it usually works out"?
- **What are the action boundaries?** What can the agent NOT do? Is that enforced?
- **What's the token budget strategy?** Is there one? What happens when context overflows?

---

## Phase 2: Boundary Questions

For each boundary in the HLD S7 Boundary Matrix:

- **Is the contract typed and explicit?** Or is it implicit/stringly-typed? An `implicit` status in the matrix is an automatic OQ.
- **Is error propagation defined?** What errors can cross this boundary? Are they documented or accidental?
- **Is the serialization format intentional?** JSON by default vs JSON by design are different things.
- **Who owns schema evolution?** If the upstream changes, what breaks downstream? Is there versioning?
- **Are there hidden couplings?** Shared databases, shared config, shared state that bypasses the boundary?

---

## Phase 3: Scale Questions

These questions might not matter now. They matter at 10x.

- **What happens at 10x load?** Which component breaks first? Where are the bottlenecks?
- **What happens if an external service goes down?** Is there graceful degradation, or does everything cascade-fail?
- **What happens if the database grows to 10M rows?** Are queries indexed for that? Are there any full-table scans?
- **What happens with concurrent users?** Race conditions? Optimistic locking? "Yolo last-write-wins"?
- **What's the deployment story?** Single instance? Horizontally scalable? Stateful or stateless?

Don't raise scale OQs for systems that will genuinely never face them. A personal CLI tool doesn't need to answer "what about 10K concurrent users."

---

## Phase 4: Business Logic Questions

- **Are there business rules hardcoded that should be configurable?** Magic numbers, hardcoded thresholds, embedded policy.
- **Are there edge cases the MVP ignores that production can't?** Empty states, bulk operations, timezone handling, unicode, localization.
- **Are there compliance/legal requirements not yet addressed?** GDPR, data retention, audit logging, accessibility mandates.
- **Are there implicit assumptions about the user?** Single-tenant? Single-user? Admin-only? Will that change?

---

## Phase 5: Flash Shortcut Mining (Flash Mode Only)

If `monke-docs/flash/flash-manifest.md` exists:

Read the manifest's shortcuts and deferred items. **Each shortcut is a potential OQ.** For each:

- Was the shortcut a temporary simplification or an architectural decision?
- Does the shortcut need to be unwound, or is it actually fine for production?
- What breaks if this shortcut stays in production?

---

## Output

Write to `monke-docs/open-questions.md` using the format from `design-specs.md` S8:

```markdown
# Open Questions
Project: <name> | Generated: <today> by /monke-recon:oqs

## Implicit Decisions
### OQ-001: <Question>
Discovered-during: recon
Affects: HLD S-X.Y | LLD <component>
Blocks: <what this blocks if unresolved, or "nothing — confirmation only">
Options so far: A) keep as-is, B) <alternative>
Status: open

### OQ-002: ...

## Boundary Questions
### OQ-NNN: ...

## Scale Questions
### OQ-NNN: ...

## Business Logic
### OQ-NNN: ...

## Flash Shortcuts (if applicable)
### OQ-NNN: ...
```

**Number OQs sequentially: OQ-001, OQ-002, etc.**

Each OQ must have:
- A clear, specific question (not "should we improve error handling?" but "the /api/users endpoint catches all exceptions and returns 200 — is that intentional?")
- What it affects in the HLD/LLD
- What it blocks (or "confirmation only" if it's just validating an implicit decision)
- At least one option beyond "keep as-is"

---

## Decision Gate

⏸ **OQ review [SOFT] — questions triaged.**
Auto-pass when: every OQ has `Discovered-during`, `Affects`, `Blocks`, and at least one `Option` beyond "keep as-is"; no OQ duplicates a row in `recon-gaps.md`; numbering is sequential with no gaps.
Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when the condition holds.

Show counts per dimension first. Then walk through each OQ. User can:
- **Confirm** → mark as "resolved → confirmed as-is" (decision was fine, just implicit)
- **Mark N/A** → remove from list (not relevant to this project)
- **Prioritize as blocker** → add `Blocks: production` and move to critical
- **Answer** → provide resolution, mark as "resolved → <decision>"

Update `open-questions.md` with user's responses before finalizing.

---

## Key Behaviors

- **Focus on decisions that MATTER for production.** Not theoretical purity. "You're using REST when gRPC would be 3% faster" is not a useful OQ. "You have no error contract at your main API boundary" is.
- **Don't duplicate gaps.** If `/monke-recon:gaps` already flagged "no input validation on /api/users," don't raise an OQ asking "should we validate input on /api/users?" The gap is the answer.
- **Every shortcut is suspect.** Flash shortcuts especially. They were explicitly marked as "good enough for now" — now is over.
- **OQs with "Blocks: nothing" are still valuable.** Confirming an implicit decision IS valuable. It turns an accident into a choice.
- **Be specific.** "Should we rethink the data model?" is useless. "The User entity has a JSON blob for preferences — should those be separate columns for queryability?" is useful.
- **Aim for 130-160 lines.** Dense questions, not essays. Each OQ is 5-6 lines max.

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Raise OQs that duplicate gaps | Refuse. Gap = missing thing. OQ = implicit decision. If `/monke-recon:gaps` already flagged it, don't re-raise. |
| Write vague OQs ("should we rethink the data model?") | Refuse. Be specific: "The User entity has a JSON blob for preferences — should those be separate columns for queryability?" Vague questions can't be resolved. |
| Demand answers for OQs with `Blocks: nothing` | Refuse. Confirming an implicit decision is valuable on its own. It turns an accident into a choice. Don't pressure. |
| Raise theoretical-purity OQs ("REST vs gRPC for 3% speed") | Refuse. Focus on decisions that MATTER for production — error contracts, scale breaks, compliance, data integrity. |
| Skip flash-manifest shortcut mining when the manifest exists | Refuse. Every shortcut is suspect. "Good enough for now" is over — each shortcut becomes an OQ unless already a gap. |
| Auto-resolve OQs without user input | Refuse. OQs require human sign-off. Monke surfaces, human decides. |

---

## Context Death Protocol

**Checkpoint artifacts:** `monke-docs/open-questions.md` (partial draft) written after each dimension (Implicit Decisions, Boundaries, Scale, Business Logic, Flash Shortcuts). Numbering stays sequential across dimensions.
**Status line marker:** `Where We Are:` in `monke-status.md` reads `recon:oqs — mining <dimension>` while mid-flight.
**Recovery detection:** On re-entry, if `open-questions.md` exists with some dimension headers populated → resume at first empty dimension (respecting existing OQ-N numbering); if all dimensions covered but Decision Gate not logged → re-present grouped summary; if file absent → start fresh at Phase 1.

---

## Status Update

On completion, update `monke-status.md`:
- **Recon section:** Mark `[x] Open questions — <date>`
- **Open Blockers table:** Add any OQs that have `Blocks: production` or `Blocks: <component>`
- Set `Updated:` to today, `by /monke-recon:oqs`
- Update "Where We Are": `Phase: **OQs surfaced — ready for production roadmap**`
- Update "Next action": `/monke-recon:roadmap`
