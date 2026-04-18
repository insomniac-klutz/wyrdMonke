# WyrdMonke Status — The North Star

> **Usage:** Copy `monke-status/` to `~/.claude/commands/monke-status/`. Then run `/monke-status:status [action]` inside a project with WyrdMonke bootstrapped.

---

## Arguments

`$ARGUMENTS` parsing:
- Single positional argument: `show` | `rebuild` | `next` | `blocked` | `resume`
- Default (empty): `show`
- `show` — display current `monke-status.md` with a one-paragraph summary
- `rebuild` — scan all artifacts on disk, regenerate `monke-status.md` from truth
- `next` — answer "what should I do next?" with the single next actionable skill
- `blocked` — show only open blockers and what they're holding up
- `resume` — read the last `Updated:` line + in-progress markers, present "resume from X?"

```
ACTION="${ARGUMENTS:-show}"
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

Verify `monke-status.md` exists at project root (not this skill file — the project's status dashboard).

- If it exists → read it
- If it doesn't exist → check if `monke-docs/` exists:
  - Yes → create `monke-status.md` from `monke-docs/status-template.md`, then proceed
  - No → tell user: "No WyrdMonke project found. Run `/monke-init` first." Stop.

**Sole-writer rule:** during parallel teammate work, only the lead/orchestrator writes `monke-status.md`. Teammates write per-component files; the lead reconciles. This skill, when invoked by the lead, is the canonical writer. If invoked by a teammate during parallel execution, display-only — do not write.

---

## Action: `show`

1. Read `monke-status.md` from project root.
2. Present the full dashboard to the user, including:
   - **Rigor level** from the header line
   - **Components table** with per-component maturity, IL gates passed, LLD confidence, override flags
   - **Gate audit log** (most recent session's gate activity)
3. Add a one-paragraph summary at the top:
   - Current phase
   - Component counts by maturity (e.g., "3 at IL-3, 2 at IL-1, 1 missing")
   - Number of open blockers
   - Time since last update
   - Rigor level

---

## Action: `rebuild`

Scan all artifacts on disk and regenerate `monke-status.md` from truth. This recovers from stale or missing status files.

### Scan Sequence

| Step | Artifact | What to check | Status derived |
|------|----------|---------------|----------------|
| 1 | `.monke-config.md` | Exists? Rigor set? | Rigor level (light/standard/thorough) |
| 2 | `monke-docs/project-specs.md` | Any `<<<` remaining? | Bootstrap: complete or incomplete (list unfilled sections) |
| 3 | `CLAUDE.md` or `monke-CLAUDE.md` | Exists? Has `<<<` remaining? Has Agent Teams section? | CLAUDE.md: merged, template-only, or missing |
| 4 | `monke-docs/hld.md` | File size > 1 line? Which sections (S1-S8) have content? | HLD: not started, L1, L2, L3, or complete |
| 5 | `monke-docs/lld/*.md` | For each file: Confidence header (auto-generated/reviewed/verified)? ADaPT tree? Signatures? Test plan? | Per-component LLD status with confidence |
| 6 | `monke-docs/decisions/*.md` | Count, list titles + status (proposed/accepted/superseded) | ADR inventory |
| 7 | `monke-docs/open-questions.md` | Parse OQ entries. Count by status. Extract `Blocks:` fields. | Open blockers |
| 8 | `monke-docs/checkpoints/*.md` | Which phases exist? Parse checkbox state. | Phase checkpoint status |
| 9 | Source files matching LLD file maps | For each component: files exist? Run IL gate commands from project-specs S3.2 (per-container). Detect maturity: missing / pre-L0 / IL-0 / IL-1 / IL-2 / IL-3. | Per-component maturity + current layer |
| 10 | Test files matching LLD test plans | Glob for test files per container's test dir (from project-specs S4.1). Classify by tier. | Test tier coverage |

### Rebuild Rules

- If a scan step fails (file missing, parse error, command crash), record that component as "unknown — scan failed" with the error message rather than guessing.
- After scanning, write the regenerated `monke-status.md` to project root.
- Preserve manual override flags (H8) — if a component was marked `Override? YYYY-MM-DD manual` in the previous status file, keep that override and flag it for re-verification.
- Show a diff summary: what changed vs the previous status file.
- **Do NOT run any IL gate commands that could modify files or have side effects.** Read-only scan only. If you need to verify a gate, note it as "needs verification" and suggest the user run `/monke-test:test-run`.

### Final Step: CDP Compliance Audit

Run this audit AFTER all scan steps and BEFORE writing `monke-status.md`. Walk the skill tree (`monke-design/*.md`, `monke-recon/*.md`, `monke-flash/*.md`, `monke-implement/*.md`, `monke-test/*.md`, `monke-rage/*.md`, `monke-seer/*.md`, `monke-ops/*.md`, `monke-status/*.md`, plus root skills: `monke.md`, `monke-init.md`, `monke-sync.md`). For each skill file:

1. Verify the file contains a heading `## Context Death Protocol`.
2. Under that heading, verify these canonical elements are present (as sub-sections, bullets, or explicit lines):
   - `Checkpoint artifact(s):` — the file(s) where mid-flight state is captured.
   - `Status line marker:` — the `Where We Are:` shape this skill writes.
   - `Recovery detection:` — how a fresh session identifies this skill is resuming.
3. Missing any of the three → flag with `[cdp-warn] <skill-path> — missing <element>`.
4. Missing the entire CDP heading → flag with `[cdp-fail] <skill-path> — no Context Death Protocol section`.

Emit a compliance report section in `monke-status.md`:
```
## CDP Compliance (updated <ISO-date> via status:rebuild)

Total skills scanned: <N>
Compliant: <M>
Warnings: <K> (see below)
Failures: <J> (see below)

<warning/failure lines>
```

When compliance is 100%, simply write: `All <N> skills CDP-compliant.` Skip the details section.

**Exemptions.** The following files are NOT skills and do NOT require CDP (skip them in the scan):
- `monke-drafter.md` (meta — describes the skeleton itself)
- `monke-phil.md` (philosophy document, not an executable skill)
- `monke-log.md`, `monke-fut.md` (changelog / gaps, not skills)
- `CLAUDE.md`, `monke-CLAUDE.md`, `README.md`, `monke-mermaid.mmd` (project-level docs)

Also skip anything under `monke-docs/` (specs, not skills).

⏸ **PG-R [HARD] — Rebuilt status confirmed before writing over monke-status.md.** Always surfaces. Never skippable. A rebuilt dashboard overwrites the canonical status file — confirm the diff before committing.
Confirm / Adjust / Reject?

---

## Action: `next`

Determine the single next actionable step. With per-component routing (fast onboarding model), `next` picks the lowest-maturity component first, then routes to its pipeline entry layer.

```
1. Is project-specs filled? (no <<< remaining in Stack/Commands/Bindings)
   No  → "Run /monke to auto-detect stack and populate project-specs."
   Yes ↓

2. Does CLAUDE.md exist with no <<< remaining?
   No  → "Run /monke-init to set up CLAUDE.md."
   Yes ↓

3. Components table populated?
   No  → "Run /monke to do fast onboarding — auto-detect components and maturity."
   Yes ↓

4. Any components at `missing` or `pre-L0`?
   Yes → Pick lowest-maturity by dependency order.
         "Run /monke-design:lld <component> (or /monke to auto-route)."
   No  ↓

5. Any components with LLD Confidence = auto-generated (not reviewed)?
   Yes → Pick first.
         "Run /monke-design:lld <component> (review mode) to verify auto-generated LLD."
   No  ↓

6. Any components at IL-0, IL-1, IL-2?
   Yes → Pick lowest by dependency order.
         "Run /monke-implement:implement <component> <next-layer>."
   No  ↓

7. Open blockers (OQs with Blocks:)?
   Yes → "Resolve blocking OQs first: /monke-design:oq triage"
   No  ↓

8. All components IL-3?
   No  → Find incomplete. "Run /monke-implement:implement <component> <layer>."
   Yes ↓

9. Phase checkpoint done?
   No  → "Run /monke-implement:checkpoint <phase> for phase sign-off."
   Yes → "All phases complete. Next: /monke rage for hardening, or ship."
```

Present: the recommended skill invocation, why it's next, and what it depends on.

---

## Action: `blocked`

1. Read `monke-docs/open-questions.md`.
2. Filter to entries with `Status: open` that have a `Blocks:` field.
3. For each blocker, show:
   - OQ ID + question
   - What it blocks (LLD, implementation, or test for which component)
   - Options so far (if any)
4. If no blockers found, say so and suggest `/monke-status:status next` instead.

---

## Action: `resume`

Recover from a dropped session. Read the last known progress markers and present a "resume from X?" prompt.

1. Read `monke-status.md`.
2. Extract:
   - **`Updated:` line** — last skill that wrote and when
   - **In-progress markers** — any row/checkbox with status "in progress", "partial", or "3/7" style fractional progress
   - **Gate audit log** — last gate event (most recent line)
   - **Components table** — any row with current layer not at IL-3
3. Present:
   > "Last session ended: `<Updated line>`
   >
   > In progress:
   > - `<component>`: L2 at 3/7 functions (was working on `<function_name>`)
   > - PG-9 LLD review pending for `<component>`
   >
   > Resume from: `/monke-implement:implement <component> 2`?
   >
   > Or: `/monke` to re-route from current state."
4. If nothing is in-progress (everything is clean or complete), fall through to `next` action.

---

## Status File Format

When creating or rebuilding `monke-status.md`, use this structure (matches `monke-docs/status-template.md`):

```markdown
# monke-status.md
Project: <from project-specs S1> | Updated: <date> by <skill that last wrote>
Rigor: <level> (from `.monke-config.md`)

## Where We Are
Phase: **<current phase description>**
Next action: `<skill invocation>`
Blockers: <count and summary, or "none">

## Components
| Component | Container | Detected Maturity | Current Layer | IL Gates Passed | LLD Confidence | Status | Override? |
|-----------|-----------|-------------------|---------------|-----------------|----------------|--------|-----------|

## Gate Audit Log (this session)
- <one line per gate event>

## Bootstrap / HLD / LLDs / Implementation / Test Gates / Phase Checkpoints / Open Blockers / Decisions
(tables per template)
```

---

## Status Update Protocol

**Every skill in the WyrdMonke package follows this protocol:**

### Updated Line Format

The canonical format is:
```
Updated: YYYY-MM-DD by /<skill-dir>:<skill-name>
```
Example: `Updated: 2026-04-14 by /monke-design:hld`

If an orchestrator delegates to a skill, the **executing skill** bumps the line (not the orchestrator). Only one `Updated:` line — replace, don't append.

### Gate Audit Log Entries

When a gate fires or auto-passes, append a single line to the Gate Audit Log section:
```
- <GATE-ID> <GATE-NAME> — <outcome> (<reason>)
```
Examples:
```
- PG-2 CONTAINERS — auto-confirmed (quality checks passed, rigor=standard)
- PG-3 COMPONENTS — human confirmed (2 ambiguous boundaries)
- IL-2 component=auth-service — auto-passed (coverage 87%)
- PG-11 SHIP — human confirmed
```

### Component Override (H8)

When a user overrides a detected maturity at a hard gate:
1. Update the component's `Detected Maturity` column to the override value.
2. Set `Override?` column to `YYYY-MM-DD manual`.
3. Log to Gate Audit Log: `- OVERRIDE <component> — <from-maturity> → <to-maturity> (user override)`.
4. On next `rebuild`, preserve the override flag and add a note to re-verify when possible.

### Entry/Exit Rules

1. **On entry:** Read `monke-status.md`. Verify prerequisites for this skill are met. If not, tell the user what to do first (use the `next` waterfall logic).

2. **On exit (success):** Update the relevant row/checkbox in `monke-status.md`. Bump the `Updated` line with current date and skill name. Append gate events to Gate Audit Log. Recalculate "Where We Are" and "Next action" using the waterfall.

3. **On exit (blocked):** Add to "Open Blockers" table. Update "Where We Are" to reflect the block. Suggest `/monke-design:oq triage`.

4. **On exit (partial):** Update progress (e.g., "3/7 functions" in L2 column). Keep status as "in progress". Note resume point so the next invocation can pick up via `/monke-status:status resume`.

### Parallel Work Rule

During parallel teammate work (multiple agents editing per-component files simultaneously), **only the lead/orchestrator writes `monke-status.md`**. Teammates write to per-component artifacts (LLD files, test files, source files) and report completion via `TaskUpdate`. The lead reconciles all teammate outputs into a single `monke-status.md` update at the end of the wave.

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Invent components, maturities, or gate outcomes not found on disk | Refuse. `rebuild` regenerates from truth — if an artifact is missing, mark it "unknown — scan failed," don't guess to make the dashboard look complete. |
| Run IL gate commands that mutate files during `rebuild` | Refuse. Read-only scan only. If a gate needs verification, note "needs verification" and route the user to `/monke-test:test-run` — never silently re-run build/test commands. |
| Overwrite manual override flags (H8) during `rebuild` | Refuse. An `Override? YYYY-MM-DD manual` entry is a deliberate human decision; preserve it and flag for re-verification, don't silently replace it with a detected value. |
| Write to `monke-status.md` while running as a teammate in a parallel wave | Refuse. Sole-writer rule — only the lead reconciles the final status. Teammates write per-component artifacts and report via `TaskUpdate`. |
| Collapse the `next` waterfall to "just tell me one thing" | Refuse. The waterfall traverses in order because earlier gates block later ones; shortcutting produces a recommendation that assumes prerequisites are met. Always walk the ladder. |
| Treat `resume` as "re-run whatever was last" | Refuse. Read the Updated line, in-progress markers, and the gate audit log — present the resume-from prompt, let the human confirm the target before dispatching. |
| Pretend `monke-status.md` doesn't exist when `rebuild` is asked | Refuse. If it exists, read it, diff against the rebuild, and present the diff; don't silently wipe in favor of a fresh scan. That loses override flags and history. |

---

## Context Death Protocol

**Checkpoint artifacts (written on context pressure):**
- `monke-status.md.rebuild-draft` — partial rebuild scan results with per-step progress (which of the 10 scan steps have completed) so a re-entry can resume the scan without restarting it.
- The authoritative `monke-status.md` is itself the durable state for `show`/`next`/`blocked`/`resume` actions — those actions are read-only and reach context death rarely; on death they simply re-run.

**Status line marker:** `Where We Are: status:rebuild — <step N/10>` during an active rebuild. Other actions (show/next/blocked/resume) are short-lived and do not leave a mid-flight marker.

**Recovery detection (on entry):**
- If `monke-status.md.rebuild-draft` exists AND `Where We Are` names an in-flight rebuild → present the unfinished draft, ask to resume from the next scan step or discard and restart.
- If draft exists but marker cleared → compare draft against current `monke-status.md`; if draft is newer, offer resume; if equal, clean up the draft.
- If neither present → run the requested action fresh.

---

## Status Update

This skill is itself the canonical writer of `monke-status.md`. Its own status protocol is the Status Update Protocol above (applied recursively). Specifically:

**Read on entry:** always `monke-status.md` itself (the file this skill owns). If it doesn't exist, fall through to the Prerequisites rebuild-or-init flow.

**Write on exit:**
- **`show` / `next` / `blocked` / `resume` (read-only actions):** do NOT bump `Updated:`. These are display-only; writing would create spurious "updated by status" churn.
- **`rebuild` (mutating action):** bump `Updated:` with today's date + `by /monke-status:status rebuild`. Append to Gate Audit Log: `- PG-R REBUILD — N components / K blockers / <diff summary>`. Preserve H8 overrides.
- **Blocked:** if `monke-status.md` can't be read/written (permissions, disk), surface the error with WHAT/WHY/HOW — don't swallow it.
- **Partial:** during a long rebuild, write the `Resume:` block referring to the scan-step ledger so the next invocation can pick up.
