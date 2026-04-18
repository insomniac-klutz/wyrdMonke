# WyrdMonke — The Drafter's Law

> *how to write a skill that doesn't embarrass monke.*

> **Self-exemption:** this meta-document defines the skill-file rules. It is not itself a skill — no slash command, no phases, no status update. The tagline + section-skeleton requirements it enforces do not apply to it. Every other root-level `.md` and every `monke-<dir>/*.md` obeys the skeleton below.

Every skill in WyrdMonke follows the same skeleton. The content is yours. The structure is sacred. Deviate and the framework fractures — the router can't dispatch, status can't track, gates can't hold.

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
## Context Death Protocol
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
- Every prerequisite failure has a **clear WHAT/WHY/HOW stop message** with a suggested next action (see Section 6).
- Read `monke-status.md` on entry (part of the Status Update Protocol — see Section 8).
- If prerequisites can't be verified → stop with message, don't guess.
- Never silently skip a failed prerequisite.

---

## Section 4: Phases

```markdown
## Phase N: <Name>

<Phase content — what to do, how to do it, what to present.>

⏸ **PG-N [AUTO/SOFT/HARD/TRIGGERED] — <Pause gate description — what the user confirms.>**
<!-- Annotate every gate with its class. See Section 5 for the canonical gate classification table. SOFT gates MUST state an auto-pass condition inline; HARD gates never auto-pass. -->

```

### Rules

- **Numbered sequentially.** Phase 1, Phase 2, ..., Phase N.
- **Each phase has one responsibility.** If a phase does two unrelated things, split it.
- **Gates at every decision point.** Each gate carries its class (AUTO / SOFT / HARD / TRIGGERED). See Section 5 and design-specs S9.4. AUTO gates run silently; HARD gates always surface; SOFT gates surface based on rigor + auto-pass; TRIGGERED gates fire on event only.
- **Explicit user confirmation language (when a gate surfaces):**
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

## Section 5: Gates (Adaptive)

Gates are the heartbeat of the framework. They prevent silent architectural drift, unchecked code generation, and "trust me" engineering. But not every gate pauses — that produced a friction wall that nobody could use. Gates are now **classified by type**, and most run silently when their quality conditions hold.

Every skill declares the **gate type** for every gate it owns. Skills do not invent new classes — they align to design-specs S9.4.

### Gate Classes (canonical — see design-specs S9.4)

| Class | Behavior | Example Gates |
|-------|----------|---------------|
| **AUTO** | Runs silently. Surfaces ONLY on failure. Emits one-line audit log on pass. | IL-0, IL-1, IL-2, IL-3 |
| **HARD** | Always surfaces. Never auto-passes. Never skippable. | PG-1 (scope), PG-6 (non-default language), PG-11 (phase / ship) |
| **SOFT** | Surfaces based on rigor level + auto-pass condition. Self-confirms with audit log when condition holds. | PG-2, PG-3, PG-4, PG-5, PG-7, PG-8, PG-9, PG-10 |
| **TRIGGERED** | Surfaces only when the triggering event fires. Ignores rigor when firing. | PG-12 (stack violation), PG-13 (persistent failure), PG-14 (HLD amend) |
| **Fail gate** | Prerequisite not met. Stop. Message. No override. | Agent Teams Gate; missing required file |

### How Skills Declare Gates

Every gate in a skill's phase section is annotated with its class:

```markdown
⏸ **PG-9 [SOFT] — LLD design converged.** Present converged design.
    Auto-pass when: reviewer reports 0 violations AND all contracts match HLD.
    Confirm / Adjust / Reject?
```

```markdown
⏸ **PG-11 [HARD] — Phase checkpoint.** Always surfaces. Never skippable.
    Confirm / Adjust / Reject?
```

```markdown
[IL-2 AUTO] Run unit tests. On pass, log `[gate:IL-2] auto-confirmed`. On fail, surface failures.
```

### Rules

