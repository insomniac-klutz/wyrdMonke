# WyrdMonke Ops:Commit — Confess in Chapters

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
   - `monke-docs/hld.md` + related ADRs → one group (design change) — message: `update : hld <short-theme>`
   - `monke-docs/critic-notes.md` → with HLD changes in the same group (critic notes travel with the HLD they annotate)
   - `monke-docs/lld/*.md` → one group per component LLD — message: `update : lld <component>`
   - `monke-docs/lld/<c>-review.md` → grouped WITH the parent LLD for `<component>` (same commit)
   - `monke-docs/open-questions.md` → with the design change that surfaced them
   - `monke-docs/decisions/*.md` standalone (ADRs without HLD changes) → one group per ADR — message: `adr : <slug>` (e.g. `adr : auth-backend-choice`)
   - `monke-docs/checkpoints/*.md` (phase checkpoint records) → one group per phase — message: `checkpoint : phase <N>`
   - `monke-docs/rage-runs/*.md` (rage-run logs) → one group per rage run — message: `rage : <mode> <short-scope>` (e.g. `rage : buggy auth-service`)
   - `monke-docs/recon/profile-*.md` (data profiles) → one group per profile — message: `profile : <source>` (e.g. `profile : user-events-table`)
   - `monke-docs/recon/*.md` other recon outputs (survey, gaps, reconstructed specs) → one group per artifact — message: `recon : <artifact>`
   - `.monke-config.md` → standalone group — message: `config : rigor` (rigor changes are deliberately isolated from other work)
   - `monke-status.md` → with the skill that updated it (never standalone — status bumps ride along with the artifact change that caused them)
   - Skill files (`monke-design/`, `monke-implement/`, etc.) → one group (skill update) — message: `update : <dir> skill — <theme>`

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
  Proposed commit groups:

  Group 1: "update : hld skill — add evolve and amend flows"
    M  monke-design/hld.md

  Group 2: "update : lld skill — add amend escalation handoff"
    M  monke-design/lld.md

  Group 3: "add : monke-ops commit skill"
    A  monke-ops/commit.md
```

### Adjustment Rules

User can:
- **Merge groups:** "Combine 1 and 2" → merge file lists, draft new combined message
- **Split groups:** "Split group 3 into X and Y" → ask which files go where
- **Move files:** "Move file X from group 1 to group 2"
- **Rename:** "Change group 1 message to ..." → update message
- **Drop:** "Drop group 3" → remove from commit plan (files stay uncommitted)
- **Reorder:** "Commit group 3 first" → change execution order

⏸ **PG-2 [SOFT] — Final grouping confirmed before committing.** Auto-pass when: no `.env`/`credentials`/`*.key`/`*.pem` in grouped files, no test files in src/ groups, and every group has ≥1 file. Rigor: light/standard auto-confirm; thorough surfaces. See design-specs §9.4 + drafter §5 for the canonical rigor-read idiom.
Confirm all / Adjust / Reject?

---

## Phase 3: Log Entry

After groups are confirmed, generate a `monke-log.md` entry before executing commits.

### 3.1 Read Current Version

Read `monke-log.md` and find the latest version number at the top of the log (the first `## <version>` heading, e.g., `0.5`).

### 3.2 Bump Version

Auto-increment the minor version: `0.5` → `0.6`.

⏸ **PG-3 [SOFT] — Version bump confirmed.** Auto-pass when: version string is syntactically valid SemVer (`^\d+\.\d+\.\d+(-[a-z0-9.]+)?$`). Rigor: light/standard auto-confirm; thorough surfaces. See design-specs §9.4 + drafter §5 for the canonical rigor-read idiom.

```
  Current version: 0.5
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
  Proposed log entry for monke-log.md:

  <draft entry>
```

⏸ **PG-4 [SOFT] — Log entry confirmed before writing to monke-log.md.** Auto-pass when: entry parses (ISO date + theme heading + ≥1 bullet). Rigor: light/standard auto-confirm; thorough surfaces. See design-specs §9.4 + drafter §5 for the canonical rigor-read idiom.
Confirm / Adjust / Skip?

If confirmed, write the entry to `monke-log.md`. The log entry file change is included in the first commit group automatically — it does not create its own separate commit.

---

## Phase 4: Execute Commits

Process groups one at a time, in order. For each group:

