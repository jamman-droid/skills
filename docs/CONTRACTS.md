# Contracts

Shared data formats and interfaces. Every skill reads this; none restates it.

## Config — `docs/agents/planproper.md` (in the target repo)

Keys and defaults:

| key | default | meaning |
|---|---|---|
| `budget_pct` | `40` | a phase must be built AND reviewed within this % of the context window |
| `plans_dir` | `work/plans` | plan files + corpus map |
| `notes_dir` | `work/notes` | research notes |
| `handoff_dir` | `work/handoffs` | session handoff docs |
| `audits_dir` | `work/plans/audits` | append-only audit logs |
| `render_dir` | `.stage` | visual artifacts (gitignored) |
| `tracker` | `local` | `local` \| `github` \| `linear` |
| `glossary` | `CONTEXT.md` | project glossary location |
| `adr_dir` | `docs/adr` | decision records |

Skills read the config when present and use these defaults silently when not.
`/planproper` bootstraps the file interactively on first run (from its sibling
`CONFIG-TEMPLATE.md`). **Budget rule:** any skill nearing `budget_pct`% of the
window checkpoints through `/relay` instead of pushing on degraded.

## Decisions ledger (produced by /press; embedded in plans and the map)

```markdown
### Decisions
- [D-01] <one-line decision> — by: user | suggested-accepted | agent-default ⚑ · opens: <what this unblocks>
```

`agent-default` entries carry `⚑` and are audit priorities. IDs are unique per
document.

## Plan file — `<plans_dir>/NN-<slug>.md`

```markdown
# <Title>
Goal: <one line>
Depends on: <NN-slug, … | none>
Status: raw | improved | decomposed | executing | done | audited

## Acceptance criteria
- [ ] <independently verifiable statement>

## Plan
<body>

## Decisions      ← ledger format above
## Phases         ← added by /slice; format below
```

## Corpus map — `<plans_dir>/MAP.md`

Sections, in order:

1. `## Vision` — goals, what the system looks like, what it does, long-term
   scope. Written from the vision interview; everything downstream anchors here.
2. `## Plans` — table: plan file · one-liner · status.
3. `## Up next` — dependency edges between plans; which are takeable now;
   an explicit "start here".
4. `## Decisions` — corpus-level ledger.
5. `## Horizon` — future work known but not yet specifiable as a plan.
6. `## Out of scope` — ruled out, with the reason.

The map is the single source of plan status.

## Phases (produced by /slice; appended to the plan file)

```markdown
## Phases
### Phase 1 — <name>   [target: one session ≤ budget_pct%]
Delivers: <demoable outcome>
Slices:
1. <end-to-end behavior> (blocked by: none)
2. <…> (blocked by: 1)
Checklist:                      ← the audit contract
- [ ] <verifiable item>
```

3–6 slices per phase. The first slice of Phase 1 threads the whole system
end-to-end, however thinly.

## Completion evidence (produced by /inspect, Completion axis)

Per checklist item: `done | partial | missing | done-differently` plus one
line of evidence (file path, test name, or observed behavior). Written to
`<audits_dir>/YYYY-MM-DD-<plan>-audit.md` — append-only, never rewritten.

## Plan-quality review (produced by /inspect, plan-quality mode)

The four plan-quality axes (Coverage, Verifiability, Consistency,
Assumptions), each with its findings and verdict, written to
`<audits_dir>/YYYY-MM-DD-<plan>-review.md`. Overwritten on re-review — unlike
the append-only audit log, a review reflects the plan's current state.

## Handoff doc (produced by /relay) — `<handoff_dir>/YYYY-MM-DD-<slug>.md`

Sections: Objective · State (**verified** vs **assumed**, split) · Decisions ·
Next steps (ordered; step 1 executable cold) · Artifacts (paths) · Re-entry
(exact command or prompt). The reply that saves it ends with
`Continuation: <absolute path>` on its own line.

## Render requests (consumed by /stage)

The caller passes: subject (one line) · artifact path or content · content
kinds (`plan | comparison | diagram | table | code`) · the decision points the
human should annotate. Artifacts land in `<render_dir>/`.

## Who fires whom

- `/planproper` (user-only) → `/press`, `/coin`, `/dig`, `/slice`, `/inspect`,
  `/stage`, `/relay`
- `/press` → `/coin` (docs mode), `/relay` (budget checkpoint)
- `/slice` → `/stage` (optional graph render)
- `/inspect` → `/stage` (optional findings render); also fired directly by
  unattended runs for completion audits
- Never: restating another skill's mechanics — delegate by name.
