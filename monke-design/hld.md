# WyrdMonke Design:HLD — Greenfield High-Level Design

> **Usage:** Copy `monke-design/` to `~/.claude/commands/monke-design/`. Invoke: `/monke-design:hld [resume]`

---

## Arguments

`$ARGUMENTS` parsing:
- Single positional: resume point — `L1` | `L2` | `L3` | `matrix`
- Default (empty): start fresh from L1
- If resume arg given → read `monke-docs/hld.md`, verify prior levels have content, resume from specified level
- If `monke-docs/hld.md` has content but no resume arg → ask user: "HLD exists. Resume from where, or start fresh?"

```
RESUME="${ARGUMENTS:-L1}"
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

## Agent Teams Check

Check if Agent Teams are available:
1. `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in env or `.claude/settings.json`
2. Claude Code `>= v2.1.32`
3. `tmux` available

| Result | Mode |
|--------|------|
| All present | **Team mode** — spawn Architect+Critic per design-specs S3.3 |
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

## Status Update

On completion, update `monke-status.md`:
- Mark completed HLD level checkboxes (`[x] L1 Context — PG-1 confirmed — <date>`)
- Populate LLDs table with all components from S3 (status: "waiting")
- Populate Phase Checkpoints table from S8
- Populate Decisions table with any ADRs created
- Add any blocking OQs to Open Blockers
- Set "Where We Are" and "Next action"
- Bump `Updated:` line
