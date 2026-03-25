# WyrdMonke Flash:Snap — Good enough. Freeze it.

> **Usage:** Copy `monke-flash/` to `.claude/commands/monke-flash/`. Invoke: `/monke-flash:snap`

---

## Arguments

`$ARGUMENTS` parsing:
- None.

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

- `/monke-flash:pulse` has run at least once with flows passing
- `monke-docs/flash/flash-scope.md` exists
- `monke-docs/flash/flash-arch.md` exists

---

## Phase 1: Verify Scope

Read `flash-scope.md`. For each IN-scope flow:

| Flow | Status | Notes |
|------|--------|-------|
| Flow 1 | working / cut / partial | ... |
| Flow 2 | ... | ... |
| Flow 3 | ... | ... |

If any flow was cut during blitz/pulse, note **why**. That's not a failure — it's honest scope management.

**Present the table.** Ask: "Anything else to address before we freeze?"

If user says yes -> suggest running `/monke-flash:pulse` for one more pass.
If user says no -> proceed.

---

## Phase 2: Build the Manifest

Write `monke-docs/flash/flash-manifest.md`:

```markdown
# Flash Manifest — <project name>

Frozen: <date> by /monke-flash:snap

## What Was Built

### Flows
| Flow | Status | Key Files |
|------|--------|-----------|
| <flow 1> | working | <files> |
| <flow 2> | working | <files> |
| <flow 3> | cut — <reason> | — |

### Components
- <component>: <what it does> (<files>)
- ...

## What Was Cut
- <feature> — <reason it was cut>
- ...

## Known Shortcuts
- <shortcut>: <what was done instead of the right thing>
- Hardcoded values: <list>
- Missing error handling: <where>
- No tests: entire codebase
- Mock/seed data: <what's fake>
- ...

## Known Bugs
<From pulse feedback marked "defer">
- <bug description>
- ...

## Stack As-Built

| Layer | Planned | Actual | Notes |
|-------|---------|--------|-------|
| ... | ... | ... | ... |

## Effort to Production

| Area | Estimate | What's Needed |
|------|----------|---------------|
| Error handling | small/medium/large | <details> |
| Tests | small/medium/large | <details> |
| Auth/security | small/medium/large | <details> |
| Data migration | small/medium/large | <details> |
| UI polish | small/medium/large | <details> |
| Monitoring/ops | small/medium/large | <details> |
```

**Be honest.** The manifest is the confession. Every shortcut, every hardcoded string, every "I'll fix that later." This is the primary input for monke-recon.

---

## Phase 3: Tag It

Suggest a git tag:

> "Tag this as `flash-<project-name>-mvp`? This marks the MVP state before any production hardening."

If user confirms:
```bash
git tag flash-<project-name>-mvp
```

If user declines — that's fine, skip it.

---

## Phase 4: Point Forward

Tell the user what comes next:

> "The flash build is frozen. You've got a working MVP with honest shortcuts documented in `flash-manifest.md`.
>
> To start the production crawl — turning this 80% into 100% — run:
>
> `/monke-recon:survey` — this scans what you've built, then `/monke-recon:reconstruct` reverse-engineers a proper HLD from your flash code. The full recon pipeline (gaps, oqs, roadmap) maps every shortcut to a fix, and the roadmap connects you back to the existing monke pipeline for the disciplined last mile.
>
> The manifest tells recon exactly where the bodies are buried."

---

## Status Update

On completion, update `monke-status.md` Flash section:
```
## Flash
Phase: **Snap — MVP frozen**
Brief: `monke-docs/flash/flash-brief.md`
Scope: `monke-docs/flash/flash-scope.md`
Arch: `monke-docs/flash/flash-arch.md`
Manifest: `monke-docs/flash/flash-manifest.md`
Tag: `flash-<name>-mvp`
Flows: <N> working | <M> cut
Next: `/monke-recon:survey` (production crawl)
```
Bump `Updated:` to today, `by /monke-flash:snap`
