# WyrdMonke — The Switchboard

> **Usage:** `/monke [rigor] [override]`
>
> One command. Monke reads the jungle, names what it sees, and points at the ripest banana. No seven orchestras. No five-step recon dance. One decision tree that handles greenfield, existing code, mid-flight projects, and everything in between.

---

## Arguments

`$ARGUMENTS` parsing:
- **First positional (optional):** rigor selector — `light` | `standard` | `thorough` | a container name | a skill override (e.g. `rage:buggy`, `recon`, `flash`, `intake`) | **a natural-language feature request in quotes** (dispatches to `/monke-intake`).
- **Second positional (optional):** override target — passed through to the overridden skill (e.g. `rage:buggy src/api`).
- **No args:** auto-detect everything from project state + `.monke-config.md`.
- **Quoted request shortcut:** `/monke "add webhook retry with exponential backoff"` → routes straight to `/monke-intake` with the request string as payload. Intake becomes the permanent lead for that feature thread; `monke.md` does not resume control until intake closes Phase 6.

```
ARG1="${1:-}"
ARG2="${2:-}"

# Parse ARG1
case "$ARG1" in
  light|standard|thorough)
    RIGOR="$ARG1"
    OVERRIDE=""
    ;;
  "")
    RIGOR=""       # read from .monke-config.md
    OVERRIDE=""
    ;;
  recon|flash|\
  rage|rage:buggy|rage:improv|rage:renounce|rage:haunt|rage:drift|rage:echo|\
  design|design:tinker|design:adr|design:oq|design:hld|design:lld|\
  implement|implement:fill|implement:implement|implement:checkpoint|\
  test|test:test-plan|test:test-run|test:coverage|\
  seer|seer:profile|seer:experiment|seer:registry|seer:agentify|\
  ops|ops:commit|\
  status|status:show|status:rebuild|status:next|status:blocked|status:resume|\
  intake)
    RIGOR=""
    OVERRIDE="$ARG1"
    OVERRIDE_ARGS="${ARG2:-}"
    ;;
  *)
    # Natural-language feature request? (quoted, or >3 words, or doesn't match any other pattern)
    # Route to /monke-intake with the full string as request payload.
    WORD_COUNT=$(echo "$ARG1" | wc -w)
    if [[ "$ARG1" == \"*\" ]] || [ "$WORD_COUNT" -gt 3 ]; then
      RIGOR=""
      OVERRIDE="intake"
      OVERRIDE_ARGS="$ARG1"
    else
      # Single short token that isn't rigor/skill-override.
      # Check if it matches a known container from monke-status.md Components table BEFORE
      # treating as scope — otherwise we silent-route a mis-quoted feature request.
      if [ -f monke-status.md ] && grep -qE "^\| *${ARG1}[[:space:]]*\|" monke-status.md; then
        # Known container → treat as scope
        RIGOR=""
        CONTAINER_SCOPE="$ARG1"
      else
        # Unknown short token. Likely a mis-quoted request. Emit hint and stop.
        cat <<EOF
WHAT: /monke received a single-word argument "$ARG1" that isn't a rigor level, skill override, or known container.
WHY:  Either you meant a natural-language feature request (needs quotes) or a container scope (not found in monke-status.md).
HOW:  If feature request: /monke "$ARG1 <finish the sentence>"
      If container scope: /monke-status:status show     # list known containers
      If skill override:  /monke --help                 # show valid skill names
EOF
        exit 0
      fi
    fi
    ;;
esac
```

Rules:
- If rigor is given → persist to `.monke-config.md` (overwriting the `rigor:` line) and announce the change.
- If override is given → skip state detection, jump straight to the named skill with `OVERRIDE_ARGS`.
- If container scope is given → limit Phase 3 routing to components within that container.
- No silent mode. Every invocation announces the detected state and the chosen route.

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**.

```
WHAT: Agent Teams Gate failed. `CLAUDE.md` exists but has no Agent Teams section.
WHY:  The unified orchestrator dispatches work to sub-skills that spawn agent teams.
      Without enablement, teammates cannot coordinate and parallel routing breaks.
HOW:  Run `/monke-sync` to pull the latest template, OR copy the Agent Teams
      section from `monke-CLAUDE.md` into your project's `CLAUDE.md`, then re-run `/monke`.
```

- `monke-docs/` exists. If missing → "Run `/monke-init` first. Can't orchestrate what hasn't been bootstrapped."
- `.monke-config.md` exists. If missing and no rigor arg given → treat as first run, prompt for rigor (HARD gate per design-specs S12.3 default escalation).

If any prerequisite fails → stop with WHAT/WHY/HOW. No guessing.

---

## Phase 1: Project State Detection

Read the signals. Decide which branch of the tree applies. One detection pass, one branch — never two.

```
Signals read (in order):
  1. monke-status.md exists?
  2. monke-docs/flash/flash-brief.md exists? (flash in progress)
  3. Source code exists? (src/, lib/, app/, or any language manifest)
  4. OVERRIDE arg set?
```

| Signal pattern | Branch |
|---------------|--------|
| `OVERRIDE` arg set | **Override** — dispatch to named skill, skip Phase 2/3 |
| Flash brief exists, no manifest | **Resume flash** — jump to flash pipeline at detected phase |
| `monke-status.md` exists, populated | **Resume** — Phase 3 (per-component routing from status) |
| No status, source code exists | **Fast Onboarding** — Phase 2 |
| No status, no code | **Greenfield** — ask: flash MVP or production build? |

**Greenfield branch (interactive HARD gate — this IS PG-1 scope):**

