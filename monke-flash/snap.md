# WyrdMonke Flash:Snap — The Honest Freeze

> **Usage:** /monke-flash:snap
>
> Freezes the MVP at its ugly-but-working moment. Every shortcut gets named, every cut gets dated — the confession that recon reads next.

---

## Arguments

`$ARGUMENTS` parsing:
- `resume:<N>` (optional positional) — skip to phase `<N>` with checkpoint re-read (see drafter §7). Phase numbers: 1=verify scope, 2=build manifest, 3=tag, 4=point forward.
- Default (empty): start from Phase 1.

```
RESUME_PHASE="$(echo "$ARGUMENTS" | grep -oE '^resume:[0-9]+$' | cut -d: -f2)"
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

- `/monke-flash:pulse` has run at least once with flows passing
- `monke-docs/flash/flash-scope.md` exists
- `monke-docs/flash/flash-arch.md` exists

---

## Gate Semantics

Flash runs under light rigor per S9.4/S12. HARD gates (PG-1 in scope, PG-11 in snap) always surface when their firing conditions are met. SOFT gates auto-pass per the adaptive system. TRIGGERED gates fire on their triggers regardless of rigor.

PG-11 lives here (Phase 2) and is HARD, but narrowed to a **form-completion check**: it surfaces only when the manifest has blank sections or placeholders; it auto-confirms silently on a mechanically complete form because scope was already locked at PG-1. Under `/thorough` rigor it surfaces regardless.

---

## Phase 1: Verify Scope

**Resume check (per drafter §7).** If arguments contain `resume:<N>`:
1. Read `monke-status.md` Flash section (the canonical recovery source for snap — this skill does not write a lock-file; `flash-manifest.md` is the deliverable, not a checkpoint).
2. Verify the `Where We Are:` marker names a section of the manifest that matches the expected phase.
3. If parse check passes → skip to Phase `<N>` with prerequisites re-validated inline.
4. If parse check fails → emit warning "resume:<N> specified but checkpoint invalid" and proceed from Phase 1 normally.
5. See Context Death Protocol below for the full recovery spec.

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

⏸ **PG-11 [HARD] — Ship manifest form check.** Fires only when the manifest has blank sections or placeholders (`<TBD>`, `<fill>`, empty bullets). Auto-pass when every section is complete and every blank has been filled. If the form passes mechanical completeness, PG-11 auto-confirms silently — scope was already locked at PG-1 (scope gate); re-confirming intent here is redundant. Surface only on missing sections or explicit user `/thorough` rigor.

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

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Hide shortcuts to make the manifest look cleaner | Refuse. The manifest is the confession — every hardcoded value, every mock, every missing catch lands here. Recon depends on honesty. |
| Skip PG-11 when the manifest has blanks or placeholders | Refuse. PG-11 is HARD on form-incomplete manifests. Blank sections (`<TBD>`, `<fill>`, empty bullets) always surface. Auto-confirmation only applies to mechanically complete manifests. |
| Treat snap as "one more blitz round" (add features, fix bugs) | Refuse. Snap freezes what IS. Fixes belong in pulse or recon R3. |
| Skip the effort-to-production table | Refuse. Recon's roadmap feeds off it. Without sized gaps, the crawl has no shape. |
| Add aspirational items to "What Was Built" | Refuse. Built = committed and working. Every flow listed must be verifiable by running the code. |

---

## Context Death Protocol

**Recovery source of truth.** The canonical recovery state for snap is the Flash section of `monke-status.md` (specifically the `Where We Are:` marker that tracks which manifest section is mid-draft). The file `monke-docs/flash/flash-manifest.md` is the **human-readable MVP manifest** — the deliverable of the flash chain — but it is NOT the recovery checkpoint during drafting. On re-entry mid-draft, read only `monke-status.md` to determine which section to resume. The partial manifest file is informational; the status file decides where work resumes.

**Status line marker:** `Where We Are: flash:snap — drafting manifest (section <N>)` while mid-flight. On Phase 6 (close), the marker is cleared to `Where We Are: flash:snap — manifest frozen (thread=<thread-id>)` until the user explicitly exits the flash chain. This is the canonical format per drafter §8 — no other marker shape is permitted for this skill.

**Recovery detection:** On re-entry, read `monke-status.md` Flash section. If the `Where We Are:` marker names a section mid-draft → resume at that section. If the marker shows manifest complete but PG-11 not logged → re-present the manifest form-completion check (PG-11 fires only if blanks remain; auto-confirms silently when complete). If `monke-status.md` Flash section is absent → prompt the user rather than infer from the partial `flash-manifest.md` file.

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
