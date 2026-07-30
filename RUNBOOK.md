# RUNBOOK — automated run entrypoint

This file is what an unattended or semi-attended Claude Code session follows to execute
one persona/prompt evaluation against the live docs site and file the report. It's
runner-agnostic: a Hermes cron job, a `claude -p "..."` invocation, or a CI job can all
point at this file the same way. A human running a fully manual, pasted-into-chat session
should use the personas directly — see `README.md` — not this file.

## Invocation contract

Each invocation names exactly one persona and one prompt letter, and optionally a
published site version, in this form:

> Follow `RUNBOOK.md` in the docs-eval repo. persona=`<NN>` prompt=`<A|B|C>`
> [version=`<VERSION>`]

Example: `persona=07 prompt=B` (defaults to `version=development`), or
`persona=07 prompt=B version=stable` to run the same pair against a different published
version. One invocation = one fixed `(persona, prompt, version)` combination. Don't
invoke this with a free-form task ("audit the docs") — the whole point of the pack is
that persona, prompt, and version are fixed in advance so runs stay comparable. If the
trigger ever passes something other than this exact form, stop and report the mismatch
instead of guessing.

**`version` is optional and defaults to `development`.** See `README.md`'s "Which
version, and why" before choosing a non-default value — most of this pack's
ground-truth findings were only verified against `development`, and other versions
(`stable`, release-candidate tags like `029-rc4`) may be structurally incomplete in ways
that look like new findings but are actually just that version missing whole sections.
That's still worth recording, but flag it as a version gap, not a fresh docs defect,
unless you'd confirm the same page is present-but-broken under `development` too.

## Steps

1. **Read `rubric.md`** — the report format and severity scale every run reports
   against.

2. **Locate the persona folder**: `personas/<NN>-*/` — the numeric prefix from the
   invocation, glob-matched, since the rest of the folder name is a descriptive slug
   (e.g. `personas/07-security-reviewer/`). Exactly one folder should match; if none or
   more than one does, stop and report the mismatch. **Read `persona.md` and
   `prompt<X>.md`** from it in full, where `<X>` is the prompt letter from the
   invocation. Treat the other two prompt files in that folder as invisible, exactly as
   a human pasting into a fresh session would. **Resolve `{VERSION}`**: every literal
   `{VERSION}` placeholder in those two files becomes the `version` from the invocation,
   or `development` if the invocation omitted it.

3. **Execute the task for real**, obeying the scope fence in `persona.md` exactly, with
   `{VERSION}` resolved as above:
   - Fetch only under `https://docs.pantavisor.io/<VERSION>/`.
   - Prefer each page's `.md` export (append `.md` to the page path) over the rendered
     HTML — see `persona.md` rule 1 for why. Fall back to the rendered page if `.md`
     404s. Never fetch `/llms.txt`, `/llms-full.txt`, or similar site-wide summaries —
     see `persona.md` rule 2.
   - Use your web-fetch tool, following links the way the persona actually would —
     don't jump straight to a URL you happen to know from this runbook or from
     `ground-truth.md`.
   - Stay in character. Do not use knowledge the persona doesn't have.
   - If a hardcoded entry-point URL in the persona/prompt files 404s under the resolved
     `<VERSION>` (the live site reorganizes; this has happened before — see
     `ground-truth.md`'s provenance note, and note that non-`development` versions are
     more likely to simply lack a section), don't fail the run. Log it as an S4 `stale`
     finding against *this pack* (or a plain "not present under `<VERSION>`" note if the
     version itself is the reason), then locate the equivalent page by navigating from
     `https://docs.pantavisor.io/<VERSION>/meta-pantavisor/overview` or
     `https://docs.pantavisor.io/<VERSION>/pantavisor/overview` and continue the task
     from there. If `<VERSION>` doesn't have an equivalent page at all, say so and stop
     that step rather than falling back to `development`.

4. **Produce the report** in the exact table format `rubric.md` specifies, plus its
   four-line closing summary. This is the content of the report file — not a summary of
   it, the actual findings table.

5. **Write the report** to:

   ```
   answers/<NN>-<persona-slug>/<YYYY-MM-DD>-prompt<X>-<model-slug>-<VERSION>.md
   ```

   using the same `<NN>-<persona-slug>` folder name found in step 2 (`answers/` mirrors
   `personas/` exactly, one flat file per run — no per-prompt subfolder), and the
   resolved `<VERSION>` from step 2. Use today's date and a filesystem-safe slug of the
   model that ran the session (e.g. `claude-sonnet-5`, `claude-opus-5`). If a file for
   this exact persona/prompt/date/model/version already exists (a second run landed
   same-day), append `-2`, `-3`, etc. — never overwrite a prior run's report. Prepend
   the report body with this header:

   ```markdown
   # Persona <NN> — Prompt <X> — <YYYY-MM-DD>

   Target: https://docs.pantavisor.io/<VERSION>/
   Date: <YYYY-MM-DD>
   Model: <model-slug>
   Version: <VERSION>
   Run by: <however this session was invoked — e.g. "Hermes cron", "claude -p", "manual">
   ```

6. **Append one row** to `answers/index.md` (create it from the template in
   `answers/README.md` if it doesn't exist yet):

   | Date | Persona | Prompt | Version | Model | Outcome | Worst finding | Report |
   |---|---|---|---|---|---|---|---|
   | 2026-07-30 | 07 — Security reviewer | B | development | claude-sonnet-5 | completed with detours | S1: ... | [link](07-security-reviewer/2026-07-30-promptB-claude-sonnet-5-development.md) |

   Pull "Outcome" and "Worst finding" straight from the report's own closing summary —
   don't re-derive them.

7. **Commit** the new report file and the updated `answers/index.md` with git. Use a
   commit message like `docs-eval: persona 07 prompt B run (2026-07-30)` — append the
   version if it isn't `development`, e.g. `... (2026-07-30, stable)`. **Do not push.**
   Whether these commits get pushed/synced anywhere is a decision for whoever configured
   this run, not something to do by default from an unattended run.

## Suggested cadence

Not enforced by this file — configure wherever runs are scheduled. A reasonable starting
point: cycle the 30 core-set combinations (personas 1–10 × prompts A/B/C) roughly
weekly, one per day, skipping weekends, at `version=development`; add the extended set
(11–12) monthly. Re-running the same `(persona, prompt, version)` combination
periodically is what lets `answers/index.md` show drift over time — a finding that stops
reproducing is as informative as a new one. Runs against a non-default `version` are a
separate, deliberate audit (e.g. "does `stable` have the same gaps as `development`?") —
don't fold them into the regular cadence silently, since mixing versions in one series
breaks the comparability the fixed pairs exist to protect.

## What this run is not

It doesn't update `ground-truth.md` automatically. If a run surfaces something that
looks like a new, real, reproducible gap, that's a judgment call for a human — flag it
prominently in the report (it already will be, at S1/S2) and let the maintainer promote
it per the process in `README.md`.
