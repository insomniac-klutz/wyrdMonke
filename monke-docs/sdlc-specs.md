# SDLC Specs

> How the specs docs connect end-to-end. Each phase follows this flow — design specs produces the inputs that implementation specs consumes, and test gates enforce quality at every transition. One orchestrator (`/monke`) routes between phases. Gates are adaptive — most surface only when their quality conditions fail or rigor demands (see design-specs S9.4 + S12).

---

## 0. Entry Point — Unified Orchestrator

All project work enters through a single orchestrator, `/monke [rigor] [override]`. There are no separate flash/recon/production orchestras — one decision tree handles every state.

```
/monke  →  state detection  →  route to phase
  │
  ├─ No status, no code           → greenfield (ask: flash or production?)
  ├─ No status, code exists       → Fast Onboarding (see S0.1)
  ├─ Status exists                → resume from per-component maturity
  │   ├─ Components at mixed maturity → route EACH to its entry layer (per-component routing)
  │   ├─ All components IL-3          → phase checkpoint
  │   └─ All phases checkpointed      → ship or rage
  └─ Full recon requested         → /monke recon (opt-in, detailed gap analysis)
```

**Rigor is declared** in `.monke-config.md` per design-specs S12 (`light` | `standard` | `thorough`). Rigor controls how many SOFT gates surface. HARD and TRIGGERED gates ignore rigor.

### 0.1 Fast Onboarding (existing projects, no status file)

Fast onboarding replaces the "run full recon first" requirement. One pass, one HARD gate, then work begins immediately.

```
Fast Onboarding:
  1. Auto-detect stack (package.json, Cargo.toml, pyproject.toml, go.mod, etc.) — no gate
  2. Auto-map structure (containers from entry points + Dockerfiles + workspaces) — no gate
  3. Per-component maturity scan (mechanical: linter, imports, tests, coverage) — no gate
     → assigns each component: pre-L0 | IL-0 | IL-1 | IL-2 | IL-3 | unknown (scan failed)
  4. Generate status + lightweight HLD + auto-generated stub LLDs — no gate
     → LLDs stamped Confidence: auto-generated
  5. ⏸ PG-1 [HARD] — present assessment. User confirms or adjusts maturity per component.
  6. Route least-mature component to its pipeline entry layer — begin work.
```

Fast onboarding is the official entry path for existing projects. Full recon remains available via `/monke recon` for users who want survey → reconstruct → gaps → OQs → roadmap before starting work. Not mandatory.

### 0.2 Per-Component Routing

Routing is per-component, not per-project. A project with 10 components where 3 are IL-3, 4 are IL-1, and 3 are missing is routed component-by-component to its correct entry layer. The orchestrator suggests parallel work (via agent team — worker pool archetype, see design-specs S3.1) when it detects independent components at the same maturity level.

### 0.3 Progressive Formalization

Documentation emerges from work, not precedes it. This sits alongside design-first as an official pattern:

| Path | When used | Docs emerge |
|------|-----------|-------------|
| **Design-first** | Greenfield, `thorough` rigor, regulated projects | HLD → LLD → code → tests. Docs up front. |
| **Progressive** | Existing code at any maturity | LLDs auto-generated from code, stamped `Confidence: auto-generated`, upgraded to `reviewed` then `verified` as gaps fill. |

Rigor gate:

- `light` / `standard`: both paths acceptable.
- `thorough`: design-first enforced for greenfield. Existing code still uses progressive but MUST pass SOFT gate PG-9 with human review — auto-generated LLDs are not sufficient.

**Compliance note:** For regulated / compliance projects (HIPAA, PCI, GDPR, SOX, ISO-27001, FedRAMP, SOC2), use `/monke thorough`. Thorough rigor enforces design-first ordering, surfaces all SOFT gates, and blocks trusting auto-generated LLDs as input to downstream work.

---

## 1. HLD (one-time, governs all phases)

Design specs owns this. Uses Agent Teams — adversarial pair archetype (Architect + Critic) per design-specs S3.1.

```
L1 Context → PG-1 [HARD] → L2 Containers → PG-2 [SOFT] → L3 Components → PG-3 [SOFT] → Boundary Matrix → PG-4 [SOFT]
```

SOFT gates auto-pass when their S9.4 conditions hold (rigor permitting). PG-1 always surfaces. LATS (PG-5 [SOFT]) fires at every design branch within each level.

Output: `monke-docs/hld.md` with phase plan (HLD S8), boundary matrix (HLD S7), data flows (HLD S4).

For existing projects entering via fast onboarding, HLD is auto-generated from discovered structure with boundary confidence flags (HIGH for typed exports, LOW for bare directories). LOW-confidence boundaries surface as questions at PG-1.

---

## 2. LLD (per component, just-in-time)

Design specs owns this. Uses Agent Teams — compliance pair archetype (Designer + Reviewer) per design-specs S3.1. Round cap: 3 peer rounds, then lead synthesizes (design-specs S3.2).

```
ADaPT decomposition → PG-8 [SOFT] → Design rounds (max 3) → PG-9 [SOFT] → Test plan → PG-10 [SOFT]
```

For sequential same-type work (e.g. multiple LLDs in one phase), the team is **kept alive and reassigned** via SendMessage per design-specs S3.7 — do not teardown + recreate.

Output: `monke-docs/lld/<component>.md` with typed signatures, file map, unit test plan, integration test plan. Header includes `Confidence: auto-generated | reviewed | verified` per design-specs S6.3.

---

## 3. Implementation (per component)

Implementation specs owns this. IL gates are AUTO — they run silently and surface only on failure (see design-specs S9.4).

