# WyrdMonke Recon:Roadmap — The slow crawl to production.

> **Usage:** Copy `monke-recon/` to `.claude/commands/monke-recon/`. Invoke: `/monke-recon:roadmap`

---

## Arguments

`$ARGUMENTS` parsing:
- No arguments. The roadmap synthesizes everything recon has produced.
- Example: `/monke-recon:roadmap`

```
# No arguments — full synthesis
```

---

## Prerequisites

Read `monke-status.md` to verify current state. Then check ALL of these:

- **`monke-docs/recon/recon-survey.md` exists** — the inventory. If missing → "Run `/monke-recon:survey` first." Stop.
- **`monke-docs/hld.md` exists** — the architecture. If missing → "Run `/monke-recon:reconstruct` first." Stop.
- **`monke-docs/recon/recon-gaps.md` exists** — the gap analysis. If missing → "Run `/monke-recon:gaps` first." Stop.
- **`monke-docs/open-questions.md` exists** — the open questions. If missing → "Run `/monke-recon:oqs` first." Stop.

Optional:
- `monke-docs/flash/flash-manifest.md` — for flash context on shortcuts and deferred items.
- `monke-docs/lld/*.md` — for component-level maturity tags.

The roadmap is the final recon skill. It needs everything else to exist.

---

## Phase 1: Input Synthesis

Read all recon artifacts and build the work inventory.

### 1.1 Load Everything

1. Read `monke-docs/recon/recon-survey.md` — what exists
2. Read `monke-docs/hld.md` — how it's structured
3. Read `monke-docs/recon/recon-gaps.md` — what's missing
4. Read `monke-docs/open-questions.md` — what's undecided
5. If flash: read `monke-docs/flash/flash-manifest.md` — what shortcuts were taken and what was deferred

### 1.2 Categorize All Work

Every gap, every unresolved OQ, and every `needs-work`/`stub` maturity tag from LLDs becomes a work item. Categorize into:

| Category | Description | Source |
|----------|-------------|--------|
| **Blockers** | Must resolve before ANY production traffic. | Critical gaps + blocking OQs |
| **Foundation** | Structural work that other fixes depend on. | Error types, test infrastructure, CI/CD setup |
| **Hardening** | Making existing code production-worthy. | Error handling, validation, logging, per component |
| **Expansion** | Features cut from MVP that are needed for production. | Flash manifest deferred items, `stub` maturity tags |
| **Polish** | Quality improvements, performance, documentation. | Medium/low gaps, non-blocking OQs |

**Rules:**
- A work item in Blockers must genuinely block production. "We should add metrics" is not a blocker unless you can't operate without them.
- Foundation items are things that, if done first, make all subsequent work easier. Test infrastructure is the classic example.
- Expansion is only for flash-mode codebases or codebases with explicit stubs. Don't invent features.
- Polish is everything else. It matters, but it ships after the other categories.

---

## Phase 2: Phase Planning

Group work into phases. Each phase has a clear entry gate, work items, and exit gate.

### Phase R1: Unblock

**What:** Resolve critical blockers and blocking OQs. Nothing else moves until these are cleared.

**Work items:**
- All critical-severity gaps from `recon-gaps.md`
- All OQs with `Blocks: production` or `Blocks: <component>`
- Security gaps that create immediate risk

**Exit gate:** All critical gaps closed. All blocking OQs resolved (confirmed, answered, or explicitly accepted-as-is with rationale).

**How:** OQ resolution via `/monke-design:oq triage`. Security fixes may need direct implementation.

---

### Phase R2: Foundation

**What:** Build the infrastructure that all subsequent hardening depends on.

**Work items:**
- Define error types at all boundaries (typed error variants, not strings)
- Set up test infrastructure (fixtures, factories, mock utilities, CI pipeline)
- Set up linting and type checking (if not already strict)
- Define shared types / Layer 0 models (if missing or incomplete)
- Set up logging infrastructure (structured logging, log levels)

**Exit gate:** IL-0 equivalent for all components — types compile, imports resolve, test runner works.

**How:** `/monke-implement:implement <component> 0` for each component's type skeleton. Test infrastructure may need its own focused implementation pass.

