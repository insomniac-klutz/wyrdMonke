<p align="center">
  <img src="monke-owns/wyrdLogo.png" alt="WyrdMonke" width="600" />
</p>

# WyrdMonke

### *monke write code, monke ship*

> **[Monke Phil](monke-phil.md)** — *the manifesto for why monke replaced your org chart with four spec files and a banana*

---

## What is this

WyrdMonke is what happens when you give a monkey a clipboard, a software engineering degree, and access to Claude Code. It's a full SDLC-in-a-box — design specs, implementation pipelines, test gates, and one unified orchestrator (`/monke`) that walks your AI through the entire process so you don't have to explain "no, write the types first" for the twenty-sixth time.

Clone it. Plug it in. Watch monke think before monke builds, for once.

---

## Install & Invoke

**30-second setup.** One file clone, one command.

**Linux/macOS:**
```bash
mkdir -p .claude/commands
curl -o .claude/commands/monke-init.md https://raw.githubusercontent.com/insomniac-klutz/wyrdMonke/trunk/monke-init.md
```

**Windows (PowerShell):**
```powershell
mkdir -Force ".claude\commands"
curl.exe -o ".claude\commands\monke-init.md" https://raw.githubusercontent.com/insomniac-klutz/wyrdMonke/trunk/monke-init.md
```

Then in Claude Code, run `/monke-init` once. Skills install, project scaffolds, `CLAUDE.md` merges. Ready.

**Then type what you want in plain English.**

```
/monke "<your wish>"
```

Examples:
```
/monke "build a Slack bot that summarizes GitHub PRs"
/monke "add webhook retry with exponential backoff"
/monke "the auth service is leaking memory somewhere, find it"
/monke "refactor the payment module to drop the stripe dep"
/monke "audit this codebase for production readiness"
```

Monke reads your wish + project state, classifies it (greenfield-mvp / greenfield-production / feature-on-existing / bug-fix / refactor-cleanup / investigation), proposes a pipeline, asks **once** to approve, then walks the whole thing end-to-end as permanent lead. One HARD gate up front, one deliverable at the back. Never auto-commits.

