# WyrdMonke Flash:Blitz — Build the happy path

> **Usage:** Copy `monke-flash/` to `.claude/commands/monke-flash/`. Invoke: `/monke-flash:blitz [component]`

---

## Arguments

`$ARGUMENTS` parsing:
- Single positional: component or flow name to focus on, or `all` (default)
- Example: `/monke-flash:blitz auth` or `/monke-flash:blitz`

```
TARGET="${ARGUMENTS:-all}"
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

- `monke-docs/flash/flash-arch.md` exists (run `/monke-flash:sketch` first)
- `monke-docs/flash/flash-scope.md` exists

---

## Phase 1: Load the Blueprint

Read `flash-arch.md` and `flash-scope.md`. Build a mental checklist:

| Flow | Status | Notes |
|------|--------|-------|
| Flow 1 | not started | |
| Flow 2 | not started | |
| Flow 3 | not started | |

If `TARGET` is not `all`, filter to just that flow/component.

---

## Phase 2: Scaffold

Create the project structure from `flash-arch.md`:
- Directories
- Config files (package.json, Cargo.toml, pyproject.toml, etc.)
- Manifest / dependency files
- Entry point stub

**Install dependencies.** Get the project to a state where it compiles/runs (even if it does nothing).

Commit: `flash : scaffold`

---

## Phase 3: Build Each Flow

For each IN-scope flow from `flash-scope.md`, in order:

### 3.1 Implement

- Write the code for this flow. **Happy path only.**
- No edge cases. No validation beyond what prevents crashes. No error handling beyond what would make the app unusable.
- Hardcode what you can. Seed data instead of building CRUD. Use defaults instead of config.
- If it needs an API key or external service that's not available — stub it with mock data, note it as blocked, move on.

### 3.2 Verify

- Run it. Does the happy path work?
- If yes -> commit and move to next flow
- If no -> fix the minimum to unblock, then verify again

### 3.3 Commit

Format: `flash : <what works now>`

Examples:
- `flash : users can submit a form`
- `flash : search returns results from seeded data`
- `flash : CLI parses input and generates output`

### Agent Teams

If agent teams are available and flows are independent — **build flows in parallel** using isolated agents. Each agent gets one flow, the arch doc, and the scope doc. Merge results.

If flows depend on each other (flow 2 needs flow 1's output), build sequentially.

---

## Phase 4: Status Check

After all targeted flows are built (or attempted), present:

```
Blitz Status — <project name>

Flow 1: <name>     [done | partial | blocked]
Flow 2: <name>     [done | partial | blocked]
Flow 3: <name>     [done | partial | blocked]

Blocked items:
- <what's blocked and why>

Shortcuts taken:
- <hardcoded values, mock data, missing validation, etc.>
```

If all flows are done -> suggest: "MVP breathes. Run `/monke-flash:pulse` to smoke-test and get feedback."

If flows remain -> suggest re-running blitz with the specific flow: `/monke-flash:blitz <flow>`

---

## Anti-Patterns

| If tempted to... | Do instead... |
|------------------|--------------|
| Add error handling for edge cases | Don't. That's recon's job. |
| Write tests | Don't. Pulse will smoke-test. Recon adds real tests. |
| Refactor for cleanliness | Don't. Ship ugly, ship fast. |
| Add auth/permissions | Hardcode a user. Move on. |
| Build a settings/config system | Use env vars or hardcoded defaults. |
| Set up CI/CD | Just run it locally. |

---

## Status Update

On completion, update `monke-status.md` Flash section:
```
## Flash
Phase: **Blitz <complete|in-progress>**
Brief: `monke-docs/flash/flash-brief.md`
Scope: `monke-docs/flash/flash-scope.md`
Arch: `monke-docs/flash/flash-arch.md`
Built: <list of working flows>
Blocked: <list of blocked items, or "none">
Next: `/monke-flash:pulse` or `/monke-flash:blitz <remaining>`
```
Bump `Updated:` to today, `by /monke-flash:blitz`
