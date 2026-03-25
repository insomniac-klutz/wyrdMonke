# WyrdMonke Ops:Commit — Grouped, Confirmed Commits

> **Usage:** Copy `monke-ops/` to `~/.claude/commands/monke-ops/`. Invoke: `/monke-ops:commit`

---

## Arguments

`$ARGUMENTS` parsing:
- None. This skill takes no arguments.

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

- `git` available on PATH
- Inside a git repository
- Working tree has changes (staged, unstaged, or untracked tracked-pattern files)

If no changes detected → tell user: "Working tree clean. Nothing to commit." Stop.

---

## Phase 1: Gather Changes

Run `git status` (short format). Collect all changed files into a flat list with their status:

| Status | Meaning |
|--------|---------|
| `M` (staged) | Modified, already staged |
| `M` (unstaged) | Modified, not yet staged |
| `A` | Added (new file, staged) |
| `D` | Deleted |
| `R` | Renamed |
| `??` | Untracked |

For untracked files: only include files that match patterns a developer would intentionally commit (source, docs, config). Exclude obvious noise (`.DS_Store`, `node_modules/`, `__pycache__/`, build artifacts, `.env` files). If unsure about an untracked file, include it in the list but flag it.

Present the raw file list to the user:

```
Changed files detected:

  M  src/api/handler.rs
  M  src/api/models.rs
  M  monke-docs/hld.md
  A  monke-ops/commit.md
  ?? src/api/new_endpoint.rs

<count> files total
```

---

## Phase 2: Propose Groups

Analyze the file list and propose logical commit groups. Grouping heuristics (in priority order):

1. **By monke artifact type** — if WyrdMonke project files are present, group them by artifact:
   - `monke-docs/hld.md` + related ADRs → one group (design change)
   - `monke-docs/lld/*.md` → one group per component LLD
   - `monke-docs/open-questions.md` → with the design change that surfaced them
   - `monke-status.md` → with the skill that updated it
   - Skill files (`monke-design/`, `monke-implement/`, etc.) → one group (skill update)

2. **By component boundary** — source files that belong to the same LLD component (check `monke-docs/lld/*/` file maps if available):
   - Types/models → "Add/update <component> types (Layer 0)"
   - Signatures → "Add <component> interface skeleton (Layer 1)"
   - Implementation + unit tests → "Implement <component> <sub-problem> (Layer 2)"
   - Integration tests → "Add <component> integration tests (Layer 3)"

3. **By directory/concern** — if no monke context, group by directory proximity and likely concern:
   - Files in same directory with related names → one group
   - Test files with their source files → one group
   - Config/tooling files → one group
   - Documentation → one group

4. **Single-file groups** — files that don't fit anywhere else get their own group

For each group, draft a commit message following the WyrdMonke commit format:

### Commit Format

```
action : description
```

- All lowercase
- Action is the verb: `add`, `update`, `fix`, `remove`, `refactor`, `rename`, etc.
- Then ` : ` (space-colon-space)
- Then a short description of what changed

Examples:
```
add : project scaffold and meta files
update : hld with revised component boundaries
fix : missing pause gate in sync phase 2
remove : deprecated recon fallback logic
refactor : test-run tier resolution
rename : status template to match new schema
```

Present the proposed grouping:

```
⏸ Proposed commit groups:

Group 1: "update : hld skill — add evolve and amend flows"
  M  monke-design/hld.md

Group 2: "update : lld skill — add amend escalation handoff"
  M  monke-design/lld.md

Group 3: "add : monke-ops commit skill"
  A  monke-ops/commit.md

Confirm all / Adjust / Reject?
```

### Adjustment Rules

User can:
- **Merge groups:** "Combine 1 and 2" → merge file lists, draft new combined message
- **Split groups:** "Split group 3 into X and Y" → ask which files go where
- **Move files:** "Move file X from group 1 to group 2"
- **Rename:** "Change group 1 message to ..." → update message
- **Drop:** "Drop group 3" → remove from commit plan (files stay uncommitted)
- **Reorder:** "Commit group 3 first" → change execution order

