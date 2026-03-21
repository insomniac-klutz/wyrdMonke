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

Install skills project-local at `.claude/commands/` so they're scoped to this project:

```bash
mkdir -p .claude/commands
cp -r "$TMPDIR/monke-design/" .claude/commands/monke-design/
cp -r "$TMPDIR/monke-implement/" .claude/commands/monke-implement/
cp -r "$TMPDIR/monke-test/" .claude/commands/monke-test/
cp -r "$TMPDIR/monke-status/" .claude/commands/monke-status/
```

**⏸ Decision gate** — show skill install plan (note if existing skills will be overwritten):

Confirm / Adjust / Reject?

- **Confirm** → install skills to `.claude/commands/`
- **Adjust** → change which skill directories to install
- **Reject** → skip skill install, proceed to Phase 3

After copying, verify all 13 skills landed:
```bash
ls .claude/commands/monke-design/*.md
ls .claude/commands/monke-implement/*.md
ls .claude/commands/monke-test/*.md
ls .claude/commands/monke-status/*.md
```

Expected: 6 + 3 + 3 + 1 = 13 skill files.

---

## Phase 3: Scaffold Project

Copy project template files into the current working directory:

1. `$TMPDIR/monke-docs/` → `./monke-docs/`
2. `$TMPDIR/monke-mermaid.mmd` → `./monke-mermaid.mmd`
3. `$TMPDIR/monke-docs/status-template.md` → `./monke-status.md` (rename on copy)

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
   ls monke-docs/ CLAUDE.md monke-mermaid.mmd monke-status.md
   ```

2. Show summary:
   - Skills installed: list the 4 directories + file counts
   - Project files scaffolded: list what was copied
   - CLAUDE.md status: new / merged / separate
   - Branch used: `$BRANCH`

3. Tell the user:
   - "Skills installed. You now have 13 slash commands available."
   - "Run `/monke-design:tinker` to detect your stack and fill in the project template."
   - "Or if you have existing code: `/monke-design:recon` to reverse-engineer an HLD."