```
⏸ PG-1 [HARD] — Project scope.
No code detected. Which path?
  (a) flash   — MVP-first. Minimum viable, happy paths only. Flash pipeline.
  (b) production — Design-first. HLD → LLD → code. Design pipeline.
  (c) other — describe and monke will route.
```

- `flash` → dispatch the flash chain: `/monke-flash:spark` → scope → sketch → blitz → pulse → snap. Each skill hands back here; `/monke` presents the next one. Return to Phase 3 after snap completes.
- `production` → hand off to `/monke-design:hld L1` and then continue through Phase 3 routing as LLDs come online.
- `other` → read the user's description, match against known patterns, propose a route, confirm before dispatch.

**Override branch:**

Skip Phase 2, skip state detection. Dispatch by exact override string. Chains (recon, flash, rage) preserve their multi-step sequencing; single-skill overrides dispatch once and return.

Before dispatch, resolve rigor from `.monke-config.md` if not already set from ARG1, then export it. Rigor is exported so downstream skills can read it via `${MONKE_RIGOR}` (see drafter §5 rigor-read idiom). Every dispatched skill inherits the env var.

The Resume parser extracts both `Phase:` and `Skill:` from the Resume block. When `Skill:` names a specific sub-skill (e.g. `/monke-intake` or `/monke-design:lld`), dispatch jumps directly to that skill with `resume:<N>` appended — `monke.md` does NOT re-run Phase 1 state detection. This is how context-death mid-intake recovers without losing the feature thread: the thread ID lives in the dispatched skill's thread file, not in `monke.md`'s memory.

```bash
# Resolve rigor before dispatch (ARG1 wins; else .monke-config.md)
if [ -z "$RIGOR" ]; then
  RIGOR="$(grep '^rigor:' .monke-config.md | awk '{print $2}')"
fi
export MONKE_RIGOR="$RIGOR"

# If Resume block exists in monke-status.md, extract Phase + Skill and dispatch accordingly
if grep -q "^## Resume$" monke-status.md 2>/dev/null || grep -q "^Resume$" monke-status.md; then
  RESUME_BLOCK="$(awk '/^## Resume$|^Resume$/,/^$/' monke-status.md)"
  RESUME_PHASE="$(echo "$RESUME_BLOCK" | grep -oE 'Phase: [0-9]+' | head -1 | awk '{print $2}')"
  RESUME_SKILL="$(echo "$RESUME_BLOCK" | grep -oE 'Skill: /monke-[a-z]+(:[a-z-]+)?' | head -1 | sed 's/^Skill: //')"

  # If Skill is named AND user didn't already pass an override, jump straight to that skill.
  # Map slash command back to override keyword (e.g. /monke-intake → intake, /monke-design:lld → design:lld).
  if [ -n "$RESUME_SKILL" ] && [ -z "$OVERRIDE" ]; then
    OVERRIDE="$(echo "$RESUME_SKILL" | sed 's|^/monke-||; s|^/monke$|intake|')"
  fi

  # Append resume:<N> to args for the dispatched skill
  [ -n "$RESUME_PHASE" ] && OVERRIDE_ARGS="${OVERRIDE_ARGS} resume:$RESUME_PHASE"
fi

case "$OVERRIDE" in
  # --- Chains (multi-step, /monke presents next after each returns) ---
  recon)                 dispatch /monke-recon:survey "$OVERRIDE_ARGS" ;;
  flash)                 dispatch /monke-flash:spark  "$OVERRIDE_ARGS" ;;
  rage|rage:buggy)       dispatch /monke-rage:buggy    "$OVERRIDE_ARGS" ;;
  rage:improv)           dispatch /monke-rage:improv   "$OVERRIDE_ARGS" ;;
  rage:renounce)         dispatch /monke-rage:renounce "$OVERRIDE_ARGS" ;;
  rage:haunt)            dispatch /monke-rage:haunt    "$OVERRIDE_ARGS" ;;
  rage:drift)            dispatch /monke-rage:drift    "$OVERRIDE_ARGS" ;;
  rage:echo)             dispatch /monke-rage:echo     "$OVERRIDE_ARGS" ;;

  # --- Design (single skill per call; design bare = hld default) ---
  design|design:hld)     dispatch /monke-design:hld    "$OVERRIDE_ARGS" ;;
  design:lld)            dispatch /monke-design:lld    "$OVERRIDE_ARGS" ;;
  design:adr)            dispatch /monke-design:adr    "$OVERRIDE_ARGS" ;;
  design:oq)             dispatch /monke-design:oq     "$OVERRIDE_ARGS" ;;
  design:tinker)         dispatch /monke-design:tinker "$OVERRIDE_ARGS" ;;

  # --- Implement (implement bare = full pipeline driver) ---
  implement|implement:implement) dispatch /monke-implement:implement "$OVERRIDE_ARGS" ;;
  implement:fill)        dispatch /monke-implement:fill       "$OVERRIDE_ARGS" ;;
  implement:checkpoint)  dispatch /monke-implement:checkpoint "$OVERRIDE_ARGS" ;;

  # --- Test (test bare = test-run default) ---
  test|test:test-run)    dispatch /monke-test:test-run  "$OVERRIDE_ARGS" ;;
  test:test-plan)        dispatch /monke-test:test-plan "$OVERRIDE_ARGS" ;;
  test:coverage)         dispatch /monke-test:coverage  "$OVERRIDE_ARGS" ;;

  # --- Seer (seer bare = profile default) ---
  seer|seer:profile)     dispatch /monke-seer:profile    "$OVERRIDE_ARGS" ;;
  seer:experiment)       dispatch /monke-seer:experiment "$OVERRIDE_ARGS" ;;
  seer:registry)         dispatch /monke-seer:registry   "$OVERRIDE_ARGS" ;;
  seer:agentify)         dispatch /monke-seer:agentify   "$OVERRIDE_ARGS" ;;

  # --- Ops (ops bare = commit default) ---
  ops|ops:commit)        dispatch /monke-ops:commit "$OVERRIDE_ARGS" ;;

  # --- Intake (natural-language request routing; becomes permanent lead) ---
  intake)                dispatch /monke-intake "$OVERRIDE_ARGS" ;;

  # --- Status (status bare = show; forwards the named action) ---
  status|status:show)    dispatch /monke-status:status show    ;;
  status:rebuild)        dispatch /monke-status:status rebuild ;;
  status:next)           dispatch /monke-status:status next    ;;
  status:blocked)        dispatch /monke-status:status blocked ;;
  status:resume)         dispatch /monke-status:status resume  ;;
esac
```

