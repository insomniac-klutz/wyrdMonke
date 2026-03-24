# WyrdMonke Recon:Gaps — What's missing for prod?

> **Usage:** Copy `monke-recon/` to `.claude/commands/monke-recon/`. Invoke: `/monke-recon:gaps [focus]`

---

## Arguments

`$ARGUMENTS` parsing:
- Single positional: focus area — `security` | `performance` | `testing` | `errors` | `observability` | `data` | `docs` | `a11y`
- Default (empty): full scan across ALL dimensions
- Example: `/monke-recon:gaps security`

```
FOCUS="${ARGUMENTS:-all}"
```

---

## Prerequisites

Read `monke-status.md` to verify current state. Then check:

- **`monke-docs/recon/recon-survey.md` exists** — raw inventory of what's there. If missing → "Run `/monke-recon:survey` first." Stop.
- **`monke-docs/hld.md` exists** — reconstructed architecture. If missing → "Run `/monke-recon:reconstruct` first. Gaps need blueprints to measure against." Stop.

Optional but recommended:
- `monke-docs/lld/*.md` — component-level detail makes gap detection more precise.

---

## Gap Analysis

Scan every dimension below unless a focus area was specified. For each finding, assign:

| Field | Values |
|-------|--------|
| **Severity** | `critical` (blocks ship) / `high` (ship-risky) / `medium` (tech debt) / `low` (polish) |
| **Effort** | `small` (hours) / `medium` (day) / `large` (days) / `xl` (week+) |
| **Component** | Which container/component this affects |

---

### Error Handling

Scan for:
- **Unhandled exceptions.** Bare `catch {}`, empty exception blocks, swallowed errors. These are time bombs.
- **Missing error types at boundaries.** Functions that return raw strings or generic errors instead of typed error variants.
- **No fallback for external service failures.** HTTP calls without timeouts, retries, or circuit breakers.
- **Missing input validation.** User-facing endpoints that trust input blindly. API boundaries with no schema validation.
- **Panic/crash paths.** `unwrap()` on fallible operations, unguarded `!` force-unwraps, `assert` in production paths.

---

### Security

Scan for:
- **Authentication gaps.** Endpoints without auth middleware. Missing token validation.
- **Authorization gaps.** No role/permission checks. Missing ownership validation on resources.
- **Injection surfaces.** String concatenation in SQL queries. Unsanitized user input in templates (XSS). Command injection via user input.
- **Secrets in code.** API keys, passwords, tokens in source files. `.env` files committed. Missing `.gitignore` entries for secret files.
- **Missing hardening.** No CORS configuration. No rate limiting. HTTP without TLS enforcement. Missing security headers.
- **Dependency vulnerabilities.** Known CVEs in current dependency versions (if detectable).

---

### Performance

Scan for:
- **N+1 queries.** Loops that make individual DB/API calls instead of batch operations.
- **Unbounded queries.** `SELECT *` without `LIMIT`. Queries that could return millions of rows.
- **Missing pagination.** List endpoints that return all results.
- **No caching.** Repeated reads of the same data with no cache layer.
- **Unbounded memory growth.** Collections that grow without eviction. Event listeners that accumulate.
- **Missing timeouts.** External calls (HTTP, DB, LLM) without timeout configuration.
- **Synchronous bottlenecks.** Blocking calls in async contexts. Sequential work that could be parallel.

---

### Observability

Scan for:
- **No logging or inconsistent logging.** Missing log statements at key decision points. Mixed log formats. No structured logging.
- **No metrics.** No request counters, latency histograms, error rates.
- **No health checks.** No `/health` or `/ready` endpoint for orchestrators.
- **No error tracking.** No Sentry/Bugsnag/equivalent integration. Errors vanish into void.
- **No alerting.** Even if metrics exist, no alert rules defined.
- **Agentic blind spots.** LLM calls without token tracking. Agent loops without iteration counting. No observability into decision quality.

---

### Testing

Scan for (cross-reference against survey test inventory):
- **Missing test types.** No unit tests? No integration tests? No system/e2e tests?
- **Missing coverage for critical paths.** The happy path through the most important data flow — is it tested?
- **No test infrastructure.** No fixtures, no factories, no mock utilities. Tests would have to be built from scratch.
- **Missing edge case coverage.** Error paths, empty inputs, boundary values, concurrent access.
- **Flaky test signals.** Tests that depend on timing, external services, or shared mutable state.
- **Test-to-code ratio.** Rough estimate. Below 0.5 for a production system is a red flag.

