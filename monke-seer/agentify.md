# WyrdMonke Seer:Agentify — Stupefy in reverse

> **Usage:** `/monke-seer:agentify <component>`
>
> Takes any component — traditional or already agentic — and asks the question nobody's asking: could this be smarter? Or is it already smarter than it needs to be? LATS the intelligence options, proposes pattern upgrades or de-escalations, records the verdict as an ADR. The seer reads the bones. The bones don't lie.

---

## Arguments

`$ARGUMENTS` parsing:
- Single positional: component name from HLD S3 (**required**)

```
COMPONENT="${ARGUMENTS:?Component name required}"
```

If missing → read HLD S3, list all components with their type tags (`traditional`/`agentic`) and current patterns, ask user to pick.

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

- HLD exists with confirmed L3 (S3 component map)
- Target component exists in HLD S3
- Read `monke-status.md` for pipeline context

If no HLD → "No architecture to assess. Run `/monke-design:hld` first. Agentify reads blueprints, not tea leaves."

---

## Phase 1: Read the Patient

Pull everything about the target component from the HLD:

```
Component: <name>
Container: <parent container>
Type tag: <traditional | agentic>
Current pattern: <Anthropic pattern, or "none" if traditional>
Responsibility: <one sentence from HLD S3>
Tools:
  - <tool 1> — <static-contract | versioned-artifact | data-dependent>
  - <tool 2> — ...
Upstream: <what feeds this component>
Downstream: <what this component feeds>
LLD status: <waiting | ready | implemented | "not started">
```

If an LLD exists → read it too. Implementation details reveal complexity the HLD hides.

Present the patient to the user. "Here's what we're looking at."

---

## Phase 2: Run the Diagnostic

Two paths. Which one depends on the patient's current state.

### Path A: Traditional Component → Agentic Candidate?

Apply the Agentic Candidacy Heuristic (design-specs S1.2) signal by signal:

| # | Signal | Present? | Evidence |
|---|--------|----------|----------|
| 1 | Selects among tools or strategies at runtime | | |
| 2 | Handles non-deterministic inputs (output isn't predictable from input) | | |
| 3 | Runs multi-step decisions with feedback loops | | |
| 4 | Consumes versioned-artifact or data-dependent tools | | |
| 5 | Needs context-dependent error recovery (not just retry) | | |
| 6 | Monitors output quality and adjusts behavior | | |

**Score it:**
- **0 signals:** Stay traditional. This is a REST endpoint that parses JSON. It does not need to be an agent. Go home.
- **1-2 signals:** Worth exploring. LATS it in Phase 3 — the cost/benefit might tip either way.
- **3+ signals:** Definitely agentic. The question isn't whether, it's which pattern.

### Path B: Agentic Component → Right Pattern?

Run a Pattern Fitness Check:

**Under-powered signals** (pattern is too simple for the complexity):
- Component has workarounds for things the pattern wasn't designed for
- Error handling is ad-hoc because the pattern's loop can't express it
- "Also does X, Y, Z" responsibilities that don't fit the pattern's decision model
- Prompt Chaining doing Orchestrator-Workers' job with duct tape

**Over-powered signals** (pattern is too complex for the task):
- Fewer than 3 dynamic decision points → the agent loop is mostly boilerplate
- Most tool calls are deterministic and predictable
- The "autonomous" agent follows the same sequence every time
- A calculator wearing a spacesuit

**Right-sized signals:**
- Pattern matches the actual decision complexity
- No workarounds needed
- Agent loop steps all serve a purpose
- Adding complexity would add overhead without capability

Present the diagnostic:

```
Diagnostic: <component>

Current: <traditional | agentic — pattern>
Candidacy score: <N/6 signals> (Path A)
  OR
Pattern fitness: <under-powered | over-powered | right-sized> (Path B)

Evidence: <the specific signals that fired, or the specific fitness indicators>
```

⏸ **PG-3 [SOFT] — Diagnostic confirmed before LATS.** Auto-pass when: Path A score is 0 (stay traditional, no LATS needed) OR Path B returns `right-sized` (no LATS needed). Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds.
Proceed to LATS / Patient is fine / Adjust assessment?

If right-sized or score 0 → skip to Phase 5 (record "assessed, no change needed" as ADR). Don't fix what isn't broken.

---

## Phase 3: LATS the Intelligence

If the diagnostic shows agentic potential (Path A, score ≥ 1) or pattern mismatch (Path B, under/over-powered):

### For Traditional → Agentic Upgrade

Walk the Anthropic complexity ladder **bottom-up** (design-specs S1.3). Start from the simplest pattern that could satisfy the signals:

```
## Design Branch: <component> — Agentic Upgrade?

Option A: Keep traditional.
  What works: <current strengths>
  What's missing: <signals that fired>
  Downstream cost: <workarounds needed, complexity absorbed elsewhere>

Option B: <simplest viable Anthropic pattern>.
  What it enables: <which signals this pattern naturally handles>
  What it costs: <additional complexity, infrastructure, token spend>
  Pattern: <Augmented LLM | Prompt Chaining | Routing | ...>

Option C (if B doesn't cover all signals): <next pattern up>.
  What it enables: <additional capabilities>
  What it costs: <additional complexity>
```

**Always recommend the simplest pattern that covers the fired signals.** If Augmented LLM handles it, don't propose Orchestrator-Workers. Complexity is not intelligence.

### For Pattern Escalation (Under-Powered)

```
## Design Branch: <component> — Pattern Escalation

Option A: Keep <current pattern>. Patch workarounds.
  What works: <what the current pattern handles well>
  What breaks: <where the workarounds live>

Option B: Escalate to <next pattern up the ladder>.
  What it enables: <which pain points this resolves>
  What it costs: <additional complexity>
```

### For Pattern De-escalation (Over-Powered)

```
## Design Branch: <component> — Pattern De-escalation

Option A: Keep <current pattern>. Accept the overhead.
  Overhead: <boilerplate, unnecessary agent loop steps, token waste>

Option B: De-escalate to <simpler pattern>.
  What simplifies: <removed complexity>
  What's preserved: <capabilities that still work at the simpler level>

Option C: De-escalate to traditional.
  What simplifies: <everything>
  Risk: <what you lose — any of the 6 signals that genuinely fire>
```

⏸ **PG-5 [SOFT] — Present LATS options.** Auto-passes when the recommended option has clear advantage — dominates runner-up on ≥2 constraints with no trade-off loss. Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when the dominance condition holds.

---

## Phase 4: Propose the Change

Based on the confirmed LATS selection:

### If Upgrading to Agentic

Draft a CoALA summary for the new pattern:

```
### Agent: <component name>
Pattern: <selected Anthropic pattern>
Loop: observe → <relevant steps> → loop/terminate
Memory: working(<budget>), episodic(<store>), semantic(<store>), procedural(<location>)
Actions: internal(<strategies>), external(<tools: subtype>), boundaries(<cannot do>)
Stops when: <condition>
Human-in-loop: <where>
Version pins: <artifact: version> (if versioned-artifact tools)
Eval thresholds: <metric: threshold> (if versioned-artifact or data-dependent tools)
```

### If De-escalating

Draft the simplified component entry — what gets removed from the CoALA summary, what stays as traditional logic.

### If Keeping Current

No draft needed. Skip to Phase 5.

---

## Phase 5: Impact Manifest + Record

Present the full blast radius before writing anything:

```
Agentify Impact Manifest

Component: <name>
Verdict: <upgrade to <pattern> | de-escalate to <pattern/traditional> | keep current>

HLD changes:
  - L3 type tag: <traditional → agentic | agentic → traditional | unchanged>
  - L3 CoALA summary: <new | updated | removed | unchanged>
  - S7 boundary matrix: <stability column changes if tool subtypes change>
  - S8 phase plan: <any re-phasing needed>

Downstream cascade:
  - LLD: <needs creation | needs rewrite | needs update | unchanged>
  - Implementation: <needs rewrite | needs update | unchanged>
  - Tests: <eval tier added/removed, integration tests affected>

ADR: will be dispatched to /monke-design:adr regardless of verdict
```

⏸ **PG-7 [HARD] — Impact manifest confirmed before any writes.** Always surfaces. Never skippable. Pattern changes cascade through HLD/LLD/implementation — confirm the blast radius before committing.
Confirm / Adjust / Reject?

### 5.1 Assemble the ADR Body

Build the structured content the ADR skill will write. Keep the body in memory; don't touch disk yet:

```markdown
# ADR-NNN: <Component> — Intelligence Assessment
Status: accepted
Date: YYYY-MM-DD  |  Component: HLD S-X.Y
Type: agentify

## Assessment
<Current state, diagnostic results, signal scores or fitness check>

## Options (LATS output)
<From Phase 3>

## Decision — <verdict> because <evidence>
## Consequences — <what changes in HLD/LLD/implementation>
```

### 5.2 Dispatch to the ADR Skill

**This skill does not write the ADR file directly.** Dispatch to `/monke-design:adr` for auto-numbering and writing:

- `type: agentify`
- `component: <HLD S3 component>`
- `slug: agentify`
- `body: <the markdown body from 5.1, with the `ADR-NNN` placeholder left untouched — the ADR skill assigns the number>`

The ADR skill handles numbering (scans existing `monke-docs/decisions/` for the next free `NNN`), writes the file, and returns the assigned path. Do not write to `monke-docs/decisions/` directly — that is how collisions happen when experiment + agentify + design skills all run in the same wave.

### 5.3 Hand Off

- **If upgrading:** "Run `/monke-design:hld evolve repattern <component>` to apply the pattern change to the HLD."
- **If de-escalating:** Same — repattern handles both directions.
- **If keeping current:** "No changes needed. The ADR records the assessment for posterity."
- **If the upgrade needs data profiling first:** "Run `/monke-seer:profile <data-source>` before evolving — the CoALA summary needs distributional expectations."
- Always: "Run `/monke-status:status` to see updated dashboard."

---

## Where This Connects

- **design-specs S1.2:** Agentic Candidacy Heuristic — the diagnostic framework
- **design-specs S1.3:** Anthropic Composable Patterns — the complexity ladder to walk
- **design-specs S1.4:** LATS — the decision exploration protocol
- **design-specs S7:** ADR — records the intelligence assessment decision
- **`/monke-design:hld evolve repattern`:** Applies the actual HLD change if upgrade/de-escalation confirmed
- **monke-seer/profile.md:** Data profiling needed before upgrading to agentic if component consumes versioned-artifact or data-dependent tools
- **monke-seer/experiment.md:** If the LATS decision needs metric evidence, run an experiment first

---

## Anti-Patterns to Refuse

| If you find yourself... | Stop. Do this instead. |
|------------------------|------------------------|
| Making everything an agent | Most components are REST endpoints that parse JSON. Leave them alone. Intelligence has a cost. |
| Skipping the diagnostic, just adding an LLM call | Adding an LLM doesn't make it intelligent. It makes it expensive and non-deterministic. Run the 6-signal check. |
| Proposing Autonomous Agent for a CRUD endpoint | That's a calculator wearing a spacesuit. Start at the bottom of the ladder. |
| De-escalating without evidence | "It seems too complex" is not evidence. Count the dynamic decision points. If it's ≥3, the pattern is earning its keep. |
| Agentifying without profiling the data first | If the component will consume versioned-artifact or data-dependent tools, profile first. You can't write a CoALA summary without knowing what the data looks like. |
| Upgrading because "AI is the future" | Agentic patterns add complexity, latency, cost, and non-determinism. They're justified when the 6 signals fire. Hype is not a signal. |
| Skipping the ADR for "keep current" verdicts | Record every assessment. Future-you needs to know that this component was evaluated and deliberately kept traditional. Otherwise someone else will agentify-assess it again in three months. |
| Writing the ADR file directly to `monke-docs/decisions/NNN-*.md` | Refuse. The `NNN` numbering is owned by `/monke-design:adr` — writing directly causes collisions when experiment + agentify run in the same wave. Dispatch to `/monke-design:adr` with the body from Phase 5.1 and let it assign the number. |

---

## Gate Classifications Used Here

| Gate | Type | Notes |
|------|------|-------|
| PG-3 | SOFT | Diagnostic gate. Auto-pass: Path A score is 0 (stay traditional) OR Path B returns `right-sized` (no change needed). |
| PG-5 | SOFT | LATS selection. Auto-pass: recommended option has clear advantage — dominates runner-up on ≥2 constraints with no trade-off loss. |
| PG-7 | HARD | Impact manifest gate. Always surfaces before HLD/LLD/implementation cascade is confirmed. Never auto-passes. |

---

## Context Death Protocol

**Checkpoint artifacts (written on context pressure):**
- `monke-docs/decisions/.agentify-drafts/<component>.md` — assembled ADR body (Phase 5.1) before dispatch to `/monke-design:adr`.
- Diagnostic ledger noting which of Path A / Path B has been run and its results.

**Status line marker:** `Where We Are:` in `monke-status.md` reads `seer:agentify — <component> (phase <N>, <verdict-draft>)` while mid-flight.

**Recovery detection (on entry):**
- If `monke-status.md` Resume block names `/monke-seer:agentify` AND draft ADR exists in `.agentify-drafts/` → resume at the phase named in Resume.
- If draft exists but marker cleared → check if `/monke-design:adr` already wrote the ADR for this component; if yes, clean up and exit; if no, re-present Phase 5 gate.
- If neither present → start fresh from Phase 1.

---

## Status Update

**Read on entry:** `monke-status.md` — check the component's current type tag (traditional/agentic) and existing pattern to frame whether this is a Path A (traditional → agentic candidacy) or Path B (pattern fitness) assessment.

**Write on exit:**
- **Success:** bump `Updated:` with today's date + `by /monke-seer:agentify`. Note the assessment in "Where We Are": `Agentify assessed <component> — <verdict>`. Append to Gate Audit Log: `- PG-7 AGENTIFY <component> — <verdict> (ADR-NNN)` using the ADR path returned by `/monke-design:adr`. If upgrade/de-escalation, flag pending HLD evolution.
- **Blocked:** if data profiling is required first (versioned-artifact or data-dependent tools) and not done, add a row to Open Blockers with WHAT/WHY/HOW pointing to `/monke-seer:profile`.
- **Partial:** write a `Resume:` block naming the phase + current verdict draft for context-death recovery.
