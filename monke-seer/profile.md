# WyrdMonke Seer:Profile — Taste the data before you cook with it

> **Usage:** `/monke-seer:profile <data-source>`
>
> Profiles a data source before you design with it. Schema, distributions, drift surface — the statistical bones that become Layer 0 types and eval baselines. Skip this and you're building a bridge without measuring the river.

---

## Arguments

`$ARGUMENTS` parsing: single positional — the data source name (what the component will consume).

```
DATA_SOURCE="${ARGUMENTS:?Data source name required}"
```

If missing → read HLD S3, list components with versioned-artifact or data-dependent tools, show their data sources, ask which one to profile.

---

## Prerequisites

- An HLD exists with at least one component that uses a versioned-artifact or data-dependent tool (design-specs S1.2)
- That component has a data source it hasn't tasted yet
- Read `monke-status.md` for context

If no component consumes this data source → stop. "Profile feeds design. No consumer = no profile needed. You're doing tourism, not engineering."

---

## When to Run

- **Before HLD L3** when a component will chew on a versioned-artifact or data-dependent tool — the data's shape is a design input, not a post-hoc discovery
- **Before LLD** when the component's input contract needs distributional expectations — you can't write Layer 0 types if you don't know what "normal" looks like
- **When an eval test fails** and the root cause might be data drift, not code — the profile tells you what "normal" was supposed to be

---

## Phase 1: Find the Source & Detect Domain

Locate the actual data. Not a description of it — the data itself (or a representative sample).

| Signal | Where to look |
|--------|--------------|
| Database table | Connect and sample |
| API endpoint | Call it, collect responses |
| LLM model output | Run representative prompts, collect completions |
| Image dataset | Read the directory, sample across classes |
| Text corpus | Read documents, sample across categories |
| Embedding index | Query it, collect vectors and distances |
| Feature store | Query it, sample across entities |

If the data doesn't exist yet → stop. "Can't profile a ghost. Get the data first."

### Domain Detection

Classify the source. This determines which profiling template you use:

| Domain | Signals |
|--------|---------|
| **LLM** | Prompt/completion pairs, model API responses, chat transcripts, tool call logs |
| **NLP** | Text documents, labeled corpora, entity annotations, classification datasets |
| **CV** | Image files, bounding box annotations, segmentation masks, classification labels |
| **Embedding** | Vector arrays, distance matrices, index dumps, nearest-neighbor results |
| **Tabular** | Structured rows/columns, CSV/Parquet/DB tables, feature matrices |

Present what you found:

```
Data source: <name>
Domain: <LLM | NLP | CV | Embedding | Tabular>
Location: <where it lives>
Sample: <N items available>
Component consumer: <which HLD S3 component eats this>
```

---

## Phase 2: Profile It

Read the data. Don't skim — monke reads every field, every distribution, every dark corner. Then apply the domain-appropriate template below. **Use the one that fits. Don't force LLM data through a Min/Max/Mean table.**

---

### 2.1 LLM Agent Data

For prompt/completion data, model API outputs, agent interaction logs:

#### Input/Output Shape

| Metric | Value |
|--------|-------|
| Input token length — min/median/p95/max | |
| Output token length — min/median/p95/max | |
| System prompt token count | |
| Tool call frequency (% of requests with tool use) | |
| Multi-turn depth — min/median/max | |

#### Response Variance

Run the same prompt N times (N ≥ 10). Measure:

| Metric | Value |
|--------|-------|
| Output length variance (std of token count) | |
| Semantic similarity across runs (cosine of embeddings) | |
| Format compliance rate (% matching expected schema) | |
| Tool selection consistency (% same tool chosen) | |

If variance is high → this is a data-dependent tool. If variance is low but version-sensitive → versioned-artifact. The profile tells you which.

#### Failure Surface

