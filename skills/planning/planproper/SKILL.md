---
name: planproper
description: The planning front door. Turns whatever the user hands over into proper plan files and a corpus map — a pile of exported plan docs gets absorbed and indexed (intake), an idea becomes a plan (plan), a plan gets cut into phases (decompose), and finished work is routed to /inspect (audit). Use when the user wants anything planned, organized into plans, broken into phases, or checked against a plan.
argument-hint: "[intake|plan|decompose|audit] [paths, plan file, or an idea in prose]"
disable-model-invocation: true
---

One command over the planning disciplines. This file routes; the disciplines
(`/press`, `/coin`, `/dig`, `/slice`, `/inspect`, `/stage`, `/relay`) own their
own mechanics — never restate them, fire them by name.

## Pick the mode

An explicit first argument (`intake` | `plan` | `decompose` | `audit`) wins.
Otherwise detect from what arrived:

| The user hands you | Mode |
|---|---|
| Multiple existing plan documents (paths or pasted) | INTAKE |
| An idea, feature request, or prose description | PLAN |
| One existing plan file + a request to break it up | DECOMPOSE |
| A request to check, verify, or review work | AUDIT |

If the input fits none of these, ask — one question, with your best guess
stated. Gate: mode chosen and announced before any other work.

## 0 — Bootstrap

1. Read the config per the Config contract (`docs/CONTRACTS.md`). If present,
   proceed. If missing, ask the user 3–4 setup questions — context budget %
   (`budget_pct`), where plans live (`plans_dir`), which tracker (`tracker`),
   and anything a non-default answer implies — then write the config from the
   sibling `CONFIG-TEMPLATE.md`. Gate: config file exists and parses.

## INTAKE — absorb a plan corpus

The flagship path. The user typically arrives with 10–30 plans exported from a
long chat: overlapping, uneven, unordered. Intake turns the pile into a mapped
corpus.

1. **Collect.** Gather every source: file paths, pasted bodies, directories.
   Save pasted content to `<plans_dir>/` so everything has a path. Gate: source
   list confirmed with the user — nothing they meant to include is missing.
2. **Vision interview.** Run `/press` on vision branches only: goals, what the
   finished system looks like, what it must do, long-term scope, foundational
   architecture. Do not press on per-plan detail here. Gate: a Decisions ledger
   exists that can seed the map's Vision section.
3. **Critique the corpus.** Per plan: gaps, vague steps, acceptance criteria
   that are missing or unverifiable. Across plans: contradictions, overlaps
   that should merge, dependency candidates. Work sequentially through the
   pile. Enhancement (optional): fan the per-plan pass out to parallel critic
   subagents, then run the cross-plan pass yourself over their findings.
   Gate: every source has a critique entry; cross-plan findings listed.
4. **Human decisions.** Resolve everything you can yourself. Bring the user
   ONLY what needs a human — contradictions to arbitrate, merges to approve,
   scope calls — batched in one round, each with your recommendation. When the
   Render requests contract's inputs are ready, render the proposed map via
   `/stage` so they decide against the picture, not a wall of text. Gate: every
   batched decision resolved and recorded in the ledger.
5. **Write the corpus.** Amend each plan per the decisions, write it to
   `<plans_dir>/NN-<slug>.md` per the Plan file contract, and write `MAP.md`
   per the Corpus map contract. Treat the map as an index only: one-liners
   that point at plan files, never a second copy of their bodies. Anything ruled out goes
   under Out of scope with the reason — a ruled-out idea that keeps its reason
   stays ruled out. Gate: `MAP.md` exists, every plan is indexed with a
   status, and Up next names an explicit "start here".

## PLAN — idea to plan file

1. **Converge.** Run `/press` on the idea until no branch is open. When a
   branch turns on an external fact, route it through `/dig` rather than
   guessing. Gate: ledger complete, no open branches.
2. **Write.** Produce `<plans_dir>/NN-<slug>.md` per the Plan file contract,
   ledger embedded. Gate: file exists with verifiable acceptance criteria.
3. **Render and annotate.** Render the plan via `/stage` and run the annotate
   loop. An annotation that reopens a decision goes back to `/press` for THAT
   branch only — never a full re-interview — then re-render. Gate: user signed
   off on the rendered plan.
4. **Index.** Set status `improved`; add or update the plan's row in `MAP.md`.
   Gate: map row and file status agree.

## DECOMPOSE — plan to phases

1. **Slice.** Fire `/slice` on the plan file; phases sized to `budget_pct`%.
   Gate: Phases section appended per the Phases contract, checklist per phase.
2. **Index.** Update the plan's status to `decomposed` in file and map. Gate:
   map row and file status agree.
3. **Hand off.** Tell the user exactly what to run next: their executor on
   Phase 1, then `/inspect` when it lands. Gate: next commands stated
   verbatim.

## AUDIT — route to /inspect

1. **Route.** Fire `/inspect` (Completion axis) on the plan in question. Gate:
   audit log written to `<audits_dir>` per the Completion evidence contract.
2. **Index.** Update the plan's status in `MAP.md` from the audit outcome.
   Gate: map reflects the audit.

## Always

- **End loudly.** Every mode's last message names the artifact produced (path)
  and the exact next command to type.
- **Budget.** Nearing `budget_pct`% of the context window mid-mode: checkpoint
  through `/relay` per the Handoff contract instead of pushing on degraded.
- **Missing discipline.** If a skill this file fires is not installed, say
  which one and how to install it, then degrade to what you can still do
  honestly. Never simulate a discipline's output.
