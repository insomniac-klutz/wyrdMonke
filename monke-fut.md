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

## Multi-Language Pipeline

project-specs has a "Supported Languages" table and design-specs has language selection rules, but implementation-specs' Layer 0-3 pipeline assumes a single language context.

If your backend is Python and frontend is TypeScript: do you run two parallel Layer 0-3 pipelines? Who owns the shared types at the cross-language boundary? How do IL gates work when Layer 0 types exist in two languages?

**What a fix looks like:** A section in `implementation-specs.md` covering: per-container pipelines, cross-language contract ownership (which side is source-of-truth), serialization verification as an IL-0 gate, and shared type generation strategies.

---

## Operations & Release Skills (`monke-ops/`)

The skills package now covers design → implement → test, but the SDLC still ends at PG-11. There's no `monke-ops/` or `monke-release/` package to carry a project from "phase checkpoint passed" to "deployed, monitored, and maintainable."

Missing skills:
- `/monke-ops:release` — tagging, changelog from ADRs+checkpoints, deploy verification gates
- `/monke-ops:rollback` — rollback triggers, canary/blue-green strategy
- `/monke-ops:maintain` — HLD revision cadence, LLD staleness signals, tech debt tracking, when to re-run `/monke-design:recon`

**What a fix looks like:** A fourth skill package (`monke-ops/`) with a `release-specs.md` or extension to `sdlc-specs.md`. Skills that carry the project past PG-11 into deployment, monitoring, and ongoing evolution.

---

## Skill Composition & Chaining

Skills reference each other (e.g., `/monke-design:lld` hands off to `/monke-test:test-plan`), but there's no formal mechanism for one skill to invoke another. The current approach is "follow the referenced skill's logic inline" — which works but duplicates intent.

If Claude Code adds native skill chaining or tool-use within skills, the cross-references should become actual invocations rather than inline re-implementations.

**What a fix looks like:** When the platform supports it, replace "follow `/monke-test:test-plan` logic inline" with actual skill invocation. Until then, the inline approach is the pragmatic choice.
