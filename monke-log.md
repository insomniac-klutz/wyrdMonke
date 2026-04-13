# monke-log.md

> *every banana has a story. here's the tree they fell from.*

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
