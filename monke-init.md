# WyrdMonke Init — Install Skills, Scaffold Project, Merge CLAUDE.md

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

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

- `git` available on PATH
- Internet access (clones from GitHub)
- Project directory is the current working directory

---

## Phase 1: Clone WyrdMonke

```bash
TMPDIR=$(mktemp -d)
git clone --depth 1 --branch "$BRANCH" https://github.com/insomniac-klutz/wyrdMonke.git "$TMPDIR"
```

If clone fails → check branch name, network. Stop.

**⏸ Decision gate** — present clone result (branch, commit SHA, contents overview):

Confirm / Adjust / Reject?

- **Confirm** → proceed to skill install
- **Adjust** → re-clone from a different branch
- **Reject** → abort init, clean up temp dir

---

## Phase 2: Install Skills

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
# Exclude monke-CLAUDE.md (handled in Phase 4)
ROOT_CMDS=$(find "$TMPDIR" -maxdepth 1 -name 'monke-*.md' ! -name 'monke-CLAUDE.md' \
  -exec sh -c 'grep -q "^> \*\*Usage:\*\*" "$1" && basename "$1"' _ {} \;)
```

Install discovered skills and root commands project-local at `.claude/commands/`:

```bash
mkdir -p .claude/commands
for DIR in $SKILL_DIRS; do
  cp -r "$TMPDIR/$DIR/" .claude/commands/$DIR/
done

# Root commands
for FILE in $ROOT_CMDS; do
  cp "$TMPDIR/$FILE" .claude/commands/$FILE
done
```

**⏸ Decision gate** — show install plan (list discovered skill directories and root commands, note if existing files will be overwritten):

Confirm / Adjust / Reject?

- **Confirm** → install all discovered skills and root commands to `.claude/commands/`
- **Adjust** → change which items to install
- **Reject** → skip install, proceed to Phase 3

After copying, verify everything landed:
```bash
for DIR in $SKILL_DIRS; do
  ls .claude/commands/$DIR/*.md
done
for FILE in $ROOT_CMDS; do
  ls .claude/commands/$FILE
done
```

Report: skill directory count, root command count, and total file count.

---

## Phase 3: Scaffold Project

Copy project template files into the current working directory:

1. `$TMPDIR/monke-docs/` → `./monke-docs/` (includes `monke-readsme.md` — the repo origin reference)
2. `$TMPDIR/monke-mermaid.mmd` → `./monke-mermaid.mmd`
3. `$TMPDIR/monke-docs/status-template.md` → `./monke-status.md` (rename on copy)
4. `$TMPDIR/monke-claude-settings.json` → `.claude/settings.json` (merge)
   - If `.claude/settings.json` does not exist → copy directly
   - If it exists → deep-merge: inject all keys from upstream without clobbering existing user settings
   - After merge, warn user: "`.claude/settings.json` updated — restart Claude Code (`/exit`) for changes to take effect."

**⏸ Decision gate** — show scaffold plan (note if existing files will be overwritten):

Confirm / Adjust / Reject?

- **Confirm** → copy project templates
- **Adjust** → change which files to scaffold
- **Reject** → skip scaffolding, proceed to Phase 4

---

## Phase 4: CLAUDE.md Merge

Copy `$TMPDIR/monke-CLAUDE.md` into the current project root.

### If `CLAUDE.md` already exists:

1. Read both existing `CLAUDE.md` and `monke-CLAUDE.md`.
2. Highlight what's in existing that's NOT in monke, and vice versa.

**⏸ Decision gate** — present diff summary:

Confirm / Adjust / Reject?

- **Confirm** → merge using monke-CLAUDE.md as skeleton, integrate existing content. Back up original as `CLAUDE.md.bak`
- **Adjust** → modify merge strategy (e.g., keep existing as skeleton instead)
- **Reject** → keep both files separate, user merges manually later

### If `CLAUDE.md` does not exist:

1. Rename `monke-CLAUDE.md` → `CLAUDE.md`.

**⏸ Decision gate** — present new CLAUDE.md:

Confirm / Adjust / Reject?

- **Confirm** → accept the new CLAUDE.md
- **Adjust** → edit CLAUDE.md before proceeding
- **Reject** → remove CLAUDE.md, skip this phase

---

## Phase 5: Cleanup

**⏸ Decision gate** — confirm ready to clean up:

Confirm / Adjust / Reject?

- **Confirm** → delete temp dir, proceed to verify
- **Adjust** → inspect temp dir contents first, then re-ask
- **Reject** → keep temp dir for manual inspection, proceed to verify

```bash
rm -rf "$TMPDIR"
```

---

## Phase 6: Verify & Next Steps

1. Verify files landed:
   ```bash
   ls monke-docs/ CLAUDE.md monke-mermaid.mmd monke-status.md .claude/settings.json
   for DIR in $SKILL_DIRS; do
     ls .claude/commands/$DIR/*.md
   done
   for FILE in $ROOT_CMDS; do
     ls .claude/commands/$FILE
   done
   ```

2. Show summary:
   - Skills installed: list each discovered directory + file counts
   - Root commands installed: list each root command file
   - Project files scaffolded: list what was copied
   - CLAUDE.md status: new / merged / separate
   - Settings: new / merged (list injected keys)
   - Branch used: `$BRANCH`

3. Tell the user:
   - "Skills installed. You now have N slash commands available across M skill directories."
   - "Run `/monke-design:tinker` to detect your stack and fill in the project template."
   - "Or if you have existing code: `/monke-recon:survey` then `/monke-recon:reconstruct` to reverse-engineer an HLD."
