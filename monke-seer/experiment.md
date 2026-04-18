# WyrdMonke Seer:Experiment — The Bake-Off

> **Usage:** `/monke-seer:experiment <component> <hypothesis>`
>
> When the design has multiple viable approaches and "I think A is better" isn't good enough — run an experiment. This is LATS with metric evidence instead of opinions. The results become ADRs. The rejected approaches stay on file for when you need to backtrack.

---

## Arguments

`$ARGUMENTS` parsing:
- First positional: component name from HLD S3 (**required**)
- Rest: hypothesis or question to test (**required**)

```
COMPONENT="${1:?Component name required}"
HYPOTHESIS="${*:2:?Hypothesis required}"
```

If either missing → ask. "Which component? And what's the question — which model? which prompt strategy? which pipeline? Monke can't experiment on vibes."

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

- Component exists in HLD S3 with at least one versioned-artifact tool
- Data source profiled (`/monke-seer:profile`) — experiments without profiled data are blind taste tests
- `monke-docs/decisions/` directory exists

If no profile exists → "Profile the data first. You wouldn't bake without tasting the flour."

---

## When to Fire

- **At HLD L3** when choosing between model approaches for a versioned-artifact component — PG-5/PG-7 fires, "both are fine" is forbidden, and you don't have numbers yet
- **At LLD** when choosing between feature sets, preprocessing strategies, prompt designs, or model configurations — your gut says Option A, the math might disagree
- **When an eval test fails** and the current approach needs reconsideration — PG-13 fires, and the question is "wrong approach, or wrong tuning?"

---

## The Flow

```
1. Profile the data first (/monke-seer:profile)
2. Frame the question (Phase 1)
3. Design the experiment (Phase 2)
4. Run it (Phase 3)
5. Read the numbers honestly (Phase 3)
6. Record the decision as an ADR (Phase 4)
7. Pin the winner (/monke-seer:registry)
```

---

## Phase 1: Frame the Question

Read the component's HLD entry. Read its CoALA summary if agentic. Read any existing profiles.

Then frame the experiment as a LATS branch:

```
## Design Branch: <component> — <question>

Contender A: <approach>.
  Metric target: <metric ≥ threshold>.
  Constraints: <what this requires>.
  Downstream cost: <what this commits you to>.

Contender B: <approach>.
  Metric target: <metric ≥ threshold>.
  Constraints: <what this requires>.
  Downstream cost: <what this commits you to>.
```

**Max 3 contenders.** More than that = unfocused. If you've got 5 options, split into sequential experiments. Narrow to 2-3 first, then compare the survivors.

### Domain-Specific Framing

#### LLM Agent Experiments
- Prompt strategy A vs B (system prompt variants, few-shot vs zero-shot, CoT vs direct, structured output vs free-form)
- Model version comparisons (`claude-sonnet-4-6` vs `gpt-4o-2024-08-06`, or same family different versions)
- Tool selection policy comparisons (which tools in the action space, tool ordering, forced vs optional)
- Temperature / sampling strategy comparisons
- Metrics to measure: coherence, factuality, instruction-following rate, hallucination rate, format compliance, latency p99, cost per call, tool selection accuracy

#### NLP Pipeline Experiments
- Pipeline architecture comparisons (regex→rules vs model→classifier, transformer vs classical, fine-tuned vs zero-shot)
- Embedding model comparisons (which embedder for this domain, which dimensionality)
- Feature set ablations (which features matter, which are noise)
- Metrics to measure: F1, precision, recall, entity match rate, latency, throughput, model size

#### CV Agent Experiments
- Architecture comparisons (backbone A vs B, pretrained vs fine-tuned, detection vs segmentation)
- Augmentation strategy comparisons (which transforms help, which hurt)
- Resolution / preprocessing strategy comparisons (input size vs accuracy tradeoff)
- Metrics to measure: mAP, IoU, classification accuracy, inference latency, model size, memory footprint

#### General Versioned-Artifact Experiments
- Any component where multiple approaches need metric evidence to resolve a LATS branch
- Define the metrics before running. Cherry-picking metrics after seeing results is not science, it's marketing.

⏸ **PG-3 [SOFT] — Contender framing confirmed before experiments run.** Auto-pass when: exactly 2 contenders, both framed with identical metric target + constraints shape, AND no contender requires infrastructure the project-specs stack doesn't already support. Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds.
Confirm / Adjust / Reject?

---

## Phase 2: Design the Experiment

