# planproper

**Plan properly, then prove it got built.**

A planning-first agent-skills system. You bring an idea — or a pile of half-baked
plans — and planproper interviews it to clarity, renders it for review, cuts it
into buildable slices sized to fit a context window, and afterward checks that
every piece actually got done.

Built for real engineering work, and simple enough to hand to someone learning it.

## The two commands

You only need two:

```
/planproper   Make plans. Hand it anything — a folder of rough plans, a one-line
              idea, or "break this into phases." It figures out which and takes it
              all the way to a reviewed, phased plan.

/inspect      Check plans. Before you build, it reviews plan quality (gaps, vague
              steps, missing acceptance criteria). After something builds it, it
              audits completion — evidence per plan item, and fixes the safe gaps.
```

Everything else is internal — the agent reaches for it on its own.

## Install

```sh
npx skills add jamman-droid/skills
```

Pick the skills, pick your agent (Claude Code, Codex, Cursor, and more are
auto-detected). Then, in an agent that exposes skills as slash commands:

```
/planproper turn these ten plans into one coherent, phased build
```

## What's inside

Two front doors over six disciplines. The disciplines follow one rule: a command
you type may call a discipline, but never another command you type — so the
pieces compose without tangling.

| Skill | You type it? | What it does |
|---|---|---|
| **planproper** | ✅ | Master planner: intake a plan corpus · plan an idea · decompose into phases · route an audit |
| **inspect** | ✅ | Plan-quality review, or completion audit — auto-detected |
| press | — | The interview: one question at a time, with a recommendation, to a signed-off decision ledger |
| slice | — | Cut a plan into vertical slices, bin them into context-budget phases |
| coin | — | Keep a glossary + decision log; audit code against them |
| dig | — | Answer one technical question from primary sources, saved as a cited note |
| stage | — | Render a dense plan as an interactive page you annotate in the browser |
| relay | — | Compress a session into a handoff a fresh agent can resume from cold |

## How it works

**Vertical, not horizontal.** planproper cuts work into slices that each thread
*every* layer they touch — schema → logic → interface → test — so each slice is
demoable on its own. The first slice runs thin but end-to-end through the whole
thing, proving the path before anything widens. No "build all the models, then all
the endpoints, then wire it up" and hope it meets in the middle.

**Sized to the window.** Slices are binned into phases, each estimated to build
*and* review inside one session under 40% of the model's context window
(configurable). A phase that would overflow gets split — so an unattended run never
degrades halfway through a chunk it can't finish.

**Proven, not assumed.** Every plan item carries an independently verifiable
acceptance criterion. After a build, `/inspect` walks the checklist and grades each
item — done / partial / missing / done-differently — with evidence, then auto-fixes
the safe gaps and re-checks. It reports what it can't safely fix rather than
papering over it.

**Reviewed by eye when it helps.** Dense plans, dependency graphs, and comparisons
render as an interactive page you mark up in a browser; your annotations flow back
into the plan.

## Design principles

- **Portable.** The core of every skill is plain instructions that run on any
  capable agent. Agent-specific tricks are optional enhancements, never load-bearing.
- **Composable.** Small, deep skills behind two simple front doors — adapt them,
  rename them, make them yours. See [`HOUSE-RULES.md`](HOUSE-RULES.md).
- **Contract-driven.** Every artifact — plans, the corpus map, phases, completion
  evidence, handoffs — has one defined shape in [`docs/CONTRACTS.md`](docs/CONTRACTS.md),
  so the skills interlock without drifting.
- **Config over hardcode.** Budget percentage, directories, and tracker choice live
  in one config file planproper writes on first run.

## Configuration

On first run, `/planproper` writes `docs/agents/planproper.md` with sensible
defaults (40% context budget, local plan storage). Edit it to taste; every skill
reads from it. Nothing else to set up.

## License

MIT — see [`LICENSE`](LICENSE). Use it, fork it, teach with it, make it your own.
