# Persona 10 — Prompt C — Jargon audit

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt C — Jargon audit

> Read `https://docs.pantavisor.io/{VERSION}/meta-pantavisor/getting-started/develop/cli-tools/pvr-cli`
> and every other page under
> `https://docs.pantavisor.io/{VERSION}/meta-pantavisor/getting-started/develop/cli-tools/`
> (`pvcontrol`, `configuration`, `workflows`).
>
> Two tables. The second is the important one.
>
> **Table 1 — undefined terms**, judged strictly from your background:
>
> | Term | First use (page URL) | Ever defined anywhere I could reach? | What I assumed it meant |
>
> **Table 2 — the false-friend audit.** For **every command whose name pvr borrowed from
> git** — `init`, `add`, `status`, `diff`, `commit`, `clone`, `get`, `merge`, `reset`,
> `checkout` — one row:
>
> | Command | What git's version does | What I therefore assumed | What the docs say it does | Do they differ? | Would the difference have bitten me? |
>
> Fill the "what I assumed" column *before* reading the page. That's the discipline that
> makes this table worth anything.
>
> The last column is what we're buying. A borrowed verb that behaves differently and is
> never flagged is worse than an unfamiliar verb, because unfamiliar words get looked up
> and familiar ones don't.
>
> Finally, one question: **while reading the `cli-tools/` pages, could you get anywhere
> else?** When a page named a Pantavisor concept, could you follow a link to its
> definition, or were you stranded? Count the outbound links in the section. Report the
> number.
>
> Report against `../../rubric.md`.
