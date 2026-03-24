# WyrdMonke Recon:Survey — What do we actually have?

> **Usage:** Copy `monke-recon/` to `.claude/commands/monke-recon/`. Invoke: `/monke-recon:survey [scope]`

---

## Arguments

`$ARGUMENTS` parsing:
- Single positional: scope limiter (subdirectory, component name, or glob pattern)
- Default (empty): analyze entire project
- Example: `/monke-recon:survey src/api` or `/monke-recon:survey "*.rs"`

```
SCOPE="${ARGUMENTS:-}"
```

---

## Prerequisites

Read `monke-status.md` (if it exists) for context. Then check for source material:

**Source detection determines mode. Two paths:**

| Signal | Mode |
|--------|------|
| `monke-docs/flash/flash-manifest.md` exists | **Flash mode** — MVP built by `/monke-flash:snap`. Manifest is primary input. |
| Source code exists without flash artifacts | **Archaeology mode** — pure reverse-engineering from existing code. |
| Neither — no code, no manifest | **Nothing to survey.** Tell user: "Build something first. Try `/monke-flash:snap` or write some code." Stop. |

Also verify:
- `monke-docs/design-specs.md` exists (needed downstream by `/monke-recon:reconstruct`)
- `monke-docs/project-specs.md` exists with stack locked (no `<<<` remaining)

If design-specs or project-specs missing → warn but don't block. Survey is about what IS, not what SHOULD BE. The reconstruct step will need them.

---

## Phase 1: Source Detection

Determine the survey mode.

1. Check for `monke-docs/flash/flash-manifest.md`.
   - **Found** → flash mode. Read the manifest. This is your primary source of truth for what was planned vs what was built.
   - **Not found** → check for source files (any `.ts`, `.rs`, `.py`, `.go`, `.java`, `.js`, `.tsx`, `.jsx`, etc.)
     - **Found** → archaeology mode. You're digging through ruins. No map, just a shovel.
     - **Not found** → stop. Nothing to survey.

2. If flash mode: also read `monke-docs/flash/flash-arch.md` (if it exists) for the original architecture plan.

