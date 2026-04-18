# CLAUDE.md

<<<project_description>>>

## Architecture

**Read `monke-status.md` first** — it shows current SDLC progress, per-component maturity, and what to do next.

**Project details** (commands, stack, env vars, directory tree) are in [`project-specs.md`](monke-docs/project-specs.md).

**Rigor level** (light / standard / thorough) is in [`.monke-config.md`](.monke-config.md). It determines which gates surface to the human and which auto-pass.

See [`monke-mermaid.mmd`](monke-mermaid.mmd) for the full doc/directory relationship graph.

Do not assume structure — derive it from `monke-status.md`, the skills, and the code. Skills inline the spec sections they need; you don't need to read spec files upfront.

## Gotchas

<<<project_invariants>>>

## Progressive Formalization

Documentation emerges from work, not precedes it. For existing components, LLDs are reverse-engineered from code and refined as gaps are filled. For greenfield components, LLDs are designed first. Either way: by the time all components reach IL-3, the project is fully documented as a side effect. See [`sdlc-specs.md`](monke-docs/sdlc-specs.md) for the full progressive formalization model.

## Skills

Skills are slash commands that orchestrate each SDLC phase. Run `/monke-init` to install everything project-local at `.claude/commands/`. Entry point for ongoing work is `/monke` (the unified orchestrator).

Core commands:

| Skill | What it does |
|-------|-------------|
| `/monke-init [branch]` | Install skills, scaffold project, set rigor — the one-command setup |
| `/monke-sync [branch]` | Update skills and specs from upstream without touching your design artifacts |
| `/monke [rigor] [override]` | Unified orchestrator — auto-detects stack, routes per-component by maturity |
| `/monke-intake "<request>"` | Natural-language feature intake — classifier + scoper team names the pipeline, one HARD approval, intake becomes permanent lead for the thread. Shortcut: `/monke "<quoted request>"`. |
| `/monke-status:status [action]` | Dashboard — show / rebuild / next / blocked / resume |

Phase skills (design / implement / test / rage / recon / flash) are discovered automatically by `/monke`. Each skill is self-contained — relevant spec sections are inlined so no external spec loading is needed at runtime.

Cross-cutting rules:
- **Adaptive gates:** AUTO gates stay silent (surface on failure), HARD gates always surface, SOFT gates surface per rigor level, TRIGGERED gates fire on events.
- **Status tracking:** Every skill reads `monke-status.md` on entry and updates it on exit. During parallel teammate work, only the lead/orchestrator writes `monke-status.md` — teammates write per-component files; lead reconciles.

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
3. **Plan**: enter plan mode. List each teammate with a two-word codename:
    - **Word 1**: an evocative, "cool" descriptor — drawn from the same *register* as examples like phantom, neon, vortex, cipher, blitz (think: cyber/noir/kinetic/arcane/elemental vibes). Do **not** reuse any word from the examples; generate fresh ones in the same spirit (e.g. specter, prism, riptide, glyph, shadow, pulse, ember, quantum, nomad, void).
    - **Word 2**: a functional noun describing the teammate's task/responsibility (parser, extractor, mapper, scorer, linker, etc.).

    Format: `codename-role` (e.g. specter-parser, prism-extractor).

    **Constraints**:
        - Every first word must be unique across the team.
        - No first word may match the examples verbatim.
        - Maintain the aesthetic — avoid bland descriptors (smart, fast, good).

    Then list role, file ownership, dependency edges. Present for approval.
4. **Execute**: `TeamCreate` → `TaskCreate` (×N) → wire `addBlockedBy` → spawn teammates → lead delegates original content creation → wait for all `TaskUpdate(completed)`.
5. **Teardown**: `SendMessage(shutdown_request)` to each → `TeamDelete`.

### Rules

- **Team sizing and archetype selection per [`monke-docs/design-specs.md`](monke-docs/design-specs.md) S3.1 (Team Decision Heuristic).** Default team size is 2. Use the heuristic — do not hardcode a team size.
- Each teammate owns distinct files — no shared-file edits.
- Embed full context into spawn prompts — teammates have no conversation history.
- **Lead coordinates and synthesizes.** Lead delegates original content creation. Lead may directly write: status updates, artifact assembly from teammate outputs, test plans, and changes under 20 lines that don't require review. If lead is drafting new feature code, architecture decisions, or substantive design content — STOP and delegate.
- Use `planModeRequired: true` for risky teammates.

### Agent Teams Fail Gate

**Every skill MUST verify agent teams are enabled before proceeding.** On entry, read `CLAUDE.md` and confirm the Agent Teams section exists with the parallel subagent directive. If missing or absent:

- **Stop immediately.** Do not proceed with the skill.
- Tell the user: "Agent teams are not configured. WyrdMonke skills require agent teams to operate. Run `/monke-sync` to pull the latest template, or copy the Agent Teams section from `monke-CLAUDE.md` into your project's `CLAUDE.md`."
- This is a **hard gate** — no skill runs without it.

**Exemptions (bootstrap paradox):** `/monke-init` and `/monke-sync` are exempt from this gate. They are the skills that CREATE or UPDATE `CLAUDE.md` itself, so requiring `CLAUDE.md`'s Agent Teams section as a prereq would be circular. They fall back to checking `monke-CLAUDE.md` in the upstream clone.

---

## Sacred Tree Invariant

**The Sacred Tree (in `README.md`), the skill directories, and `monke-mermaid.mmd` are a single source of truth that must stay in sync.**

1. **Any change to a skill directory or the root dir** (add, remove, rename a `.md` file) **MUST update the Sacred Tree** in `README.md`. Every line in the tree has a punchy, irreverent comment — new lines are no exception. Match the tone: short metaphor, vivid verb, personality. Bland descriptions are a crime against monke.
2. **Any change to the Sacred Tree MUST update `monke-mermaid.mmd`** — add/remove nodes and edges to match the new structure.
3. **The chain is non-negotiable:** skill dir change → Sacred Tree update → mermaid update. Skip a step, shame on monke.

## Dependency Pinning

**Pin every installed library to an exact version and commit the manifest change in the same action.**

- Python: `pip install foo==1.2.3` → add `foo==1.2.3` to `requirements.txt` (or `pyproject.toml` / `Pipfile`).
- Node: `npm install foo@1.2.3 --save-exact` → verify `package.json` shows `"foo": "1.2.3"` (no `^` / `~`).
- Rust: `cargo add foo@=1.2.3` → verify `Cargo.toml` shows `foo = "=1.2.3"`.
- Other stacks: use the equivalent exact-version syntax and update the manifest.

No floating versions. No missing manifest entry. If the language has no `==` analogue, pin however that ecosystem pins and say so in the commit. Reproducibility is non-negotiable.

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
