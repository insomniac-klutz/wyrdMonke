# WyrdMonke Seer:Orchestra — The Bone Reader's Autopilot

> **Usage:** `/monke-seer:orchestra`

---

## Purpose

Reads the dashboard, sniffs for components that need data profiling, experiments that need running, or artifacts that need pinning. Points at the right ritual and waits. One command to navigate the seer workflow.

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

Read `monke-status.md`. Then read `monke-docs/hld.md` (if it exists) to understand what components are in play.

---

## Decision Tree

Walk this tree top-to-bottom. Recommend the FIRST applicable action. One recommendation per invocation.

```
START
  │
  ├─ Is there a component about to start L3 (or LLD) that uses
  │  versioned-artifact or data-dependent tools, AND no profile exists
  │  for its data sources?
  │  YES → recommend `/monke-seer:profile <data-source>`
  │         "This component consumes data that hasn't been profiled.
  │          Profile first — distributions become Layer 0 types and eval baselines."
  │
  ├─ Is there a LATS decision branch (PG-5/PG-7) with multiple viable
  │  model/approach options that need metric evidence to resolve?
  │  YES → recommend `/monke-seer:experiment <component> <hypothesis>`
  │         "Multiple approaches on the table. Run an experiment —
  │          LATS with metrics, not vibes."
  │
  ├─ Is there a component with a versioned-artifact tool that has no
  │  version pin registered (no entry in LLD header or Layer 0 types)?
  │  YES → recommend `/monke-seer:registry <component>`
  │         "This artifact is unpinned. Register it —
  │          version pin, eval threshold, retraining trigger."
  │
  ├─ Is there a component tagged `traditional` in HLD S3 that exhibits
  │  agentic candidacy signals (design-specs S1.2) but hasn't been assessed?
  │  OR: Is there an agentic component whose pattern might be wrong-sized?
  │  YES → recommend `/monke-seer:agentify <component>`
  │         "This component might be smarter than it looks — or dumber
  │          than it's pretending to be. Assess before you build."
  │
  └─ None of the above?
     → "The bones are quiet. No seer work needed right now.
        When a component needs data, models, or experiments — monke will know."
```

---

## After Recommendation

Present the recommendation with context:

```
⏸ Seer recommends: `/monke-seer:<skill> <target>`
Reason: <why this is the next seer action>
Context: <what component/data source/artifact is involved>

Run it? (yes / skip / explain more)
```

If user says "skip" → move to next applicable action in the tree.
If user says "explain more" → describe what the skill will do and what it produces.

---

## Where This Connects

- **monke-design/orchestra.md:** Design orchestra may hand off to seer when it detects DS components
- **monke-status.md:** Seer orchestra reads the dashboard to find what needs profiling/experimenting/registering
- **design-specs S1.2:** Tool taxonomy determines which components trigger seer recommendations
- **monke-seer/profile.md:** Data profiling — the most common first seer action
- **monke-seer/experiment.md:** Experiment tracking — triggered by LATS branches with model options
- **monke-seer/registry.md:** Artifact registration — triggered by unpinned versioned-artifacts
- **monke-seer/agentify.md:** Intelligence assessment — triggered by traditional components with agentic signals or agentic components with pattern mismatch
