---
name: stage
description: Render a dense artifact — a heavy table, a dependency graph, a findings set, a dense comparison matrix, a phased plan, or a large diff — as an interactive HTML page the human annotates in their browser, then fold the annotations back into the work. Use when the structure has real visual payoff and a person with a local browser is present to review; never in headless, CI, or unattended runs.
argument-hint: <what to render and which decisions need the human's marks>
---

# Stage

Turn one dense artifact into a browser page the human can mark up, collect
the annotations, and apply every one. Drives the `lavish-axi` CLI via
`npx -y lavish-axi`; carry only the judgment the tool cannot know. When unsure
of its current flags, run `npx -y lavish-axi --help`.

## Render gate

Render only when BOTH hold. Otherwise answer in Markdown in chat and stop —
that reply is the recorded "no".

- **Structural payoff.** Three or more options to weigh, a multi-phase plan, a
  dependency graph, a dense table, or a diff. Short prose, mid-debug status,
  and anything a paragraph carries fine all fail here.
- **An interactive human at a local browser.** Headless, CI, and unattended
  runs always fail — audits fired unattended deliver Markdown findings instead.

## Process

1. **Accept the request.** Take the Render request per `docs/CONTRACTS.md`.
   Resolve `render_dir` from the config. Gate: every contract field present,
   including at least one decision point.

2. **Pick the design system.** Style the page to match what it depicts. An
   explicitly requested look wins outright; failing that, borrow the subject
   project's own theme, tokens, or component styling — go read that repo,
   wherever it lives, since the artifact's subject is rarely the directory you
   happen to be running in — and only then lean on the CLI's default styling
   guidance. Gate: one source chosen, and you can say why the stronger options
   yielded nothing.

3. **Read the playbooks.** For every content kind named in the request, open
   the CLI's matching content playbook before writing any markup. Gate: no
   kind in the request lacks a consulted playbook.

4. **Write the page** to `<render_dir>/<slug>.html`, structured so the
   decision points from the request are what the eye lands on first. Gate:
   file exists inside the render dir.

5. **Open it** with `npx -y lavish-axi <file>`. Port trouble: retry once, then
   drop to the error ladder. Gate: session live in the browser, or ladder
   engaged.

6. **Poll, apply, repeat.** Long-poll for feedback, leading with a brief
   agent reply that points the reviewer at the highest-stakes decision on the
   page. For each annotation that arrives: acknowledge in one line, apply it
   to the underlying work, re-poll. A dropped or timed-out wait costs
   nothing — the CLI holds pending annotations until the next request, so
   issue it again. Gate: zero annotations left unapplied.
   Enhancement (optional): where the harness time-caps foreground commands,
   move the wait into a background task.

7. **Close.** Finished when all feedback is addressed and one further quiet
   re-poll returns nothing new. Gate: the quiet re-poll was observed.

## Etiquette

- Treat a browser-side close as final. Finish whatever changes remain and
  report them in the conversation; relaunching the page takes a fresh request
  from the human.
- The CLI's share/publish command posts to third-party hosting, public by
  default. Never run it without the human's explicit yes; after publishing,
  store the returned update key inside `<render_dir>/`.

## Error ladder

1. `npx` fails or no browser appears → still write the HTML, export a
   standalone single-file copy, and hand the human its path.
2. Port conflicts → one retry, then step 1's export route.

## Output

Named artifact: `<render_dir>/<slug>.html` — plus its standalone export when
the ladder fired — with every applied annotation reflected in the underlying
work product. A failed render gate terminates in the Markdown answer in chat.
