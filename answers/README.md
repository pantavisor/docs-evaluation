# answers/

Output of persona/prompt runs against `https://docs.pantavisor.io/<VERSION>/`
(`development` unless a run was pointed at another version — see the root `README.md`'s
"Which version, and why"), produced by `../RUNBOOK.md`. Runs can be triggered any way a
Claude Code session can be started unattended or semi-attended — a Hermes cron job,
`claude -p "..."`, or a manual session that chooses to file its report here instead of
just pasting output in chat.

## Layout

```
answers/
  <NN>-<persona-slug>/
    <YYYY-MM-DD>-promptA-<model-slug>-<VERSION>.md
    <YYYY-MM-DD>-promptB-<model-slug>-<VERSION>.md
    <YYYY-MM-DD>-promptC-<model-slug>-<VERSION>.md
    ...
  index.md
```

- **`<NN>-<persona-slug>/`** — one folder per persona, mirroring
  `../personas/<NN>-<persona-slug>/` exactly. Every run for that persona — any prompt,
  any date, any model, any version — lands as a flat file directly in this folder; there
  is no per-prompt subfolder.
- **`<YYYY-MM-DD>-prompt<X>-<model-slug>-<VERSION>.md`** — the full findings table +
  closing summary for one run, in the format `../rubric.md` specifies, with a header
  carrying the run date, model, and version (see `../RUNBOOK.md` step 5 for the exact
  header). `<VERSION>` is `development` for a default run, or whatever version the run
  was pointed at otherwise — always present in the filename so runs never collide, no
  matter how many prompts/models/versions accumulate in one persona's folder. If a file
  for the exact same persona/prompt/date/model/version already exists, append `-2`,
  `-3`, etc. — never overwrite a prior run's file.
- **`index.md`** — one-row-per-run log: date, persona, prompt, version, model, outcome,
  worst finding, link to the full report.

Don't hand-edit answer files after the fact — if a finding turns out to be wrong, note
that in a later run rather than rewriting history. If a run surfaces a gap worth keeping
permanently, promote it into `../ground-truth.md` (see that file's maintenance section)
rather than leaving it only here.
