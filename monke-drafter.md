# monke-drafter.md

> *how to write a skill that doesn't embarrass monke.*

Every skill in WyrdMonke follows the same skeleton. The content is yours. The structure is sacred. Deviate and the framework fractures — orchestras can't dispatch, status can't track, gates can't hold.

This document is the canonical reference. When creating or modifying any skill, follow this exactly.

---

## The Skeleton

Every skill file has these sections, in this order. No exceptions.

```
# WyrdMonke <Dir>:<Skill> — <Tagline>
> Usage block
---
## Arguments
---
## Prerequisites
---
## Phase 1: <Name>
## Phase 2: <Name>
...
---
## Anti-Patterns to Refuse
---
## Status Update
```

Skip a section, shame on monke. Reorder them, shame on the tree.

---

## Section 1: Title + Usage Block

```markdown
# WyrdMonke <Dir>:<Skill> — <Punchy Tagline>

> **Usage:** `/<dir>:<skill> [args]`
>
> <One to three sentences. Vivid verb. What it does, not what it is. Monke voice.>
```

### Rules

- **Title format:** `# WyrdMonke <Dir>:<Skill> — <Tagline>` — always. The tagline is a short metaphor, not a description. Think newspaper headline, not documentation.
- **Usage line:** the exact slash command with argument placeholders.
- **Description:** present tense, active voice, personality. "Hunts bugs" not "A tool for finding bugs." "Decomposes until it begs for mercy" not "Performs recursive decomposition."

### Examples (good)

```
# WyrdMonke Rage:Buggy — Angry Monke Smells Something Wrong
# WyrdMonke Flash:Snap — Good enough. Freeze it.
# WyrdMonke Seer:Agentify — Stupefy in reverse
# WyrdMonke Design:HLD — The Grand Dreaming
```

### Examples (bad — never do these)

```
# Buggy Scan Tool                          ← no Dir:Skill, no personality
# WyrdMonke Design:HLD — High Level Design ← tagline is a definition, not a metaphor
# WyrdMonke Test:Coverage — Coverage Tool   ← "Tool" is a banned word in taglines
```

---

## Section 2: Arguments

```markdown
## Arguments

`$ARGUMENTS` parsing:
- <description of each positional argument>
- <defaults, fallbacks, validation>

\```
ARG="${ARGUMENTS:-default}"
\```
```

### Rules

- Always show the bash variable assignment, even if trivial.
- If arguments are missing → list available options, ask user to pick. Never guess.
- If arguments are invalid → show valid options, stop.
- Default behavior when called with no arguments must be defined.

---

## Section 3: Prerequisites

```markdown
## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

- <skill-specific prerequisite 1>
- <skill-specific prerequisite 2>
- ...

If <prerequisite fails> → "<clear message with what to run instead>." Stop.
```

### Rules

- **Agent Teams Gate is ALWAYS the first prerequisite.** No exceptions. No skills run without it.
- Every prerequisite failure has a **clear stop message** with a suggested next action.
- Read `monke-status.md` on entry (part of the Status Update Protocol — see Section 7).
- If prerequisites can't be verified → stop with message, don't guess.
- Never silently skip a failed prerequisite.

---

## Section 4: Phases

```markdown
## Phase N: <Name>

<Phase content — what to do, how to do it, what to present.>

⏸ **<Pause gate description — what the user confirms.>**
```

### Rules

- **Numbered sequentially.** Phase 1, Phase 2, ..., Phase N.
- **Each phase has one responsibility.** If a phase does two unrelated things, split it.
- **Pause gates (⏸) at every decision point.** Monke presents, human decides. Never auto-confirm.
- **Explicit user confirmation language:**
  - `Confirm / Adjust / Reject` for design artifacts
  - `Save / Focus / Dismiss / Rerun / Done` for scan results
  - `Yes / Skip / Explain` for recommendations
- **Separation of concerns:**
  - Phase reads context → Phase does work → Phase presents result → Phase gates on user
  - One phase should not silently modify an artifact from another phase's domain.
- **Agent teams parallelization:** If the phase involves scanning >20 files or processing >3 components, mention: "If >N items, use agent teams to parallelize." Provide the split strategy (by directory, by container, by component).

### Phase Naming Conventions

| Pattern | When to use |
|---------|------------|
| `Phase 1: Target Acquisition` | Scope resolution (rage skills) |
| `Phase 1: Read the Patient` | Context loading from existing artifacts |
| `Phase 1: Load Context` | General context gathering |
| `Phase N: Scan` | Reading/analyzing files |
| `Phase N: Triage` | Classifying findings by severity/priority |
| `Phase N: Report` | Presenting results to user |
| `Phase N: Save` | Writing output artifacts |
| `Phase N: Finalize` | Cleanup, status update, next steps |