Define how you'll actually measure. Be specific — vague experiment design produces vague conclusions.

```
### Experiment Design
- Dataset: <what data, how much, from where — link to profile if profiled>
- Metrics: <exact metrics, exact computation method — "accuracy" is not specific enough>
- Success criteria: <metric thresholds that determine the winner — decided BEFORE running>
- Isolation: <what's held constant between contenders — if nothing, your experiment is garbage>
- Cost budget: <how much compute/time/money each contender burns>
- Sample size: <how many runs/examples — enough to be statistically meaningful, not just "a few">
```

**Isolation matters.** If you change the model AND the prompt AND the temperature between Contender A and B, you've learned nothing. Change one variable. Hold everything else constant. This is 8th-grade science, not novel methodology.

⏸ **PG-4 [SOFT] — Experiment design confirmed before execution.** Auto-pass when: dataset + metrics + success criteria + isolation + cost budget + sample size all populated (no hand-wave), AND exactly one variable differs between contenders. Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds. Execution burns compute — a fail here is cheap; a fail in Phase 3 is expensive.
Confirm / Adjust / Reject?

---

## Phase 3: Run It & Read the Numbers

Execute each contender. Measure everything you said you'd measure. No cherry-picking metrics after the fact — you defined success criteria in Phase 2, now honor them.

Collect results:

| Contender | Metric 1 | Metric 2 | Metric 3 | Latency p99 | Cost | Notes |
|-----------|----------|----------|----------|-------------|------|-------|

Fill every cell. "N/A" is acceptable. Empty cells are not.

**Be honest.** If Contender A wins on accuracy but loses on latency, say so. If both contenders are within noise of each other, say that too — it means the decision should be made on other grounds (complexity, maintainability, cost). "Too close to call" is a valid experimental result. It means the cheaper/simpler option wins by default.

---

## Phase 4: Record the Decision

Experiments are ADRs. Not notebooks. Not Slack threads. Not "I remember we tried that once."

**This skill does not write the ADR file directly.** Dispatch to `/monke-design:adr` for auto-numbering and writing — that skill owns the `monke-docs/decisions/` namespace and prevents `ADR-NNN` collisions when multiple teammates write ADRs concurrently.

### 4.1 Assemble the ADR Body

Build the structured content the ADR skill will write. Keep the body in memory; don't touch disk yet:

```markdown
# ADR-NNN: <Component> — <Experiment Title>
Status: proposed
Date: YYYY-MM-DD  |  Component: HLD S-X.Y
Type: experiment

## Hypothesis
<What you tested. One sentence. Crisp.>
<Bad: "We wanted to see if model A was better." Good: "A fine-tuned classifier outperforms a prompt-chain for intent detection at <50ms p99.">

## Contenders (LATS output)
Contender A: <approach>. Metric target: <threshold>.
Contender B: <approach>. Metric target: <threshold>.

## Experiment Design
- Dataset: <what, how much, from where>
- Metrics: <what's measured, how>
- Success criteria: <what determines winner — defined before running>
- Isolation: <what was held constant>

## Results
| Contender | Metric 1 | Metric 2 | Latency p99 | Cost | Notes |
|-----------|----------|----------|-------------|------|-------|

## Threats to Validity
<What could invalidate these results. Data leakage? Overfitting to test set? Distribution shift between test and production? Small sample size? Monke's inner critic speaks here — if you can't think of threats, you're not thinking hard enough.>

## Decision — chose <X> because <metric evidence + project constraints>
## Consequences — eval thresholds set to <values>, version pin set to <version>, retraining trigger set to <condition>
```

### 4.2 Gate on the Selection

⏸ **PG-5 [SOFT] — Present the ADR body. User confirms the selection.** Auto-passes when the recommended option has clear advantage — dominates runner-up on ≥2 constraints with no trade-off loss. Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when the dominance condition holds.
Confirm / Adjust / Reject?

### 4.3 Dispatch to the ADR Skill

On confirm, dispatch to `/monke-design:adr` with:

- `type: experiment`
- `component: <HLD S3 component>`
- `slug: <experiment-slug>`
- `body: <the markdown body from 4.1, with the `ADR-NNN` placeholder left untouched — the ADR skill assigns the number>`

The ADR skill handles numbering (scans existing `monke-docs/decisions/` for the next free `NNN`), writes the file, and returns the assigned path. Do not write to `monke-docs/decisions/` directly — that is how collisions happen when experiment + agentify + design skills all run in the same wave.

