# Future Gaps — things monke hasn't figured out yet

> Not broken. Not blocking. But you'll hit these walls eventually, and when you do, monke has no scroll to hand you.

---

## Deployment & Release

The SDLC ends at PG-11 (phase checkpoint). Code works. Tests pass. Then what?

No guidance for: how to tag a release (README has banana tags but no process), how to verify a deploy, rollback strategy, or changelog generation. A project following WyrdMonke gets from "code works" to "code ships" on vibes alone.

**What a fix looks like:** A `release-specs.md` or an addition to `sdlc-specs.md` covering: tagging convention enforcement, deploy verification gates, rollback triggers, changelog from ADRs + checkpoints.

---

## Maintenance & Refactoring

The specs handle greenfield (tinker) and brownfield-bootstrap (update), but not ongoing evolution.

- When should the HLD be revisited?
- When does tech debt warrant an ADR?
- If a component outgrows its LLD, is the process "write a new LLD" or "revise the existing one"?

Design-specs S5.4 covers HLD maintenance triggers, but there's no equivalent for project-level lifecycle beyond the initial build. Monke builds the temple, but nobody wrote the maintenance manual.

**What a fix looks like:** A maintenance section in `sdlc-specs.md` or a standalone doc covering: HLD revision cadence, LLD staleness signals, tech debt tracking via open questions, and when to re-run `/monke-update`.

---

## Monorepo Support

Container discovery (monke-update §1.2) detects monorepo workspace members, but the doc structure assumes a single `monke-docs/` per project root.

In a monorepo with 3 services: one shared HLD with 3 containers, or 3 separate `monke-docs/` directories? The specs are silent.

**What a fix looks like:** Guidance in `design-specs.md` or `project-specs.md` on: shared vs per-workspace `monke-docs/`, HLD scope (one HLD per repo vs per service), and how boundaries between workspaces map to the boundary matrix.

---

## Polyglot Boundary Ownership

v2.0 added per-container project-specs tables (S8/S9), so different containers can bind to different languages and test runners. That solves the binding side of polyglot. What's still open:

- Cross-language contract ownership — which side is source-of-truth when a TS frontend calls a Python backend? The boundary matrix has a contract column, but no rule for which container OWNS it.
- Shared type generation — OpenAPI → TS client + Python server? Protobuf? Hand-written dual types? No prescribed strategy.
- Serialization verification as an IL-0 gate — when the contract crosses a language boundary, IL-0 should verify the wire format round-trips, not just that each side's types compile.

**What a fix looks like:** A section in `implementation-specs.md` covering: boundary contract ownership rules, supported shared-type strategies with tradeoffs, and a cross-language IL-0 serialization gate.

---

## Operations & Release Skills (`monke-ops/`)

The skills package now covers design → implement → test, but the SDLC still ends at PG-11. There's no full ops skill suite to carry a project from "phase checkpoint passed" to "deployed, monitored, and maintainable." `/monke-ops:commit` exists; the rest is open.

Deferred to v2.1:
- `/monke-ops:release` — deploy gate pipeline: tagging, changelog from ADRs + checkpoints, deploy verification gates
- `/monke-ops:rollback` — rollback triggers, canary / blue-green strategy
- `/monke-ops:maintain` — HLD revision cadence, LLD staleness signals, tech debt tracking, when to re-run `/monke-recon:survey`
- `/monke-ops:observe` — observability scaffolding generated from the HLD S7 boundary matrix (dashboards per boundary, SLOs per contract, alerts per IL gate)
- `/monke-ops:ci` — CI config generation from IL gate bindings in project-specs (each gate becomes a CI job)

**What a fix looks like:** Extend `monke-ops/` with these skills plus a `release-specs.md` or extension to `sdlc-specs.md` covering the deploy → observe → maintain loop.

---

## Performance Profiling (`/monke-rage:bench`)

Rage covers bugs, drift, dead code, security, duplication, ambitious improvements — but not performance measurement. `improv` suggests perf wins in the abstract; there's no skill that actually runs a benchmark, records baselines, and flags regressions.

Deferred to v2.1. Would complement the rage suite with a measurement-first mode that produces a bench-run log analogous to rage-run.

---

## Dry-Run / Preview Mode

Every skill in v2.0 dispatches and executes. There's no `--dry-run` mode that shows "what would I do if you confirmed?" without writing artifacts or spawning teams. Useful for learning the framework and for high-stakes gates where the user wants to preview the full action plan before committing.

**What a fix looks like:** A drafter-level convention — every skill supports a `preview` argument that walks its decision tree and emits the would-be dispatches + artifact writes, with no side effects. Could be a dispatcher-level feature in `/monke preview`.

---

## Declarative Skill Dependency Graph

The Sacred Tree is currently a shape; the mermaid is currently edges by hand. There's no machine-readable declaration of what each skill reads, writes, and dispatches. This costs us at audit time (drift between mermaid and reality) and at onboarding (users learn the graph by reading 36 files).

**What a fix looks like:** Drafter defines a `Reads / Writes / Dispatches` frontmatter block at the top of every skill. A tool (or a rage mode) parses these and verifies the mermaid, README skills tables, and the actual dispatch edges in code are all in sync.

---

## Fast Onboarding: IaC and ML Notebook Support

`/monke` fast onboarding detects manifests (package.json, pyproject.toml, Cargo.toml, etc.) but not:
- Terraform / OpenTofu / Pulumi / CloudFormation — infrastructure containers are invisible.
- Jupyter notebooks (`.ipynb`) — ML projects with notebook-driven experimentation show up as pre-L0 components because there's no type-check toolchain, but they're often more mature than that.

Deferred to v2.1. Would extend Step 2.1's signal file table.

---

## Meta-Learning from Execution Annotations

The Gate Audit Log records every gate outcome. The rage-runs directory records every scan finding. There's no skill that reads these across sessions to surface patterns: "this project's drift scans always find the same 3 boundaries misaligned" or "this container's IL-2 fails on the same assertion pattern."

Deferred to v2.1. Could become a `/monke learn` or `/monke-rage:patterns` skill that reads the audit + rage logs and proposes ADRs or spec amendments.

---

## Persistent Cross-Session Teams

Agent teams exist for the duration of one invocation. Teams are torn down at the end of each skill. For long-running projects with recurring review pairs or critic loops, this means re-spawning and re-briefing teammates every session — and losing whatever implicit coordination they built up.

Deferred to v2.1. Would require Claude Code platform support for team persistence across sessions.

---

## Rage Skill Skeleton Extraction

The six rage skills share ~80% of their structure (args parsing, prerequisites, rage-run template, dispatch, output format). That duplication is fine for discoverability but painful to maintain — a drafter change today requires 6 parallel edits.

Deferred to v2.1. Could extract a `monke-rage/common.md` skeleton that each mode references, keeping only the mode-specific scan heuristics in the individual skill files.

---

## Skill Composition & Chaining

v2.0 largely addressed this: the unified `/monke` orchestrator dispatches skills in the right order per project state, so users rarely need to chain manually. Still open: skills calling skills natively (without the orchestrator in the middle) — today, when one skill references another, it's an inline re-implementation or a handoff via status file, not a platform-native invocation.

**What a fix looks like:** When Claude Code adds native skill invocation, replace "follow `/monke-test:test-plan` logic inline" with actual skill calls. The orchestrator-centric model remains; composition would just get cleaner.
