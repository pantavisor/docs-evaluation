# Persona 06 — Prompt C — Jargon audit

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt C — Jargon audit

> Read `https://docs.pantavisor.io/{VERSION}/meta-pantavisor/overview/port` end to end, then
> `https://docs.pantavisor.io/{VERSION}/meta-pantavisor/overview/boot-flow`.
>
> Produce a table of **every term used before it is defined**, judged strictly from your
> background:
>
> | Term | First use (page URL) | Ever defined anywhere I could reach? | What I assumed it meant | Was I right? |
>
> Your boundary is unusual and makes two categories likely:
>
> - **Container vocabulary you genuinely lack.** You know Yocto cold, so `bitbake` and
>   `MACHINE` aren't gaps for you — but `container`, `LXC`, `image` (Docker sense),
>   `registry`, `object`, `revision`, `group` are. Don't skip them because they look like
>   words; you do not know them.
> - **`platform`.** Watch this one specifically. In your world it means an SoC family.
>   Track every use across the pages you read and record whether it's used consistently.
>   If it means different things in different places and nothing says so, that's a serious
>   finding — you'd wire it up wrong and lose a day.
>
> **A term you guessed wrong about is worth ten you merely didn't know.** Report against
> `../../rubric.md`.
