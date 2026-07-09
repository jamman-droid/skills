# planproper config

<!-- Written to <repo>/docs/agents/planproper.md by /planproper on first run.
     Every value is optional; defaults shown. See docs/CONTRACTS.md. -->

budget_pct: 40        # a phase must be built AND reviewed within this % of the context window
plans_dir: work/plans
notes_dir: work/notes
handoff_dir: work/handoffs
audits_dir: work/plans/audits
render_dir: .stage    # keep gitignored
tracker: local        # local | github | linear
glossary: CONTEXT.md
adr_dir: docs/adr