Notes on behavior:
- **Chains** (`recon`, `flash`, `rage*`): `/monke` presents the next skill in the chain after each returns. `recon` runs `survey` → `reconstruct` → `gaps` → `oqs` → `roadmap` (opt-in deep path per sdlc-specs §5). `flash` runs `spark` → `scope` → `sketch` → `blitz` → `pulse` → `snap`. `rage:<mode>` dispatches one mode and returns (no chain).
- **Single-skill overrides** (`design:*`, `implement:*`, `test:*`, `seer:*`, `ops:*`, `status:*`): dispatch once, return to Phase 1 state detection. User re-invokes `/monke` (or another override) for the next step.
- **`implement:implement`** is the full pipeline driver (walks Layers 0–3 for one component). `implement:fill` fills `project-specs.md` and is typically called indirectly by fast onboarding — exposed here for manual re-run. `implement:checkpoint` formalizes PG-11 phase checkpoints.
- **`design:tinker`** is primarily invoked via `/monke-init` during bootstrap (it fills `project-specs.md` + `CLAUDE.md` placeholders). Exposed under `/monke` for re-runs after manual edits.
- **`status:*`** actions map 1:1 to `/monke-status:status <action>` — `show`/`rebuild`/`next`/`blocked`/`resume`. No chain.
- **`intake`** is the natural-language intake skill. Invoked directly (`/monke intake "<request>"`) or via shortcut when ARG1 is a quoted/multi-word string (`/monke "<request>"`). Intake becomes the permanent lead for the entire feature thread — once dispatched, `monke.md` does not resume routing until the thread closes. Sub-skills run as workers under intake's coordination.

After override completes, return to Phase 1 state detection for the next loop unless the user says "done".

**Resume flash branch:**

A flash brief exists but no manifest → re-enter the flash chain at the first incomplete step. Detect prior artifacts (`flash-brief.md`, `flash-scope.md`, `flash-arch.md`, any blitz code) and dispatch to the next uncompleted flash skill. Return to Phase 3 after snap.

**Resume branch (status file present):**