- **Align to design-specs S9.4.** Do not invent new classes. Do not relabel canonical classes.
- **Declare the class inline.** Every `⏸` line names `[AUTO]` / `[SOFT]` / `[HARD]` / `[TRIGGERED]`.
- **SOFT gates need an auto-pass condition.** State it inline. No hand-wave "if it looks fine."
- **HARD gates never auto-pass.** Do not add an auto-pass line to a HARD gate.
- **Never interpret silence as confirmation.** If the gate surfaces, wait for explicit response.
- **One recommendation at a time when surfacing.** Don't dump a list of 10 actions.
- **Audit log on every outcome.** Pass, auto-pass, fail, user-confirm, user-reject — all get a one-line entry.
- **Rigor is read from `.monke-config.md`**, not hard-coded in skills.
- **Skills do NOT decide when to skip pause.** The gate CLASS decides. `light` rigor is not a license to bypass HARD.

### Rigor read idiom

Every skill with SOFT gates MUST read the active rigor level using the canonical idiom:

```bash
RIGOR="${MONKE_RIGOR:-$(grep '^rigor:' .monke-config.md | awk '{print $2}')}"
```

Env-var first (set by `monke.md` on dispatch), file fallback for direct invocations. Never hardcode. Never default silently — if both are missing, fire an error that asks the user to run `/monke-init`.

### Non-canonical gate prefix

Only the 14 canonical gates in `monke-docs/design-specs.md` §9.4 may use the `PG-<number>` namespace (IL-0..3, PG-1..14). Skill-internal pauses that don't map to a canonical gate MUST use the prefix `SKILL-GATE:<slug>` instead of `PG-<slug>`. Example: `⏸ **SKILL-GATE:draft-confirm [SOFT]** — …` (not `PG-adr-draft`). The classification tag (`[AUTO]/[SOFT]/[HARD]/[TRIGGERED]`) still applies and rigor still governs SOFT behavior; only the namespace differs. This keeps design-specs §9.4's PG-N audit trail clean.

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
| Auto-confirm a HARD gate | Refuse. HARD gates always surface. Human decides. |
| Relabel a HARD gate as SOFT to skip it | Refuse. Gate class is set in design-specs S9.4. Skills do not reclassify. |
| Claim a SOFT gate auto-passed without stating the condition | Refuse. Auto-pass requires an explicit condition check and audit log. |
| Proceed without reading `monke-status.md` | Refuse. Status first, always. |
| Modify an artifact outside this skill's domain | Refuse. Stay in your lane. Suggest the right skill. |
| Continue after a fail gate | Refuse. Stop means stop. |

### Standardized Error Message Format

Every failure message a skill emits — prerequisite fail, gate fail, scan fail, missing file — MUST have three parts:

| Part | What it says | Example |
|------|-------------|---------|
| **WHAT failed** | The specific check, gate, file, or step that failed. | "IL-2 (unit tests) failed for component `auth-service`: 3 of 17 tests failed." |
| **WHY it matters** | What this breaks downstream if ignored. | "Blocks phase checkpoint. Integration tests cannot run. LLD contract for `verify_token` is unproven." |
| **HOW to fix** | The specific command, file, or skill the user should run next. | "Run `/monke-test:test-run auth-service` to see failures, then `/monke-implement:fill auth-service` to patch." |

Bad error (banned):

```
Something went wrong. Please try again.
```

Good error (mandatory shape):

```
WHAT: Agent Teams Gate failed. `CLAUDE.md` exists but has no Agent Teams section.
WHY:  This skill spawns agent teams. Without CLAUDE.md enablement, teammates cannot coordinate.
HOW:  Run `/monke-sync` to pull the latest template, OR copy the Agent Teams section from
      `monke-CLAUDE.md` into your project's `CLAUDE.md`, then re-run this skill.
```

Rules:

- **Three parts, every time.** If you can't name what failed, why it matters, and how to fix it, you shouldn't be emitting the error.
- **No vague verbs.** "Something went wrong" is banned. "Unable to continue" is banned.
- **Cite the artifact.** File path, gate ID, section number — point at it.
- **HOW is actionable.** A command or a file edit, not "investigate."

### Progressive Formalization (design-first vs. code-first)