These are conventions, not mandates. Name phases for clarity.

---

## Section 5: Pause Gates

Pause gates are the heartbeat of the framework. They prevent silent architectural drift, unchecked code generation, and "trust me" engineering.

### Gate Types

| Gate | When | User options |
|------|------|-------------|
| **Decision gate** | Before writing any artifact or making any choice | Confirm / Adjust / Reject |
| **Presentation gate** | Before proceeding past a diagnostic or analysis | Acknowledged / Investigate / Skip |
| **PG-N gates** | Design-specs defined gates (PG-1 through PG-14) | As specified in design-specs |
| **IL-N gates** | Implementation-specs defined gates (IL-0 through IL-3) | Automatic (run commands, check output) |
| **Fail gate** | Prerequisite not met, persistent failure | Stop. Message. No override. |

### Rules

- **Every gate pauses. Every. Single. One.** Auto-confirm is forbidden.
- **Never interpret silence as confirmation.** If the user doesn't respond, wait.
- **One recommendation at a time.** Don't dump a list of 10 actions. Present one, let the user decide, then present the next.
- **IL gates are self-verification** (run commands, check output). They don't require user confirmation — but they DO block progress if they fail.
- **PG gates require explicit user confirmation.** The user says "confirm" or the gate doesn't open.

---

## Section 6: Anti-Patterns to Refuse

```markdown
## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| <bad pattern> | Refuse. <why and what to do instead> |
| <bad pattern> | Refuse. <why and what to do instead> |
```

### Rules

- **Every skill has this section.** No exceptions.
- **Table format.** Two columns: temptation and correction.
- **Start with "Refuse."** — the verb is non-negotiable. Then explain why and suggest the alternative.
- **Minimum 4 rows.** If you can't think of 4 anti-patterns, you haven't thought hard enough about how the skill could be misused.
- **Skill-specific, not generic.** "Don't skip tests" is generic. "Don't mock both sides of a boundary in integration tests" is specific.

### Universal Anti-Patterns (every skill inherits these)

These apply to ALL skills and don't need to be repeated in each file — they're enforced by the framework:

| If asked to... | Do instead... |
|----------------|--------------|
| Skip the Agent Teams Gate | Refuse. Hard gate. No exceptions. |
| Auto-confirm a pause gate | Refuse. Every gate pauses. Human decides. |
| Proceed without reading `monke-status.md` | Refuse. Status first, always. |
| Modify an artifact outside this skill's domain | Refuse. Stay in your lane. Suggest the right skill. |
| Continue after a fail gate | Refuse. Stop means stop. |

---

## Section 7: Status Update

```markdown
## Status Update

On completion, update `monke-status.md`:
- <what to update — specific section, row, checkbox>
- Bump `Updated:` line with current date and skill name
```

### The Status Update Protocol

Every skill follows this protocol. It's not optional.

**On entry:**
1. Read `monke-status.md`.
2. Verify prerequisites are met (including Agent Teams Gate).
3. If prerequisites fail → stop with message and suggested next action.

**On exit (success):**
1. Update the relevant row/checkbox in `monke-status.md`.
2. Bump the `Updated:` line with current date and `by /<dir>:<skill>`.
3. Recalculate "Where We Are" and "Next action" if applicable.

**On exit (blocked):**
1. Add to "Open Blockers" table.
2. Update "Where We Are" to reflect the block.
3. Suggest `/monke-design:oq triage`.

**On exit (partial):**
1. Update progress (e.g., "3/7 functions" in L2 column).
2. Keep status as "in progress".
3. Note resume point so the next invocation can pick up.

---

## The Voice

WyrdMonke skills speak with personality. Not corporate. Not academic. Not passive.

### Do

- **Vivid verbs:** hunts, smells, decomposes, traces, freezes, exorcises, buries
- **Short metaphors:** "the anxiety manager," "stupefy in reverse," "the honest confession"
- **Direct address:** "If it's broken, monke finds it." Not "The tool identifies defects."
- **Irreverent but precise:** personality in the wrapper, rigor in the content
- **Present tense, active voice:** "Monke scans" not "Scanning will be performed"

### Don't

- **Corporate speak:** "leverage," "synergize," "best practices," "enterprise-grade"
- **Passive voice:** "The file is read" → "Read the file"
- **Hedging:** "This might help" → "This finds X"
- **Verbose explanations where a table works:** if it's structured data, use a table
- **Emojis in skill bodies:** emojis live in templates (rage-run) and nowhere else

