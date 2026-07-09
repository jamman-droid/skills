---
name: press
description: Close every open decision in a plan through a single-question-per-turn dialogue that ends only when the user signs off on a complete decision ledger. Use when a plan still has open calls, when the user wants a design vetted before build starts, or before executing anything that still contains ambiguity.
argument-hint: "[plan file, topic, or nothing to press the current discussion]"
---

Drive every open decision to a terminal state — decided, deferred, or
out-of-scope — and prove convergence with a signed-off decision list before
any execution begins.

## Standing rules (hold for the whole interview)

- **One question per turn.** Each carries your recommended answer and a
  one-line reason for it. Never batch; a second question waits for the first
  answer.
- **Facts are yours, decisions are theirs.** If the project is readable,
  answer factual questions yourself (read the code, config, docs). Only
  genuine choices reach the user.
- **Never fill in the user's answer.** If they say "you decide" or "whatever
  you think", record your recommendation in the ledger as `agent-default ⚑`
  per the decisions ledger contract in `docs/CONTRACTS.md` — the flag marks
  it for later audit. It still counts as a terminal state.
- **No execution before landing.** Nothing gets built, edited, or scaffolded
  until step 6 passes.
- **Budget.** Nearing `budget_pct`% of the context window (config per
  CONTRACTS): checkpoint the ledger through `/relay` and offer to resume in
  a fresh session.

## Process

1. **Situate.** Identify the subject: a plan file (`<plans_dir>/NN-<slug>.md`),
   the corpus map, or an in-flight discussion. Read the config
   (`docs/agents/planproper.md`) when present; take contract defaults
   silently when not. Note whether you are inside a readable repo — that
   sets docs mode (step 5).
   Gate: subject named, config resolved, docs mode set.

   Enhancement (optional): by default, look facts up lazily as each branch
   reaches them (sequential). If the harness supports parallel subagents,
   fan them out before question 1 to pre-scan the project for facts the
   interview will need (stack, existing conventions, prior decisions).

2. **Map the branches.** Do one breadth pass and show the user a tree of
   every decision branch you can see. Vision branches come first — goals,
   what the system looks like when done, what it must do, long-term scope,
   foundational architecture — then the detail branches under them. Mark
   each branch open.
   Gate: user has seen the tree and added or struck branches.

3. **Walk one branch at a time.** Order branches by dependency (what other
   choices hinge on), and within that, riskiest first. Descend the branch
   question by question. Every question fits this 3–5 line shape:
   - why this matters right now (one line)
   - the question itself
   - your recommendation, with the one-line why
   - what picking each answer closes off

   As answers land, append them to the decisions ledger (format per
   CONTRACTS) with the correct provenance: `user`, `suggested-accepted`,
   or `agent-default ⚑`.
   Gate: the branch has reached exactly one terminal state — decided,
   deferred (reason recorded), or out-of-scope — before the next branch opens.

4. **Show position at every branch boundary.** When a branch closes,
   refresh a compact view: the ledger so far, which branches remain, which
   is next. The user should never have to ask "where are we?".
   Gate: compact view posted; user has not objected to the route.

5. **Docs mode (automatic).** Inside a repo: as a term crystallizes or a
   decision passes its gate, route it through `/coin` so the glossary and
   decision records stay current with the interview. No repo: keep the
   ledger in-conversation only.
   Gate: every gate-passed decision either sent through `/coin` or
   explicitly held ledger-only.

6. **Landing checkpoint.** When no open branches remain, replay every
   decision as a numbered list — including deferrals and out-of-scope calls
   with their reasons — and ask for sign-off on that exact list. Amendments
   reopen the affected branch and return to step 3.
   Gate: explicit user sign-off on the numbered list. This is the only exit.

## Output

- **With a plan file or corpus map:** write the final ledger into the
  document's `## Decisions` section (plan file or `MAP.md`, formats per
  CONTRACTS). That updated file is the artifact.
- **Without a repo:** end with a pasteable decision summary — the numbered
  sign-off list plus each entry's ledger line — as the artifact.
- **If the user aborts before landing:** record the partial ledger the same
  way, marked `Status: interview incomplete`, so nothing evaporates.
