# monke-log.md

> *every banana has a story. here's the tree they fell from.*

---

## 2.1 — 2026-04-18 — monke opens the front door

**Theme:** One leader, many workers. The user types a wish in plain English and the framework figures out the rest — classifier and scoper team name the kind of work, a single HARD gate approves the pipeline, and `monke-intake` stays as permanent lead through every step. No more "which skill do I run?" cognitive tax. Natural language becomes the entry point; the sub-skill zoo becomes a backstage workforce.

### Added
- **`monke-intake.md`** — the front door skill at repo root. Accepts natural-language requests (`/monke-intake "<wish>"` or `/monke "<wish>"` shortcut). Phases: read project state → spawn classifier+scoper worker-pool (2 teammates, parallel) → synthesize pipeline proposal → PG-1 HARD approval → walk pipeline as permanent lead → close feature thread. Writes feature thread file to `monke-docs/intake/<thread-id>.md` with lock-file for context-death recovery. Full drafter §1-8 compliance (canonical status line, `resume:<N>` convention, CDP).
- **`monke.md` dispatch** — natural-language shortcut: `/monke` ARG1 that is quoted OR exceeds 3 words routes to `/monke-intake` with the string as payload. Added `intake` as an explicit override in the case table.

### Changed
- **`README.md`** — Sacred Tree adds `monke-intake.md` line under `monke.md`. Skill-count bumped 36 → 37. Quick Start rewritten with **The Front Door** as the primary entry (`/monke "<wish>"`), 5 worked examples, 6-step walkthrough, quotation rules, and 8-row command-shortcuts table. Flash Way + Already-got-code sections updated to lead with the natural-language shortcut.
- **`monke-mermaid.mmd`** — `INTAKE` node added to root subgraph. Edge from `MONKE` to `INTAKE` for quoted-request dispatch. Outbound edges from `INTAKE` to every downstream skill it dispatches (flash/design/implement/test/rage/recon/ops) matching the per-class pipeline table.
- **`monke-CLAUDE.md`** — Core commands table gains `/monke-intake` row.
- **`monke-docs/status-template.md`** — `## Features` section added (10-column schema: Thread ID, Request, Class, Rigor, Started, Status, Progress, Last step, Last outcome, Finished) so `/monke-intake` has a canonical row template to write to. Sole-writer rule: only intake writes to Features.