| Failure Mode | Frequency | Example |
|-------------|-----------|---------|
| Hallucination (fabricated facts) | | |
| Refusal (model declines) | | |
| Format violation (wrong output shape) | | |
| Truncation (hit token limit) | | |
| Tool misuse (wrong tool or wrong args) | | |

#### Cost Profile

| Metric | Value |
|--------|-------|
| Avg tokens per request (input + output) | |
| Cost per 1K requests at current pricing | |
| Latency p50 / p95 / p99 | |

---

### 2.2 NLP Data

For text corpora, labeled datasets, entity annotations:

#### Corpus Shape

| Metric | Value |
|--------|-------|
| Document count | |
| Token length — min/median/p95/max | |
| Language distribution | |
| Encoding (UTF-8 clean? mojibake?) | |
| Script diversity (Latin, CJK, Cyrillic, etc.) | |

#### Vocabulary & Coverage

| Metric | Value |
|--------|-------|
| Unique tokens | |
| OOV rate against target model's tokenizer | |
| Top-20 tokens by frequency | |
| Domain-specific term density | |

#### Label Distribution (if labeled)

| Label | Count | % | Examples |
|-------|-------|---|---------|

Flag class imbalance. If the rarest class is <5% of the dataset, that's an eval risk.

#### Annotation Quality (if annotated)

| Metric | Value |
|--------|-------|
| Inter-annotator agreement (Cohen's κ / Fleiss' κ) | |
| Ambiguous / contested labels (%) | |
| Missing annotations (%) | |

---

### 2.3 CV Data

For image datasets, video frames, visual annotations:

#### Image Shape

| Metric | Value |
|--------|-------|
| Image count | |
| Dimensions — min/median/max (W×H) | |
| Aspect ratio distribution | |
| Color channels (RGB, grayscale, RGBA) | |
| File formats | |
| Avg file size | |
| Corruption rate (unreadable files %) | |

#### Class Distribution (if labeled)

| Class | Count | % | Example description |
|-------|-------|---|-------------------|

Flag imbalance. Same rule as NLP — rarest class <5% is an eval risk.

#### Edge Case Density

| Condition | Frequency | Impact |
|-----------|-----------|--------|
| Occlusion (partial visibility) | | |
| Poor lighting (under/over-exposed) | | |
| Unusual angles | | |
| Small objects (<32px) | | |
| Crowded scenes (overlapping instances) | | |

#### Augmentation Surface

| Transform | Safe? | Notes |
|-----------|-------|-------|
| Horizontal flip | | |
| Rotation ±15° | | |
| Color jitter | | |
| Crop/resize | | |
| Blur/noise | | |

"Safe" = doesn't destroy the label semantics. Flipping a "left turn" sign makes it a "right turn" sign — not safe.

---

### 2.4 Embedding Spaces

For vector indices, embedding model outputs, similarity search systems:

#### Space Geometry

| Metric | Value |
|--------|-------|
| Dimensions | |
| Source model + version | |
| Vector count | |
| Value range (min/max per dimension) | |
| Norm distribution (L2 norms — min/median/max) | |

#### Distance Distributions

| Metric | Value |
|--------|-------|
| Nearest-neighbor distance — min/median/p95 | |
| Same-class distance — mean/std | |
| Cross-class distance — mean/std | |
| Separation ratio (cross / same) | |
| Recall@k for k=1,5,10 | |

If separation ratio < 2.0 → the embedding space is muddy. The component will struggle to distinguish classes. Flag this as a design input.

#### Drift Characteristics

| Metric | Value |
|--------|-------|
| Embedding shift when source model updates (cosine of same-input vectors across versions) | |
| Index rebuild frequency | |
| Staleness window (how long before stale embeddings degrade retrieval) | |

---

### 2.5 Tabular / Classical

For structured data, feature matrices, database tables:

#### Schema

| Field | Type | Nullable | Unique | Example |
|-------|------|----------|--------|---------|

Every field. No skipping. If the schema has 47 columns, you profile 47 columns. "That column is probably fine" is how drift kills you six months later.

