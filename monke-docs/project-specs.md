# Project Specs — <<<project_name>>>

> Project-specific bindings for the design, implementation, and test specs. This file maps abstract rules to concrete tools.

**This is a living document.** Update it in the same commit whenever code, config, or infrastructure changes make any section inaccurate.

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

## 2. Stack Bindings

Maps implementation specs' abstract references to <<<project_name>>>'s locked stack:

| Abstract Concept | <<<project_name>>> Binding |
|-----------------|-------------------|
<<<stack_bindings_table>>>

---

## 3. Dependency Configuration

<<<dependency_manifest_and_lockfile>>>

### Commands

| Action | Command |
|--------|---------|
<<<package_manager_commands>>>

---

## 4. Environment Configuration

### Files

| File | Purpose | Committed? |
|------|---------|-----------|
<<<env_files>>>

### Required Variables

| Variable | Purpose | Example |
|----------|---------|---------|
<<<env_variables>>>
---

## 5. Directory Structure

### Implementation

<<<source_tree>>>

### Tests

<<<test_tree>>>

---

## 6. <<<additional_tooling>>> Configuration

---

## 7. CI/CD Pipeline (GitHub Actions)

<<<ci_cd_pipeline>>>
---

## 8. Implementation Specs Bindings

Maps abstract IL gate verifications to <<<project_name>>> commands:

| Gate | Abstract Verification | <<<project_name>>> Command |
|------|----------------------|-------------------|
<<<il_gate_commands>>>

---

## 9. Test Specs Bindings

| Abstract Concept | <<<project_name>>> Binding |
|-----------------|-------------------|
| Test framework | <<<test_framework>>> |
| Test runner command | <<<test_runner_command>>> |
| Unit test dir | <<<unit_test_dir>>> |
| Integration test dir | <<<integration_test_dir>>> |
| System test dir | <<<system_test_dir>>> |
| Object factory library | <<<object_factory_library>>> |
| HTTP mock library | <<<http_mock_library>>> |
| Async test support | <<<async_test_support>>> |
| DB fixture strategy | <<<db_fixture_strategy>>> |
| Coverage tool | <<<coverage_tool>>> |
| Coverage threshold | <<<coverage_threshold>>> |
| Shared fixture file | <<<shared_fixture_file>>> |

---

## 10. Design Specs Bindings

### 10.1 Locked Stack

| Layer | Technology | Notes |
|-------|-----------|-------|
<<<locked_stack_table>>>

### 10.2 Supported Languages

| Language | Best For | LLM Integration | Test Framework |
|----------|---------|-----------------|----------------|
<<<supported_languages_table>>>

### 10.3 Stack Enforcement Rules

<<<stack_enforcement_rules>>>

### 10.4 Modifications

<<<design_specs_modifications>>>

### 10.5 Pause Gate Artifacts

Maps design specs pause gates to <<<project_name>>>-specific artifacts:

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