### Pre-flight fixes (post-audit hardening)
- **`monke-intake.md` Prerequisites** — greenfield cold-start auto-chains `/monke-init` (invisible bootstrap step) when `.monke-config.md` is missing AND `$REQUEST` is set. Rigor inferred from request keywords: `quick` / `mvp` / `prototype` → light, `production` / `compliance` / `enterprise` → thorough, else standard.
- **`monke-intake.md` Phase 1** — Step 1.0 (cold-start bootstrap dispatch) + Step 1.5 (ensure `monke-docs/intake/` directory exists) added. Original reading list renumbered to Step 1.1.
- **`monke-intake.md` Phase 4** — `## Features` section is created from template if missing before the row is written.
- **`monke-intake.md` Phase 2** — `ops:commit` removed from all default pipeline chains (commit is never auto-dispatched; it's a Phase 6 closing suggestion only). Bare `test` disambiguated to `test:test-plan` (design-time) or `test:test-run` (post-implement).
- **`monke-intake.md` Phase 5** — Multi-LLD dispatch spec: when feature-on-existing has N new components, intake spawns a worker-pool per design-specs S3.1 (one worker per component), relays PG-9 gates in dependency order, single-LLD fallback, boundary-shift guard.
- **`monke.md`** — Resume-block parser extended to extract both `Phase:` AND `Skill:` fields. When Skill names a specific sub-skill (e.g. `/monke-intake`), dispatch jumps directly to that skill with `resume:<N>` appended — no Phase 1 re-run, no intake-thread loss on context death.
- **`monke.md`** — unquoted-token else-branch validates against `monke-status.md` Components table before treating as container scope. Unknown short token → WHAT/WHY/HOW hint asking if the user meant a quoted request. No more silent misroute.
- **`monke-rage/buggy.md`** — `resume:<N>` added as second positional (per drafter §7) with explicit phase-resume branching. Previously treated the arg as scope path.
- **`monke-flash/snap.md`** — status line marker explicitly declared in CDP (`Where We Are: flash:snap — drafting manifest (section <N>)` mid-flight; Phase-6 close marker documented). Cites drafter §8 as canonical.

### Philosophy shift
- **One leader, many workers.** Intake is the permanent lead for a feature thread. `monke.md` continues as general orchestrator for resume / status / direct skill overrides, but once a natural-language request dispatches to intake, intake owns the thread end-to-end. No cascade between orchestrators; sub-skills run as workers.
- **Zero onboarding required.** The user no longer needs to know the framework's shape to use it. Type a wish; monke names the pipeline; one approval starts everything. Missing config? Intake bootstraps invisibly.

---

## 2.0 — 2026-04-14 — monke unifies

**Theme:** Seven orchestras collapse into one all-seeing eye. Correctness becomes invisible infrastructure — gates auto-confirm silently, hard gates surface uncompromisingly, rigor tunes everything in between. The framework learns to onboard an existing codebase in one hard gate, auto-generate stub HLDs and LLDs, and route per-component instead of per-phase. Progressive formalization replaces all-or-nothing ceremony.

### Added
- **`monke.md`** — the unified orchestrator at repo root: fast onboarding (auto-detect stack → auto-map structure → per-component maturity scan → one hard gate PG-1), per-component routing tree, rigor-aware gate handling, Context Death Protocol, recovery via status + artifact verification, override dispatch for any skill
- **`monke-docs/design-specs.md`** — S9.4 gate classification (AUTO / SOFT / HARD / TRIGGERED), S12 rigor system (light / standard / thorough), S3.1 Team Decision Heuristic + 4 archetypes (worker pool, review pair, multi-role, critic loop), S3.2 team failure modes, S3.7 team re-use rules, S6.3 LLD Confidence ladder (auto-generated → reviewed → approved), S7 namespace disambiguation note
- **`monke-docs/sdlc-specs.md`** — §0 Unified Orchestrator section describing `/monke` as the front door
- **`monke-docs/project-specs.md`** — per-container tables in S8 (stack bindings) and S9 (test bindings) so polyglot projects can bind per container
- **`monke-docs/status-template.md`** + **`monke-status/status.md`** — per-component maturity table with Confidence column (auto-generated / reviewed / approved) and Override column, Resume block for Context Death Protocol, rigor display, next-action line
- **`monke-drafter.md`** — Context Death Protocol section, gate classification rules, WHAT/WHY/HOW error template, Progressive Formalization doctrine
- **`monke-design/hld.md`** — Auto-Generated HLD Flow (H9) with mixed-confidence header, per-section confidence flags, boundary matrix confidence tags (HIGH / LOW)
- **`monke-design/lld.md`** — Confidence header (auto-generated / reviewed / approved), stub LLD algorithm (B1) with minimum fields: File Map, Signatures, Maturity Tags, Decomposition Tree, Unit Test Plan skeleton, Integration Test Plan skeleton
- **`monke-implement/implement.md`** — `LAYER="auto"` support with stub LLD prerequisite check, H3 critic-notes read for LLD team dispatch
- **`monke-CLAUDE.md`** + **`CLAUDE.md`** — references design-specs S3.1 Team Decision Heuristic and S3.7 team re-use

### Changed
- **All 11 major skills** — specs inlined, 11 taglines fixed from definitions to metaphors (H4), gate classifications applied, no remaining "follow design-specs.md S..." cross-refs. Self-contained.
- **`monke-init.md`** — collapsed to 1 gate, B6 exempt (init doesn't need Agent Teams Gate since it's bootstrapping them)
- **`monke-sync.md`** — B6 exempt from Agent Teams Gate (sync bootstraps it too)
- **`monke-CLAUDE.md`** + **`CLAUDE.md`** — stripped inline team heuristics, now reference design-specs S3.1 (B7 fixed)
- **`monke-drafter.md`** — gate rules tightened, Context Death Protocol section added (H1 fixed)
- **`monke-docs/project-specs.md`** — S8 and S9 rewritten as per-container tables (B4 fixed, supports polyglot)
- **`monke-docs/status-template.md`** + **`monke-status/status.md`** — per-component maturity table with Confidence + Override columns (H8, H10 fixed), rigor display, resume action
- **`README.md`** — Sacred Tree updated (removed 7 orchestras, added `monke.md`), Skills table rewritten (added `/monke` top-level row, removed orchestra rows), Quick Start leads with `/monke`, Full Arc diagram updated to show unified orchestrator as front door
- **`monke-mermaid.mmd`** — removed 7 orchestra nodes, added single `MONKE` node with fast-onboarding subflow (scan → status → hard gate → dispatch) and edges from MONKE to every skill

### Deleted
- **`monke-design/orchestra.md`**, **`monke-implement/orchestra.md`**, **`monke-test/orchestra.md`**, **`monke-flash/orchestra.md`**, **`monke-recon/orchestra.md`**, **`monke-rage/orchestra.md`**, **`monke-seer/orchestra.md`** — replaced by the unified `monke.md` orchestrator. Seven fury menus become one all-seeing eye.

### Philosophy shift
- **Correctness as invisible infrastructure** — AUTO gates pass silently, surface only on failure. Rigor tunes which SOFT gates auto-pass. Hard gates (PG-1, PG-6, PG-11) remain unconditionally human.
- **Progressive formalization** — fast onboarding produces `auto-generated` confidence artifacts. Review upgrades to `reviewed`. Full design upgrades to `approved`. Components advance independently.
- **Adaptive rigor** — `light` / `standard` / `thorough` per design-specs S12. Auto-escalation triggers (>15 components, compliance keywords, versioned-artifact tools, polyglot) recommend rigor at PG-1; user always decides.
- **Per-component routing** — the orchestrator recommends one next action per loop, not a roadmap. Worker-pool parallelism offered when independent components share maturity.

### Complexity budget
- 1 new root file (monke.md), 7 files deleted (orchestras), 4 new doctrine sections in design-specs, 1 gate classification system, 1 rigor system, 4 team archetypes, 0 new IL gates, 0 new PG gates

---

## 0.9 — 2026-04-14 — monke bridges the gap

**Theme:** Recon meets implementation. Every skill learns to read maturity tags, route recon-origin LLDs, and die gracefully when context runs out. Agent teams get the TeamCreate/teammate rewrite. The terminology table drops so everyone speaks the same monke.

### Changed
- **`CLAUDE.md`** + **`monke-CLAUDE.md`** — agent teams rewrite: TeamCreate/teammate model with tool chain, named codenames, mandatory `team_name` rule
- **`monke-docs/sdlc-specs.md`** — +§5 recon-to-implementation bridge flow, +§8 system terminology table (7 canonical terms), section renumbering
- **`monke-recon/reconstruct.md`** — maturity tag synthesis (flash-mode cross-ref from manifest), file map example with Maturity column
- **`monke-recon/roadmap.md`** — +HLD S8 sync after roadmap confirmation, bridge-to-implementation next steps with test plan generation
- **`monke-recon/orchestra.md`** — +context death protocol: write status, tell user where you stopped
- **`monke-design/lld.md`** — recon-origin LLD review mode (preserve reconstruction, add test plan), seer artifact conflict check with user escalation
- **`monke-design/orchestra.md`** — rage-sourced OQ routing: drift→HLD amend, haunt critical→OQ resolve, other blockers→triage
- **`monke-implement/implement.md`** — recon-origin awareness: maturity tag layer overrides (as-is/needs-work/stub), recon-gaps cross-reference
- **`monke-implement/orchestra.md`** — reconstructed LLD routing: test plan check before implementation, recon-origin layer behavior note
- **`monke-rage/orchestra.md`** — +downstream routing section: auto-promote rules per mode, dedup against existing OQs, OQ template
- **`monke-test/orchestra.md`** + **`monke-test/test-plan.md`** — recon-origin awareness: reconstructed LLDs count for PG-10
- **`monke-status/status.md`** — +reconstructed-no-test-plan waterfall check (step 5b), updated line format spec, entry/exit rules
- **`monke-flash/orchestra.md`** — context death protocol: status write + checkpoint + resume instruction

---

## 0.8 — 2026-03-28 — monke raises the flag

**Theme:** Commit skill learns to fling PRs. Init and sync get smarter about what's a skill and what's a settings file. The birth certificate lands. The Sacred Tree widens its roots.

### Added
- **`monke-claude-settings.json`** — settings template: agent teams env var, ready for `.claude/settings.json` merge
- **`monke-docs/monke-readsme.md`** — the birth certificate: origin links, repo pitch, sacred references

### Changed
- **`monke-ops/commit.md`** — +Phase 6 Raise the Flag: PR/MR creation with platform detection, single/chain flows, hop-by-hop confirmation gates
- **`monke-init.md`** — root command auto-discovery (filters by `> **Usage:**`), settings merge into `.claude/settings.json`, monke-readsme scaffolding
- **`monke-sync.md`** — root command discovery filter, settings deep-merge, protected file guard assertions, monke-status.md reconciliation on template drift
- **`CLAUDE.md`** — Sacred Tree invariant broadened: root dir changes now trigger tree sync
- **`README.md`** — Sacred Tree: +monke-readsme.md entry
- **`monke-mermaid.mmd`** — +READSME node, +init→readsme edge

---

## 0.7 — 2026-03-25 — monke locks the gates

**Theme:** Every skill gets a bouncer. The drafter drops the skeleton law. Implement tightens its grip on boundary drift. Recon learns to count.

### Added
- **`monke-drafter.md`** — the skeleton law: mandatory structure for every skill file, voice rules, gate patterns, SRP boundaries
- **`CLAUDE.md`** — Agent Teams Fail Gate (hard stop if agent teams missing) + Skill Structural Standard (drafter is the law)

### Changed
- **`monke-CLAUDE.md`** — mirrored Agent Teams Fail Gate + rule 8 (skills reference agent teams) into template
- **`README.md`** — Sacred Tree: +monke-drafter.md entry
- **`monke-mermaid.mmd`** — +DRAFTER node in root subgraph
- **All 42 skill files** — Agent Teams Fail Gate added to Prerequisites (hard stop if CLAUDE.md missing agent teams section)
- **`monke-flash/spark.md`** — arguments expanded: orchestra napkin-bypass documented
- **`monke-flash/pulse.md`**, **`scope.md`**, **`spark.md`** — new Anti-Patterns to Refuse tables
- **`monke-implement/checkpoint.md`** — trace failure procedure (stack→boundary→HLD S7→integration test), rejection recovery protocol, +anti-patterns table
- **`monke-implement/implement.md`** — layer resume re-validates prior IL gates, LLD escalation hard pause, deferred test bookkeeping, boundary drift → hard gate with HLD amend required
- **`monke-implement/fill.md`** — +anti-patterns table (no TBD placeholders, no force-adding .claude/)
- **`monke-status/status.md`** — explicit source-file glob patterns for detection, phase ordering references HLD S8
- **`monke-rage/orchestra.md`** — drift-skip logic now explicit: skip drift only, continue remaining modes
- **`monke-docs/rage-run/template.md`** — fixed <<<mode>>> → <<<scope>>> placeholder bug
- **`monke-recon/orchestra.md`** — partial-LLD handling: cross-ref HLD S3, reconstruct only missing
- **`monke-recon/reconstruct.md`** — container-based agent grouping (one agent per HLD S2 container)
- **`monke-recon/roadmap.md`** — effort-size table gains time ranges (~1-3h, ~4-8h, ~2-4d, ~5-10d)

---

## 0.6 — 2026-03-25 — monke keeps receipts

**Theme:** The commit skill learns to journal. Seer gains an intelligence assessor. Rage fixes its filing cabinet.

### Added
- **`monke-seer/agentify.md`** — LATS any component for agentic rightsizing: candidate checks, pattern fitness audits, upgrade/de-escalation verdicts as ADRs

### Changed
- **`monke-ops/commit.md`** — new Phase 3 Log Entry: auto-generates monke-log entries with version bumping after commit groups confirmed; Stage & Commit merged into single `git add && git commit` command
- **`monke-rage/buggy.md`** — output path `rage-run/` → `rage-runs/`
- **`monke-rage/drift.md`** — output path `rage-run/` → `rage-runs/`
- **`monke-rage/echo.md`** — output path `rage-run/` → `rage-runs/`
- **`monke-rage/haunt.md`** — output path `rage-run/` → `rage-runs/`
- **`monke-rage/improv.md`** — output path `rage-run/` → `rage-runs/`
- **`monke-rage/renounce.md`** — output path `rage-run/` → `rage-runs/`

---

## 0.5 — 2026-03-25 — monke sees the future

**Theme:** Data science components become first-class tools in the agentic pipeline. The notebook kingdom falls — for real this time.

### Added
- **`monke-seer/`** — new skill directory for DS-native design concerns
  - `orchestra.md` — the bone reader's autopilot: reads status, recommends profiling/experiments/registry
  - `profile.md` — data profiling protocol: schema, distributions, drift surface as design inputs
  - `experiment.md` — experiments as LATS branches with metric evidence, recorded as ADRs
  - `registry.md` — versioned artifact pinning: version pins, eval thresholds, retraining triggers
  - `agentify.md` — stupefy in reverse: LATS any component's intelligence, proposes agentic upgrades or de-escalations

### Changed
- **`monke-docs/design-specs.md`**
  - S1.2: Added 3-subtype tool taxonomy (static-contract, versioned-artifact, data-dependent)
  - S1.2: Added Agentic Candidacy Heuristic — containers default to agentic when consuming non-static tools
  - S1.2: "All external capabilities are tools" principle — REST, ML, LLM, MCP, NLP all classified by contract behavior
  - S1.2: Protocol-orthogonal statement (HTTP, gRPC, MCP = implementation detail, not taxonomy)
  - S1.2: Updated CoALA template with `external(<tools: subtype>)` + conditional version/eval/drift fields
  - S5.1 S7: Added `Stability` column to boundary matrix
  - S9.3: Added 4 anti-pattern rows (refuse ML-as-separate-system, domain dirs, LLM-special, standalone pipelines)
  - S11: Added `Eval` test tier (runs within IL-2/IL-3, no new gates)
  - S11: Added Eval Tests subsection
  - S10 Glossary: +6 terms (3 subtypes, eval, stability, MCP)
- **`monke-docs/implementation-specs.md`**
  - Layer 0: Versioned types with artifact version pin + distributional expectations
  - Layer 2: Pure/IO decomposition guidance for inference components + eval test step
  - Layer 3: Integration-level eval tests
  - IL gate table: Updated IL-0, IL-2, IL-3 for version pins and eval
  - Glossary: +1 term (versioned type)
- **`monke-docs/test-specs.md`**
  - Tiers table: +Eval row (metric thresholds, no separate gate)
  - Tier Rules: +Eval Tests subsection
  - Mock Boundaries: +model artifacts, +data sources rows
  - Fixtures: +Test Dataset Fixtures subsection
  - Anti-patterns: +2 eval-specific rows
- **`monke-docs/project-specs.md`**
  - S9: +eval metric library, +eval test dataset dir bindings
  - S10.6: New optional DS infrastructure section (model registry, feature store, experiment tracker, eval thresholds)
- **`monke-design/hld.md`**
  - Phase 2: Agentic Candidacy Heuristic reference — LATS both options at L2
  - Phase 3: Updated CoALA template with tool subtypes + conditional fields
  - Phase 3: Tool-type LATS step — evaluate each capability as potential agent tool
  - Phase 4: +Stability column in boundary matrix, +eval note in phase plan
- **`monke-design/lld.md`**
  - Phase 2: Versioned-artifact decomposition guidance in ADaPT
  - Phase 3: +tool subtype fields in LLD header (tool subtypes, version pins, eval thresholds)
  - Phase 4: +Eval Test Plan table, +eval in test checklist
- **`README.md`** — Sacred Tree: +monke-seer block, +monke-ops block, +monke-phil, +monke-log, +monke-owns entries, +Seer skills table, +Ops skills table, skill count → 43
- **`monke-mermaid.mmd`** — +seer_skills subgraph with orchestra + 4 skills, +ops_skills subgraph, +LOG/PHIL root nodes, +14 edges

### Complexity budget
- 3 tool subtypes, 1 boundary matrix column, 1 test tier, 0 new pause gates, 0 new IL gates, 5 seer files, 1 Sacred Tree structural sync

---

## 0.4 — 2026-03-24 — monke multiplies

**Theme:** Skill explosion. Rage gets six flavors. Recon and Flash pipelines born. Every skill directory gets an orchestra. The framework goes from "design+build+test" to "flash+recon+design+build+test+rage."

### Added
- **`monke-rage/`** — sonar split into 6 specialized rage skills
  - `buggy.md` — logic errors, null bombs, swallowed exceptions
  - `drift.md` — spec-code divergence, status dashboard lies
  - `echo.md` — dead code, zombie imports, orphaned files
  - `haunt.md` — injection, auth gaps, hardcoded secrets
  - `improv.md` — perf wins, pattern upgrades, north stars
  - `renounce.md` — dead weight, cargo cult, duplication
- **`monke-recon/`** — promoted from design sub-skill to full pipeline (80%→100%)
  - `orchestra.md` — chains survey→reconstruct→gaps→oqs→roadmap
  - `survey.md` — deep codebase scan: components, dependencies, coverage
  - `reconstruct.md` — reverse-engineer HLD+LLDs from existing code
  - `gaps.md` — gap analysis against production requirements
  - `oqs.md` — surface implicit decisions and deferred problems
  - `roadmap.md` — phased production roadmap from survey+gaps
- **`monke-flash/`** — zero to MVP pipeline (0%→80%)
  - `orchestra.md` — chains spark→scope→sketch→blitz→pulse→snap
  - `spark.md` — the barstool pitch, capture the idea
  - `scope.md` — draw the 80% line, cut ruthlessly
  - `sketch.md` — napkin architecture, core entities + stack
  - `blitz.md` — happy path speedrun, no ceremony
  - `pulse.md` — smoke test, feedback loop
  - `snap.md` — freeze MVP, manifest every shortcut taken
- **Orchestra skills across all dirs**
  - `monke-design/orchestra.md` — design phase autopilot
  - `monke-implement/orchestra.md` — implementation assembly line
  - `monke-rage/orchestra.md` — pick your fury menu
  - `monke-test/orchestra.md` — quality loop dispatcher

### Changed
- **`monke-rage/sonar.md`** — removed (replaced by 6 specialized skills)
- **`monke-design/recon.md`** — removed (promoted to `monke-recon/`)
- **`monke-design/tinker.md`** — minor refinements
- **`monke-init.md`** — expanded with better bootstrap flow
- **`monke-sync.md`** — major rewrite: smarter diff handling, spec version tracking
- **`monke-status/status.md`** — minor tweaks
- **`monke-fut.md`** — minor update
- **`monke-docs/rage-run/template.md`** — template adjustments
- **`monke-docs/status-template.md`** — template adjustments
- **`CLAUDE.md`** — added agent teams directive and Sacred Tree invariant
- **`monke-CLAUDE.md`** — expanded project instructions

---

## 0.3 — 2026-03-23 — monke gets angry

**Theme:** The design skills get their big rewrite. Rage scanning arrives. The README grows teeth. The logo gets a face.

### Added
- **`monke-rage/sonar.md`** — the original codebase scanner (353 lines, 6 modes in one skill)
- **`monke-ops/commit.md`** — commit skill with semantic message generation
- **`monke-docs/rage-run/template.md`** — sonar scan log template

### Changed
- **`monke-design/hld.md`** — massive expansion (+307 lines): evolve flow, amend flow, agent teams check, mode detection, solo/team modes, full greenfield pipeline, status update protocol
- **`monke-design/lld.md`** — refinements to ADaPT flow
- **`monke-design/recon.md`** — added brownfield reconstruction guidance
- **`monke-design/tinker.md`** — expanded stack detection and placeholder filling
- **`monke-implement/fill.md`** — improved placeholder resolution workflow
- **`monke-sync.md`** — sync flow updates
- **`monke-mermaid.mmd`** — added rage skill nodes and edges
- **`monke-CLAUDE.md`** — added rage integration reference
- **`monke-phil.md`** — expanded ML/Data Science section, refined rhetoric
- **`README.md`** — updated skill tables, added rage section, link fixes
- **`monke-owns/wyrdLogo.png`** — the monke gets a proper face

---

## 0.2 — 2026-03-22 — monke scaffolds the world

**Theme:** Everything gets created. The four spec files. All design, implementation, and test skills. The status dashboard. Init and sync lifecycle. The philosophy drops. The Sacred Tree is planted.

### Added
- **`monke-docs/`** — the ancient texts
  - `sdlc-specs.md` — the prophecy connecting all specs (88 lines)
  - `design-specs.md` — how monke thinks before monke builds (630 lines)
  - `implementation-specs.md` — how monke builds, shape before behavior (243 lines)
  - `test-specs.md` — how monke proves it works (144 lines)
  - `project-specs.md` — the binding scroll with <<<placeholders>>> (162 lines)
  - `status-template.md` — blank dashboard template
  - `hld.md` — empty, waiting for dreams
  - `open-questions.md` — empty, waiting for 3am hauntings
  - `lld/`, `decisions/`, `checkpoints/` — empty directories
- **`monke-design/`** — thinking skills
  - `hld.md` — L1→L2→L3 pipeline with LATS and pause gates
  - `lld.md` — ADaPT decomposition for one component
  - `adr.md` — architecture decision records
  - `oq.md` — open question triage
  - `tinker.md` — stack detection and placeholder filling
  - `recon.md` — brownfield codebase reconnaissance (later promoted to `monke-recon/`)
- **`monke-implement/`** — building skills
  - `implement.md` — Layer 0→3 pipeline with IL gates
  - `fill.md` — placeholder customs agent
  - `checkpoint.md` — PG-11, the final boss
- **`monke-test/`** — testing skills
  - `test-plan.md` — plot every failure scenario
  - `test-run.md` — run, fail, categorize, retry
  - `coverage.md` — find the untouched bananas
- **`monke-status/status.md`** — the all-seeing dashboard
- **`monke-init.md`** — one ring to rule them all
- **`monke-sync.md`** — pull without nuking
- **`monke-phil.md`** — the manifesto: roles died, disciplines didn't
- **`monke-CLAUDE.md`** — project instructions for Claude
- **`monke-mermaid.mmd`** — the relationship graph
- **`README.md`** — the Sacred Tree, branch flow, quick start
- **`.gitignore`** — the usual suspects

---

## 0.1 — 2026-03-21 — a monke is born

**Theme:** A repo appears. The primordial banana.

### Added
- `LICENSE` — Apache 2.0, monke shares freely
- `README.md` — two lines of hope

---

*the log remembers what monke forgets.*
