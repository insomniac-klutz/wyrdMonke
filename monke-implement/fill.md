# WyrdMonke Implement:Fill — Bind Abstractions to Reality

> **Usage:** `/monke-implement:fill [group]`
>
> Walks the eight placeholder groups in `project-specs.md`, suggests detected values from the actual codebase, and binds the abstract stack to the concrete one — one group, one pause, one confirmation at a time.

---

## Arguments

`$ARGUMENTS` parsing:
- Single positional: group number `1`–`8` or `all`
- Default (empty): `all`
- Example: `/monke-implement:fill 3` (fill only Group 3: Dependencies)

```
GROUP="${ARGUMENTS:-all}"
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

Read `monke-status.md`. Verify `monke-docs/project-specs.md` exists.

If called from `/monke-design:tinker`, a detection summary (languages, frameworks, tools found) should be available in conversation context. Use it to suggest values.

If called standalone: run detection first (check for `package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, etc. and derive suggestions).

---

## How This Works

For each placeholder group:
- **Existing codebase:** suggest a value based on detection. Show suggestion, ask user to confirm or edit.
- **Fresh init:** ask the user directly.
- Replace `<<<placeholder>>>` text with confirmed value using Edit tool.
- `<<<project_name>>>` appears multiple times — use `replace_all: true`.

⏸ **SKILL-GATE:group-confirm [SOFT] — Group <N> values confirmed.** Pause after each group — show filled values, get confirmation before next group.
Auto-pass when: every placeholder in the group was resolved by mechanical detection (manifest parse, `.env.example` read, `project-specs.md` cross-reference) AND the detected value matches the abstract concept unambiguously (one obvious candidate per placeholder).
Rigor: surfaces under `thorough`; auto-confirms under `light`/`standard` when condition holds. **Group 2 (stack bindings) is HARD under `thorough` only** — the locked stack drives every downstream gate.

Groups 1–8 each fire this gate independently with their own scope:
- Group 1 (Identity): auto-pass when directory name or manifest `name` is unambiguous.
- Group 2 (Stack & Design): auto-pass when every locked layer has exactly one detected candidate AND project-specs S2 mapping is unambiguous.
- Group 3 (Dependencies): auto-pass when manifest + lock file are present and package manager commands derive directly.
- Group 4 (Environment): auto-pass when `.env.example` parses cleanly; surfaces if any variable lacks a comment or has an unknown format.
- Group 5 (Directory Structure): auto-pass when source/test dirs exist and match convention; surfaces if dirs are empty or ambiguous.
- Group 6 (Tooling & CI): auto-pass when linter + CI config both parse; surfaces if either is absent or custom.
- Group 7 (IL Gates): auto-pass when stack has canonical commands (e.g., `tsc --noEmit`, `cargo check`); surfaces for custom toolchains.
- Group 8 (Test Bindings): auto-pass when test framework + runner + directories all detected and coverage tool config present; surfaces for new / missing frameworks.

---

## Group 1 — Identity

| Placeholder | How to suggest |
|---|---|
| `<<<project_name>>>` | Directory name, or `name` from manifest (package.json, Cargo.toml, pyproject.toml). Appears multiple times — replace all. |

---

## Group 2 — Stack & Design Bindings (S2, S10)

| Placeholder | How to suggest |
|---|---|
| `<<<stack_bindings_table>>>` | Map detected stack to abstract concepts. Example: `\| Package manager \| pnpm \|` |
| `<<<locked_stack_table>>>` | Format: `\| Layer \| Technology \| Notes \|`. Example: `\| Runtime \| Node 20 \| LTS \|` |
| `<<<supported_languages_table>>>` | Detected languages. Format: `\| TypeScript \| Backend + Frontend \| Native \| Vitest \|` |
| `<<<stack_enforcement_rules>>>` | Derive from stack. Example: "All backend code in TypeScript. No direct SQL — use Prisma." |
| `<<<design_specs_modifications>>>` | Ask user. Default: "None — using design-specs.md as-is" |

---

## Group 3 — Dependencies (S3)

| Placeholder | How to suggest |
|---|---|
| `<<<dependency_manifest_and_lockfile>>>` | Detect manifest + lock. Example: "`package.json` + `pnpm-lock.yaml`" |
| `<<<package_manager_commands>>>` | Derive from package manager. Example: `\| Install all \| pnpm install \|` |

---

## Group 4 — Environment (S4)

| Placeholder | How to suggest |
|---|---|
| `<<<env_files>>>` | List detected env files. Example: `\| .env \| Runtime secrets \| No \|` |
| `<<<env_variables>>>` | Parse `.env.example` if exists, otherwise ask. |

---

## Group 5 — Directory Structure (S5)

| Placeholder | How to suggest |
|---|---|
| `<<<source_tree>>>` | Run `ls` or `tree` on source dirs. Present as code-fenced tree. |
| `<<<test_tree>>>` | Same for test directories. |

---

## Group 6 — Tooling & CI (S6–S7)

