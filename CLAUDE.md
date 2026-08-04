# CLAUDE.md — running evaluations and applying fixes

This file covers the operational side of two things: running a persona evaluation
against the live docs, and turning one finding from an `answers/` report into a
draft PR. The processes themselves are `RUNBOOK.md` and `FIXBOOK.md` — read the
relevant one first; this file adds the concrete steps and conventions this repo has
settled on in practice.

## Running a persona evaluation

Follow `RUNBOOK.md` end to end — it's the authoritative process (locate the persona
folder, read `persona.md` plus the one named `promptX.md`, resolve `{VERSION}`,
fetch only under `https://docs.pantavisor.io/<VERSION>/`, produce the report, write
it to `answers/`, append a row to `answers/index.md`, commit). This section only
adds what to do when the request arrives inline in a chat session rather than via a
fresh `claude -p` invocation.

- **Invocation shorthand:** `persona=<NN> prompt=<A|B|C> [version=<VERSION>]` —
  e.g. "persona=07 prompt=B" or "persona=07 prompt=B version=stable". Don't accept
  a free-form request ("audit the docs" / "check a few personas") — `RUNBOOK.md`
  requires this exact `(persona, prompt, version)` form; ask for whichever piece is
  missing rather than guessing. Pick values from `README.md`'s persona/prompt
  tables if the user needs help choosing.
- **Fresh-session rule matters even inline.** `RUNBOOK.md`'s comparability across
  runs depends on the persona not bleeding context from other personas/prompts or
  prior conversation. If this session already has unrelated context loaded (a
  previous persona run, a fix applied to another repo, general discussion of the
  pack itself), say so and recommend a fresh session before proceeding — don't
  silently run it anyway.
- **Stay in character and in-fence.** Fetch only under
  `https://docs.pantavisor.io/<VERSION>/`, prefer each page's `.md` export, never
  fetch `/llms.txt` or similar site-wide summaries — see `RUNBOOK.md` step 3 and
  `persona.md`'s own rules for the full list.
- **Don't push.** Commit the new report file and the updated `answers/index.md` in
  this repo, same as any `RUNBOOK.md` run, but never push automatically — whether
  and where these commits get pushed is a separate, explicit decision, not a
  default.

## Applying a fix from an `answers/` report

### Invocation

A request to "apply the fix(es) from `answers/<NN>-<slug>/<report>.md`" means:
follow `FIXBOOK.md`'s steps 1–5 (parse the finding, route to the owning repo,
locate and re-verify the source file) for each finding in scope, then follow
the git/PR workflow below instead of `FIXBOOK.md`'s generic step 6.

**Scope check first.** `FIXBOOK.md` defaults to one finding per invocation. If
the user's request doesn't say how many findings to fix, ask — don't silently
pick one or silently do all. If they say "the fix" (singular) on a
single-finding report, or explicitly say "all", proceed without asking.

### Repo checkout location

Clone target repos (`meta-pantavisor`, `pantavisor`, `pvr`) into
`../docs-fix-repos/<repo-name>/` (i.e. a sibling of this `docs-framework`
checkout), not directly beside other project checkouts. Create the folder if
it doesn't exist yet. Reuse an existing checkout there if one already exists
instead of re-cloning.

### Making the edit

- One commit per report (even when fixing multiple findings from the same
  report in one pass) — the commit message lists each finding fixed.
- Match `FIXBOOK.md` step 5: minimal edit per finding, matching the
  surrounding file's style. Don't fold in unrelated cleanup you notice along
  the way.
- If a finding turns out already fixed (re-verify per `FIXBOOK.md` step 4),
  don't touch that file — note it as already-resolved in the commit message
  and PR description instead of making a no-op edit.
- Prefer linking to existing content (a glossary entry, an existing benchmark
  page) over writing new prose from scratch when the docs already have the
  answer somewhere unlinked — that's usually the actual gap, not a missing
  page.
- Never fabricate numbers, benchmarks, or claims not already present
  elsewhere in that repo's docs.

### Git workflow

1. Branch from the target repo's default branch:
   `docs-eval/<short-descriptive-slug>` (e.g.
   `docs-eval/persona01-promptA-overview-gaps`).
2. Commit with a message that: summarizes the fix, explains which persona/
   prompt/date report it closes gaps from, and lists each finding addressed
   (or already-resolved) by page/section.
3. Push the branch to `origin` and open a **draft** PR against the default
   branch with `gh pr create --draft`. PR body sections:
   - **Origin** — link (or path) to the source `answers/` report file, plus
     persona/prompt/date.
   - **What blocked the reader** — one bullet per finding: severity, a short
     quote of "What blocked me" and the Evidence, straight from the report.
   - **What changed and why** — per file, what was edited and which
     finding(s) it closes; note any finding found already-resolved.
   - **Status** — explicit note that this is a draft opened from an
     automated docs-eval fix pass, not verified against the rendered site,
     for human review before merging.
4. **Never merge.** Report the PR URL and stop — same rule as `FIXBOOK.md`.
5. Never push directly to the target repo's default branch, and never open a
   PR against `docs.pantavisor`'s generated `reference/` content (see
   `FIXBOOK.md`'s "Where the docs actually live").

### Reference

See `FIXBOOK.md` for: the URL-segment → repo routing table, how to locate the
real source file from a finding's URL, and what counts as out-of-scope
(off-domain findings, `outside-docs` type findings needing human triage).
