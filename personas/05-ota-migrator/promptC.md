# Persona 05 — Prompt C — Jargon audit

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt C — Jargon audit

> Read `https://docs.pantavisor.io/{VERSION}/meta-pantavisor/getting-started/migrate/mender` (or `.../migrate/rauc`), then
> follow it into whatever it points at — likely `https://docs.pantavisor.io/{VERSION}/pantavisor/overview/updates` and
> `https://docs.pantavisor.io/{VERSION}/pantavisor/overview/revisions`.
>
> Produce a table of **every term used before it is defined**, judged strictly from your
> background:
>
> | Term | First use (page URL) | Ever defined anywhere I could reach? | What I assumed it meant | Was I right? |
>
> Watch especially for the **false friends** — this is the heart of your run. Your field
> has a precise vocabulary, and these docs use some of the same words:
>
> - **`revision`** — you'll read this as "version number." Is it?
> - **`trail`**, **`step`** — new to you entirely.
> - **`object`** — you'll assume a file. Is it?
> - **`state`** — you'll read it as "device health/status." Is it?
> - **`atomic`**, **`rollback`**, **`commit`** — you have exact meanings for all three.
>   Do the docs use them the same way? Where they differ and don't say so, that's the
>   most dangerous kind of gap for you, because you'd deploy on a wrong understanding.
>
> **A term you guessed wrong about is worth ten you merely didn't know.** Report against
> `../../rubric.md`.
