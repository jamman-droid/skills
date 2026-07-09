---
name: coin
description: Keep the project's memory in two ledgers — a glossary of canonical terms and a log of decision records — and audit code against them. Use when a concept needs one agreed name, when a choice worth remembering has just been settled, or when checking whether the codebase still speaks the glossary's language.
argument-hint: "[term or decision to record | drift-audit]"
---

# Coin

Mint the project's vocabulary and bank its decisions — written down inline, the
instant they settle, never batched to the end of a session.

## The two ledgers

- **Glossary** — path from the `glossary` config key (see docs/CONTRACTS.md).
  Each entry: a bold canonical term, a definition of at most two sentences
  saying what the thing IS, an `Avoid:` line naming every rejected synonym,
  and optionally a single invariant line. Pure language — an entry records
  what a term means and nothing about how the system is built; working notes
  belong elsewhere.
- **Decision log** — files in the `adr_dir` config directory, named
  `NNNN-slug.md` with sequential numbers (scan for the highest, add one).
  A record is usually one tight paragraph: what was decided, why, and what
  was turned down.

Both are lazy: the first real entry creates the file or directory. Never
create either empty.

## Capture (default mode)

1. **Locate the governing ledgers.** Read the config per docs/CONTRACTS.md for
   `glossary` and `adr_dir`. If `GLOSSARIES.md` exists at the repo root, the
   repo has several glossaries — that map states which one governs which area;
   pick the one covering the code under discussion, and ask if it is unclear.
   Gate: you can name the exact glossary path this session writes to.
2. **Catch the settling point.** A term is ready when a naming dispute
   gets settled, a vague word gets a precise replacement, or two words turn
   out to mean one thing. Write the entry right then, in the same turn.
   Gate: the entry is on disk before the conversation moves past it.
3. **Take a side.** One canonical term per concept; every losing synonym goes
   on the `Avoid:` line so it can never quietly return. Admit only concepts
   specific to this project — general programming vocabulary stays out.
   Gate: the entry has a canonical term, a ≤2-sentence definition, and its
   rejected synonyms listed.
4. **Run the decision-record gate** (this skill is its single home). Write a
   file in `adr_dir` only when ALL three hold:
   - **Costly to unwind** — reversing it later means real rework.
   - **Opaque in hindsight** — someone reading the code cold could not
     reconstruct the reasoning behind it.
   - **Contested** — more than one live option existed and the pick had
     stated reasons.
   Any one missing → no record. Capture it as a glossary entry or as a
   Decisions-ledger line (format in docs/CONTRACTS.md) instead, and say which
   you chose. Gate: the three-part test is answered out loud before any
   record file is created.
5. **Show every write.** Quote each new or changed entry verbatim in chat at
   the moment it is written, so the human sees exactly what the ledger now
   says. Gate: no ledger write is invisible.
6. **Close the books.** End the session with a one-line delta, e.g.
   "+2 terms, 1 sharpened; decision record 0009." Say "no ledger changes"
   when that is the truth. Gate: the summary line was emitted.

## Drift audit (on request — typically fired by /planproper audit)

1. **Sweep the code.** Grep the codebase for every canonical term and every
   `Avoid:` term in the governing glossary — identifiers, comments, docs,
   config keys. Gate: every glossary entry was checked against the code.
2. **Report contradictions.** Flag `Avoid:` terms that appear in code, and
   canonical terms the code uses with a different meaning. For each finding
   give file locations and offer the concrete rename; never rename without
   approval. Gate: every finding carries locations and a proposed fix, or
   the audit states the code and glossary agree.
3. **Deliver.** When fired by another skill, hand the findings back to the
   caller (it owns the enclosing audit artifact). When run standalone, append
   them to `<audits_dir>/YYYY-MM-DD-glossary-drift.md`. Gate: the findings
   landed with whoever asked, in writing.

## Outputs

- Updated glossary and/or new `NNNN-slug.md` decision records, at the
  config-declared paths — quoted in chat as written.
- Drift audit: findings returned to the calling skill, or the standalone
  drift file above.
- An explicitly recorded "no ledger changes" when nothing settled.

Callers: /press (docs mode) and /planproper fire this skill by name.
