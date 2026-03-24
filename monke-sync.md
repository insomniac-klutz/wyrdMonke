# WyrdMonke Sync — Update Skills & Specs Without Touching Your Work

> **Usage:** Copy this file to `~/.claude/commands/monke-sync.md` (global). Then run `/monke-sync [branch]` inside a project already bootstrapped by `/monke-init`.

---

## Arguments

`$ARGUMENTS` parsing:
- Single positional: the wyrdMonke branch to sync from
- Default: `trunk`
- Example: `/monke-sync screectch`

```
BRANCH="${ARGUMENTS:-trunk}"
```

---

## Prerequisites

- Project already bootstrapped (`monke-docs/` exists, `CLAUDE.md` exists)
- `git` available on PATH
- Internet access

If not bootstrapped → tell user: "No WyrdMonke project found. Run `/monke-init` first." Stop.

---

## What Gets Touched

| Category | Action | Why |
|----------|--------|-----|
| **Skills** (`.claude/commands/monke-*`) | Overwrite | Prompt templates — no user state |
| **Root commands** (`monke-*.md` at repo root) | Overwrite | Prompt templates — no user state |
| **CLAUDE.md** (from `monke-CLAUDE.md`) | Diff + confirm | May have user modifications (filled placeholders) |
| **Syncable specs** (auto-discovered `*.md` in `monke-docs/`) | Diff + confirm | May have user modifications |
| **Protected artifacts** (`hld.md`, `lld/`, `decisions/`, `checkpoints/`, `open-questions.md`, `project-specs.md`, `rage-run/`, `rage-runs/`) | NEVER | Your design work — hands off |
| **monke-mermaid.mmd** | Overwrite | Skill handoff graph — no user state |
| **Project files** (`monke-status.md`) | NEVER | Your configuration — hands off |

---

## Phase 1: Clone

```bash
TMPDIR=$(mktemp -d)
git clone --depth 1 --branch "$BRANCH" https://github.com/insomniac-klutz/wyrdMonke.git "$TMPDIR"
```

If clone fails → check branch name, network. Stop.

**⏸ Decision gate** — present clone result (branch, commit SHA):

Confirm / Adjust / Reject?

- **Confirm** → proceed to skill update
- **Adjust** → re-clone from a different branch
- **Reject** → abort sync, clean up temp dir

---

## Phase 2: Update Skills & Commands

Auto-discover skill directories from the upstream clone:

```bash
# Find all monke-* dirs that contain at least one .md file
# Exclude monke-docs/ (specs) and monke-owns/ (assets)
SKILL_DIRS=$(find "$TMPDIR" -maxdepth 1 -type d -name 'monke-*' \
  ! -name 'monke-docs' \
  ! -name 'monke-owns' \
  -exec sh -c 'ls "$1"/*.md >/dev/null 2>&1 && basename "$1"' _ {} \;)
```

Auto-discover root-level command files (includes `monke-sync.md` itself):

```bash
# Find all root-level monke-*.md files, excluding monke-CLAUDE.md (handled in Phase 3)
ROOT_CMDS=$(find "$TMPDIR" -maxdepth 1 -name 'monke-*.md' ! -name 'monke-CLAUDE.md' -exec basename {} \;)
```

For each discovered skill directory and root command, overwrite in `.claude/commands/`:

```bash
for DIR in $SKILL_DIRS; do
  cp -r "$TMPDIR/$DIR/" .claude/commands/$DIR/
done

# Root commands
for FILE in $ROOT_CMDS; do
  cp "$TMPDIR/$FILE" .claude/commands/$FILE
done

# Skill handoff graph
cp "$TMPDIR/monke-mermaid.mmd" ./monke-mermaid.mmd
```

**⏸ Decision gate** — show update plan (list discovered skill directories, root commands, and files that will be overwritten):

Confirm / Adjust / Reject?

- **Confirm** → overwrite all discovered skill directories and root commands
- **Adjust** → change which items to update
- **Reject** → skip skill/command update, proceed to Phase 3