| Placeholder | How to suggest |
|---|---|
| `<<<additional_tooling>>>` | Primary linter/formatter detected (e.g., "ESLint + Prettier"). Becomes S6 heading. |
| `<<<ci_cd_pipeline>>>` | Read workflow files if they exist. Summarize. If none, "TBD". |

---

## Group 7 — Implementation Gates (S8)

| Placeholder | How to suggest |
|---|---|
| `<<<il_gate_commands>>>` | Derive from stack. Example: `\| IL-0 \| Imports resolve \| tsc --noEmit \|` |

---

## Group 8 — Test Bindings (S9)

| Placeholder | How to suggest |
|---|---|
| `<<<test_framework>>>` | Detect from devDependencies or config |
| `<<<test_runner_command>>>` | Derive: `pnpm vitest run`, `pytest`, `cargo test` |
| `<<<unit_test_dir>>>` | Detect or suggest: `tests/unit/` |
| `<<<integration_test_dir>>>` | Detect or suggest: `tests/integration/` |
| `<<<system_test_dir>>>` | Suggest: `tests/system/` |
| `<<<object_factory_library>>>` | Detect or suggest based on language |
| `<<<http_mock_library>>>` | Detect or suggest |
| `<<<async_test_support>>>` | Detect or suggest |
| `<<<db_fixture_strategy>>>` | Ask user. Suggest: "Test containers with fresh schema per suite" |
| `<<<coverage_tool>>>` | Detect from config |
| `<<<coverage_threshold>>>` | Ask user. Suggest: `80%` |
| `<<<shared_fixture_file>>>` | Detect or suggest |

---

## CLAUDE.md Sync

After filling project-specs, check `CLAUDE.md` for consistency:

1. Read `CLAUDE.md`. Compare stack references (languages, frameworks, tools, commands) against the values just confirmed in project-specs groups 2, 3, 6, 7, 8.
2. If `<<<project_description>>>` or `<<<project_invariants>>>` placeholders still exist — fill them using the same detect-suggest-confirm flow as tinker Phase 3.
3. If stack references are stale (e.g., CLAUDE.md says "npm" but project-specs now says "pnpm"), surface each mismatch and offer to update.
4. If CLAUDE.md is already consistent — skip silently.

⏸ **SKILL-GATE:claudemd-sync [HARD] — CLAUDE.md sync confirmed.** Show proposed CLAUDE.md changes (if any), get confirmation before applying.
Always surfaces when any change is proposed. CLAUDE.md is durable project instruction; human must review every stack-reference edit before it lands. When CLAUDE.md is already consistent (no changes proposed), this gate is skipped silently with an audit log entry.

---

## Finalize

1. Verify no `<<<` patterns remain:
   ```bash
   grep -n '<<<' monke-docs/project-specs.md CLAUDE.md
   ```
   If any remain, surface them.

2. Show summary: groups filled, placeholders replaced, CLAUDE.md changes (if any), any left as TBD.

3. Check `.gitignore` — verify `CLAUDE.md` and `.claude/` are listed. If either is missing, warn the user:

   > "⚠ `CLAUDE.md` and `.claude/` contain project-specific AI instructions and should not be committed to your repo. Add them to `.gitignore`:"
   > ```
   > CLAUDE.md
   > .claude/
   > ```

---

## Context Death Protocol

**Checkpoint artifacts:** in-progress edits to `monke-docs/project-specs.md` (per-group replacements) and `CLAUDE.md` (description, invariants, stack references). `monke-status.md` Bootstrap section records `N/8 groups filled`.
**Status line marker:** `Where We Are:` reads `implement:fill — group <N> (<pending | confirming>)` while mid-flight.
**Recovery detection:** On re-entry, grep `<<<` in `project-specs.md`: if any remain → resume at the group containing the first remaining placeholder. If all placeholders filled but CLAUDE.md still has stale references → resume at CLAUDE.md Sync. If fully complete → tell user "All 8 groups already filled" and exit.

---

## Status Update

**Read on entry:** `monke-status.md` — check Bootstrap section for per-group progress; if called by `/monke-design:tinker`, detection summary is in context.
**Write on exit:**
- Success: `- [x] Project-specs filled (N/8 groups) — <date>` in Bootstrap section; bump `Updated:` line with date + `by /monke-implement:fill`.
- Blocked: add row to Open Blockers with WHAT (which placeholder) / WHY (detection unclear or user deferred) / HOW (suggested next action).
- Partial: record `Resume:` block naming which group paused and what was the last confirmed value.

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Leave placeholders as TBD without asking | Refuse. Detect, suggest, confirm — every placeholder gets a real attempt. |
| Fill placeholders without user confirmation | Refuse. Present the suggestion, wait for yes. Every group gets a pause. |
| Modify project-specs outside of fill's scope | Refuse. Fill replaces `<<<placeholders>>>` — it doesn't rewrite spec sections. |
| Skip CLAUDE.md sync after filling project-specs | Warn. Stack references may diverge. Offer to sync. |
| Force-add `.claude/` or `CLAUDE.md` to git | Refuse. Warn if not in `.gitignore`, but never stage them. |
