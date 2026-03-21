# WyrdMonke Design:Recon — Reverse-Engineer HLD & Surface Open Questions

> **Usage:** Copy `monke-design/` to `.claude/commands/monke-design/` (project-local). Then run `/monke-design:recon [scope]` inside a project with WyrdMonke bootstrapped.

---

## Arguments

`$ARGUMENTS` parsing:
- Single positional: scope limiter (subdirectory, component name, or glob)
- Default (empty): analyze entire project
- Example: `/monke-design:recon src/api`

```
SCOPE="${ARGUMENTS:-}"
```

---

## Prerequisites

Read `monke-status.md` to verify current state. Then check:

- `monke-docs/design-specs.md` exists
- `monke-docs/project-specs.md` exists and has no `<<<` remaining
- `CLAUDE.md` or `monke-CLAUDE.md` exists

If any are missing → tell user: "Run `/monke-init` then `/monke-design:tinker` first." Stop.

---

## Phase 1: Codebase Reconnaissance

Gather raw facts. Do NOT design yet — just catalog what exists.

### 1.1 Stack Detection

Read `monke-docs/project-specs.md` S2 (stack bindings) and S10 (locked stack). Verify against actual code:

| Check | How |
|-------|-----|
| Languages present | File extensions, manifest files |
| Frameworks in use | Import statements, config files |
| Database | Migration files, ORM config, connection strings |
| LLM usage | LLM client imports, API key references |
| External APIs | HTTP client calls, SDK imports |

Flag any **drift** between declared stack and actual code.

### 1.2 Container Discovery (L2 candidates)

Identify separately deployable units:

| Signal | Likely container |
|--------|-----------------|
| Separate manifest (`package.json`, `Cargo.toml`, `pyproject.toml`) | Independent service/package |
| `Dockerfile` or `docker-compose` service | Deployable container |
| Separate `main` / entry point | Distinct runtime |
| `/api`, `/web`, `/worker`, `/cli` dirs | Container per concern |
| Monorepo workspace members | One container per workspace |

For each: name, tech, entry point, likely responsibility.

### 1.3 Component Discovery (L3 candidates)

Within each container, identify modules per `design-specs.md` S1.1 (C4 L3):

| Signal | Likely component |
|--------|-----------------|
| Directory with `__init__.py` / `mod.rs` / `index.ts` | Module boundary |
| Exported types/interfaces | Boundary contract |
| Route handlers / controllers | API surface component |
| Service classes / use-case modules | Business logic component |
| Repository / data-access layers | Persistence component |
| Agent / LLM orchestration code | Agentic component (tag for CoALA) |
| Shared types / models dir | Cross-cutting types (Layer 0) |

For each: name, responsibility, exports, imports, `traditional` or `agentic` tag.

### 1.4 Data Flow Discovery

Trace the top 3 most common paths through the codebase. Follow entry points through to terminal actions. Note contracts passed at each hop. For agentic flows: identify decision loop, tools, memory access.

### 1.5 Existing Test Inventory

Catalog what tests exist: files, types (unit/integration/system), coverage config, fixtures, mock patterns.

**⏸ Present reconnaissance summary. Confirm containers, components, and flows match user's understanding.**

---

## Phase 2: Generate HLD

Write `monke-docs/hld.md` following `design-specs.md` S5.1 structure. Every section below is required — reference S5.1 for the exact format.

| Section | Content | Source |
|---------|---------|--------|
| **S1 System Context (L1)** | What/whom/why, external actors, scope IN/OUT | README, manifest, CLAUDE.md |
| **S2 Container Diagram (L2)** | Per container from 1.2: name, tech, language, responsibility, type tag. Protocols, topology, DB strategy. | Phase 1.2 |
| **S3 Component Map (L3)** | Per container: modules, boundary contracts, dependency direction, LLD owner. CoALA summary for agentic. | Phase 1.3 |
| **S4 Primary Data Flows** | Max 3, from Phase 1.4 | Phase 1.4 |
| **S5 Decision Index** | Existing ADRs or "No ADRs yet" | `decisions/` scan |
| **S6 Open Questions** | Link to `open-questions.md` | Phase 3 |
| **S7 Boundary Matrix** | Upstream, contract, downstream, error, serialization, status (verified/untested/implicit) | Phase 1.3 contracts |
| **S8 Phase Plan** | Components grouped by dependency order. Test gate status. | Dependency analysis |