Verify all skills and commands landed:
```bash
for DIR in $SKILL_DIRS; do
  ls .claude/commands/$DIR/*.md
done
for FILE in $ROOT_CMDS; do
  ls .claude/commands/$FILE
done
```

Report: "Skills & commands updated from `$BRANCH`. X directories, Y root commands, Z files replaced."

---

## Phase 3: Diff Specs

Auto-discover syncable spec files from the upstream clone:

```bash
# All .md files directly in monke-docs/, minus protected user artifacts
PROTECTED="project-specs.md hld.md open-questions.md"
SPEC_FILES=$(find "$TMPDIR/monke-docs" -maxdepth 1 -name '*.md' -exec basename {} \; \
  | grep -v -F "$(printf '%s\n' $PROTECTED)")
```

For each discovered spec file:

1. Compare `$TMPDIR/monke-docs/<file>` against `./monke-docs/<file>`
2. If identical → skip, report "no changes"
3. If local file doesn't exist → flag as new upstream spec, offer to copy
4. If different → show a summary of what changed (sections added, removed, or modified)

**⏸ Decision gate** — for each changed spec, present diff summary:

```
<file> has upstream changes:
  - <summary of changes>
```

Confirm / Adjust / Reject?

- **Confirm** → overwrite with new version
- **Adjust** → show full diff before deciding, then re-ask
- **Reject** → keep current version, leave untouched

**NEVER touch these files regardless of discovery:**
- `project-specs.md` — user's stack bindings
- `hld.md` — user's design
- `open-questions.md` — user's questions
- Anything in subdirectories (`lld/`, `decisions/`, `checkpoints/`, `rage-run/`, `rage-runs/`)

### CLAUDE.md Sync

Compare upstream `monke-CLAUDE.md` against local `CLAUDE.md`:

1. If identical → skip, report "no changes"
2. If local `CLAUDE.md` doesn't exist → flag as missing, offer to copy template
3. If different → show a summary of what changed (sections added, removed, or modified), preserving user-filled placeholder content

**⏸ Decision gate** — present diff summary:

```
CLAUDE.md has upstream changes:
  - <summary of structural changes>
  - User-filled sections (<<<placeholders>>>) will be preserved
```

Confirm / Adjust / Reject?

- **Confirm** → merge structural changes, preserve user content in placeholder sections
- **Adjust** → show full diff before deciding, then re-ask
- **Reject** → keep current version, leave untouched

---

## Phase 4: Cleanup

**⏸ Decision gate** — confirm ready to clean up:

Confirm / Adjust / Reject?

- **Confirm** → delete temp dir, proceed to report
- **Adjust** → inspect temp dir contents first, then re-ask
- **Reject** → keep temp dir for manual inspection, proceed to report

```bash
rm -rf "$TMPDIR"
```

---

## Phase 5: Report

Present sync summary:

```
WyrdMonke Sync Complete
Branch: <branch>

Skills & Commands:
  - <N> skill directories updated (<M> files total)
  - <list each discovered skill dir and file count>
  - <P> root commands updated
  - <list each root command file>
  - monke-mermaid.mmd: replaced

CLAUDE.md: <applied | skipped | no changes>

Specs:
  - <for each discovered spec file>: <applied | skipped | new | no changes>

Untouched (your work):
  - project-specs.md, monke-status.md
  - hld.md, open-questions.md, lld/, decisions/, checkpoints/
```

If any specs were applied → suggest:
- "Review applied spec changes. If your `project-specs.md` references section numbers that shifted, verify with `/monke-status:status rebuild`."

If specs were skipped → note:
- "Skipped specs may diverge from skill expectations over time. Consider reviewing them later."

Check `.gitignore` — verify `CLAUDE.md` and `.claude/` are listed. If either is missing, warn the user:

> "⚠ `CLAUDE.md` and `.claude/` contain project-specific AI instructions and should not be committed to your repo. Add them to `.gitignore`:"
> ```
> CLAUDE.md
> .claude/
> ```
