# WyrdMonke Flash:Blitz — One Lane, Full Throttle

> **Usage:** /monke-flash:blitz [component]
>
> Code first, clean later. Flows get built, shortcuts get logged, the MVP gets legs — monke doesn't polish what isn't proven.

---

## Arguments

`$ARGUMENTS` parsing:
- First positional: component or flow name to focus on, or `all` (default)
- Second positional: `resume:<N>` — skip to phase `<N>` with checkpoint re-read (see drafter §7). Phase numbers: 1=blueprint load, 2=scaffold, 3=build flows, 4=status check.
- Example: `/monke-flash:blitz auth` or `/monke-flash:blitz all resume:3`

```
TARGET="${1:-all}"
RESUME_PHASE="$(echo "$2" | grep -oE '^resume:[0-9]+$' | cut -d: -f2)"
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

- `monke-docs/flash/flash-arch.md` exists (run `/monke-flash:sketch` first)
- `monke-docs/flash/flash-scope.md` exists

---

## Gate Semantics

Flash runs under light rigor per S9.4/S12. HARD gates (PG-1 in scope, PG-11 in snap) always surface. SOFT gates auto-pass per the adaptive system. TRIGGERED gates fire on their triggers regardless of rigor.

⏸ **PG-12 [TRIGGERED] — Stack violation during implementation.** Fires when the code being written drifts from the stack locked in `flash-arch.md` (unplanned framework, new database, new LLM provider). Surfaces regardless of rigor. Resolution: either update `flash-arch.md` with the new choice + why, or revert the drift. Never silently absorb a stack change.

---

## Phase 1: Load the Blueprint

**Resume check (per drafter §7).** If arguments contain `resume:<N>`:
1. Read the companion lock-file `monke-docs/flash/flash-blitz.md.lock`. Lock-file is authoritative — if it's inconsistent with `monke-status.md` Flash section, regenerate from scratch.
2. Verify lock-file `skill:` field matches `flash:blitz`.
3. If parse/lock check passes → skip to Phase `<N>` with prerequisites re-validated inline.
4. If parse/lock check fails → emit warning "resume:<N> specified but checkpoint invalid" and proceed from Phase 1 normally.
5. See Context Death Protocol below for the full recovery spec.

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
- If yes -> proceed to PG-12 stack-drift check, then commit and move to next flow
- If no -> fix the minimum to unblock, then verify again

### 3.3 PG-12 stack-drift check

Before committing any flow's code, parse imports/requires in the files being added or modified. Cross-check each import against the locked stack in `monke-docs/flash/flash-arch.md` (the flash chain's equivalent of `project-specs.md` §10.1). If any import names a framework/library/runtime NOT in the locked stack → auto-fire:

⏸ **PG-12 [TRIGGERED] — Stack violation detected.** Evidence: `<file>:<line> imports <framework> not in locked flash-arch stack (locked: <locked-list>)`. Options: (a) Amend `flash-arch.md` (may trigger pulse re-iteration). (b) Rewrite the import to a locked equivalent. (c) Abort this flow commit.

Ignores rigor. Always surfaces.

Import parsing: simple line-match — Python (`^import `, `^from `), TypeScript/JS (`^import `, `require(`), Rust (`^use `), Go (`^import `).

### 3.4 Commit

Format: `flash : <what works now>`

Examples:
- `flash : users can submit a form`
- `flash : search returns results from seeded data`
- `flash : CLI parses input and generates output`

**Write lock-file.** After each flow commits, update `monke-docs/flash/flash-blitz.md.lock` per drafter §7 — capture `phase: 3`, `progress: <current-flow-index>/<total-flows>`, timestamp. This persists per-flow progress across context death within Phase 3.

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

**Delete lock-file.** When blitz finalizes (all targeted flows committed and status pushed to `monke-status.md`), delete `monke-docs/flash/flash-blitz.md.lock`. Stale lock-files confuse future recovery.

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Add error handling for edge cases | Refuse. That's recon's job. Happy path only — every catch you write now is a catch recon has to justify later. |
| Write unit tests during blitz | Refuse. Pulse smoke-tests. Recon adds real tests once architecture has hardened. |
| Refactor for cleanliness mid-blitz | Refuse. Ship ugly, ship fast. Refactor pressure means the flow wasn't cut narrow enough — revisit scope. |
| Add auth/permissions/admin | Refuse. Hardcode a user. Auth is a flow, not infrastructure — it lands in recon if it lands at all. |
| Build a settings/config system | Refuse. Env vars or hardcoded defaults. Config systems are a quarterly commitment, not a blitz artifact. |
| Set up CI/CD during blitz | Refuse. Run it locally. CI/CD is an R5 concern. |
| Swap stacks silently mid-blitz | Refuse. That's PG-12 [TRIGGERED] — surface the drift, resolve the arch doc, then continue. |

---

## Context Death Protocol

**Recovery source of truth.** The canonical recovery state for the flash chain is the Flash section of `monke-status.md`. Only this file is read on re-entry to decide where blitz is in its per-flow cycle. Intermediate artifacts (`monke-docs/flash/flash-brief.md`, `flash-scope.md`, `flash-arch.md`, and any `flash-<slug>.md` per-flow files) are human-readable trails and NOT checkpoints — they are not sufficient to reconstruct the skill's state alone. If `monke-status.md` is missing or incomplete after context death, recovery MUST prompt the user rather than infer from artifacts.

**Checkpoint artifacts:** the per-flow status table in Phase 4 is rewritten to `monke-status.md` Flash section after each flow commits. `monke-docs/flash/flash-blitz.md.lock` (phase + progress ledger per drafter §7) tracks per-flow progress. Commits (`flash : <what works now>`) serve as a human-readable trail but are NOT the recovery checkpoint — `monke-status.md` is.
**Status line marker:** `Where We Are: flash:blitz — flow <X>/<Y>` while mid-flight.
**Recovery detection:** On re-entry, read `monke-status.md` Flash section. Cross-check against the lock-file if present (authoritative — if inconsistent with the status file, regenerate the lock from status). If status names a flow mid-flight → resume at that flow. If it shows blitz complete → move forward per `Next:`. If `monke-status.md` is missing or its Flash section is empty → prompt the user rather than infer from commit history or scaffold files.

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
