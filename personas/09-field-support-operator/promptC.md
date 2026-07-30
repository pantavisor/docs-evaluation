# Persona 09 — Prompt C — Jargon audit

> Open `persona.md` from this folder alongside this file in a fresh session.
> Report against [`../../rubric.md`](../../rubric.md).

## Prompt C — Jargon audit

> Read only the troubleshooting and operations pages — `getting-started/troubleshooting/`
> and `getting-started/operate/`. **Stay out of the conceptual pages.** That restriction is
> the point: these pages must work standalone for someone who will never read the theory.
>
> Produce a table of **every term or command used without enough context to act on it**:
>
> | Term/command | First use (page URL) | Explained here? | What I'd have to go learn first | Could I run it at 2am? |
>
> The last column is the one that matters. It's not "is this defined somewhere" — it's
> "could a competent operator, mid-incident, with no background, use this line safely?"
>
> Flag with particular care:
>
> - **Any command handed over with no explanation of its output.** Being told to run
>   something and not knowing what a good result looks like is worse than nothing — you'll
>   read a bad result as fine.
> - **Anything destructive without a warning.** If a documented command could make things
>   worse and doesn't say so, that's automatically S1. At 2am, someone will run it.
>
> Report against `../../rubric.md`.
