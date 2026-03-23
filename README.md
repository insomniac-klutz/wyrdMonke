# 🐒 WyrdMonke

### *monke write code, monke ship*

> **[Five roles walked into a standup. None walked out.](monke-phil.md)** — *the manifesto for why monke replaced your org chart with four spec files and a banana*

---

## What is this

WyrdMonke is what happens when you give a monkey a clipboard, a software engineering degree, and access to Claude Code. It's a full SDLC-in-a-box — design specs, implementation pipelines, test gates, and 15 slash commands that walk your AI through the entire process so you don't have to explain "no, write the types first" for the fourteenth time.

Clone it. Plug it in. Watch monke think before monke builds, for once.

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
├── monke-init.md                # the one ring — copy this, run it, everything installs
├── monke-sync.md                # pull new skills and specs without nuking your work
├── monke-CLAUDE.md              # the scroll Claude reads first
├── monke-mermaid.mmd            # the map of all sacred texts
├── monke-status/                # the all-seeing eye
│   └── status.md                # where am I, what's next, what's broken
├── monke-fut.md                 # walls monke hasn't climbed yet
│
├── monke-design/                # monke thinks before monke builds
│   ├── tinker.md                # detect your stack, fill every placeholder, get the dashboard running
│   ├── recon.md                 # archaeologist mode — dig up the HLD from existing ruins
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
├── monke-rage/                  # monke hunts what you're ignoring
│   └── sonar.md                 # scan the codebase, find the rot, log the rage
│
├── monke-docs/                  # the ancient texts
│   ├── sdlc-specs.md            # the prophecy (connects all specs)
│   ├── design-specs.md          # how monke thinks before monke builds
│   ├── implementation-specs.md  # how monke builds (shape before behavior)
│   ├── test-specs.md            # how monke proves it works
│   ├── project-specs.md         # the binding scroll (<<<your stuff here>>>)
│   ├── status-template.md       # blank dashboard, ready to fill
│   ├── hld.md                   # the grand blueprint (empty until you dream)
│   ├── open-questions.md        # the 3am hauntings (empty until you ask)
│   ├── lld/                     # component blueprints (one per brain cell)
│   ├── decisions/               # ADRs — receipts for every banana chosen
│   ├── checkpoints/             # proof monke finished the thing
│   └── rage-run/                # sonar scan logs — receipts for every rot found
│
├── LICENSE                      # Apache 2.0 — monke shares freely
└── README.md                    # you are here. hello.
```

> **Why are some files empty?**
> `hld.md`, `open-questions.md`, `lld/`, `decisions/`, `checkpoints/` — these are boilerplate blanks on purpose. They're the clay, not the pot. Run `/monke-init` then `/monke-design:tinker` (or `/monke-design:recon` for existing code) and monke fills them with your project's actual design. The specs define the shape. Your project provides the substance.

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

Sixteen rituals, organized by when monke needs them.

### The Summoning

| Skill | What it does |
|-------|-------------|
| `/monke-init` | The one ring. Copy this single file, run it, and it clones the repo, installs all 13 skills project-local, scaffolds your project, and merges CLAUDE.md — then tells you what to do next. |
| `/monke-sync` | The updater. Pulls fresh skills and specs from upstream without touching your HLD, LLDs, decisions, or any other work. Diffs changed specs and asks before overwriting. |

### The All-Seeing Eye

| Skill | What it does |
|-------|-------------|
| `/monke-status:status` | The dashboard that knows everything. Where you are, what's next, what's blocking you. `rebuild` when it goes stale. `next` when you're lost. `blocked` when something smells funny. |

### Design — monke thinks (`monke-design/`)

| Skill | What it does |
|-------|-------------|
| `/monke-design:tinker` | The caffeinated census taker. Detects your stack, asks you 47 questions, fills every `<<<placeholder>>>` in project-specs and CLAUDE.md, and lights up the dashboard. Run this right after `/monke-init`. |
| `/monke-design:recon` | Stares at your existing codebase like an archaeologist at a crime scene, reverse-engineers an HLD from the wreckage, and surfaces every design decision you've been pretending doesn't exist. Therapy for your repo. |
| `/monke-design:hld` | The grand dreaming. Monke builds the blueprint level by level — context, containers, components — arguing with itself (or an actual Critic agent) at every fork. Resume when your context window inevitably dies mid-vision. |
| `/monke-design:lld` | Zooms into one component and decomposes it until there's nowhere left to hide. Designer proposes, Reviewer attacks, monke refines. Three rounds, then the adults get involved. |
| `/monke-design:adr` | Records why monke chose this banana over that banana. Auto-numbers, links back to the HLD, closes the open questions that were keeping everyone up at night. |
| `/monke-design:oq` | The anxiety manager. Lists everything monke doesn't know yet, triages by "is this actually blocking us?", and resolves them one by one before they metastasize. |

### Implementation — monke builds (`monke-implement/`)

| Skill | What it does |
|-------|-------------|
| `/monke-implement:fill` | The boring-but-necessary one. Detects your stack, suggests values for every `<<<placeholder>>>` in project-specs, and makes you confirm each one like a very thorough customs agent. |
| `/monke-implement:implement` | The Layer 0→3 pipeline. Types first (shape), then stubs (interface), then bodies interleaved with tests (behavior), then integration tests (proof). Each layer has a gate. No skipping. Resume at any layer when you come back tomorrow. |
| `/monke-implement:checkpoint` | The final boss of each phase. Checks every component's test gates, runs system tests end-to-end, writes the checkpoint record, and demands your signature. PG-11 — the gate that never sleeps, never forgives, never skips. |

### Rage — monke hunts (`monke-rage/`)

| Skill | What it does |
|-------|-------------|
| `/monke-rage:sonar` | The codebase scanner with six modes of fury. `buggy` finds bugs, `improv` finds north stars, `renounce` finds dead weight, `haunt` finds security holes, `drift` finds spec-code divergence, `echo` finds dead code. Reads every file in scope, triages by severity, logs to `monke-docs/rage-run/`. Sonar doesn't fix — sonar finds. You fix. |

### Testing — monke proves it (`monke-test/`)

| Skill | What it does |
|-------|-------------|
| `/monke-test:test-plan` | Reads your LLD and plots every way the component could break. Happy paths, edge cases, error cases, and if your component is agentic, the special horrors: stale memory, infinite loops, tool failures. Writes the plan directly into the LLD. |
| `/monke-test:test-run` | Runs the tests and checks the gates. Unit→IL-2, integration→IL-3, system→PG-11. When things fail, it doesn't just cry — it categorizes the failure and tells you where to dig. Escalates to you after two failed cycles because at that point, maybe the design is wrong. |
| `/monke-test:coverage` | Counts the untouched bananas. Runs your coverage tool, compares against your threshold, shows you exactly which functions and boundaries have no tests, and maps the gaps back to your LLD test plan so you know what to write next. Coverage is a floor, not a trophy. |

---

## Quick Start

### The Ritual Way (recommended)

1. Grab the one ring:

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
2. Open your target project in Claude Code
3. Whisper `/monke-init`
4. Skills install. Project scaffolds. CLAUDE.md merges. Monke is ready.
5. Run `/monke-design:tinker` — answer the questions, monke fills the scrolls, you sip coffee.
6. Run `/monke-status:status` to see your dashboard.

### The Bare Hands Way

1. Clone the repo: `git clone https://github.com/insomniac-klutz/wyrdMonke.git`
2. Copy `monke-docs/`, `monke-CLAUDE.md`, and `monke-mermaid.mmd` into your project
3. Copy skill dirs to `.claude/commands/` in your project
4. Fill in every `<<<placeholder>>>` in `monke-docs/project-specs.md` by hand
5. Rename `monke-CLAUDE.md` to `CLAUDE.md`
6. Question your life choices

### Already got code? Monke adapts.

Run `/monke-init` then `/monke-design:recon` — it reads your existing codebase, reverse-engineers an HLD from the wreckage, and surfaces every question you've been ignoring across design, implementation, and testing.

It's like a code review, but the reviewer is a monkey with a clipboard.

---

## How the specs connect

```
sdlc-specs.md (the prophecy)
    ├── design-specs.md    → monke thinks, produces HLD + LLD
    │   └── rituals: tinker, update, hld, lld, adr, oq
    ├── implementation-specs.md → monke builds, layer by layer
    │   └── rituals: fill, implement, checkpoint
    ├── test-specs.md      → monke proves it wasn't hallucinating
    │   └── rituals: test-plan, test-run, coverage
    └── project-specs.md   → YOUR tools, YOUR stack, YOUR problem

monke-status.md ← the all-seeing eye, reads everything, tells you what's next
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