#### Distribution Summary

| Field | Min | Max | Mean | Median | Std | Missing % | Top-3 Values |
|-------|-----|-----|------|--------|-----|-----------|--------------|

For numeric: full spread. For categorical: top values + frequencies. For timestamps: range, gaps, seasonality.

#### Drift Surface

| Metric | Baseline Value | Acceptable Range | Detection Method |
|--------|---------------|-----------------|-----------------|

Pick metrics that actually matter:
- Distribution shift (KL divergence, PSI, KS statistic)
- Schema drift (new fields, type changes, new enum values)
- Volume drift (sudden drops, spikes, gaps)
- Quality drift (null rate changes, cardinality shifts)

---

## Phase 3: Draw the Design Lines

This is where profiling stops being a data exercise and starts being a design input.

```
### Design Implications — <data-source>
- Layer 0 types: <what versioned types to define, what distributional expectations to encode>
- Eval thresholds: <what metric baselines this profile establishes>
- Drift detection: <what to monitor post-deployment, what triggers a re-profile>
- Tool subtype confirmation: <versioned-artifact, data-dependent, or both>
```

Be concrete:
- "Embedding dimension is 768, model version is `text-embedding-3-small` v2" → Layer 0 versioned type with `dimensions: 768`, `model_version: "v2"`.
- "Hallucination rate on medical queries is 12%" → eval threshold: `hallucination_rate ≤ 0.12`.
- "Null rate for `user_id` is 0.02%, anything above 1% means upstream is broken" → drift detection trigger.
- "LLM output format compliance is 94%" → eval threshold: `format_compliance ≥ 0.94`.

---

## Phase 4: Confirm

Present the profile summary to the user. Not the raw tables — the design implications.

> "Here's what the data tells us about `<source>`: the shape is `<X>`, the variance is `<Y>`, the drift surface is `<Z>`. This means Layer 0 types need `<A>`, eval tests should threshold on `<B>`, and we should monitor `<C>`. Sound right?"

**⏸ Wait for confirmation.**

- **Confirm** → write the profile
- **Adjust** → revise, re-present
- **Reject** → discard, ask what's wrong

---

## Phase 5: Write It

Write the profile to `monke-docs/recon/profile-<data-source>.md`. If pre-HLD, present inline and let the HLD skill pick it up during L3.

---

## Where This Connects

- **design-specs S1.2:** Tool taxonomy — data-dependent tools require baseline profiles
- **design-specs S7:** Boundary matrix — `data-dependent(<baseline>)` stability annotation references this profile
- **implementation-specs Layer 0:** Versioned types encode distributional expectations from this profile
- **test-specs §4:** Test dataset fixtures derived from this profile
- **monke-seer/registry.md:** Artifact version pins reference the data the artifact was trained/built on
- **monke-seer/experiment.md:** Experiment datasets profiled before experiments run

---

## Anti-Patterns to Refuse

| If you find yourself... | Stop. Do this instead. |
|------------------------|------------------------|
| Profiling without a consuming component in mind | Profile feeds design. No consumer = no profile. You're doing EDA tourism. |
| Writing a 200-cell Jupyter notebook | This is a profile, not an analysis. Domain template + design implications. Done. |
| Forcing LLM data through a Min/Max/Mean table | Use the right domain template. Prompt distributions are not column statistics. |
| Forcing images through a schema table | Images don't have schemas. They have dimensions, classes, and edge cases. Use 2.3. |
| Profiling production data in CI | Profile once at design time. Eval tests use fixture data derived from the profile. Don't burn CI minutes. |
| Creating a separate profiling pipeline | This is a one-time design input, not a runtime system. If you need runtime monitoring, that's a different component. |
| Skipping the drift surface | The drift surface is the whole point. Without it, you've made a pretty spreadsheet, not a design input. |
| Profiling everything "just in case" | Profile what the component consumes. If it doesn't touch it, don't profile it. Thoroughness is not the same as relevance. |
