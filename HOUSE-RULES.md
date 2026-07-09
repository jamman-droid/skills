# House Rules

How every skill in this repo is written. Each SKILL.md is reviewed against this
before it ships. (Becomes the /hone skill in Phase 2.)

## Tiers and composition

- **user-invoked** — typed by a human; frontmatter carries
  `disable-model-invocation: true`. May fire model-invoked skills by name.
  NEVER fires another user-invoked skill — it names it for the human to type.
- **model-invoked** — the agent reaches for it when the task fits; humans can
  type it too. The description doubles as the trigger.
- **dual-surface** — a model-invoked skill we also teach humans to type
  (`/inspect`, `/press`). Same rules as model-invoked.

## Frontmatter

`name` (the slug), `description`, `argument-hint` when arguments help.
The description is SEMANTIC: what the skill does plus a plain-language
"Use when …". Never a brand word, never a circular trigger.

## Structure

1. One-line purpose.
2. Numbered process steps. Each step ends in a checkable gate ("Gate: …").
3. Outputs: every skill terminates in a named artifact (paths and formats per
   `docs/CONTRACTS.md`) or an explicitly recorded "no".

Prefer ≤120 lines. Reference detail moves to sibling `.md` files loaded only
when needed (progressive disclosure).

## Voice

Imperative, addressed to the agent. Lead every section with the rule, not the
story. No filler, no atmosphere.

## Portability

The core text must run on any capable agent: plain instructions, standard
shell, no harness-specific tools. Harness-specific mechanics (parallel
subagents, background tasks) appear only under an "Enhancement (optional):"
line, with the sequential fallback stated first. External CLIs are invoked via
`npx` with a graceful failure path.

## Shared contracts

Data formats (config, decisions ledger, plan files, corpus map, phases,
completion evidence, handoffs, render requests) are defined once in
`docs/CONTRACTS.md`. Skills reference them. Restating or forking a contract is
a review failure.

## Originality

Everything is written fresh, in this repo's own voice and vocabulary — no
text, structure, or signature coinages from other skill authors. The
`lavish-axi` CLI is named only inside `/stage`'s body, as the tool it drives.

## Review checklist (run before a skill ships)

- **Trigger** — would the model fire this at the right moment from the
  description alone?
- **Exit** — does every step have a gate? Does the skill end in an artifact?
- **Duplication** — does it restate a contract or another skill's mechanics?
  (Delegate by name instead.)
- **Portability** — any hard harness dependency outside an Enhancement line?
- **Length** — over ~120 lines: what moves to a sibling file?
- **Fresh wording** — zero copied phrasing against the reference corpus.
