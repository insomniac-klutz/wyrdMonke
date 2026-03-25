# WyrdMonke Seer:Registry — Pin your artifacts or they'll pin you

> **Usage:** `/monke-seer:registry <component>`
>
> An unversioned artifact is a time bomb with a random fuse. This skill pins versioned artifacts — models, indices, rulesets, prompt templates — the same way monke pins library versions. Explicitly, with eval thresholds, and with a retraining trigger so you know when the pin needs to move. The model registry is a convention, not a platform.

---

## Arguments

`$ARGUMENTS` parsing:
- Single positional: component name from HLD S3 that owns the artifact (**required**)

```
COMPONENT="${ARGUMENTS:?Component name required}"
```

If missing → read HLD S3, list components with versioned-artifact tools, ask user to pick. If no components have versioned-artifact tools → "Nothing to pin. Registry is for things that change when retrained — models, indices, rulesets, prompts. Static-contract tools don't need pinning."

---

## Prerequisites

**Agent Teams Gate:** Read `CLAUDE.md`. If the Agent Teams section is missing → **stop**. Tell the user: "Agent teams not configured. Run `/monke-sync` or copy the Agent Teams section from `monke-CLAUDE.md` into your `CLAUDE.md`." Do not proceed.

- Component exists in HLD S3 with at least one versioned-artifact tool in its CoALA summary
- Read `monke-status.md` and the component's HLD entry

---

## What Gets Pinned

Any artifact referenced as a versioned-artifact tool in the CoALA summary (design-specs S1.2). If its output changes when the artifact is retrained, rebuilt, or updated — it gets pinned. Period.

### LLM Artifacts

- **Model version pins:** The exact model ID — `claude-sonnet-4-6`, `gpt-4o-2024-08-06`, not "GPT-4" or "Claude." Vague pins are not pins. They're prayers.
- **Prompt template versions:** Templates drift. An engineer "improves" the system prompt, coherence drops 8%, nobody knows why. Version your prompts like you version your code.
- **Tool schema versions:** When the agent's tool definitions change — new parameters, renamed fields, removed tools — that's an artifact version change that affects behavior.
- **Eval metrics:** Coherence thresholds, hallucination rate ceilings, instruction-following rate floors, format compliance rates, cost per call budgets.

### NLP Artifacts

- **Trained classifiers:** Intent models, sentiment models, NER models, topic classifiers — any model trained on your data.
- **Embedding models:** Which embedder, which version, which dimensionality, trained on what. Swapping `text-embedding-3-small` for `text-embedding-3-large` without re-indexing is a data corruption event, not an upgrade.
- **Compiled rulesets:** Regex pattern sets, gazetteers, lookup tables, normalization dictionaries — things that get rebuilt periodically.
- **Eval metrics:** F1, precision, recall, entity match rate, latency thresholds.

### CV Artifacts

- **Model weights:** Architecture + training checkpoint + fine-tuning dataset reference. "ResNet50" is not a version pin — "ResNet50-ImageNet-v2-finetuned-medical-2024-08-01" is.
- **Feature extractors:** Pretrained backbone version. The feature extractor IS the lens — swap it and everything downstream sees differently.
- **Detection/segmentation models:** Architecture + weights + training data + confidence thresholds.
- **Eval metrics:** mAP, IoU, classification accuracy, inference latency, model size.

### Artifact Chains

Some components depend on multiple versioned artifacts. An NLP pipeline might use embedding model v2.1 + classifier v3.0. A multimodal agent might use an LLM + a vision model + an embedding index. Each artifact gets its own pin, but **chains must be tested together.**

When one link updates, you need to know if the rest of the chain still works with it. A new embedder might produce vectors the old classifier can't handle. A new LLM version might format tool calls differently than the old one.

```
Chain: <component> artifact chain
  1. <artifact A> @ <version> — <role in chain>
  2. <artifact B> @ <version> — <role in chain>
  Coupling: <how they interact — shared vector space? shared schema? sequential pipeline?>
  Chain eval: <metric that tests the chain end-to-end, not just individual artifacts>
```

---

## Phase 1: Inventory the Artifacts

For the target component, read its CoALA summary. List every tool tagged `versioned-artifact`:

```
Component: <name>
Artifacts found:
  1. <artifact name> — <type>
     Currently pinned: <version if pinned, "UNPINNED" if not>
     Referenced in: <HLD S7 / LLD header / Layer 0 types / nowhere>
  2. ...

Chain detected: <yes/no — do multiple artifacts interact?>
```

**Every unpinned artifact is a ticking clock.** You don't know what version is running. You can't reproduce results. You can't tell if a regression is your code or an artifact update. Pin it or accept chaos.

---

## Phase 2: Pin Each Artifact

For each artifact, build the registry entry:

