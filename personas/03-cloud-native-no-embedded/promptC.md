# Persona 03 — Prompt C — Jargon audit

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt C — Jargon audit

> Read `https://docs.pantavisor.io/{VERSION}/meta-pantavisor/getting-started/start` and
> `https://docs.pantavisor.io/{VERSION}/meta-pantavisor/getting-started/develop`, following links a developer would
> plausibly follow.
>
> Produce a table of **every term used before it is defined**, judged strictly from your
> background:
>
> | Term | First use (page URL) | Ever defined anywhere I could reach? | What I assumed it meant | Was I right? |
>
> Your boundary makes two categories especially likely — hunt for both:
>
> - **Embedded vocabulary used as though universal**: bootloader, BSP, flashing, image
>   (in the *disk* sense, not the *Docker* sense), serial console, boot mode, carrier
>   board.
> - **Words that mean something different in your world.** This is the dangerous
>   category. `image`, `platform`, `container`, `volume`, `deploy`, `revision`, `state`
>   all have cloud meanings you'll import without noticing. Where you'd read one as its
>   cloud meaning and be wrong, log it — you'd never have known to look it up.
>
> **A term you guessed wrong about is worth ten you merely didn't know.** Report against
> `../../rubric.md`.
