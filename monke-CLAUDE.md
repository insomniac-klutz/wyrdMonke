# CLAUDE.md

<<<project_description>>>

## Architecture

**Read `monke-status.md` first** — it shows current SDLC progress and what to do next.

**Project details** (commands, stack, env vars, directory tree) are in [`project-specs.md`](monke-docs/project-specs.md).

See [`monke-mermaid.mmd`](monke-mermaid.mmd) for the full doc/directory relationship graph.

Do not assume structure — derive it from `monke-status.md`, the skills, and the code. Skills read the specs they need; you don't need to read them upfront.

## Gotchas

<<<project_invariants>>>

## Specs Index

Reference only — skills load these as needed. Do not read upfront.

| Doc | Governs |
|-----|---------|
| [`sdlc-specs.md`](monke-docs/sdlc-specs.md) | End-to-end phase flow, doc hand-offs |
| [`design-specs.md`](monke-docs/design-specs.md) | HLD/LLD creation, pause gates, ADRs |
| [`implementation-specs.md`](monke-docs/implementation-specs.md) | Layer pipeline, code standards |
| [`test-specs.md`](monke-docs/test-specs.md) | Test tiers, fixtures, coverage |

## Skills

Skills are slash commands that orchestrate each SDLC phase. Run `/monke-init` to install everything project-local at `.claude/commands/`.

| Skill | What it does |
|-------|-------------|
| `/monke-init [branch]` | Install skills, scaffold project, merge CLAUDE.md — the one-command setup |
| `/monke-sync [branch]` | Update skills and specs from upstream without touching your design artifacts |
| `/monke-status:status [action]` | Dashboard — show progress, rebuild from artifacts, find next step, list blockers |
| `/monke-design:tinker` | Detect stack, fill project-specs and CLAUDE.md placeholders, initialize dashboard |
| `/monke-design:recon [scope]` | Reverse-engineer HLD from existing codebase, surface open questions |
| `/monke-design:hld [resume]` | Greenfield HLD creation — L1→L2→L3 with LATS, Agent Teams, PG-1 through PG-4 |
| `/monke-design:lld <component>` | Component LLD — ADaPT decomposition, Designer+Reviewer teams, PG-8 through PG-10 |
| `/monke-design:adr <title> [component]` | Architecture Decision Record from LATS output |
| `/monke-design:oq [action] [id]` | Open question management — list, triage, resolve |
| `/monke-implement:fill [group]` | Fill project-specs placeholder groups 1-8 |
| `/monke-implement:implement <component> [layer]` | Layer 0→3 pipeline — types, stubs, bodies+tests, integration |
| `/monke-implement:checkpoint <phase>` | Phase checkpoint — verify all IL-3s, run system tests, PG-11 sign-off |
| `/monke-test:test-plan <component>` | Generate unit + integration test plans from LLD |
| `/monke-test:test-run <tier> [scope]` | Execute tests by tier (unit/integration/system) and verify gates |
| `/monke-test:coverage [scope]` | Coverage analysis against project threshold |

Cross-cutting rules:
- **Pause gates:** Claude stops and waits for user confirmation at every decision point.
- **Status tracking:** Every skill reads `monke-status.md` on entry and updates it on exit.
