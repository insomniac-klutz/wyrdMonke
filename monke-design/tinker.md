# WyrdMonke Design:Tinker — Fill Project Template

> **Usage:** Run `/monke-design:tinker` inside a project scaffolded by `/monke-init`. It detects your stack, fills every placeholder, and gets your SDLC dashboard running.

---

## Arguments

`$ARGUMENTS` parsing:
- None. This skill takes no arguments.

---

## Prerequisites

Verify scaffolding exists (run `/monke-init` first if missing):

- `monke-docs/project-specs.md` exists
- `CLAUDE.md` exists (merged or renamed by `/monke-init`)
- `monke-mermaid.mmd` exists
- `monke-status.md` exists (the dashboard, not the skill)

If any are missing → tell user: "Project not scaffolded. Run `/monke-init` first." Stop.

---

## Phase 1: Detect Project Context

Determine if this is an **existing codebase** or a **fresh init**. Check for:

| Signal | What to look for |
|--------|-----------------|
| Language/runtime | `package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, `*.sln`, `*.csproj`, `pom.xml`, `build.gradle` |
| Source dirs | `src/`, `lib/`, `app/`, `cmd/`, `internal/`, `pkg/` |
| Test dirs | `tests/`, `test/`, `__tests__/`, `spec/`, `*_test.go` patterns |
| Existing CLAUDE.md | `CLAUDE.md` at project root |
| CI/CD | `.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`, `.circleci/` |
| Env files | `.env`, `.env.example`, `.env.local`, `.env.development` |
| Linter/formatter | `.eslintrc*`, `ruff.toml`, `pyproject.toml [tool.ruff]`, `rustfmt.toml`, `.prettierrc*`, `biome.json` |
| Database | `migrations/`, `alembic.ini`, `prisma/schema.prisma`, `diesel.toml` |

Store your findings as a detection summary.

- **Existing codebase:** Tell the user what you detected, confirm it matches expectations.
- **Fresh init:** Tell the user you'll ask for each value directly.

### Agent Teams Detection

Check if Claude Code Agent Teams are available:

1. Look for `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in environment or `.claude/settings.json`
2. Check Claude Code version `>= v2.1.32` (`claude --version`)
3. Check if `tmux` is available (`which tmux`)

| Result | Action |
|--------|--------|
| All three present | Agent Teams enabled. Tell user: "HLD/LLD design will use Architect+Critic / Designer+Reviewer teams." |
| Flag set but tmux missing | Teams available, tmux recommended. |
| Flag not set | Teams not available. Single-session mode with subagents. |

Store agent teams status — later skills reference it.

**⏸ Present detection summary to user.**

---

## Phase 2: Fill Project Specs

Hand off to `/monke-implement:fill` to walk through all 8 placeholder groups in `monke-docs/project-specs.md`.

Tell the user:

> "Now filling project-specs.md — this binds the abstract specs to your concrete stack. Running `/monke-implement:fill all`."

Then follow the instructions in `/monke-implement:fill` inline (the user may not have that skill installed separately during initial bootstrap). Pass the detection summary from Phase 1 as context for suggesting values.

**⏸ Confirm** all groups filled before proceeding.

---

## Phase 3: Fill `CLAUDE.md`

### `<<<project_description>>>`

- **Existing codebase:** Read `README.md`, manifest description fields, top-level source structure. Draft 2-3 sentences covering: what the project does, primary tech stack, deployment target. Present for confirmation.
- **Fresh init:** Ask the user to describe their project in 2-3 sentences.

### `<<<project_invariants>>>`

- **Existing codebase:** Analyze for patterns — auth/middleware, state management, error handling, database access. Present as bullet list for confirmation.
- **Fresh init:** Ask for known invariants, constraints, or gotchas. Suggest they can add more later.

**⏸ Confirm** the filled `CLAUDE.md` with the user.

---

## Phase 4: Finalize

1. Verify no `<<<placeholder>>>` patterns remain:
   ```bash
   grep -r '<<<' monke-docs/project-specs.md CLAUDE.md 2>/dev/null
   ```
   Surface any remaining to user.

2. Update `monke-mermaid.mmd` if any file paths changed during bootstrap.

3. Initialize `monke-status.md`:
   - Set project name from project-specs
   - Mark bootstrap checkboxes as complete
   - Set "Where We Are" to: `Phase: **Bootstrap complete — ready for HLD**`
   - Set "Next action" based on codebase type:
     - Existing code → `/monke-design:recon`
     - Fresh init → `/monke-design:hld`

4. Show summary:
   - Files modified
   - Placeholders filled
   - Agent Teams status

5. Suggest next steps:
   - "Read `monke-docs/sdlc-specs.md` for the end-to-end workflow."
   - "Run `/monke-status:status` to see your project dashboard."
   - If existing code: "Run `/monke-design:recon` to reverse-engineer an HLD."
   - If fresh: "Run `/monke-design:hld` to create your HLD."
   - "`git add monke-docs/ CLAUDE.md monke-mermaid.mmd monke-status.md && git commit -m 'Bootstrap WyrdMonke SDLC templates'`"

---

## Status Update

On completion, update `monke-status.md`:
- Mark all Bootstrap checkboxes as `[x]`
- Set `Updated:` to today's date, `by /monke-design:tinker`
- Set "Where We Are" and "Next action" per Phase 4 step 3
