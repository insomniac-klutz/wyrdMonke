# WyrdMonke Design:HLD — The Grand Dreaming

> **Usage:** `/monke-design:hld [L1|L2|L3|matrix|evolve|amend "<directive>"|auto]`
>
> Architects the skeleton of the beast — L1 context, L2 containers, L3 components, boundary matrix, phase plan — one zoom level at a time, Architect sparring with Critic until the map can be built from.

---

## Arguments

`$ARGUMENTS` parsing:
- `L1` | `L2` | `L3` | `matrix` — greenfield creation or resume from specified level
- `evolve` — add, split, merge, recontract, repattern, or remove components in a complete HLD
- `amend "<directive>"` — corrective change to HLD, triggered by user or LLD escalation
- `auto` — auto-generate lightweight HLD from discovered structure (invoked by fast onboarding in `/monke`)
- `resume:<N>` (optional second positional) — skip to Phase `<N>` with checkpoint re-read (see drafter §7). Phase numbers map: 1=L1, 2=L2, 3=L3, 4=matrix, 5=finalize.
- Default (empty): mode detected from HLD state (see Mode Detection below)

```
MODE="${ARGUMENTS%%[[:space:]]*}"   # first word
REST="${ARGUMENTS#* }"              # rest (for amend directive or resume:<N>)
RESUME_PHASE="$(echo "$REST" | grep -oE '^resume:[0-9]+$' | cut -d: -f2)"
DIRECTIVE="$REST"                   # used when MODE == amend
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

Read `monke-status.md`. Then verify:

- `monke-docs/project-specs.md` has no `<<<` remaining (stack is locked)
- `CLAUDE.md` exists with project description
- `monke-docs/design-specs.md` exists (the rules you follow — this skill inlines the relevant sections, spec file remains source of truth)

If bootstrap incomplete → tell user: "Run `/monke-init` then `/monke-design:tinker` first." Stop.

If existing code detected and no HLD exists → suggest: "You have existing code. Consider `/monke` (fast onboarding auto-generates a lightweight HLD) or `/monke-recon:reconstruct` for full reverse-engineering."

---

## Mode Detection

If explicit mode given (`L1`/`L2`/`L3`/`matrix`/`evolve`/`amend`/`auto`), use it. Otherwise detect:

| HLD State | No args → |
|-----------|-----------|
| No `monke-docs/hld.md` | Greenfield — start from L1 |
| HLD exists, incomplete (missing L1/L2/L3/matrix) | "HLD in progress. Resume from `<next incomplete level>`?" |
| HLD exists, all levels confirmed | "Complete HLD detected. Evolve (grow/restructure) / Amend (corrective fix) / Start fresh?" |

If user picks **Evolve** → jump to [Evolve Flow](#evolve-flow).
If user picks **Amend** → ask for directive, jump to [Amend Flow](#amend-flow).
If user picks **Start fresh** or a resume level → continue to Agent Teams Check below.
If mode is `auto` → jump to [Auto-Generated HLD Flow](#auto-generated-hld-flow).

---

## Agent Teams Check

Follow the **Agent Teams Detection** procedure inlined in `/monke-design:tinker` (Phase 1 — the bootstrap skill owns the full block with the `.claude/settings.json` JSON snippet and `/exit` flow). HLD runs after bootstrap, so in the normal case the flag is already set and detection is a quick verification pass.

Verify:
1. `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in env or `.claude/settings.json`
2. Claude Code `>= v2.1.32`

If the flag is NOT set, defer to tinker's setup instructions rather than duplicating them here; tell the user: "Agent teams flag missing. Run `/monke-design:tinker` (or re-read its Phase 1 Agent Teams Detection) to enable, then resume HLD." This is rare post-bootstrap; treat as a regression.

After verification:

| Result | Mode |
|--------|------|
| Both present | **Team mode** — spawn Architect+Critic per the HLD Team kickoff below |
| Not available | **Solo mode** — single-session with self-critique at each level |

Tell the user which mode you're using.

---

## Inlined Design Paradigm (from design-specs S1)

These are the rules this skill follows. Spec file `monke-docs/design-specs.md` remains source of truth for maintenance — do not re-load it at runtime.

### Frameworks (S1)

- **C4 Model** for traditional request-response systems (APIs, pipelines, CRUD).
- **CoALA dimensions** (Memory, Action Space, Decision Procedure) for agentic / autonomous-loop systems.
- **Anthropic Composable Patterns** as the complexity ladder for LLM-involving components.

C4 defines outer boundaries. CoALA defines agent internals. A container is tagged `traditional` or `agentic` at L2 — this determines which framework applies at L3/L4.

**Agentic Candidacy Heuristic:** When tagging a container at L2, apply this test — if the container must (a) select among tools or strategies based on runtime context, (b) handle non-deterministic inputs where the correct action isn't known at design time, or (c) make multi-step decisions with feedback loops — it is an agentic candidate. LATS should explore both `traditional` and `agentic` at PG-5. Default: if the container consumes any versioned-artifact or data-dependent tool, it is agentic until proven otherwise.

