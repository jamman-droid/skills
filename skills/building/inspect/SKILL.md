---
name: inspect
description: Check a plan before building, or audit the work after. Auto-detects which mode applies — no implementation yet means a plan-quality review (coverage, verifiability, consistency, assumptions); an existing diff or executing phase means a completion audit (standards, spec fidelity, checklist evidence). Each axis reports separately with its own verdict, and completion evidence lands in the append-only audit log. Use when a plan needs vetting before work starts, when in-progress or finished work should be checked against its plan, or when asked to inspect, audit, or review a plan or a change.
argument-hint: "[plan-file | git-ref] [plan | work]"
---

Review front door: judge a plan's quality, or judge a diff against its plan — never both in one pass.

## Core model: axes

An **axis** is a triple: a source document, a review brief, and a capped report (≤400 words).
Axes never merge, never rerank each other, and are always presented side by side — a change can
clear one axis and fail another, and keeping them apart stops a pass on one from hiding a fail
on the other. Every finding carries a severity: **blocker** / **should-fix** / **nit**. Every
axis closes with a verdict: **pass** / **pass-with-nits** / **fail** (any blocker ⇒ fail).

## Process

1. **Load context.** Read the config (`docs/agents/planproper.md`, defaults per CONTRACTS).
   Resolve the plan under review: an explicit path argument wins; otherwise pick the plan in
   `<plans_dir>/MAP.md` whose status or recent activity matches the conversation. Read the plan
   file and the map. Gate: exactly one plan file is open, or the run stops with "no plan found —
   run /planproper first".

2. **Detect the mode.** An explicit second argument (`plan` or `work`) overrides detection.
   Otherwise: plan status `raw`/`improved`/`decomposed`, or no reviewable diff anywhere →
   **plan-quality review**. A phase marked `executing`/`done`, or a diff since a fixed point →
   **completion audit**. Gate: one mode is named aloud before any axis runs.

3. **(Completion audit only) Pin the comparison point.** Take the ref the caller gave, or the
   plan's recorded start point, or ask. Resolve it (`git rev-parse <ref>`) and capture the diff
   once: `git diff <ref>...HEAD` plus `git log <ref>..HEAD --oneline`. If the working tree is
   dirty, say so and treat staged + unstaged changes as part of the diff under review. A ref
   that won't resolve, or an empty diff with a clean tree, fails the run here — before any axis
   spends effort. Gate: the diff is captured, non-empty, and its boundaries stated.

4. **Assemble the axes for the detected mode.**

   **Plan-quality axes** (source: the plan file, the map, sibling plans):
   - **Coverage** — walk the Goal and Vision line by line: does every stated goal trace to at
     least one acceptance criterion? List orphaned goals and criteria that answer to no goal.
   - **Verifiability** — for each acceptance criterion: could a stranger check it alone, with a
     command, a file, or an observable behavior? Flag anything that needs taste to judge.
   - **Consistency** — does the plan contradict the map's Vision, Up next edges, Out-of-scope
     entries, or any sibling plan's decisions ledger? Cite both sides of each conflict.
   - **Assumptions** — surface dependencies the plan uses but never states, and options the
     plan silently forecloses. Check `agent-default ⚑` ledger entries first.

   **Implementation axes** (source: the pinned diff, plan file, repo docs):
   - **Standards** — the repo's own written conventions (CONTRIBUTING, style docs, ADRs),
     plus this design-quality checklist. Repo docs override the checklist; every checklist hit
     is a judgment call, never a hard violation; skip anything lint or style tooling already
     polices.
     1. A name that hides what the thing does or holds.
     2. The same logic shape appearing twice inside the change.
     3. A handful of values that only ever travel as a group, never bundled into a type.
     4. A bare string or number carrying a domain meaning it can't enforce.
     5. One logical change fanning edits across many files.
     6. One file edited for several unrelated reasons.
     7. An abstraction built for a need no plan item has.
     8. A caller walking deep through another object's internals to get what it wants.
   - **Spec** — does the diff faithfully build what the plan says? Report plan items that are
     missing or partial, behavior nothing asked for, and items that look built but look wrong.
     Quote the plan line beside every finding.
   - **Completion** — walk the phase Checklist item by item and grade each against the
     completion-evidence contract in CONTRACTS, attaching the one line of evidence it asks for
     (a file path, a test name, or an observed behavior).

   Gate: each axis has its source document in hand (or is explicitly skipped with a reason,
   e.g. no phase checklist exists yet for the Completion axis).

5. **Run the axes in fresh contexts, sequentially.** Evaluate one axis at a time, each from a
   clean slate — its source, its brief, nothing carried over from the previous axis — and cap
   each report at 400 words.
   Enhancement (optional): dispatch the axes as parallel isolated sub-agents, one per axis,
   and collect the reports; the sequential path above is the required baseline.
   Gate: every non-skipped axis has produced a report with severities and a verdict.

6. **Report side by side.** One section per axis, verbatim or lightly trimmed — no merging, no
   cross-axis ranking, no single overall winner. Close with one line per axis: verdict + worst
   remaining severity. Gate: the reader can see each axis's verdict without inferring it.

7. **Land the artifact.**
   - *Plan-quality review*: write the findings to
     `<audits_dir>/YYYY-MM-DD-<plan>-review.md`. If a render would help the human annotate
     decisions, hand the findings to `/stage` per the render-request contract; if `/stage`
     declines or is unavailable, the Markdown file stands on its own.
   - *Completion audit*: append the Completion axis's evidence to
     `<audits_dir>/YYYY-MM-DD-<plan>-audit.md` per CONTRACTS — append-only, never rewrite
     earlier entries. Then handle gaps:
     - **Safe gaps only** may be auto-fixed: re-running a check that flaked, or finishing an
       item that is trivially and unambiguously incomplete. Re-verify after the fix and append
       the reversal (item, fix, fresh evidence) to the same log.
     - Anything correctness-critical, ambiguous, or design-shaped is **reported, never
       silently patched**.
     Finally update the plan's status in `MAP.md` (the single source of plan status): zero
     blockers and every checklist item `done`/`done-differently` → `audited`; otherwise leave
     the status and record what blocks it.
   Gate: the artifact file exists at its contract path and the map reflects reality.

8. **Re-review loop.** After the caller applies fixes, re-run **only the axes that failed** —
   passing axes keep their verdicts. Repeat until no axis holds a blocker; that is the done
   condition. Should-fixes and nits may survive into "done" if the caller accepts them, noted
   in the artifact. Gate: zero blockers across all axes, or an explicit caller decision to stop
   is recorded in the artifact.

## Outputs

- Plan-quality review → `<audits_dir>/YYYY-MM-DD-<plan>-review.md` (+ optional `/stage` render).
- Completion audit → `<audits_dir>/YYYY-MM-DD-<plan>-audit.md` (append-only) + updated `MAP.md`.
