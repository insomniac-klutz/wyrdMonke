# wyrdMonke — Project Instructions

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

## Rigor & Progressive Formalization

**Rigor level** (light / standard / thorough) for this project is in [`.monke-config.md`](.monke-config.md) (created by `/monke-init`). It determines which gates surface to the human and which auto-pass.

**Progressive formalization:** documentation emerges from work, not precedes it. Existing components get reverse-engineered LLDs refined as gaps are filled; greenfield components get design-first LLDs. See [`monke-docs/sdlc-specs.md`](monke-docs/sdlc-specs.md) for the full model.

---

## Sacred Tree Invariant

**The Sacred Tree (in `README.md`), the skill directories, and `monke-mermaid.mmd` are a single source of truth that must stay in sync.**

1. **Any change to a skill directory or the root dir** (add, remove, rename a `.md` file) **MUST update the Sacred Tree** in `README.md`. Every line in the tree has a punchy, irreverent comment — new lines are no exception. Match the tone: short metaphor, vivid verb, personality. Bland descriptions are a crime against monke.
2. **Any change to the Sacred Tree MUST update `monke-mermaid.mmd`** — add/remove nodes and edges to match the new structure.
3. **The chain is non-negotiable:** skill dir change → Sacred Tree update → mermaid update. Skip a step, shame on monke.

---

## Skill Structural Standard

**When creating or modifying any skill, follow [`monke-drafter.md`](monke-drafter.md) exactly.** It defines the mandatory skeleton, voice rules, gate patterns, SRP boundaries, and the checklist every skill must pass before shipping. No exceptions — the drafter is the law.