### C4 Zoom Levels (S1.1)

| Level | Answers | Document |
|-------|---------|----------|
| L1 Context | What is this system? Who uses it? External systems? | `monke-docs/hld.md` S1 |
| L2 Container | Deployable units and communication? | `monke-docs/hld.md` S2 |
| L3 Component | Modules inside each container? Boundary contracts? | `monke-docs/hld.md` S3 |
| L4 Code | Implementation of one component. | `monke-docs/lld/component.md` |

**HLD = L1+L2+L3.** LLD = L4. Each level is a separate document created at a separate time. Collapsing levels produces plans that read well but cannot be built from.

### CoALA Dimensions (S1.2) — for agentic components

Every agentic component's HLD section MUST address all three dimensions:

- **Memory** — Working (context window, token budget), Episodic (past experiences, retrieval strategy, retention), Semantic (domain knowledge, RAG pipeline, index strategy), Procedural (prompt templates, tool definitions, code).
- **Action Space** — Internal (reasoning, retrieval, memory-writes) vs External (tool calls, API requests). Define action boundaries (what the agent CANNOT do).
- **Decision Procedure** — The control loop: observe → retrieve → reason → plan → execute → learn → loop/terminate. Specify which steps, stopping condition, max iterations, human-in-the-loop points.

**External tools are classified by contract behavior (3 subtypes):**

| Subtype | Behavior | Examples | Extra fields | Test obligation |
|---------|----------|----------|--------------|-----------------|
| **Static-contract** | Deterministic contract; schema fixed between deployments | REST APIs, databases, filesystems, fixed-schema MCP servers | (default — none) | Unit + integration |
| **Versioned-artifact** | Contract shape stable, output quality depends on artifact version | LLM model versions, embedding models, trained classifiers, CV weights, vector indices | Version pin, retraining/rebuild trigger, eval threshold | Unit + integration + **eval** |
| **Data-dependent** | Contract shape stable, output semantics drift with input distribution | Feature stores, search indices, user profile stores, document corpora | Baseline profile, drift detection threshold | Unit + integration + **eval** |

Static-contract is default. Classify by contract behavior, not ML domain or access protocol (HTTP/gRPC/MCP are orthogonal).

**CoALA output format in HLD (for each agentic component at L3):**

    ### Agent: <name>
    Pattern: <Anthropic pattern from S1.3>
    Loop: observe → retrieve → reason → execute → loop
    Memory: working(<budget>), episodic(<store>), semantic(<store>), procedural(<location>)
    Actions: internal(<strategies>), external(<tools: subtype>), boundaries(<cannot do>)
    Stops when: <condition>
    Human-in-loop: <where>
    # If any tool is versioned-artifact or data-dependent:
    Version pins: <artifact: version>
    Eval thresholds: <metric: threshold>
    Drift thresholds: <metric: threshold>   # data-dependent only

### Anthropic Composable Patterns (S1.3) — complexity ladder

Select the **simplest pattern that satisfies requirements**. Escalation requires an ADR with concrete evidence.

    Augmented LLM → Prompt Chaining → Routing → Parallelization
    → Orchestrator-Workers → Evaluator-Optimizer → Autonomous Agent

| If the task... | Start with... |
|----------------|--------------|
| Needs one LLM call ± tools | Augmented LLM |
| Has fixed sequential steps | Prompt Chaining |
| Has distinct input categories | Routing |
| Has independent parallel subtasks | Parallelization |
| Has unpredictable subtask structure | Orchestrator-Workers |
| Needs iterative refinement | Evaluator-Optimizer |
| Is fully open-ended | Autonomous Agent |

**De-escalation signal:** fewer than 3 dynamic decision points → over-engineered. De-escalate + ADR.

### LATS — Design Space Exploration (S1.4)

At each design branch:
1. **EXPAND** 2-3 options
2. **EVALUATE** against constraints
3. **SELECT** with reason
4. **PAUSE** — present options to user, wait for confirmation
5. Preserve runner-up for **BACKTRACK**

**Triggers:** L2 container decisions, L3 decomposition choices, ambiguous boundary contracts, Anthropic pattern selection, CoALA dimension decisions, user uncertainty.

    ## Design Branch: <name>
    Option A: <sentence>. Constraints: ... Downstream cost: ... Open questions: ...
    Option B: <sentence>. Constraints: ... Downstream cost: ... Open questions: ...
    Recommended: A because <reason>. Runner-up: B — revisit if <condition>.
    ⏸ WAITING FOR USER CONFIRMATION before proceeding.

Never say "both are fine." Every option set has a recommended pick with a concrete reason. Do NOT auto-commit — user must confirm or override.

### Tech Stack Rules (S2)

Locked choices declared in `project-specs.md` S10.1:

| Layer | What to lock |
|-------|--------------|
| Backend language | Primary language + version. Alternatives require ADR + PG-6 (HARD). |
| Frontend framework | UI framework + bundler. No alternatives without ADR. |
| Database | Primary persistent store. Auxiliaries permitted with ADR. |
| LLM interface | LLM client library. No direct provider SDKs without ADR. |

**Stack violation protocol (S2.4):** If a locked tech cannot satisfy a requirement: (1) stop, (2) document blocker, (3) LATS alternatives, (4) ⏸ PG-12 (TRIGGERED) — user decides, (5) ADR with status "exception." No silent swaps.

---

## Phase 1: L1 — System Context

**Resume check (per drafter §7).** If arguments contain `resume:<N>`:
1. Read the companion lock-file `monke-docs/hld-draft.md.lock`. Lock-file is authoritative — if it's inconsistent with the draft, regenerate from scratch.
2. Verify lock-file `skill:` field matches `design:hld` and `monke-docs/hld-draft.md` exists and parses.
3. If parse/lock check passes → skip to Phase `<N>` with prerequisites re-validated inline.
4. If parse/lock check fails → emit warning "resume:<N> specified but checkpoint invalid" and proceed from Phase 1 normally.
5. See Context Death Protocol below for the full recovery spec.

**Goal:** Answer: What is this system? Who uses it? What's external?

### Team Mode

Spawn the HLD Team with this kickoff prompt (inlined from design-specs S3.3):

```
Create an agent team to design the HLD for <project>.
Two teammates:

Teammate 1 — Architect:
Your job: propose system structure for <project>. Follow LATS at every
decision branch (EXPAND 2-3 options → EVALUATE → SELECT with reason →
present). Write proposals to monke-docs/hld-draft.md.
For each L1/L2/L3 level, produce a draft, then message the Critic directly
and wait for their attack before finalizing. Comply with the tech stack
rules inlined in this skill. Use the project's default language from
project-specs S10.1. For agentic containers, select Anthropic pattern
(simplest first) and specify CoALA dimensions (Memory, Action Space,
Decision Procedure).
You own: monke-docs/hld-draft.md

Teammate 2 — Critic:
Your job: attack every Architect proposal. Find failure modes, implicit
assumptions, missing edge cases, scope creep, tech stack violations.
Do NOT propose alternatives — only identify weaknesses. When Architect
messages you a proposal, respond with: (a) what breaks first, (b) what
implicit assumption exists, (c) what they're not telling you. Challenge
pattern complexity and non-default language choices. Write your attacks
to monke-docs/critic-notes.md. If you find no flaw, say so explicitly.
You own: monke-docs/critic-notes.md
```

Lead acts as Coordinator: synthesizes Architect + Critic outputs, presents at each pause gate, resolves conflicts. Lead also audits the boundary matrix after L3 (or spawns a Contracts Auditor teammate if the system has >5 cross-boundary contracts).

**Flow:**
1. Architect drafts L1 → messages Critic directly.
2. Critic attacks via peer message → Architect revises or defends.
3. Lead synthesizes → ⏸ PG-1 (HARD) presents to user.
4. User confirms → proceed to L2.

### Solo Mode

1. Read `CLAUDE.md` project description + `project-specs.md` locked stack.
2. Draft L1 per HLD Required Sections S1 below:
   - What / for whom / why (one paragraph)
   - External actors / systems
   - Scope: IN v1 vs explicitly OUT
3. Self-critique: Is every external system listed? Is scope honest?

⏸ **PG-1 [HARD] — L1 context confirmed.** Present L1 draft. Confirm / Adjust / Reject?
Always surfaces. Never auto-passes regardless of rigor.

**Write lock-file.** After PG-1 passes, update `monke-docs/hld-draft.md.lock` per drafter §7 — capture `phase: 1`, progress, timestamp.

---

## Phase 2: L2 — Container Diagram

**Goal:** Identify separately deployable units and how they communicate.

Read confirmed L1 scope. For each capability in scope:

1. Propose containers per HLD S2 Required Sections below:
   - Name, tech, backend language, one-sentence responsibility
   - Container type tag: `traditional` or `agentic`
   - Apply the Agentic Candidacy Heuristic (above). LATS both options at PG-5 if ambiguous. Do not default to `traditional` without evaluating.
2. Communication protocols between containers.
3. Deploy topology.
4. Database instances + connection strategy per `project-specs.md` S10.1.

### Non-Default Language Check (PG-6 fire point)

For each container: compare its `backend language` to the project's default from `project-specs.md` S10.1. If they differ → PG-6 fires BEFORE the container is added to the L2 draft.

⏸ **PG-6 [HARD] — Non-default language for container `<name>`.**
Always surfaces. Never auto-passes regardless of rigor. User must confirm with:
- The measurable justification (performance, ecosystem, team expertise, library-only-exists-in-X).
- An ADR entry (follow `/monke-design:adr` inline — status `accepted`, linked to this L2 decision).
- Explicit "Confirm / Adjust / Reject?" response.

