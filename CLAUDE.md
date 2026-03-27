# wyrdMonke — Project Instructions

## Agent Teams: Default Mode of Operation

**Always use agent teams (parallel subagents) as the default mode of operation in this repository.** This is a hard requirement, not a suggestion.

### Rules

1. **Decompose every non-trivial task into independent subtasks and dispatch them as parallel agents.** Do not work sequentially when parallel execution is possible.
2. **Use the Agent tool liberally.** Prefer spawning specialized subagents (Explore, Plan, data-analyst-expert, general-purpose, etc.) over doing everything in the main conversation context.
3. **Run independent agents in parallel within a single message.** If two or more subtasks have no dependency on each other, launch them simultaneously — never sequentially.
4. **Use background agents (`run_in_background: true`) for long-running work** so the main conversation remains responsive.
5. **Prefer agent isolation (`isolation: "worktree"`) for any agent that writes code**, to avoid conflicts between parallel agents editing the same files.
6. **Reserve the main context for coordination, synthesis, and user communication.** Heavy research, exploration, code generation, and testing should be delegated to agents.
7. **You are free to create your own agent orchestration and prompts.** Design custom agent workflows, compose novel multi-agent pipelines, and craft specialized prompts tailored to the task at hand. You are not limited to predefined patterns — invent new orchestration strategies when the situation calls for it. Think of yourself as an agent architect, not just an agent dispatcher.
8. **Skills reference agent teams.** When a skill says "use agent teams," "spawn parallel scanners," or "if >N files/components, parallelize" — it means: use the Claude Code Agent tool. For code-writing agents, use `isolation: "worktree"`. For read-only scans, agents can share the workspace. The skill provides the orchestration logic; the Agent tool provides the mechanism.

### Agent Teams Fail Gate

**Every skill MUST verify agent teams are enabled before proceeding.** On entry, read `CLAUDE.md` and confirm the Agent Teams section exists with the parallel subagent directive. If missing or absent:

- **Stop immediately.** Do not proceed with the skill.
- Tell the user: "Agent teams are not configured. WyrdMonke skills require agent teams to operate. Run `/monke-sync` to pull the latest template, or copy the Agent Teams section from `monke-CLAUDE.md` into your project's `CLAUDE.md`."
- This is a **hard gate** — no skill runs without it.

### Anti-Patterns (Do NOT Do These)

- Working through a large task entirely in the main context when it could be parallelized.
- Running agents sequentially when they have no dependencies on each other.
- Doing research and implementation in the same agent when they could be split.
- Spawning a single agent for work that could be split across multiple parallel agents.

---

## Sacred Tree Invariant

**The Sacred Tree (in `README.md`), the skill directories, and `monke-mermaid.mmd` are a single source of truth that must stay in sync.**

1. **Any change to a skill directory or the root dir** (add, remove, rename a `.md` file) **MUST update the Sacred Tree** in `README.md`. Every line in the tree has a punchy, irreverent comment — new lines are no exception. Match the tone: short metaphor, vivid verb, personality. Bland descriptions are a crime against monke.
2. **Any change to the Sacred Tree MUST update `monke-mermaid.mmd`** — add/remove nodes and edges to match the new structure.
3. **The chain is non-negotiable:** skill dir change → Sacred Tree update → mermaid update. Skip a step, shame on monke.

---

## Skill Structural Standard

**When creating or modifying any skill, follow [`monke-drafter.md`](monke-drafter.md) exactly.** It defines the mandatory skeleton, voice rules, gate patterns, SRP boundaries, and the checklist every skill must pass before shipping. No exceptions — the drafter is the law.
