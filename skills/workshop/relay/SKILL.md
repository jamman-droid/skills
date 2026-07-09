---
name: relay
description: Compress the live session into a handoff doc a cold agent can resume from without asking questions. Use when a session is ending, when work must move to a fresh agent, or when a budget-aware skill nears budget_pct% of the context window and must checkpoint instead of degrading.
argument-hint: "[what the next session should tackle]"
---

Freeze this session into a document a stranger can execute from.

1. **Resolve the destination.** Read `handoff_dir` from the config
   (`docs/agents/planproper.md`; defaults per `docs/CONTRACTS.md`). The doc
   goes there — durable, in-repo. A temp directory is never acceptable: the
   doc must outlive the machine state.
   Gate: absolute target path resolved inside `handoff_dir`.

2. **Sort what you know.** Walk the session and split every claim into two
   piles: VERIFIED (you ran it, read it, or watched it happen — cite the
   evidence) and ASSUMED (believed but unproven). The next agent inherits the
   first pile as fact and the second as open questions; a belief filed as
   fact poisons the whole next session.
   Gate: every state claim carries exactly one label, and each VERIFIED
   claim names its evidence.

3. **Draft the doc.** Use the Handoff doc contract in `docs/CONTRACTS.md` —
   its sections, its filename scheme, nothing invented. Fill them so that:
   - Objective is one sentence a newcomer parses without context.
   - Step 1 of Next steps runs cold: exact command or exact file edit,
     zero clarifying questions needed.
   - Specs, decision records, plan files, and commits appear as paths or
     hashes only — never summarized back into the doc.
   - If the human passed an argument, aim Next steps at that focus.
   - Secrets, tokens, and personal data are redacted before writing.
   Gate: doc contains all contract sections and restates no external
   artifact's content.

4. **Self-test cold.** Save, then re-read the doc as if you had amnesia:
   could you execute step 1 of Next steps using only what is on the page
   plus the referenced paths? If anything would force a question back to the
   human, fix the doc and re-test. Do not skip this — it is the whole point.
   Gate: cold read yields zero open questions on step 1.

5. **Hand over the baton.** End the reply that saved the doc with the
   Continuation line the Handoff doc contract specifies.
   Gate: the Continuation line is the final line of the reply.

**Consuming side:** an agent resuming from a handoff reads the doc before
touching anything else, and treats ASSUMED items as unverified until proven.

**Artifact:** the handoff doc at `<handoff_dir>/YYYY-MM-DD-<slug>.md` per the
Handoff doc contract.
