---
name: slice
description: Decompose a plan file into vertical slices — each a thin increment that exercises the full stack it needs — then bin the slices into context-budgeted phases and append them to the plan. Use when a plan is ready to be broken into buildable increments, when the user asks to decompose, split, or phase a plan, or when /planproper routes a decompose request.
argument-hint: <plan file path or NN-slug>
---

# Slice

Turn one plan into an ordered set of vertical slices, bin them into phases that fit the context budget, and record the result in the plan's Phases section.

## What counts as a vertical slice

- A slice is the thinnest cut that still exercises the whole stack a behavior needs — storage, logic, surface, verification — never a single stratum delivered alone. "Build all the models" is a layer, not a slice; "a user can save one record and see it listed" is a slice.
- A finished slice can be shown working, or checked against its criteria, without waiting on any other slice; write those criteria as independently checkable statements.
- The first slice of Phase 1 must thread the **whole system** end to end, as thinly as the plan allows. It proves the route exists; every later slice widens a proven path instead of gambling on an unproven one.

## Process

1. **Load the plan.** Resolve the argument to a plan file (Plan file contract, `docs/CONTRACTS.md`) inside `plans_dir`; read its goal, acceptance criteria, body, and Decisions ledger. Read `budget_pct` from the config (Config contract), applying the default silently if absent. Gate: plan file read and every acceptance criterion listed out for tracing.

2. **Draft slices and their blockers.** Cut the plan body into vertical slices. For each, write: the end-to-end behavior it lands (stated as an outcome, not an implementation list), its acceptance criteria, and **blocked by** — the slices that must land before it can start (or "none"). Keep slice text durable: no file paths or code, since both go stale. Sole exception — a snippet that *is* the decision (a schema, a state shape) may be inlined, trimmed to just the lines that carry the decision. Gate: every slice states a demoable behavior, carries checkable criteria, and declares its blockers explicitly.

3. **Pull out codebase-wide edits.** Some changes are a single uniform edit spread across every call site — renaming a column, retyping a shared symbol — with a blast radius too wide for any one slice to land green. Keep these out of the slice graph and run them as three ordered moves:
   1. Introduce the replacement beside what it replaces, so nothing breaks yet.
   2. Move call sites onto it in batches — one slice each, every batch waiting on the introduce move.
   3. Retire the original once no caller is left, in a closing slice that waits on all the batches.
   When a batch can't stay green on its own, hold them all on one integration branch and let a single verify-slice at the end carry the green promise. Gate: no slice hides a codebase-wide edit; each appears as an introduce → batch-migrate → retire chain.

4. **Validate the graph.** Check three properties: (a) the blocker links form no cycle; (b) at least one slice has no blockers, so work can start immediately; (c) every acceptance criterion from the plan traces to at least one slice. Fix failures by redrawing links, splitting slices, or adding a missing slice — then re-check. Gate: acyclic, at least one open starting slice, full criteria coverage.

5. **Bin slices into phases.** Walk the graph in topological order (blockers before dependents) and pack slices into phases: 3–6 slices per phase, with the phase's cumulative work — building it *and* reviewing it — fitting inside `budget_pct`% of the context window. Start a new phase the moment the next slice would put the running total at risk; a small phase beats a degraded one. Give each phase a name, a **Delivers** line stating the demoable outcome the phase leaves behind, and a **Checklist** of verifiable items — the checklist is the audit contract `/inspect` will later score against. Gate: every phase holds 3–6 slices, fits the budget, and has a Delivers a stakeholder could watch and a Checklist an auditor could tick.

6. **Write the Phases section.** Append the result to the plan file per the Phases contract in `docs/CONTRACTS.md` and set the plan's `Status` to `decomposed`. Gate: the section matches the contract exactly and the status line is updated.

7. **Present phases for approval.** Show the user the *phases* — names, Delivers lines, slice counts, and which phases wait on which — not the raw slice list. Put two things to them: whether the phases are cut at the right size, and whether every declared dependency is a true gate rather than an assumed one. Fold revisions back through steps 4–6 and re-present until they approve.
   Enhancement (optional): render the dependency graph with phase lanes by firing `/stage` (Render requests contract) — sequential fallback is the plain-text presentation above.
   Gate: user has approved the phase breakdown, and the plan file on disk reflects the approved version.

## Output

The plan file, updated in place: a contract-conformant `## Phases` section and `Status: decomposed`. Execution then pulls whichever slices have every blocker cleared, one slice per fresh context.
