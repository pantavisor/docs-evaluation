# Persona 11 — Prompt C — Jargon audit

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt C — Jargon audit

> Read `getting-started/how-to-install/` end to end, including the board pages, then
> follow anything that touches identity or claiming.
>
> Produce a table of **every term used before it is defined**, judged strictly from your
> background:
>
> | Term | First use (page URL) | Ever defined anywhere I could reach? | What I assumed it meant | Line-stopping if I'm wrong? |
>
> The last column is yours alone: would being wrong about this term cost you units, a
> recall, or a re-spin of the station? Those rows are automatically S1 regardless of how
> well the run went otherwise.
>
> Hunt specifically for:
>
> - **`claim`, `challenge`, `device ID`, `token`** — authentication primitives being used
>   as product nouns. You don't know what they are, and they smell per-unit, which makes
>   them yours.
> - **Anything implying a network call.** You'll read past it if it's phrased as a
>   developer convenience. Look for it deliberately.
> - **`provisioning`** — you have a precise meaning for this word. Do the docs use it the
>   same way? If they mean "configuring software after the fact" and you read "injecting
>   per-unit identity at manufacture," that mismatch is expensive.
>
> Report against `../../rubric.md`.
