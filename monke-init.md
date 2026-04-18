# WyrdMonke Init — The One Ring

> **Usage:** Copy this single file to `~/.claude/commands/monke-init.md` (global). Then run `/monke-init [branch]` inside your target project. Skills install project-local at `.claude/commands/`.

---

## Arguments

`$ARGUMENTS` parsing:
- Single positional: the wyrdMonke branch to clone from
- Default: `trunk`
- Example: `/monke-init screectch`

```
BRANCH="${ARGUMENTS:-trunk}"
```

---

## Prerequisites

**Agent Teams Gate — EXEMPT.** Init is the skill that CREATES `CLAUDE.md`, so it cannot require `CLAUDE.md`'s Agent Teams section as a prereq (bootstrap paradox). Fallback check: after clone, verify `$TMPDIR/monke-CLAUDE.md` exists and contains the `## Agent Teams` header. If that file is missing from the upstream clone, fail with:

> "Upstream clone missing `monke-CLAUDE.md` — cannot bootstrap agent teams. Check branch name and repo integrity."

- `git` available on PATH
- Internet access (clones from GitHub)
- Project directory is the current working directory

---

## Phase 1: Clone & Install (no gate)

Clone upstream and install skills + templates mechanically. No human gate — this phase is pure plumbing.

```bash
TMPDIR=$(mktemp -d)
git clone --depth 1 --branch "$BRANCH" https://github.com/insomniac-klutz/wyrdMonke.git "$TMPDIR"
```

If clone fails → check branch name, network. Stop with error.

**Agent Teams fallback check:** verify `$TMPDIR/monke-CLAUDE.md` exists and contains `## Agent Teams`. If not → stop with the error above.

### Install skills

Auto-discover skill directories and root command files from the upstream clone:

```bash
# Find all monke-* dirs that contain at least one .md file
# Exclude monke-docs/ (specs) and monke-owns/ (assets)
SKILL_DIRS=$(find "$TMPDIR" -maxdepth 1 -type d -name 'monke-*' \
  ! -name 'monke-docs' \
  ! -name 'monke-owns' \
  -exec sh -c 'ls "$1"/*.md >/dev/null 2>&1 && basename "$1"' _ {} \;)

# Root-level monke-*.md + monke.md that are slash commands. Only files with a
# literal "> **Usage:**" line get copied — italic-tagline meta-docs (drafter,
# phil, log, fut) are reference material, not slash commands, and stay upstream.
# Exclude monke-CLAUDE.md (handled in Phase 3) and monke-mermaid.mmd (scaffolded separately).
ROOT_CMDS=$(find "$TMPDIR" -maxdepth 1 \( -name 'monke-*.md' -o -name 'monke.md' \) ! -name 'monke-CLAUDE.md' \
  -exec sh -c 'head -10 "$1" | grep -q "^> \*\*Usage:\*\*" && basename "$1"' _ {} \;)
# head -10 bound matters: monke-drafter.md shows a "> **Usage:**" line at L45
# as an EXAMPLE of what skills should contain. Real skills put Usage at L3.
# The line-10 cap excludes drafter's example-in-doc without a named-file skip.

mkdir -p .claude/commands
for DIR in $SKILL_DIRS; do
  cp -r "$TMPDIR/$DIR/" .claude/commands/$DIR/
done
for FILE in $ROOT_CMDS; do
  cp "$TMPDIR/$FILE" .claude/commands/$FILE
done
```

### Scaffold project files

```bash
cp -r "$TMPDIR/monke-docs/" ./monke-docs/
cp "$TMPDIR/monke-mermaid.mmd" ./monke-mermaid.mmd
cp "$TMPDIR/monke-docs/status-template.md" ./monke-status.md

# Merge settings (inject keys without clobbering)
if [ -f .claude/settings.json ]; then
  # deep-merge — inject upstream keys, preserve user overrides
  # (implementation: use jq or manual merge)
  :
else
  cp "$TMPDIR/monke-claude-settings.json" .claude/settings.json
fi
```

