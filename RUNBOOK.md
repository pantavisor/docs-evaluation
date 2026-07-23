# RUNBOOK — automated run entrypoint

This file is what a headless Claude Code session (invoked by Hermes on a cron schedule)
follows to execute one persona/prompt evaluation against the live docs site and file the
report. A human running a manual session should use the personas directly — see
`README.md` — not this file.

## Invocation contract

Each Hermes cronjob is configured with a fixed trigger prompt naming exactly one
persona and one prompt letter, in this form:

> Follow `RUNBOOK.md` in the docs-eval repo. persona=`<NN>` prompt=`<A|B|C>`

Example: `persona=07 prompt=B`. One cronjob = one fixed `(persona, prompt)` pair. Don't
configure a cronjob with a free-form task ("audit the docs") — the whole point of the
pack is that persona and prompt are fixed in advance so runs stay comparable. If Hermes
ever passes something other than this exact form, stop and report the mismatch instead
of guessing.

## Steps

1. **Read `rubric.md`** — the report format and severity scale every run reports
   against.

2. **Read `personas/<NN>-*.md`** in full. Only the numbered prompt matching the `prompt=`
   parameter is active for this run — treat the other two prompts in the same file as
   invisible, exactly as a human pasting into a fresh session would. Each cron
   invocation is itself a cold session, so this holds automatically; it's stated here
   because the source file contains all three.

3. **Execute the task for real**, obeying the scope fence in the persona file exactly:
   - Fetch only under `https://docs.pantavisor.io/development/`.
   - Use your web-fetch tool, following links the way the persona actually would —
     don't jump straight to a URL you happen to know from this runbook or from
     `ground-truth.md`.
   - Stay in character. Do not use knowledge the persona doesn't have.
   - If a hardcoded entry-point URL in the persona file 404s (the live site
     reorganizes; this has happened before — see `ground-truth.md`'s provenance note),
     don't fail the run. Log it as an S4 `stale` finding against *this pack*, then
     locate the equivalent page by navigating from
     `https://docs.pantavisor.io/development/meta-pantavisor/overview` or
     `https://docs.pantavisor.io/development/pantavisor/overview` and continue the task
     from there.

4. **Produce the report** in the exact table format `rubric.md` specifies, plus its
   four-line closing summary. This is the content of the report file — not a summary of
   it, the actual findings table.

5. **Write the report** to:

   ```
   reports/persona-<NN>-prompt-<X>-<YYYY-MM-DD>.md
   ```

   Use today's date. If a file for this exact persona/prompt/date already exists (a
   second run landed same-day), append `-2`, `-3`, etc. — never overwrite a prior run's
   report. Prepend the report body with this header:

   ```markdown
   # Persona <NN> — Prompt <X> — <YYYY-MM-DD>

   Target: https://docs.pantavisor.io/development/
   Run by: Hermes cron
   ```

6. **Append one row** to `reports/index.md` (create it from the template below if it
   doesn't exist yet):

   | Date | Persona | Prompt | Outcome | Worst finding | Report |
   |---|---|---|---|---|---|
   | 2026-07-23 | 07 — Security reviewer | B | completed with detours | S1: ... | [link](persona-07-prompt-B-2026-07-23.md) |

   Pull "Outcome" and "Worst finding" straight from the report's own closing summary —
   don't re-derive them.

7. **Commit** the new report file and the updated `reports/index.md` with git. Use a
   commit message like `docs-eval: persona 07 prompt B run (2026-07-23)`. **Do not
   push.** Whether these commits get pushed/synced anywhere is a decision for whoever
   configured this Hermes job, not something to do by default from an unattended run.

## Suggested cron cadence

Not enforced by this file — configure in Hermes. A reasonable starting point: cycle the
30 core-set combinations (personas 1–10 × prompts A/B/C) roughly weekly, one per day,
skipping weekends; add the extended set (11–12) monthly. Re-running the same
`(persona, prompt)` pair periodically is what lets `reports/index.md` show drift over
time — a finding that stops reproducing is as informative as a new one.

## What this run is not

It doesn't update `ground-truth.md` automatically. If a run surfaces something that
looks like a new, real, reproducible gap, that's a judgment call for a human — flag it
prominently in the report (it already will be, at S1/S2) and let the maintainer promote
it per the process in `README.md`.
