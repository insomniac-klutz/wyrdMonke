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
| `/monke-design:hld [resume]` | Greenfield HLD creation — L1→L2→L3 with LATS, Agent Teams, PG-1 through PG-4 |
| `/monke-design:lld <component>` | Component LLD — ADaPT decomposition, Designer+Reviewer teams, PG-8 through PG-10 |
| `/monke-design:adr <title> [component]` | Architecture Decision Record from LATS output |
| `/monke-design:oq [action] [id]` | Open question management — list, triage, resolve |
| `/monke-flash:spark` | Capture the idea — ask the right questions, produce a flash brief |
| `/monke-flash:scope` | Draw the 80% line — what's in the MVP, what's out, what "done" looks like |
| `/monke-flash:sketch` | Napkin architecture — core entities, stack picks, folder structure, one data flow |
| `/monke-flash:blitz` | Build the happy path — scaffold, implement core flows, skip ceremony |
| `/monke-flash:pulse` | Smoke test + feedback loop — run MVP, collect feedback, loop to blitz |
| `/monke-flash:snap` | Freeze the MVP — tag it, manifest what was built/cut/shortcut, hand off to recon |
| `/monke-recon:survey` | Deep codebase scan — map components, dependencies, test coverage, code quality |
| `/monke-recon:reconstruct` | Reverse-engineer HLD and LLDs from existing code |
| `/monke-recon:gaps` | Gap analysis against production requirements — prioritized by blast radius |
| `/monke-recon:oqs` | Surface implicit decisions — every shortcut and default exposed |
| `/monke-recon:roadmap` | Phased production roadmap from survey + gaps + OQs |
| `/monke-rage:orchestra [scope]` | Pick a rage mode and scan — menu entrypoint for all 6 modes |
| `/monke-rage:buggy [scope]` | Hunt bugs — logic errors, null bombs, swallowed exceptions, contract violations |
| `/monke-rage:improv [scope]` | Hunt improvements — perf wins, pattern upgrades, type safety, north stars |
| `/monke-rage:renounce [scope]` | Hunt redundancy — dead weight, duplication, cargo cult, over-abstraction |
| `/monke-rage:haunt [scope]` | Hunt security — injection, auth gaps, secrets, OWASP top 10 |
| `/monke-rage:drift [scope]` | Hunt divergence — spec says X, code does Y, status is lying |
| `/monke-rage:echo [scope]` | Hunt dead code — unreachable paths, zombie imports, orphaned files |
| `/monke-implement:fill [group]` | Fill project-specs placeholder groups 1-8 |
| `/monke-implement:implement <component> [layer]` | Layer 0→3 pipeline — types, stubs, bodies+tests, integration |
| `/monke-implement:checkpoint <phase>` | Phase checkpoint — verify all IL-3s, run system tests, PG-11 sign-off |
| `/monke-test:test-plan <component>` | Generate unit + integration test plans from LLD |
| `/monke-test:test-run <tier> [scope]` | Execute tests by tier (unit/integration/system) and verify gates |
| `/monke-test:coverage [scope]` | Coverage analysis against project threshold |

Cross-cutting rules:
- **Pause gates:** Claude stops and waits for user confirmation at every decision point.
- **Status tracking:** Every skill reads `monke-status.md` on entry and updates it on exit.

## Agent Teams (mandatory for all work)

### Tool Chain

```
TeamCreate → TaskCreate (×N) → TaskUpdate (deps) → Agent w/ team_name (×N) → SendMessage → TaskUpdate (complete) → TeamDelete
```

### Tools

- `TeamCreate({ team_name, description })` — create team
- `TaskCreate({ subject, description, activeForm })` — add task to shared list
- `TaskUpdate({ taskId, owner, status, addBlockedBy })` — claim/complete/block tasks
- `TaskList` — check task statuses
- `Agent({ name, team_name, subagent_type: "general-purpose", prompt, run_in_background: true })` — **spawn teammate**
- `SendMessage({ type, recipient, content })` — teammate messaging / shutdown
- `TeamDelete` — cleanup after shutdown

### The One Rule That Matters

`Agent` WITHOUT `team_name` = subagent (isolated, no coordination). **NEVER use this.**
`Agent` WITH `team_name` + `name` = teammate (shared task list + mailbox). **ALWAYS use this.**

### Steps

1. **Explore**: `TeamCreate` → spawn 2–3 scout teammates → gather findings via `SendMessage`.
2. **Clarify**: ask user targeted questions based on findings.
3. **Plan**: enter plan mode. List each teammate (two-word cool and quircky codename , first word cool phrase , second word describing the task/responsibility ex phantom-parser, neon-extractor, vortex-mapper, cipher-scorer, blitz-linker), role, file ownership, dependency edges. Present for approval.
4. **Execute**: `TeamCreate` → `TaskCreate` (×N) → wire `addBlockedBy` → spawn teammates → lead delegates only, does NOT write code → wait for all `TaskUpdate(completed)`.
5. **Teardown**: `SendMessage(shutdown_request)` to each → `TeamDelete`.

### Rules

- ALL work goes through agent teams. Single-file / <20-line exceptions require explicit user permission.
- Each teammate owns distinct files — no shared-file edits.
- 3–5 teammates, 5–6 tasks each.
- Embed full context into spawn prompts — teammates have no conversation history.
- Lead coordinates only. If lead starts writing code, STOP and delegate.
- Use `planModeRequired: true` for risky teammates.

### Agent Teams Fail Gate

**Every skill MUST verify agent teams are enabled before proceeding.** On entry, read `CLAUDE.md` and confirm the Agent Teams section exists with the parallel subagent directive. If missing or absent:

- **Stop immediately.** Do not proceed with the skill.
- Tell the user: "Agent teams are not configured. WyrdMonke skills require agent teams to operate. Run `/monke-sync` to pull the latest template, or copy the Agent Teams section from `monke-CLAUDE.md` into your project's `CLAUDE.md`."
- This is a **hard gate** — no skill runs without it.

## Commit Format

```
action : description
```

- All lowercase
- Action is the verb: `add`, `update`, `fix`, `remove`, `refactor`, `rename`, etc.
- Then ` : ` (space-colon-space)
- Then a short description of what changed

Examples:
```
add : project scaffold and meta files
update : hld with revised component boundaries
fix : missing pause gate in sync phase 2
remove : deprecated recon fallback logic
refactor : test-run tier resolution
rename : status template to match new schema
```