### Merge CLAUDE.md

- If `CLAUDE.md` exists → back up as `CLAUDE.md.bak`, merge upstream as skeleton, integrate existing content (gotchas, project description, custom rules).
- If `CLAUDE.md` does not exist → copy `$TMPDIR/monke-CLAUDE.md` to `./CLAUDE.md`.

Warn if settings were merged: "`.claude/settings.json` updated — restart Claude Code (`/exit`) for changes to take effect."

---

## Phase 2: Auto-detect Stack (no gate)

Scan the current project directory for stack signals. No human gate — auto-detection is mechanical.

| Signal | Detects |
|--------|---------|
| `package.json` / `pnpm-lock.yaml` / `yarn.lock` | JS/TS, package manager, scripts |
| `pyproject.toml` / `requirements.txt` / `poetry.lock` | Python, package manager, test runner |
| `Cargo.toml` / `Cargo.lock` | Rust |
| `go.mod` / `go.sum` | Go |
| `pom.xml` / `build.gradle` | Java |
| `Gemfile` | Ruby |
| `*.csproj` / `*.sln` | C# |
| `Dockerfile` / `docker-compose.yml` | containers |
| `.github/workflows/*.yml` | CI commands (extract for IL gate bindings) |
| `openapi.yaml` / `swagger.json` | existing API contracts |
| `docs/` directory with markdown | existing docs (consume as input) |

Record detected stack into `monke-docs/project-specs.md` Stack section (auto-populate placeholders where confident, leave `<<<...>>>` where ambiguous).

**Existing code detected?** Note it — Phase 4 will trigger fast onboarding.

---

## Phase 3: Set Rigor (SINGLE HUMAN GATE)

⏸ **PG-INIT [HARD] — Rigor selection for this project.**
<!-- Bootstrap-tier gate: not in S9.4's numbered schedule. HARD because the project's entire gate behavior downstream depends on this answer; auto-passing would force a default rigor the user never agreed to. Never auto-passes. Never skippable. -->

This is the **only** human gate in init. One question, three answers.

Present to user:

> **Rigor level for this project?**
>
> - `light` — ~2 human gates total (scope + ship). Use for MVPs, small tools, personal projects.
> - `standard` — ~4-5 gates (scope + components + tradeoffs + LLD + ship). Use for medium projects, team work.
> - `thorough` — ~8+ gates (all SOFT gates surface). Use for large, critical, or regulated projects.
>
> Default: `standard`.

Write the answer to `.monke-config.md` at project root:

```markdown
# Monke Config

rigor: standard
set_by: /monke-init
set_on: <YYYY-MM-DD>
```

---

## Phase 4: Finalize (no gate)

Mechanical cleanup + routing.

1. Delete `$TMPDIR`.
2. Verify files landed:
   ```bash
   ls monke-docs/ CLAUDE.md monke-mermaid.mmd monke-status.md .claude/settings.json .monke-config.md
   ```
3. **If existing code was detected in Phase 2 → trigger fast onboarding automatically.** Invoke `/monke` (the unified orchestrator) — it will run the fast onboarding scan and route to the first component needing work. No separate "run tinker then fill then survey" dance.
4. **If greenfield (no code detected) →** tell the user:
   > "Init complete. Rigor: `<level>`. Run `/monke` to begin — it will ask whether this is a flash MVP or production build."

