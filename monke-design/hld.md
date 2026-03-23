# WyrdMonke Design:HLD — High-Level Design (Create / Evolve / Amend)

> **Usage:** Copy `monke-design/` to `~/.claude/commands/monke-design/`. Invoke: `/monke-design:hld [L1|L2|L3|matrix|evolve|amend "<directive>"]`

---

## Arguments

`$ARGUMENTS` parsing:
- `L1` | `L2` | `L3` | `matrix` — greenfield creation or resume from specified level
- `evolve` — add, split, merge, recontract, repattern, or remove components in a complete HLD
- `amend "<directive>"` — corrective change to HLD, triggered by user or LLD escalation
- Default (empty): mode detected from HLD state (see Mode Detection below)

```
MODE="${ARGUMENTS%%[[:space:]]*}"   # first word
DIRECTIVE="${ARGUMENTS#* }"         # rest (for amend)
```

---

## Prerequisites

Read `monke-status.md`. Then verify:

- `monke-docs/project-specs.md` has no `<<<` remaining (stack is locked)
- `CLAUDE.md` exists with project description
- `monke-docs/design-specs.md` exists (the rules you follow)

If bootstrap incomplete → tell user: "Run `/monke-init` then `/monke-design:tinker` first." Stop.

If existing code detected and no HLD exists → suggest: "You have existing code. Consider `/monke-design:recon` to reverse-engineer an HLD instead."

---

## Mode Detection

If explicit mode given (`L1`/`L2`/`L3`/`matrix`/`evolve`/`amend`), use it. Otherwise detect:

| HLD State | No args → |
|-----------|-----------|
| No `monke-docs/hld.md` | Greenfield — start from L1 |
| HLD exists, incomplete (missing L1/L2/L3/matrix) | "HLD in progress. Resume from `<next incomplete level>`?" |
| HLD exists, all levels confirmed | "Complete HLD detected. Evolve (grow/restructure) / Amend (corrective fix) / Start fresh?" |

