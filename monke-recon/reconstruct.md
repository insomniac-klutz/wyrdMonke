# WyrdMonke Recon:Reconstruct — Blueprints from the Rubble

> **Usage:** Copy `monke-recon/` to `.claude/commands/monke-recon/`. Invoke: `/monke-recon:reconstruct [mode]`

---

## Arguments

`$ARGUMENTS` parsing:
- `hld` (default) — reverse-engineer the High-Level Design only
- `lld <component>` — reverse-engineer a single component's Low-Level Design
- `all` — HLD + all component LLDs in one pass
- `resume:<N>` (optional trailing positional) — skip to phase `<N>` with checkpoint re-read (see drafter §7). Phase numbers: 1=HLD reconstruction, 2=LLD reconstruction, 3=agent teams review.
- Example: `/monke-recon:reconstruct lld auth-service` or `/monke-recon:reconstruct hld resume:1`

```
MODE="${1:-hld}"
COMPONENT="${2:-}"
RESUME_PHASE="$(echo "$@" | grep -oE 'resume:[0-9]+' | head -n1 | cut -d: -f2)"
```

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

Read `monke-status.md` to verify current state. Then check:

- **`monke-docs/recon/recon-survey.md` exists** — this is the primary input. If missing → "Run `/monke-recon:survey` first. Can't blueprint what hasn't been mapped." Stop.
- **`monke-docs/design-specs.md` exists** — the structural rules for HLD/LLD format. If missing → warn but proceed with best-effort structure.
- **`monke-docs/project-specs.md` exists** with stack locked (no `<<<` remaining). If missing → warn but proceed.

If mode is `lld <component>`:
- HLD must already exist (`monke-docs/hld.md`). If not → "Run `/monke-recon:reconstruct hld` first. LLDs need an HLD to anchor to." Stop.
- The component must appear in the survey or HLD.

---

## Phase 1: HLD Reconstruction

**Resume check (per drafter §7).** If arguments contain `resume:<N>`:
1. Verify partial `monke-docs/hld.md` or partial `monke-docs/lld/<component>.md` exists and parses (this skill does not write a lock-file — section-by-section writes to the real artifact serve as checkpoints).
2. If parse check passes → skip to Phase `<N>` with prerequisites re-validated inline.
3. If parse check fails → emit warning "resume:<N> specified but checkpoint invalid" and proceed from Phase 1 normally.
4. See Context Death Protocol below for the full recovery spec.

**Runs when mode is `hld` or `all`.**

This is archaeology. You are describing what IS, not what SHOULD BE. Every section is sourced from the survey and verified against actual code. Do not dream. Do not improve. Just document.

### 1.1 Load Sources

1. Read `monke-docs/recon/recon-survey.md` — primary source
2. Read `monke-docs/project-specs.md` — for stack bindings and project context
3. If flash mode (survey says "Mode: flash"): read `monke-docs/flash/flash-manifest.md` for shortcuts and deferred items

### 1.2 Write HLD

**Dual-writer guard.** Before writing to `monke-docs/hld.md`, check whether the file already exists with a design-origin header. If the first 20 lines contain any of: `Source: greenfield` / `Source: auto-generated via fast onboarding` / `Origin: design:hld` — the HLD was created (or is mid-creation) by `/monke-design:hld`. STOP and surface:

⏸ **SKILL-GATE:hld-overwrite-guard [HARD] — HLD already exists from design mode.** Options:
1. Resume the design session: `/monke-design:hld resume:<N>` (if a Resume block points there).
2. Force overwrite (you acknowledge destroying the partial design): `/monke-recon:reconstruct force`.
3. Abort.

Never silently overwrite a design-origin HLD. If the existing HLD has a recon-origin header (`Origin: recon:reconstruct`), proceed normally — this is our own prior run.

Write `monke-docs/hld.md` following `design-specs.md` S5.1. All 8 sections required.

Add header:
```
Version: 1.0 | Date: <today> | Source: reverse-engineered from existing codebase | Origin: recon:reconstruct
```

