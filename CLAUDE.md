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

---

## Sacred Tree Invariant

**The Sacred Tree (in `README.md`), the skill directories, and `monke-mermaid.mmd` are a single source of truth that must stay in sync.**

1. **Any change to a skill directory or the root dir** (add, remove, rename a `.md` file) **MUST update the Sacred Tree** in `README.md`. Every line in the tree has a punchy, irreverent comment — new lines are no exception. Match the tone: short metaphor, vivid verb, personality. Bland descriptions are a crime against monke.
2. **Any change to the Sacred Tree MUST update `monke-mermaid.mmd`** — add/remove nodes and edges to match the new structure.
3. **The chain is non-negotiable:** skill dir change → Sacred Tree update → mermaid update. Skip a step, shame on monke.

---

## Skill Structural Standard

**When creating or modifying any skill, follow [`monke-drafter.md`](monke-drafter.md) exactly.** It defines the mandatory skeleton, voice rules, gate patterns, SRP boundaries, and the checklist every skill must pass before shipping. No exceptions — the drafter is the law.
