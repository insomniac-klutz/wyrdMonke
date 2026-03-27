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

---

## Phase 6: Raise the Flag

After reporting, ask the user if they want to raise a PR or MR.

### 6.1 Platform & Intent

Detect the remote hosting platform:

```bash
git remote get-url origin
```

- If URL contains `github.com` → **GitHub** (Pull Request)
- If URL contains `gitlab.com` or a known GitLab self-hosted domain → **GitLab** (Merge Request)
- If neither or ambiguous → ask: "Is this GitHub (PR) or GitLab (MR)?"

Present:

```
⏸ Create a Pull Request / Merge Request?

  Platform: GitHub (detected from remote)
  Current branch: <branch>

  Yes / No
```

If **No** → stop. Commits are done, user pushes manually if they want.

**⏸ Wait for user confirmation before proceeding.**

### 6.2 Branch Flow

Ask the user whether this is a single PR or a multi-hop chain.

Detect the default branch from `git remote show origin` or fallback to `main`/`master`/`trunk` in that order.

```
⏸ PR/MR flow:

  Current branch: <branch>

  1) Single — <branch> → <default branch>
  2) Chain  — define a multi-hop flow (e.g. feature → develop → main)

  Pick [1/2]:
```

**⏸ Wait for user to pick.**

#### Single flow

Ask which remote branch to target:

```
⏸ Target branch for PR/MR?

  Enter target branch name [default: <default branch>]:
```

**⏸ Wait for user to confirm target branch.**

Result: a single hop — `[source → target]`.

#### Chain flow

Ask the user to define the full branch chain, starting from the current branch:

```
⏸ Define the branch chain (current branch is the start):

  Example: feature/auth → develop → main

  <branch> → _____ → _____ → ...

  Enter chain (arrow-separated):
```

Validate:
- First branch in the chain must be the current branch. If it isn't → warn and re-ask.
- Minimum 3 branches (2 hops). If only 2 → that's a single flow, confirm switch.
- Each branch name must be a valid git ref. If not → reject and re-ask.

Present the parsed chain:

```
⏸ PR/MR chain:

  Hop 1: feature/auth → develop
  Hop 2: develop → main

  <N> PRs/MRs will be created, one at a time.
  You review/merge each before the next is raised.

  Confirm / Adjust / Abort
```

**⏸ Wait for user to confirm the chain.**

Result: an ordered list of hops — `[A → B, B → C, ...]`.

### 6.3 Execute Hops

Process each hop in order. For every hop (single flow = one hop, chain flow = N hops):

#### 6.3.1 Draft PR/MR Message

Synthesize a PR/MR title and body from the committed groups:

**Title:** short, under 70 characters — derive from the commit messages. If single commit, use its message. If multiple, synthesize a theme. For chain hops beyond the first, prefix with the hop context: `[develop → main]`.

**Body:** structured summary:

```markdown
## Summary
- <bullet per commit group — what changed and why>

## Changes
- `<hash>` <commit message>
- `<hash>` <commit message>
- ...
```

For chain hops, append a chain context section:

```markdown
## Chain
- Hop <N>/<total>: `<source>` → `<target>`
- Previous: <link to prior PR/MR, if any>
```

Present the draft:

```
⏸ PR/MR <N>/<total>:

  Title: "<title>"
  Target: <target> ← <source>

  Body:
  <draft body>

  Confirm / Adjust title / Adjust body / Abort
```

- **Confirm** → proceed to push and create
- **Adjust title** → user provides new title, re-present
- **Adjust body** → user provides edits, re-present
- **Abort** → stop. Already-created PRs/MRs are preserved. Remaining hops are skipped.

**⏸ Wait for user to confirm the PR/MR draft.**

#### 6.3.2 Push & Create

Push the source branch to the remote:

```bash
git push -u origin <source-branch>
```

Then create the PR/MR:

**GitHub:**
```bash
gh pr create --head <source-branch> --base <target-branch> --title "<title>" --body "<body>"
```

**GitLab:**
```bash
glab mr create --source-branch <source-branch> --target-branch <target-branch> --title "<title>" --description "<body>"
```

If the CLI tool (`gh` or `glab`) is not installed → tell the user: "Install `gh` (GitHub CLI) / `glab` (GitLab CLI) to create PR/MRs from the terminal." Stop.

If push or creation fails → report the error, suggest manual steps. Do not retry automatically.

On success, report the PR/MR URL:

```
PR/MR created: <url>
```

#### 6.3.3 Chain Gate (chain flow only)

If there are remaining hops, pause:

```
⏸ Hop <N>/<total> complete: <url>

  Next hop: <next source> → <next target>
  Review and merge the PR/MR above before continuing.

  Continue to next hop / Abort remaining
```

- **Continue** → proceed to draft the next hop's PR/MR (back to 6.3.1)
- **Abort remaining** → stop. Already-created PRs/MRs are preserved.

**⏸ Wait for user to confirm before proceeding to the next hop.**

### 6.4 Chain Summary (chain flow only)

After all hops are processed, present the full chain result:

```
Chain Summary:

  Hop 1: feature/auth → develop — <url1>
  Hop 2: develop → main         — <url2>
  --aborted-- staging → prod    (skipped by user)

  Created: <N>/<total> PRs/MRs
```

---

## Safety Rules

1. **Never force-add ignored files.** If a file is in `.gitignore`, do not stage it. Warn if user asks.
2. **Never commit secrets.** If any file looks like it contains secrets (`.env`, `credentials.*`, files with `API_KEY=`, `SECRET=`, `PASSWORD=` patterns), warn the user and exclude from groups. User must explicitly override.
3. **Never auto-commit without confirmation.** Every group gets its own pause gate.
4. **Preserve staging state.** If user aborts mid-way, only the current group's files are unstaged. Prior commits and other staged files are untouched.
5. **No amend.** This skill creates new commits only. If user wants to amend, they do it manually.