**Command shortcuts (the ones you'll actually use):**

| Shape | Behavior |
|-------|----------|
| `/monke "<wish>"` | Natural-language intake — **the front door**. Classifier names the pipeline, one approval, walks end-to-end. |
| `/monke` | State-reader — resume after context death or eyeball current state before acting. |
| `/monke <rigor>` | Set rigor: `light` / `standard` / `thorough`. Persists to `.monke-config.md`. |
| `/monke <skill>` | Direct skill dispatch: `/monke flash`, `/monke design:lld auth-service`, `/monke rage:buggy`, etc. |
| `/monke-status:status [action]` | Dashboard: `show` / `rebuild` / `next` / `blocked` / `resume`. |
| `/monke-ops:commit` | Grouped, confirmed commits. Never auto-invoked by any pipeline. |

Context death? Just re-run `/monke` bare. The Resume-block parser extracts both `Phase:` and `Skill:` from `monke-status.md` and dispatches directly back to the right sub-skill with `resume:<N>` appended. No state loss, no re-approval.

That's the whole flow. Everything below is reference material — the file tree, the skill catalog, the deeper invocation variants (flash brief template, bare-hands install, existing-codebase adapter), the full arc diagram, and philosophy.

---

## Branch Flow

```
screectch → ooh-aah → trunk
```

| Branch | Vibe |
|---|---|
| `screectch` | experimental — things break, monke panic |
| `ooh-aah` | stabilizing — monke groomed the bugs |
| `trunk` | production — ripe banana, ship it |

---

## The Sacred Tree

```
wyrdMonke/
├── monke.md                     # the switchboard — picks the right skill for the job
├── monke-intake.md              # the front door — speak your wish, monke builds the pipeline
├── monke-init.md                # the one ring — copy this, run it, everything installs
├── monke-sync.md                # pull new skills and specs without nuking your work
├── monke-CLAUDE.md              # the scroll Claude reads first
├── monke-mermaid.mmd            # the map of all sacred texts
├── CLAUDE.md                    # the house rules — read before every skill fires
├── monke-claude-settings.json   # the permissions dial — seeded once into .claude/
├── monke-status/                # the all-seeing eye
│   └── status.md                # where am I, what's next, what's broken
├── monke-fut.md                 # walls monke hasn't climbed yet
├── monke-phil.md                # the manifesto — roles died, disciplines didn't, monke thrives
├── monke-log.md                 # every banana has a story — the changelog
├── monke-drafter.md             # the skeleton law — how to write a skill that doesn't embarrass monke
│
├── monke-design/                # monke thinks before monke builds
│   ├── tinker.md                # detect your stack, fill every placeholder, get the dashboard running
│   ├── hld.md                   # the grand dreaming — L1, L2, L3, argue at every fork
│   ├── lld.md                   # decompose one component until it begs for mercy
│   ├── adr.md                   # receipts for every banana chosen over another banana
│   └── oq.md                    # the anxiety manager — triage the 3am hauntings
│
├── monke-implement/             # monke builds (shape before behavior)
│   ├── fill.md                  # customs agent for <<<placeholders>>> — thorough, relentless
│   ├── implement.md             # types → stubs → bodies → tests, gate after gate after gate
│   └── checkpoint.md            # PG-11 — the final boss, demands your signature in blood
│
├── monke-test/                  # monke proves it wasn't hallucinating
│   ├── test-plan.md             # plot every happy path, edge case, and nightmare scenario
│   ├── test-run.md              # run it, watch it fail, categorize the grief, try again
│   └── coverage.md              # find the untouched bananas and shame them into existence
│
├── monke-ops/                   # monke ships (the boring, sacred bits)
│   └── commit.md                # grouped, confirmed commits — never reckless, always receipted
│
├── monke-rage/                  # monke hunts what you're ignoring
│   ├── buggy.md                 # angry monke smells a bug — logic errors, null bombs, swallowed exceptions
│   ├── improv.md                # ambitious monke sees the mountain — perf wins, pattern upgrades, north stars
│   ├── renounce.md              # minimalist monke with a machete — dead weight, cargo cult, duplication
│   ├── haunt.md                 # paranoid monke checking the locks — injection, auth gaps, secret leaks
│   ├── drift.md                 # auditor monke with a magnifying glass — spec says X, code does Y
│   └── echo.md                  # archaeologist monke sweeping the tomb — dead code, zombie imports, ghosts
│
├── monke-seer/                  # monke reads the bones before the build
│   ├── profile.md               # taste the data before you cook with it
│   ├── experiment.md            # LATS with loss curves instead of architecture diagrams
│   ├── registry.md              # pin your artifacts or they'll pin you
│   └── agentify.md              # stupefy in reverse — LATS any component's intelligence
│
├── monke-flash/                # monke builds fast, regrets later (0→80%)
│   ├── spark.md                # the barstool pitch — convince monke before monke writes a line
│   ├── scope.md                # the machete — hack the dream down to size, draw the 80% line
│   ├── sketch.md               # napkin architecture — entities on a cocktail napkin, stack on a dare
│   ├── blitz.md                # happy path speedrun — no tests, no gates, just vibes and working code
│   ├── pulse.md                # poke it, does it scream? smoke test until it stops crying
│   └── snap.md                 # the honest confession — freeze the MVP, list every sin, walk away
│
├── monke-recon/                # monke cleans up what monke broke (80→100%)
│   ├── survey.md               # the morning-after inventory — count the damage, map the wreckage
│   ├── reconstruct.md          # archaeologist mode — dig proper blueprints from the flash rubble
│   ├── gaps.md                 # the gap hunter — everything between "it works" and "ship it"
│   ├── oqs.md                  # the implicit decision exorcist — surface every "I'll fix that later"
│   └── roadmap.md              # the slow crawl plan — phase the pain from 80 to 100
│
├── monke-docs/                  # the ancient texts
│   ├── sdlc-specs.md            # the prophecy (connects all specs)
│   ├── design-specs.md          # how monke thinks before monke builds
│   ├── implementation-specs.md  # how monke builds (shape before behavior)
│   ├── test-specs.md            # how monke proves it works
│   ├── project-specs.md         # the binding scroll (<<<your stuff here>>>)
│   ├── monke-readsme.md         # the origin myth — where these docs crawled out of
│   ├── status-template.md       # blank dashboard, ready to fill
│   ├── hld.md                   # the grand blueprint (empty until you dream)
│   ├── open-questions.md        # the 3am hauntings (empty until you ask)
│   ├── lld/                     # component blueprints (one per brain cell)
│   ├── decisions/               # ADRs — receipts for every banana chosen
│   ├── checkpoints/             # proof monke finished the thing
│   ├── rage-run/                # the scan template — how to format the fury
│   │   └── template.md          # blank rage log, fill with findings
│   └── rage-runs/               # the receipts — every rot found, every scan saved
│
├── monke-owns/                  # monke's face and other sacred relics
│   └── wyrdLogo.png             # the face that launched a thousand commits
│
├── LICENSE                      # Apache 2.0 — monke shares freely
└── README.md                    # you are here. hello.
```

> **Why are some files empty?**
> `hld.md`, `open-questions.md`, `lld/`, `decisions/`, `checkpoints/` — these are boilerplate blanks on purpose. They're the clay, not the pot. Run `/monke-init` then `/monke-design:tinker` (or `/monke-recon:reconstruct` for existing code) and monke fills them with your project's actual design. The specs define the shape. Your project provides the substance.

---

## Release Tags

```
banana-0.1-green    # early, still growing
banana-0.5-yellow   # getting there
banana-1.0-ripe     # ship it
banana-X.X-bruised  # hotfix
```

---

## The Skills

Thirty-seven rituals, organized by when monke needs them. One command to rule them all.

### The One Ring

| Skill | What it does |
|-------|-------------|
| `/monke` | The all-seeing eye. One command, reads your project, names the state, points at the ripest banana. Handles greenfield, existing code, mid-flight projects, resume after context death. Dispatches to every other skill below. Start here. |
| `/monke-intake` | The front door. Type a wish in plain English — `/monke-intake "add webhook retry with exponential backoff"` or use the shortcut `/monke "<request>"`. Classifier + scoper worker-team name what kind of work it is and which components get touched. One HARD approval gate, then intake becomes permanent lead and walks the whole pipeline — dispatching flash / design / implement / test / rage / ops skills as workers, tracking the feature thread end-to-end in `monke-docs/intake/`. |

### The Summoning

| Skill | What it does |
|-------|-------------|
| `/monke-init` | The one ring. Copy this single file, run it, and it clones the repo, installs skills project-local, scaffolds your project, and merges CLAUDE.md — then tells you what to do next. |
| `/monke-sync` | The updater. Pulls fresh skills and specs from upstream without touching your HLD, LLDs, decisions, or any other work. Diffs changed specs and asks before overwriting. |

### The All-Seeing Eye

| Skill | What it does |
|-------|-------------|
| `/monke-status:status` | The dashboard that knows everything. Where you are, what's next, what's blocking you. `rebuild` when it goes stale. `next` when you're lost. `blocked` when something smells funny. |

### Design — monke thinks (`monke-design/`)

| Skill | What it does |
|-------|-------------|
| `/monke-design:tinker` | The caffeinated census taker. Detects your stack, asks you 47 questions, fills every `<<<placeholder>>>` in project-specs and CLAUDE.md, and lights up the dashboard. Run this right after `/monke-init`. |
| `/monke-design:hld` | The grand dreaming. Monke builds the blueprint level by level — context, containers, components — arguing with itself (or an actual Critic agent) at every fork. Resume when your context window inevitably dies mid-vision. |
| `/monke-design:lld` | Zooms into one component and decomposes it until there's nowhere left to hide. Designer proposes, Reviewer attacks, monke refines. Three rounds, then the adults get involved. |
| `/monke-design:adr` | Records why monke chose this banana over that banana. Auto-numbers, links back to the HLD, closes the open questions that were keeping everyone up at night. |
| `/monke-design:oq` | The anxiety manager. Lists everything monke doesn't know yet, triages by "is this actually blocking us?", and resolves them one by one before they metastasize. |

### Flash — monke sprints (`monke-flash/`)

| Skill | What it does |
|-------|-------------|
| `/monke-flash:spark` | "What are we building?" — monke asks the right questions, captures the idea on a napkin, and refuses to let you scope-creep before the first line of code exists. |
| `/monke-flash:scope` | Draws the 80% line. What's in the MVP, what's out, what "done" looks like. Ruthlessly cuts scope because monke knows you'll try to sneak auth in. |
| `/monke-flash:sketch` | Napkin architecture. Core entities, stack picks, folder structure, one data flow. No C4, no LATS — just enough shape to start building. |
| `/monke-flash:blitz` | Build the happy path. Scaffolds, implements core flows, skips ceremony. Uses agent teams for parallel work. Commits after each working flow. |
| `/monke-flash:pulse` | Does it breathe? Runs the MVP, shows it to you, collects feedback, categorizes into fix-now vs defer. Loops back to blitz until you say "good enough." |
| `/monke-flash:snap` | Freezes the MVP. Tags it, produces a manifest of what was built, what was cut, and every shortcut taken. The honest confession that feeds monke-recon. |

### Recon — monke reconstructs (`monke-recon/`)

| Skill | What it does |
|-------|-------------|
| `/monke-recon:survey` | Deep scan of the codebase. Maps components, dependencies, test coverage, code quality. Works with flash MVPs or any existing code. The inventory before the renovation. |
| `/monke-recon:reconstruct` | Reverse-engineers proper HLD and LLDs from existing code. The archaeologist that turns your scrappy MVP into formally documented architecture. |
| `/monke-recon:gaps` | Gap analysis against production requirements. Error handling, security, performance, observability, testing — everything missing, prioritized by blast radius. |
| `/monke-recon:oqs` | Surfaces the implicit decisions. Every shortcut, every "I'll deal with it later," every architectural choice made by defaulting rather than deciding. |
| `/monke-recon:roadmap` | The slow crawl plan. Takes survey + gaps + OQs and builds a phased production roadmap. Each phase connects back to the existing monke pipeline. |

### Implementation — monke builds (`monke-implement/`)

| Skill | What it does |
|-------|-------------|
| `/monke-implement:fill` | The boring-but-necessary one. Detects your stack, suggests values for every `<<<placeholder>>>` in project-specs, and makes you confirm each one like a very thorough customs agent. |
| `/monke-implement:implement` | The Layer 0→3 pipeline. Types first (shape), then stubs (interface), then bodies interleaved with tests (behavior), then integration tests (proof). Each layer has a gate. No skipping. Resume at any layer when you come back tomorrow. |
| `/monke-implement:checkpoint` | The final boss of each phase. Checks every component's test gates, runs system tests end-to-end, writes the checkpoint record, and demands your signature. PG-11 — the gate that never sleeps, never forgives, never skips. |

### Rage — monke hunts (`monke-rage/`)

| Skill | What it does |
|-------|-------------|
| `/monke-rage:buggy` | Angry monke smells something wrong. Logic errors, null bombs, swallowed exceptions, contract violations, race conditions, resource leaks. Also scans tests for false passes and flakiness. |
| `/monke-rage:improv` | Ambitious monke sees the mountain. Performance wins, API ergonomics, pattern upgrades, type safety gaps, readability crimes, architecture smells, north stars you haven't reached yet. |
| `/monke-rage:renounce` | Minimalist monke with a machete. Duplication, dead weight, over-abstraction, cargo-culted patterns, config bloat, tech debt, vestigial code that served its purpose three migrations ago. |
| `/monke-rage:haunt` | Paranoid monke checking the locks. Injection surfaces, auth gaps, hardcoded secrets, weak crypto, vulnerable dependencies, PII in logs, unsafe deserialization. OWASP's greatest hits. |
| `/monke-rage:drift` | Auditor monke with a magnifying glass. HLD says X, code does Y. LLD promises a function that doesn't exist. Status dashboard is lying. ADR chose Option A, code built Option B. |
| `/monke-rage:echo` | Archaeologist monke sweeping the tomb. Unreachable code, unused exports, orphaned files, zombie imports, phantom config, ghost routes, abandoned tests for deleted features. |

### Seer — monke reads the signs (`monke-seer/`)

| Skill | What it does |
|-------|-------------|
| `/monke-seer:profile` | Profiles a data source before you design with it. Schema, distributions, drift surface — the statistical properties that become Layer 0 types and eval baselines. Not an EDA notebook. A design input. |
| `/monke-seer:experiment` | Runs model/approach comparisons as LATS branches with metric evidence. Records results as ADRs. The experiment tracker is `monke-docs/decisions/`, not a separate platform. |
| `/monke-seer:registry` | Tracks versioned artifacts (models, indices, rulesets) with version pins, eval thresholds, and retraining triggers. The model registry as a project-specs binding, not a service. |
| `/monke-seer:agentify` | Stupefy in reverse. Takes any component — traditional or agentic — and LATS whether it should be smarter, dumber, or left alone. Proposes CoALA summaries, Anthropic patterns, and upgrade paths. Records the verdict as an ADR. |

### Testing — monke proves it (`monke-test/`)

| Skill | What it does |
|-------|-------------|
| `/monke-test:test-plan` | Reads your LLD and plots every way the component could break. Happy paths, edge cases, error cases, and if your component is agentic, the special horrors: stale memory, infinite loops, tool failures. Writes the plan directly into the LLD. |
| `/monke-test:test-run` | Runs the tests and checks the gates. Unit→IL-2, integration→IL-3, system→PG-11. When things fail, it doesn't just cry — it categorizes the failure and tells you where to dig. Escalates to you after two failed cycles because at that point, maybe the design is wrong. |
| `/monke-test:coverage` | Counts the untouched bananas. Runs your coverage tool, compares against your threshold, shows you exactly which functions and boundaries have no tests, and maps the gaps back to your LLD test plan so you know what to write next. Coverage is a floor, not a trophy. |

### Ops — monke ships (`monke-ops/`)

| Skill | What it does |
|-------|-------------|
| `/monke-ops:commit` | Grouped, confirmed commits. Takes your changes, proposes logical commit groups, generates a changelog entry, and walks you through each commit with a pause gate. Never auto-commits, never commits secrets, always asks before staging. |

---

## Quick Start — deeper variants

The [Install & Invoke](#install--invoke) section at the top covers the front door. These are the longer paths for when you want more control, are onboarding an existing codebase, or want to pre-stress-test an idea with a pitch doctor before burning Claude Code tokens.

**What happens when you type `/monke "<wish>"` (in more detail):**

1. **Route.** If `.monke-config.md` doesn't exist yet, monke silently chains `/monke-init` first (no ceremony — you don't have to know about it).
2. **Classify + scope.** A 2-teammate scout team (classifier + scoper) reads your wish + project state and names the pipeline: greenfield MVP / greenfield production / feature on existing / bug-fix / refactor-cleanup / investigation.
3. **Propose.** Monke writes a pipeline proposal (steps, expected gates, affected components, estimated human touchpoints) to `monke-docs/intake/<feature-thread-id>.md`.
4. **One HARD gate — PG-1.** You approve / adjust / abort. This is the ONLY gate you MUST hit before work starts.
5. **Walk the pipeline.** `/monke-intake` becomes permanent lead. Dispatches sub-skills (flash / design / implement / test / rage / recon / ops) as workers. Sub-skills surface their own HARD/TRIGGERED gates when something genuinely needs a decision; SOFT gates auto-pass under `light`/`standard` rigor.
6. **Close.** Phase 6 summary. Changes are in the working tree — run `/monke-ops:commit` when you're ready.

Quotation rules:
- **Quoted multi-word** → always routes to intake: `/monke "..."`.
- **Unquoted but >3 words** → also routes to intake: `/monke build a slack bot`.
- **Single unquoted word** → checked against known container names. If unknown, monke asks if you meant a quoted request.
- **Recognized keyword** (rigor level, skill override) → direct dispatch.

### The Flash Way (new idea? start here)

Before you touch Claude Code, open [claude.ai](https://claude.ai) and paste this. Spend 10 minutes. It's cheaper than building the wrong thing.

```
You are a startup advisor with a BS detector. I have a product idea and I need you
to help me stress-test it before I start building. Be direct. Be skeptical. Be useful.

Walk me through these one at a time — don't dump them all at once:

1. THE PITCH — What am I building? Make me say it in one sentence.
   If I can't, the idea isn't clear yet. Help me sharpen it.

2. THE PAIN — Who has this problem? What do they do today instead?
   "Everyone" is not an answer. "Nobody does anything" means the pain isn't real.

3. THE PROOF — What is the absolute minimum thing I could build to prove
   this idea works? Not the product. The experiment. The one flow that makes
   a skeptic say "oh, that's interesting."

4. THE WALLS — What are my hard constraints?
   Timeline, budget, platform, regulatory, "my boss requires X," existing tech
   I'm locked into. Be honest — constraints are features, not bugs.

5. THE TECH — Based on what I've described, what's the core tech surface?
   Language/framework suggestions welcome. LLM/AI components if applicable.
   Data storage needs. External services/APIs. Keep it minimal.

6. THE NAPKIN — Summarize everything into a tight brief I can hand to my
   build tool. Format:
   - Problem (2 sentences)
   - Target user (specific)
   - MVP flows (max 3, as "user does X, system does Y, user sees Z")
   - Hard constraints (bullet list)
   - Core tech requirements (bullet list)
   - What's explicitly OUT of the MVP

Don't let me scope-creep. If I start describing feature #47, drag me back
to the one thing that proves the idea.
```

When you've got the napkin, two ways:

**Shortest path (v2.1+):** just use the front door with your napkin as the wish.
```
/monke "<paste napkin here — or a one-line summary of it>"
```
Monke bootstraps `/monke-init` automatically if config is missing, classifies as greenfield-mvp, and walks the flash chain `spark → scope → sketch → blitz → pulse → snap`. MVP in hours, one HARD gate at the front.

**Explicit path:** traditional flash invocation.
1. Run `/monke-init` in your project.
2. Run `/monke flash` — enters the flash chain directly, `spark` asks its conversation questions.
3. Each flash skill hands back to `/monke`; `/monke` presents the next step.

### The Bare Hands Way

1. Clone the repo: `git clone https://github.com/insomniac-klutz/wyrdMonke.git`
2. Copy `monke-docs/`, `monke-CLAUDE.md`, and `monke-mermaid.mmd` into your project
3. Copy skill dirs to `.claude/commands/` in your project
4. Fill in every `<<<placeholder>>>` in `monke-docs/project-specs.md` by hand
5. Rename `monke-CLAUDE.md` to `CLAUDE.md`
6. Question your life choices

### Already got code? Monke adapts.

Two ways in:

**Front door (v2.1+):** type what you want to change.
```
/monke "<add this feature | fix this bug | refactor that module | audit for prod-readiness>"
```
Monke fast-onboards (scans stack, maps components, tags maturity), intake classifier+scoper names the pipeline against your actual components, one HARD approval gate, walks it end-to-end.

**State-reader:** bare `/monke`.
Fast onboarding scans your codebase, writes a stub HLD + stub LLDs, and presents PG-1 with a rigor recommendation. Confirm or ask for the deeper path via `/monke recon` (full survey → reconstruct → gaps → OQs → roadmap).

It's like a code review, but the reviewer is a monkey with a clipboard.

---

## The Full Arc — from "I have an idea" to "it's in prod and nobody's crying"

```
              you + claude.ai
              ┌─────────────┐
              │  "hear me    │
              │   out..."    │
              │              │
              │  business    │
              │  case +      │
              │  core tech   │
              │  reqs        │
              └──────┬───────┘
                     │
                     │  (bring the napkin to Claude Code)
                     ▼
              ┌──────────────────┐
              │     /monke       │  the all-seeing eye
              │                  │  reads project state
              │  scan → status   │  picks the next move
              │  → hard gate     │  dispatches one skill
              │  → per-component │  loops until shipped
              │    routing       │
              └────────┬─────────┘
                       │
         ┌─────────────┼─────────────┬──────────────┐
         ▼             ▼             ▼              ▼
    monke-flash   monke-recon   production       monke-rage
    ┌─────────┐   ┌──────────┐  ┌──────────┐    ┌─────────┐
    │ spark → │   │ survey → │  │ tinker   │    │ buggy   │
    │ scope → │   │ recon →  │  │ hld/lld  │    │ improv  │
    │ sketch →│   │ gaps →   │  │ adr/oq   │    │ renounce│
    │ blitz → │   │ oqs →    │  │ fill     │    │ haunt   │
    │ pulse ↺ │   │ roadmap  │  │ implement│    │ drift   │
    │ snap    │   │          │  │ checkpt  │    │ echo    │
    │         │   │          │  │ test-*   │    │         │
    └─────────┘   └──────────┘  └──────────┘    └─────────┘
      0→80%         80→100%      80→100%         always
                                                (eternal loop)

                     every path converges.
                  /monke points at one banana.
                        you pick. you confirm.
                        monke dispatches. repeat.
                                  │
                                  ▼
                             ship ──→ rage ──→ ship ──┐
                              ↑                       │
                              └───────────────────────┘
```

**One command. Four modes of building. One monke.**

- **/monke** is the all-seeing eye — the front door for every path below. Reads the project, names the state, recommends one next move. You never need to remember which skill is right; monke picks.
- **claude.ai** is the barstool — hash out the idea before you write a line of code. Business case, core tech reqs, "is this even worth building?" If the answer is no, you just saved yourself a week. If yes, bring the napkin.
- **Flash** is jazz — improvisational, fast, conversational. Monke builds before monke thinks too hard. Gets something breathing in hours, not weeks.
- **Recon** is archaeology — methodical, thorough, reconstructive. Monke digs through the wreckage of what flash built and maps every shortcut, every implicit decision, every "I'll fix that later."
- **The production pipeline** is the disciplined last mile — where monke puts on a lab coat and actually follows the specs. Types before logic, gates before progress, tests before shipping. The boring part that keeps you employed.
- **Rage** is the eternal loop — you're never at 100%. You ship a version, rage finds what's wrong, you fix it, ship again, rage again. Six scans, six flavors of "not yet." There is no clean bill of health. There is only the next version.
- **Ship** is a banana flung into the void. You don't wait for perfection. You ship, rage, ship, rage. This is the way.

### How the specs connect

```
/monke (the all-seeing eye) — reads state, routes to the right skill below

sdlc-specs.md (the prophecy)
    ├── design-specs.md    → monke thinks, produces HLD + LLD
    │   └── rituals: tinker, hld, lld, adr, oq
    ├── implementation-specs.md → monke builds, layer by layer
    │   └── rituals: fill, implement, checkpoint
    ├── test-specs.md      → monke proves it wasn't hallucinating
    │   └── rituals: test-plan, test-run, coverage
    └── project-specs.md   → YOUR tools, YOUR stack, YOUR problem

monke-flash pipeline → 0 to 80% MVP sprint
    └── rituals: spark, scope, sketch, blitz, pulse, snap

monke-recon pipeline → 80 to 100% production crawl
    └── rituals: survey, reconstruct, gaps, oqs, roadmap
```

See [`monke-mermaid.mmd`](monke-mermaid.mmd) for the full relationship graph (it's beautiful, you'll cry).

---

## Philosophy

```
1. keep it flat, keep it simple
2. every file should work standalone
3. if it needs a framework, it's not wyrd enough
4. the best config is the one you actually reuse
5. no code here — english is a programming language now
6. ooh ooh aah aah, the prophecy compiles
```

---

## Contributing

1. Fork the tree
2. Swing to `screectch`
3. Make monke noises (write code)
4. PR into `ooh-aah`
5. If banana ripe, it merges to `trunk`

Commit messages follow the way of the monke:

```
screectch: monke try thing, tree shaking
ooh-aah: monke groomed the bug
trunk: banana flung 🍌
```

---

## License

[Apache 2.0](LICENSE) — do what you want, monke not liable.

---

*the wyrd ones ship faster*

`est. 2026 · ancient monke, modern spells`