---

### Data Integrity

Scan for:
- **Missing migrations.** Schema changes without migration files. Manual DB modifications.
- **No backup strategy.** No backup configuration, no recovery plan.
- **Missing DB constraints.** Nullable fields that shouldn't be. No foreign key constraints. No unique constraints where needed.
- **No data sanitization.** User input stored raw without cleaning.
- **Missing transaction boundaries.** Multi-step operations that can leave data in inconsistent state on partial failure.

---

### Documentation

Scan for:
- **Missing README.** No project-level README or bare-bones boilerplate only.
- **Missing API docs.** No OpenAPI/Swagger spec. No endpoint documentation.
- **Missing environment setup.** No instructions for getting a dev environment running.
- **Missing deployment instructions.** No deployment guide. No infra-as-code.
- **Stale documentation.** Docs that reference removed features or old patterns.

---

### Accessibility (UI projects only)

Skip this section if no UI exists. Scan for:
- **Missing ARIA labels.** Interactive elements without accessible names.
- **No keyboard navigation.** Custom components not reachable via Tab/Enter/Escape.
- **Missing alt text.** Images without `alt` attributes.
- **Color contrast.** Low-contrast text. Color as the only differentiator.
- **Missing focus indicators.** Custom styles that remove `:focus` outlines without replacement.

---

## Output

Write `monke-docs/recon/recon-gaps.md` with findings grouped by severity.

**Structure:**
```markdown
# Recon Gap Analysis
Project: <name> | Date: <today> | Focus: <focus or "full scan">

## Summary
| Severity | Count |
|----------|-------|
| Critical | N |
| High | N |
| Medium | N |
| Low | N |

## Critical — Blocks Ship
| # | Dimension | Component | Finding | Effort |
|---|-----------|-----------|---------|--------|
| G-1 | ... | ... | ... | ... |

## High — Ship-Risky
| # | Dimension | Component | Finding | Effort |
|---|-----------|-----------|---------|--------|

## Medium — Tech Debt
| # | Dimension | Component | Finding | Effort |
|---|-----------|-----------|---------|--------|

## Low — Polish
| # | Dimension | Component | Finding | Effort |
|---|-----------|-----------|---------|--------|
```

**Aim for 150-180 lines.** Every finding is a row, not a paragraph.

---

## Decision Gate

⏸ **Present gaps grouped by severity.**

Show the summary table first. Then walk through criticals and highs. User confirms, dismisses, or reprioritizes.

**Common user actions:**
- "That's not critical for our use case" → downgrade
- "We actually have X, you missed it" → remove
- "This is worse than you think" → upgrade
- "Add Y to the list" → add

Update `recon-gaps.md` with user's adjustments before finalizing.

---

## Key Behaviors

- **Be thorough but honest about severity.** An MVP demo app doesn't need enterprise observability. A public-facing API does need input validation. Context matters.
- **Focus on what genuinely blocks production**, not wishlist items. "No Kubernetes autoscaling" is not a gap for a hobby project.
- **Cross-reference against the HLD boundary matrix.** Gaps at boundaries are the most dangerous — they affect multiple components. An `implicit` boundary status in the matrix is almost always a gap.
- **Don't duplicate the survey.** The survey says what exists. Gaps says what's missing. No overlap.
- **Don't prescribe solutions.** That's the roadmap's job. Gaps just identifies the holes.
- **Severity is contextual.** Missing auth is critical for a public API, low for an internal CLI tool. Assess against the system's actual exposure surface.
- **Effort estimates are rough.** `small` = a focused PR. `xl` = a multi-day effort touching multiple components. Don't fake precision.

---

## Status Update

On completion, update `monke-status.md`:
- **Recon section:** Mark `[x] Gap analysis — <date>`
- Set `Updated:` to today, `by /monke-recon:gaps`
- Update "Where We Are": `Phase: **Gaps identified — surface open questions next**`
- Update "Next action": `/monke-recon:oqs`
- If critical gaps found → add to Open Blockers table:
  | ID | Blocks | Summary |
  |----|--------|---------|
  | G-1 | production | <finding summary> |