On Reject → strip the container proposal, return to Phase 2 step 1 with the default language. On Adjust → user specifies alternative, re-fire PG-6. On Confirm → proceed; the ADR is part of L2 finalization.

### LATS at Each Decision Branch

At each L2 decision point (how many containers? which protocol? shared DB vs separate?):

```
## Design Branch: <name>
Option A: <sentence>. Constraints: ... Downstream cost: ...
Option B: <sentence>. Constraints: ... Downstream cost: ...
Recommended: A because <reason>. Runner-up: B — revisit if <condition>.
⏸ PG-5 [SOFT] — LATS decision branch. Waiting for user confirmation.
Auto-pass when: recommended option has clear advantage (not a close call) and runner-up is documented for backtrack.
Rigor: surfaces under `standard`/`thorough` when tradeoffs are ambiguous; auto-confirms under `light` when condition holds.
```

### Agentic Container Selection

For any container tagged `agentic`:
- Select Anthropic pattern from complexity ladder — **simplest first**.
- This is a LATS branch → PG-5 + PG-7 (both SOFT, auto-pass when the simplest viable pattern is selected and no escalation is needed).

⏸ **PG-2 [SOFT] — L2 containers confirmed.** Present L2 draft. Confirm / Adjust / Reject?
Auto-pass when: all containers have tech + responsibility + type tag assigned AND no open LATS branch is left unresolved AND PG-6 has fired and resolved for every non-default language container.
Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds.

**Write lock-file.** After PG-2 passes, update `monke-docs/hld-draft.md.lock` per drafter §7 — capture `phase: 2`, progress, timestamp.

---

## Phase 3: L3 — Component Map

**Goal:** Identify modules inside each container with typed boundary contracts.

Per container from L2:

1. Decompose into components per HLD S3 Required Sections:
   - Module name + responsibility.
   - Boundary contracts (exact typed models — define the types now).
   - Dependency direction (consumer → provider).
   - LLD owner assignment.
2. For `agentic` containers — CoALA summary using the format above.
3. **Tool-type LATS:** For each external capability this component needs (API call, model inference, LLM call, data query, MCP tool, etc.) — LATS whether it should be a tool in the agent's action space vs hardcoded logic. If the capability's output is non-deterministic, version-dependent, or context-dependent, it is a tool. Classify per the 3-subtype taxonomy above. Skip for trivially deterministic operations.
4. LATS at decomposition branches (PG-5 per branch).

⏸ **PG-3 [SOFT] — L3 components confirmed.** Present L3 component map. Confirm / Adjust / Reject?
Auto-pass when: all boundaries typed AND no orphan components (every component has at least one upstream or downstream).
Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds.

**Write lock-file.** After PG-3 passes, update `monke-docs/hld-draft.md.lock` per drafter §7 — capture `phase: 3`, progress, timestamp.

---

## Phase 4: Boundary Matrix & Phase Plan

### Boundary Matrix

Build from L3 contracts:

| Upstream | Contract | Downstream | Error Type | Serialization | Stability | Status |
|----------|----------|------------|------------|---------------|-----------|--------|

Status is `planned` in greenfield. Stability values: `static` (default — omit for standard contracts) | `versioned(<artifact>, <pin>)` | `data-dependent(<baseline>)`.

### Lead Audits Boundary Matrix

Verify: every component has at least one upstream or downstream. No orphans. Contract types consistent across boundaries.

If system has >5 cross-boundary contracts and Agent Teams available → consider spawning a Contracts Auditor teammate.

⏸ **PG-4 [SOFT] — Boundary matrix audited.** Present boundary matrix. Confirm no gaps?
Auto-pass when: matrix complete AND no orphans AND contracts consistent across boundaries.
Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds.

**Write lock-file.** After PG-4 passes, update `monke-docs/hld-draft.md.lock` per drafter §7 — capture `phase: 4`, progress, timestamp.

### Phase Plan

Group components into implementation phases by dependency order:
- Phase 1: components with no upstream dependencies (Layer 0 types, foundational services).
- Phase 2: components depending on Phase 1 outputs.
- Continue until all components assigned.

```
| Phase | Components | Dependencies | Test Gate Status |
|-------|-----------|-------------|-----------------|
```

Test gate status: `pending` for all in greenfield.

For components with versioned-artifact or data-dependent tools: note eval obligations in the phase plan. Eval tests run within IL-2 and IL-3 — no separate phase or gate.

### Data Flows

Define max 3 primary data flows through the system:
```
Trigger -> Module (contract: ModelA) -> Module (contract: ModelB) -> Terminal
```

For agentic flows:
```
Input -> Agent (observe) -> [retrieve] -> [reason] -> [tool] -> [learn] -> Output/Loop
```

---

## Phase 5: Quality Gate & Finalize

### HLD Quality Checks (inlined from design-specs S5.3)