---

### Phase R3: Harden

**What:** Make existing code production-worthy, component by component.

**Work items per component:**
- Add error handling (replace bare catches, add fallbacks, add input validation)
- Add unit tests for existing functionality
- Fix code smells (break up god functions, reduce nesting, extract pure logic from IO)
- Add logging at key decision points
- For agentic: add stopping conditions, token budgets, action boundaries

**Exit gate:** Unit tests passing per component. Linter clean. Error handling at all boundaries.

**How:** `/monke-implement:implement <component> 2` (Layer 2 — bodies + unit tests). Each component goes through the implementation pipeline.

---

### Phase R4: Integrate

**What:** Verify cross-component contracts and add integration tests.

**Work items:**
- Write integration tests for every boundary in the HLD matrix
- Verify serialization contracts (especially cross-language boundaries)
- Add contract tests for external API dependencies
- For agentic: verify memory consistency, tool contracts, agent-to-agent communication
- Upgrade `implicit` boundaries in the HLD matrix to `verified`

**Exit gate:** IL-3 equivalent — all integration tests passing, full suite green, coverage threshold met.

**How:** `/monke-test:test-run` for integration test execution. `/monke-implement:implement <component> 3` for writing integration tests.

---

### Phase R5: Ship

**What:** System tests, documentation, deployment configuration. The final mile.

**Work items:**
- Write system/e2e tests for each HLD S4 data flow
- Write or update README, API docs, environment setup guide
- Configure deployment (Dockerfile, compose, CI/CD pipeline, env vars)
- Add health checks, metrics endpoints (if applicable)
- Final security review pass
- Resolve remaining medium/low gaps and polish OQs

**Exit gate:** PG-11 checkpoint — all system tests passing, documentation complete, deployment tested.

**How:** `/monke-implement:checkpoint <phase>` for the final sign-off. `/monke-test:coverage` for coverage verification.

---

## Phase 3: Component Priority

Within each phase, order components by:

### Priority Ranking

1. **Dependency order.** Foundations first. If component B imports from component A, A goes first.
2. **Risk.** Highest blast-radius gaps first. A gap in the auth middleware affects every endpoint. A gap in a utility function affects one.
3. **Effort.** Within the same priority band, quick wins before heavy lifts. Shipping a fixed auth check (small) before rewriting the data layer (xl) keeps momentum.

**Build the priority table:**

```markdown
### Component Priority
| Phase | Order | Component | Why First | Effort | Key Gaps |
|-------|-------|-----------|-----------|--------|----------|
| R2 | 1 | shared-types | Foundational — everyone imports | small | G-3 (no error types) |
| R2 | 2 | auth | Security risk + dependency | medium | G-1, G-5 |
| R3 | 1 | api-handler | Most boundaries | large | G-7, G-9, G-12 |
| ... | | | | | |
```

---

## Phase 4: Effort Estimation

Per component per phase. These are rough estimates, not commitments. Honesty over precision.

| Size | Meaning |
|------|---------|
| `small` | A focused session. Hours, not days. One or two files. |
| `medium` | A solid day of work. Multiple files, some complexity. |
| `large` | Multiple days. Touches several components or requires careful refactoring. |
| `xl` | A week or more. Significant architectural work, possibly requiring design iteration. |

**Totals per phase:**
```markdown
### Effort Summary
| Phase | Small | Medium | Large | XL | Estimated Total |
|-------|-------|--------|-------|----|-----------------|
| R1 | 2 | 1 | 0 | 0 | ~1 day |
| R2 | 3 | 2 | 1 | 0 | ~3 days |
| R3 | 1 | 4 | 2 | 0 | ~5 days |
| R4 | 0 | 3 | 1 | 0 | ~3 days |
| R5 | 2 | 2 | 1 | 1 | ~5 days |
```

**NOTE:** These are vibes-calibrated estimates. The actual time depends on codebase complexity, how many OQs need design discussion, and how many gaps turn out to be deeper than they look. Expect R3 to take longer than estimated — hardening always does.

---

## Output

Write `monke-docs/recon/recon-roadmap.md` with the full production crawl plan.