Read `monke-status.md`. Extract:
- `Rigor:` line (reconcile with `.monke-config.md`).
- `Where We Are:` line.
- `Components` table → per-component current maturity + LLD confidence + override flag.
- `Gate Audit Log` tail (this session's gate outcomes).
- `Phase Checkpoints` table → current phase pointer.
- Any `Resume` block (Context Death Protocol checkpoint from a prior invocation that died).

Go to Phase 3.

---

## Phase 2: Fast Onboarding

Runs once per project. Code exists, status does not. Walk through Steps 2.1–2.5 in order. Only Step 2.5 has a human gate.

### Step 2.1: Auto-Detect Stack (no gate)

Scan the project root and subdirectories for manifest files. Extract: language(s), framework(s), test runner(s), build tool(s), package manager(s), CI config, existing docs.

| Signal file | Extracts |
|------------|----------|
| `package.json` + `pnpm-lock.yaml` / `yarn.lock` / `package-lock.json` | JS/TS, package manager, scripts, deps |
| `pyproject.toml` + `poetry.lock` / `uv.lock` / `requirements.txt` | Python, package manager, test runner, type-check tool |
| `Cargo.toml` + `Cargo.lock` | Rust, cargo, workspace members |
| `go.mod` + `go.sum` | Go modules |
| `pom.xml` / `build.gradle` / `build.gradle.kts` | Java/Kotlin, Maven/Gradle |
| `Gemfile` + `Gemfile.lock` | Ruby |
| `composer.json` | PHP |
| `*.csproj` + `*.sln` | C#/.NET |
| `Dockerfile`, `docker-compose.yml`, `.dockerignore` | Containerization — candidate container boundaries |
| `.github/workflows/*.yml` | CI jobs → extract `test`, `lint`, `build` commands for IL gate bindings |
| `.gitlab-ci.yml` / `Jenkinsfile` / `.circleci/config.yml` | Alternative CI — extract same |
| `openapi.yaml` / `openapi.json` / `swagger.json` | Existing API contract → boundary matrix input |
| `docs/decisions/` or `docs/adr/` (markdown files) | Existing ADRs → preserve, reference in generated HLD |
| `docs/` or `README.md` architecture sections | Existing architecture docs → HLD S1 seed |
| `tsconfig.json`, `mypy.ini`, `pyrightconfig.json`, `.rubocop.yml` | Type/lint config → IL-0 command bindings |

Write findings into `monke-docs/project-specs.md` (populate the per-container tables with autodetected values; leave `<<<placeholder>>>` where scan cannot decide).

### Step 2.2: Auto-Map Structure (no gate)

Identify **containers** (deployable units) and **components** (modules within containers).

Container signals:
- Entry points: `main.*`, `index.*`, `app.*`, `cmd/*/main.go`, `bin/*`, `src/bin/*.rs`.
- Dockerfiles → one container per Dockerfile (path to Dockerfile = container identity).
- Docker Compose `services:` map → one container per service.
- Monorepo workspaces: `pnpm-workspace.yaml`, `Cargo.toml [workspace] members`, `go.work`, `nx.json`, `turbo.json`, `lerna.json`, `package.json workspaces`.
- If no container signals → treat the whole project as one container named after the root dir.

Component signals (per container):
- Barrel files: `index.ts`, `mod.rs`, `__init__.py` re-exports.
- Top-level directories under `src/`, `lib/`, `app/`, `internal/`, `pkg/` containing their own typed exports.
- Public APIs declared in `package.json#exports` or equivalent manifests.

Build an import graph per container:
- Parse imports/uses/requires per file.
- Collapse to module-level edges.
- **Detect circular dependencies** — flag cycles for presentation at the hard gate (H7 — boundary triage).

### Step 2.3: Per-Component Maturity Scan (no gate)

For each discovered component, mechanically assess its maturity level. Run the actual language toolchain (auto-detected from Step 2.1); do not infer from file existence alone.

**Language-specific IL-0 heuristics (B2 — critical):**

| Language | IL-0 passed requires |
|----------|---------------------|
| **Python** | Lint clean + imports resolve + at least one of: dataclasses / pydantic / attrs / TypedDict declarations OR type annotations on >50% of public function signatures. Bare Python with no typing hints → `pre-L0`. |
| **JavaScript (no TS)** | Lint clean + imports resolve + at least one of: JSDoc `@type` annotations on public exports OR Zod / Yup / Joi schemas. Bare JS with no runtime typing → `pre-L0`. |
| **TypeScript** | `tsc --noEmit` passes + lint clean + imports resolve. |
| **Rust** | `cargo check` passes + `cargo clippy` clean. |
| **Go** | `go vet` clean + `go build` compiles. |
| **Java / Kotlin** | Standard static compile + lint clean. |
| **Other statically typed** | Standard compile + lint clean. |

If the IL-0 heuristic returns false for a component with files present → `maturity: pre-L0` regardless of file count. Never trust "lint passes" alone for dynamically typed languages.

**IL-2 assertion + coverage check (B3 — critical):**

"Unit tests pass" is NOT sufficient for IL-2. Require BOTH:
1. Test files contain assertion patterns — scan for `assert`, `expect`, `require`, `check`, `should`, `assertTrue`, `assertEquals`, `panic_eq`, or language equivalent. Empty test bodies (`pass`, `it.todo`, `ignore!`) do NOT count.
2. Coverage > 0% on the component's source files (not just on test files themselves). Run the coverage tool detected in Step 2.1.

If tests exist but (a) contain no assertions OR (b) produce 0% source coverage → classify as IL-1, not IL-2. Empty test suites fake IL-2 and must be caught.

**Scan failure handling (B5 — critical):**

If a mechanical check command fails (command not found, missing system deps, compilation error, timeout):
- **Do NOT crash the scan.** Continue with remaining components.
- Classify the component as `maturity: unknown (scan failed: <specific error>)`.
- Record the exact command that failed and its stderr tail.
- Surface at the hard gate (Step 2.5) for user triage.

**Maturity classification table (applied per component):**

| Signal | Maturity | Pipeline Entry |
|--------|----------|----------------|
| Language-specific types + lint clean + imports resolve | `IL-0` passed | Start at Layer 1 |
| Interfaces/stubs with full types, compilable | `IL-1` passed | Start at Layer 2 |
| Impl + unit tests with assertions + coverage > 0% on source | `IL-2` passed | Start at Layer 3 |
| Integration tests present + coverage meets project threshold | `IL-3` passed | Done (route to rage scan if user wants) |
| Directory exists, empty or placeholder files only | `pre-L0` | Start at Layer 0 |
| Nothing exists for this component | `missing` | Generate LLD, then Layer 0 |
| Scan command failed | `unknown` | Manual triage at hard gate |

**Boundary confidence flag (H9):**

Tag each component's boundary with confidence when building the map:
- **HIGH** — explicit typed exports at a barrel file, clear package boundary with public API, separate crate / module / package.
- **LOW** — inferred from directory structure, no explicit exports, bare directory that may actually be multiple modules.

Surface LOW-confidence boundaries separately at the hard gate for user review.

### Step 2.4: Generate Artifacts (no gate)

Write three artifact sets, mechanically, from Steps 2.1–2.3:

**(a) `monke-status.md`** — populate from `status-template.md`:
- `Project:` line from the auto-detected root package name.
- `Rigor:` line — recommended rigor (see Step 2.5 auto-escalation). Left blank if user hasn't confirmed yet.
- `Components` table — one row per discovered component with: container, detected maturity, current layer (derived from maturity), IL gates passed list, LLD confidence (`auto-generated` for all stub LLDs), status (`auto-scanned`), override column blank.
- `Where We Are:` set to `Fast onboarding complete — awaiting user confirmation (PG-1)`.

**(b) `monke-docs/hld.md`** — auto-generated lightweight HLD:

Header:
```
# High-Level Design

Source: auto-generated via fast onboarding
Confidence: mixed (see per-section confidence flags)
Generated: <YYYY-MM-DD> by /monke
HLD Version: 0.1 (auto)
```

Sections populated:
- **S1 Context** — system name, detected languages, rough purpose from README if present.
- **S2 Containers** — one row per detected container with tech stack, auto-inferred responsibility (from directory name / Docker label / workspace member name), type tag stub (`traditional` default — flag `agentic` candidacy if versioned-artifact / data-dependent signals detected per design-specs S1.2).
- **S3 Components** — one row per discovered component with container, **confidence flag (HIGH / LOW)**, maturity from scan, pointer to stub LLD file.
- **S4 Data Flows** — `TBD — surface from code trace at PG-1`. Leave skeleton.
- **S7 Boundary Matrix** — populated where confidence is HIGH (typed contracts extractable). LOW boundaries left as `TBD`.
- **S8 Phase Plan** — `TBD — define at PG-1 if user chooses to phase; otherwise single-phase project`.

Dispatch to `/monke-design:hld auto` for the actual generation (see `monke-design/hld.md` Auto-Generated HLD Flow). This orchestrator does not duplicate HLD-writing logic.

**(c) `monke-docs/lld/<component>.md` stub per component needing work (maturity < IL-3) (B1 — critical):**

Each stub LLD MUST include the B1 minimum so the implement skill's prerequisite check passes:

```
# LLD: <component>

Parent HLD: S3.<component>  |  HLD Version: 0.1 (auto)
Confidence: auto-generated
Source: fast-onboarding scan
Backend: <detected language>
Type: traditional (stub)
Upstream: <inferred from import graph — UPSTREAM components>
Downstream: <inferred from import graph — DOWNSTREAM components>
Phase: 1 (stub — adjust at PG-1)
Test gate: pending

## File Map
| File | Layer | Exports | Depends On |
|------|-------|---------|------------|
<extracted mechanically per file in component dir>

## Signatures (extracted from code)
<actual function signatures from source files — not inferred>

## Maturity Tags (per function)
| Function | Tag | Note |
|----------|-----|------|
<as-is if present and tested, needs-work if present untested, stub if missing>

## Decomposition Tree (mechanical — from import graph)
<tree rooted at public exports, leaves at imports from outside component>
- leaves: <functions/files with no internal deps>
- middle: <functions/files with both internal callers and internal deps>
- roots: <public API surface — exported functions>

## Unit Test Plan (skeleton)
| # | Function | Input | Expected | Category | Mocks |
|---|----------|-------|----------|----------|-------|
<one row per public function with Input/Expected = TBD, Category = happy, Mocks = TBD>

## Integration Test Plan (skeleton)
| # | Boundary | Upstream Call | Expected Downstream Effect | Error Scenario | Mocks |
|---|----------|--------------|----------------------------|----------------|-------|
<one row per inferred boundary with cells = TBD>

## Review Log
(auto-generated — not reviewed)
```

Dispatch to `/monke-design:lld <component> stub` for actual stub generation. Verify after dispatch that each stub contains the B1 minimum fields (File Map, Signatures, Maturity Tags, Decomposition Tree, Unit Test Plan skeleton, Integration Test Plan skeleton). If any field is missing from a stub → re-run stub generation with that field explicitly.

### Step 2.5: ONE HARD GATE — Present Assessment

This is the ONLY human pause in fast onboarding. Group findings by container. Surface uncertainty up-front.

**Auto-escalation (H7):** Before presenting, compute recommended rigor per design-specs S12.3:
- >15 components discovered → `thorough`.
- Any container tagged `agentic` candidate (versioned-artifact / data-dependent tool signals) → `thorough`.
- Compliance keywords found in README, package metadata, or file paths (`hipaa`, `pci`, `gdpr`, `sox`, `iso-27001`, `fedramp`, `soc2`, `ccpa`, `regulated`) → `thorough`.
- Any versioned-artifact / data-dependent tool in scan → at least `standard`.
- Polyglot project (>1 backend language) → at least `standard`.
- Otherwise → `standard` (default).

**Presentation format (container-grouped):**

```
⏸ PG-1 [HARD] — Fast onboarding complete. Project scope.

STACK: <detected languages, frameworks, test runners>
CI:    <detected CI config path, or "none detected">
DOCS:  <existing openapi/adr/docs dirs found, or "none">

CONTAINERS (<N>):
  <container-1>      [<M> components, stack: <lang>]
    - <K> at IL-<X>, <L> at IL-<Y>, ...
  <container-2>      ...
  ...

LOW-CONFIDENCE BOUNDARIES (review carefully):
  - <component-name> — <reason, e.g. "bare directory, no exports">
  <or "none — all boundaries HIGH confidence">

SCAN FAILURES (manual triage needed):
  - <component-name>: <command> failed — <error tail>
  <or "none">

CIRCULAR DEPENDENCIES (flagged):
  - <cycle: A → B → A>
  <or "none detected">

RIGOR RECOMMENDATION: <level>
  Reasoning: <which S12.3 triggers matched, or "default for N-component project">

ARTIFACTS WRITTEN (drafts):
  - monke-status.md (Components table populated)
  - monke-docs/hld.md (Confidence: mixed, auto-generated)
  - monke-docs/lld/<component>.md × <N> (Confidence: auto-generated)

Options:
  (a) confirm                 — accept all findings, route to first component
  (b) adjust                  — correct maturity / container / responsibility
  (c) override maturity <c>   — e.g. "payment-service is actually IL-3"
  (d) see detailed table      — show full per-component scan output
  (e) scope to one container  — limit routing to one container
  (f) run full recon instead  — `/monke recon` for survey → gaps → OQs → roadmap
  (g) rigor <level>           — override recommended rigor

Does this look right?
```

**Manual override handling (H8):**

If user says e.g. "payment-service is actually IL-3, the scan couldn't run tests because it needs a sandbox key":
- Store in `monke-status.md` Components table with override flag: `IL-3 (manual override <YYYY-MM-DD> — <user reason>)`.
- Append to Gate Audit Log: `[override] payment-service: scan=unknown → user=IL-3 (reason: sandbox key required)`.
- Mark for later verification when the blocker clears (e.g., when sandbox key becomes available) — re-scan to confirm.

**Container scoping (H7):**

If user says "scope to auth-service":
- Filter Components table to only that container's components for Phase 3 routing.
- Persist scope in `.monke-config.md` under `scope: <container>` (session-level; clears on next `/monke` invocation unless explicitly set).

**Rigor override:**

If user sets rigor at this gate → write to `.monke-config.md` and proceed with that rigor. Never silently upgrade or downgrade.

**On confirm:**
- Finalize `monke-status.md` — set `Rigor:` line, mark Components table confirmed, update `Where We Are:` to `Routing from fast-onboarding assessment`.
- Append PG-1 outcome to Gate Audit Log: `PG-1 SCOPE — human confirmed (fast-onboarding, rigor=<level>, <N> containers, <M> components)`.
- Proceed to Phase 3.

**On reject or full-recon request:**
- Keep the generated artifacts (they're useful input).
- Dispatch the recon chain starting at `/monke-recon:survey`; present each subsequent step (reconstruct → gaps → oqs → roadmap) after the previous returns.
- Do not auto-rerun fast onboarding after recon — recon produces its own status, HLD, LLDs.

---

## Phase 3: Unified Routing Tree

Routing is **per component**, not per project. Different components can be at different pipeline entry points simultaneously. The orchestrator recommends ONE next action per loop iteration — it does not dump a roadmap.

Read `monke-status.md` Components table. For each component, walk this tree top-to-bottom. **First match wins.**

```
FOR EACH COMPONENT (ordered: lowest maturity first, then dependency order from HLD S3):
│
├─ Component has blocking OQ? (open-questions.md: "Blocks: implementation of <component>")
│  └─ Recommend: `/monke-design:oq triage` or rage-origin resolution
│     "<component> blocked by OQ-<NNN>. Resolve first."
│
├─ LLD status = "reconstructed" (recon-origin)?
│  ├─ Test plan missing? → Recommend: `/monke-design:lld <component>` (review mode)
│  │   "Recon mapped <component> but no test plan. Review + formalize before building."
│  └─ Test plan present? → Recommend: `/monke-implement:implement <component> <layer>`
│      "Recon-origin LLD ready. Layer derived from maturity tags."
│
├─ LLD confidence = "auto-generated" (stub from fast onboarding)?
│  ├─ Rigor = thorough AND component is downstream of another component?
│  │   → Recommend: `/monke-design:lld <component>` (review mode — upgrade to "reviewed")
│  │     "Thorough rigor: auto-generated LLD needs PG-9 review before trusted as input to neighbors."
│  ├─ Rigor = standard/light AND stub has B1 minimum fields?
│  │   → Recommend: `/monke-implement:implement <component> <layer>`
│  │     "Stub LLD sufficient at <rigor>. Starting at Layer <layer> (maturity: <level>)."
│  └─ Stub missing B1 fields (decomposition tree / test plan skeleton)?
│      → Recommend: `/monke-design:lld <component> stub` (re-run)
│        "Stub incomplete — regenerate with full B1 minimum before implementation."
│
├─ LLD missing entirely AND maturity = missing?
│  └─ Recommend: `/monke-design:lld <component>`
│     "No code, no LLD. Generate from scratch."
│
├─ Maturity = pre-L0?
│  └─ Recommend: `/monke-implement:implement <component> 0`
│     "Placeholder directory. Starting at Layer 0 (type skeleton)."
│
├─ Maturity = IL-0 passed?
│  └─ Recommend: `/monke-implement:implement <component> 1`
│     "Types present. Layer 1 next (interface skeleton)."
│
├─ Maturity = IL-1 passed?
│  └─ Recommend: `/monke-implement:implement <component> 2`
│     "Interfaces present. Layer 2 next (bodies + unit tests, interleaved per function)."
│
├─ Maturity = IL-2 passed?
│  ├─ Upstream/downstream neighbors at IL-2 or higher?
│  │   → Recommend: `/monke-implement:implement <component> 3`
│  │     "Neighbors ready. Layer 3 integration tests."
│  └─ Neighbors not ready?
│      → Skip this component; mark deferred. Try next component.
│        "<component> waiting on <neighbor> at IL-2. Deferred."
│
├─ Maturity = IL-3 passed?
│  ├─ Other components in same phase not yet IL-3? → Skip, try next.
│  └─ All phase components IL-3? → Recommend: `/monke-implement:checkpoint <phase>`
│      "Phase <N> ready. Checkpoint requires PG-11 [HARD]."
│
├─ Maturity = unknown (scan failed)?
│  └─ Recommend: user triage (present the scan error + last-known good state)
│     "<component> scan failed: <error>. Manual decision: override maturity, fix env, or skip."
│
└─ All components IL-3 AND all phases checkpointed?
   └─ Recommend: (a) ship (b) `/monke rage` for final paranoia scan (cycles through buggy / drift / haunt / echo / improv / renounce as the user picks)
      "All gates green. Shipment or final sweep."
```

**Recommendation presentation format (single banana):**

```
⏸ Next action for <component>:
   /<dir>:<skill> <args>

Reason: <one sentence from the tree branch that matched>
Maturity: <component's current maturity>
Rigor: <active rigor>
Gate class(es) expected: <AUTO/SOFT/HARD list for the dispatched skill's phases>

Options: (yes / override / explain / done)
```

**Parallel work suggestion (worker pool archetype per design-specs S3.1):**

After presenting one recommendation, scan for other components at the same maturity with no cross-dependencies. If 2+ found:

```
Parallel candidates (independent, same maturity):
  - <component-a>, <component-b>, <component-c>

Spawn an agent team (worker pool) to work on these in parallel?
  (y / n / pick subset)
```

User decides whether to parallelize. If yes → create a worker-pool team per design-specs S3.1, one worker per component, lead orchestrator reconciles to `monke-status.md` (sole-writer rule).

**Scope limiter:**

If `CONTAINER_SCOPE` is set → filter the Components table to only that container's rows before walking the tree.

---

## Phase 4: Rigor-Aware Gate Handling

Every gate dispatched by a sub-skill carries a class (AUTO / SOFT / HARD / TRIGGERED) per design-specs S9.4. The orchestrator does not enforce rigor — the sub-skills do — but it DOES read outcomes and maintain the audit log.

### Gate Behavior Summary (per sdlc-specs §0 + design-specs S9.4 / S12)

| Gate Class | light | standard | thorough |
|------------|-------|----------|----------|
| **AUTO** (IL-0/1/2/3) | Silent; surface on fail | Silent; surface on fail | Silent; surface on fail |
| **HARD** (PG-1, PG-6, PG-11) | Always surfaces | Always surfaces | Always surfaces |
| **SOFT** (PG-2/3/4/5/7/8/9/10) | Auto-pass aggressively; surface only if auto-pass condition fails | Auto-pass when condition holds; surface otherwise | Always surfaces regardless of auto-pass |
| **TRIGGERED** (PG-12/13/14) | Surfaces when triggered | Surfaces when triggered | Surfaces when triggered |

**Critical invariants:**
1. AUTO gates that FAIL become visible. IL-2 with assertion-less tests fails → surfaces.
2. TRIGGERED gates fire regardless of rigor. PG-12 at `light` still pauses.
3. HARD gates never skippable. PG-11 at `light` still pauses.
4. Rigor hides confirmations, not failures.

### Audit Log Writes

After every dispatched skill returns, read its reported gate outcomes. Append one line per outcome to `monke-status.md` Gate Audit Log:

| Outcome | Line format |
|---------|-------------|
| Pass (AUTO) | `[gate:IL-<N>] auto-confirmed (<component>, <details>)` |
| Auto-pass (SOFT) | `[gate:PG-<N>] auto-confirmed (<auto-pass condition>, rigor=<level>)` |
| Surface + confirm | `[gate:PG-<N>] human confirmed (<component>)` |
| Surface + reject | `[gate:PG-<N>] human rejected — <reason>` |
| Failure (AUTO) | `[gate:IL-<N>] FAILED (<component>, <error>) — surfaced` |
| Trigger fire | `[gate:PG-<N>] TRIGGERED (<event>, <component>)` |

### Session Summary

On `done` or context death, emit a session summary:

```
Session gates (<date> <time> UTC):
  - <N> AUTO confirmed
  - <M> SOFT auto-passed (rigor=<level>)
  - <K> HARD surfaced → <confirmed>/<rejected>
  - <L> TRIGGERED fired → <resolved>/<open>
Components advanced: <count>
Next entry point on resume: <recommendation>
```

---

## Recovery Protocol

Recovery combines status-based resume + artifact-based verification + per-component progress tracking.

### Status-Based Resume

On every `/monke` invocation:
1. Read `monke-status.md` — `Updated:` line, `Where We Are:`, `Components` table, `Resume` block (if present from Context Death Protocol).
2. Reconcile against `.monke-config.md` rigor. If rigor differs from the active session's persisted rigor → announce the change, do not silently apply.
3. Check for a `Resume` block with this orchestrator's signature → jump to the named next action.

### Artifact-Based Verification

Trust but verify. The status file can lie (human edits, partial writes, stale updates).

For each component claiming `IL-N` passed in the Components table:
- If `N >= 0` → confirm via the IL-0 command from project-specs S8 (per container).
- If `N >= 2` → re-run the IL-2 assertion + coverage heuristic from Step 2.3. Empty-test suites that snuck through as IL-2 get downgraded.
- If any verification disagrees with the status → mark the component `stale: scan-mismatch`, surface at the next gate, ask user whether to trust the scan or the status.

Do this verification only on invocations where state looks "surprisingly advanced" (e.g., components claim IL-3 but no checkpoint file exists, or coverage dropped since last session). Skip on rapid successive invocations to avoid re-scanning the whole project every loop.

### Per-Component Progress Tracking

Components resume independently. If Component A is at IL-2 and Component B is at IL-1, the next `/monke` invocation routes Component B first (lower maturity, assuming no dependency inversion). A half-finished Layer 2 on Component A shows up as `progress: L2 3/7 functions` in the Implementation table — on resume, dispatch `implement A 2` and the sub-skill picks up at function 4.

### Loop Resume

On re-invocation mid-session:
- If last recommendation was unaccepted (user did not confirm, context died before dispatch) → re-present it.
- If last dispatch completed but status not yet updated (crash between skill exit and status write) → re-scan that component's artifacts to reconcile, then update status before proposing the next action.

---

## Context Death Protocol

The orchestrator runs long loops. Context pressure will hit. Die gracefully, resume cleanly.

**Checkpoint artifacts (written on context pressure):**
- `monke-status.md` — always updated with current state before every recommendation. No intermediate state lives only in memory.
- `monke-docs/hld.md` — if mid-fast-onboarding, partial HLD is written; `Source: auto-generated via fast onboarding (partial)` header flag.
- `monke-docs/lld/<component>-draft.md` — if stub generation died mid-component; dispatched sub-skill owns these.

**Status line format (written to `monke-status.md`):**

```
## Resume
Skill: /monke
Phase: <1 | 2.N | 3 | 4>
Last step: <specific action completed — e.g. "Step 2.3 scan 7/10 components done">
Last gate: <gate ID — outcome — e.g. "PG-1 awaiting confirmation">
Next action: <exactly what `/monke` should do on next invocation>
Scope: <container name if scoped, else "all">
Rigor: <active rigor>
Died at: <YYYY-MM-DD HH:MM UTC>
```

**Recovery detection (on entry):**
- If `monke-status.md` `Resume` block names `/monke` AND `Next action` is unfinished → jump to that action, skip Phase 1 state detection.
- If `Resume` block exists but names a sub-skill (e.g. `/monke-implement:implement`) AND the skill's own resume logic should run → dispatch that skill directly; it handles its own checkpoint.
- If no `Resume` block → standard Phase 1 detection.
- After successful recovery → clear the `Resume` block (write Phase 3 normal state).

**What the orchestrator checkpoints on pressure:**
1. Write the current Components table state.
2. Write the `Resume` block with current Phase / Last step / Next action.
3. Write any pending Gate Audit Log entries.
4. Emit a terse user-facing line: `Context exhausted. Resume with /monke — status checkpointed at <Phase X>.`

Never exit without writing the checkpoint. A silent death with unwritten status is the single worst failure mode in this framework — it leaks work, corrupts state, and makes the next invocation misroute.

---

## Status Update

On every iteration of the loop — not just on exit.

**On entry:**
1. Read `monke-status.md`, `.monke-config.md`, and (if present) `Resume` block.
2. Verify Agent Teams Gate + monke-docs existence.
3. If prerequisites fail → stop with WHAT/WHY/HOW.

**After every sub-skill dispatch:**
1. Let the sub-skill own its status update (per its own Status Update Protocol — e.g. `/monke-implement:implement` updates the Implementation + Test Gates tables).
2. After the sub-skill returns, read the updated status, append any gate outcomes to the Gate Audit Log, bump `Updated:` line with current date and `by /monke`.
3. Recompute "Where We Are" from the current state (e.g., "Phase 1 in progress — 3/5 components at IL-2").
4. Recompute "Next action" by re-walking Phase 3 routing.

**On exit (user says "done"):**
1. Write session summary (see Phase 4 → Session Summary) into the status file under a `## Last Session` section, replacing any prior one.
2. Clear any `Resume` block.
3. Bump `Updated:` line.

**On exit (blocked — e.g. all remaining components waiting on an OQ):**
1. Add blocker rows to Open Blockers table.
2. Update Where We Are: `Blocked — <blocker summary>`.
3. Suggest `/monke-design:oq triage`.

**On exit (partial — context death):**
See Context Death Protocol above.

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Auto-confirm a HARD gate (PG-1, PG-6, PG-11) because "it's obviously fine" | Refuse. HARD gates always surface regardless of rigor. Present, wait for explicit user response. Every time. |
| Skip Phase 1 state detection and assume a default route | Refuse. State detection is the first instruction. Greenfield, fast-onboarding, and resume branches behave differently; misrouting corrupts everything downstream. |
| Use the same gate behavior for `light` and `thorough` | Refuse. Rigor exists to differentiate. `light` auto-passes SOFT gates aggressively; `thorough` surfaces all SOFT gates. Ignoring rigor violates design-specs S12 and defeats the point of the adaptive gate system. |
| Write code as the lead orchestrator | Refuse. `/monke` is a router, not an implementer. Dispatch to `/monke-design:*`, `/monke-implement:*`, `/monke-test:*`, or spawn an agent team per design-specs S3.1. Lead-writes-code is forbidden except for <20-line status/config synthesis. |
| Dump a full per-component roadmap as one response | Refuse. One recommendation per loop iteration. The orchestrator presents the single ripest banana; the human picks, confirms, or redirects. Roadmaps belong in `/monke recon` output, not in every loop. |
| Silently upgrade rigor when >15 components detected | Refuse. Auto-escalation is a SUGGESTION at the HARD gate (PG-1), not an enforcement. User can decline per design-specs S12.3. |
| Treat an `auto-generated` LLD as trusted input for a neighbor component's LLD | Refuse. Stub LLDs from fast onboarding have confidence `auto-generated`; they are hypotheses, not contracts. Before using as input for a neighbor, run `/monke-design:lld <neighbor-of-stub>` in review mode per design-specs S6.3 confidence ladder. Under `thorough` rigor, this refusal escalates to a PG-9 surface. |
| Re-run fast onboarding on a project that already has `monke-status.md` | Refuse. Fast onboarding runs once. Subsequent invocations resume per the status file. If a user wants to re-scan, they can delete status or run `/monke recon` for the deeper path. |
| Continue after a fail gate by weakening the check (e.g. lowering coverage threshold, disabling assertions) | Refuse. Gates exist to prevent this exact shortcut. Surface the failure, let the human or the sub-skill fix the root cause. Never move the goalposts. |
| Dispatch to a sub-skill without reading `monke-status.md` first | Refuse. Status-first is mandatory per design-specs S8. The orchestrator's whole job is reading state and routing from it. |

---

*the all-seeing eye does not do the work. it sees the work, names the work, and hands the right tool to the right monke.*