Add version header:
```
Version: 1.0 | Date: <today> | Source: reverse-engineered from existing codebase
```

**⏸ Present the complete HLD draft. Walk through each section. Confirm or adjust before writing.**

---

## Phase 3: Surface Open Questions

Analyze gaps across three dimensions. Write to `monke-docs/open-questions.md` using format from `design-specs.md` S8:

```
### OQ-NNN: Question
Discovered-during: design | implementation | testing
Affects: HLD S-X.Y  |  Blocks: LLD for <component> | implementation of <component> | testing of <boundary>
Options so far: ...
Status: open | resolved -> ADR-NNN
```

### 3.1 Design Questions

Scan for: stack drift, implicit boundaries (no typed contract), missing error handling at boundaries, circular dependencies, god modules (>5 responsibilities), agentic components without stopping conditions or action boundaries, missing CoALA dimensions, pattern over-engineering, undeclared external dependencies.

### 3.2 Implementation Questions

Scan for: business logic in IO functions, mutable boundary models, bare exceptions/panics, no pure/IO separation, dead code/unused exports, TODO/FIXME/HACK comments, hardcoded config, missing Layer 0 types.

### 3.3 Testing Questions

Scan for: no tests at all, unit-only (no integration), mocked DB in integration tests, both sides mocked in integration, no coverage config, shared mutable test state, missing test tiers, untested boundaries, tests asserting on implementation detail.

**⏸ Present all OQs grouped by dimension. Ask user to confirm, mark N/A, prioritize blockers.**

---

## Phase 4: Quality Gate

Run every check from `design-specs.md` S5.3:

| Check | Test |
|-------|------|
| Implementable | Can a dev build without unrecorded decisions? |
| Bounded | Every module has a typed boundary contract? |
| Navigable | Find any component's LLD in 30 seconds? |
| Honest | Unknowns in open-questions.md, not papered over? |
| Minimal | Every sentence: constraint, decision, contract, or question? |
| Audited | Boundary matrix verified? |
| Attacked | Critic reviewed, challenges recorded? |
| Stack-compliant | All containers use locked tech per S2? |
| Language-justified | Non-default language choices backed by ADR? |
| Pattern-justified | Agentic components have pattern ADR? Simplest? |
| CoALA-complete | Agentic components specify all three dimensions? |
| Phased | Components assigned to phases with test criteria? |

Failing check → add OQ if not already raised.

### Agent Teams Review (Optional)

If Agent Teams available: suggest Architect+Critic team to review the reverse-engineered HLD adversarially. Ask user.

If not available: self-critique based on quality gate results.

**⏸ Present quality gate results.**

---

## Phase 5: Finalize

1. Write confirmed HLD to `monke-docs/hld.md`.
2. Write confirmed OQs to `monke-docs/open-questions.md`.
3. Update `monke-mermaid.mmd` if new artifacts created.
4. Verify no HLD section is empty — each has content or "TBD — see OQ-NNN".

5. Show summary:
   - Containers / components / data flows / boundary contracts / OQs / test coverage / quality gate results

6. Suggest next steps:
   - "Resolve blocking OQs: `/monke-design:oq triage`"
   - "Create LLDs for Phase 1 components: `/monke-design:lld <component>`"
   - If Agent Teams: "Spawn Designer+Reviewer team per design-specs S3.4."

---

## Status Update

On completion, update `monke-status.md`:
- Mark all HLD checkboxes as `[x]` with dates
- Populate LLDs table with components from HLD S3 (all status: "waiting")
- Populate Phase Checkpoints table from HLD S8
- Add any blocking OQs to Open Blockers table
- Set `Updated:` to today, `by /monke-design:recon`
- Set "Where We Are" / "Next action" based on whether blockers exist
