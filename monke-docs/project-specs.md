# Project Specs — <<<project_name>>>

> Project-specific bindings for the design, implementation, and test specs. This file maps abstract rules to concrete tools.

**This is a living document.** Update it in the same commit whenever code, config, or infrastructure changes make any section inaccurate.

**Most fields auto-populate** during fast onboarding. Placeholders (`<<<...>>>`) remain for manual override when auto-detection can't decide. The "Auto-detect hint" column indicates which file fast onboarding reads to fill each field.

### Continuous Improvement Directive

Claude MUST keep this document current:

- **After adding a dependency:** Update the dependency table and relevant commands.
- **After changing directory structure:** Update the directory tree.
- **After modifying CI/CD:** Update the pipeline section.
- **After adding/changing environment variables:** Update the variables table.
- **After resolving a test specs binding (e.g., choosing factory-boy vs manual):** Replace "TBD" with the decision.
- **After any phase completion:** Review all sections for staleness.
- **After an ADR that affects tooling:** Propagate the decision into the relevant binding table.

---

## 1. Specs References

| Spec | Document | Governs |
|------|----------|---------|
| Design | [monke-docs/design-specs.md](design-specs.md) | HLD/LLD creation, pause gates, ADR, teams |
| Implementation | [monke-docs/implementation-specs.md](implementation-specs.md) | Layer pipeline, IL gates, code standards |
| Test | [monke-docs/test-specs.md](test-specs.md) | Test tiers, fixtures, coverage, mock boundaries |
| **This document** | monke-docs/project-specs.md | Tooling, config, CI/CD for <<<project_name>>> |

---

## 2. Stack

Consolidated stack info for <<<project_name>>>. Polyglot projects: one row per container.

### 2.1 Containers & Languages

| Container | Language | Framework | Package Manager | Auto-detect hint |
|-----------|----------|-----------|-----------------|------------------|
<<<stack_containers>>>

### 2.2 Locked Stack Rules

| Layer | Technology | Notes | Auto-detect hint |
|-------|-----------|-------|------------------|
<<<locked_stack_table>>>

<<<stack_enforcement_rules>>>

### 2.3 Environment

| Variable | Purpose | Example | Auto-detect hint |
|----------|---------|---------|------------------|
<<<env_variables>>>

Env files:

| File | Purpose | Committed? | Auto-detect hint |
|------|---------|-----------|------------------|
<<<env_files>>>

### 2.4 Directory Structure

<<<source_tree>>>

Test dirs:

<<<test_tree>>>

---

## 3. Commands

Per-container commands for install, build, run, and IL gate verification. Polyglot projects keep each container's toolchain separate — a Python backend and TS frontend don't share a linter or test runner.

### 3.1 Package Manager & Scripts

| Container | Install | Build | Run | Auto-detect hint |
|-----------|---------|-------|-----|------------------|
<<<package_manager_commands>>>

### 3.2 IL Gate Commands (per container)

Maps abstract IL gate verifications to per-container shell commands. **Polyglot support:** each container declares its own linter, type checker, test runner, and coverage tool.

| Container | IL-0 Command | IL-1 Command | IL-2 Command | IL-3 Command | Auto-detect hint |
|-----------|--------------|--------------|--------------|--------------|------------------|
<<<il_gate_commands>>>

IL gate semantics:
- **IL-0**: types/models exist, linter passes, imports resolve
- **IL-1**: interfaces/stubs exist with types, compilable
- **IL-2**: implementations + unit tests with assertions, coverage > 0%
- **IL-3**: integration tests + coverage meets threshold

### 3.3 CI/CD Pipeline

| Container | CI File | Jobs | Auto-detect hint |
|-----------|---------|------|------------------|
<<<ci_cd_pipeline>>>

---

## 4. Bindings

Test framework and fixture bindings per container. Populate one row per container with its own toolchain.

### 4.1 Test Bindings (per container)

| Container | Test Framework | Test Runner | Unit Dir | Integration Dir | System Dir | Coverage Tool | Threshold | Auto-detect hint |
|-----------|----------------|-------------|----------|-----------------|------------|---------------|-----------|------------------|
<<<test_bindings>>>

### 4.2 Fixture & Mock Bindings (per container)

| Container | Object Factory | HTTP Mock | Async Support | DB Fixture Strategy | Shared Fixture File | Auto-detect hint |
|-----------|----------------|-----------|---------------|---------------------|---------------------|------------------|
<<<fixture_bindings>>>

### 4.3 Design Specs Bindings

Pause gate artifact locations:

| Gate | Artifact Location |
|------|------------------|
| PG-1 (L1 Context) | `monke-docs/hld.md` S1 |
| PG-2 (L2 Containers) | `monke-docs/hld.md` S2 |
| PG-3 (L3 Components) | `monke-docs/hld.md` S3 |
| PG-4 (Boundary Matrix) | `monke-docs/hld.md` S7 |
| PG-5 (LATS decision) | `monke-docs/decisions/NNN-slug.md` (ADR for selected option) |
| PG-6 (Language choice) | `monke-docs/decisions/NNN-slug.md` (ADR for exception) |
| PG-7 (Pattern selection) | `monke-docs/decisions/NNN-slug.md` (ADR for pattern) |
| PG-8 (ADaPT decomposition) | `monke-docs/lld/<component>-draft.md` |
| PG-9 (LLD converged) | `monke-docs/lld/<component>-draft.md` (final) |
| PG-10 (Test plan) | Unit + integration test tables in LLD |
| PG-11 (Phase checkpoint) | `monke-docs/checkpoints/phase-N-checkpoint.md` |
| PG-12 (Stack violation) | `monke-docs/decisions/NNN-slug.md` (ADR with status "exception") |
| PG-13 (Test failure escalation) | `monke-docs/open-questions.md` |
| PG-14 (HLD revision from LLD) | `monke-docs/hld.md` (updated section) |
| ADRs | `monke-docs/decisions/NNN-slug.md` |
| Open questions | `monke-docs/open-questions.md` |

Supported languages (for language-choice exceptions, PG-6):

| Language | Best For | LLM Integration | Test Framework |
|----------|---------|-----------------|----------------|
<<<supported_languages_table>>>

Design specs modifications specific to <<<project_name>>>:

<<<design_specs_modifications>>>

### 4.4 AI & Data Science Infrastructure (Optional)

If the project uses versioned-artifact or data-dependent tools (design-specs S1.2) — LLM model versions, NLP pipeline artifacts, CV model weights, embedding indices, or any artifact that changes when retrained — bind these:

| Concept | <<<project_name>>> Binding | Auto-detect hint |
|---------|----------------------------|------------------|
| Model registry | <<<model_registry>>> | `mlflow.yaml`, `wandb/` |
| Feature store | <<<feature_store>>> | `feast.yaml`, `feature_store.yaml` |
| Experiment tracker | <<<experiment_tracker>>> | `wandb/`, `mlruns/`, `.neptune/` |
| Eval metric library | <<<eval_metric_library>>> | `requirements.txt` (e.g. evaluate, ragas) |
| Eval metric thresholds | <<<eval_metric_thresholds>>> | project config |
| Eval test dataset dir | <<<eval_test_dataset_dir>>> | `tests/eval/`, `data/eval/` |

Same `<<<placeholder>>>` convention. If the project has no versioned-artifact or data-dependent components, leave this section empty or delete it.
