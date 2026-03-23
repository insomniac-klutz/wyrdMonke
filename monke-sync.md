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
| **Core specs** (`design-specs.md`, `implementation-specs.md`, `test-specs.md`, `sdlc-specs.md`) | Diff + confirm | May have user modifications |
| **Status template** (`status-template.md`) | Diff + confirm | Template only — user's `monke-status.md` is untouched |
| **Artifacts** (`hld.md`, `lld/*.md`, `decisions/*.md`, `checkpoints/*.md`, `open-questions.md`) | NEVER | Your design work — hands off |
| **Project files** (`CLAUDE.md`, `project-specs.md`, `monke-status.md`, `monke-mermaid.mmd`) | NEVER | Your configuration — hands off |

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

## Phase 2: Update Skills

Overwrite all skill directories in `.claude/commands/`:

```bash
cp -r "$TMPDIR/monke-design/" .claude/commands/monke-design/
cp -r "$TMPDIR/monke-implement/" .claude/commands/monke-implement/
cp -r "$TMPDIR/monke-test/" .claude/commands/monke-test/
cp -r "$TMPDIR/monke-status/" .claude/commands/monke-status/
```

**⏸ Decision gate** — show skill update plan (list files that will be overwritten):

Confirm / Adjust / Reject?

- **Confirm** → overwrite all skill directories
- **Adjust** → change which skill directories to update
- **Reject** → skip skill update, proceed to Phase 3

Verify all skills landed:
```bash
ls .claude/commands/monke-design/*.md
ls .claude/commands/monke-implement/*.md
ls .claude/commands/monke-test/*.md
ls .claude/commands/monke-status/*.md
```

Report: "Skills updated from `$BRANCH`. X files replaced."

---

## Phase 3: Diff Specs

For each core spec file:

```
sdlc-specs.md
design-specs.md
implementation-specs.md
test-specs.md
status-template.md
```

1. Compare `$TMPDIR/monke-docs/<file>` against `./monke-docs/<file>`
2. If identical → skip, report "no changes"
3. If different → show a summary of what changed (sections added, removed, or modified)

**⏸ Decision gate** — for each changed spec, present diff summary:

```
<file> has upstream changes:
  - <summary of changes>
```

Confirm / Adjust / Reject?

- **Confirm** → overwrite with new version
- **Adjust** → show full diff before deciding, then re-ask
- **Reject** → keep current version, leave untouched

**NEVER touch these files regardless of diff:**
- `project-specs.md` — user's stack bindings
- `hld.md` — user's design
- `open-questions.md` — user's questions
- Anything in `lld/`, `decisions/`, `checkpoints/`

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

Skills: updated (13 files)
Specs:
  - sdlc-specs.md: <applied | skipped | no changes>
  - design-specs.md: <applied | skipped | no changes>
  - implementation-specs.md: <applied | skipped | no changes>
  - test-specs.md: <applied | skipped | no changes>
  - status-template.md: <applied | skipped | no changes>

Untouched (your work):
  - CLAUDE.md, project-specs.md, monke-status.md, monke-mermaid.mmd
  - hld.md, lld/*.md, decisions/*.md, checkpoints/*.md, open-questions.md
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