```
Layer 0: Types/models/schemas → IL-0 [AUTO] (imports resolve, migration up/down, language-specific typed contracts per B2)
Layer 1: Function stubs + annotations → IL-1 [AUTO] (skeleton compiles)
Layer 2: Bodies + unit tests (interleaved per function) → IL-2 [AUTO] (tests have assertions + coverage > 0%)
Layer 3: Integration tests → IL-3 [AUTO] (full suite green, coverage met)
```

Dependency order: implement functions per LLD decomposition tree. Each function body is immediately followed by its unit tests — never batch. On IL gate failure, the gate becomes visible.

---

## 4. Phase Checkpoint

Design specs owns the gate. Requires ALL components in the phase to have passed IL-3.

```
All component IL-3 gates green → System tests (end-to-end HLD data flows) → PG-11 [HARD] (user sign-off — NEVER skippable)
```

Output: `monke-docs/checkpoints/phase-N-checkpoint.md`. Phase N must pass before Phase N+1 begins.

---

## 5. Recon → Implementation Bridge (opt-in deep path)

Fast onboarding is the default for existing projects (S0.1). Recon is the deeper opt-in path — used when the user wants explicit survey, gap analysis, OQ mining, and a phased roadmap before work begins.

```
Recon: survey → reconstruct (HLD + LLDs) → gaps → oqs → roadmap
                                                          │
                                                     roadmap syncs R1-R5 to HLD S8
                                                          │
Bridge: per component: /monke-design:lld <component> (review mode) → PG-9 + PG-10
                                                          │
                                                     converges to standard pipeline
                                                          ▼
Implementation: Layer 0-3 (maturity tags override layer behavior) → IL gates → checkpoint
```

Recon LLDs carry maturity tags (`as-is`/`needs-work`/`stub`) instead of clean designs. The implement skill reads these and adjusts layer behavior: `as-is` = verify only, `needs-work` = improve, `stub` = build from scratch.

Recon LLDs also carry `Confidence: auto-generated` (pre-review) or `reviewed` (after the bridge PG-9). Downstream skills MUST NOT treat `auto-generated` LLDs as trusted input for neighbor contracts.

---

## 6. Doc Interaction Map

```
design-specs.md ──produces──→ HLD + LLD artifacts
                                        │
                                   (hand-off: LLD file map + signatures + test plans)
                                        │
                                        ▼
implementation-specs.md ──consumes──→ Layer 0-3 pipeline
                                        │
                                   (hand-off: passing IL-3 gate)
                                        │
                                        ▼
design-specs.md §11 ──owns──→ Phase checkpoint (PG-11)

project-specs.md ──binds──→ All of the above to concrete tools + locked stack (S2, S8-S10)
test-specs.md ──governs──→ HOW tests are written (tiers, fixtures, mocks, coverage rules)
```

---

## 7. Failure Protocol

IL gates are AUTO — they surface only on failure (design-specs S9.4). PG-13 is TRIGGERED — it fires on persistent failure regardless of rigor.

| Failure at | Action |
|------------|--------|
| Unit test (IL-2 AUTO fail) | Surface failure. Fix implementation or surface LLD issue to user. |
| Integration test (IL-3 AUTO fail) | Surface failure. Trace to boundary — update HLD matrix + both LLDs if contract wrong. |
| System test | Trace to boundary, apply integration protocol. |
| Persistent (>2 cycles) | ⏸ PG-13 [TRIGGERED] — escalate to user. Likely LATS backtrack required. Fires regardless of rigor. |
| Stack violation | ⏸ PG-12 [TRIGGERED] — user grants or denies exception. Fires regardless of rigor. |
| HLD revision from LLD | ⏸ PG-14 [TRIGGERED] — user confirms HLD update. Fires regardless of rigor. |

Claude MUST NOT weaken a test to pass a gate. Claude MUST NOT advance past a failing gate.

---

## 8. System Terminology

Canonical names for concepts that appear across multiple skills. When in doubt, use the canonical name.

| Concept | Canonical Name | Aliases Used | Defined In |
|---------|---------------|-------------|------------|
| LLD from recon:reconstruct | **recon-origin LLD** | reconstructed, reverse-engineered | sdlc-specs §5 |
| Function/module code quality tag | **maturity tag** (`as-is`/`needs-work`/`stub`) | completion status, readiness | recon:reconstruct |
| Component state from fast-onboarding scan | **maturity level** (`pre-L0`/`IL-0`/`IL-1`/`IL-2`/`IL-3`/`unknown`) | scan result, pipeline entry level | sdlc-specs §0.1 |
| Integration test postponed for missing neighbor | **deferred test** | integration deferred, deferred integration test | implement.md Phase 5 |
| OQ that prevents downstream work | **blocking OQ** | blocker, open blocker, P0 blocker | design:oq, status dashboard |
| Rage scan output file | **rage-run log** | scan log, rage log, findings report | rage-run/template.md |
| HLD header `Source: reverse-engineered` | **recon-origin marker** | reconstructed header, source tag | recon:reconstruct |
| LLD status after review-mode PG-9+PG-10 | **ready** | confirmed, implementation-ready | design:lld, status dashboard |
| LLD header confidence field | **LLD confidence** (`auto-generated`/`reviewed`/`verified`) | confidence tag, LLD maturity | design-specs S6.3 |
| Framework rigor level | **rigor** (`light`/`standard`/`thorough`) | mode, profile, strictness | design-specs S12 |
| Gate classification | **gate class** (AUTO/SOFT/HARD/TRIGGERED) | gate type, gate kind | design-specs S9.4 |
| Team shape pattern | **team archetype** (adversarial pair / compliance pair / worker pool / pipeline pair) | team type, team shape | design-specs S3.1 |
