# reports/

Output of automated persona/prompt runs against `https://docs.pantavisor.io/development/`,
produced by `../RUNBOOK.md` on a Hermes cron schedule. Manual, human-run sessions don't
need to write here — this directory exists so unattended runs accumulate a history
instead of each overwriting the last.

- **`index.md`** — one-row-per-run log: date, persona, prompt, outcome, worst finding,
  link to the full report.
- **`persona-<NN>-prompt-<X>-<YYYY-MM-DD>.md`** — the full findings table + closing
  summary for one run, in the format `../rubric.md` specifies.

Don't hand-edit reports after the fact — if a finding turns out to be wrong, note that in
a later run rather than rewriting history. If a run surfaces a gap worth keeping
permanently, promote it into `../ground-truth.md` (see that file's maintenance section)
rather than leaving it only here.