### The Three Laws of Monke Voice

1. **Lead with the verb.** "Hunts bugs" not "A comprehensive bug detection mechanism."
2. **One sentence beats three.** If you need a paragraph, you're over-explaining.
3. **Personality in the chrome, precision in the engine.** The tagline is irreverent. The phase logic is surgical.

---

## Single Responsibility Principle

Every skill does ONE thing. If your skill description needs "and" — split it.

| Signal | What to do |
|--------|-----------|
| Skill has >8 phases | Split into two skills or promote to an orchestra |
| Skill reads AND writes artifacts from different domains | Split. Design skills don't implement. Implement skills don't test. |
| Skill has two unrelated decision trees | Two skills, one orchestra to route between them |
| Skill's tagline needs a comma | One tagline, one skill. The comma is a code smell. |

### Domain Boundaries

| Domain | Skills own | Skills DON'T own |
|--------|-----------|-----------------|
| Design | HLD, LLD, ADR, OQ | Source code, test code |
| Implement | Source code, Layer 0-3 pipeline | HLD/LLD creation (escalate via PG-14) |
| Test | Test plans, test execution, coverage | Source code fixes (surface to user) |
| Rage | Scan findings, rage-run logs | Code fixes (present, don't apply without confirm) |
| Seer | Profiles, experiments, registry pins | Implementation changes (hand off to design/implement) |
| Flash | Brief, scope, arch, MVP code, manifest | Production architecture (hand off to recon) |
| Recon | Survey, reconstructed HLD/LLDs, gaps, OQs, roadmap | Implementation (connects back to implement pipeline) |

If a skill needs to modify something outside its domain → **suggest the right skill**, don't do it inline.

---

## Orchestra Skills

Orchestra skills are special. They route, they don't execute. They read the status board, diagnose, recommend ONE action, wait for user confirmation, execute the recommended skill inline, update status, and loop.

### Orchestra Structure

```markdown
# WyrdMonke <Dir>:Orchestra — <Tagline>

## Arguments
## Prerequisites (with Agent Teams Gate)
## The Loop (read → diagnose → recommend → wait → execute → update → loop)
## Decision Tree (top-to-bottom, first match wins)
## Behaviors (phase order, parallelization, escalation rules)
## Anti-Patterns to Refuse
## Status Update
```

### Orchestra Rules

- **One recommendation per iteration.** Not a list. One banana.
- **Decision tree, not a flowchart.** Top-to-bottom, first match wins.
- **User can override.** Orchestra recommends, human decides.
- **Context death recovery.** If the context window exhausts, write status and die gracefully. Next invocation picks up from status.
- **Never add meta-status.** Orchestra doesn't write "orchestra is running" to the status file. It writes what the dispatched skill accomplished.

---

## The Sacred Tree Contract

When you create a new skill:

1. **Add the file** to its skill directory.
2. **Update the Sacred Tree** in `README.md` — add a line with a punchy comment. Match the tone.
3. **Update `monke-mermaid.mmd`** — add the node to the subgraph, add edges for what it reads/writes/dispatches.
4. **Update the skill table** in `README.md` — add a row in the appropriate section.
5. **Update `monke-CLAUDE.md`** — add a row to the Skills table.
6. **Update `monke-log.md`** — record the addition in the current version's Added section.

**The chain is non-negotiable:** file → Sacred Tree → mermaid → skill table → CLAUDE template → changelog. Skip a step, shame on monke.

---

## Checklist — Before You Ship a Skill

```
[ ] Title: # WyrdMonke <Dir>:<Skill> — <Tagline> (punchy, not bland)
[ ] Usage block: slash command + one-sentence monke-voice description
[ ] Arguments: $ARGUMENTS parsing with bash variable, defaults, fallback behavior
[ ] Prerequisites: Agent Teams Gate FIRST, then skill-specific checks
[ ] Every prerequisite failure: clear stop message with suggested next action
[ ] Phases: numbered, sequential, one responsibility each
[ ] Pause gates: ⏸ at every decision point, user confirms before proceeding
[ ] Anti-patterns table: minimum 4 rows, "Refuse." verb, skill-specific
[ ] Status Update: read on entry, update on exit (success/blocked/partial)
[ ] Voice: vivid verbs, short metaphors, no corporate speak
[ ] SRP: one skill, one job, no domain crossing
[ ] Sacred Tree: file + tree + mermaid + skill table + CLAUDE template + changelog
[ ] Agent teams: parallelization mentioned where applicable (>20 files, >3 components)
```

---

*monke doesn't wing it. monke follows the skeleton, then puts its own skin on.*
