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

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

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
| **Syncable specs** (auto-discovered `*.md` in `monke-docs/`, incl. `monke-readsme.md`) | Diff + confirm | May have user modifications |
| **Protected artifacts** (`hld.md`, `lld/`, `decisions/`, `checkpoints/`, `open-questions.md`, `project-specs.md`, `rage-run/`, `rage-runs/`) | NEVER | Your design work — hands off |
| **monke-mermaid.mmd** | Overwrite | Skill handoff graph — no user state |
| **Settings** (`monke-claude-settings.json` → `.claude/settings.json`) | Merge + confirm | May have user settings — deep-merge, never clobber |
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

Auto-discover root-level skill files (only files with `> **Usage:**` — excludes reference docs like drafter, phil, log, fut):

```bash
# Root-level monke-*.md that are actual slash commands, not reference docs
# Exclude monke-CLAUDE.md (handled in Phase 3) and user project files (NEVER overwrite)
ROOT_CMDS=$(find "$TMPDIR" -maxdepth 1 -name 'monke-*.md' \
  ! -name 'monke-CLAUDE.md' \
  ! -name 'monke-status.md' \
  -exec sh -c 'grep -q "^> \*\*Usage:\*\*" "$1" && basename "$1"' _ {} \;)
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

### Settings Merge

Merge `$TMPDIR/monke-claude-settings.json` into `.claude/settings.json`:

- If `.claude/settings.json` does not exist → copy directly
- If it exists → deep-merge: inject all keys from upstream without clobbering existing user settings
- Show which keys will be added or already present

**⏸ Decision gate** — show update plan (list discovered skill directories, root commands, settings changes, and files that will be overwritten):

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

Verify protected project files were NOT modified:
```bash
# These must remain untouched — assert no upstream copy landed here
for GUARD in ./monke-status.md ./monke-docs/project-specs.md ./monke-docs/hld.md ./monke-docs/open-questions.md; do
  # If file exists locally, confirm it was not overwritten by this phase
  [ -f "$GUARD" ] && echo "✓ $GUARD untouched"
done
```

If any protected file was modified → **abort sync**, report the violation, and restore from git.

Report: "Skills & commands updated from `$BRANCH`. X directories, Y root commands, Z files replaced."

---

## Phase 3: Diff Specs

Auto-discover syncable spec files from the upstream clone:

```bash
# Protected user project files — NEVER overwrite regardless of upstream changes
# Protected dirs (lld/, decisions/, checkpoints/, rage-run/, rage-runs/) excluded by -maxdepth 1
PROTECTED="project-specs.md hld.md open-questions.md"
SPEC_FILES=$(find "$TMPDIR/monke-docs" -maxdepth 1 -type f -name '*.md' -exec basename {} \; \
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

**NEVER touch these files regardless of discovery — hard gate, not a suggestion:**
- `project-specs.md` — user's stack bindings
- `hld.md` — user's design
- `open-questions.md` — user's questions
- `monke-status.md` (project root) — user's progress dashboard
- Anything in subdirectories (`lld/`, `decisions/`, `checkpoints/`, `rage-run/`, `rage-runs/`)

If any SPEC_FILES entry matches a protected file, **remove it from the list before proceeding**. Log the skip.

### monke-status.md Reconciliation

**Trigger:** Only runs if `status-template.md` was updated (confirmed) in the spec sync above. If the template was skipped, rejected, or unchanged → skip this subsection entirely.

When the template structure changes, the user's `monke-status.md` at the project root may drift from the expected format. Reconcile — never replace:

1. If `monke-status.md` doesn't exist at project root → skip (nothing to reconcile)
2. Compare the **structure** of the new `status-template.md` against the existing `monke-status.md`:
   - Identify new sections/tables added to the template
   - Identify columns added, removed, or renamed in existing tables
   - Identify sections removed from the template
3. **Merge structure, preserve data:**
   - Add new sections from the template (empty, with placeholder rows)
   - Add new columns to existing tables (blank cells for existing rows)
   - **Preserve ALL user data** — filled rows, checked boxes, progress values, component entries, blocker entries
   - If a section was removed from the template, keep it in the user's file but flag it: `<!-- deprecated by upstream template change -->`
   - If columns were renamed, map old column data to new column names where the intent is obvious; flag ambiguous renames for user review

**⏸ Decision gate** — present reconciliation plan:

```
monke-status.md reconciliation (status-template.md changed):
  Adding:    <new sections/columns>
  Renaming:  <column renames, if any>
  Keeping:   <all user data rows, checkboxes, progress>
  Flagged:   <deprecated sections or ambiguous renames needing review>
```

Confirm / Adjust / Reject?

- **Confirm** → apply structural merge, preserve all user data
- **Adjust** → show side-by-side of old vs new template structure, then re-ask
- **Reject** → keep current `monke-status.md` as-is (may drift from template expectations)

If rejected → note in Phase 5 report: "Status template updated but `monke-status.md` was not reconciled. Run `/monke-status:status rebuild` to regenerate from artifacts if structure drift causes issues."

### CLAUDE.md Sync

Merge upstream `monke-CLAUDE.md` INTO the existing local `CLAUDE.md` — never replace it:

1. If identical → skip, report "no changes"
2. If local `CLAUDE.md` doesn't exist → flag as missing, offer to copy template
3. If different → **merge, not replace:**
   - Identify WyrdMonke-managed sections in `monke-CLAUDE.md` (Agent Teams, Sacred Tree Invariant, Skill Structural Standard, and any section with a `monke-` prefix or WyrdMonke header)
   - Add new WyrdMonke sections that don't exist locally
   - Update existing WyrdMonke sections with upstream changes
   - **Preserve ALL user-added content** — custom sections, rules, project notes, filled `<<<placeholders>>>`, anything not from the WyrdMonke template stays untouched
   - Show what will be added/updated vs what will be preserved

**⏸ Decision gate** — present merge plan:

```
CLAUDE.md merge from monke-CLAUDE.md:
  Adding:    <list new sections>
  Updating:  <list changed WyrdMonke sections>
  Keeping:   <list user sections + filled placeholders that won't be touched>
```

Confirm / Adjust / Reject?

- **Confirm** → apply merge, preserve all user content
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

Settings (.claude/settings.json): <merged | created | no changes>
CLAUDE.md: <merged | skipped | no changes>

Specs:
  - <for each discovered spec file>: <applied | skipped | new | no changes>

monke-status.md: <reconciled | skipped | not triggered | no file>

Untouched (your work):
  - project-specs.md, monke-status.md (data preserved)
  - hld.md, open-questions.md, lld/, decisions/, checkpoints/
```

If any specs were applied → suggest:
- "Review applied spec changes. If your `project-specs.md` references section numbers that shifted, verify with `/monke-status:status rebuild`."

If specs were skipped → note:
- "Skipped specs may diverge from skill expectations over time. Consider reviewing them later."

If settings were created or merged → warn:
- "`.claude/settings.json` was updated — restart Claude Code (`/exit`) for changes to take effect."

Check `.gitignore` — verify `CLAUDE.md` and `.claude/` are listed. If either is missing, warn the user:

> "⚠ `CLAUDE.md` and `.claude/` contain project-specific AI instructions and should not be committed to your repo. Add them to `.gitignore`:"
> ```
> CLAUDE.md
> .claude/
> ```