```markdown
### Artifact: <name>
Component: <HLD S3 component>
Type: <llm-model | embedding-model | trained-classifier | compiled-ruleset | cv-model | prompt-template | tool-schema | other>
Current version: <exact version identifier — be specific>
Trained/built on: <date> with <dataset reference — link to profile if profiled>
Chain: <standalone | chained with <artifact B>> (if part of an artifact chain)
Eval thresholds:
  - <metric>: ≥ <threshold>
  - <metric>: ≤ <threshold>
Retraining trigger: <condition — "eval metric drops below threshold" | "weekly schedule" | "data drift detected via profile baseline" | "model provider releases new version" | "manual">

Storage: per project-specs S10.6 <<<model_registry>>> binding
Pinned in: Layer 0 versioned type (implementation-specs §3)
Tested by: Eval tests in LLD test plan (test-specs §2)
```

**⏸ Present each artifact's registry entry. User confirms before it becomes binding.**

---

## Phase 3: Propagate the Pins

The version pin isn't just a note — it lives in three places, and all three must agree:

1. **HLD S7 boundary matrix:** `versioned(<artifact>, <pin>)` stability annotation on the relevant row
2. **LLD header:** `Version pins: <artifact: version>`
3. **Layer 0 types:** `model_version` field (or equivalent) in the versioned type declaration

If any of these don't exist yet (HLD incomplete, LLD not started, Layer 0 not written), note where the pin WILL go. If they already exist, verify they match. Mismatched pins across documents = guaranteed confusion at 3am.

**For artifact chains:** all pins in the chain propagate together. If embedder v2.1 and classifier v3.0 are chained, both appear in the same boundary matrix row, LLD header, and Layer 0 type declarations.

---

## Phase 4: Define the Update Protocol

Retraining a model is not an ops task. It's a design decision that changes the component's behavior. The registry makes this explicit:

### New Version Passes Eval

Update the pin through the normal flow:
1. Update Layer 0 type's version field
2. Update LLD header
3. Update HLD S7 boundary matrix row
4. Run eval tests with the new version — all must pass
5. For chains: run chain eval too — individual pass doesn't mean chain pass
6. Commit. This is an LLD amendment, not a silent swap.

### New Version Fails Eval

The old pin stays. Investigate:
- Was the training data different? Check against profile baseline.
- Was the approach wrong? Consider `/monke-seer:experiment` to re-evaluate.
- Was the threshold wrong? Fix the design, not the threshold. Weakening thresholds to accommodate a new version is not an upgrade — it's a surrender.

### Model Provider Releases New Version

For external models (LLM APIs, hosted embedding services): a provider version update (e.g., `gpt-4o-2024-08-06` → `gpt-4o-2024-11-20`) is treated the same as a retraining event. Run eval tests. If pass → update pin. If fail → old pin stays. If the old version is deprecated → escalate as an OQ.

### Never Silently Swap

A version change without updating the pin is a bug you'll find in production at 3am when the on-call engineer can't reproduce the issue because the model silently rolled over three weeks ago.

---

## Where This Connects

- **design-specs S1.2:** Versioned-artifact tool subtype requires version pin
- **design-specs S7:** Boundary matrix `versioned(<artifact>, <pin>)` stability annotation
- **implementation-specs Layer 0:** Versioned types carry the pin as a declarative field
- **test-specs §2:** Eval tests verify the pinned artifact meets thresholds
- **project-specs S10.6:** `<<<model_registry>>>` binding declares storage location
- **monke-seer/profile.md:** Training data profiled before the artifact is built
- **monke-seer/experiment.md:** Model selection recorded as experiment ADR

---

## Anti-Patterns to Refuse

| If you find yourself... | Stop. Do this instead. |
|------------------------|------------------------|
| Building a model registry platform | The registry is a binding in project-specs + version pins in code. A convention, not a service. No MLOps cathedral. |
| Storing models in the git repo | Models are binary artifacts. Git is for code. Use the project's `<<<model_registry>>>` binding — S3, GCS, HuggingFace Hub, whatever your team uses. |
| Updating model versions without updating the pin | Silent swaps break reproducibility. You'll deploy a new model, it'll regress, and nobody will know what changed. Update the pin through the normal amendment flow. |
| Pinning "GPT-4" instead of `gpt-4o-2024-08-06` | That's not a pin. That's a prayer. The provider can change what "GPT-4" means tomorrow. Pin the exact version identifier. |
| Creating separate versioning per ML domain | NLP models, CV models, LLM configs, embedding indices — all get the same treatment. One convention, one registry. Domain is not special. |
| Weakening eval thresholds to accommodate a new version | Fix the artifact, not the threshold. If the new version is worse, the old pin stays. That's the whole point of having thresholds. |
| Retraining on a cron job without eval gates | A cron job is not a quality gate. Retrain whenever you want — but the new version doesn't replace the pin until eval tests pass. |
| Updating one artifact in a chain without testing the chain | Individual artifact eval isn't enough. If the embedder changes, test the embedder AND the downstream classifier together. Chains break at the joints. |