Not every project starts with a blank page. Skills that operate on existing code follow a different default than skills that operate on new code. Both paths are legitimate.

| Starting state | Default path | How docs emerge |
|---------------|-------------|-----------------|
| **New code** (greenfield, pre-L0) | Design-first: HLD → LLD → code → tests | Docs are written up front, code follows. Canonical path for `thorough` rigor. |
| **Existing code** (any maturity) | Code-first / progressive: auto-generate LLDs from what exists → refine as implementation gaps are filled | LLDs are reverse-engineered, confidence starts at `auto-generated`, upgrades to `reviewed` then `verified` as work progresses. |

Rigor influences the default:

- `light`: progressive formalization acceptable for both greenfield and existing.
- `standard`: progressive acceptable for existing code; design-first default for greenfield.
- `thorough`: design-first enforced for greenfield (see design-specs S12 + sdlc-specs). Existing code still uses progressive but with mandatory SOFT-gate review at PG-9.

Skills declare which path they support in Prerequisites. A skill that only supports design-first MUST stop if it detects existing code without a reviewed LLD. A skill that supports progressive MUST stamp the LLD `Confidence: auto-generated` and refuse to treat auto-generated LLDs as trusted input for neighbor-component contracts.

---

## Section 7: Context Death Protocol

Every skill MUST declare what it does when the context window runs low. Skills that silently die mid-phase leave artifacts half-written and status files lying. Context death is not an edge case — it's a failure mode every long-running skill hits.

Every skill declares three things:

1. **Checkpoint artifacts.** What gets written to disk when context pressure is detected, so a later invocation can resume. Minimum: the draft artifact being worked on, and a resume line in `monke-status.md`.
2. **Status line format for resume.** A one-line signature the next invocation reads to know where to pick up. Include: skill name, phase number, last completed step, last gate outcome, next action.
3. **Recovery detection logic.** How the skill detects, on entry, that the prior invocation died mid-phase (vs. started clean vs. completed).

### Skeleton

```markdown
## Context Death Protocol

**Checkpoint artifacts (written on context pressure):**
- <artifact 1 — e.g. `monke-docs/hld-draft.md` — state of L2 containers so far>
- <artifact 2 — e.g. partial decomposition in `monke-docs/lld/<component>-draft.md`>

**Status line format (written to `monke-status.md`):**

    ## Resume
    Skill: /<dir>:<skill>
    Phase: <N> — <phase name>
    Last step: <specific step completed>
    Last gate: <gate ID — outcome>
    Next action: <what to do when resumed>

**Recovery detection (on entry):**
- If `monke-status.md` Resume block names this skill AND last step is not "Status Update complete" → resume from Next action.
- If checkpoint artifact exists AND status shows this skill mid-phase → read artifact, re-enter at the phase that owns it.
- If neither present → start clean from Phase 1.
```

### Rules

- **Every skill declares this. No exceptions.** A skill without Context Death Protocol is not shippable.
- **Checkpoints are real files, not promises.** The artifact path must exist in the skill's domain.
- **Resume lines are parseable.** The next invocation mechanically reads them — no prose.
- **Recovery is explicit, not implicit.** The skill says "if X then resume from Y." No "probably just run again."
- **The `/monke` dispatcher checkpoints before every dispatch.** The router dies mid-route; its checkpoint is "what was the last dispatched skill and did it complete."

### Resume argument convention

Skills that support mid-flight resume MUST accept `resume:<N>` as the second positional argument (where `<N>` is the phase number to resume at). The orchestrator (`monke.md`) parses the Resume block in `monke-status.md` and appends `resume:<N>` to the dispatched skill's arguments.

Canonical parsing shape inside the skill:

```bash
# Example: /monke-design:lld auth-service resume:3
COMPONENT="$1"
RESUME_PHASE="$(echo "$2" | grep -oE '^resume:[0-9]+$' | cut -d: -f2)"
```

