# WyrdMonke Implement:Fill — Fill Project Specs Placeholders

> **Usage:** Copy `monke-implement/` to `~/.claude/commands/monke-implement/`. Invoke: `/monke-implement:fill [group]`

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

**⏸ Pause after each group** — show filled values, get confirmation before next group.

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

**⏸ Pause** — show proposed CLAUDE.md changes (if any), get confirmation before applying.

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

## Status Update

On completion, update `monke-status.md`:
- Update Bootstrap section: `- [x] Project-specs filled (N/8 groups) — <date>`
- Bump `Updated:` line