Report:
- Skills installed: count + list
- Project files scaffolded: count + list
- CLAUDE.md status: new / merged
- Settings status: new / merged
- Rigor level: `<level>`
- Next step: auto-triggered fast onboarding, or `/monke` invocation

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Skip the Agent Teams fallback check (accept an upstream clone without `monke-CLAUDE.md` or without `## Agent Teams`) | Refuse. Init is the skill that seeds agent teams into the project; if the upstream seed is broken, installing it corrupts every future skill invocation. Stop with the WHAT/WHY/HOW error from Prerequisites. |
| Overwrite an existing `CLAUDE.md` without first writing `CLAUDE.md.bak` | Refuse. Users put project secrets, gotchas, and custom rules in `CLAUDE.md`. Silent overwrite deletes work. Always back up, then merge — never replace. |
| Skip Phase 3 rigor prompt and write a default `standard` silently | Refuse. PG-INIT is HARD. Every downstream gate behavior (SOFT surfaces, auto-pass thresholds) reads from `.monke-config.md`. A default the user never confirmed corrupts the entire session's gate math. |
| Re-run `/monke-init` on a project that already has `monke-status.md` + populated `monke-docs/` | Refuse. Init is a one-shot bootstrap. Re-running clobbers skills but can't safely reinitialize status. Suggest `/monke-sync` to pull upstream skill updates without touching user state. |
| Copy italic-tagline meta-docs (`monke-drafter.md`, `monke-phil.md`, `monke-log.md`, `monke-fut.md`) into target-project `.claude/commands/` | Refuse. `.claude/commands/` is for slash-invocable skills only — files with a literal `> **Usage:**` line. Meta-docs are reference material that lives upstream (users consult them in the wyrdMonke source repo, not their target project). The strict filter keeps the target namespace clean. |
| Proceed past a failed clone (branch not found, network error) by defaulting to `trunk` | Refuse. The user asked for a specific branch for a reason (feature preview, pinned release). Stop with the error. Let the user pick. |

---

## Context Death Protocol

**Status Update exemption:** Init CREATES `monke-status.md` — there is no status file to read on entry. Recovery therefore leans on filesystem artifacts, not a Resume block.

**Checkpoint artifacts (written on context pressure):**
- `CLAUDE.md.bak` — written BEFORE any CLAUDE.md merge begins; presence of a fresh `.bak` without a merged `CLAUDE.md` proves init died mid-merge.
- `monke-docs/` (partial directory) — presence without `monke-status.md` or `.monke-config.md` proves scaffold phase died.
- `$TMPDIR` leftover — if the clone temp dir survives (Phase 4 deletes it on success), init died before cleanup.
- Partial skill copies under `.claude/commands/monke-*/` — presence of some but not all skill dirs proves Phase 1 copy loop died.

**Status line format:** N/A for init — status file does not yet exist. Init writes its initial `Updated:` line at the end of Phase 4 when status is seeded, not before.

**Recovery detection (on entry):**
- If `CLAUDE.md.bak` exists AND `CLAUDE.md` is missing or contains upstream skeleton with unfilled placeholders → prior init died in the CLAUDE.md merge. Restore from `.bak`, tell the user, re-run init cleanly.
- If `monke-docs/` exists BUT `monke-status.md` is missing AND `.monke-config.md` is missing → prior init died before Phase 3 rigor gate. Safe to re-run — Phase 1 copy is idempotent.
- If `.claude/commands/monke-*/` exists partially (some skill dirs present, some missing) → prior init died in Phase 1. Safe to re-run — `cp -r` overwrites cleanly.
- If `$TMPDIR` leftover detected (stale `/tmp/tmp.*` with `.git/` inside it) → report and clean up before re-running.
- If none of the above → start clean from Phase 1.

---

## Status Update

**Exemption:** Init is the skill that CREATES `monke-status.md`. It cannot read it on entry (it does not yet exist). On exit (success), init writes the initial status file from the template and stamps:

```
Updated: <YYYY-MM-DD> by /monke-init
```

On exit (failure before Phase 4): init does NOT write a partial `monke-status.md` — a partial status file would mislead future `/monke` invocations into skipping fast onboarding. Instead, init surfaces the error, leaves a clean filesystem (per Context Death Protocol recovery), and asks the user to re-run.

On exit (failure after Phase 4 status seed): treat as Context Death — user state exists, recovery is forward-only via `/monke` or `/monke-sync`, never a re-run of init.
