---
name: dig
description: Answer one narrow technical question by tracing it to primary sources — source code, specs, official docs for a pinned version — and save the findings as a dated note that plans cite by path. Use when a plan or decision hinges on a factual unknown (API behavior, version support, config semantics, limits, compatibility) that must be verified rather than assumed.
argument-hint: <question, plus the library/tool versions in play if known>
---

Establish a small set of technical facts and leave behind a note other skills can cite. Depth over breadth: one question, fully traced.

## Trust ladder

When sources disagree, the higher rung wins. Cite the highest rung you actually reached, not the highest that exists.

1. Source code of the pinned version
2. Formal spec or RFC
3. Official docs matching the pinned version
4. Changelogs, release notes, maintainer replies in issue threads
5. Third-party posts — usable, but every claim from this rung is flagged `[third-party]`

Anything you believe from training memory alone is rung zero: it may appear in the note only labeled `[memory — unverified]`, never presented as checked.

## Process

1. **Frame.** Restate the question in one line, then break it into concrete sub-questions. Pin the exact version of every library, tool, or API involved — from lockfiles or manifests in the repo if present, otherwise from the caller; if neither yields a version, record "version unpinned" and treat matching docs as rung 4, not rung 3. Gate: a written list of sub-questions, each answerable yes/no or with a specific value, plus a version pin (or explicit unpinned marker) per subject.
2. **Locate the note home.** Read `notes_dir` from the config per `docs/CONTRACTS.md` (defaults apply silently when the config file is absent). Gate: target path `<notes_dir>/YYYY-MM-DD-<topic>.md` is determined, with `<topic>` a short kebab-case slug of the question.
3. **Trace.** Work each sub-question down the ladder from the top rung you can reach. Read the actual source, spec section, or versioned doc page — not a summary of it. Record the link and the ladder rung the moment you confirm a claim; do not reconstruct citations afterward. An authoritative silence — the docs genuinely do not say — is itself a finding: record it with a link to where the answer should have been. Gate: every sub-question is either (a) answered with at least one source link and a rung, or (b) moved to Unresolved with a note on where you looked.
4. **Assign confidence.** Mark each claim: `[confirmed]` (rungs 1–3, unambiguous), `[likely]` (rung 4–5 or indirect inference from a high rung), `[memory — unverified]`, or `[third-party]` combined with one of the former. Gate: no claim in the draft lacks both a marker and a source link (memory claims carry the marker in place of a link).
5. **Stop.** When every sub-question from step 1 is sourced or listed under Unresolved, the dig is over. Do not widen scope, add background reading, or pad thin findings into prose. Gate: sub-question list from step 1 reconciles one-to-one against Findings + Unresolved.
6. **Write the note.** Save to the path from step 2 using the format below. Gate: file exists on disk and its Unresolved section is present — even if its only content is "none".
7. **Report.** Reply with the note's absolute path and a one-line verdict per sub-question. Gate: caller has the citable path.

## Note format

```markdown
# <Question, verbatim from step 1>
Date: YYYY-MM-DD
Pinned: <name>@<version> per <lockfile/manifest/caller>, one line each

## Findings
- <claim> [confirmed] — <source link>
- <claim> [likely] [third-party] — <source link>
- <claim> [memory — unverified]
- The docs for <subject>@<version> do not address <sub-question> [confirmed] — <link to the section searched>

## Unresolved
- <sub-question> — looked in <where>; why it stayed open
```

Enhancement (optional): if the caller wants to keep working while the dig runs, delegate steps 1–6 to a background agent with this skill's text and the framed question, then collect the note path when it finishes. The inline sequential run above is the default on any agent.