### 4.1 Confirm

Present the diff summary and commit message for the group:

```
  Commit <N>/<total>: "<commit message>"

  Files:
    <file list>

  Diff summary:
    <N> files changed, <N> insertions(+), <N> deletions(-)
```

⏸ **PG-5 [SOFT] — Per-group commit confirmed.** Auto-pass when: commit message has a type prefix (`feat/fix/chore/docs/refactor/test`), files match the group roster, message length >20 chars. Rigor: light/standard auto-confirm; thorough surfaces. See design-specs §9.4 + drafter §5 for the canonical rigor-read idiom.
Commit / Adjust message / Skip / Abort remaining?

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

⏸ **PG-6 [SOFT] — PR/MR creation confirmed.** Auto-pass when: branch is ahead of remote, no conflicts detected, `gh`/`glab` available. Rigor: light/standard auto-confirm; thorough surfaces. See design-specs §9.4 + drafter §5 for the canonical rigor-read idiom.

```
  Create a Pull Request / Merge Request?

  Platform: GitHub (detected from remote)
  Current branch: <branch>

  Yes / No
```

If **No** → stop. Commits are done, user pushes manually if they want.

### 6.2 Branch Flow

Ask the user whether this is a single PR or a multi-hop chain.

Detect the default branch from `git remote show origin` or fallback to `main`/`master`/`trunk` in that order.

⏸ **PG-7 [SOFT] — PR/MR flow shape picked.** Auto-pass when: N commits on branch ≤ 2 → single; N ≥ 3 AND user didn't pass `--chain` explicitly → single; explicit `--chain` → chain. Rigor: light/standard auto-confirm; thorough surfaces. See design-specs §9.4 + drafter §5 for the canonical rigor-read idiom.

```
  PR/MR flow:

  Current branch: <branch>

  1) Single — <branch> → <default branch>
  2) Chain  — define a multi-hop flow (e.g. feature → develop → main)

  Pick [1/2]:
```

#### Single flow

Ask which remote branch to target:

⏸ **PG-8 [SOFT] — Target branch confirmed.** Auto-pass when: target branch is the project's configured `main_branch` from `.monke-config.md` (fallback: `trunk`/`main` detected via `git remote show origin`). Rigor: light/standard auto-confirm; thorough surfaces. See design-specs §9.4 + drafter §5 for the canonical rigor-read idiom.

```
  Target branch for PR/MR?

  Enter target branch name [default: <default branch>]:
```

Result: a single hop — `[source → target]`.

#### Chain flow

Ask the user to define the full branch chain, starting from the current branch:

```
  Define the branch chain (current branch is the start):

  Example: feature/auth → develop → main

  <branch> → _____ → _____ → ...

  Enter chain (arrow-separated):
```

Validate:
- First branch in the chain must be the current branch. If it isn't → warn and re-ask.
- Minimum 3 branches (2 hops). If only 2 → that's a single flow, confirm switch.
- Each branch name must be a valid git ref. If not → reject and re-ask.

Present the parsed chain:

⏸ **PG-9 [SOFT] — PR/MR chain confirmed.** Auto-pass when: all chain hops have valid base/target mapping (no circular, no gaps). Rigor: light/standard auto-confirm; thorough surfaces. See design-specs §9.4 + drafter §5 for the canonical rigor-read idiom.

```
  PR/MR chain:

  Hop 1: feature/auth → develop
  Hop 2: develop → main

  <N> PRs/MRs will be created, one at a time.
  You review/merge each before the next is raised.
```
Confirm / Adjust / Abort

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

⏸ **PG-10 [SOFT] — PR/MR draft confirmed before push + create.** Auto-pass when: title ≤ 70 chars + body has `## Summary` + `## Test plan` sections. Rigor: light/standard auto-confirm; thorough surfaces. See design-specs §9.4 + drafter §5 for the canonical rigor-read idiom.

```
  PR/MR <N>/<total>:

  Title: "<title>"
  Target: <target> ← <source>

  Body:
  <draft body>
```

- **Confirm** → proceed to push and create
- **Adjust title** → user provides new title, re-present
- **Adjust body** → user provides edits, re-present
- **Abort** → stop. Already-created PRs/MRs are preserved. Remaining hops are skipped.

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