When `RESUME_PHASE` is set, Phase 1 of the skill MUST:
1. Verify the checkpoint artifact exists and is parseable.
2. Skip to Phase `$RESUME_PHASE` with the phase's prerequisites re-validated.
3. If the checkpoint is missing or malformed, fall back to Phase 1 (warn the user: "resume:N specified but checkpoint invalid, restarting").

### Draft-verification lock-file (for multi-section skills)

Skills with substantive multi-phase drafts (LLD generation, HLD generation per-section, flash blitz per-flow, rage:buggy file-scan progress) MUST write a companion lock-file alongside the draft artifact. Format:

**Filename:** `<artifact>.lock` — e.g., `monke-docs/lld/auth-service-draft.md.lock`.

**Content (one key per line, colon-separated):**

```
skill: <cluster>:<skill>
phase: <current-phase-number>
progress: <last-completed-unit-index>/<total-units>
component: <if applicable>
timestamp: <ISO-8601 UTC>
```

**Write protocol.** Update the lock-file at the END of every phase (and every unit within a long phase — e.g., after each function in Layer 2, after each section in HLD).

**Read protocol.** On re-entry with `resume:<N>`:
1. Read `<artifact>.lock`. If missing → warn: "resume:<N> specified but lock-file missing — restarting from Phase 1."
2. If present, verify `skill:` field matches this skill's name. If mismatch → warn and restart.
3. If match, use `progress:` to pick up exactly where work stopped within phase `<N>`.
4. Re-validate any partial artifact content parses (open the draft file and verify structure markers).

**Cleanup.** Delete `<artifact>.lock` when the skill finalizes its deliverable (final artifact written, draft promoted or archived). Stale lock-files confuse future recovery.

---

## Section 8: Status Update

```markdown
## Status Update

On completion, update `monke-status.md`:
- <what to update — specific section, row, checkbox>
- Bump `Updated:` line with current date and skill name
```

### Canonical status line format

Every skill's status line marker MUST follow this exact shape:

```
Where We Are: <cluster>:<skill> — <phase-descriptor> (<progress-counter>)
```

- `<cluster>:<skill>` uses slash-command namespace without the leading slash: `design:hld`, `recon:reconstruct`, `flash:blitz`, `implement:implement`, `test:test-run`, `rage:buggy`, `seer:profile`, `ops:commit`, `status:rebuild`.
- `<phase-descriptor>` is a short present-tense clause (e.g., `drafting manifest section 4`, `Layer 2 — function 3/7`, `iteration 2 fixing`).
- `<progress-counter>` is a parenthesized progress quantifier (e.g., `(3/7)`, `(2 of 4)`, `(cycle=2)`). When no counter makes sense, use the phase name only and omit the parens.

`monke.md` parses this line deterministically to identify which skill is mid-flight for recovery. Skills that deviate from this shape cannot be auto-resumed.

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
| Skill has >8 phases | Split into two skills or promote to `/monke` dispatch |
| Skill reads AND writes artifacts from different domains | Split. Design skills don't implement. Implement skills don't test. |
| Skill has two unrelated decision trees | Two skills, the `/monke` orchestrator to route between them |
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
[ ] Every prerequisite failure: WHAT/WHY/HOW error message with suggested next action
[ ] Phases: numbered, sequential, one responsibility each
[ ] Gates: every ⏸ names its class [AUTO/SOFT/HARD/TRIGGERED]; SOFT gates state auto-pass condition
[ ] Anti-patterns table: minimum 4 rows, "Refuse." verb, skill-specific
[ ] Context Death Protocol: checkpoint artifacts + resume status line + recovery detection declared
[ ] Status Update: read on entry, update on exit (success/blocked/partial)
[ ] Voice: vivid verbs, short metaphors, no corporate speak
[ ] SRP: one skill, one job, no domain crossing
[ ] Sacred Tree: file + tree + mermaid + skill table + CLAUDE template + changelog
[ ] Agent teams: align to Team Decision Heuristic (design-specs S3.1); parallelization noted for >3 components or >20 files
[ ] Progressive formalization: skill declares whether it supports design-first, code-first, or both
```

---

*monke doesn't wing it. monke follows the skeleton, then puts its own skin on.*