**S1 System Context (L1):**
- What this system is, who uses it, why it exists. One paragraph.
- External actors and systems (from survey's external dependencies + entry points).
- Scope: IN v1 vs explicitly OUT. If flash mode: "built" = IN, "cut" = OUT.

⏸ **PG-1 [HARD]: Present L1 Context.** Lighter than greenfield — user just confirms what was found. Never auto-passes; scope lock always surfaces.

**S2 Container Diagram (L2):**
- Each container from the survey: name, tech, backend language, one-sentence responsibility, type tag (`traditional`/`agentic`).
- Communication protocols between containers.
- Deploy topology (if detectable from Dockerfiles, compose, k8s manifests).
- Database instances and connection strategy.
- If flash mode: note which containers are MVP-quality vs production-ready.

⏸ **PG-2 [SOFT]: Present L2 Containers.** Confirm / Adjust / Reject?
Auto-passes when: all containers have tech + one-sentence responsibility + type tag (`traditional`/`agentic`) assigned. Surfaces under `thorough`; auto-passes under `light`/`standard` when the condition holds.

**S3 Component Map (L3) per container:**
- Module name + responsibility from survey components.
- Boundary contracts — infer typed models from actual function signatures and exports.
- Dependency direction (consumer → provider).
- LLD owner (component name).
- For agentic containers: CoALA summary per design-specs S1.2.

⏸ **PG-3 [SOFT]: Present L3 Components.** Confirm / Adjust / Reject?
Auto-passes when: all boundaries typed, no orphan components, every component has an LLD owner. Surfaces under `thorough`; auto-passes under `light`/`standard` when the condition holds.

**S4 Primary Data Flows (max 3):**
- Trace from survey entry points through components to terminal actions.
- Use format: `Trigger -> Module (contract: ModelA) -> Module (contract: ModelB) -> Terminal`
- For agentic flows: `Input -> Agent (observe) -> [retrieve] -> [reason] -> [tool] -> Output/Loop`

**S5 Decision Index:**
- Scan for existing ADRs in `monke-docs/decisions/` or `decisions/`.
- If none → "No ADRs yet — implicit decisions cataloged in OQs."

**S6 Open Questions:**
- Link to `monke-docs/open-questions.md`. Will be populated by `/monke-recon:oqs`.

**S7 Boundary Matrix:**
- Build from survey's internal dependencies + component boundary contracts.

| Upstream | Contract | Downstream | Error Type | Serialization | Status |
|----------|----------|------------|------------|---------------|--------|

- Status: `verified` (tested), `untested` (code exists but no test), `implicit` (no typed contract).

⏸ **PG-4 [SOFT]: Present Boundary Matrix.** This is where the most dangerous gaps hide. Confirm / Adjust / Reject?
Auto-passes when: matrix complete (no empty rows), no orphan boundaries, contracts consistent across upstream/downstream, stability column filled. Surfaces under `thorough`; auto-passes under `light`/`standard` when the condition holds.

**S8 Phase Plan:**
- Group components by dependency order.
- Write preliminary phases based on dependency analysis. These are placeholders — `/monke-recon:roadmap` will refine them into R1-R5 and sync back to S8 after gap analysis and OQ resolution.
- Note test gate status per component from survey test inventory.

### 1.3 Quality Gate

Run every check from `design-specs.md` S5.3:

| Check | Test |
|-------|------|
| Implementable | Can a dev understand the system from this HLD alone? |
| Bounded | Every module has a typed boundary contract (or "implicit" flag)? |
| Navigable | Find any component's section in 30 seconds? |
| Honest | Gaps flagged, not papered over? |
| Minimal | No decorative prose — constraint, decision, contract, or question? |
| Audited | Boundary matrix complete? |
| Stack-compliant | All containers use locked tech per project-specs? |

**For reconstruction, these checks are diagnostic, not blocking.** A reverse-engineered HLD will naturally fail some checks — that's the point. Each failure feeds `/monke-recon:gaps`.

---

## Phase 2: LLD Reconstruction

**Runs when mode is `lld <component>` or `all`.**

For each component (or the specified one): write `monke-docs/lld/<component>.md`.

### 2.1 Load Sources

1. Read `monke-docs/hld.md` — for boundary contracts and component placement
2. Read `monke-docs/recon/recon-survey.md` — for raw code inventory
3. Read the actual source files for this component — the code IS the design
4. If flash mode: read `monke-docs/flash/flash-manifest.md` — shortcuts and deferred work map directly to maturity tags

### 2.1b Maturity Tag Synthesis (Flash Mode)

If `flash-manifest.md` exists, cross-reference manifest entries against this component's functions:

| Manifest Entry | → Maturity Tag |
|---|---|
| Function works, no shortcuts noted | `as-is` |
| Shortcut noted (hardcoded values, missing validation, no error handling) | `needs-work` |
| Listed in "What Was Cut" or "Known Shortcuts" as placeholder/fake | `stub` |
| Listed in "Known Bugs" affecting this function | `needs-work` (minimum) |

For archaeology mode (no manifest): derive tags from code analysis — has tests + error handling = `as-is`, functional but rough = `needs-work`, placeholder/todo = `stub`.

### 2.2 Write LLD

Follow `design-specs.md` S6.3 structure, but reconstruct from existing code.

**Header:**
```
Parent HLD: S3.<section> | HLD Version: 1.0
Source: reverse-engineered from existing code
Confidence: auto-generated | reviewed | verified
Type: traditional/agentic
Upstream: <Model> from <module> | Downstream: <Model> to <module>
```

**Confidence values:**
- `auto-generated` — freshly reconstructed, not yet reviewed by human or downstream skill
- `reviewed` — human confirmed at PG-9, or `/monke-recon:gaps` / `/monke-design:lld review` has passed over it
- `verified` — tests pass against it, or implementation has been built from it without drift

**Internal Design:**
- Read every function in the component's source files
- Document actual signatures with actual types
- Classify: pure functions vs IO functions
- Map state management patterns
- For agentic: document the actual decision loop, tools, memory access

**Maturity Tags:**

Mark each function/module with a maturity tag:

| Tag | Meaning |
|-----|---------|
| `as-is` | Production-ready. Clean code, error handling, tested. |
| `needs-work` | Functional but rough. Missing error handling, no tests, code smells. |
| `stub` | Placeholder. Not implemented or minimal implementation. |

**File Map:**

| File | Layer | Exports | Depends On | Maturity |
|------|-------|---------|------------|----------|

Example:
```
| src/auth/handler.ts   | 2 | handleLogin(), handleLogout() | types, db    | as-is      |
| src/auth/validate.ts  | 2 | validateToken(), parseJWT()   | types        | needs-work |
| src/auth/types.ts     | 0 | AuthUser, AuthError           | —            | as-is      |
| src/auth/middleware.ts | 1 | authGuard()                   | validate, db | stub       |
```

Downstream consumers (`/monke-implement:implement`, `/monke-design:lld` review mode) parse this table to determine layer behavior overrides per function. The Maturity column is the canonical source — keep it honest.

**Test Coverage:**
- What tests exist for this component (from survey)?
- What test plan would be needed (reference design-specs S6.3)?
- Don't write the test plan — just note the gap.

⏸ **PG-9 [SOFT]: Present each LLD for confirmation.** User verifies the reconstruction matches reality. Confirm / Adjust / Reject?
Auto-passes when: reviewer found 0 violations AND all contracts match HLD boundary matrix exactly. Surfaces under `thorough`; auto-passes under `light`/`standard` when the condition holds.

### 2.3 All Mode

If mode is `all`: iterate through every component in HLD S3. Use agent teams if >3 components — group components by their parent container (HLD S2), assign one agent per container for parallel LLD reconstruction. Each agent reconstructs all components within its container.

---

## Phase 3: Agent Teams Review (Optional)

If Agent Teams are available and user wants adversarial review:

**Spawn Architect + Critic:**
- Architect reviews the reconstructed HLD for internal consistency
- Critic attacks: are boundaries honest? Are contracts typed or just vibes? Are data flows complete or aspirational?
- Critic writes findings to `monke-docs/critic-notes.md`

If Agent Teams not available: self-critique based on quality gate results. Note weaknesses directly in the HLD as inline comments.

---

## Output

- `monke-docs/hld.md` — the reconstructed High-Level Design
- `monke-docs/lld/<component>.md` — one per component (if LLD mode)
- Quality gate results noted in HLD footer

---

## Key Behaviors

- **This is archaeology, not design.** Describe what IS, not what SHOULD BE. Improvements go to `/monke-recon:gaps`, not here.
- **Use the survey as primary source, code as verification.** If something exists in code but isn't in the survey → the survey missed it. Note and include it.
- **Boundary contracts may be implicit.** That's fine — document them as `implicit` in the boundary matrix. Making them explicit is future work.
- **Maturity tags are honest assessments.** A function with no error handling and no tests is `needs-work`, not `as-is`. A function that returns hardcoded values is `stub`.
- **Flash mode context matters.** If the flash manifest lists shortcuts or deferred items, those directly inform maturity tags and the boundary matrix status.
- **Adaptive gating.** HARD gates (PG-1) surface immediately; SOFT gates (PG-2/3/4) auto-confirm when their conditions hold per design-specs S9.4. Do not combine surfaced gates — one response per surfaced gate.
- **If code contradicts the survey** → trust the code. Update the HLD to match reality, note the discrepancy.

---

## Gate Classifications Used Here

| Gate | Type | Notes |
|------|------|-------|
| PG-1 | HARD | L1 Context confirmation. Never auto-passes. |
| PG-2 | SOFT | L2 Containers. Auto-pass: all containers have tech + responsibility + type tag assigned. |
| PG-3 | SOFT | L3 Components. Auto-pass: all boundaries typed, no orphan components, every component has an LLD owner. |
| PG-4 | SOFT | Boundary matrix. Auto-pass: matrix complete, no orphans, contracts consistent, stability column filled. |
| PG-9 | SOFT | LLD reconstruction converged. Auto-pass: 0 reviewer violations AND all contracts match HLD. |
| PG-14 | TRIGGERED | HLD-code divergence during reverse-engineering. Fires when reconstruction finds code that contradicts the prior HLD, or LLD archaeology reveals a component whose boundary contract disagrees with the HLD boundary matrix. Always surfaces. Resolution: amend the HLD, not the code. |

⏸ **PG-14 [TRIGGERED] — HLD-code divergence.** Fires when reverse-engineering surfaces a mismatch between observed code behavior and the reconstructed HLD (or a prior HLD version). Surfaces regardless of rigor. Resolution: update the HLD to describe what IS, log the discrepancy, and confirm with user before continuing LLD work.

Rigor (`light` / `standard` / `thorough`) in `.monke-config.md` controls SOFT behavior:
- `light`: all SOFTs auto-confirm when their conditions are met, log as `[gate:PG-N] auto-confirmed`.
- `standard`: SOFTs fire only if auto-pass conditions fail.
- `thorough`: all SOFTs fire regardless; user sees every decision.

HARD gates always surface regardless of rigor. TRIGGERED gates fire only when the triggering event occurs, regardless of rigor.

---

## Anti-Patterns to Refuse

| If asked to... | Do instead... |
|----------------|--------------|
| Improve the design during reconstruction ("this would be cleaner as...") | Refuse. Reconstruct describes what IS. Improvements belong in `/monke-recon:gaps` and `/monke-recon:roadmap`. Archaeology, not architecture. |
| Mark every function `as-is` because the flash demo works | Refuse. Maturity tags are honest. Functions without error handling or tests are `needs-work`. Hardcoded-value stubs are `stub`. The Maturity column is canonical. |
| Silently fix code-HLD divergence by editing the code | Refuse. That's PG-14 [TRIGGERED] — surface the divergence, amend the HLD, log it. Code is source of truth. |
| Ship an LLD without the `Confidence:` header | Refuse. Every recon-origin LLD carries `Confidence: auto-generated | reviewed | verified`. Downstream skills (implement, test-plan) read it. |
| Invent boundary contracts that the code doesn't express | Refuse. If a boundary is implicit in code, mark it `implicit` in the matrix. Don't fabricate types. |
| Skip PG-1 because "the user knows what they built" | Refuse. PG-1 is HARD. L1 Context scope-lock surfaces regardless of rigor, even for reconstruction. |

---

## Context Death Protocol

**Checkpoint artifacts:** `monke-docs/hld.md` (partial draft — section-by-section); `monke-docs/lld/<component>.md` (one partial per component in flight). Each written section or file is a checkpoint.
**Status line marker:** `Where We Are: recon:reconstruct — HLD S<N>` or `Where We Are: recon:reconstruct — LLD <component> (layer <N>)` while mid-flight.
**Recovery detection:** On re-entry, if `hld.md` exists with sections incomplete → resume at first missing section (S1-S8); if HLD complete but some LLDs still missing their `Confidence:` header → resume LLD pass for those components; if `all` mode and some LLDs unwritten → resume at first missing component; if nothing exists → start fresh at Phase 1.

**LLD header validation on resume.** For each LLD file in `monke-docs/lld/`, read the file header. Verify the `Confidence:` line exists AND its value is one of `auto-generated`, `reviewed`, `verified`. If the line is missing or malformed (typo, unknown value), treat the LLD as corrupted → rewrite from scratch for that component. Emit an audit log line: `[reconstruct:resume] LLD <component> header invalid — regenerating`.

---

## Status Update

On completion, update `monke-status.md`:
- **Recon section:**
  - If HLD mode: mark `[x] Reconstruct (HLD) — <date>`
  - If LLD mode: mark `[x] Reconstruct (LLDs) — <date>` (or partial: `[~] Reconstruct (LLDs) — 2/5 done`)
- **HLD section:** Mark L1-L3 + Boundary Matrix as `[x]` with dates
- **LLDs table:** Populate with components from HLD S3, all status: "reconstructed" or "waiting". Record `Confidence:` value for each LLD.
- Set `Updated:` to today, `by /monke-recon:reconstruct`
- Update "Where We Are": `Phase: **HLD reconstructed — ready for gap analysis**`
- Update "Next action": `/monke-recon:gaps`