**Structure:**
```markdown
# Recon Roadmap — Production Crawl Plan
Project: <name> | Date: <today>

## Work Inventory
| Category | Count | Example |
|----------|-------|---------|
| Blockers | N | <top blocker> |
| Foundation | N | <top foundation item> |
| Hardening | N | <top hardening item> |
| Expansion | N | <top expansion item> |
| Polish | N | <top polish item> |

## Phase R1: Unblock
<work items, exit gate>

## Phase R2: Foundation
<work items, exit gate, pipeline connection>

## Phase R3: Harden
<work items per component, exit gate, pipeline connection>

## Phase R4: Integrate
<work items, exit gate, pipeline connection>

## Phase R5: Ship
<work items, exit gate, pipeline connection>

## Component Priority
<priority table>

## Effort Summary
<effort table per phase>

## Next Steps
<actionable first moves>
```

**Aim for 150-180 lines.** Tables over prose. Actionable over aspirational.

---

## Decision Gate

⏸ **Present the full roadmap to user.**

Walk through:
1. Work inventory summary — "here's everything that needs doing"
2. Phase plan — "here's the order"
3. Component priority — "here's what to hit first"
4. Effort estimates — "here's how long it might take"

User can:
- **Adjust priorities** — "auth is more urgent than types for us"
- **Cut scope** — "skip phase R5 polish, we'll do that post-launch"
- **Add constraints** — "we need to ship R1-R3 in 2 weeks"
- **Challenge estimates** — "that's way more than XL, that's a rewrite"

Update `recon-roadmap.md` with adjustments before finalizing.

---

## Next Steps

After the roadmap is confirmed, suggest the first concrete actions:

1. **"Start Phase R1: resolve blockers via `/monke-design:oq triage`"** — if there are blocking OQs
2. **"Once blockers clear, begin Phase R2: `/monke-implement:implement <first-component> 0`"** — first foundation component
3. **"Run `/monke-status:status` to see the full dashboard with recon phases."** — the dashboard now shows the complete picture

**The roadmap connects recon back to the existing monke pipeline.** From here, it's the standard loop:
- Design questions → `/monke-design:oq`
- Implementation → `/monke-implement:implement`
- Testing → `/monke-test:test-run`
- Checkpoints → `/monke-implement:checkpoint`

Recon's job is done. The jungle has been mapped. Now you build the paths.

---

## Key Behaviors

- **The roadmap connects recon to the existing monke pipeline.** Every phase maps to existing skills. No new workflow needed — just a different starting point.
- **Each phase has a clear gate that maps to existing monke gates.** R2 → IL-0, R3 → IL-2 (unit tests), R4 → IL-3 (integration), R5 → PG-11.
- **Don't over-plan.** The roadmap is a guide, not a contract. Phase R1 is precise. Phase R5 is directional. That's fine.
- **Be honest about effort.** "This is a large effort" beats fake precision. If the effort is uncertain, say "large-to-xl depending on what we find during R3."
- **Quick wins matter for morale.** Front-load small items in each phase. Shipping fixes builds momentum.
- **The roadmap is a living document.** It will change as OQs are resolved and gaps are fixed. That's expected.

---

## Status Update

On completion, update `monke-status.md`:
- **Recon section:** Mark `[x] Roadmap — <date>`. All recon items should now be checked.
- **Phase Checkpoints table:** Populate with R1-R5 phases:
  | Phase | Components | All IL-3? | System Tests | PG-11 | Status |
  |-------|-----------|-----------|--------------|-------|--------|
  | R1: Unblock | <blockers> | N/A | N/A | — | pending |
  | R2: Foundation | <components> | — | — | — | pending |
  | R3: Harden | <components> | — | — | — | pending |
  | R4: Integrate | <components> | — | — | — | pending |
  | R5: Ship | all | — | — | — | pending |
- Set `Updated:` to today, `by /monke-recon:roadmap`
- Update "Where We Are": `Phase: **Recon complete — production crawl planned**`
- Update "Next action": first action from the roadmap (usually `/monke-design:oq triage` for R1 blockers, or `/monke-implement:implement <component> 0` if no blockers)