**⏸ Wait for user to confirm the final grouping before proceeding to commits.**

---

## Phase 3: Log Entry

After groups are confirmed, generate a `monke-log.md` entry before executing commits.

### 3.1 Read Current Version

Read `monke-log.md` and find the latest version number at the top of the log (the first `## <version>` heading, e.g., `0.5`).

### 3.2 Bump Version

Auto-increment the minor version: `0.5` → `0.6`.

Ask the user:

```
⏸ Current version: 0.5
  Next version: 0.6 (minor bump)

  Bump major version instead? (e.g., 0.5 → 1.0) [y/N]
```

If user says yes → bump to next major (e.g., `0.5` → `1.0`, `1.3` → `2.0`).

### 3.3 Generate Entry

Build a structured log entry from the confirmed commit groups:

```markdown
## <version> — <YYYY-MM-DD> — <theme>
```

- **Version:** the bumped version from 3.2
- **Date:** today's date
- **Theme:** a short, punchy tagline synthesized from the commit groups (match the irreverent tone of existing entries)

Categorize each group's files under `### Added` or `### Changed` based on their git status:
- New files (`A`, `??`) → **Added**
- Modified files (`M`) → **Changed**
- Deleted files (`D`) → **Changed** (note removal)
- Renamed files (`R`) → **Changed** (note rename)

Each bullet should reference the file/artifact and include the commit message as context.

### 3.4 Insert & Confirm

Insert the new entry at the top of the log — directly after the header and tagline (`> *every banana...*`), before the first existing version entry. Prepend `---` separator.

Present the draft entry to the user:

```
⏸ Proposed log entry for monke-log.md:

<draft entry>

Confirm / Adjust / Skip?
```

**⏸ Wait for user to confirm the log entry before proceeding to commits.**

If confirmed, write the entry to `monke-log.md`. The log entry file change is included in the first commit group automatically — it does not create its own separate commit.

---

## Phase 4: Execute Commits

Process groups one at a time, in order. For each group:

### 4.1 Confirm

Present the diff summary and commit message for the group:

```
⏸ Commit <N>/<total>: "<commit message>"

Files:
  <file list>

Diff summary:
  <N> files changed, <N> insertions(+), <N> deletions(-)

Commit / Adjust message / Skip / Abort remaining?
```

- **Commit** → stage and commit in a single command (see 4.2)
- **Adjust message** → user provides new message, re-present
- **Skip** → move to next group, files remain unstaged
- **Abort remaining** → stop. Already-committed groups are preserved.

### 4.2 Stage & Commit

Stage and commit in a single command:

```bash
git add <file1> <file2> ... && git commit -m "<message>"
```

For deleted files: `git add` handles deletions. For renamed files: ensure both old and new paths are included.

Report the commit hash. Move to next group.

---

## Phase 5: Report

After all groups processed (committed, skipped, or aborted), present summary:

```
Commit Summary:

  <hash1> "<message 1>" (N files)
  <hash2> "<message 2>" (N files)
  --skip-- "<message 3>" (skipped by user)

Committed: <N>/<total> groups
Skipped: <N> groups
Files still uncommitted: <list, if any>
```

If files remain uncommitted (from skipped/aborted groups or files not included in any group):
- "Uncommitted changes remain. Run `/monke-ops:commit` again to address them."

Do NOT push. If user wants to push, they do it themselves.

---

## Safety Rules

1. **Never force-add ignored files.** If a file is in `.gitignore`, do not stage it. Warn if user asks.
2. **Never commit secrets.** If any file looks like it contains secrets (`.env`, `credentials.*`, files with `API_KEY=`, `SECRET=`, `PASSWORD=` patterns), warn the user and exclude from groups. User must explicitly override.
3. **Never auto-commit without confirmation.** Every group gets its own pause gate.
4. **Preserve staging state.** If user aborts mid-way, only the current group's files are unstaged. Prior commits and other staged files are untouched.
5. **No amend.** This skill creates new commits only. If user wants to amend, they do it manually.