⏸ **PG-11 [SOFT] — Continue to next hop confirmed.** Auto-pass when: previous PR in chain is merged (detected via `gh pr view <num> --json state`). Rigor: light/standard auto-confirm; thorough surfaces. See design-specs §9.4 + drafter §5 for the canonical rigor-read idiom.

```
  Hop <N>/<total> complete: <url>

  Next hop: <next source> → <next target>
  Review and merge the PR/MR above before continuing.
```
Continue to next hop / Abort remaining

- **Continue** → proceed to draft the next hop's PR/MR (back to 6.3.1)
- **Abort remaining** → stop. Already-created PRs/MRs are preserved.

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
3. **Never auto-commit without confirmation.** Every group gets its own confirmation gate.
4. **Preserve staging state.** If user aborts mid-way, only the current group's files are unstaged. Prior commits and other staged files are untouched.
5. **No amend.** This skill creates new commits only. If user wants to amend, they do it manually.

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Commit everything in one big "WIP" commit | Refuse. The whole point of this skill is chaptered confession. If the user insists, ask what theme they want and group by that — but never a single amorphous commit. |
| Skip the grouping phase and use `git add .` + commit | Refuse. `git add .` is how secrets leak and how a rage-run accidentally ships with a feature change. Grouping is the firewall. |
| Amend a prior commit to "clean up" | Refuse. This skill writes new commits only. Amending rewrites history other humans may have pulled; it's a separate deliberate operation the user runs by hand. |
| Force-push after the PR/MR is raised | Refuse. Force-push is never part of this skill's flow. If the user wants to rewrite a published branch, that's a conscious, manual choice. |
| Commit a file in `.gitignore` because the user says "just this once" | Refuse. `.gitignore` exists for a reason; silent override is how credentials get committed. If the user really wants it tracked, they update `.gitignore` first and re-run. |
| Route `.monke-config.md` into the same commit as feature work | Refuse. Rigor changes are deliberately isolated — they change how every subsequent gate behaves, so the commit log must call them out as their own event. |
| Auto-generate a PR body that references files the commit group didn't touch | Refuse. The PR body is synthesized ONLY from the commit messages and files in the hop's range. No speculative additions. |
| Treat PG-N as HARD without checking rigor | Refuse. PG-2..PG-11 are SOFT with mechanical auto-pass conditions — only PG-1 (scope confirm) stays HARD. Rigor level gates surfacing; never hardcode HARD on a SOFT gate. |

---

## Context Death Protocol

**Checkpoint artifacts (written on context pressure):**
- `.monke-commit-state.json` (staged plan) — proposed groups with messages and files, so a later invocation can resume Phase 4 without re-diffing.
- Already-committed group hashes are durable in git history — no separate checkpoint needed for those.
- For PR/MR flow: already-created PR/MR URLs logged to `.monke-commit-state.json` so a later invocation can skip them and resume at the next hop.

**Status line marker:** `Where We Are:` in `monke-status.md` reads `ops:commit — phase <N> (<K>/<M> groups committed, <J>/<P> hops raised)` while mid-flight.

**Recovery detection (on entry):**
- If `.monke-commit-state.json` exists AND `monke-status.md` Resume block names `/monke-ops:commit` → present the unfinished plan, ask to resume or discard.
- If the plan references files that have since changed on disk → warn the user and re-diff the affected groups before resuming.
- If neither present → start fresh from Phase 1.

---

## Status Update

**Read on entry:** `monke-status.md` — note which skill last ran (commit groups the `Updated:` bump with the artifact change that caused it; do NOT double-bump on a standalone commit-only invocation).

**Write on exit:**
- **Success:** bump `Updated:` with today's date + `by /monke-ops:commit`. Append to Gate Audit Log: `- PG-5 COMMIT — N groups committed (<hashes>)`. If PR/MR flow ran, note each URL in the log. Do NOT rewrite "Where We Are" — commit.md is a mechanical writer of history, not a phase tracker.
- **Blocked:** if no changes to commit or prerequisites failed (not in git repo, etc.), add a row to Open Blockers with WHAT/WHY/HOW only if this block represents stalled work; trivial "nothing to commit" exits don't warrant a blocker row.
- **Partial:** write a `Resume:` block naming the committed-groups / remaining-groups / raised-hops / remaining-hops state so the next invocation can pick up cleanly.
