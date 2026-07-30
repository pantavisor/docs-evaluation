# Persona 02 — Prompt C — Jargon audit

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt C — Jargon audit

> Read `https://docs.pantavisor.io/{VERSION}/meta-pantavisor/overview` and then follow the docs' own reading
> order for as long as a real engineer plausibly would.
>
> Produce a table of **every term used before it is defined**, judged strictly from your
> background:
>
> | Term | First use (page URL) | Ever defined anywhere I could reach? | What I assumed it meant | Was I right? |
>
> Two specific things to watch for, because your boundary is unusual:
>
> - **The Yocto vocabulary is invisible to the people who wrote these docs.** Words like
>   *recipe*, *layer*, *bitbake*, *sstate*, *KAS*, *machine*, *image* will be used as
>   though universal. Log every one. You genuinely do not know them.
> - **`image` and `machine` mean something in Buildroot too.** Where you'd read a Yocto
>   word as its Buildroot meaning, log it as a wrong-guess — those are the dangerous ones,
>   because you'd never know to look it up.
>
> **A term you guessed wrong about is worth ten you merely didn't know.** Report against
> `../../rubric.md`.