The winning approach's artifact gets pinned via `/monke-seer:registry`. The losing approaches stay in the ADR as runner-ups — when the world changes, you know what to try next without starting from scratch.

---

## Gate Classifications Used Here

| Gate | Type | Notes |
|------|------|-------|
| PG-3 | SOFT | Contender framing. Auto-pass: exactly 2 contenders, identical metric-target shape, no new infrastructure required. |
| PG-4 | SOFT | Experiment design. Auto-pass: every field populated, exactly one variable differs between contenders. |
| PG-5 | SOFT | Experiment selection. Auto-pass: recommended option has clear advantage — dominates runner-up on ≥2 constraints with no trade-off loss. |

---

## Where This Connects

- **design-specs S1.4 (LATS):** Experiments ARE LATS branches — expand options, evaluate with metrics, select with evidence
- **design-specs S7 (ADR):** Experiment records ARE ADRs with metric evidence
- **design-specs S9.4 (Pause Gates):** PG-5 fires per experiment decision — user confirms the metric-backed selection
- **monke-seer/registry.md:** Winning experiment's artifact gets pinned in the registry
- **monke-seer/profile.md:** Experiment dataset profiled before the experiment runs

---

## Anti-Patterns to Refuse

| If you find yourself... | Stop. Do this instead. |
|------------------------|------------------------|
| Running experiments outside the ADR framework | An experiment without an ADR is a notebook. Record the decision or it didn't happen. |
| Creating a separate experiment tracking system | Experiments are ADRs. The tracker is `monke-docs/decisions/`. Not MLflow. Not Weights & Biases. Not a spreadsheet. You have a design system — use it. |
| Running experiments after implementation | Experiments inform design decisions (HLD/LLD). After implementation, it's eval testing. You don't experiment on production — you test it. |
| Comparing >3 contenders in one experiment | LATS expands 2-3 options. More = unfocused. Split into sequential experiments. First narrow, then compare. |
| Changing multiple variables between contenders | Isolate one variable. Hold everything else constant. Otherwise you've learned nothing and wasted compute on noise. |
| Cherry-picking metrics after seeing results | Define success criteria BEFORE running. If the metrics surprise you, that's data — don't hide it, don't spin it. |
| Skipping the profile and jumping straight to experiments | Profile first. Experimenting without understanding your data is like taste-testing with a cold — you can't tell what you're measuring. |
| Reporting "Contender A is better" without numbers | Better how? By how much? At what cost? At what latency? Show the table or it's an opinion, not an experiment. |
| Writing the ADR file directly to `monke-docs/decisions/NNN-*.md` | Refuse. The `NNN` numbering is owned by `/monke-design:adr` — writing directly causes collisions when multiple skills emit ADRs in the same wave. Dispatch to `/monke-design:adr` with the body from Phase 4.1 and let it assign the number. |

---

## Context Death Protocol

**Checkpoint artifacts (written on context pressure):**
- `monke-docs/decisions/.experiment-drafts/<component>-<slug>.md` — draft ADR body assembled in Phase 4.1, before dispatch to `/monke-design:adr`.
- Per-contender results ledger noting which contenders have completed their runs.

**Status line marker:** `Where We Are:` in `monke-status.md` reads `seer:experiment — <component> (<N>/<M> contenders run, phase <N>)` while mid-flight.

**Recovery detection (on entry):**
- If `monke-status.md` Resume block names `/monke-seer:experiment` AND draft ADR exists in `.experiment-drafts/` → resume at Phase 3 or 4 as the draft indicates.
- If draft exists but marker cleared → check if an ADR was already written for this slug; if yes, clean up the draft and exit; if no, re-present Phase 4 gate.
- If neither present → start fresh from Phase 1.

---

## Status Update

**Read on entry:** `monke-status.md` — confirm the target component has a profiled data source (Phase 0 prerequisite) and note which contenders have been run by prior invocations.

**Write on exit:**
- **Success:** bump `Updated:` with today's date + `by /monke-seer:experiment`. Note the assigned ADR path (returned by `/monke-design:adr`) in Gate Audit Log: `- PG-5 EXPERIMENT <component>-<slug> — accepted (ADR-NNN)`. If the winner has a versioned artifact, suggest `/monke-seer:registry` as next step.
- **Blocked:** if contenders couldn't be run (infrastructure, data access, budget), add a row to Open Blockers with WHAT/WHY/HOW.
- **Partial:** write a `Resume:` block naming the phase + which contenders are complete for context-death recovery.