If user picks **Evolve** → jump to [Evolve Flow](#evolve-flow).
If user picks **Amend** → ask for directive, jump to [Amend Flow](#amend-flow).
If user picks **Start fresh** or a resume level → continue to Agent Teams Check below.

---

## Agent Teams Check

Check if Agent Teams are available:
1. `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in env or `.claude/settings.json`
2. Claude Code `>= v2.1.32`

Optional: `tmux` recommended for split-pane visibility, not required.

If the flag is not set, tell the user to merge this into their `.claude/settings.json`:
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```
Then use `/exit` and resume the thread for it to take effect.

**⏸ Wait for user response before proceeding.**

After user accepts (and restarts) or declines, re-check `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` in the environment — the `/exit` breaks context so you must verify the current state:

| Result | Mode |
|--------|------|
| Both present | **Team mode** — spawn Architect+Critic per design-specs S3.3 |
| Not available | **Solo mode** — single-session with self-critique at each level |

Tell the user which mode you're using.

---

## Phase 1: L1 — System Context

**Goal:** Answer: What is this system? Who uses it? What's external?

### Team Mode

Spawn team per `design-specs.md` S3.3:

```
Create an agent team to design the HLD for <project>.
Two teammates:

Teammate 1 — Architect:
<spawn prompt from design-specs S3.3, populated with project context from CLAUDE.md and project-specs>

Teammate 2 — Critic:
<spawn prompt from design-specs S3.3>
```

Architect drafts L1. Critic attacks. Lead synthesizes.

### Solo Mode

1. Read `CLAUDE.md` project description + `project-specs.md` locked stack.
2. Draft L1 per `design-specs.md` S5.1 S1:
   - What / for whom / why (one paragraph)
   - External actors / systems
   - Scope: IN v1 vs explicitly OUT
3. Self-critique: Is every external system listed? Is scope honest?

**⏸ PG-1: Present L1 draft. Confirm / Adjust / Reject?**

---

## Phase 2: L2 — Container Diagram

**Goal:** Identify separately deployable units and how they communicate.

Read confirmed L1 scope. For each capability in scope:

1. Propose containers per `design-specs.md` S5.1 S2:
   - Name, tech, backend language, one-sentence responsibility
   - Container type tag: `traditional` or `agentic`
2. Communication protocols between containers
3. Deploy topology
4. Database instances + connection strategy per `project-specs.md` S10.1

### LATS at Each Decision Branch

At each L2 decision point (how many containers? which protocol? shared DB vs separate?):

```
## Design Branch: <name>
Option A: <sentence>. Constraints: ... Downstream cost: ...
Option B: <sentence>. Constraints: ... Downstream cost: ...
Recommended: A because <reason>. Runner-up: B — revisit if <condition>.
⏸ PG-5: WAITING FOR USER CONFIRMATION
```

Per `design-specs.md` S1.4: expand 2-3 options, evaluate, select with reason, present. Never say "both are fine."

### Agentic Container Selection

For any container tagged `agentic`:
- Select Anthropic pattern from complexity ladder (`design-specs.md` S1.3) — **simplest first**
- This is a LATS branch → PG-5 + PG-7

**⏸ PG-2: Present L2 draft. Confirm / Adjust / Reject?**

---

## Phase 3: L3 — Component Map

**Goal:** Identify modules inside each container with typed boundary contracts.

Per container from L2:

1. Decompose into components per `design-specs.md` S5.1 S3:
   - Module name + responsibility
   - Boundary contracts (exact typed models — define the types now)
   - Dependency direction (consumer → provider)
   - LLD owner assignment
2. For `agentic` containers — CoALA summary per `design-specs.md` S1.2:
   ```
   ### Agent: <name>
   Pattern: <Anthropic pattern>
   Loop: observe → retrieve → reason → execute → loop
   Memory: working(<budget>), episodic(<store>), semantic(<store>), procedural(<location>)
   Actions: internal(<strategies>), external(<tools>), boundaries(<cannot do>)
   Stops when: <condition>
   Human-in-loop: <where>
   ```
3. LATS at decomposition branches (PG-5 per branch)

**⏸ PG-3: Present L3 component map. Confirm / Adjust / Reject?**

---

## Phase 4: Boundary Matrix & Phase Plan

### Boundary Matrix (S7)

Build from L3 contracts:

| Upstream | Contract | Downstream | Error Type | Serialization | Status |
|----------|----------|------------|------------|---------------|--------|

Status is `planned` for all (no implementation yet in greenfield).

### Lead Audits Boundary Matrix

Verify: every component has at least one upstream or downstream. No orphans. Contract types are consistent across boundaries.

If system has >5 cross-boundary contracts and Agent Teams available → consider spawning Contracts Auditor teammate.

**⏸ PG-4: Present boundary matrix. Confirm no gaps?**

### Phase Plan (S8)

Group components into implementation phases by dependency order:
- Phase 1: components with no upstream dependencies (Layer 0 types, foundational services)
- Phase 2: components depending on Phase 1 outputs
- Continue until all components assigned

```
| Phase | Components | Dependencies | Test Gate Status |
|-------|-----------|-------------|-----------------|
```

Test gate status: `pending` for all in greenfield.

### Data Flows (S4)

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

Run quality gate from `design-specs.md` S5.3. Any failing check → create OQ via inline `/monke-design:oq` logic (or suggest user runs it).

### Assemble Final HLD

Write `monke-docs/hld.md` with all confirmed sections. Add header:
```
Version: 1.0 | Date: <today> | Source: greenfield design
```

### Decision Index (S5)

Link all ADRs created during LATS branches. If none → "No ADRs yet."

### Open Questions (S6)

Link to `monke-docs/open-questions.md`. Create any OQs surfaced during design.

### Team Cleanup (if Agent Teams used)

Before shutting down team:
- Copy Critic attacks → ADR "Challenges Considered" sections
- Clean up working files (`hld-draft.md`, `critic-notes.md`)

### Suggest Next Steps

- "Resolve blocking OQs: `/monke-design:oq triage`"
- "Create LLDs for Phase 1: `/monke-design:lld <first-component>`"
- "Run `/monke-status:status` to see updated dashboard."

---

## Evolve Flow

> Entry: `/monke-design:hld evolve` or selected from Mode Detection.

### E0: Evolve What?

Read current HLD + `monke-status.md`. Present current architecture summary (containers, components, phase progress). Then ask:

```
⏸ "What are you evolving?"

  1. Add         — new component or container
  2. Split       — decompose an existing component into multiple
  3. Merge       — combine existing components
  4. Recontract  — change a boundary contract between components
  5. Repattern   — escalate/de-escalate an agentic pattern
  6. Remove      — cut a component from scope

Which? And which component(s)?
```

**⏸ Wait for user response.** User must specify both the evolution type and the target.

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

If L1 scope changes (Add, Remove) → draft L1 delta. **⏸ PG-1 re-confirm on the delta only.**

---

### E2: Place in Architecture

Work through the affected HLD levels. Only touch what changes — do not rebuild confirmed levels.

**For Add:**
- New container? → LATS at L2 (placement, protocol, deploy). PG-5 per branch.
- New module in existing container? → skip L2, go to L3.
- Define component at L3: name, responsibility, boundary contracts against existing neighbors. LATS at decomposition branches. PG-5 per branch.
- For agentic: CoALA summary + Anthropic pattern selection (PG-7).

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

**⏸ PG-2/PG-3 as applicable: Present the L2/L3 delta. Confirm / Adjust / Reject?**

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

**⏸ PG-4: Present updated boundary matrix. Confirm no gaps?**

---

### E4: Impact Manifest

Before writing anything to `monke-docs/hld.md`, present the full blast radius:

```
⏸ PG-14: HLD Evolution Impact Manifest

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
4. Run quality gate from `design-specs.md` S5.3 on the modified sections.
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

**From LLD skill (design-specs S6.2, `/monke-design:lld` Phase 5):**
During LLD creation, Designer or Reviewer discovers HLD boundary contract is wrong, missing, or inconsistent. The LLD skill cannot silently fix it — it escalates with a directive describing what's wrong and a proposed fix.

**From user directly:**
User provides directive via `/monke-design:hld amend "<directive>"`. Could be any corrective change — realization, external requirement shift, stakeholder feedback.

**In both cases:** A directive is required. No directive → ask for one before proceeding.

---

### A1: Parse Directive & Locate

1. Read the directive. Extract:
   - What HLD section(s) are affected (S1-S8)?
   - What's wrong with the current state?
   - What's the proposed fix?
2. Read current `monke-docs/hld.md`. Locate the affected sections.
3. Read `monke-status.md` to understand current build state (which LLDs exist, what's implemented).

If directive is vague → ask user to clarify before proceeding. "Which boundary? Which component? What's the concrete mismatch?"

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
⏸ PG-14: HLD Amendment
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

**User MUST confirm.** If rejected → return to caller (LLD skill or user) with "amendment rejected." LLD skill must find another path or escalate as OQ.

---

### A4: Apply

1. Write HLD changes to `monke-docs/hld.md`. Version bump:
   ```
   Version: X.Y+1 | Date: <today> | Change: amend — <summary from directive>
   ```
2. Write ADR if the amendment represents a non-trivial decision change.
3. Mark affected LLDs as "boundary stale" in `monke-status.md`.

**Amend does NOT execute the downstream cascade.** It updates the HLD and reports what's affected. The LLD skill resumes with the corrected boundary. Other affected LLDs are flagged for update — the user decides when to address them.

### Return to Caller

- **If invoked from LLD skill:** Return control. LLD skill re-reads the updated HLD boundary and continues from where it paused.
- **If invoked by user:** Suggest next steps:
  - "Update stale LLDs: `/monke-design:lld <component>` for each flagged component."
  - "Run `/monke-status:status` to see affected components."

---

## Status Update

On completion, update `monke-status.md`:

**Greenfield:**
- Mark completed HLD level checkboxes (`[x] L1 Context — PG-1 confirmed — <date>`)
- Populate LLDs table with all components from S3 (status: "waiting")
- Populate Phase Checkpoints table from S8
- Populate Decisions table with any ADRs created
- Add any blocking OQs to Open Blockers
- Set "Where We Are" and "Next action"
- Bump `Updated:` line

**Evolve:**
- Add new components to LLDs table (status: "waiting")
- Mark removed components as "removed" (do not delete row — preserve history)
- Mark affected existing LLDs as "boundary stale" if their contracts changed
- Update Phase Checkpoints table if phases changed
- Add ADRs to Decisions table
- Note evolution in "Where We Are": `HLD evolved (vX.Y) — <type>: <summary>`
- Bump `Updated:` line, `by /monke-design:hld evolve`

**Amend:**
- Mark affected LLDs as "boundary stale"
- Note amendment in "Where We Are": `HLD amended (vX.Y) — <summary>`
- Bump `Updated:` line, `by /monke-design:hld amend`
