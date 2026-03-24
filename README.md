```
                 _             _
  ___  ___| |_ __ _  ___| | __
 / _ \/ __| __/ _` |/ __| |/ /
| (_) \__ \ || (_| | (__|   <
 \___/|___/\__\__,_|\___|_|\_\
```

### Your AI engineering team. One terminal. 28 skills. Zero meetings.

---

**Built by [Oussama Ammar](https://oussamaammar.com)** ([@daedalium](https://x.com/daedalium)) — solo capitalist and venture builder operating as a "Holding Company of One" from Dubai. Early investor in Stripe, Algolia, PayFit, Trainline. Previously co-founded [The Family](https://www.thefamily.co), a European startup accelerator whose portfolio raised $1.5B+.

> **Standing on the shoulders of a giant.**
>
> ostack is a fork of [gstack](https://github.com/garrytan/gstack) by [Garry Tan](https://x.com/garrytan) — Y Combinator President and legendary builder. Garry open-sourced gstack as an AI-powered engineering team for Claude Code, and made it free and MIT-licensed for everyone. That generosity is rare at his level, and it gave people like me a foundation that would have taken months to build from scratch. All credit for the original architecture, skills system, browser engine, and sprint philosophy goes to Garry and the gstack contributors.
>
> ostack takes that foundation and injects a different soul — my own. Different philosophy, different mental models, different decision frameworks. Same incredible bones.

---

## What is ostack?

ostack turns [Claude Code](https://docs.anthropic.com/en/docs/claude-code) into a virtual engineering team. Not a copilot that completes your lines — a full team that thinks, plans, reviews, tests, and ships.

You get a **CEO** who rethinks the product, an **eng manager** who locks architecture, a **designer** who catches AI slop, a **staff engineer** who finds production bugs, a **QA lead** who opens a real browser and clicks through your app, a **security officer** who runs OWASP + STRIDE audits, and a **release engineer** who ships the PR.

28 specialists. All slash commands. All Markdown. All free.

## Who this is for

- **Solo founders** — one person, institutional output. The org chart is gone.
- **Technical CEOs** — who still want to ship, not just manage.
- **First-time Claude Code users** — structured roles instead of a blank prompt.
- **Staff engineers** — rigorous review, QA, and release automation on every PR.

## How ostack differs from gstack

ostack is not a cosmetic rebrand. It's a philosophical fork.

| Dimension | gstack | ostack |
|-----------|--------|--------|
| **Philosophy** | Engineering-first | Operator-first — the [Sovereign Builder](ETHOS.md) worldview |
| **Mental models** | Standard best practices | Hybrid: SHU-HA-RI, Power Law concentration, Contrarian by Construction |
| **Decision framework** | Process-driven | Taste-driven — "Context Over Principle" as the meta-rule |
| **Communication** | Comprehensive | Compressed — no hedging, no filler, conviction first |
| **CEO review** | Strategic review | Merged cognitive patterns — thinks like a founder who builds |
| **Builder ethos** | Implicit | Explicit — [ETHOS.md](ETHOS.md) codifies the worldview |
| **Code quality** | High | Garry's engineering rigor cranked to 11 |
| **Telemetry** | Opt-in analytics | Same opt-in approach, removed external references |
| **Identity** | gstack | ostack — "Ouss's stack" |

The core idea: AI collapses the org chart. A single operator with the right tools achieves institutional output — not because the tools replace judgment, but because they multiply it.

## Quick start

```bash
# 1. Install (30 seconds)
git clone https://github.com/mr-daedalium/ostack.git ~/.claude/skills/ostack
cd ~/.claude/skills/ostack && ./setup

# 2. Try it
/office-hours          # Describe what you're building
/plan-ceo-review       # Challenge the scope
/review                # Find bugs in your branch
/qa https://your.app   # QA with a real browser
/ship                  # Ship the PR
```

Stop after `/qa`. You'll know if this is for you.

## See it work

```
You:    I want to build a daily briefing app for my calendar.
You:    /office-hours
Claude: [asks about the pain — specific examples, not hypotheticals]

You:    Multiple Google calendars, events with stale info, wrong locations.
        Prep takes forever and the results aren't good enough...

Claude: I'm going to push back on the framing. You said "daily briefing
        app." But what you actually described is a personal chief of
        staff AI.
        [extracts 5 capabilities you didn't realize you were describing]
        [challenges 4 premises — you agree, disagree, or adjust]
        [generates 3 implementation approaches with effort estimates]
        RECOMMENDATION: Ship the narrowest wedge tomorrow, learn from
        real usage. The full vision is a 3-month project — start with
        the daily briefing that actually works.
        [writes design doc -> feeds into downstream skills automatically]

You:    /plan-ceo-review
        [reads the design doc, challenges scope, runs 10-section review]

You:    /plan-eng-review
        [ASCII diagrams for data flow, state machines, error paths]
        [test matrix, failure modes, security concerns]

You:    Approve plan. Exit plan mode.
        [writes 2,400 lines across 11 files. ~8 minutes.]

You:    /review
        [AUTO-FIXED] 2 issues. [ASK] Race condition -> you approve fix.

You:    /qa https://staging.myapp.com
        [opens real browser, clicks through flows, finds and fixes a bug]

You:    /ship
        Tests: 42 -> 51 (+9 new). PR: github.com/you/app/pull/42
```

You said "daily briefing app." The agent said "you're building a chief of staff AI" — because it listened to your pain, not your feature request. Eight commands, end to end. That is not a copilot. That is a team.

## The sprint

ostack is a process, not a collection of tools. The skills run in the order a sprint runs:

```
Think -> Plan -> Build -> Review -> Test -> Ship -> Reflect
```

Each skill feeds into the next. `/office-hours` writes a design doc that `/plan-ceo-review` reads. `/plan-eng-review` writes a test plan that `/qa` picks up. `/review` catches bugs that `/ship` verifies are fixed. Nothing falls through the cracks because every step knows what came before it.

### Your team

| Phase | Skill | Specialist | What they do |
|-------|-------|-----------|--------------|
| **Think** | `/office-hours` | YC Office Hours | Six forcing questions that reframe your product before code. Challenges premises, generates alternatives. Design doc feeds downstream. |
| **Plan** | `/plan-ceo-review` | CEO / Founder | Find the 10-star product. Four modes: Expansion, Selective Expansion, Hold Scope, Reduction. |
| | `/plan-eng-review` | Eng Manager | Lock architecture, data flow, edge cases, and tests. Forces hidden assumptions into the open. |
| | `/plan-design-review` | Senior Designer | Rates each design dimension 0-10. Explains what a 10 looks like. Fixes the plan. AI Slop detection. |
| | `/design-consultation` | Design Partner | Build a complete design system from scratch. Typography, color, layout, spacing, motion. |
| | `/autoplan` | Review Pipeline | One command, fully reviewed plan. CEO + design + eng review with encoded decision principles. |
| **Build** | `/browse` | QA Engineer | Real Chromium browser, real clicks, real screenshots. ~100ms per command. |
| | `/investigate` | Debugger | Systematic root-cause debugging. Iron Law: no fixes without investigation. |
| **Review** | `/review` | Staff Engineer | Find bugs that pass CI but blow up in production. Auto-fixes the obvious ones. |
| | `/codex` | Second Opinion | Independent review from OpenAI Codex CLI. Cross-model analysis. |
| | `/design-review` | Designer Who Codes | Live-site visual audit + fix loop. Atomic commits, before/after screenshots. |
| | `/cso` | Chief Security Officer | OWASP Top 10 + STRIDE. Zero-noise: 8/10+ confidence gate, exploit scenarios. |
| **Test** | `/qa` | QA Lead | Real browser testing. Find bugs, fix them, re-verify. Auto-generates regression tests. |
| | `/qa-only` | QA Reporter | Same methodology, report only. No code changes. |
| | `/benchmark` | Performance Engineer | Baseline page loads, Core Web Vitals, resource sizes. Before/after on every PR. |
| **Ship** | `/ship` | Release Engineer | Sync main, run tests, audit coverage, push, open PR. |
| | `/land-and-deploy` | Release Engineer | Merge, wait for CI, verify production health. One command. |
| | `/canary` | SRE | Post-deploy monitoring. Console errors, perf regressions, page failures. |
| **Reflect** | `/retro` | Eng Manager | Team-aware weekly retro. Per-person breakdowns, shipping streaks, growth areas. `/retro global` across all projects. |
| | `/document-release` | Technical Writer | Update all project docs to match what shipped. Catches stale READMEs. |

### Power tools

| Skill | What it does |
|-------|-------------|
| `/careful` | Warns before destructive commands (rm -rf, DROP TABLE, force-push). Override any warning. |
| `/freeze` | Restrict file edits to one directory. Prevents accidental changes while debugging. |
| `/guard` | `/careful` + `/freeze` in one command. Maximum safety for prod work. |
| `/unfreeze` | Remove the `/freeze` boundary. |
| `/setup-deploy` | One-time deploy config for `/land-and-deploy`. Detects your platform automatically. |
| `/setup-browser-cookies` | Import cookies from Chrome, Arc, Brave, or Edge into the headless session. |
| `/ostack-upgrade` | Self-updater. Detects global vs vendored install, syncs both, shows what changed. |

**[Deep dives with examples for every skill ->](docs/skills.md)**

## The effort compression table

This is why ostack exists. The ratio between human-team time and AI-assisted time:

| Task type | Human team | With ostack | Compression |
|-----------|-----------|-------------|-------------|
| Boilerplate / scaffolding | 2 days | 15 min | ~100x |
| Test writing | 1 day | 15 min | ~50x |
| Feature implementation | 1 week | 30 min | ~30x |
| Bug fix + regression test | 4 hours | 15 min | ~20x |
| Architecture / design | 2 days | 4 hours | ~5x |
| Research / exploration | 1 day | 3 hours | ~3x |

The last 10% of completeness that teams used to skip? It costs seconds now. Do the full thing. Every time.

## Install

**Requirements:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [Git](https://git-scm.com/), [Bun](https://bun.sh/) v1.0+, [Node.js](https://nodejs.org/) (Windows only)

### Option A: Global install (recommended)

Open Claude Code and paste this. Claude does the rest.

> Install ostack: run **`git clone https://github.com/mr-daedalium/ostack.git ~/.claude/skills/ostack && cd ~/.claude/skills/ostack && ./setup`** then add a "ostack" section to CLAUDE.md that says to use the /browse skill from ostack for all web browsing, never use mcp\_\_claude-in-chrome\_\_\* tools, and lists the available skills: /office-hours, /plan-ceo-review, /plan-eng-review, /plan-design-review, /design-consultation, /review, /ship, /land-and-deploy, /canary, /benchmark, /browse, /qa, /qa-only, /design-review, /setup-browser-cookies, /setup-deploy, /retro, /investigate, /document-release, /codex, /cso, /autoplan, /careful, /freeze, /guard, /unfreeze, /ostack-upgrade. Then ask the user if they also want to add ostack to the current project so teammates get it.

### Option B: Per-project install (teammates get it too)

> Add ostack to this project: run **`cp -Rf ~/.claude/skills/ostack .claude/skills/ostack && rm -rf .claude/skills/ostack/.git && cd .claude/skills/ostack && ./setup`** then add a "ostack" section to this project's CLAUDE.md that says to use the /browse skill from ostack for all web browsing, never use mcp\_\_claude-in-chrome\_\_\* tools, lists the available skills: /office-hours, /plan-ceo-review, /plan-eng-review, /plan-design-review, /design-consultation, /review, /ship, /land-and-deploy, /canary, /benchmark, /browse, /qa, /qa-only, /design-review, /setup-browser-cookies, /setup-deploy, /retro, /investigate, /document-release, /codex, /cso, /careful, /freeze, /guard, /unfreeze, /ostack-upgrade, and tells Claude that if ostack skills aren't working, run `cd .claude/skills/ostack && ./setup` to build the binary and register skills.

Real files get committed to your repo (not a submodule), so `git clone` just works. Everything lives inside `.claude/`. Nothing touches your PATH or runs in the background.

### Option C: Codex, Gemini CLI, Kiro, or Cursor

ostack works on any agent that supports the [SKILL.md standard](https://github.com/anthropics/claude-code).

```bash
# Per-repo
git clone https://github.com/mr-daedalium/ostack.git .agents/skills/ostack
cd .agents/skills/ostack && ./setup --host codex

# Global
git clone https://github.com/mr-daedalium/ostack.git ~/ostack
cd ~/ostack && ./setup --host codex

# Auto-detect installed agents
git clone https://github.com/mr-daedalium/ostack.git ~/ostack
cd ~/ostack && ./setup --host auto
```

All 28 skills work across all supported agents.

## Parallel sprints

ostack works well with one sprint. It gets interesting with ten running at once.

[Conductor](https://conductor.build) runs multiple Claude Code sessions in parallel — each in its own isolated workspace. One session on `/office-hours`, another on `/review`, a third implementing a feature, a fourth running `/qa`. All at the same time. The sprint structure is what makes parallelism work — without a process, ten agents is ten sources of chaos. With a process, each agent knows exactly what to do and when to stop.

## Philosophy

ostack is built on seven principles. The full manifesto is in [ETHOS.md](ETHOS.md). The short version:

1. **The Sovereign Builder** — AI collapses the org chart. One operator with compound tools achieves institutional output.
2. **Often Wrong, Never in Doubt** — Conviction expressed as speed. Ship, learn, course-correct.
3. **Context Over Principle** — The meta-rule. When a principle becomes dogma, it becomes the enemy.
4. **Search Before Building** — Four layers of knowledge: solved, tried-and-true, new-and-popular, first principles.
5. **Leverage Over Effort** — Code, media, capital. Master at least two.
6. **Taste as Quality Bar** — AI makes shipping easy. That makes taste more important, not less.
7. **SHU-HA-RI** — Obey the rules, break them deliberately, transcend them entirely.

> *"The worst outcome is a complete, well-engineered product that nobody needed. The best outcome is a product born from conviction, built with leverage, filtered through taste, and shipped at a speed that would have been impossible two years ago — by one person who knew when to follow the rules and when to throw them away."*

## Architecture

```
ostack/
├── browse/              # Headless Chromium CLI (Playwright, ~100ms/cmd)
│   ├── src/             # CLI + persistent server + 50+ commands
│   └── dist/            # Compiled binary
├── [skill-name]/        # 28 skill directories, each with SKILL.md.tmpl
├── scripts/             # Build tooling: template generator, health checks
├── test/                # 3-tier test suite: validation, E2E, LLM-as-judge
├── bin/                 # CLI utilities (config, update, repo-mode detection)
├── ETHOS.md             # Builder philosophy
├── SKILL.md.tmpl        # Root skill template (generates SKILL.md)
└── setup                # One-command installer for all agent platforms
```

**Tech stack:** TypeScript, Bun, Playwright, Anthropic SDK

**How skills work:** Each skill is a `SKILL.md.tmpl` template that gets compiled into `SKILL.md` — a prompt that Claude reads and executes. Templates use shared partials (preamble, browse setup, effort tables) so all 28 skills stay consistent. Skills are just Markdown. No plugins, no APIs, no lock-in.

**How browse works:** A persistent headless Chromium server that starts on first use (~3s) and stays alive for 30 minutes. Each command takes ~100ms. 50+ commands: navigate, click, fill, screenshot, snapshot (accessibility tree with clickable refs), responsive testing, cookie import, network/console inspection. The agent treats it like a real browser because it is one.

## Docs

| Doc | What it covers |
|-----|---------------|
| [Skill Deep Dives](docs/skills.md) | Philosophy, examples, and workflow for every skill |
| [Builder Ethos](ETHOS.md) | The Sovereign Builder philosophy |
| [Architecture](ARCHITECTURE.md) | Design decisions and system internals |
| [Browser Reference](BROWSER.md) | Full command reference for `/browse` |
| [Contributing](CONTRIBUTING.md) | Dev setup, testing, contributor mode |
| [Changelog](CHANGELOG.md) | What's new in every version |

## Privacy & Telemetry

- **Default is off.** Nothing is sent unless you opt in.
- **What's sent (if you opt in):** skill name, duration, success/fail, version, OS.
- **What's never sent:** code, file paths, repo names, prompts, or any content.
- **Change anytime:** `ostack-config set telemetry off`

Local analytics are always available via `ostack-analytics`.

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Skill not showing up | `cd ~/.claude/skills/ostack && ./setup` |
| `/browse` fails | `cd ~/.claude/skills/ostack && bun install && bun run build` |
| Stale install | Run `/ostack-upgrade` or set `auto_upgrade: true` in `~/.ostack/config.yaml` |
| Windows issues | Node.js required alongside Bun ([bun#4253](https://github.com/oven-sh/bun/issues/4253)). Both `bun` and `node` must be on PATH. |
| Claude can't see skills | Add an ostack section to your project's `CLAUDE.md` listing the available skills |

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for dev setup, the 3-tier test suite, and contributor mode.

```bash
bun install             # install dependencies
bun test                # free tests (<2s)
bun run test:evals      # paid evals (~$4/run, needs ANTHROPIC_API_KEY)
bun run skill:check     # health dashboard for all skills
```

## License

MIT. Forked from [gstack](https://github.com/garrytan/gstack) by [Garry Tan](https://x.com/garrytan).

Go build something.

## Links

[\![Hypercommit](https://img.shields.io/badge/Hypercommit-DB2475)](https://hypercommit.com/ostack)