| Check | Test |
|-------|------|
| Implementable | Can a dev build without unrecorded decisions? |
| Bounded | Every module has a typed boundary contract? |
| Navigable | Find any component's LLD in 30 seconds? |
| Honest | Unknowns in `open-questions.md`, not papered over? |
| Minimal | Every sentence: constraint, decision, contract, or question? |
| Audited | Boundary matrix verified? |
| Attacked | Critic reviewed, challenges recorded in ADR? |
| Stack-compliant | All containers use locked tech per S2? |
| Language-justified | Non-default language choices backed by ADR with measurable justification? |
| Pattern-justified | Agentic components have pattern ADR? Simplest that works? |
| CoALA-complete | Agentic components specify all three dimensions? |
| Phased | Components assigned to phases with system test criteria? |

Any failing check → create OQ via inline `/monke-design:oq` logic.

### Banned From HLD (S5.2)

Function bodies, algorithms, exhaustive schemas, features beyond v1, internal mechanics prose, dependency versions, CI setup, decorative language, agent internals.

### Assemble Final HLD

Write `monke-docs/hld.md` with all confirmed sections. Add header:
```
Version: 1.0 | Date: <today> | Source: greenfield design
```

### Decision Index

Link all ADRs created during LATS branches. If none → "No ADRs yet."

### Open Questions

Link to `monke-docs/open-questions.md`. Create any OQs surfaced during design.

### Team Cleanup (if Agent Teams used)

Before shutting down team:
- Copy Critic attacks → ADR "Challenges Considered" sections
- Clean up working files (`hld-draft.md`, `critic-notes.md`)
- **Delete lock-file.** `monke-docs/hld-draft.md.lock` is no longer needed after finalization; stale lock-files confuse future recovery.

### Archive critic-notes

After LLD teams have copied Critic attacks into ADR "Challenges Considered" sections (or acknowledged no ADRs were warranted), archive `monke-docs/critic-notes.md`:

- If `monke-docs/decisions/` exists: `mv monke-docs/critic-notes.md monke-docs/decisions/critic-notes-archive.md`
- If not: create the archive file via content-copy (don't use `mv` in a way that fails). Append a dated header line at the top of the archive: `Archived: <ISO-date> — from HLD <greenfield|evolve> run`.

Archive prevents `critic-notes.md` from confusing future HLD/LLD recovery (the file is one-shot per HLD run).

### Suggest Next Steps

- "Resolve blocking OQs: `/monke-design:oq triage`"
- "Create LLDs for Phase 1: `/monke-design:lld <first-component>`"
- "Run `/monke-status:status` to see updated dashboard."

---

## Auto-Generated HLD Flow

> Entry: `/monke-design:hld auto` — invoked by `/monke` fast onboarding when a project has existing code but no HLD.

**Goal:** Produce a lightweight HLD from discovered structure without running the full team-based design pipeline. Documentation emerges from work; this HLD is the starting map.

### A1: Collect Discovery Inputs

Fast onboarding (in `/monke`) provides these discovered inputs. If they aren't in context, detect them here:

- **Containers:** from entry points (Dockerfiles, workspace roots, `package.json` workspaces, Cargo workspace members, `pyproject.toml` projects, `go.mod` modules, `.sln`/`.csproj` projects).
- **Components:** from barrel files (`index.ts`, `mod.rs`, `__init__.py`), public exports, top-level directories.
- **Boundaries:** from typed function signatures crossing module edges, HTTP route definitions, RPC schemas, message bus publishers/subscribers.

### A2: Assign Confidence Flags

Each discovered container and each boundary gets a confidence flag:

| Signal | Confidence |
|--------|-----------|
| Typed exports (TS interface, Python dataclass/pydantic, Rust pub struct, Go exported type) | **HIGH** |
| Explicit route/RPC/schema definition | **HIGH** |
| Barrel file with named exports | **HIGH** |
| Bare directory with no exports | **LOW** |
| Inferred from import graph only | **LOW** |
| Contract shape guessed from usage patterns | **LOW** |

Write each boundary matrix row with a `Confidence: HIGH | LOW` column. HIGH boundaries can feed downstream work. LOW boundaries need human review before any LLD is generated from them.

### A3: Draft Lightweight HLD Sections

Fill the Required Sections with discovered data:

- **S1 Context:** 1-2 sentence stub from `README.md` + top-level directory names. Mark "**Auto-generated — refine during first LLD pass.**"
- **S2 Containers:** one row per discovered container. Tech + language from detected manifests. Type tag = `traditional` by default unless an LLM/agent framework is detected (`anthropic`, `openai`, `langchain`, `llamaindex`) → then `agentic` with `Confidence: LOW`.
- **S3 Components:** one entry per discovered component inside each container. Boundary contracts inlined where types are found, `TBD` where not.
- **S7 Boundary Matrix:** rows per detected boundary with Confidence column. Set Status = `existing` (not `planned`) since code already exists.
- **S8 Phase Plan:** derive from import-graph topology. Components with no incoming dependencies go in Phase 1.

### A4: Present With Confidence Summary

Present to the user (triggered by `/monke` fast-onboarding hard gate, not a separate pause here):

```
Auto-generated HLD summary:
  Containers:   <N> (HIGH: M, LOW: K)
  Components:   <N> (HIGH: M, LOW: K)
  Boundaries:   <N> (HIGH: M, LOW: K)

LOW-confidence areas that need first-LLD review:
  - <container/component/boundary>: <why LOW>
  ...
```

### A5: Write HLD

Write `monke-docs/hld.md` with header:
```
Version: 0.1 | Date: <today> | Source: auto-generated from fast onboarding
```

Version 0.1 (not 1.0) signals incomplete. It becomes 1.0 after the first full LLD pass confirms the discovered structure.

### A6: Return Control

This flow does NOT fire PG-1/2/3/4 individually. The parent `/monke` fast-onboarding gate handles overall confirmation. If the user rejects at that gate, fall back to the full greenfield flow above.

---

## Evolve Flow

> Entry: `/monke-design:hld evolve` or selected from Mode Detection.

### E0: Evolve What?

Read current HLD + `monke-status.md`. Present current architecture summary (containers, components, phase progress). Then ask:

```
⏸ **PG-1 [HARD] — Scope evolution — What are you evolving?**

  1. Add         — new component or container
  2. Split       — decompose an existing component into multiple
  3. Merge       — combine existing components
  4. Recontract  — change a boundary contract between components
  5. Repattern   — escalate/de-escalate an agentic pattern
  6. Remove      — cut a component from scope

Which? And which component(s)?
```

Always surfaces. This is the scope-redefinition form of PG-1 — evolving the HLD changes the project's scope boundary, so PG-1's scope-confirmation contract re-applies. User must specify both the evolution type and the target before any reads or diffs begin.

Store: `EVOLVE_TYPE` and `EVOLVE_TARGET`.

---

### E1: Scope the Evolution

Based on `EVOLVE_TYPE`:

| Type | Read | Determine |
|------|------|-----------|
| Add | Existing L1 scope, L2 containers, L3 map | Does L1 scope need expansion? New container or module in existing? |
| Split | Target component's HLD entry + LLD (if exists) | What are the new sub-components? Which boundaries split? |
| Merge | Both target components' HLD entries + LLDs | Which component absorbs? Which contracts collapse? |
| Recontract | Target boundary row in S7 + both sides' LLDs | What's the new contract? Why is the old one wrong? |
| Repattern | Target agentic component's CoALA summary | Which pattern? Escalation or de-escalation? ADR required. |
| Remove | Target component's HLD entry + dependents | What absorbs its responsibilities? Any orphaned boundaries? |

If L1 scope changes (Add, Remove) → draft L1 delta.

⏸ **PG-1 [HARD] — L1 scope delta re-confirm.** Present the L1 delta only (not the full L1). Confirm / Adjust / Reject?
Always surfaces.

---

### E2: Place in Architecture

Work through the affected HLD levels. Only touch what changes — do not rebuild confirmed levels.

**For Add:**
- New container? → LATS at L2 (placement, protocol, deploy). PG-5 per branch (SOFT).
- New module in existing container? → skip L2, go to L3.
- Define component at L3: name, responsibility, boundary contracts against existing neighbors. LATS at decomposition branches. PG-5 per branch.
- For agentic: CoALA summary + Anthropic pattern selection (PG-7 SOFT).

**For Split:**
- Remove old component from L3. Add new sub-components.
- Reassign boundary contracts — existing neighbors now point to one of the new sub-components.
- LATS at "which sub-component owns which boundary." PG-5.

**For Merge:**
- Remove absorbed component from L3. Expand surviving component's responsibility.
- Collapse boundary contracts between the two. Update neighbor contracts.
- If container-level merge → update L2.

**For Recontract:**
- Update the specific S7 row and both components' L3 entries.
- LATS if multiple valid contract shapes. PG-5.

**For Repattern:**
- Update CoALA summary in L3.
- ADR mandatory (escalation or de-escalation). PG-7.

**For Remove:**
- Remove from L3 (and L2 if removing a container).
- Reassign or delete boundary contracts. Flag orphaned downstream.
- Update L1 scope OUT.

⏸ **PG-2 / PG-3 [SOFT] — L2/L3 delta confirmed.** Present the delta (only the affected levels). Confirm / Adjust / Reject?
Auto-pass when: delta preserves all untouched boundaries AND PG-6 has fired for any non-default language in new containers AND no orphan components introduced.
Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds.

---

### E3: Define Contracts Against Existing Neighbors

For each existing component that gains, loses, or changes a boundary with the evolution target:

```
Existing component: <name>
  LLD status: waiting | ready | implemented
  Current boundaries: [list from S7]
  Change: new upstream/downstream | contract modified | boundary removed
  New contract: <typed model>
  Impact on LLD: <what changes in their LLD>
  Impact on code: <what changes if already implemented>
```

LATS at any ambiguous contract shape. PG-5.

⏸ **PG-4 [SOFT] — Updated boundary matrix audited.** Present updated boundary matrix. Confirm no gaps?
Auto-pass when: every evolution-affected row re-validated AND no dangling contracts on removed/split components.
Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds.

---

### E4: Impact Manifest

Before writing anything to `monke-docs/hld.md`, present the full blast radius:

```
⏸ PG-14 (TRIGGERED): HLD Evolution Impact Manifest

HLD changes:
  - L1 scope: expanded / narrowed / unchanged
  - L2 containers: added / removed / unchanged
  - L3 components: [list changes]
  - S4 data flows: new / modified / unchanged
  - S7 boundary matrix: [N rows added / M modified / K removed]
  - S8 phase plan: [component slotted into Phase <N> / re-phased]

Downstream cascade:
  - LLDs needing creation: [list]
  - LLDs needing boundary update: [list with specific contract changes]
  - LLDs to delete: [list, if merge/remove]
  - Implementation needing update: [list, if code exists]
  - Tests needing update: [integration tests for affected boundaries]
  - Checkpoints affected: [any passed phases invalidated?]

ADRs to write: [list decisions made during LATS branches]

Confirm / Adjust / Reject?
```

**User MUST confirm before any writes.** If rejected → discard, return to E0. If adjusted → revise and re-present.

---

### E5: Execute & Finalize

1. Write HLD amendments to `monke-docs/hld.md`. Version bump:
   ```
   Version: X.Y+1 | Date: <today> | Change: <evolution type> — <summary>
   ```
2. Write ADRs for decisions made during LATS branches.
3. Update `monke-docs/open-questions.md` with any OQs surfaced.
4. Run the HLD Quality Checks (above) on the modified sections.
5. Mark affected LLDs as "boundary stale" in `monke-status.md` if their contracts changed.

### Suggest Next Steps (Evolve)

Based on evolution type:
- **Add:** "Create LLD for new component: `/monke-design:lld <component>`"
- **Split:** "Create LLDs for new sub-components. Update existing neighbor LLDs."
- **Merge:** "Update surviving component's LLD. Delete absorbed LLD."
- **Recontract:** "Update LLDs on both sides of the changed boundary."
- **Repattern:** "Rewrite component's LLD with new pattern: `/monke-design:lld <component>`"
- **Remove:** "Delete component's LLD. Update neighbor LLDs."
- Always: "Run `/monke-status:status` to see updated dashboard."

---

## Amend Flow

> Entry: `/monke-design:hld amend "<directive>"`, from user or LLD escalation.

### Amend Sources

**From LLD skill (`/monke-design:lld` Phase 5):**
During LLD creation, Designer or Reviewer discovers HLD boundary contract is wrong, missing, or inconsistent. The LLD skill cannot silently fix it — it escalates with a directive describing what's wrong and a proposed fix.

**From user directly:**
User provides directive via `/monke-design:hld amend "<directive>"`. Could be any corrective change.

**In both cases:** A directive is required. No directive → ask for one before proceeding.

---

### A1: Parse Directive & Locate

1. Read the directive. Extract:
   - What HLD section(s) are affected (S1-S8)?
   - What's wrong with the current state?
   - What's the proposed fix?
2. Read current `monke-docs/hld.md`. Locate the affected sections.
3. Read `monke-status.md` to understand current build state (which LLDs exist, what's implemented).

If directive is vague → ask user to clarify before proceeding.

---

### A2: Impact Trace

Trace the amendment's blast radius:

- **Boundary matrix (S7):** Which rows change?
- **LLDs:** Which reference the affected boundary/component? What specifically changes in each?
- **Implementation:** Any code already built against the old contract?
- **Tests:** Integration tests asserting on the old contract?
- **Phase plan (S8):** Does dependency order change?

---

### A3: Present Amendment

```
⏸ PG-14 (TRIGGERED): HLD Amendment
Source: LLD/<component> | user
Directive: <directive>

Proposed HLD changes:
  - <section>: <diff description>
  - S7 row <N>: <diff description>

Downstream cascade:
  - LLDs needing update: [list with specific changes]
  - Implementation needing update: [list, if code exists]
  - Tests needing update: [list]

Confirm / Adjust / Reject?
```

**User MUST confirm.** If rejected → return to caller (LLD skill or user) with "amendment rejected."

---

### A4: Apply

1. Write HLD changes to `monke-docs/hld.md`. Version bump:
   ```
   Version: X.Y+1 | Date: <today> | Change: amend — <summary from directive>
   ```
2. Write ADR if the amendment represents a non-trivial decision change.
3. Mark affected LLDs as "boundary stale" in `monke-status.md`.

**Amend does NOT execute the downstream cascade.** It updates the HLD and reports what's affected. The LLD skill resumes with the corrected boundary. Other affected LLDs are flagged for update.

### Return to Caller

- **If invoked from LLD skill:** Return control. LLD skill re-reads the updated HLD boundary and continues from where it paused.
- **If invoked by user:** Suggest next steps:
  - "Update stale LLDs: `/monke-design:lld <component>` for each flagged component."
  - "Run `/monke-status:status` to see affected components."

---

## Gate Classifications Used Here

| Gate | Type | Notes |
|------|------|-------|
| PG-1 | HARD | L1 Context confirmation. Never auto-passes. |
| PG-2 | SOFT | L2 Containers. Auto-pass: all containers tagged, no open LATS. |
| PG-3 | SOFT | L3 Components. Auto-pass: all boundaries typed, no orphans. |
| PG-4 | SOFT | Boundary matrix. Auto-pass: matrix complete, no orphans, contracts consistent. |
| PG-5 | SOFT | LATS decision. Auto-pass: recommended option has clear advantage. |
| PG-6 | HARD | Non-default language exception. Always surfaces. |
| PG-7 | SOFT | Anthropic pattern. Auto-pass: simplest viable pattern, no escalation needed. |
| PG-12 | TRIGGERED | Stack violation. Fires only when a locked-tech swap is attempted. |
| PG-14 | TRIGGERED | HLD amendment. Fires only when LLD / evolve / amend edits the HLD. |

Rigor (`light` / `standard` / `thorough`) in `.monke-config.md` controls SOFT behavior:
- `light`: all SOFTs auto-confirm if their conditions are met, log as `[gate:PG-N] auto-confirmed`.
- `standard`: SOFTs fire only if auto-pass conditions fail or tradeoffs look ambiguous.
- `thorough`: all SOFTs fire regardless; user sees every decision.

HARD and TRIGGERED always respect their triggers regardless of rigor.

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Collapse L1 + L2 into a single "architecture doc" | Refuse. Each level is a separate document — collapsing produces plans that read well but cannot be built from. |
| Draft L3 components without a confirmed L2 | Refuse. L3 is decomposition of L2 containers — skipping backward invalidates dependency direction and boundary typing. |
| Default a container to `traditional` without applying the Agentic Candidacy Heuristic | Refuse. Run the three-part test (tool selection / non-deterministic input / multi-step feedback) and LATS both paths if ambiguous. |
| Amend HLD silently from inside the LLD skill | Refuse. Any boundary correction fires PG-14 via `/monke-design:hld amend` — the human confirms the cascade. |
| Skip PG-6 for a non-default language because "it's just one service" | Refuse. PG-6 is HARD — every container carrying a language exception gets an ADR and user confirmation. |
| Escalate Anthropic pattern to Orchestrator-Workers without LATS de-escalation check | Refuse. Complexity ladder says simplest-first — escalations require ADR with concrete evidence of <3 dynamic decisions being insufficient. |

---

## Context Death Protocol

**Checkpoint artifacts:** `monke-docs/hld-draft.md` (Architect's WIP), `monke-docs/hld-draft.md.lock` (phase + progress ledger per drafter §7), `monke-docs/critic-notes.md` (Critic's attacks), partial sections in `monke-docs/hld.md` when finalization began.
**Status line marker:** `Where We Are: design:hld — L<N> <phase-name>` (greenfield) or `Where We Are: design:hld — evolve on <target>` (evolve) while mid-flight.
**Recovery detection:** On re-entry, if `monke-docs/hld-draft.md` exists AND status marker names this skill → read the lock-file (authoritative — if it's inconsistent with the draft, regenerate from scratch) and resume from the phase it records. If critic-notes.md exists but hld.md has no corresponding section → re-enter at the ⏸ for that level. If mode detection finds a complete HLD but a draft file exists → ask user "Resume prior session or discard draft?" before touching `hld.md`.

---

## Status Update

**Read on entry:** `monke-status.md` — verify bootstrap complete, detect mid-flight HLD resume state, check for existing LLDs whose boundaries may be affected by an evolve/amend.

**Write on exit**, update `monke-status.md`:

**Greenfield:**
- Mark completed HLD level checkboxes (`[x] L1 Context — PG-1 confirmed — <date>`).
- Populate LLDs table with all components from S3 (status: "waiting").
- Populate Phase Checkpoints table from S8.
- Populate Decisions table with any ADRs created.
- Add any blocking OQs to Open Blockers.
- Set "Where We Are" and "Next action."
- Bump `Updated:` line.

**Auto-generated:**
- Populate LLDs table with discovered components (status: "discovered", maturity from fast-onboarding scan).
- Boundary matrix rows include Confidence flags.
- Note in "Where We Are": `HLD v0.1 auto-generated — refine during first LLD pass`.

**Evolve:**
- Add new components to LLDs table (status: "waiting").
- Mark removed components as "removed" (do not delete row — preserve history).
- Mark affected existing LLDs as "boundary stale" if their contracts changed.
- Update Phase Checkpoints table if phases changed.
- Add ADRs to Decisions table.
- Note evolution in "Where We Are": `HLD evolved (vX.Y) — <type>: <summary>`.

**Amend:**
- Mark affected LLDs as "boundary stale".
- Note amendment in "Where We Are": `HLD amended (vX.Y) — <summary>`.
- Bump `Updated:` line, `by /monke-design:hld amend`.