3. Create output directory: `monke-docs/recon/` (if it doesn't exist).

**Log the mode.** Every subsequent phase behaves slightly differently based on flash vs archaeology.

---

## Phase 2: Stack Scan

Read every manifest file in scope. Catalog the actual technology stack.

| What | How to find it |
|------|---------------|
| Languages | File extensions across the project. Count files per language. |
| Frameworks | Import statements, config files (`next.config.*`, `vite.config.*`, `angular.json`, etc.) |
| Package managers | `package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, `pom.xml`, `build.gradle` |
| Databases | Migration files, ORM config, connection strings, schema files |
| External services | HTTP client calls, SDK imports, API key references, `.env` files |
| LLM usage | LLM client imports (`openai`, `anthropic`, `langchain`, AI SDK patterns), API key refs |
| Build tools | `tsconfig.json`, `Makefile`, `justfile`, `Dockerfile`, `docker-compose.yml` |
| CI/CD | `.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile` |

**If flash mode:** Compare as-built stack against `flash-arch.md` plans. Note drift — anything planned but not used, or used but not planned.

**Output format:**
```
### Stack
| Layer | Planned (flash) | Actual | Drift |
|-------|-----------------|--------|-------|
```

For archaeology mode, skip the "Planned" and "Drift" columns.

---

## Phase 3: Structure Scan

Map the file tree and identify architectural patterns.

### 3.1 File Tree Analysis

Read the directory structure. Identify patterns:
- Flat vs nested organization
- Feature-based vs layer-based grouping
- Monorepo vs single-package
- Convention signals (`src/`, `lib/`, `app/`, `pkg/`, `internal/`, `cmd/`)

### 3.2 Container Candidates

Identify separately deployable units:

| Signal | Likely container |
|--------|-----------------|
| Separate manifest file | Independent package/service |
| `Dockerfile` or compose service | Deployable container |
| Separate entry point (`main.*`, `index.*`, `app.*`) | Distinct runtime |
| `/api`, `/web`, `/worker`, `/cli` directories | Container per concern |
| Monorepo workspace members | One container per workspace |

For each: **name, tech, entry point, responsibility (one sentence).**

### 3.3 Component Candidates

Within each container, identify module boundaries:

| Signal | Likely component |
|--------|-----------------|
| Directory with barrel file (`index.ts`, `mod.rs`, `__init__.py`) | Module boundary |
| Exported types/interfaces | Boundary contract |
| Route handlers / controllers | API surface |
| Service classes / use-case modules | Business logic |
| Repository / data-access layers | Persistence |
| Agent / LLM orchestration code | Agentic component |
| Shared types / models directory | Cross-cutting types (Layer 0) |

For each: **name, responsibility, exports, imports, `traditional` or `agentic` tag.**

### 3.4 Entry Points & Data Access

- Map all entry points (HTTP routes, CLI commands, event handlers, scheduled jobs)
- Map all data access patterns (direct DB queries, ORM calls, API client calls)
- For agentic code: identify LLM calls, agent loops, tool definitions, memory access

**If flash mode:** Note what was built vs what was planned but cut. Every cut is a gap signal.

---

## Phase 4: Dependency Scan

### 4.1 Internal Dependencies

Map which modules import which. Build the dependency graph:
- Direction: consumer → provider
- Flag circular dependencies (these are always a smell)
- Identify god modules (imported by everything)
- Identify orphan modules (imported by nothing)

### 4.2 External Dependencies

- List all third-party packages with versions
- Identify vendored vs registry dependencies
- Flag outdated or deprecated packages (if detectable)
- List all external API calls and their targets
- Note any pinned vs floating version strategies

---

## Phase 5: Test Inventory

Catalog what testing exists.

| Dimension | Check |
|-----------|-------|
| Test files | Glob for `test_*`, `*.test.*`, `*.spec.*`, `*_test.*`, `tests/` dirs |
| Test types | Classify: unit, integration, system/e2e by directory or naming |
| Coverage config | `.nycrc`, `jest.config` coverage section, `tarpaulin.toml`, etc. |
| Fixtures | Test data files, factories, builders |
| Mock patterns | Mock libraries in use, custom mock utilities |
| Test infrastructure | Test helpers, setup/teardown utilities, test databases |

**Critical question: what's NOT tested?** Cross-reference discovered components against test files. Components with zero test coverage are the biggest gap signal.

---

## Phase 6: Quality Snapshot

Quick health check on code quality practices.

| Dimension | What to look for |
|-----------|-----------------|
| Linting | Config files (`.eslintrc`, `clippy.toml`, `ruff.toml`, `biome.json`). Is it strict? |
| Type safety | TypeScript `strict: true`? Rust (inherently strict)? Python type hints? `any` count? |
| Error handling | Result types vs exceptions. Bare catches. Swallowed errors. Panics in non-panic code. |
| Code smells | God files (>500 lines). Deep nesting (>4 levels). Huge functions (>50 lines). |
| Secrets | Anything in `.env` committed? Hardcoded API keys? `.gitignore` coverage? |
| Documentation | README exists? API docs? Inline comments quality? |

**Be honest.** Don't sugarcoat. "This error handling is a dumpster fire" is more useful than "error handling could be improved."

---

## Output

Write `monke-docs/recon/recon-survey.md` with the full inventory.

**Structure:**
```markdown
# Recon Survey
Project: <name> | Date: <today> | Mode: flash / archaeology | Scope: <scope or "full">

## Stack
<table from Phase 2>

## Structure
### Containers
<list from Phase 3.2>

### Components
<per-container list from Phase 3.3>

### Entry Points & Data Access
<from Phase 3.4>

## Dependencies
### Internal
<graph summary from Phase 4.1 — flag circulars>

### External
<package list from Phase 4.2>

## Tests
<inventory from Phase 5 — emphasize what's NOT tested>

## Quality
<snapshot from Phase 6 — be direct>

## Flash Drift (flash mode only)
<planned vs built comparison>
```

**Aim for 150-200 lines.** Dense, factual, no filler.

---

## Decision Gate

⏸ **Present survey summary to user.**

Show: container count, component count, test coverage (rough), quality rating (green/yellow/red per dimension), and top 3 concerns.

Confirm findings match user's understanding. If user corrects something → update the survey before proceeding.

---

## Key Behaviors

- **Read every file in scope.** No sampling, no guessing. If the codebase is large (>20 files), use agent teams for parallel scanning — split by directory or container.
- **Be honest about quality.** This is a medical exam, not a pep talk.
- **If flash mode:** Explicitly note what was built vs what was planned but cut. Each cut is future work.
- **Don't prescribe fixes.** That's the job of `/monke-recon:gaps` and `/monke-recon:roadmap`. Survey just catalogs what IS.
- **Dependency direction matters.** Always note consumer → provider, never just "these are related."
- **Agentic code gets special attention.** LLM calls, agent loops, tool definitions — these are architecturally significant and often the most fragile.

---

## Status Update

On completion, update `monke-status.md`:
- Add a **Recon** section (if not present) after Bootstrap:
  ```
  ## Recon
  - [x] Survey — <date>
  - [ ] Reconstruct (HLD)
  - [ ] Reconstruct (LLDs)
  - [ ] Gap analysis
  - [ ] Open questions
  - [ ] Roadmap
  ```
- Set `Updated:` to today, `by /monke-recon:survey`
- Update "Where We Are": `Phase: **Survey complete — ready for reconstruction**`
- Update "Next action": `/monke-recon:reconstruct`
